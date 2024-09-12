## babylon飞线
需求：曲线的飞行路径，并且带有贴图的飞行标识物，因为设计稿就是很复杂的飞行箭头，只能用贴图

### 创建飞行路径
采用二次贝塞尔曲线
```ts
    var start = new BABYLON.Vector3(-3, 0, 3);
    var end = new BABYLON.Vector3(3, 0, -3);
    var mid = new BABYLON.Vector3(0, 2, 0);

    var arc = BABYLON.Curve3.CreateQuadraticBezier(start, mid, end, 100);
```

### 创建飞行标识
因为需求原因我使用的是ribbon创建，不是一个三角形，而是一个弯曲片面
```ts
    var ribbonWidth = 1;
    var ribbonLength = 40; // ribbon的长度

    // 创建初始ribbon路径
    function createInitialRibbonPath() {
        var ribbonPath = [];
        for (let i = 0; i < ribbonLength; i++) {
            const point = points[i];
            const normal = normals[i];
            const offset = normal.scale(ribbonWidth / 2);
            ribbonPath.push([
                point.subtract(offset),
                point.add(offset)
            ]);
        }
        return ribbonPath;
    }

    // 创建ribbon,这里的sideOrientation一定要注意不同的值会有不同的效果
    var ribbon = BABYLON.MeshBuilder.CreateRibbon("ribbon", {
        pathArray: createInitialRibbonPath(),
        updatable: true,
        sideOrientation: BABYLON.Mesh.FLIP_TILE
    }, scene);
```


### 创建飞行标识贴图
由于我需要贴图，并且贴图需要设置透明通道，否者就是图片加黑底 
```ts
  const material = new BABYLON.StandardMaterial("ribbonMaterial", scene);
    const texture = new BABYLON.Texture("xxx")
    ribbon.material = material;
    material.diffuseTexture = texture
    // 材质将基于纹理中的 Alpha 通道来控制透明度，使得纹理的透明部分在场景中显示为透明
    material.useAlphaFromDiffuseTexture = true;
    // 开启透明通道
    material.diffuseTexture.hasAlpha = true; 
```


### 飞行标识动画效果
尝试过多种方式，例如通过不断地创建ribbon设置新的位置，后面还是觉得只用更新位置信息，所以就要用到`updateVerticesData`方法,这个方法生效的前提是必须设置`updatable`为`true`
```ts
    // 动画
    var currentFrame = 0;
    var animationSpeed = 0.5; // 控制动画速度

    scene.registerBeforeRender(function () {
        currentFrame += animationSpeed;
        if (currentFrame >= points.length - ribbonLength) {
            currentFrame = 0;
        }

        var startIndex = Math.floor(currentFrame);
        var positions = ribbon.getVerticesData(BABYLON.VertexBuffer.PositionKind);

        for (let i = 0; i < ribbonLength; i++) {
            const pointIndex = (startIndex + i) % points.length;
            const point = points[pointIndex];
            const normal = normals[pointIndex];
            const offset = normal.scale(ribbonWidth / 2);

            const bottomPoint = point.subtract(offset);
            const topPoint = point.add(offset);

            const baseIndex = i * 6; // 每个段有6个坐标（两个三角形）
            positions[baseIndex] = bottomPoint.x;
            positions[baseIndex + 1] = bottomPoint.y;
            positions[baseIndex + 2] = bottomPoint.z;
            positions[baseIndex + 3] = topPoint.x;
            positions[baseIndex + 4] = topPoint.y;
            positions[baseIndex + 5] = topPoint.z;
        }

        ribbon.updateVerticesData(BABYLON.VertexBuffer.PositionKind, positions);
    });

```

### 完整代码
```ts
var createScene = function() {
    var scene = new BABYLON.Scene(engine);

    var camera = new BABYLON.ArcRotateCamera("camera", BABYLON.Tools.ToRadians(90), BABYLON.Tools.ToRadians(65), 12, BABYLON.Vector3.Zero(), scene);
    camera.attachControl(canvas, true);

    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);
    light.intensity = 0.7;

    var ground = BABYLON.MeshBuilder.CreateGround("ground", {width: 6, height: 6}, scene);

    var start = new BABYLON.Vector3(-3, 0, 3);
    var end = new BABYLON.Vector3(3, 0, -3);
    var mid = new BABYLON.Vector3(0, 2, 0);

    var arc = BABYLON.Curve3.CreateQuadraticBezier(start, mid, end, 100);
    const points = arc.getPoints();
    const path3d = new BABYLON.Path3D(points);
    const normals = path3d.getNormals();

    var ribbonWidth = 1;
    var ribbonLength = 40; // ribbon的长度

    // 创建初始ribbon路径
    function createInitialRibbonPath() {
        var ribbonPath = [];
        for (let i = 0; i < ribbonLength; i++) {
            const point = points[i];
            const normal = normals[i];
            const offset = normal.scale(ribbonWidth / 2);
            ribbonPath.push([
                point.subtract(offset),
                point.add(offset)
            ]);
        }
        return ribbonPath;
    }

    // 创建ribbon
    var ribbon = BABYLON.MeshBuilder.CreateRibbon("ribbon", {
        pathArray: createInitialRibbonPath(),
        updatable: false,
        sideOrientation: BABYLON.Mesh.FLIP_TILE
    }, scene);

    const material = new BABYLON.StandardMaterial("ribbonMaterial", scene);
    const texture = new BABYLON.Texture("xxx")
    ribbon.material = material;
    material.diffuseTexture = texture
    // material.useAlphaFromDiffuseTexture = true;
    material.diffuseTexture.hasAlpha = true;

    // 动画
    var currentFrame = 0;
    var animationSpeed = 0.5; // 控制动画速度

    scene.registerBeforeRender(function () {
        currentFrame += animationSpeed;
        if (currentFrame >= points.length - ribbonLength) {
            currentFrame = 0;
        }

        var startIndex = Math.floor(currentFrame);
        var positions = ribbon.getVerticesData(BABYLON.VertexBuffer.PositionKind);

        for (let i = 0; i < ribbonLength; i++) {
            const pointIndex = (startIndex + i) % points.length;
            const point = points[pointIndex];
            const normal = normals[pointIndex];
            const offset = normal.scale(ribbonWidth / 2);

            const bottomPoint = point.subtract(offset);
            const topPoint = point.add(offset);

            const baseIndex = i * 6; // 每个段有6个坐标（两个三角形）
            positions[baseIndex] = bottomPoint.x;
            positions[baseIndex + 1] = bottomPoint.y;
            positions[baseIndex + 2] = bottomPoint.z;
            positions[baseIndex + 3] = topPoint.x;
            positions[baseIndex + 4] = topPoint.y;
            positions[baseIndex + 5] = topPoint.z;
        }

        ribbon.updateVerticesData(BABYLON.VertexBuffer.PositionKind, positions);
    });

    return scene;
};
```