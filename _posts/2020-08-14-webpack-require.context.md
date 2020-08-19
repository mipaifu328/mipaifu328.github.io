---
layout: post
title: "require.context"
subtitle: 前端工程化自动引入js文件或svg文件
date: 2020-08-19 12:10
author: "mipaifu328"
header-img: "img/in-post/2019-03/sea.jpeg"
tag: 
    - webpack
---

# API 介绍

> webpack 会在构建中解析代码中的 require.context() 。

- `require.context()`可以给这个函数传入三个参数：一个要搜索的目录，一个标记表示是否还搜索其子目录， 以及一个匹配文件的正则表达式。
- `返回一个（require）函数`，此函数可以接收一个参数：request。
此导出函数有三个属性：`resolve, keys, id`
  - resolve 是一个函数，它返回 request 被解析后得到的模块 id (传入keys某个模块，返回绝对路径)
  - keys 也是一个函数，返回匹配成功模块的名字组成的数组
  - id 是 context module 的模块 id. 它可能在你使用 module.hot.accept 时会用到。

``` js
  require.context('./test', false, /\.test\.js$/);
  //（创建出）一个 context，其中文件来自 test 目录，request 以 `.test.js` 结尾。

  const cache = {};

  function importAll (r) {
    r.keys().forEach(key => cache[key] = r(key));
  }

  importAll(require.context('../components/', true, /\.js$/));
  // 在构建时(build-time)，所有被 require 的模块都会被填充到 cache 对象中。

```

[webpack require.context API](https://webpack.docschina.org/guides/dependency-management/#requirecontext)

# 应用场景

## js自动导入

如果遇到从一个文件夹引入很多模块的情况,,可以使用这个api,它会遍历文件夹中的指定文件,然后自动导入,使得不需要每次显式的调用import导入模块。

## svg自动导入
[手摸手，带你优雅的使用 icon](https://juejin.im/post/6844903517564436493)