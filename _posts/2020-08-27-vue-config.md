---
layout: post
title: "vue.config.js常见配置项"
subtitle: vue-cli 中 vue.config.js 中关于webpack的配置知识点
date: 2020-08-26 20:10
author: "mipaifu328"
header-img: "img/about-bg.jpg"
tag: 
    - VUE
    - webpack
---

> vue.config.js配置文档，请移步之[官网文档](https://cli.vuejs.org/zh/config/#vue-config-js)  和 [webpack 相关](https://cli.vuejs.org/zh/guide/webpack.html)。

# 用configureWebpack简单的配置


configureWebpack可以使用2种方式定义：
1. `对象` , 该对象将会被 [webpack-merge](https://github.com/survivejs/webpack-merge) 合并入最终的 webpack 配置。  
  ``` js
    // vue.config.js
    module.exports = {
      configureWebpack: {
        plugins: [
          new MyAwesomeWebpackPlugin()
        ]
      }
    }
  ```
2. `函数` , 如果你需要基于环境有条件地配置行为，或者想要直接修改配置，那就换成一个函数 (该函数会在环境变量被设置之后懒执行)。该方法的第一个参数会收到已经解析好的配置。在函数内，你可以直接修改配置，或者返回一个将会被合并的对象。  
  ``` js
    // vue.config.js
    module.exports = {
      configureWebpack: config => {
        if (process.env.NODE_ENV === 'production') {
          // 为生产环境修改配置...
        } else {
          // 为开发环境修改配置...
        }
      }
    }
  ```

# 用chainWebpack做高级配置

Vue CLI 内部的 webpack 配置是通过 [webpack-chain](https://github.com/mozilla-neutrino/webpack-chain) 维护的。这个库`提供了一个 webpack 原始配置的上层抽象`，使其可以定义具名的 loader 规则和具名插件，并有机会在后期进入这些规则并对它们的选项进行修改。

它允许我们`更细粒度`的控制其内部配置。接下来有一些常见的在 vue.config.js 中的 chainWebpack 修改的例子。

1. 修改 Loader 选项
  ``` js
    // vue.config.js
    module.exports = {
      chainWebpack: config => {
        config.module
          .rule('vue')
          .use('vue-loader')
            .loader('vue-loader')
            .tap(options => {
              // 修改它的选项...
              return options
            })
      }
    }
  ```
  > 提示  
    对于 `CSS 相关 loader` 来说，我们推荐使用 `css.loaderOptions` 而不是直接链式指定 loader。这是因为每种 `CSS 文件类型都有多个规则`，而 css.loaderOptions 可以确保你通过一个地方影响所有的规则`。  
2. 添加一个新的 Loader  
  ``` js
    // vue.config.js
    module.exports = {
      chainWebpack: config => {
        // GraphQL Loader
        config.module
          .rule('graphql')
          .test(/\.graphql$/)
          .use('graphql-tag/loader')
            .loader('graphql-tag/loader')
            .end()
          // 你还可以再添加一个 loader
          .use('other-loader')
            .loader('other-loader')
            .end()
      }
    }
  ```
3. 替换一个规则里的 Loader
  ``` js
    // vue.config.js
    module.exports = {
      chainWebpack: config => {
        const svgRule = config.module.rule('svg')

        // 清除已有的所有 loader。
        // 如果你不这样做，接下来的 loader 会附加在该规则现有的 loader 之后。
        svgRule.uses.clear()

        // 添加要替换的 loader
        svgRule
          .use('vue-svg-loader')
            .loader('vue-svg-loader')
      }
    }
  ```
4. 修改插件选项
  ``` js
    // vue.config.js
    module.exports = {
      chainWebpack: config => {
        config
          .plugin('html')
          .tap(args => {
            return [/* 传递给 html-webpack-plugin's 构造函数的新参数 */]
          })
      }
    }
  ```

> 详细文档可以查看 [webpack-chain 的 API](https://github.com/mozilla-neutrino/webpack-chain#getting-started) ，比起直接修改 webpack 配置，它的表达能力更强，也更为安全。

# 审查项目的 webpack 配置

因为 `@vue/cli-service` 对 `webpack` 配置进行了抽象，所以理解配置中包含的东西会比较困难，尤其是当你打算自行对其调整的时候。

`vue-cli-service` 暴露了 `inspect` 命令用于审查解析好的 `webpack` 配置。那个全局的 `vue` 可执行程序同样提供了 `inspect` 命令，这个命令只是简单的把 `vue-cli-service inspect` 代理到了你的项目中。

``` shell
  # 输出重定向到一个文件以便进行查
  vue inspect > output.js

  # 只审查第一条规则
  vue inspect module.rules.0

  # 指向一个规则或插件的名字 
  vue inspect --rule vue
  vue inspect --plugin html

  # 列出所有规则和插件的名字
  vue inspect --rules
  vue inspect --plugins 
```

# 常见的一些配置

1. 配置目录别名  
``` js
  module.exports = {  
      // 方式1
      chainWebpack: config => {
        config.resolve.alias
          .set('@', resolve('src'))
          .set('assets', resolve('src/asset'))
      }  
      // 方式2
      configureWebpack: {
        resolve: {
          alias: {
            '@': resolve('src'),
            'assets', resolve('src/asset')
          }
        }
      }
  }
``` 
后续补充……

