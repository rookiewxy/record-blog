## BI组件

BI已经开发到了组件部分，对于组件的复用以及如何提效做了一些封装

#### 一、 模板

1.  对于代码的复用模板文件必不可少，使用`plop`来生成对应的文件复用，使用方式如下

   a. 在`package.json`中，添加对应的执行脚本，命令如下

   ```json
   
     "scripts": {
       "new": "plop --plopfile ./plop-templates/plopfile.js"
     },
   ```

   b. 在根目录添加`plop-templates`文件夹，组件主要分为图表组件，和非图表组件，例如是图表组件，执行代码`yarn new chart-component -p 组件名称`

   ![image-20220618163431578](C:\Users\xt09\Desktop\blog\BI-component.assets\image-20220618163431578.png)

   c. 讲图表组件在细分，例如柱状图有几种类型的组件图，堆叠、三维立体等，但是基本配置都是一样的，执行完`yarn new chart-component -p 组件名称`执行完此命令后，会出现选择对应图表类型列表，选择即可

   ![image-20220618163810975](C:\Users\xt09\Desktop\blog\BI-component.assets\image-20220618163810975.png)

   ![img](https://cdn.nlark.com/yuque/0/2022/png/22370352/1655195312400-be10b29e-f281-4486-97f4-7299a643cbdb.png)

   

#### 二、 自定义package.json

1. 由于每次`lerna create  包名`后，经常要修改`package.json`中的代码，很麻烦，于是我先能不能自定义`package.json`,，是可以的，必须是在当前windows用户的文件夹根目录下新建`.npm-init.js`，执行命令，新建项目，在npm init你会发现，就是你`npm-init.js`文件中的内容

   ```bash
   npm config set init-module ~/.npm-init.js
   ```

   2. 由于此项目使用的是`monorepo`的管理方式，采用了`lerna`，使用`lerna create 包名`时，使用的是他自己内部封装的`package.json`，所以`npm-init`的方式不行


#### 三、 动态修改package.json

1. 上面说了希望自定义，但是还有一个需求是能够对`package.json`进行动态写入，由于我们的配置参数都是放在package.json都是放在中的，但是有一部分配置也是重复的，比如每个组件都会有字体样式这种配置，所以也可以单独抽离，使用命令行进行写入

2. 拆分如下

   ![image-20220618170333930](C:\Users\xt09\Desktop\blog\BI-component.assets\image-20220618170333930.png)

3. 执行命令`yarn new generate -p 组件名称 schema:schema模板名称`

   ```json
    "scripts": {
       "generate": "node ./scripts/generate-schema.js"
     },
   ```

4. 代码如下

   ```js
   /*
    * @Description: package.json动态注入schema配置
    * @Autor: wxy
    * @Date: 2022-04-24 20:17:04
    * @LastEditors: wxy
    * @LastEditTime: 2022-06-01 17:17:29
    */
   const _ = require("loadsh");
   const fs = require('fs');
   const path = require('path');
   const minimist = require('minimist');
   const rawArgs = process.argv.slice(3);
   const args = minimist(rawArgs)._;
   const resolve = p => path.resolve(__dirname, '../', p);
   
   class GenerateSchema {
   
     /**
      * @description 获取对应的schema和组件信息
      */
     static getPkgsSchemaInfo () {
       // 获取命令行中设置的schema类型
       const schemaTypeList = args.filter(a => a.includes("schema")).map(s => s.replace("schema:", ""));
       // 获取命令行中设置的组件
       const targetPkgList = args.filter(a => !a.includes("schema"));
       this.setPkgsSchameaTypes(schemaTypeList, targetPkgList);
     }
   
     /**
      * 
      * @param {*} schemaTypeList 需要设置的schema列表
      * @param {*} targetPkgList 需要设置的目标组件列表
      */
     static setPkgsSchameaTypes (schemaTypeList, targetPkgList) {
       // 读取对应的schema信息
       const schemaInfo = schemaTypeList.map(schemaName => {
         return require(resolve(`schema-templates/${schemaName}.json`));
       });
       schemaInfo.forEach(s=>{
         targetPkgList?.forEach(packName => {
           const pkgDir = `ui-components/${packName}/package.json`;
           // !readFileSync采用同步方法，异步方法会有bug
           let fileContents = fs.readFileSync(pkgDir, { encoding: 'utf8' });
             fileContents = _.cloneDeep(JSON.parse(fileContents));
             fileContents.config.config = {
               ...s,
               ...fileContents.config.config
             };
             fs.writeFileSync(pkgDir, JSON.stringify(fileContents, null, 2), { encoding: 'utf8' });
         });
       });
     }
   
   }
   
   GenerateSchema.getPkgsSchemaInfo();
   ```

   

#### 四、 代码的抽象

1. 刚刚说到的图表组件，图表组件是有很多相同的配置，页面配置面板需要和渲染组件之间，肯定需要修改对应的参数，可以提出一个公共的基类，例如图裂、动画、tooltip这些是每个图表组件都需要的，所以放在基类中、在根据图表类型提供不同的类