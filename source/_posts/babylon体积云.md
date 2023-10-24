### babylon体积云
#### 功能点如下
1. 雾
2. 云层密度
3. 实时时间，也可以设置白昼时长 
```ts
// 获取当前时间（以毫秒为单位）
const currentTimeMs = new Date().getTime();

// 将毫秒转换成秒
const currentTimeSec = currentTimeMs * 0.001;

// 计算一天的总长度
const totalDayLength = dayLength + nightLength;

// 计算当前时间相对于一天总长度的位置
const timeOfDay = currentTimeSec % totalDayLength;

// 计算白天和夜晚的时间段
const isDaytime = timeOfDay < dayLength;

// 计算时间相对于白天或夜晚的百分比
const timePercentage = isDaytime ? timeOfDay / dayLength : (timeOfDay - dayLength) / nightLength;

// 根据时间百分比设置sunPosition
const sunX = Math.sin(timePercentage * 2 * Math.PI);
const sunY = Math.cos(timePercentage * 2 * Math.PI);
shaderMaterial.setVector3("sunPosition", new BABYLON.Vector3(sunX, sunY, 1));
```