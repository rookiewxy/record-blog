

# Threejs主要部分

## 1. 概述


Threejs是什么
- Three.js是一款基于JavaScript的3D图形库，用于创建和显示动态的3D图形，它可以在WebGL的基础上进行封装，使得开发者可以更加方便地使用WebGL进行3D图形的开发。下面是一个简单的Three.js的例子：

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Three.js Example</title>
        <style>
            body { margin: 0; }
            canvas { width: 100%; height: 100% }
        </style>
    </head>
    <body>
        <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
        <script>
            var scene = new THREE.Scene();
            var camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
            var renderer = new THREE.WebGLRenderer();
            renderer.setSize( window.innerWidth, window.innerHeight );
            document.body.appendChild( renderer.domElement );
            var geometry = new THREE.BoxGeometry();
            var material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
            var cube = new THREE.Mesh( geometry, material );
            scene.add( cube );
            camera.position.z = 5;
            var animate = function () {
                requestAnimationFrame( animate );
                cube.rotation.x += 0.01;
                cube.rotation.y += 0.01;
                renderer.render( scene, camera );
            };
            animate();
        </script>
    </body>
</html>
```

这个例子中创建了一个场景、一个相机、一个渲染器、一个立方体和一个动画函数，最后将渲染器的输出添加到了页面中。

Threejs的历史
- Three.js的历史：

  Three.js是由Ricardo Cabello在2010年创建的一个开源JavaScript 3D库。它的目标是使创建复杂的、高性能的3D动画变得更加容易。随着WebGL技术的发展，Three.js变得越来越受欢迎，并且得到了广泛的应用。目前，它已经成为了WebGL的一个重要组成部分，成为了许多3D应用程序的首选库之一。

Threejs的优势- threejs的优势：
  - 可以在浏览器中渲染3d图形，无需安装插件，支持多种浏览器；
  - 支持多种3d模型格式，包括obj、fbx、collada等；
  - 提供了丰富的材质和光源，可以创建各种真实的3d场景；
  - 支持多种相机类型，包括****和正交相机等；
  - 可以使用webgl渲染器，提高渲染性能；
  - 提供了大量的示例和文档，方便开发者学习和使用。

## 2. 基本概念


场景
- 场景

```javascript
// 创建场景
var scene = new THREE.Scene();

// 添加网格到场景
var geometry = new THREE.BoxGeometry(1, 1, 1);
var material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
var cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// 渲染场景
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
renderer.render(scene, camera);
```

相机
- 相机类型

  ```javascript
  // ****
  const camera = new three.perspectivecamera(75, window.innerwidth / window.innerheight, 0.1, 1000);

  // 正交相机
  const camera = new three.orthographiccamera(-window.innerwidth / 2, window.innerwidth / 2, window.innerheight / 2, -window.innerheight / 2, 1, 1000);
  ```

- 相机属性

  ```javascript
  camera.position.set(0, 0, 5); // 相机位置
  camera.lookat(0, 0, 0); // 相机看向的位置
  ```

渲染器
- 渲染器是Three.js中最重要的组件之一，它能够将3D场景渲染成2D图像或者视频。
- 创建渲染器实例的代码如下：

```
var renderer = new THREE.WebGLRenderer();
```

- 渲染器的一些常用属性和方法：

| 属性/方法 | 描述 |
| --- | --- |
| setSize(width, height) | 设置渲染器的输出画布的大小 |
| setClearColor(color, alpha) | 设置渲染器的背景颜色 |
| setPixelRatio(ratio) | 设置渲染器的像素比 |
| render(scene, camera) | 渲染场景 |
| dispose() | 释放渲染器占用的内存 |

- 示例代码：

```javascript
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0x000000, 1.0);
document.body.appendChild(renderer.domElement);

var geometry = new THREE.BoxGeometry(1, 1, 1);
var material = new THREE.MeshBasicMaterial({color: 0xff0000});
var cube = new THREE.Mesh(geometry, material);

var scene = new THREE.Scene();
scene.add(cube);

var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 5;

function animate() {
    requestAnimationFrame(animate);
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render(scene, camera);
}

animate();
```

材质
- 材质

  ```javascript
  // 创建一个基本材质
  const material = new THREE.MeshBasicMaterial({
      color: 0xff0000, // 红色
      wireframe: true // 显示线框
  });

  // 创建一个 Lambert 材质
  const material = new THREE.MeshLambertMaterial({
      color: 0xff0000 // 红色
  });

  // 创建一个 Phong 材质
  const material = new THREE.MeshPhongMaterial({
      color: 0xff0000, // 红色
      specular: 0xffffff, // 高光颜色
      shininess: 100 // 高光强度
  });

  // 创建一个标准材质
  const material = new THREE.MeshStandardMaterial({
      color: 0xff0000, // 红色
      roughness: 0.5, // 粗糙度
      metalness: 0.5 // 金属度
  });
  ```
  
  或者使用一个已经存在的材质：
  
  ```javascript
  const material = new THREE.MeshBasicMaterial();
  material.copy(existingMaterial);
  ```

几何体
- 立方体

  ```javascript
  var geometry = new THREE.BoxGeometry(1, 1, 1);
  var material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  var cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
  ```

- 球体

  ```javascript
  var geometry = new THREE.SphereGeometry(0.5, 32, 32);
  var material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
  var sphere = new THREE.Mesh(geometry, material);
  scene.add(sphere);
  ```

- 圆柱体

  ```javascript
  var geometry = new THREE.CylinderGeometry(0.5, 0.5, 1, 32);
  var material = new THREE.MeshBasicMaterial({ color: 0xff00ff });
  var cylinder = new THREE.Mesh(geometry, material);
  scene.add(cylinder);
  ```

- 圆锥体

  ```javascript
  var geometry = new THREE.ConeGeometry(0.5, 1, 32);
  var material = new THREE.MeshBasicMaterial({ color: 0x00ffff });
  var cone = new THREE.Mesh(geometry, material);
  scene.add(cone);
  ```

- 平面

  ```javascript
  var geometry = new THREE.PlaneGeometry(1, 1);
  var material = new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide });
  var plane = new THREE.Mesh(geometry, material);
  scene.add(plane);
  ```

灯光
- 灯光类型

  | 类型 | 描述 |
  | --- | --- |
  | AmbientLight | 环境光 |
  | DirectionalLight | 方向光 |
  | HemisphereLight | 半球光 |
  | PointLight | 点光源 |
  | SpotLight | 聚光灯 |

- 灯光属性

  | 属性 | 描述 |
  | --- | --- |
  | color | 光的颜色 |
  | intensity | 光的强度 |
  | distance | 光的距离 |
  | angle | 光的角度 |
  | penumbra | 光的锐度 |
  | decay | 光的衰减率 |

控制器- 控制器：

  ```javascript
  // 创建一个 OrbitControls 控制器
  const controls = new OrbitControls(camera, renderer.domElement);

  // 设置控制器的属性
  controls.enableDamping = true; // 开启阻尼效果
  controls.dampingFactor = 0.05; // 阻尼系数
  controls.autoRotate = true; // 开启自动旋转
  controls.autoRotateSpeed = 2.0; // 自动旋转速度
  controls.enableZoom = false; // 禁用缩放
  controls.enablePan = false; // 禁用平移
  ```
  - 说明：控制器是用来控制相机在场景中移动和旋转的工具，可以通过控制器来实现用户与场景的交互。Three.js 中提供了多种类型的控制器，包括 OrbitControls、TrackballControls 等，可以根据实际需求选择不同的控制器。在上面的示例中，我们创建了一个 OrbitControls 控制器，并设置了一些属性来控制控制器的行为。

## 3. Threejs主要组件
### 3.1 渲染器组件


WebGLRenderer
- WebGLRenderer是three.js中用于渲染3D场景的组件之一。
- 它使用WebGL作为底层渲染API，并提供了很多有用的功能，如阴影、反锯齿等。
- WebGLRenderer的使用非常简单，只需要创建一个实例并将其连接到HTML页面中的元素中即可。
- 以下是创建WebGLRenderer实例的示例代码：

```javascript
var renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );
```

- 上述代码将创建一个WebGLRenderer实例，并将其连接到HTML页面中的body元素中。同时，设置了渲染器的大小为窗口的大小。

CSS3DRenderer
- CSS3DRenderer是一种基于CSS3D的渲染器，可以在3D场景中呈现DOM元素。它可以用于创建具有2D和3D元素的交互式Web应用程序。

  ```javascript
  // 创建CSS3DRenderer
  const renderer = new THREE.CSS3DRenderer();
  
  // 设置渲染器尺寸
  renderer.setSize( window.innerWidth, window.innerHeight );
  
  // 将渲染器添加到DOM中
  document.body.appendChild( renderer.domElement );
  ```

SVGRenderer- SVGRenderer是一个渲染器组件，用于将场景渲染到SVG格式的画布上。

```javascript
// 创建SVGRenderer
const renderer = new THREE.SVGRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );

// 创建场景和相机
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
camera.position.z = 5;

// 创建几何体和材质
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
const cube = new THREE.Mesh( geometry, material );
scene.add( cube );

// 渲染场景
function animate() {
    requestAnimationFrame( animate );
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render( scene, camera );
}
animate();
```

### 3.2 场景组件


Scene
- Scene是three.js中最基本的组件之一，它是一个容器，可以用来存放所有的物体、灯光和摄像机。
- 可以通过以下代码创建一个场景对象：

  ```javascript
  const scene = new THREE.Scene();
  ```

- 可以向场景中添加物体、灯光和摄像机等组件。例如，以下代码可以向场景中添加一个立方体：

  ```javascript
  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
  ``` 

- 在渲染场景之前，需要创建一个摄像机对象并将其添加到场景中。例如，以下代码可以创建一个透视摄像机并将其添加到场景中：

  ```javascript
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  scene.add(camera);
  ``` 

- 最后，我们需要创建一个渲染器对象并将其添加到HTML页面中。例如，以下代码可以创建一个WebGL渲染器并将其添加到页面中：

  ```javascript
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);
  ``` 

- 现在，我们可以使用以下代码来渲染场景：

  ```javascript
  function animate() {
    requestAnimationFrame(animate);
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render(scene, camera);
  }
  animate();
  ``` 

表格语法输出：

| Scene | 
| --- |
| - Scene是three.js中最基本的组件之一，它是一个容器，可以用来存放所有的物体、灯光和摄像机。 |
| - 可以通过以下代码创建一个场景对象：<br><br>```const scene = new THREE.Scene();``` |
| - 可以向场景中添加物体、灯光和摄像机等组件。例如，以下代码可以向场景中添加一个立方体：<br><br>```const geometry = new THREE.BoxGeometry(1, 1, 1);```<br>```const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });```<br>```const cube = new THREE.Mesh(geometry, material);```<br>```scene.add(cube);``` |
| - 在渲染场景之前，需要创建一个摄像机对象并将其添加到场景中。例如，以下代码可以创建一个透视摄像机并将其添加到场景中：<br><br>```const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);```<br>```scene.add(camera);``` |
| - 最后，我们需要创建一个渲染器对象并将其添加到HTML页面中。例如，以下代码可以创建一个WebGL渲染器并将其添加到页面中：<br><br>```const renderer = new THREE.WebGLRenderer();```<br>```renderer.setSize(window.innerWidth, window.innerHeight);```<br>```document.body.appendChild(renderer.domElement);``` |
| - 现在，我们可以使用以下代码来渲染场景：<br><br>```function animate() {```<br>```requestAnimationFrame(animate);```<br>```cube.rotation.x += 0.01;```<br>```cube.rotation.y += 0.01;```<br>```renderer.render(scene, camera);```<br>```}```<br>```animate();``` |

Fog
- Fog

Fog组件可以用来模拟场景中的雾气效果。以下是一个简单的例子：

```javascript
var scene = new THREE.Scene();
scene.fog = new THREE.Fog(0xffffff, 0, 750);

var geometry = new THREE.BoxGeometry( 200, 200, 200 );
var material = new THREE.MeshBasicMaterial( { color: 0xff0000 } );
var mesh = new THREE.Mesh( geometry, material );
scene.add( mesh );

var camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
camera.position.z = 500;

var renderer = new THREE.WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );

function animate() {
    requestAnimationFrame( animate );
    mesh.rotation.x += 0.01;
    mesh.rotation.y += 0.02;
    renderer.render( scene, camera );
}
animate();
```

在上面的例子中，我们创建了一个场景，设置了雾气效果，然后向场景中添加了一个红色的立方体。在渲染循环中，我们让立方体不断旋转，并使用渲染器渲染场景。当我们运行这个例子时，我们会看到一个带有雾气效果的旋转立方体。

FogExp2- FogExp2

```javascript
// 创建FogExp2对象
const fog = new THREE.FogExp2(0xffffff, 0.05);
// 将FogExp2对象添加到场景中
scene.fog = fog;
```

说明：以上代码创建了一个FogExp2对象，并设置了颜色为白色，浓度为0.05。将该对象添加到场景中，可以在场景中呈现出浓雾效果。

### 3.3 相机组件


Camera
- camera的创建和设置

```javascript
// 创建****
var camera = new three.perspectivecamera(45, window.innerwidth / window.innerheight, 0.1, 1000);

// 设置相机位置
camera.position.x = -30;
camera.position.y = 40;
camera.position.z = 30;

// 设置相机看向的位置
camera.lookat(scene.position);
```

- 相机控制器

```javascript
// 引入orbitcontrols.js
import { orbitcontrols } from 'three/examples/jsm/controls/orbitcontrols.js';

// 创建控制器
var controls = new orbitcontrols(camera, renderer.domelement);

// 设置控制器的目标点
controls.target.set(0, 0, 0);

// 设置控制器的最大和最小缩放距离
controls.mindistance = 10;
controls.maxdistance = 100;

// 设置控制器的最大仰角和俯角
controls.maxpolarangle = math.pi / 2;
controls.minpolarangle = 0;
```

OrthographicCamera
- orthographiccamera是一种**投影相机，适用于平面渲染，可以设置left、right、top、bottom、near、far等参数来控制相机的视角和范围。
- 实例：

  ```js
  const camera = new three.orthographiccamera(-window.innerwidth/2, window.innerwidth/2, window.innerheight/2, -window.innerheight/2, 1, 1000);
  camera.position.set(0, 0, 500);
  ``` 

  这段代码创建了一个orthographiccamera相机，视角为窗口大小的一半，位置在屏幕**，远近裁剪面为1到1000，可以用于平面渲染。

PerspectiveCamera- perspectivecamera

```javascript
// 创建****
var camera = new three.perspectivecamera(45, window.innerwidth / window.innerheight, 0.1, 1000);

// 设置相机位置
camera.position.x = -30;
camera.position.y = 40;
camera.position.z = 30;

// 设置相机指向的位置
camera.lookat(scene.position);
```

### 3.4 几何体组件


BoxGeometry
- BoxGeometry

  ```javascript
  // 创建一个长宽高均为1的正方体
  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
  ```

  这段代码创建了一个长宽高均为1的正方体，并将其加入场景中。BoxGeometry还可以传入更多参数，如widthSegments、heightSegments、depthSegments来控制几何体的精度。

CircleGeometry
- CircleGeometry

  ```
  // 创建一个半径为1的圆形几何体
  const geometry = new THREE.CircleGeometry(1);
  // 设置几何体的位置
  geometry.position.set(0, 0, 0);
  // 创建一个红色的材质
  const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
  // 创建一个网格对象
  const mesh = new THREE.Mesh(geometry, material);
  // 将网格对象添加到场景中
  scene.add(mesh);
  ```

CylinderGeometry
- CylinderGeometry

  ```javascript
  // 创建一个半径为1，高度为2，分段数为32的圆柱体
  const geometry = new THREE.CylinderGeometry(1, 1, 2, 32);
  // 创建一个材质
  const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
  // 创建一个网格
  const cylinder = new THREE.Mesh(geometry, material);
  // 将网格添加到场景中
  scene.add(cylinder);
  ```

SphereGeometry- SphereGeometry

  ```javascript
  const geometry = new THREE.SphereGeometry( 5, 32, 32 );
  const material = new THREE.MeshBasicMaterial( { color: 0xffffff } );
  const sphere = new THREE.Mesh( geometry, material );
  scene.add( sphere );
  ```

### 3.5 材质组件


MeshBasicMaterial
- MeshBasicMaterial是最简单的材质，它不考虑光照的影响，只使用一种颜色来渲染物体。

```js
const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
const geometry = new THREE.BoxGeometry(1, 1, 1);
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

- 表格语法：

| 参数 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| color | Color | 0xffffff | 材质的颜色 |
| opacity | number | 1 | 材质的透明度 |
| transparent | boolean | false | 是否开启透明度 |
| map | Texture | null | 材质的贴图 |
| wireframe | boolean | false | 是否显示线框模式 |
| side | Constants | THREE.FrontSide | 材质的渲染面 |
| blending | Constants | THREE.NormalBlending | 材质的混合模式 |
| depthTest | boolean | true | 是否开启深度测试 |
| depthWrite | boolean | true | 是否开启深度写入 |
| alphaTest | number | 0 | 透明度测试的阈值 |
| visible | boolean | true | 物体是否可见 |

MeshLambertMaterial
- MeshLambertMaterial是一种基础的材质组件，用于创建具有Lambertian（漫反射）外观的物体。
- 它的属性包括颜色、环境色、发光色、透明度、混合模式等。
- 示例代码：

  ```
  var material = new THREE.MeshLambertMaterial( { color: 0xff0000 } );
  var mesh = new THREE.Mesh( geometry, material );
  scene.add( mesh );
  ```

MeshPhongMaterial
- MeshPhongMaterial是一种具有光泽感的材质，常用于渲染具有金属或塑料外观的物体。

  ```javascript
  var material = new THREE.MeshPhongMaterial( { color: 0xff0000 } );
  var mesh = new THREE.Mesh( geometry, material );
  scene.add( mesh );
  ```

- 可以通过设置不同的属性来调整材质的外观，例如：

  ```javascript
  var material = new THREE.MeshPhongMaterial( {
      color: 0xff0000, // 材质颜色
      specular: 0xffffff, // 镜面高光颜色
      shininess: 100, // 镜面高光强度
      map: texture, // 纹理贴图
      normalMap: normalMap, // 法线贴图
      transparent: true, // 是否透明
      opacity: 0.5 // 透明度
  } );
  ```

ShaderMaterial- ShaderMaterial

```javascript
var material = new THREE.ShaderMaterial( {
    uniforms: {
        time: { value: 1.0 },
        resolution: { value: new THREE.Vector2() }
    },
    vertexShader: document.getElementById( 'vertexShader' ).textContent,
    fragmentShader: document.getElementById( 'fragmentShader' ).textContent
} );
```

其中，uniforms是一个包含uniform变量的对象，vertexShader和fragmentShader是字符串类型的着色器代码，可以从HTML文件中获取。这个材质可以用于创建基于自定义着色器的材质，可以达到更高的渲染效果。

### 3.6 灯光组件


AmbientLight
- `AmbientLight`：环境光，是一种均匀的光照，对物体的影响是不受物体位置和朝向影响的。

  ```javascript
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  ```
  这里创建了一个白色的环境光，强度为0.5，被添加到场景中。

DirectionalLight
- DirectionalLight是一种产生平行光的光源组件，可以用来模拟太阳光等。

```javascript
// 创建一个平行光源
var light = new THREE.DirectionalLight(0xffffff, 1);

// 设置光源位置
light.position.set(0, 1, 0);

// 将光源添加到场景中
scene.add(light);
```

- 可以通过设置光源的位置和颜色来调整光照效果。

HemisphereLight
- HemisphereLight是一种环境光，它模拟了来自天空的光线和地面反射的光线。
- 它需要两种颜色参数：天空颜色和地面颜色。
- 实例代码：

  ```javascript
  const hemisphereLight = new THREE.HemisphereLight(0xffffbb, 0x080820, 1);
  scene.add(hemisphereLight);
  ``` 

- 表格形式的实例代码：

  | 参数 | 类型 | 描述 |
  | --- | --- | --- |
  | skyColor | Color | 天空颜色 |
  | groundColor | Color | 地面颜色 |
  | intensity | Number | 光强度 |

PointLight
- PointLight实例：
```javascript
const light = new THREE.PointLight(0xffffff, 1, 100);
light.position.set(0, 0, 10);
scene.add(light);
```

SpotLight- SpotLight

  ```js
  // 创建一个聚光灯
  const spotLight = new THREE.SpotLight(0xffffff, 1);
  spotLight.position.set(0, 100, 0);
  spotLight.angle = Math.PI / 4;
  spotLight.penumbra = 0.05;
  spotLight.decay = 2;
  spotLight.distance = 200;

  // 设置光源投射阴影
  spotLight.castShadow = true;
  spotLight.shadow.mapSize.width = 1024;
  spotLight.shadow.mapSize.height = 1024;
  spotLight.shadow.camera.near = 10;
  spotLight.shadow.camera.far = 200;
  scene.add(spotLight);
  ```

### 3.7 控制器组件


OrbitControls
- OrbitControls是three.js中的一个控制器组件，可以用来控制相机在三维空间中的移动和旋转。
  
  ```javascript
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  
  const controls = new OrbitControls(camera, renderer.domElement);
  
  controls.enableDamping = true; // 启用阻尼效果，使相机移动更平滑
  controls.dampingFactor = 0.05; // 阻尼系数
  controls.rotateSpeed = 0.5; // 旋转速度
  controls.zoomSpeed = 1.2; // 缩放速度
  controls.panSpeed = 0.8; // 平移速度
  controls.target.set(0, 0, 0); // 设置控制器的焦点，即相机注视的位置
  ```
  
  通过设置控制器的各种属性，可以实现不同的相机控制效果。

FlyControls
- FlyControls是一个three.js中的控制器组件，可以让用户使用鼠标和键盘控制场景中的相机飞行。
  ```javascript
  // 创建控制器
  const controls = new THREE.FlyControls(camera);
  // 设置控制器的移动速度
  controls.movementSpeed = 100;
  // 设置控制器的旋转速度
  controls.rollSpeed = Math.PI / 6;
  // 将控制器添加到场景中
  scene.add(controls.getObject());
  ```

TrackballControls- TrackballControls是一个基于鼠标交互的控制器组件，可以让用户通过鼠标旋转、平移和缩放场景中的物体。
- 以下是一个使用TrackballControls的示例代码：

```js
import * as THREE from 'three';
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

camera.position.z = 5;

const controls = new TrackballControls(camera, renderer.domElement);

function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}
animate();
```

- 在这个示例中，我们创建了一个立方体，并使用TrackballControls控制了相机，使用户可以通过鼠标旋转、平移和缩放场景中的物体。

## 4. Threejs应用实例


3D场景展示
- 3D场景展示

  ```javascript
  // 创建场景
  const scene = new THREE.Scene();
  
  // 创建相机
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  
  // 创建渲染器
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);
  
  // 创建立方体
  const geometry = new THREE.BoxGeometry();
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
  
  // 移动相机位置
  camera.position.z = 5;
  
  // 渲染场景
  function animate() {
    requestAnimationFrame(animate);
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render(scene, camera);
  }
  animate();
  ```
  
  以上代码创建了一个简单的3D场景，包含一个绿色的立方体，并且实现了立方体的旋转动画。

3D动画制作
- 3D动画制作

  以下是一个简单的3D动画制作实例：

  ```html
  <!DOCTYPE html>
  <html>
    <head>
      <meta charset="utf-8">
      <title>Three.js 3D Animation</title>
      <style>
        body {
          margin: 0;
        }
        canvas {
          display: block;
        }
      </style>
    </head>
    <body>
      <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
      <script>
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const geometry = new THREE.BoxGeometry();
        const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);

        camera.position.z = 5;

        function animate() {
          requestAnimationFrame(animate);

          cube.rotation.x += 0.01;
          cube.rotation.y += 0.01;

          renderer.render(scene, camera);
        }

        animate();
      </script>
    </body>
  </html>
  ```

  这是一个简单的3D动画，它显示了一个绿色的立方体，不断旋转。可以通过修改代码中的参数来调整动画的效果。

3D游戏开发- 3D游戏开发：

  ```javascript
  // 创建场景
  var scene = new THREE.Scene();

  // 创建相机
  var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

  // 创建渲染器
  var renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  // 创建立方体
  var geometry = new THREE.BoxGeometry(1, 1, 1);
  var material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  var cube = new THREE.Mesh(geometry, material);
  scene.add(cube);

  // 移动相机位置
  camera.position.z = 5;

  // 渲染场景
  function animate() {
    requestAnimationFrame(animate);
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render(scene, camera);
  }
  animate();
  ```

## 5. 总结


Threejs的优点
- Threejs的优点：
  - 能够创建高效的3D图形，支持WebGL渲染器。
  - 提供了丰富的几何体、材质和光源等基础组件，方便快速搭建3D场景。
  - 支持多种导入格式，如OBJ、FBX、Collada等，方便与其他3D软件进行交互。
  - 社区活跃，有大量的插件和工具可供选择，能够满足不同需求。

Threejs的不足
- Threejs的不足：
  - 对于大型场景的性能优化不够好，需要手动进行调整。
  - 在移动端设备上的性能表现不如在PC端。
  - 对于初学者而言，学习曲线较陡峭，需要一定的编程基础和数学基础。
  - 相对于其他3D引擎，文档和教程相对较少，需要自行摸索和探索。
- 实例内容：
  - 在使用Threejs开发一个3D游戏时，我们发现在场景中添加过多的复杂模型和纹理会导致性能下降，因此我们需要进行性能优化。一种解决方案是使用LOD技术，即在远处使用低多边形的模型，在近处使用高多边形的模型，以减少渲染负载。

Threejs的发展前景- Threejs的发展前景：
  - Threejs在虚拟现实、增强现实、游戏开发等领域有广泛应用，随着这些领域的不断发展，Threejs也将得到更多的应用机会。
  - Threejs的开源特性和活跃的社区使得其不断更新和完善，未来的发展前景非常广阔。
  - 随着WebGL技术的发展，Threejs可以在浏览器中实现更加复杂的3D场景和交互效果，这也将为其未来的发展提供更多的可能性。