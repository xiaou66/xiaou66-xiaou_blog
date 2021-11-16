---
title: Rollup 打包工具
date: 2021/11/07 13:00
math: true
categories:
  - [webpack]
tags:
  - [Rollup]
---

# Rollup 打包工具

>  [rollupjs](https://rollupjs.org/guide/en/) 下一代 ES 模块捆绑器

# 快速入门

1. 使用命令行工具安装 rollupjs

```bash
yarn add rollup -D 或者 npm install rollup -save-dev
```

2. 创建 「rollup.config.js」 文件

   ```js
   export default {
       // 入口文件
       input: 'src/main.js',
       output: {
           // 编译后文件
           file: 'dist/bundle.cjs.js',
           // 文件输出格式
           format: 'cjs',
           // 包的全局变量名称
           name: 'bundleName'
       }
   }
   ```

3. 创建辅助文件「MyModule.js」文件

   ```js
   const sayHello = (message) => {
       alert(message)
   }
   export default sayHello;
   ```

4. 创建入口「main.js」文件

   ```js
   import sayHello from "./modules/MyModule";
   
   sayHello('hello form Rollup');
   ```

5. 在 package.json 文件中编写运行脚本

   ```json
   "scripts": {
       "build": "rollup -c"
   }
   ```

6. 会在当前工程目录上生成 dist 文件夹 在文件夹中有 bundle.cjs.js 打包后的文件

   ```js
   'use strict';
   
   const sayHello = (message) => {
       alert(message);
   };
   
   sayHello('hello form Rollup');
   ```

7. 可以在 「index.html」文件中使用 bundle.cjs.js

   ```html
   <!DOCTYPE html>
   <html lang="zh">
   <head>
       <title>rollupjs 快速入门</title>
   </head>
   <body>
       <script src="./bundle.cjs.js"></script>
   </body>
   </html>
   ```



## 主要配置

1. input

   入口文件地址

2. output

   ```js
   output:{
       file:'bundle.js', // 输出文件
       format: 'cjs,  //  五种输出格式：amd /  es6 / iife / umd / cjs
       name:'A',  //当format为iife和umd时必须提供，将作为全局变量挂在window(浏览器环境)下：window.A=...
       sourcemap:true  //生成bundle.map.js文件，方便调试
   }
   ```

3. plugins

   配置插件

4. external

   ```js
   external:['lodash'] //告诉rollup不要将此lodash打包，而作为外部依赖
   ```

5. global

   ```js
   global:{
       'jquery':'$' //告诉rollup 全局变量$即是jquery
   }
   ```

   

##  插件使用

### 使用 Babel

1. 安装 rollup-plugin-babel

   ```bash
   yarn add rollup-plugin-babel @babel\core @babel/preset-env -D
   ```

2. 配置 rollup.config.js

   ```js
   plugins: [
       babel({
           // 排除 node_modules 文件夹下, 只编译我们的源代码
           exclude: 'node_modules/**'
       })
   ]
   ```

3. 添加 Babel 配置文件 .babelrc

   ```json
   {
       "presets": [
           [
               "@babel/env",{ "modules": false }
           ]
       ]
   }
   ```

:::info

首先，我们设置 "modules": false ，否则 Babel 会在 Rollup 有机会做处理之前，将我们的模块转成 CommonJS ，导致 Rollup 的一些处理失败。

:::

::: info

第二，我们将 .babelrc 文件放在 src 中，而不是根目录下。 这允许我们对于不同的任务有不同的 .babelrc 配置，比如像测试，如果我们以后需要的话 - 通常为单独的任务单独配置会更好。

:::

最后运行 `npm run build` 我们看到打包后出来的文件内容经过 babel 转换后有 es6 语法变成了 es5 语法

```js
'use strict';

var sayHello = function sayHello(message) {
  alert(message);
};

sayHello('hello form Rollup');
```

### node 模块的引用

#### 安装

在某些时候，您的项目可能取决于从 yarn 安装到 node_modules 文件夹中的软件包。

与 Webpack 和 Browserify 等其他捆绑软件不同，Rollup 不知道如何「开箱即用」如何处理这些依赖项-我们需要添加一些插件配置。

一共需要两个库

1. [rollup-plugin-node-resolve](https://www.npmjs.com/package/@rollup/plugin-node-resolve) 插件允许我们加载第三方模块

2. [@rollup/plugin-commons](https://www.npmjs.com/package/@rollup/plugin-commonjs) 插件将它们转换为ES6版本

```bash
yarn add @rollup/plugin-node-resolve @rollup/plugin-commonjs -D
```

#### 配置 rollup.config.js

```js
plugins: [
    resolve(),
    commonjs(),
    babel({
        // 排除 node_modules 文件夹下, 只编译我们的源代码
        exclude: 'node_modules/**'
    })
]
```

##### 使用一个第三方库 lodash

```bash
yarn add lodash -D
```

修改 「main.js」文件

```js
import sayHello from "./modules/MyModule";
import _ from 'lodash';
const arr = _.concat([1, 2, 3], 4, [5]);
console.log(arr);
sayHello('hello form Rollup');
```

运行 `npm run build` 这时在 「bundle.cjs.js」文件中多出了很多内容这些代码就是 ladash 的代码。

#### 外部引用

在 「rollup.config.js」增加 「external」配置

```js
external: ['lodash']
```

### 使用 TypeScript

1. 安装 @rollup/plugin-typescript

   ```bash
   yarn add tslib typescript @rollup/plugin-typescript  -D
   ```

2. 配置 rollup.config.js

   ```js
   import babel from 'rollup-plugin-babel';
   import commonjs from '@rollup/plugin-commonjs';
   import resolve from '@rollup/plugin-node-resolve';
   import typescript from '@rollup/plugin-typescript';
   export default {
       // 入口文件
       input: 'src/main.js',
       output: {
           // 编译后文件
           file: 'dist/bundle.cjs.js',
           // 文件输出格式
           format: 'cjs',
           // 包的全局变量名称
           name: 'bundleName'
       },
       plugins: [
           typescript(),
           resolve(),
           commonjs(),
           babel({
               // 排除 node_modules 文件夹下, 只编译我们的源代码
               exclude: 'node_modules/**'
           })
       ],
       external: ['lodash']
   }
   ```

3. 创建 tsconfig.json 文件

   ```json
   {
       "compilerOptions": {
           "lib": ["ES6"],
           "module": "esnext",
           "allowJs": true,
       },
       "exclude": [
           "node_modules/**/*"
       ],
       "include": [
           "src/**/*"
       ]
   }
   ```

4. 创建 Hello.ts 文件

   ```tsx
   class Hello {
       greeting: string;
       constructor(message: string) {
           this.greeting = message;
       }
       greet() {
           console.log(this.greet);
   
       }
   }
   export default Hello;
   ```

5. 在 main.js 使用

   ```js
   import Hello from "./modules/Hello";
   const hello = new Hello("123木头人...");
   hello.greet();
   ```
