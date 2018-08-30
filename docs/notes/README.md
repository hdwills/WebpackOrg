---
sidebar: auto
---

# 笔记

## layer.js
最近有新的项目开了，虽然还是传统的混合结构，虽然非前后端分离的，但是仍然希望将一些新的技术能用上，在构建上引入了 webpack 来尝试一下。

多页应用，而且目录结构也比较杂，gulp 用来处理静态资源，webpack 还是主要来处理 js 文件。不过有时也难免遇到用第三方库时，在打包的时候希望将除 js 之外的文件也一并打包进来。比如用到了这个依赖 jQuery 的 layer.js，其中还有对应的 css 和 image。下面就来试试，如何用 webpack 将 layer 打包进来。

### webpack 配置
``` javascript
// webpack.config.js
module.exports = {
  module: {
    // 在这个 loader 中，其中 style-loader 后面的 url，将会在页面中生成 <link> 标签，而非内联的 <style>
    // file-loader 输出的文件在模板文件中会引用到。负责将要加载的文件输出到指定目录，并且生成对应的路径。其中的输入、输出路径，文件名、扩展名等更多的配置像可在文档中详细查看。
    loaders: [
      rules: [
        test: /\.css$/,
        use: {
          { loader: 'style-loader/url' },
          {
            loader: 'file-loader',
            options: {
              publicPath: '../css/',
              outoutPath: '../css/',
              name: '[name].[ext]'
            }
          }
        }
      ]
    ],
    resolve: {
      alias: {
        jquery: 'your/path/js/lib/jquery.min.js',
        layer: 'your/path/js/ui/layer.js',
        layerCss: 'your/path/css/layer/layer.css'
      }
    }
  }
}
```

以上是 webpack 中基础的配置，具体细节大家可查阅文档。下面是页面中的调用。其中遇到的几个问题，也都一一列举下。
1. layer.js 文件在项目中引入时最好用非混淆压缩的版本，便于模块之间相互在依赖时有报错后的调试。
2. 特别是依赖 jQuery 的第三方库，要注意 jQuery 的引入方式。
3. 我采用的是直接引入，加入模块解析的一些配置项，比如你在页面中使用 `import 'loadsh'` 时，webpack.config.js 中的 `resolve` 就会对 webpack 查找 `'loadsh'` 的方式去做修改。
4. 而 `resolve.alias` 是为 `import` 或 `require` 两种引入方式而创建的别名，便于页面中的模块引入。
5. 但是但是，我在页面中引入模块时却出了问题，从而出现了 `import` 和 `require` 混用，有一点不舒服。先出现了 layer.js 识别不到引入的 jQuery，接着是 layer.js 中 `$ is not defined`。建议将 jQuery 包装，然后提供一个包装后的对象，以后随便引入，也都能兼容。各库里的依赖方式不同，会有此问题。

### 页面引入

``` javascript
// layer.js
var $ = require('jauery');
window.$ = $;
window.jQuery = $;
var layer = require('layer');
import style from = 'layerCss';
...
layer.msg('message');
```

### 最终实现
最后再来看看最终的效果：

``` html
dist/
  /app
    /layer.html
  /css
    /layer.css
  /js
    /runtime.js
    /verdor.js
    /layer.js

<!-- 再看 layer.html 的 <head> 里已经注入了处理过的 css 文件 -->
<link rel="styleshtte" tyle="text/css" href="../css/layer.css">
```