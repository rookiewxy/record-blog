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

### 2024/9/20
1. 使用`unplugin-auto-import`不起作用
一定要在配置项中写dts，这样启动项目会自动生成`auto-imports.d.ts`文件，在引入tsconfig.json中的include中就可以了
```ts
autoImports({
        imports: ["react", "react-router-dom", "ahooks", { classnames: [["default", "c"]] }],
        dirs: ['src/*'],
        dts: "auto-imports.d.ts"
      })
```
