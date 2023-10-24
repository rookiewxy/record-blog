### babylon动态天空盒

#### 实现体积云
着色器代码与babylon融合，里面的uniform变量需要外部传递，说说遇到的坑，此代码参考https://www.shadertoy.com/view/MstBWs，如果在webgl环境中是没有问题的，但是在webgpu中会报错，因为babylon不能使用线性过滤，`Get3DNoise`方法中的对纹理使用了线性过滤，需要进行更改，线性过滤的作用就是让云层平滑，没有齿轮的感觉，这时我们需要换成`双线性差值`
```js
uniform sampler2D iChannel0;
uniform vec2 iMouse;
uniform vec2 iResolution;
uniform float iTime;
uniform float cloudSpeed;


#define VOLUMETRIC_LIGHT
#define cameraMode 1
#define cloudHeight 600.0
#define cloudThickness 500.0 // 云层的厚度
#define cloudDensity 0.03 // 云层的密度
#define fogDensity 0.0


#define volumetricCloudSteps 30			//Higher is a better result with rendering of clouds.
#define volumetricLightSteps 8			//Higher is a better result with rendering of volumetric light.

#define cloudShadowingSteps 12			//Higher is a better result with shading on clouds.
#define volumetricLightShadowSteps 4	//Higher is a better result with shading on volumetric light from clouds

#define rayleighCoeff (vec3(0.27, 0.5, 1.0) * 1e-5)	//Not really correct
#define mieCoeff vec3(0.5e-6)						//Not really correct

const float sunBrightness = 3.0;

#define earthRadius 6371000.0

float bayer2(vec2 a){
    a = floor(a);
    return fract( dot(a, vec2(.5, a.y * .75)) );
}

vec2 rsi(vec3 position, vec3 direction, float radius) {
    float PoD = dot(position, direction);
    float radiusSquared = radius * radius;

    float delta = PoD * PoD + radiusSquared - dot(position, position);
    if (delta < 0.0) return vec2(-1.0);
          delta = sqrt(delta);

    return -PoD + vec2(-delta, delta);
}



#define bayer4(a)   (bayer2( .5*(a))*.25+bayer2(a))
#define bayer8(a)   (bayer4( .5*(a))*.25+bayer2(a))
#define bayer16(a)  (bayer8( .5*(a))*.25+bayer2(a))
#define bayer32(a)  (bayer16(.5*(a))*.25+bayer2(a))
#define bayer64(a)  (bayer32(.5*(a))*.25+bayer2(a))
#define bayer128(a) (bayer64(.5*(a))*.25+bayer2(a))

//////////////////////////////////////////////////////////////////

#define cloudMinHeight cloudHeight
#define cloudMaxHeight (cloudThickness + cloudMinHeight)

#define sunPosition vec3(-1.0, 1.0, 1.0)

const float pi = acos(-1.0);
const float rPi = 1.0 / pi;
const float hPi = pi * 0.5;
const float tau = pi * 2.0;
const float rLOG2 = 1.0 / log(2.0);

mat3 rotationMatrix(vec3 axis, float angle)
{
    axis = normalize(axis);
    float s = sin(angle);
    float c = cos(angle);
    float oc = 1.0 - c;
    
    float xx = axis.x * axis.x;
    float yy = axis.y * axis.y;
    float zz = axis.z * axis.z;
    
    float xy = axis.x * axis.y;
    float xz = axis.x * axis.z;
    float zy = axis.z * axis.y;
    
    return mat3(oc * xx + c, oc * xy - axis.z * s, oc * xz + axis.y * s,
                oc * xy + axis.z * s, oc * yy + c, oc * zy - axis.x * s, 
                oc * xz - axis.y * s, oc * zy + axis.x * s, oc * zz + c);
}

struct positionStruct
{
	vec2 texcoord;
    vec2 mousecoord;
    vec3 worldPosition;
    vec3 worldVector;
    vec3 sunVector;
} pos;

vec3 sphereToCart(vec3 sphere) {
    vec2 c = cos(sphere.xy);
    vec2 s = sin(sphere.xy);
    
    return sphere.z * vec3(c.x * c.y, s.y, s.x * c.y);
}

vec3 calculateWorldSpacePosition(vec2 p)
{
	p = p * 2.0 - 1.0;
    
    vec3 worldSpacePosition =  vec3(p.x, p.y, 1.0);
    
    #ifdef SPHERICAL_PROJECTION
		worldSpacePosition = sphereToCart(worldSpacePosition * vec3(pi, hPi, 1.0));
	#endif
    
    return worldSpacePosition;
}

void gatherPositions(inout positionStruct pos, vec2 FragCoord, vec2 mouseCoord, vec2 screenResolution)
{
	pos.texcoord = FragCoord / screenResolution;
    pos.mousecoord = mouseCoord / screenResolution;
    

     vec2 rotationAngle = radians(vec2(360.0, 90.0) * pos.mousecoord - vec2(0.0, 45.0));
    mat3 rotateH = rotationMatrix(vec3(0.0, 1.0, 0.0), rotationAngle.x);
    mat3 rotateV = rotationMatrix(vec3(1.0, 0.0, 0.0), rotationAngle.y);
    
    pos.worldPosition = calculateWorldSpacePosition(pos.texcoord);
    
    	pos.worldPosition = rotateH * (rotateV * pos.worldPosition);
        
    	pos.sunVector = normalize(sunPosition);
    pos.worldVector = normalize(pos.worldPosition);
}

#define d0(x) (abs(x) + 1e-8)
#define d02(x) (abs(x) + 1e-3)

const vec3 totalCoeff = rayleighCoeff + mieCoeff;

vec3 scatter(vec3 coeff, float depth){
	return coeff * depth;
}

vec3 absorb(vec3 coeff, float depth){
	return exp2(scatter(coeff, -depth));
}

float calcParticleThickness(float depth){
   	
    depth = depth * 2.0;
    depth = max(depth + 0.01, 0.01);
    depth = 1.0 / depth;
    
	return 100000.0 * depth;   
}

float calcParticleThicknessH(float depth){
   	
    depth = depth * 2.0 + 0.1;
    depth = max(depth + 0.01, 0.01);
    depth = 1.0 / depth;
    
	return 100000.0 * depth;   
}

float calcParticleThicknessConst(const float depth){
    
	return 100000.0 / max(depth * 2.0 - 0.01, 0.01);   
}

float rayleighPhase(float x){
	return 0.375 * (1.0 + x*x);
}

float hgPhase(float x, float g)
{
    float g2 = g*g;
	return 0.25 * ((1.0 - g2) * pow(1.0 + g2 - 2.0*g*x, -1.5));
}

float miePhaseSky(float x, float depth)
{
 	return hgPhase(x, exp2(-0.000003 * depth));
}

float powder(float od)
{
	return 1.0 - exp2(-od * 2.0);
}

float calculateScatterIntergral(float opticalDepth, float coeff){
    float a = -coeff * rLOG2;
    float b = -1.0 / coeff;
    float c =  1.0 / coeff;

    return exp2(a * opticalDepth) * b + c;
}

vec3 calculateScatterIntergral(float opticalDepth, vec3 coeff){
    vec3 a = -coeff * rLOG2;
    vec3 b = -1.0 / coeff;
    vec3 c =  1.0 / coeff;

    return exp2(a * opticalDepth) * b + c;
}


vec3 calcAtmosphericScatter(positionStruct pos, out vec3 absorbLight){
    const float ln2 = log(2.0);
    
    float lDotW = dot(pos.sunVector, pos.worldVector );
    float lDotU = dot(pos.sunVector, vec3(0.0, 1.0, 0.0));
    float uDotW = dot(vec3(0.0, 1.0, 0.0), pos.worldVector);
    
	float opticalDepth = calcParticleThickness(uDotW);
    float opticalDepthLight = calcParticleThickness(lDotU);
    
    vec3 scatterView = scatter(totalCoeff, opticalDepth);
    vec3 absorbView = absorb(totalCoeff, opticalDepth);
    
    vec3 scatterLight = scatter(totalCoeff, opticalDepthLight);
         absorbLight = absorb(totalCoeff, opticalDepthLight);
    	 
    vec3 absorbSun = abs(absorbLight - absorbView) / d0((scatterLight - scatterView) * ln2);
    
    vec3 mieScatter = scatter(mieCoeff, opticalDepth) * miePhaseSky(lDotW, opticalDepth);
    vec3 rayleighScatter = scatter(rayleighCoeff, opticalDepth) * rayleighPhase(lDotW);
    
    vec3 scatterSun = mieScatter + rayleighScatter;
    
    vec3 sunSpot = smoothstep(0.9999, 0.99993, lDotW) * absorbView * sunBrightness;
    
    return (scatterSun * absorbSun + sunSpot) * sunBrightness;
}

vec3 calcAtmosphericScatterTop(positionStruct pos){
    const float ln2 = log(2.0);
    
    float lDotU = dot(pos.sunVector, vec3(0.0, 1.0, 0.0));
    
	float opticalDepth = calcParticleThicknessConst(1.0);
    float opticalDepthLight = calcParticleThickness(lDotU);
    
    vec3 scatterView = scatter(totalCoeff, opticalDepth);
    vec3 absorbView = absorb(totalCoeff, opticalDepth);
    
    vec3 scatterLight = scatter(totalCoeff, opticalDepthLight);
    vec3 absorbLight = absorb(totalCoeff, opticalDepthLight);
    
    vec3 absorbSun = d02(absorbLight - absorbView) / d02((scatterLight - scatterView) * ln2);
    
    vec3 mieScatter = scatter(mieCoeff, opticalDepth) * 0.25;
    vec3 rayleighScatter = scatter(rayleighCoeff, opticalDepth) * 2.5;
    
    vec3 scatterSun = mieScatter + rayleighScatter;
    
    return (scatterSun * absorbSun) * sunBrightness;
}

float Get3DNoise(vec3 pos) 
{
    float p = floor(pos.z);
    float f = pos.z - p;
    
    const float invNoiseRes = 1.0 / 67.0;
    
    float zStretch = 17.0 * invNoiseRes;
    
    vec2 coord = pos.xy * invNoiseRes + (p * zStretch);
    
    // vec2 noise = vec2(texture(iChannel0, coord).x,
	// 				  texture(iChannel0, coord + zStretch).x);

    vec2 noise = vec2(textureLod(iChannel0, coord, 0.0).x, textureLod(iChannel0, coord + vec2(zStretch, 0.0), 0.0).x);

    vec2 noiseLerp = mix(noise, vec2(textureLod(iChannel0, coord + zStretch, -0.5).x), f);
    
    return mix(noiseLerp.x, noiseLerp.y, f);
}

float getClouds(vec3 p)
{
    p = vec3(p.x, length(p + vec3(0.0, earthRadius, 0.0)) - earthRadius, p.z);
    
    if (p.y < cloudMinHeight || p.y > cloudMaxHeight || length(p) > 5500.0)
        return 0.0;
    
    vec3 movement = vec3(cloudSpeed, 0.0, cloudSpeed);
    
    vec3 cloudCoord = (p * 0.001) - movement;
    
	float noise = Get3DNoise(cloudCoord) * 0.5;
    	  noise += Get3DNoise(cloudCoord * 2.0 + movement) * 0.25;
    	  noise += Get3DNoise(cloudCoord * 7.0 - movement) * 0.125;
    	  noise += Get3DNoise((cloudCoord + movement) * 16.0) * 0.0625;
    
    const float top = 0.004;
    const float bottom = 0.01;
    
    float horizonHeight = p.y - cloudMinHeight;
    float treshHold = (1.0 - exp2(-bottom * horizonHeight)) * exp2(-top * horizonHeight);
    
    float clouds = smoothstep(0.45, 0.6, noise); // 数量
          clouds *= treshHold;
    
    return clouds * cloudDensity;
}
    
float getCloudShadow(vec3 p, positionStruct pos)
{
	const int steps = volumetricLightShadowSteps;
    float rSteps = cloudThickness / float(steps) / abs(pos.sunVector.y);
    
    vec3 increment = pos.sunVector * rSteps;
    vec3 position = pos.sunVector * (cloudMinHeight - p.y) / pos.sunVector.y + p;
    
    float transmittance = 0.0;
    
    for (int i = 0; i < steps; i++, position += increment)
    {
		transmittance += getClouds(position);
    }
    
    return exp2(-transmittance * rSteps);
}

float getSunVisibility(vec3 p, positionStruct pos)
{
	const int steps = cloudShadowingSteps;
    const float rSteps = cloudThickness / float(steps);
    
    vec3 increment = pos.sunVector * rSteps;
    vec3 position = increment * 0.5 + p;
    
    float transmittance = 0.0;
    
    for (int i = 0; i < steps; i++, position += increment)
    {
		transmittance += getClouds(position);
    }
    
    return exp2(-transmittance * rSteps);
}

float phase2Lobes(float x)
{
    const float m = 0.6;
    const float gm = 0.8;
    
	float lobe1 = hgPhase(x, 0.8 * gm);
    float lobe2 = hgPhase(x, -0.5 * gm);
    
    return mix(lobe2, lobe1, m);
}

vec3 getVolumetricCloudsScattering(float opticalDepth, float phase, vec3 p, vec3 sunColor, vec3 skyLight, positionStruct pos)
{
    float intergal = calculateScatterIntergral(opticalDepth, 1.11);
    
    float beersPowder = powder(opticalDepth * log(2.0));
    
	vec3 sunlighting = (sunColor * getSunVisibility(p, pos) * beersPowder) * phase * hPi * sunBrightness;
    vec3 skylighting = skyLight * 0.25 * rPi;
    
    return (sunlighting + skylighting) * intergal * pi;
}

float getHeightFogOD(float height)
{
	const float falloff = 0.001;
    
    return exp2(-height * falloff) * fogDensity;
}

vec3 getVolumetricLightScattering(float opticalDepth, float phase, vec3 p, vec3 sunColor, vec3 skyLight, positionStruct pos)
{
    float intergal = calculateScatterIntergral(opticalDepth, 1.11);
    
	vec3 sunlighting = sunColor * phase * hPi * sunBrightness;
         sunlighting *= getCloudShadow(p, pos);
    vec3 skylighting = skyLight * 0.25 * rPi;
    
    return (sunlighting + skylighting) * intergal * pi;
}

vec3 calculateVolumetricLight(positionStruct pos, vec3 color, float dither, vec3 sunColor)
{
    #ifndef VOLUMETRIC_LIGHT
    	return color;
    #endif
    
	const int steps = volumetricLightSteps;
    const float iSteps = 1.0 / float(steps);
    
    vec3 increment = pos.worldVector * cloudMinHeight / clamp(pos.worldVector.y, 0.1, 1.0) * iSteps;
    vec3 rayPosition = increment * dither;
    
    float stepLength = length(increment);
    
    vec3 scattering = vec3(0.0);
    vec3 transmittance = vec3(1.0);
    
    float lDotW = dot(pos.sunVector, pos.worldVector);
    float phase = hgPhase(lDotW, 0.8);
    
    vec3 skyLight = calcAtmosphericScatterTop(pos);
    
    for (int i = 0; i < steps; i++, rayPosition += increment)
    {
        float opticalDepth = getHeightFogOD(rayPosition.y) * stepLength;
        
        if (opticalDepth <= 0.0)
            continue;
        
		scattering += getVolumetricLightScattering(opticalDepth, phase, rayPosition, sunColor, skyLight, pos) * transmittance;
        transmittance *= exp2(-opticalDepth);
    }
    
    return color * transmittance + scattering;
}

vec3 calculateVolumetricClouds(positionStruct pos, vec3 color, float dither, vec3 sunColor)
{
	const int steps = volumetricCloudSteps;
    const float iSteps = 1.0 / float(steps);
    
    
    float bottomSphere = rsi(vec3(0.0, 1.0, 0.0) * earthRadius, pos.worldVector, earthRadius + cloudMinHeight).y;
    float topSphere = rsi(vec3(0.0, 1.0, 0.0) * earthRadius, pos.worldVector + 0.0, earthRadius + cloudMaxHeight).y;
    
    vec3 startPosition = pos.worldVector * bottomSphere;
    vec3 endPosition = pos.worldVector * topSphere;
    
    vec3 increment = (endPosition - startPosition) * iSteps;
    vec3 cloudPosition = increment * dither + startPosition;
    
    float stepLength = length(increment);
    
    vec3 scattering = vec3(0.0);
    float transmittance = 1.0;
    
    float lDotW = dot(pos.sunVector, pos.worldVector);
    float phase = phase2Lobes(lDotW);
    
    vec3 skyLight = calcAtmosphericScatterTop(pos);
    
    for (int i = 0; i < steps; i++, cloudPosition += increment)
    {
        float opticalDepth = getClouds(cloudPosition) * stepLength;
        
        if (opticalDepth <= 0.0)
            continue;
        
		scattering += getVolumetricCloudsScattering(opticalDepth, phase, cloudPosition, sunColor, skyLight, pos) * transmittance;
        transmittance *= exp2(-opticalDepth);
    }
    
    return mix(color * transmittance + scattering, color, clamp(length(startPosition) * 0.00001, 0.0, 1.0));
}

vec3 robobo1221Tonemap(vec3 color)
{
    #define rTOperator(x) (x / sqrt(x*x+1.0))

    float l = length(color);

    color = mix(color, color * 1.3, l / (l+1.0));
    color = rTOperator(color);

    return color;
}

void main()
{
    vec2 fragCoord = gl_FragCoord.xy;
    gatherPositions(pos, fragCoord, iMouse.xy, iResolution.xy);
    float dither = bayer16(fragCoord);
    
    vec3 lightAbsorb = vec3(0.0);

    vec3 color = vec3(0.0);
         color = calcAtmosphericScatter(pos, lightAbsorb);
         color = calculateVolumetricClouds(pos, color, dither, lightAbsorb);
         color = calculateVolumetricLight(pos, color, dither, lightAbsorb);
         color = pow(color, vec3(1.0 / 2.2));
         color = robobo1221Tonemap(color);

    gl_FragColor = vec4(color, 1.0);
}

```


#### 实现网格平面
网格平面其实非常好实现，由于是基于babylon，所以可以直接使用他的api`GridMaterial`,需要下载`babylon-material`插件,
直接使用gridMaterial并不能满足我的需求，因为在天空盒当中，camera旋转网格面板肯定是需要一个最高高度的限制，以及在y轴上进行旋转时，网格如何和天空盒进行同步，以及当我缩放相机，到达天空盒内部时，天空盒如何继续绘制，而不是黑的
```ts
// 网格平面代码示例
    let groundMaterial = new BABYLON.GridMaterial("groundMaterial", this.scene);
    groundMaterial.gridRatio = 2;
    groundMaterial.opacity = 0.99;
    groundMaterial.useMaxLine = false;
    groundMaterial.mainColor = new BABYLON.Color3(1, 1, 1);
    groundMaterial.lineColor = new BABYLON.Color3(1.0, 1.0, 1.0);
    
	var ground = BABYLON.MeshBuilder.CreateDisc("ground", {radius: 1000}, scene);
    ground.material = groundMaterial;
    ground.rotation = new BABYLON.Vector3(Math.PI / 2, 0, 0);
```

- 最高高度的限制
```ts
camera.upperBetaLimit =5 * Math.PI / 10.2
```

- 缩放相机，到达天空盒内部,天空盒继续绘制，需设置`backFaceCulling`
```ts
 var shaderMaterial = new BABYLON.ShaderMaterial("starfieldMaterial", scene, {
        vertexSource: vertexShader,
        fragmentSource: fragmentShader
    });
    shaderMaterial.backFaceCulling = false;
```

- 网格如何和天空盒进行同步
这个问题耗时最久，不同步指的是在`camera`进行y轴旋转时，网格平面的移动速度大于天空盒的移动速度，但是网格显示在天空中去了，改了很久还是没有办法，后来看见一个平台，非常聪明的设计，就是网格只显示一半，这里的一半可不是遮盖和裁剪，而是`camera`的视口`viewport`,于是我进行了调整，虽然我设置了两个`camera`，但是只要有一个`camera`设置了`viewport`整个场景都只会显示在那一块地方，但是我想要的是网格平面显示一半，其他的物体还是正常显示呀，真是难住我了，终于让我发现了layerMask属性，在两个相机和对应的物体上设置layermask做到分层,这样我设置`camera2`的`viewport`并不会影响`camera`
```ts
    camera.layerMask = 1;
    box.layerMask = 1;

    camera2.layerMask = 2;
    ground.layerMask = 2;
```