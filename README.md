# WebpackOrg

webpack 学习笔记，跟着官网的收藏学习一遍。记录其中的重点和有疑问的地方。

## 个人总结
- 文章中的代码片段都是针对 webpack.config.js
- “古老”的标签 `<script>`，`<link>` 加载方式
- 模块化按需请求，定量加载 CommonJS, AMD, CMD, UMD, ES6 模块
- 静态分析，依赖关系，不同类型的代码交给不同的加载器进行处理


- 从 webpack 4.0.0 开始，可以不用引入一个配置文件。可以通过 `mode` 选项为 webpack 指定一些默认的配置。
- webpack 4 里将命令行相关的都迁移到 `webpack-cli` 包，在单独执行打包命令时会提示是否安装此包