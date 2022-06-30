# Rollup 学习笔记



![rollup-logo](https://github.com/Mr-Welson/learn/blob/main/rollup/rollup.png?raw=true)

> Q：为什么要学 rollup.js ？
>
> A：因为大家都在卷(roll)

## 介绍

[Rollup 中文文档](https://www.rollupjs.com/) 是对英文文档的翻译，但在配置项一章忽略了部分可用配置及相关的配置介绍，更全更详细的配置项可以查看 [Rollup 英文文档](https://rollupjs.org/guide/en/#introduction) 。

本文根据 Rollup 中文文档整理，并补充了部分英文文档上有的配置项说明。
完整代码地址： [Rollup 笔记](https://github.com/Mr-Welson/learn/tree/main/rollup)

**Rollp 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序**。

Rollup 对代码模块使用新的标准化格式 (**ES6语法**)，而不是以前的特殊解决方案，如 CommonJS 和 AMD。加载 CommonJS 模块和使用 Node 模块位置解析逻辑都被实现为可选插件，需要安装对应插件然后在 `rollup.config.js` 中启用他们，后文中会具体示例。

ES6 模块可以使你自由、无缝地使用你最喜爱的 library 中那些最有用独立函数，而不必携带其他未使用的代码。ES6 模块最终还是要由浏览器原生实现，但当前 Rollup 可以使你提前体验。

Rollup 已被许多主流的 JavaScript 库使用，也可用于构建绝大多数应用程序。但是 Rollup 还不支持一些特定的高级功能，尤其是用在构建一些应用程序的时候，特别是代码拆分和运行时态的动态导入 [dynamic imports at runtime](https://github.com/tc39/proposal-dynamic-import). 如果你的项目中更需要这些功能，那使用 [Webpack](https://webpack.js.org/)可能更符合你的需求。

## 教程

开始前，需要安装 [Node.js](https://nodejs.org/)， 这样才可以使用 [npm](https://npmjs.com/)

### 安装

```
// 全局安装
npm install rollup --global

// 查看版本号
rollup -v
```

### JS-API

Rollup 可以通过 Node.js 来使用 JavaScript API，通常不会这样使用，除非你想扩展 Rollup 本身，或者用于一些难懂的任务。

源文件目录  ` rollup/src/00_api`，共有4个文件，介绍了 `input`、`output` 的常用配置项及简单示例。首先需要在本项目中下载 `rollup` 依赖

```
npm i rollup
```

1. 需要打包的代码  `src/00_api/main.js`

```
import $ from 'jquery'
// 已在 outputOptions 中配置了 jquery 的 cnd 引用及全局变量 $ 申明，故不用在本地下载 jquery 依赖包

function test() {
  console.log('jquery', $);
}

function hello() {
  console.log('hello rollup');
}

module.exports = {
  test, hello
}
```

2. 打包入口文件 `src/00_api/index.js`

```
const rollup = require('rollup');
const inputOptions = require('./inputOptions');
const outputOptions = require('./outputOptions');

async function build() {
  // create a bundle
  const bundle = await rollup.rollup(inputOptions);

  console.log(bundle.imports); // an array of external dependencies
  console.log(bundle.exports); // an array of names exported by the entry point
  console.log(bundle.modules); // an array of module objects

  // generate code and a sourcemap
  // const { code, map } = await bundle.generate(outputOptions);

  // or write the bundle to disk
  await bundle.write(outputOptions);
}

build();
```

3. `rollup` 的` input ` 配置文件: `src/00_api/inputOptions.js`

```
const inputOptions = {
  // 核心参数
  /**
   * 包文件入口 
   * -i, --input
   * String 唯一必填参数
   */
  input: "src/00_api/main.js",
  /**
   * 外链
   * -e, --external
   * object[] 
   */
  external: [
    'jquery'
  ],
  /**
   * 插件
   * object[] 
   */
  plugins: [],

  // 高级参数
  /**
   * 警告监听
   * Function 可以拦截警告信息并进行相应操作
   */
  onwarn(warning) {
    // 跳过某些警告
    if (warning.code === 'UNUSED_EXTERNAL_IMPORT') {
      return
    };
    // 抛出异常
    if (warning.code === 'NON_EXISTENT_EXPORT') {
      throw new Error(warning.message)
    };
    // 部分警告会有一个 loc 和 frame 属性，可以定位到警告的位置
    if (warning.loc) {
      console.warn(`${warning.loc.file} (${warning.loc.line}:${warning.loc.column}) ${message}`);
      if (warning.frame) {
        console.warn(warning.frame)
      };
    } else {
      // 控制台打印一切警告
      console.warn(warning.message);
    }
  },
  /**
   * 缓存
   * 以前生成的包。使用它来加速后续的构建, Rollup只会重新分析已经更改的模块
   * Object | false 
   */
  // cache,

  // 危险参数
  /**
   * 默认情况下，模块的上下文 - 即顶级的this的值为undefined。在极少数情况下，您可能需要将其更改为其他内容，如 'window'。
   */
  context: undefined,
  /**
   * Object | id => context
   * 同 context, 但是可以定义每个模块的上下文。
   */
  // moduleContext,
  // 兼容 IE8 及一些老版本浏览器，通常不需要
  // legacy
};

module.exports = inputOptions;
```

4. `rollup` 的` output` 配置文件: `src/00_api/ioutputOptions.js`

```
const outputOptions = {
  // 核心参数
  /**
   * 输出包或sourcemap的路径和文件名
   * -o, --output.file
   * String
   */
  file: "bundle.js",
  /**
   * 输出包的格式
   * -f, --output.format
   * String 必填, 枚举值：
   * amd – 异步模块定义，用于像RequireJS这样的模块加载器
   * cjs – CommonJS，适用于 Node 和 Browserify/Webpack
   * esm – 将软件包保存为 ES 模块文件，在现代浏览器中可以通过 <script type=module> 标签引入
   * iife – 一个自动执行的功能，适合作为<script>标签。（如果要为应用程序创建一个捆绑包，您可能想要使用它，因为它会使文件大小变小）
   * umd – 通用模块定义，以amd，cjs 和 iife 为一体
   * system - SystemJS 加载器格式
   */
  format: "umd",
  /**
   * 生成包的名称(变量名称)
   * -n, --name
   * String 代表你的 iife/umd 包，同一页上的其他脚本可以访问它
   */
  // name: "MyBundle",
  /**
   * 全局变量申明
   * -g, --globals
   * Object 用于 umd/iife 包
   */
  globals: {
    jquery: '$'
  },

  // 高级参数
  /**
   * 路径
   * Function | Object 可用于配置 externals 的 cdn 引用
   */
  paths: {
    jquery: 'https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js'
  },
  /**
   * 文件前置信息
   * String 自动添加到bundle最前面，可以添加作者、插件地址、版本信息等
   */
  banner: '/* my-library version 1.0 */',
  /**
   * 文件后置信息
   * String 追加到bundle文件末尾的信息
   */
  footer: '/* follow me on Github! */',
  /**
   * 代码包装器
   * String 类似banner/footer
   * intro 会放到正式代码开头
   * outro 会放到正式代码末尾
   * 可在打包的 bundle 中观察具体添加位置
   */
  intro: '/* intro test */',
  outro: '/* outro test */',
  /**
   * -m, --sourcemap
   * Boolean | 'inline' | 'hidden'
   * 默认false: 不创建 sourcemap 
   * true：将创建一个单独的sourcemap文件
   * inline：sourcemap 将作为数据 URI附加到生成的output文件中
   * hidden： 跟 true 类似，但是会屏蔽相应文件的注释信息
   */
  sourcemap: false,
  /**
   * String 生成的包的位置
   */
  // sourcemapFile: '',
  /**
   * Boolean
   * 默认为 true
   */
  // interop,

  // 危险区域
  // exports,
  // amd,
  // indent,
  /**
   * Boolean 默认为true
   * 是否在生成的非ES6包的顶部添加 'use strict' 语法
   */
  strict: true
};

module.exports = outputOptions;
```

5. 使用 `node` 命令执行打包

```
// rollup 根目录
node src\00_api\index.js
```

打包完成后，可以观察 `bundle` 中 `banner/footer/intro/outro`  对应的具体位置

### 命令行打包

源文件目录  ` rollup/src/01_command ` 

- 打包

```
 rollup src/01_command/main.js -f cjs
```

`-f` 选项是 `--output.format` 的缩写，指定了所创建的 bundle 类型为 CommonJS（在 Node.js 中运行），由于没有指定输出文件，所以会直接打印在控制台

- 打包并保存为 ` bundle.js `

```
rollup src/01_command/main.js -o bundle.js -f cjs
```

`-o` 选项是 `--output.file` 的缩写，指定需要保存的文件名称

### 使用配置文件

源文件目录  ` rollup/src/01_command ` ，详细的 `input/output` 配置项可以查看 [JS-API](#JS API) 一章

1. 在项目根目录创建 ` rollup.config.js`

```
export default {
  input: 'src/01_command/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
}
```

2. 打包

```
// 默认使用项目根目录下的配置文件(rollup.config.js) 
rollup -c

// 命令行参数会覆盖配置文件（打包文件命名为 bundle1.js）
rollup -c -o bundle1.js

// 指定配置文件路径(使用 rollup.config.dev.js)
rollup -c rollup.config.dev.js
```

### 使用插件

源文件目录  ` rollup/src/02_plugin` 

这里以读取 json 文件为例，需要使用 ` `rollup-plugin-json` `  插件

1.  初始化 npm 环境

```
npm init -y
```

2. 安装 rollup-plugin-json 插件

```
npm i rollup-plugin-json -D

// 等同于 npm install --save-dev rollup-plugin-json
```

3. 添加 `src/02_plugin/main.js` 文件，读取 `package.json` 中的版本号

```
import { version } from '../../package.json';

export default function () {
  console.log('version ' + version);
}
```

4. 修改  `rollup.config.js` , 使用 `rollup-plugin-json` 插件

```
import json from 'rollup-plugin-json'
export default {
  input: 'src/02_plugin/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [json()]
}
```

5. 执行打包命令

```
rollup -c
```

6. 使用 npm 命令打包

修改 ` package.json `, 添加 `build` 命令

```
{
  "devDependencies": {
    "rollup-plugin-json": "^4.0.0"
  },
  "scripts": {
    "build": "rollup -c"
  }
}
```

7. 执行打包

```
npm run build
```

### 使用 npm packages

源文件目录  ` rollup/src/03_packages`

在项目中我们可能会使用到一些从 npm 安装到 `node_modules` 文件夹中的依赖包，但 Rollup 不知道如何去处理这些依赖，所以我们需要添加一些配置。

[rollup-plugin-node-resolve](https://github.com/rollup/rollup-plugin-node-resolve)  插件可以告诉 Rollup 如何查找外部模块。同时，由于不同的库导出方式不一样，有 `ES6` 导出，也有 `CommonJS` 导出的，而 `Rollup` 是不支持 `CommonJS` 语法的，所以需要使用另一个 插件  [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs)  进行编译。

#### ESModule

依赖导出方式为 ESModule，需要使用  `rollup-plugin-node-resolve`  插件来告诉 `Rollup` 查找外部依赖 。

以`the-answer ` 依赖为例：

1. 安装 `the-answer`

```
npm i the-answer
```

2. 添加 `src/03_packages/esm.js` 文件

```
import answer from 'the-answer';

export function showAnswer() {
  console.log('答案: ' + answer);
}
```

3. 安装 `rollup-plugin-node-resolve` 插件

```
npm i rollup-plugin-node-resolve -D
```

4. 修改  `rollup.config.js` , 使用 `rollup-plugin-node-resolve` 插件，修改打包入口文件

```
import resolve from 'rollup-plugin-node-resolve'

export default {
  input: 'src/plugin/esm.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    resolve()
  ]
}
```

5. 执行打包

```
npm run build
```

#### CommonJS Module

依赖导出方式为 ` CommonJS` ，需要添加 `rollup-plugin-commonjs` 插件将 CommonJS 转换成 ES2015 模块。

以  `dayjs ` 依赖为例：

1. 安装 `dayjs `

```
npm i dayjs
```

2. 添加 `src/03_packages/commonjs.js` 文件

```
import dayjs from 'dayjs';

export default function () {
  console.log('当前时间: ' + dayjs().format('YYYY-MM-DD HH:mm:ss'));
}
```

3. 安装 `rollup-plugin-commonjs` 插件

```
npm i rollup-plugin-commonjs -D
```

4. 修改  `rollup.config.js` , 使用 `rollup-plugin-node-resolve` 插件

```
import resolve from 'rollup-plugin-node-resolve'
import commonjs from 'rollup-plugin-commonjs'

export default {
  input: 'src/packages/commonjs.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    commonjs(),
    resolve()
  ]
}
```

5. 执行打包

```
npm run build
```

### 使用 externals

上述样例解决了 ` packages`  的引用问题，但是查看打包后的 bundle 文件会发现，引用的依赖也被打包进了 bundle，有些情况是我们想要构建一个具有对等依赖关系（peer dependency）的库，例如 React 或 Lodash，即不把这些依赖打包到最终的输出文件中。此时，就需要设置 `externals` 属性。

源文件目录  ` rollup/src/04_externals`

1. 只需要修改配置文件即可，修改  `rollup.config.js` , 设置 `externals` 属性

```

import resolve from 'rollup-plugin-node-resolve'
import commonjs from 'rollup-plugin-commonjs'

export default {
  input: 'src/04_externals/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    commonjs(),
    // 将自定义选项传递给解析插件
    resolve({
      customResolveOptions: {
        moduleDirectory: 'node_modules'
      }
    })
  ],
  // 指出应将哪些模块视为外部模块
  external: ['dayjs']
}
```

2. 执行打包

```
npm run build
```

### 使用 Babel 

源文件目录  ` rollup/src/05_babel `

使用 ES6 语法或其他一些新特性开发已经是一件很平常且高效的事情了，通常都需要使用 `babel.js` 进行编译，在 Rollup 中也不例外，直接使用  [rollup-plugin-babel](https://github.com/rollup/rollup-plugin-babel) 插件就可以实现。

1. 添加 `src/05_babel/main.js` 文件。

```
import dayjs from 'dayjs';

// 使用了 ES6 箭头函数及模板字符串语法
export const showCurrent = () => {
  console.log(`当前时间: ${dayjs().format('YYYY-MM-DD HH:mm:ss')}`);
}
```

 	在未配置  `babel` 转换的情况下，打包出来的文件不会对我们使用的一些新特性语法进行编译，在使用中可能会引起一些兼容性问题。可以在进行以下设置之前，先执行 ` npm run build`  看以下 ` bundle.js` 中的内容和后面作对比。

2. 安装 `rollup-plugin-babel` 

```
npm i rollup-plugin-babel -D
```

3. 修改  `rollup.config.js` ,  使用 `rollup-plugin-babel` 插件

```
// rollup.config.js
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';

export default {
  input: 'src/05_babel.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    resolve(),
    babel({
      exclude: 'node_modules/**' // 只编译我们的源代码
    })
  ]
};
```

4. 创建 ` src/05_babel/.babelrc`  配置文件。与日常开发的项目不同的是，通常 `.babelrc` 放在项目的根目录下，在 Rollup 中，我们将 `.babelrc` 放在你需要打包的入口文件同目录下， 这样就允许我们对于不同的任务有不同的 `.babelrc` 配置。 

```
{
  "presets": [
    ["latest", {
      "es2015": {
        "modules": false
      }
    }]
  ],
  "plugins": ["external-helpers"]
}
```

5. ` .babelrc` 中用到了2个插件，需要安装一下

```
npm i -D babel-core babel-preset-latest babel-plugin-external-helpers
```

6. 执行打包

```
npm run build
```

	这里可能会遇到打包报错的提示，是因为官方文档的示例 (本文也是参考的官方文档) 使用的是 `babel 6`，而 `babel` 现在已经升级到 `babel 7` 了。最快速的解决办法就是切换 ` babel 6` ，当然也可以升级到 `babel 7`，具体升级办法可以自行百度。下文简述使用 `babel 6` 的方法。 

- 使用 `npm view babel-core versions` 查看 `babel` 所有的版本，v6 最后一个版本是  6.26.3，直接安装

```
npm i babel-core@6.26.3 -D
```

- 再次执行 `npm run build `， 还是报错，根据错误信息提示 ` babel-plugin-external-helpers ` 也存在版本不兼容问题，于是同样的降版本

```
npm i babel-core@6.22.0 -D
```

- 再次执行 `npm run build` ，打包成功。

查看此时打包后的 bundle.js 就会发现箭头函数已经变成了普通 `function` ，模板字符串也变成了字符串拼接，可以确保 js 能在较低版本的浏览器中顺利执行。

### 使用 Gulp 

[Gulp官网](https://www.gulpjs.com.cn/docs/getting-started/quick-start/)

须先安装 `gulp` 环境

```
npm i gulp-cli -g
npm i gulp -D
```

源文件目录  ` rollup/src/06_gulp `

`rollup` 的 `js-api` 返回 `promise`，可以很容易的跟 `Gulp` 结合

1. 创建 `src/06_gulp/gulpfile.js` ， `gulpfile.js` 为 `gulp` 读取的文件，文件名不能修改

```
const gulp = require('gulp');
const rollup = require('rollup');

gulp.task('build', async function () {
  const bundle = await rollup.rollup({
    input: './main.js',
  });

  await bundle.write({
    file: 'bundle.js',
    format: 'umd',
    name: 'library'
  });
});
```

2. 源码文件   `src/06_gulp/main.js`

```

function hello() {
  console.log('hello gulp');
}

module.exports = {
  hello
}
```

3. 在 `src/06_gulp` 文件目录下执行 `gulp` 命令

```
gulp build
```

4. 执行完成，会在 `src/06_gulp` 目录下生成 `bundle.js`


## 仓库

> 完