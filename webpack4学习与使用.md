# webpack的学习与使用

>最近打算在学习之余，做一些笔记。有助于梳理和记忆，为了以后可以更好地查看仓库里的笔记，于是做了个简单的项目，把博客都陈列出来，以更为便捷的方式呈现出来。虽然是一个非常mini的项目，但是麻雀虽小，一应俱全。由于项目中用到webpack4打包器，于是为此做笔记如下：

当下最流行的模块打包器 webpack 于2018年2月25日正式发布v4.0.0版本，代号legato。从官方的[发布日志](https://github.com/webpack/webpack/releases/tag/v4.0.0)来看, 本次大版本更新带来了很多新特性更新和改善，这将会让webpack的配置更加简单由于目前对于webpack4的资料已经很多了， 我在此只做一些个人以为重要，需要的地方做些记录。具体详细配置可以移步[我的repo](https://github.com/toBeUrself/toBeUrself.github.io)查看。

# mode模式

webpack4中需要设置mode属性，可以是 development，production 和 none, 其中production为默认值[官网介绍](https://webpack.docschina.org/concepts/mode/),production模式下会自动启用压缩。

 选项|描述
--|:--:
development|会将 process.env.NODE_ENV 的值设为 development。启用 NamedChunksPlugin 和 NamedModulesPlugin。
production|会将 process.env.NODE_ENV 的值设为 production。启用 FlagDependencyUsagePlugin,FlagIncludedChunksPlugin,ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 UglifyJsPlugin
none|不选用任何默认优化选项

# 开箱即用WebAssembly

WebAssembly(wasm)会带来运行时性能的大幅度提升，由于在社区的热度，webpack4对它做了开箱即用的支持。你可以直接对本地的wasm模块进行import或者export操作，也可以通过编写loaders来直接import C++、C或者Rust。

# 下面对一些常用的插件做一个简单的了解

+ html-webpack-plugin[参考](https://www.cnblogs.com/sunflowerGIS/p/6820912.html)

*这个插件还是很好用的，用于生成一个html文件并且自动引入一些需要用到的css和js文件。*

```javascript
new HtmlWebpackPlugin({
    title: 'Tims blogs',
    template: './main.html',
    filename: './index.html',
    chunks: ['main', 'vender'],
    minify: {
      removeAttributeQuotes: true, // 移除属性的引号
      removeComments: true, //移除注释
      removeEmptyAttributes: true, //是否移除空属性
      collapseWhitespace: true, //是否移除空格
      caseSensitive: false,// 是否大小写敏感
  }
})
```

+ clean-webpack-plugin copy-webpack-plugin

*插件看起来就很字面上的作用*


`new CleanWebpackPlugin(['dist'])`

```javascript 
new CopyWebpackPlugin([{
    from: __dirname + '/css',
    to: __dirname + '/dist/css'
}])
```

+ mini-css-extract-plugin

*webpack4中用这个插件提取css到单独的文件，以前的extract-text-webpack-plugin插件其实也是可以用的，在安装的时候需要后面加个参数extract-text-webpack-plugin@next,该插件的具体使用[请参考](https://www.jianshu.com/p/91e60af11cc9),下面也给出我在项目中的配置*

`const devMode = process.env.NODE_ENV !== 'production';`
```
{
    test: /\.(sa|sc|c)ss$/,
    use: [
        devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
        'css-loader',
        'sass-loader',
    ]
}
```
```
new MiniCssExtractPlugin({
    filename: 'style.[hash].css',
    chunkFilename: "style.[hash].css"
})
```

+ webpack-bundle-analyzer

*这个插件很有用,可以看到打包后的文件具体包含了哪些部分，模块。配置较为简单[参考](https://www.jianshu.com/p/4cdaeaa01fd5).以下给出我在项目中的配置。默认端口是8888，可以自己配置。*

`const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;`

`new BundleAnalyzerPlugin({ analyzerPort: 8919 })`

+ compression-webpack-plugin

*这个插件以gzip的压缩方式打包，让打包后的文件更小，文件已.gz后缀结尾，浏览器是可以解析.gz文件的。因为html里面是引入的.css和.js文件，所以具体怎么让浏览器下载.gz文件，需要额外的配置。以下给出我在项目中使用该插件的配置*

```
new CompressionWebpackPlugin({
    filename: '[path].gz[query]',
    algorithm: 'gzip',
    test: new RegExp('\\.(js|css)$'),
    minRatio: 0.8 //0.8是默认值
})
```

+ cross-env

*这个插件是用于设置env的参数的，没有过多地去了解，以下给出在项目中的配置。设置这个参数可以告诉一些插件做一些生产环境的优化。*

`"build": "cross-env NODE_ENV=production webpack --mode production"`
```
new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify('production')
})
```

+ webpack-merge

*这个插件用于将两个config做一个merge的操作，在把development和production两种环境不同配置提出到不同文件后，用此插件把环境配置和通用配置做merge。具体使用情况请参考[我的项目](https://github.com/toBeUrself/toBeUrself.github.io)*

***以上便是对这次使用webpack4做的一些比较粗略的笔记。如果相对webpack4有一个完整详细的了解与学习，去[官网](https://webpack.docschina.org/concepts/)详读文档是再好不过的选择。***

[@天地独舞](https://tobeurself.github.io/)










