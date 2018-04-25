# WebpackDev

webpack 学习笔记，跟着官网的收藏学习一遍。记录其中的重点和有疑问的地方。

- 文章中的代码片段都是针对 webpack.config.js
- “古老”的标签 `<script>`，`<link>` 加载方式
- 模块化按需请求，定量加载 CommonJS, AMD, CMD, UMD, ES6 模块
- 静态分析，依赖关系，不同类型的代码交给不同的加载器进行处理


## 概念
*webpack* 是一个现代 JavaScript 应用程序的*静态模块打包器(module bundler)*。当 webpack 处理应用程序时，它会递归地构建一个*依赖关系图(dependency graph)*，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 *bundle*。

核心概念：

- 入口 entry    
    指示 webpack 应该使用那个模块，来作为构建其内部*依赖图*的**开始**。进入入口起点后，webpack 会**找出**有哪些模块和库是入口起点（直接和间接）依赖的。每个依赖**随即**被处理，最后输出到称之为 *bundles* 的文件中。可指定一个或多个入口起点。

    ```javascript
    module.exports = {
      entry: './path/to/my/entry/file.js'
    }
    ```

- 输出 output    
    在哪里输出它所创建的 bundles，以及命名等。
    ```javascript
    const path = require('path');

    module.exports = {
      entry: './path/to/my/entry/file.js',
      output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
      }
    }
    ```

- loader    
    loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将**所有类型**的文件转换为 webpack 能够处理的有效模块，然后再利用 webpack 去打包处理。

    * `test` 用于标识出应该被对应的 loader 进行转换的某个或某些文件。
    * `use` 表示转换时，应该使用哪个 loader。

    先看看基础使用，对一个单独的 module 对象定义了 `rules` 属性，里面包含两个属性：`test` 和 `use`。这告诉 webpack compiler，当你碰到在 `require()` / `import` 语句中被解析为 '.txt' 的路径时，在你对它打包之前，先使用 `raw-lader` （把文件内容作为字符串返回）转换一下。

    ```javascript
    const path = require('path');

    const config = {
      entry: './path/to/my/entry/file.js',
      output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
      },
      module: {
        rules: [
          {
            test: /\.txt$/,
            use: 'raw-loader'
          }
        ]
      }
    };

    module.exports = config;
    ```

- 插件 plugins    
    可以用于执行范围更广的任务（打包优化、压缩、**一直到重新定义环境中的变量**）

    `require()` 插件后，加入到 `plugins` 数组中，通过选项自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例。

    ```javascript
    const HtmlWebpackPlugin = require('html-webpack-plugin'); // 外部插件
    const webpack = require('webpack');
    const path = require('path');

    const  config = {
      entry: './path/to/my/entry/file.js',
      output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
      },
      modules: {
        rules: [
          {
            test: /\.txt$/,
            use: 'raw-loader'
          }
        ]
      },
      plugins: [
        new webpack.optimize.UglifyJsPlugin(),
        new HtmlWebpackPlugin({template: './src/index.html'})
      ]
    };

    module.exports = config;
    ```
    
以上只是概念介绍，接下来将根据文档逐步展开。

## 入口起点 Entry Points
如何去配置 entry 属性

- 单个入口（简写）语法

```javascript
const config = {
  entry: './path/to/my/entry/file.js'
};

module.exports = config;
```

这里还支持传入数组，但是均会被当成主入口。**文档中引用解释的地方对我来说并不是很容易理解**。而且单个引入，在扩展上也不够灵活。

- 对象语法

```javascript
const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
}
```

对象语法会比较繁琐。然而，这是应用程序中定义入口的最可扩展的方式。

- 常见场景

    - 分离应用程序 `app` 和第三方库 `vendor`
    
    入口，比如上面的对象语法。webpack 从 `app.js` 和 `verdors.js` 开始创建依赖图（dependency graph）。这些依赖图是彼此完全分离、相互独立的（每个 bundle 中都有一个 webpack 引导（bootstrap））。这种方式比较常见于，只有一个入口起点（不包括 vendor）的单页应用程序（single page application）中。
    
    这里面提到了**依赖图（dependency graph）**，**引导（bootstrap）**
    
    - 多页应用程序
    
    ```javascript
    const config = {
      entry: {
        pageOne: './src/pageOne/index.js',
        pageTwo: './src/pageTwo/index.js',
        pageThree: './src/pageThree/index.js'
      }
    }
    ```
    
    这部分代码中可以看到有 3 个独立分离的依赖图。详细的解释暂不做分析，只有实际使用过之后才能更好的理解。
    
### 输出

配置 `output` 选项可以控制 webpack 如何向硬盘写入编译文件。注意，即使可以存在多个**入口**起点，但是指指定一个**输出**配置。

- 用法

    在 webpack 中配置 `output` 属性的最低要求是，将它的指设置为一个对象，其中包括如下：
    - `filename` 用于输出文件的文件名
    - 目标输出目录 `path` 的绝对路径
    将一个单独的 `bundle.js` 文件输出到 `/home/proj/public/assets` 目录中
    
    ```javascript
    const config = {
      output: {
        filename: 'bundle.js',
        path: '/home/proj/public/assets'
      }
    }
    
    module.exports = config;
    ```
    
- 多个入口起点

    如果配置创建了多个单独的“chunk”（例如，使用多个入口起点货使用像 CommonsChunkPlugin 这样的插件），则英噶使用占位符（substitutions） 来却表每个文件具有唯一的名称。

    ```javascript
    const config = {
      entry: {
        app:  './src/app.js',
        search: './src/search.js'
      },
      output: {
        filename: '[name].js',
        path: __dirname + '/dist'
      }
    }
    
    module.exports = config;
    ```
    
    这样就能输出对应的文件：`./dist/app.js`，`./dist/search.js`
    
- 进阶

    因为在实际项目中并没有使用到 CDN，所以对这里的理解只能靠文档里的。暂时不做记录。
