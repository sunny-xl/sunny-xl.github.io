---
title: webpack 初探
date: 2018-03-06 19:40:24
categories: 前端
tags: [webpack, 前端]
comments: false
---

## introduce

> webpack 是现代 js 应用的模块打包器，它是非常可配置，但要明确以下四个概念。

1. Entry
   webpack 构建一个你应用程序依赖的图，起点便是所谓的 erntry point。它告诉 wenpack 从哪开始并且
   按照依赖图去知道什么要打包。可以将此比作 contextual root or the first file to kick off your app.。
   The simplest example：

```
//webpack.config.js
module.exports={
    entry:'./path/to/my/entry/file.js'
};
```

2. Output
   一旦打包好了资源，依然需要告诉 webpack 哪里打包应用程序，output 属性告诉 webpack 如何处理打包过的代码。
   The simplest example：
   `//webpack.config.js module.exports={ entry:'./path/to/my/entry/file.js'， output: { path: path.resolve(__dirname, 'dist'), filename: 'my-first-webpack.bundle.js' } };`
   示例中用 output.filename 和 output.path 属性告诉 webpack 包名和将之 emit to 哪。

  <!--more-->

3. Loaders
   目的在于将所有项目中的资源作为 webpack 而非浏览器关心的。webpack 眼中一切(.css,.hmtl,.scss,.jpg,etc)
   皆模块。但其只知道 js，而 Loaders 在 webpack 中将这些文件转化成依赖图中的模块。有两个目的在高水准要求上：
   a.确定什么文件该被相应 loader 转换(test)，b：转换相应文件以加入到依赖图(use)。

```
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  }
};
module.exports = config;
```

4. Plugins
   因 loaders 仅基于每个文件转换，故 plugins 是最常用的对打包模块的 "compilations" or "chunks"执行操作和自定义功能等。
   要用 plugin 需要用 require()且将其放入 plugins 数组。多数 plugins 可按需自定义
   因可在配置中按不同目的多次使用，此时需要用 new 来构建它的一个实例。

```
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      {test: /\.(js|jsx)$/, use: 'babel-loader'}
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```
