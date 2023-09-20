#### babylon的camera封装
1. 设计思路
创建一个`Camera`的父类，以及对应四种`Camera`类型的子类进行集成，通过公用的`interface`进行`implement`，例如都拥有`createCamera`方法，在该方法中使用`Babylon`创建对应的`Camera`，进行返回，在创建一个`CameraManager`的类，进行管理，因为二次封装的意义是方便进行调用，通过`Manager`只用传入相机类型以及对应的参数，就可以创建一个相机，所以`CameraManager`中依赖了四种`camera`类型的子类，当然`CameraManager`不仅仅用来做创建，还可以用来协调对象的逻辑，以及销毁,因为销毁方法，`babylon`自身以及提供所以可以直接在返回的实例当中进行调用
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
  private camera: Camera;

  constructor(type: T, params: CameraParams<T>) {
    this.createCamera(type, params);
  }

  createCamera(type: T, params: CameraParams<T>) {
    const CameraClass = new cameraClasses[type]();
    if (CameraClass) {
      this.camera = merge(CameraClass.createCamera(params), CameraClass);
    } else {
      throw new Error(`Unsupported camera type: ${type}`);
    }
  }

  getCamera() {
    return this.camera;
  }
}

```