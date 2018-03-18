# ts_log1
## 参考写前头：
* [入门材料](https://basarat.gitbooks.io/typescript/content/docs/getting-started.html)
* [关于noImplicitAny](https://basarat.gitbooks.io/typescript/docs/options/noImplicitAny.html)
* [关于strictNullChecks](https://basarat.gitbooks.io/typescript/docs/options/strictNullChecks.html)
* [TypeScript handbook](https://www.typescriptlang.org/docs/tutorial.html)
## 配置TypeScript
1:通过`yarn init`新建项目，然后在项目中加入TypeScript`yarn add typescript`.

2:介绍一个tsc工具，这个工具可以编译 TypeScript，例执行`tsc test1.ts`可以将ts转换成功js文件。<br />当然说到这里，我们一定想到了这个检查和转换过程是经过默认配置项的。So，我们可以通过`tsc --init`或者`./node_modules/.bin/tsc --init` 获取到这个配置文件，并且根据个人习惯修改配置。
<br />以下是比较适合于初学者的配置：

```
{
  "compilerOptions": {
    "module": "es6", // 使用 ES2015 模块（Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. ）
    "target": "es6", // 编译成 ES2015 (Babel 将做剩下的事情) 'ES3' (default) 
    "allowSyntheticDefaultImports": true, // Allow default imports from modules with no default export. This does not affect code emit, just typechecking
    "baseUrl": "src", // 可以相对这个目录 import 文件
    "sourceMap": true, // 使 TypeScript 生成 sourcemaps
    "outDir": "ts-build", // 构建输出目录 (因为我们大部分时间都在使用 Webpack，所以不太相关)
    "jsx": "preserve", // 开启 JSX 模式, 但是 "preserve" 告诉 TypeScript 不要转换它(我们将使用 Babel)
    "strict": true,//严格模式1:noImplicitAny取消引入变量any的默认值，不声明变量的类型时会报错。2: strictNullChecks检查a.b.c格式，当以免a.b为undefined时候出现的报错。
  },
  "exclude": [
    "node_modules" // 这个目录下的代码不会被 typescript 处理
  ]
}
```
## 配置webpack
因为 TypeScript 输出 React 和 es6。所以我们还需要 babel。让我们安装 Webpack，Babel 和相关的 presets 及 ts-loader，ts-loader 是 TypeScript 在 Webpack 中的插件。<br />awesome-typescript-loader 也是同样的ts插件，但是在webpack4.0中发生热重载的时候会报错<br />
`yarn add webpack babel-core babel-loader babel-preset-es2015 babel-preset-react ts-loader webpack-dev-server` 安装依赖。

以下是webpack配置。
```
const webpack = require('webpack')
const path = require('path')

module.exports = {
  // 设置 sourcemaps 为 eval 模式，将模块封装到 eval 包裹起来
  devtool: 'eval',

  // 我们应用的入口, 在 `src` 目录 (我们添加到下面的 resolve.modules):
  entry: [
    'index.tsx'
  ],

  // 配置 devServer 的输出目录和 publicPath
  output: {
    filename: 'app.js',
    publicPath: 'dist',
    path: path.resolve('dist')
  },

  // 配置 devServer 
  devServer: {
    port: 3000,
    historyApiFallback: true,
    inline: true,
  },

  // 告诉 Webpack 加载 TypeScript 文件
  resolve: {
    // 首先寻找模块中的 .ts(x) 文件, 然后是 .js 文件
    extensions: ['.ts', '.tsx', '.js'],

    // 在模块中添加 src, 当你导入文件时，可以将 src 作为相关路径
    modules: ['src', 'node_modules'],
  },

  module: {
    loaders: [
      // .ts(x) 文件应该首先经过 Typescript loader 的处理, 然后是 babel 的处理
      { test: /\.tsx?$/, loaders: ['babel-loader', 'ts-loader'], include: path.resolve('src') }
    ]
  },
}
```
从webpack配置中可以看到.ts(x)文件先按照tsconfig的配置执行ts-loader，输出ES2015，然后，我们使用 Babel 将它降级到 ES5。所以我们需要配置presets 的 .babelrc 文件：<br />
我们新增本地服务脚本
```
{
  "presets": ["es2015", "react"]
}
```
```
"scripts": {
  "start": "webpack-dev-server"
}
```
## 接入React
首先我们建立 src/index.tsx 为改项目的入口文件。常规写法如下

```
import React from 'react'
import ReactDOM from 'react-dom'

const App = () => {
  return (
    <div>
      <p>Hello world!</p>
    </div>
  )
}
```
这时候运行webpack，你会发现，如下报错：<br />Cannot find module 'react'.<br />Cannot find module 'react-dom'.
<br />发生上面的错误是因为 TypeScript 试图确认 React，React DOM 的类型、React，React DOM 导出了什么。<br />React,React DOM 并不是使用 TypeScript 编写的，所以它并没有包含那些信息。<br />所以我们要引入@type/xx申明文件。
```
yarn add @types/react
yarn add @types/react-dom
```
最后我们再运行
`yarn start`，可以发现http://localhost:3000/，显示hello world!

## 关于TS的.d.ts文件的作用
原则上，TypeScript 需要开发者做到先声明后使用。<br />这就导致开发者在调用很多原生接口（浏览器、Node.js）或者第三方模块的时候，因为某些全局变量或者对象的方法并没有声明过，导致编译器的类型检查失败。

```
$.get('/'); // => Can not find name '$'
```

```
declare var $: any;
$.get('/') // => ok
```
一组 declare 的合集组成了 .d.ts 文件。同 .ts 的区别在于 .d.ts 不会生成额外的代码（因为 declare 语句不会生成代码）。<br />
[更多关于.d.ts](https://segmentfault.com/a/1190000009247663)
