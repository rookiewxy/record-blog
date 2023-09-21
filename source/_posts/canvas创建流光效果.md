## 流光效果
模仿easyv使用canvas创建流光效果
代码如下
``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Canvas with Flowing Line along SVG Path</title>
  </head>
  <body>
    <!-- SVG路径元素 -->
    <svg width="256" height="112" id="mySVG">
      <path
        fill="none"
        stroke="currentColor"
        stroke-width="1"
        d="M8,56 C8,33.90861 25.90861,16 48,16 C70.09139,16 88,33.90861 88,56 C88,78.09139 105.90861,92 128,92 C150.09139,92 160,72 160,56 C160,40 148,24 128,24 C108,24 96,40 96,56 C96,72 105.90861,92 128,92 C154,93 168,78 168,56 C168,33.90861 185.90861,16 208,16 C230.09139,16 248,33.90861 248,56 C248,78.09139 230.09139,96 208,96 L48,96 C25.90861,96 8,78.09139 8,56 Z"
      ></path>
    </svg>

    <!-- Canvas元素 -->
    <canvas id="myCanvas" width="256" height="112"></canvas>

    <script>
      // 获取SVG路径元素
      const svgPath = document.querySelector("#mySVG path");
      const svgPathData = svgPath.getAttribute("d");

      // 获取Canvas元素
      const canvas = document.getElementById("myCanvas");
      const ctx = canvas.getContext("2d");

      // 解析SVG路径数据的函数
      function parseSvgPath(pathData) {
        const commands = [];
        const pathSegments = pathData.match(/([a-df-zA-DF-Z])[^a-df-zA-DF-Z]*/g);

        for (const segment of pathSegments) {
          const cmd = segment.match(/([a-df-zA-DF-Z])|[-+]?(?:\d+\.?\d*|\.\d+)(?:[eE][-+]?\d+)?/g);
          commands.push(cmd);
        }

        return commands;
      }

      // 根据解析后的数据绘制路径
      function drawPath(pathCommands) {
        // 创建新路径
        ctx.beginPath();

        for (const cmd of pathCommands) {
          const cmdName = cmd[0];
          if (cmdName === "M") {
            ctx.moveTo(cmd[1], cmd[2]);
          } else if (cmdName === "L") {
            ctx.lineTo(cmd[1], cmd[2]);
          } else if (cmdName === "C") {
            ctx.bezierCurveTo(cmd[1], cmd[2], cmd[3], cmd[4], cmd[5], cmd[6]);
          }
        }

        // 设置路径样式
        ctx.strokeStyle = "currentColor";
        ctx.lineWidth = 1;

        // 绘制路径
        ctx.stroke();
      }

      // 解析SVG路径数据
      const pathCommands = parseSvgPath(svgPathData);
      // 创建流光元素
      const flowingElement = {
        pathLength: svgPath.getTotalLength(),
        distance: 0,
        speed: 2, // 流光速度
        color: "red", // 流光颜色
        lineWidth: 5, // 线条宽度
      };

      function animate() {
        // 清除Canvas
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 获取当前点的坐标
        const point = svgPath.getPointAtLength(flowingElement.distance);

        // 设置矩形样式
        ctx.fillStyle = flowingElement.color;

        // 计算矩形的位置和大小
        const rectWidth = 20; // 长方形的宽度
        const rectHeight = 10; // 长方形的高度

        // 获取下一个点的坐标，用于计算方向
        const nextPoint = svgPath.getPointAtLength(flowingElement.distance + 1);

        // 计算矩形的旋转角度（方向）
        const angle = Math.atan2(nextPoint.y - point.y, nextPoint.x - point.x);


        // 保存当前的绘图状态
        ctx.save();

        // 位移Canvas原点到矩形中心点
        ctx.translate(point.x, point.y);

        // 旋转Canvas以适应路径方向
        ctx.rotate(angle);

        // 绘制矩形
        ctx.fillRect(-rectWidth / 2, -rectHeight / 2, rectWidth, rectHeight);

        // 恢复绘图状态
        ctx.restore();

        // 更新流光的位置
        flowingElement.distance += flowingElement.speed;

        // 如果流光超出路径长度，重新设置其位置
        if (flowingElement.distance > flowingElement.pathLength) {
          flowingElement.distance = 0;
        }

        // 绘制SVG路径到Canvas
        drawPath(pathCommands);

        // 请求下一帧动画
        requestAnimationFrame(animate);
      }

      // 启动动画
      animate();
    </script>
  </body>
</html>
```