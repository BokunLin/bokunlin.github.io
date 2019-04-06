---
title: webpack 配置初探
date: 2019-04-06 17:01:39
tags: [webpack,vue,web]
---

> 这篇拖了很久了，想想还是趁着清明把债给补上把。

## 前言
之前遇到个需求， 要把前端所有图片变成CDN链接。但是还没有接触过 webpack 相关配置， 只能硬着头皮上了。

## 常用参数
* entry - 程序的启动入口（多页应用的话可以配置数组）
* context - 上下文路径（绝对路径）
* output - 输出消息
  * path - 输出文件存放的路径
  * filename - 输出文件名
* module - 对不同模块的处理
  * rules - 根据规则对不同的文件做处理 （ loader ）
* plugins - 插件数组
* resolve - 解析
  * alias - 别名 （在`import`和`require`中可以使用）
* devServer - webpack 提供的本地 server
  * contentBase - 启动目录
  * compress - gzip 压缩
  * port - 服务的端口号

## 简单的例子
```
const webpack = require('webpack')
const path = require('path')

// 代码混淆插件
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
  // 模块处理
  module: {
    // 处理规则规则
    rules: [{
      // 处理的文件夹
      include: [path.resolve(__dirname, 'src')],
      // 处理的 loader
      loader: 'babel-loader',
      // loader 的 参数
      options: {
        plugins: ['syntax-dynamic-import'],

        presets: [['@babel/preset-env', {
          'modules': false
        }]]
      },
      // 匹配规则
      test: /\.js$/
    }, {
      test: /\.(scss|css)$/,
      // loader 的另一种写法
      use: [{
        loader: 'style-loader'
        // 跟上面一样， loader 的参数配置
        options: {}
      }, {
        loader: 'css-loader'
      }, {
        loader: 'sass-loader'
      }]
    }]
  },
  // 输出设置
  output: {
    path: '/dist',  // 输出路径
    filename: '[name].[chunkhash].js' // 输出文件名
  },
  // 开发模式
  mode: 'development',
  plugins: [
    // 混淆插件
    new UglifyJSPlugin()
  ]
}
```
## 话说回来
简单的介绍过后，那如何去实现一个图片CDN化的操作。简单来说就两步：
1. 找到所有图片
2. 将图片的路径替换成 CDN 链接

我这里用了`file-loader`去做这个处理，顺便将替换的域名提取出来，与公共的配置放到一起方便管理，代码如下。

```
...
{
  // 匹配图片
  test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
  use: [
    {
      loader: 'file-loader',
      options: {
        name: '[path][name].[ext]',
        // context 用来规定loader执行上下文
        context: '/',
        // 将图片的输出路径做统一处理，实际上url取到的是文件存放的绝对路径
        publicPath: url => {
          // 默认有个 public 的文件夹前缀
          let tmpStr = path.resolve('../public');

          /* windows 的路径兼容 */
          // 兼容 Windows 将 Windows 下的 \\ 替换成 /
          tmpStr = tmpStr.split(path.sep).join('/');
          // 去除盘符 C: 和 最前面的 / 
          tmpStr = tmpStr.replace(/^([C-Z]:)?\//, '');
          /* 兼容结束 */

          // 清除多余路径 留下图片目录
          url = url.replace(tmpStr, '');
          // 获取 cnd 域名，这个可以根据实际环境来
          return utils.getStaticHost(debug) + url;
        },
        // 是否输出文件
        emitFile: false,
      }
    }
  ]
}
...
```
### 补充一下
想要文件被`loader`解析到需要以`import`或 `require`引入，然后以变量的方式引用。
> vue 的 template 会转换成 js 引用，所以不需要特殊处理
```
// 伪代码
  <template>
    <img :src="img"/>
    <img :src="require('./1.jpg')"/>
  <template>
  <script>
    import img from './1.jpg';  
  <script/>>
```