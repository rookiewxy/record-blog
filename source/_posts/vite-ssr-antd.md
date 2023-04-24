### Vite+SSR+Antd+Nginx
1. 公司官网需要用ssr来做，刚好vite有ssr的插件---[vite-plugin-ssr](https://vite-plugin-ssr.com/)
2. 根据vite-plugin-ssr的文档，来添加目录,根目录下
pages 页面结合，文件需要xxx.page.tsx
renderer 客户端和服务端渲染的文件需放在该目录下，其中_default.page.server中有一段插入html的代码，需要注意（antd 踩坑）
server 由于是ssr打包之后是没有index.html文件的，需要启动一个服务，在nginx部署时，直接用pm2进行管理就可以了
[项目地址](https://github.com/rookiewxy/react-vite-ssr)
3. 踩坑一、当我自信满满打包之后部署的时候，发现antd的样式都么有加载，所以页面上的组件样式都失效了，在网上看到了@antd-design/cssinjs的方式，需要用`createCache`, `extractStyle`, `StyleProvider`在_default.page.server文件中进行配置，因为antd5改用了css-in-js的方式，这些方法就是用来获取antd组件的css的，但是没有效果，我始终获取不到，于是我换到了四点几的版本，有了下段代码,重点是下面的link标签，antd的样式问题可算解决了
```
async function render(pageContext) {
  const { Page, pageProps, urlPathname } = pageContext;
  const pageHtml = renderToString(
    <StaticRouter location={urlPathname}>
      <Page />
    </StaticRouter>,
  );
  return escapeInject`<!DOCTYPE html>
    <html>
      <head>
        <title>巡天科技</title>
        <link rel='stylesheet' href='https://cdn.bootcss.com/antd/4.9.0/antd.css'/>
        <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
      <head>
      <body>
        <div id="root">${dangerouslySkipEscape(pageHtml)}</div>
      </body>
    </html>`;
}
```
4. 踩坑二，在服务端使用pm2进行启动时，一直失败，可算同样的命令，同样的代码，在我本地是完成ok的，这是为啥呢，于是我看了pm2的日志，发现了一下错误，意思就是，大多数时候@brillout/vite-plugin-import-build可以自动导入服务器构建(文件在dist/server/)。但是，在某些情况下，它不起作用，您必须手动导入服务器构建。需要自己手动导入!!!
```
Most of the time @brillout/vite-plugin-import-build can automatically import the server build (files at dist/server/).

But, in some situations, it doesn't work and you have to import the server build manually.
```
所以我在server/index.js添加了以下代码
```
require('../dist/server/importBuild.cjs')
```
5. 用了七个小时左右终于成功部署上线，现在头都是晕的