---
title: vue-cli项目打包多个与static文件同级的静态资源目录（copy-webpack-plugin插件的使用）
date: 2018-05-08
categories:
- [VUE]
tags:
- [vue-cli]
---

#### 场景
业务要求能够直接通过 “域名+/file”的方式访问静态资源的html，然而产品绝对static暴露在url中不好看又不能直接将html放在static中。所以想到了既然static可以直接访问，那么给他新加几个文件目录应该不是问题。
#### 重点
在webpack.dev.conf.js和webpack.prod.conf.js两个文件中，都有这样一段配置代码：

```javascript
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
// 作用：将static目录拷贝到打包之后的dist文件下
```
相关配置详见：[copy-webpack-plugin](https://github.com/webpack-contrib/copy-webpack-plugin)
so lucky！
下面就简单了，照葫芦画瓢，配置好你要拷贝的文件目录即可：

```javascript
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.dev.assetsSubDirectory,
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../file1'),
        to: 'file1',
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../file2'),
        to: 'file2',
        ignore: ['.*']
      }
    ])
```
项目目录只需在与static的同级目录新建对应的file1,file2即可，里面可以放任何你想放的东西，且访问时只需使用绝对路径即可（建议css，js等静态资源也都使用绝对路径）
#### YY
既然这样可行，那么将通过这种方式将静态站点融入到vue项目中也是可行的，当你一个站点上有很多静态页面时，或者是成品非vue项目代码，只需要通过这种方式即可合并到vue项目中，且只需要在url上加上你定义的file即可，有相关经验的大佬欢迎交流。