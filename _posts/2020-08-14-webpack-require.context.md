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

如果遇到从一个文件夹引入很多模块的情况,可以使用这个api,它会遍历文件夹中的指定文件,然后自动导入,使得不需要每次显式的调用import导入模块。

> 举例`vue-element-admin`里的vuex store > index.js

``` js
import Vue from 'vue'
import Vuex from 'vuex'
import getters from './getters'

Vue.use(Vuex)

// https://webpack.js.org/guides/dependency-management/#requirecontext
const modulesFiles = require.context('./modules', true, /\.js$/)

// you do not need `import app from './modules/app'`
// it will auto require all vuex module from modules file
const modules = modulesFiles.keys().reduce((modules, modulePath) => {
  // set './app.js' => 'app'
  const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
  const value = modulesFiles(modulePath)
  modules[moduleName] = value.default
  return modules
}, {})

const store = new Vuex.Store({
  modules,
  getters
})

export default store
```

## svg自动导入

1. 下载安装`svg-sprite-loader`

``` 
npm install svg-sprite-loader -D
```

2. 配置`vue.config.js`文件

``` js 
  chainWebpack (config) {

    // set svg-sprite-loader
    config.module
      .rule('svg')
      .exclude.add(resolve('src/svg'))
      .end()
    config.module
      .rule('icons')
      .test(/\.svg$/)
      .include.add(resolve('src/svg'))
      .end()
      .use('svg-sprite-loader')
      .loader('svg-sprite-loader')
      .options({
        symbolId: 'icon-[name]'
      })
      .end()
  }
```

3. 创建`svg-icon`组件

``` html
  <template>
    <svg :class="svgClass" aria-hidden="true" v-on="$listeners">
      <use :href="iconName" />
    </svg>
  </template>
```

```js
export default {
  name: 'SvgIcon',
  props: {
    iconClass: {
      type: String,
      required: true
    },
    className: {
      type: String,
      default: ''
    }
  },
  computed: {
    iconName () {
      return `#icon-${this.iconClass}`
    },
    svgClass () {
      if (this.className) {
        return 'svg-icon ' + this.className
      } else {
        return 'svg-icon'
      }
    }
  }
}
```

``` css
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}

```

4. 自动化导入svg文件

``` js
  const req = require.context('@/svg', false, /\.svg$/)
  const requireAll = requireContext => requireContext.keys().map(requireContext)
  requireAll(req)
```

5. 最终使用

``` js
  import Vue from 'vue'
  import SvgIcon from '@/components/SvgIcon.vue'
  Vue.component('svg-icon', SvgIcon)
```

``` html
  <svg-icon icon-class="wechat"/>
```

> Tip: svg文件目录为：`src/svg` ,svg文件列表为： `wechat.svg、user.svg……`

> [手摸手，带你优雅的使用 icon](https://juejin.im/post/6844903517564436493)