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
