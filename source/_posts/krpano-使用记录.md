# Krpano案例

#### 技术栈
1. react
2. krpano
3. typescript

#### 功能点
- 场景切换
- 分片加载
- 动态创建热点
- 热点点击
- 热点动画
- 小行星加载


#### React集成Krpano
1. 在根目录的`index.html`导入`krpano.js`文件，这样全局的window对象上会有krpano对象
2. 挂载成功之后，需要加载对应的xml文件，在根目录的`public`文件夹下，添加一个`krpano.xml`，同样上面的`krpano.js`也是相同位置
3. 通过`onready`函数获取加载成功的实例对象
4. 获取实例对象之后可以根据自己的业务要求存放在`ref`中，配合`createContext`使用，下发到对应的组件


#### 分片加载
使用krpano的分片工具进行加载，切片工具类型分为`Convert Sphere Cube`的互转，以及`MASK VTOR Droplet`，我们使用后者，因为这样分片的大小更小，如果遇到一些比较大的全景图使用第一种就会有卡顿感，krpano的代码如下，注意multires一定不能错，如果不符合当前分片的层级就会造成生成的全景有空隙，不过一般情况下我们使用`MASK VTOR Droplet`进行切片，会在上传图片的相同目录生成一个`vour`文件存放对应的`xml`文件以及切片数据，可以直接粘贴复制，以下为一个场景内容，注意如果在view中设置了`maxpixelzoom`属性，那么`fovmax`是失效的！！！
```xml
	<scene name="scene0" title="xx" onstart="" thumburl="panos/0/thumb.jpg" lat="" lng="" alt="" heading="">
		
		<control bouncinglimits="calc:image.cube ? true : false" />

		<view hlookat="0.0" vlookat="0.0" fovtype="MFOV" fov="120" fovmin="70" fovmax="120" limitview="auto" />

		<preview url="panos/0/preview.jpg" />

		<image>
			<cube url="panos/0/%s/l%l/%0v/l%l_%s_%0v_%0h.jpg" multires="512,640,1280,2560,4864,9856" />
		</image>

	</scene>
```

#### 热点动画
动画效果在视觉上来说是非常有帮助的，例如前进东海，热点扩散效果，这个代码可以直接在官网案例上找到，在热点load的时候加载对应方法就可以了
```xml
	<action name="do_crop_animation" scope="local" args="framewidth, frameheight, framerate">
		calc(local.xframes, (caller.imagewidth /framewidth) BOR 0);
		calc(local.frames, xframes * ((caller.imageheight / frameheight) BOR 0));
		def(local.frame, integer, 0);
		
		calc(caller.crop, '0|0|' + framewidth + '|' + frameheight);
		
		setinterval(calc('crop_anim_' + caller.name), calc(1.0 / framerate),
			if(caller.loaded,
				inc(frame);
				if(frame GE frames, if(caller.onlastframe !== null, callwith(caller, onlastframe() ) ); set(frame,0); );
				mod(xpos, frame, xframes);
				div(ypos, frame, xframes);
				Math.floor(ypos);
				mul(xpos, framewidth);
				mul(ypos, frameheight);
				calc(caller.crop, xpos + '|' + ypos + '|' + framewidth + '|' + frameheight);
			  ,
				clearinterval(calc('crop_anim_' + caller.name));
			);
		);
	</action>

```

#### 初始化鱼眼加载
如果需要使用小行星的加载方式，可以引入`skin/vtourskin.xml`文件，当然了，里面有一些配置项不用的可以删掉，小行星的配置就是`skin_settings`中的`littleplanetintro`属性，设置为`true`就可以了，注意这里可能和`krpano.xml`中的初始化加载第一个场景有冲突，可能会导致失效，因为刚开始我没有加载该插件的时候，写了一个初始化加载的`action`导入之后发现没有效果，去掉初始化函数就好了


#### 交互操作
- 热点交互
- 场景切换

针对这两点，我们可以封装一个类，用于切换场景和热点的创建，因为热点的类型非常多，你不可能重复写很多的配置项，以及他们的监听事件

关于动态加载热点的方法可以使用官网的addhotspot方法，热点的点击事件可以使用onclick事件

```ts
import { delay } from 'lodash-es'
import { ISceneData, IHotSpot, CategoryEnum } from '@/mock/data'

class Krpano{
    krpano: any
    hotspotList:any[] = []

    constructor(krpano: any){
        this.krpano = krpano
    }

    create(data:ISceneData){
        this.addHotSpot(data.hotspot, data)
    }

    createHotSpot(data:IHotSpot){
        const hs = this.krpano.addhotspot()
        hs.url = data.icon.url
        hs.ath = data.icon.ath
        hs.atv = data.icon.atv
        hs.edge = data.icon.edge
        hs._type = data.type
        hs.videoUrl = data?.videoUrl
        hs.imgs = data?.imgs
        hs.content = data?.content
        hs.title = data?.title
        return hs
    }

    addHotSpot(hotspot:IHotSpot[], data:ISceneData){
        delay(()=>{
            this.hotspotList = hotspot?.map(d=>{
                const hs = this.createHotSpot(d)
                hs.onclick = this.loadScene(data.next)
                const text_hs = d.title && this.addTextHotSpot(d, data)

                if(d.title && !d.icon.url){
                    return text_hs
                }else{
                    return hs
                }
            })
        },0)
    }

    addTextHotSpot(hotSpot: IHotSpot, data:ISceneData) {
        const hs = this.createHotSpot(hotSpot)
        hs.onclick =  this.loadScene(hotSpot.next ?? data.next)
        return hs
    }
    
    loadScene(name:string){
        return `loadscene(${name},null,MERGE,BLEND(1.0, easeInCubic))`
    }

    getHotSpotList(type:`${CategoryEnum}`){
        return this.hotspotList?.filter((d: any) => d._type === type)
    }

    addEventListner(type:`${CategoryEnum}`, callback:(...args:Parameters<any>)=>void){
        const hotspot = this.getHotSpotList(type)
        hotspot?.map((d: any) => d.onclick = callback)
    }
}

export {
    Krpano
}
```