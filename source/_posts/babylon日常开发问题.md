### 2024/5/11
1. 添加物体阴影，在threejs。开启阴影非常简单就是在灯光、场景、对应的`mesh`做一下`castShadow`的设置，以及被投影的物体设置一下`receiveShadows`就可以了，但是在babylon中首先需要使用ShadowGenerator创建一个对象在使用addShadowCaster添加对应的mesh，因为我的天空盒使用的是GridMaterial做的地面，但是GridMaterial不能接受shadow，在社区找到了这个[解决方案](https://forum.babylonjs.com/t/is-gridmaterial-unable-to-receive-shadows/27056)，使用NodeMaterial来做，以下是介绍

NodeMaterial提供了一种直观且灵活的方式来定义材质的外观和行为，使开发人员能够轻松地实现各种视觉效果。通过NodeMaterial，开发人员可以使用各种节点（如颜色、纹理、数学运算等）来构建自定义的着色器效果。这些节点可以连接在一起，形成一个图形化的着色器网络，每个节点代表着不同的功能或计算步骤。开发人员可以通过调整节点之间的连接和参数来实现所需的材质效果，而无需深入了解着色器语言。

代码示例
```tsx
// "I4DJ9Z"代表snippetId,此id代表的是服务器上的片段代码，因此，如果要做Grid相关的代码就要使用这个id，我个人觉得很有局限性，还有一点就是，使用了该方法，场景当中添加了light之后就会有一个巨大的焦点，不知道怎么去掉，因此这种方法不考虑
    let groundMaterial = await BABYLON.NodeMaterial.ParseFromSnippetAsync("I4DJ9Z", this.scene);
    
    groundMaterial.getBlockByName("Grid ratio").value = 1;
    groundMaterial.getBlockByName("Major unit frequency").value = 2;
    groundMaterial.getBlockByName("Diffuse color").value = new BABYLON.Color3(0.86, 0.85, 0.82);
    groundMaterial.getBlockByName("Line color").value = new BABYLON.Color3(0, 1, 1);
    
```

但是要实现mesh的阴影投射怎么办，我想的是使用一张网格贴图，不用gridMaterial了


### 2024/5/22
1. `GizmoManager`遇到的问题，如果有多个场景的情况，它只会在第一个场景生效，例如当一个场景组件实例化之后进行删除，在进行实例化，这个时候Gizmo就没有效果了，只会在初始实例化的时候有效果，这个时候需要使用`UtilityLayerRenderer`进行场景绑定
```ts
const gizmoManager = new BABYLON.GizmoManager(scene, undefined, new BABYLON.UtilityLayerRenderer(scene));
```
2. 在三维编辑器中一种常见的场景就是`gizmo`作用于模型、灯光、相机等，方便调试，在我attachToMesh的时候一切正常，正常情况就是我通过`gizmo`拖动一个模型改变他的位置，在我再次添加模型的时候（这里我做了数据缓存，如果是统一的模型不用重新创建而是在缓存中获取`meshes`）位置应该是模型原始的位置，因为light需要借助`UtilityLayerRenderer`和`LightGzimo`达到格式化的效果,所以需要使用`attachToNode`,但是现在就不正常，他的拖动会改变原始模型的位置，排查了好久以为是我某个地方修改到了原始的数据，解决方式是在长江`LightGizmo`的时候传一个cloneMesh，这样就不会影响
```tsx
const utilLayer = UtilityLayerRenderer.DefaultUtilityLayer;
this.lightGizmo = new BABYLON.LightGizmo(utilLayer);
this.lightGizmo.light = cloneMesh(light);


this.gizmo.attachToNode(this.gizmo.lightGizmo.attachedNode);
```


### 2024/6/27
1. babylonjs中渲染大量灯光的问题，默认他只支持4个光源，可以通过设置maxSimultaneousLights来改变光源数量，但是最多也就10个左右，超过就会报错黑屏，于是在网上看到了延迟着色的方法，一起来看一下这篇文章，https://learnopengl.com/Advanced-Lighting/Deferred-Shading

`Forward rendering` 和 `forward shading` 都是指一种渲染技术，通常可以互换使用。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)[3](https://unrealartoptimization.github.io/book/pipelines/forward-vs-deferred/)

简单来说，forward rendering 就是指在渲染每个物体的时候，立即计算光照效果。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)[3](https://unrealartoptimization.github.io/book/pipelines/forward-vs-deferred/) 这与另一种渲染技术“deferred rendering”形成对比，后者会先将场景中所有物体的几何信息和材质信息渲染到一个称为 G-buffer 的缓冲区中，然后再根据 G-buffer 中的信息计算光照效果。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)[3](https://unrealartoptimization.github.io/book/pipelines/forward-vs-deferred/)

Forward rendering 的优势在于：

* 效率更高，尤其是在光源数量较少的情况下。 [5](https://forums.unrealengine.com/t/why-use-forward-shading/1712026)
* 对硬件要求更低，可以兼容更老的显卡。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)
* 支持 MSAA (多重采样抗锯齿) 和 alpha2coverage (半透明物体抗锯齿)。 [5](https://forums.unrealengine.com/t/why-use-forward-shading/1712026)

而 Deferred rendering 的优势在于：

* 能够处理更多光源，尤其是在场景中有多个动态光源的情况下。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)
* 可以对灯光进行更灵活的处理，例如阴影和反射等效果。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)

最终，选择哪种渲染技术取决于具体项目的需要。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a) 如果你的游戏需要处理大量的动态光源，并且对硬件性能要求较高，那么 Deferred rendering 可能更适合。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a) 否则，Forward rendering 可以提供更简单、更高效的解决方案。 [2](https://gamedevelopment.tutsplus.com/forward-rendering-vs-deferred-rendering--gamedev-12342a)



**MSAA**

MSAA，即 **多重采样抗锯齿** (Multisample Anti-Aliasing)，是一种常用的抗锯齿技术，主要用于消除图形边缘的锯齿现象。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)

MSAA 的原理是：在渲染每个像素时，不是只采样一次，而是对该像素进行多次采样，然后将这些采样结果进行平均，从而得到一个更平滑的像素颜色值。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)

以下是 MSAA 的工作原理：

1. **渲染场景：** 场景被渲染到一个称为“多重采样缓冲区” (Multisample Buffer) 的特殊缓冲区中。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)
2. **多重采样：** 在渲染过程中，每个像素会被分成多个子像素，每个子像素都进行单独的采样。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)
3. **合并采样：** 所有子像素的采样结果被合并成一个最终的像素颜色值。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)
4. **显示：** 最终的像素颜色值被显示到屏幕上。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)

MSAA 的优点：

* **能够有效消除锯齿现象，** 使得图形边缘更平滑。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)
* **对性能影响较小，** 尤其是在高分辨率屏幕上。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)

MSAA 的缺点：

* **无法消除所有锯齿，** 尤其是那些非常小的几何细节，以及那些在屏幕上移动速度非常快的物体。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)
* **会增加渲染负担，** 尤其是在高采样率的情况下。 [1](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)

总的来说，MSAA 是一种有效且常用的抗锯齿技术，能够显著改善图形质量。 


### 2024/9/19
1. 控制场景中的物体大小不受相机缩放的影响，方法如下
```ts
scene.registerBeforeRender(() => {
      const d = 0.1 * Math.max(this.scene.getActiveCamera().position.subtract(mesh.position).length(), 0.1)
      mesh.scaling = new Vector3(d, d, d);
})
```

### 2024/9/21
1. 以第一人称的形式在室内进行移动，并且进行碰撞检测，刚开始设置了`ellipsoid`还是会有穿透效果，感觉画面有一部分是破损，是因为要设置camera.minZ，并不是`ellipsoid`的问题，刚开始一直以为是它的问题,并且在移动
过程中，现在相机的y是不固定的，还需在每帧重新设置y的坐标
```ts
camera.minZ = 0.05;
camera.ellipsoid = new BABYLON.Vector3(0.5, 0.8, 0.5);
camera.ellipsoidOffset = new BABYLON.Vector3(0, 0.5, 0);

this.scene!.onBeforeRenderObservable.add(() => {
   camera.position.y = 1.5; // 保持固定高度
});
```

### 2024/9/25
1. arcRotateCamera切换到universalCamera，也就是第三人称切换到第一人称，其实不用太多的计算，设置对应的position和rotation就行
```ts
    camera.position.copyFrom(arcRotateCamera.position);
    camera.rotation.copyFrom(arcRotateCamera.rotation)
```
2.在使用babylon构建场景的过程中，有的电脑设备比较老，报下图的错误，需降级为webgl1的版本
 ![alt text](image-6.png)


 ### 2024/9/26
 1. 还是25号 VAO报错的问题，降级为webgl1还是有时候报错，并且设备其实也是支持webgl2的语法，一直觉得是模型贴图的问题，但是换了几种类型的材质贴图都可以正常加载，只不过大小都很小，所以最后尝试了白膜加贴图的方式，在笔记本电脑上也可以加载了，但是也遇到了不少问题
 2. 贴图是反的
 3. 光照效果不够
 因为建模人员都是在sandbox上进行测试的，它里面的贴图都是pbrMaterial的类型，而我使用的是标准贴图，最后我也换成了pbr材质，贴图是反的，可以设置vSCale为-1，绕y轴旋转，还有一些透明材质就不需要贴图了,但是现在这种方式是比较蠢的因为我是手动匹配的，最好的方式是导出一个文件夹，其中有白膜模型其中引入了相对根目录的材质图片，我直接加载这个模型就可以了，之前在sandbox上试了好像做不到分离贴图，还是要使用专业的建模软件，例如`blender`
 ```ts
import * as BABYLON from "babylonjs";
import { GLTFFileLoader } from "babylonjs-loaders";

BABYLON.SceneLoader.RegisterPlugin(new GLTFFileLoader());


const excludedMaterialMesh = [
    "__root__",
    "SM_ZhanGuan_A032A",
    "SM_ZhanGuan_A038A",
    "SM_ZhanGuan_A037A",
    "SM_ZhanGuan_A037A_1",
    "SM_ZhanGuan_A043A",
]

class Model {
    static async load(scene: BABYLON.Scene, onProgress?: (event: BABYLON.ISceneLoaderProgressEvent) => void) {
        const res = await BABYLON.SceneLoader.ImportMeshAsync("", "model.glb", "", scene, onProgress);

        res.meshes.forEach((mesh) => {
            mesh.checkCollisions = true;
            if (!excludedMaterialMesh.includes(mesh.name)
            ) {
                const material = new BABYLON.PBRMaterial("", scene);
                material.roughness = 0.5;
                material.metallic = 0;

                const texture = new BABYLON.Texture(`material/${mesh.name}.png`, scene);
                texture.vScale = -1;
                material.albedoTexture = texture;
                texture.hasAlpha = true
                mesh.material = material;
            }
        });
        return res;
    }
}

export { Model };

 ```


 ### 2024/9/29
1. 最近一直在做三维展厅的功能，以第一人称的视角进行移动，有一个功能是需要知道当前移动到了展厅的那个区域，其实看到了射线检测，尝试过但是射线的长度不好控制，于是根据每个区块的边界值来判断，获取每个mesh的getBoundingInfo中的minimumWorld和maximumWorld进行camera.position的判断，一定是世界坐标系进行比较，因为camera.position就是世界坐标系
```ts
function checkCameraPosition(camera: BABYLON.UniversalCamera) {
    const cameraPosition = camera.position

    for (const roomKey in rooms) {
      const room = (rooms as Record<string, IRegionalItem>)[roomKey];
      const minimumWorld = room.minimumWorld;
      const maximumWorld = room.maximumWorld;

      const isInside = (
        cameraPosition.x >= minimumWorld.x && cameraPosition.x <= maximumWorld.x &&
        cameraPosition.y >= minimumWorld.y && cameraPosition.y <= maximumWorld.y &&
        cameraPosition.z >= minimumWorld.z && cameraPosition.z <= maximumWorld.z
      );

      if (isInside) {
        return roomKey;
      }
    }

    returnnull;
  }
```

2. 9/26号的模型加载问题，可以使用`blender`到处`babylon`格式的模型，这样可以不用手动添加贴图，而是在`babylon`格式中以相对路径的方法引入贴图

### 2024/10/10
1. [计算绘制面在哪一些区域](https://playground.babylonjs.com/#2FDQT5#2826)

### 2024/10/18
[演示效果](https://xtspace.cc:10092/juliusuo-web/)
#### icon创建
因为之前都是使用的是动态纹理贴图的方式，所以最开始也是使用这种方式，但是图片会有锯齿状并且文字贴图中的文字也非常的不清晰，特别是文字小的时候，最后使用`GUI3D`来创建图标,代码如下,有一种情况上图标旁边还有一个`tooltip`的效果，这个时候，其实两者需要用的都是一个坐标，但是`tooltip`要像左边偏一点，这个时候需要修改的是`linkOffsetX`，而不是`x`的坐标，因为`GUI3D`在场景中放大缩小会有偏移的感觉，如果调整`x`，在放大的时候两者就会有偏移
```ts
    // 创建图片
    const mesh = BABYLON.MeshBuilder.CreatePlane("", { width: 0.01, height: 0.01 });
    mesh.visibility = 0;

    const icon = new GUI.Image(data.id, data.icon);
    icon.width = width + "px";
    icon.height = height + "px";
    this.advancedDynamicTexture = this.scene.getTextureByName("GUI") as GUI.AdvancedDynamicTexture;
    this.advancedDynamicTexture.addControl(icon);

    icon.linkWithMesh(mesh);
    mesh.position.set(data.x, data.y, data.z);

    // 创建文本
    const textBlock = new GUI.TextBlock(data.id, data.name);
    textBlock.fontSize = fontSize;
    textBlock.width = width + "px";
    textBlock.height = height + "px";
    textBlock.color = "white";
    textBlock.linkOffsetX = linkOffsetX;
    textBlock.linkOffsetY = linkOffsetY;
    textBlock.textHorizontalAlignment = GUI.Control.HORIZONTAL_ALIGNMENT_CENTER;
    textBlock.textVerticalAlignment = GUI.Control.VERTICAL_ALIGNMENT_CENTER;
    textBlock.fontWeight = isBold ? "bold" : "normal"; 
    this.advancedDynamicTexture?.addControl(textBlock);
    // mesh表示你要设置的背景图mesh对象
    textBlock.linkWithMesh(mesh);
```
#### Gui事件绑定
因为之前使用的是动态贴图的方式，所以可以直接使用`mesh.actionManager`的方式注册事件,也可以直接动过`scene.getMeshById`去找到对应的`mesh`,但是使用`gui`之后，`scene`中的`meshes`是有没有的，但是可以通过上面的`this.advancedDynamicTexture`中的`rootContainer.children``可以获取到有哪些gui，触发事件的方式也变化了，使用onPointerUpObservable`
```ts
// mesh绑定事件
const mesh = scene.getMeshById("mesh")
mesh.actionManager = new BABYLON.ActionManager(mesh.getScene());
mesh.actionManager.registerAction(new BABYLON.ExecuteCodeAction(
    BABYLON.ActionManager.OnPickTrigger,
    ()=>{}
));
// gui绑定事件
const controls = this.gui.rootContainer.children.filter(d => d.name == name || d.name?.startsWith(name))
controls.forEach(d=>{
  d.onPointerUpObservable.add(() => {});
})
```

#### 相机动画
```ts
import * as BABYLON from "babylonjs";
import { get } from 'lodash-es'

class Camera {
    camera: BABYLON.ArcRotateCamera;

    constructor(scene: BABYLON.Scene) {
        const camera = new BABYLON.ArcRotateCamera(
            "camera",
            BABYLON.Tools.ToRadians(-90),
            BABYLON.Tools.ToRadians(75),
            200,
            new BABYLON.Vector3(0, 0, 0),
            scene
        );
        camera.minZ = 0.1;
        camera.lowerRadiusLimit = 3;
        camera.upperRadiusLimit = 600;
        camera.attachControl();
        this.camera = camera;
    }

    /**
     * 相机视角切换（带动画过渡）
     * @param beta  沿x轴旋转
     * @param alpha 沿y轴旋转
     * @param radius 相机半径
     * @param target 目标点（可选）
     * @param duration 动画持续时间（可选）
     */
    switch(beta: number, alpha: number, radius: number, target?: BABYLON.Vector3, duration: number = 60) {
        target && this.camera.setTarget(target);

        // 创建动画
        const alphaAnimation = this.createAnimation("alpha", BABYLON.Tools.ToRadians(alpha), duration);
        const betaAnimation = this.createAnimation("beta", BABYLON.Tools.ToRadians(beta), duration);
        const radiusAnimation = this.createAnimation("radius", radius, duration);

        this.camera.animations = [alphaAnimation, betaAnimation, radiusAnimation];
        this.camera.getScene().beginAnimation(this.camera, 0, duration, false);
    }

    switchDefault(duration: number = 60) {
        this.switch(75, -90, 200, new BABYLON.Vector3(0, 0, 0), duration);
    }

    /**
     * 创建动画
     * @param property 动画属性
     * @param endValue 终止值
     * @param duration 动画持续时间
     * @returns 返回创建的动画对象
     */
    createAnimation(property: string, endValue: number, duration: number): BABYLON.Animation {
        const animation = new BABYLON.Animation(
            `${property}Animation`,
            property,
            60,
            BABYLON.Animation.ANIMATIONTYPE_FLOAT,
            BABYLON.Animation.ANIMATIONLOOPMODE_CONSTANT
        );

        const keys = [
            { frame: 0, value: get(this.camera, property) },
            { frame: duration, value: endValue }
        ];

        animation.setKeys(keys);
        return animation;
    }
}

export { Camera };

```
#### 开门动画
门的mesh在建模的使用中心轴要放在门把手的另一边
```ts
export const registerDoorAnimation = ({
  scene,
  meshRef,
  meshId,
}: {
  scene: BABYLON.Scene;
  meshRef: Mesh;
  meshId: string;
}) => {
  const door = meshRef.getMesh(meshId)! as AbstractMesh;
  const startRotation = door.rotation.z;
  const endRotation = door._isOpen ? 0 : Math.PI / 2;

  const doorAnimation = new BABYLON.Animation(
    "doorAnimation",
    "rotation.z",
    30,
    BABYLON.Animation.ANIMATIONTYPE_FLOAT,
    BABYLON.Animation.ANIMATIONLOOPMODE_CONSTANT
  );

  const keys = [
    { frame: 0, value: startRotation },
    { frame: 30, value: endRotation },
  ];

  doorAnimation.setKeys(keys);

  door.animations = [];
  door.animations.push(doorAnimation);
  door._isOpen = !door._isOpen;

  scene.beginAnimation(door, 0, 30, false);
};
```

#### 体积雾和镜像
```ts
scene.fogMode = BABYLON.Scene.FOGMODE_LINEAR; // 不写没效果
scene.fogColor = new BABYLON.Color3(10 / 255, 82 / 255, 140 / 255);
scene.fogDensity = 0.01;

const mirror = new BABYLON.MirrorTexture("mirror", 1024);
mirror.mirrorPlane = new BABYLON.Plane(0, -1, 0, 0);
mirror.level = .1;
const mirrorMesh = this.scene?.meshes ?? []
mirror.renderList?.push(...mirrorMesh);
(plane.material as BABYLON.StandardMaterial).reflectionTexture = mirror
```