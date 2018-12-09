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
