# Babylon踩坑记录（一）

### 一、加载glb（gltf）文件
安装官方playground的示例，是不能直接运行的，写了加载的代码也不会报错，只是在返回的结果中显示空数组，以为是路径问题，尝试了很多方法，`append`和`ImportMesh`以为是方法的问题，后面下载了babylonjs-loaders在文件中导入也没有效果，网上很多案例也是跟官网差不多，没什么参考价值，后来发现直接导入不行，还要注册！！！，代码如下
```ts
BABYLON.SceneLoader.RegisterPlugin(new GLTFFileLoader());
```

### 二、webgpu和webgl性能对比
写[demo](https://github.com/rookiewxy/webgpu-demo.git)的主要意图就是想对比一下两者之前的性能差异,但是我陷入了一个误区，就是看fps，后来发现其实加载大几百M的模型两者在fps上也是没有什么区别的，都差不多，查阅资料之后发现影响fps的因素主要分为cpu传递Draw Call的时间和GPU的计算时间，我加载的模型并不复杂只是大，所以Draw Call都是差不多的，再加上GPU端都是通过光栅化管线来渲染，所以两者在GPU时间上都是一样的，webgpu胜在有缓存，当数据不断增加时，效果明显

babylonjs自身提供了监测性能的api，主要用到了两个
`EngineInstrumentation`可以获取GPU的状态
`SceneInstrumentation`监测和分析场景的性能表现，例如渲染时间、帧率、渲染调用次数等

代码如下
```ts
 this.engine?.runRenderLoop(() => {
      text1.text =
        "current frame time (GPU): " +
        (this.instrumentation.gpuFrameTimeCounter.current * 0.000001).toFixed(
          2
        ) +
        "ms";
      text1.text +=
        "\r\naverage frame time (GPU): " +
        (this.instrumentation.gpuFrameTimeCounter.average * 0.000001).toFixed(
          2
        ) +
        "ms";
      text1.text +=
        "\r\ntotal shader compilation time: " +
        this.instrumentation.shaderCompilationTimeCounter.total.toFixed(2) +
        "ms";
      text1.text +=
        "\r\naverage shader compilation time: " +
        this.instrumentation.shaderCompilationTimeCounter.average.toFixed(2) +
        "ms";
      text1.text +=
        "\r\ncompiler shaders count: " +
        this.instrumentation.shaderCompilationTimeCounter.count;
      text1.text +=
        "\r\nDraw calls: " + sceneInstrumentation.drawCallsCounter.count;
      text1.text +=
        "\r\nframe time: " + sceneInstrumentation.frameTimeCounter.count;
      this.scene.render();
    });
```

### 三、TextBlock
这个问题花了大几个小时，直接按照官网使用也是不行的，后面下载了`babylonjs-gui`插件,就报`Invalid engine. Unable to create a canvas`错误,一脸懵逼，网上找的资料也都是这么写的呀，怎么我就报错了，后来以为是版本问题，因为现在我用的版本是`6.18.0`，是不是我用的太新了，后面我退到了5的测试版，就可以了，到了5的正式版本又会报`camera`的错误,但是我不可能为了一个`TextBlock`就用5的测试版或者更早的版本，因为v6做了很多的优化，并且我也需要使用webgpu的功能，后面在网上早到了一个案例，使用的是在html中加载https://preview.babylonjs.com/babylon.js的方式，这个明明也是最新版本，为啥别人的就不会报错呢？这让我意识到，可能是我的`babylonjs`全局加载的时间有问题,因为我是封装了一个工具函数，在哪里导入的`babylonjs`，在React组件里面进行调用，于是我将`babylon`直接放在`React`组件中注册，工具函数中不进行导入，结果就可以了！！！由于之前的v5测试版可以，给我造成了是版本问题的错觉，一直在纠结版本，不过v5版本之后肯定做了更新，具体什么更新不得而知，终于可以和官网的`playground`一样使用`GUI`了代码如下
```ts
const advancedTexture =
      BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI(
        "UI",
        true,
        this.scene
      );

    const stackPanel = new BABYLON.GUI.StackPanel();
    stackPanel.width = "25%";
    stackPanel.top = 10;
    stackPanel.verticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_TOP;
    advancedTexture.addControl(stackPanel);

    text1 = new BABYLON.GUI.TextBlock();
    text1.color = "white";
    text1.height = "180px";
    text1.fontFamily = "RuslanDisplay";
    text1.textVerticalAlignment = BABYLON.GUI.Control.VERTICAL_ALIGNMENT_TOP;

    stackPanel.addControl(text1);
```
