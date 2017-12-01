# 前言

前端逃避不了浏览器兼容问题，这篇文章介绍一下前端css的兼容问题有哪些，应该如何做到浏览器的兼容。

思路：

> 1. 了解css的浏览器兼容性问题有哪些；
> 
> 2. 通过工具自动完成浏览器兼容；

本文写时的主要开发工具版本：

> webpack: 1.15.0
> 
> vue: 2.5.7
> 
> vue-loader: 12.2.2 (官方文档显示，vue-loader版本大于11.0.0时，vue的postcss不需要单独配置browserslist，可以自动从package.json等公用——与其他需要browserslist的插件公用，如autoprefixer——配置文件获取。但升级到\^13.0.0以后，在我的项目中报错，因为13.0.0以后，vue对一些代码格式做了调整。详情在“一些疑问”——vue-loader升级到\^13.0.0后报错)
>  
> postcss-loader: 5.2.18
> 
> css-loader: 0.28.7
> 
> style-loader: 0.19.0


# css兼容问题

css的兼容问题在网络上有一大堆的文章。需要找到一个比较好的总结。参考文档中有一个。

请帮忙找一个“最全的”总结。

# 通过工具解决问题

## 通过webpack的配置，解决样式文件的提取和分割问题

```
// 这不是一个全的配置文件，根据本文的介绍重点提取出部分配置
module: {
    loaders: [
      {
        test: /\.vue$/,
        loader: 'vue'
      },
      {
        test: /\.js$/,
        loader: 'babel',
        include: projectRoot,
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        // 这里要将style-loader放到extract外面，否则在通过UglifyJsPlugin插件压缩的时候会报错，具体看下面的参考链接“window is not defined issue”
        loaders: ["style-loader"].concat(ExtractTextPlugin.extract("css-loader" ,'postcss-loader'))
      },
      {
        test: /\.json$/,
        loader: 'json'
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  },
  vue: {
    // 这里做简化显示，实际中是通过函数返回的loaders
    // 将vue中的样式表也提取出来
    loaders: {
     css: ExtractTextPlugin.extract('vue-style-loader', 'css-loader')
    }
  },
  plugins: [
    new webpack.optimize.OccurenceOrderPlugin(),
    // extract css into its own file
    new ExtractTextPlugin(utils.assetsPath('css/[name].[contenthash].css')),
    // IE9及一下对样式文件有限制，这里将样式文件控制在4000 selectors，多余此限制就分成多个文件。参考链接“css-split-webpack-plugin”
    new CSSSplitWebpackPlugin({
      size: 4000,
      filename: 'css/[name]-[part].[ext]'
    })
  ]
```



## 通过postcss解决浏览器的兼容问题。

浏览器版本配置：

 ```
// package.json
 "browserslist": [
    "Chrome >= 53",
    "Firefox  >= 45",
    "ie >= 9"
  ]
```

postcss配置文件：

```
module.exports = {
  plugins: [
    //////////////////////////预处理/////////////////////////////////
    // 项目引用了其他样式库。
    // 因为有些浏览器对样式的扩展可能已经改变，所以去掉原样式库中所有的浏览器扩展，
    require('postcss-unprefix'),
    ////////////////////////////////////////////////////////////////

    /////////////////////////浏览器兼容处理///////////////////////////
    // IE不支持css动画中的will-change属性，通过postcss-will-change插件转化
    require('postcss-will-change'),
    // 根据浏览器配置(package.json中browserslist配置)增加样式的浏览器扩展
    require('autoprefixer'),
    // IE9 & IE10 在"font shorthand property"和伪元素中不支持rem，需要将其转化为px；
    require('pixrem')({
      rootValue: 12,  // 将默认字体大小设置为12px
      html: false  // 不使用html{}或者:root{}中font-size的设置覆盖此设置
    }),
    ////////////////////////////////////////////////////////////////

    /////////////////////////////压缩和优化/////////////////////////
    // 删除样式文件中的注释，包括“重要注释”。cssnano插件中可通过配置discardComments的removeAll为true达到此目的。
    // 但经过测试发现没有效果，顾单独引入此插件清理注释
    require('postcss-discard-comments')({removeAll:true}),
    // 压缩css文件。在使用postcss之前，css-loader默认压缩文件，使用postcss之后，不能自动压缩，原因不详，先通过手动压缩
    require('cssnano')({
      // preset: ['default', {
      //   discardComments: {
      //     removeAll: true
      //   }
      // }]
      preset: 'default'
    })
    ////////////////////////////////////////////////////////////////
  ]
}
```


# 一些疑问

本文在整理过程中遇到一些疑问：

[vue-loader升级到\^13.0.0后报错——未解决](https://forum.vuejs.org/t/there-is-compile-errors-when-i-update-vue-loader-to-13-0-0/22055)

[vue如何处理js中手写的样式——未解决](https://forum.vuejs.org/t/how-the-vue-loader-do-with-inline-styles-in-vue/22131)

[使用postcss-loader后，css-loader不能自动将符合条件的png转化为base64——未解决](https://stackoverflow.com/questions/47446953/the-png-can-not-compile-to-the-css-file-when-i-use-postcss-loader)

# YY

有这么一个工具：

  你只要选择需要支持的浏览器版本：
  
  ```
  "browserslist": [
        "Chrome >= 53",
        "Firefox  >= 45",
        "ie >= 9"
      ]
  ```

  你只要选择你需要的功能：

> **上传** 

> **动画**

>  …… 

  你只要选择你目前使用的语言版本

>   js: **es6**、**es7**

>   css: **sass**、**css**

>  ……

就能帮你解决所有css、js的浏览器兼容问题。帮不了你的给你提示。

*请问你激动不激动？*

# 参考链接

[浏览器兼容性问题及解决方案(CSS部分)](http://www.jianshu.com/p/eba18372a3c1)

[PostCSS深入学习: 跨浏览器兼容性](https://www.w3cplus.com/PostCSS/using-postcss-for-cross-browser-compatibility.html)

[window is not defined issue](https://github.com/webpack-contrib/extract-text-webpack-plugin/issues/503)

[css-split-webpack-plugin](https://github.com/metalabdesign/css-split-webpack-plugin/issues/26)