#### babylon的camera封装
1. 设计思路
创建一个`Camera`的父类，以及对应四种`Camera`类型的子类进行集成，`camera`中我们需要二次封装4中相机，依次继承`babylon`中的`camera`，然后再单独写一个`camera`的基类进行合并，以防有相同的属性和方法，在该方法中使用`Babylon`创建对应的`Camera`，进行返回，在创建一个`CameraManager`的类，进行管理，因为二次封装的意义是方便进行调用，通过`Manager`只用传入相机类型以及对应的参数，就可以创建一个相机，所以`CameraManager`中依赖了四种`camera`类型的子类，当然`CameraManager`不仅仅用来做创建，还可以用来协调对象的逻辑，以及销毁,因为销毁方法，`babylon`自身以及提供所以可以直接在返回的实例当中进行调用
`Manager`代码示例如下
```ts

interface CameraConstructor<T> {
  new (): ICamera<T>;
}

type CameraParams<T extends `${CameraEnum}`> = T extends "arcRotateCamera"
  ? IArcRotateCamera
  : T extends "followCamera"
  ? IFollowCamera
  : T extends "freeCamera"
  ? IFreeCamera
  : T extends "universalCamera"
  ? IUniversalCamera
  : never;

const cameraClasses: Record<
  `${CameraEnum}`,
  CameraConstructor<CameraParams<`${CameraEnum}`>>
> = {
  [CameraEnum.ARCROTATE]: ArcRotateCamera,
  [CameraEnum.FOLLOW]: FollowCamera,
  [CameraEnum.FREE]: FreeCamera,
  [CameraEnum.UNIVERSAL]: UniversalCamera,
};

export class CameraManager<T extends `${CameraEnum}`> {
  camera: CameraType<T>;

  constructor(type: T, params?: CameraParams<T>) {
    this.createCamera(type, params);
  }

  createCamera(type: T, params?: CameraParams<T>) {
    this.camera = new cameraClasses[type](params);
    if (!this.camera) {
      throw new Error(`Unsupported camera type: ${type}`);
    }
  }

  getCamera() {
    return this.camera;
  }

  dispose() {
    this.camera.dispose();
  }
}


```

#### babylon的light封装
同`camera`

#### babylon的gameobject封装
`gameobject`我在`babylon`里面分成了两种类型，第一种是形状类型，第二种是模型加载，
首先形状有很多种，这里我们选择比较常用的三种进行封装，添加一个`shapeManager`类

```ts
export class ShapeManager<T extends `${ShapeEnum}`>{

  createShape(type: T, params: IShapeParams){
    const { name, options, scene } = params;
    const createFunction: CreateFunction = {
      [ShapeEnum.BOX]: BABYLON.MeshBuilder.CreateBox,
      [ShapeEnum.SPHERE]: BABYLON.MeshBuilder.CreateSphere,
      [ShapeEnum.GROUND]: BABYLON.MeshBuilder.CreateGround,
    }[type];

    if (createFunction) {
      return createFunction(name, options, scene);
    } else {
      throw new Error("Unsupported shape type");
    }
  }
}

```
其次就是模型加载，`Babylon`中的模型加载方式非常多，没有考虑在针对于他们进行二次封装，所以直接返回`loader`方法，让客户端自己调用


完成了以上两种，就是整个`gameobject`的封装了，一个游戏对象需要创建功能也需要更新功能，和销毁，这里说一下更新功能，目前我会设计一个`Task`类，专门负责工作流，其中有一个方法是`tick`就是负责更新的，那么更新无非就是属性的更新，在传入的属性中可能有在`gameobject`中存在也可能不存在，那么不存在的属性，我会进行返回
```ts
// task.ts
export class Task implements ITask {
  tick<T>(
    params: Properties,
    handler?: (instance: T, fieldErrors: Properties) => void
  ): void {
    const fieldErrors = {} as Properties;
    for (let p in params) {
      if (Reflect.has(this, p)) {
        Reflect.set(this, p, params[p]);
      } else {
        fieldErrors[p] = params[p];
      }
    }
    handler?.(this as unknown as T, fieldErrors);
  }
}

```

```ts
import * as BABYLON from "babylonjs"
import { GLTFFileLoader } from "babylonjs-loaders";

BABYLON.SceneLoader.RegisterPlugin(new GLTFFileLoader());

export const ModelLoader = BABYLON.SceneLoader;
export class GameObject {
  private lastGameObj: BABYLON.Mesh;
  create(type: `${ShapeEnum}`, params: IShapeParams) {
  }

  load() {
  }

  tick(
    gameObj: BABYLON.Mesh & ITask,
    params: Properties,
    handler?: (params: this, fieldErrors: Properties) => void
  ) {
    if (gameObj !== this.lastGameObj) {
      mixin(gameObj, Task);
      this.lastGameObj = gameObj;
    }
    gameObj?.tick(params, handler);
  }
}
```

#### 渐进加载的封装
实现原理，加载camera视口的模型，监听camera的方向和范围的变化，加载模型
在scene中添加获取视锥面的方法
```ts
  onFrustumPlanesObserver(callback: (frustumPlanes: BABYLON.Plane[]) => void) {
    const camera = this.activeCamera;
    return camera?.onViewMatrixChangedObservable.add(() => {
      const viewMatrix = camera.getViewMatrix(); // 视口矩阵
      const projectionMatrix = camera.getProjectionMatrix(); // 投影矩阵
      const frustumPlanes = BABYLON.Frustum.GetPlanes(
        viewMatrix.multiply(projectionMatrix) // 视口矩阵乘以投影矩阵得到变换矩阵
      );
      callback(frustumPlanes);
    });
  }
```
在客户端进行调用,其中`modelsToLoad`是需要加载的模型列表，`isModelInFrustum`方法判断模型是否在当前视口
```ts
 scene.onFrustumPlanesObserver((frustumPlanes) => {
    this.loadModelsInFrustum(frustumPlanes);
  });

  loadModelsInFrustum(frustumPlanes) {
    // 遍历所有待加载的模型
    for (const model of modelsToLoad) {
      // 检查模型是否在视野内
      if (this.scene.isModelInFrustum(model.position, frustumPlanes)) {
        if (!inFrustumModels.includes(model.name)) {
          this.loadModel(model.name);
          inFrustumModels.push(model.name);
        }
      }
    }
  }

  
  loadModel(model: string) {
    BABYLON.SceneLoader.ImportMeshSync(
      "",
      "xxx",
      model,
      this.scene,
      function () {
        // 模型加载完成后执行一些操作
      },
    );
  }
```

```ts
// scene.js
  isModelInFrustum(position: Vector3, frustumPlanes: BABYLON.Plane[]) {
    return BABYLON.Frustum.IsPointInFrustum(position, frustumPlanes);
  }
```