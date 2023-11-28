### babylon在BI中的实践

#### 一、模型的修改
在三维场景中必不可少的就是模型的修改，我的实现方式是，初始化加载模型，加载过的模型进行缓存，如果再次加载相同的模型，直接从缓存中获取，再搭配`scene`的`onPointerMove`和`onPointerDown`事件实现，在搭配`@xt-plugins/fusion3D`中的task就行mesh的修改，一切顺利进行，但是出现了一个问题就是当我修改了一个模型的材质之后，再次加载模型时，是修改过的，也就是说，他们是相互影响的不是相同独立，但是我每次添加的模型其实都进行`clone`操作，以及在我修改他们的位置、旋转属性的时候，他们是相互独立的，但是修改材质就不行，原来`clone`之后的模型他们的材质依然共享，所以他们的材质也需要`clone`达到深克隆的效果，代码如下
```ts
 function cloneMesh(mesh: AbstractMesh) {
        const clonedModel = mesh.clone('', null);
        const childMeshes = clonedModel?.getChildMeshes();

        childMeshes?.forEach((m: AbstractMesh) => {

            if (m.material) {
                const clonedMaterial = m.material.clone(``);
                m.material = clonedMaterial;
            }
        });

        return clonedModel
    }
```