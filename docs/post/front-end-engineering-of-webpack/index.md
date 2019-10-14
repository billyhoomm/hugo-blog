
## 基于Webpack的前端工程化初探

## webpack的重点配置介绍

### 出入口文件配置
```js
 module.exports = {
  entry:{ 
  app:'./src/main.js'//入口文件
  },
  output: {
      path:path.resolve(__dirname, '../dist'),//存放打包后文件的输出目录
      publicPath:'/',//指定资源文件引用的目录
      filename: 'app.bundle.js',//打包文件名
  },
```
### 打包成多个资源文件
在开发多页面的站点的时候，会希望能有多个资源文件。如下：
```js
// webpack.config.js

entry: {
    page1: "entry.js",
    page2: "entry2.js"
},
output: {
    path: path.join(__dirname, 'out'),
    publicPath: "./out/",
    filename: '[name].js'
}
```
entry属性可以是一个对象，而对象名也就是key会作为下面output的filename属性的[name]

### 独立出css样式
如果我们希望样式通过<link>引入，而不是统一放在打包好的JS文件中的style标签内，可以这样：
```js
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
    // ...省略各种代码
    plugins: [
        new ExtractTextPlugin("[name].css")
    ]
}
```
这里以CSS后缀结尾的模块用插件,在ExtractTextPlugin的extract方法有两个参数，第一个参数是经过编译后通过style-loader单独提取出文件来，而第二个参数就是用来编译代码的loader。

插件也支持所有独立样式打包成一个css文件。增加多一个参数即可：
```js
new ExtractTextPlugin("style.css", {allChunks: true})
```
### loader配置
```js
 module: {
        loaders: [
            { test: /\.js?$/,exclude: [node_modules],loader: 'babel'},
            {test: /\.css$/, loader: "style!css"},
            {test: /\.(jpg|png)$/, loader: "url?limit=8192"},
            {test: /\.scss$/, loader: "style!css!sass"},
        ]
    }
```
### 使用别名
看如下配置：
```js
 resolve: {
    extensions: ['', '.js', '.vue'],
    fallback: [path.join(__dirname, '../node_modules')],
    alias: {
      'src': path.resolve(__dirname, '../src'),
      'assets': path.resolve(__dirname, '../src/assets'),
      'components': path.resolve(__dirname, '../src/components'),
      "jquery": path.resolve(__dirname, '../node_modules/jquery/dist/jquery.slim.min.js'),
      "bootstrap": path.resolve(__dirname, '../src/plugin/bootstrap'),
    }
  }
```
**说明：**

- extensions：用于指明程序自动补全识别哪些文件后缀
- alias：它的作用是把用户的一个请求重定向到另一个路径，以便在其他文件中进行引用的时候可以更方便，这个配置在插件和依赖以及静态资源比较多的时候可以用，大大节省时间。

### 关于全局模块/全局变量

如果习惯了使用全局模块，例如 jQuery 的 $，而不想每次都写 $ = require('jquery')， 可以使用 ProvidePlugin 插件:
```js
plugins: [
  new webpack.ProvidePlugin({ 
    $: "jquery",
    jQuery: "jquery"

  })
]
```

### 公共文件打包配置
如果是多入口页面的项目，多个 Entry 之间可能会有一些公共的基础库，这时候就要用到公共文件提取打包，使用 CommonsChunkPlugin 插件来处理。这个插件支持很多种传参和设置，我比较喜欢下面这种对象传递，这样可以指定生成多个包:
```js
entry: {
  a:"./a.js",
  b:"./b.js",
  common1:[ //以下库文件及其下游依赖都会被打到 common1 中
    "./lib/common.js",
    "react", "react-dom", "redux", "react-redux", "redux-thunk",
    "react-router", "react-router-redux"
  ],
},
plugins:[
  new webpack.optimize.CommonsChunkPlugin({ 
    names: ['common1'], //可以指定多个 entryName，打出多个common 包
    minChunks: Infinity
  }),
}
```
这时再在 a.js 或 b.js 及其依赖中引用 common1 包中包含的库时，将不会再被重复打包到各自的 bundle 中。（注意：bundle 在页面中的载入顺序为: common1 => a/b ）

### CDN资源文件处理（图片等）
css文件中边我们可能用“./test.png”这样的url来加载图片，但是在生产模式下“test.png”文件会定位到CDN上。所以得想办法自动更新路径。
```js
output: {
    path: path.join(__dirname, 'out'),
    publicPath: "http://cdn.upchina.com/",
    filename: 'XXX.js'
}
module: {
        loaders: [
            {test: /\.(jpg|png)$/, loader: "url-loader?limit=1"},
        ]
    }
```
url-loader配合publicPath可以做到在构建时自动更新这些url

### HtmlWebpackPlugin

这个插件的基本作用就是生成html文件，将 webpack中`entry`配置的相关入口thunk  和  `extract-text-webpack-plugin`抽取的css样式   插入到该插件提供的`template`或者`templateContent`配置项指定的内容基础上生成一个html文件，具体插入方式是将样式`link`插入到`head`元素中，`script`插入到`head`或者`body`中。
```js
plugins: [new HtmlWebpackPlugin({
    title: '按照ejs模板生成出来的页面',
    filename: 'index.html',
    template: 'index.ejs',
  })
```
**关于template补充几点：**

- 为template指定的模板文件没有指定任何loader的话，默认使用ejs-loader
- inject：向template或者templateContent中注入所有静态资源，不同的配置值注入的位置不经相同。可以为true（body）、body、head、false。
- favicon: 添加特定favicon路径到输出的html文档中，这个同title配置项，需要在模板中动态获取其路径值
- hash：true|false，是否为所有注入的静态资源添加webpack每次编译产生的唯一hash值，添加hash形式如下所示：
```js
html <script type="text/javascript" src="common.js?a3e1396b501cdd9041be"></script>
```
- chunks：允许插入到模板中的一些chunk，不配置此项默认会将entry中所有的thunk注入到模板中。在配置多个页面时，每个页面注入的thunk应该是不相同的，需要通过该配置为不同页面注入不同的thunk；