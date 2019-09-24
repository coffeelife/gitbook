# webpack使用学习

本分享学习借鉴webpack中文官网，官网链接(中文文档)：https://www.webpackjs.com

此教程本人demo 地址(develop分支代码)：[https://github.com/coffeelife/WebpackStudy/tree/develop](https://github.com/coffeelife/WebpackStudy/tree/develop)

### 使用github Demo教程

```
git clone https://github.com/coffeelife/WebpackStudy.git
cd WebpackStudy
npm install
webpack//安装了webpack的前提下
webpack-dev-server//安装了webpack-dev-server的情况下，在浏览器localhost:8080/dist/index.html查看效果
```



![5cac4aeae47e4](https://i.loli.net/2019/04/09/5cac4aeae47e4.png)

## 概念

本质上，*webpack*是一个现代 JavaScript 应用程序的*静态模块打包器(module bundler)*。当 webpack 处理应用程序时，它会递归地构建一个*依赖关系图(dependency graph)*，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个*bundle*。

### 四个核心概念

- 入口(entry)
- 输出(output)
- loader
- 插件(plugins)

## 入口(entry)

**入口起点(entry point)**指示 webpack 应该使用哪个模块，来作为构建其内部*依赖图*的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。可以通过在webpack 配置中配置`entry`属性，来指定一个入口起点（或多个入口起点）。默认值为`./src`。

### 单入口模式

用法：`entry: string|Array<string>`

**webpack.config.js**

```
//简写
module.exports = {
  entry: './path/to/my/entry/file.js'
};

module.exports = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

### 多入口模式

用法：`entry: {[entryChunkName: string]: string|Array<string>}`

**webpack.config.js**

```
module.exports = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

**根据经验：每个 HTML 文档只使用一个入口起点。**

## 出口(output)

### 用法(Usage)

**output**属性告诉 webpack 在哪里输出它所创建的*bundles*，以及如何命名这些文件，默认值为`./dist`。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个`output`字段，来配置这些处理过程。

在 webpack 中配置`output`属性的最低要求是，将它的值设置为一个对象，包括以下两点：

- `filename`用于输出文件的文件名。
- 目标输出目录`path`的绝对路径。

### 单入口起点

**webpack.config.js**

```
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

我们通过`output.filename`和`output.path`属性，来告诉 webpack bundle 的名称，以及我们想要 bundle 生成(emit)到哪里。

### 多入口起点

**webpack.config.js**

```
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}
//写入到硬盘：./dist/app.js, ./dist/search.js
```

如果配置创建了多个单独的 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用占位符(substitutions)来确保每个文件具有唯一的名称。

## 模式(mode)

### 用法

只在配置中提供`mode`选项：

```
module.exports = {
  mode: 'production'
};
```

或者从CLI参数中传递：

```
webpack --mode=production
```

| 选项          | 描述                                                                                                                                                                                                                     |
|:-----------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| development | 会将`process.env.NODE_ENV`的值设为`development`。启用`NamedChunksPlugin`和`NamedModulesPlugin`。                                                                                                                                  |
| production  | 会将`process.env.NODE_ENV`的值设为`production`。启用`FlagDependencyUsagePlugin`,`FlagIncludedChunksPlugin`,`ModuleConcatenationPlugin`,`NoEmitOnErrorsPlugin`,`OccurrenceOrderPlugin`,`SideEffectsFlagPlugin`和`UglifyJsPlugin`. |

 **mode: development**

```
// webpack.development.config.js
module.exports = {
+ mode: 'development'
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

**mode: production**

```
// webpack.production.config.js
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]
}
```

## loader

loader 用于对模块的源代码进行转换。loader 可以使你在`import`或"加载"模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的强大方法。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中`import`CSS文件！

### 安装

```
npm install --save-dev css-loader
npm install --save-dev ts-loader
```

**webpack.config.js**

```
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
};
```

指示 webpack 对每个`.css`使用[`css-loader`](https://www.webpackjs.com/loaders/css-loader)，以及对所有`.ts`文件使用[`ts-loader`](https://github.com/TypeStrong/ts-loader)：

### 使用loader

在你的应用程序中，有三种使用 loader 的方式：

- 配置（推荐）：在**webpack.config.js**文件中指定 loader。
- 内联：在每个`import`语句中显式指定 loader。
- CLI：在 shell 命令中指定它们。

### 配置(Configuration)

[`module.rules`](https://www.webpackjs.com/configuration/module/#module-rules)允许你在 webpack 配置中指定多个 loader。 这是展示 loader 的一种简明方式，并且有助于使代码变得简洁。同时让你对各个 loader 有个全局概览：

```
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
    ]
  }
```

### 内联

可以在`import`语句或任何等效于 "import" 的方式中指定 loader。使用`!`将资源中的 loader 分开。分开的每个部分都相对于当前目录解析。

```
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

通过前置所有规则及使用`!`，可以对应覆盖到配置中的任意 loader。

选项可以传递查询参数，例如`?key=value&foo=bar`，或者一个 JSON 对象，例如`?{"key":"value","foo":"bar"}`。

> 尽可能使用`module.rules`，因为这样可以减少源码中的代码量，并且可以在出错时，更快地调试和定位 loader 中的问题。

### CLI

你也可以通过 CLI 使用 loader：

```
webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
```

这会对`.jade`文件使用`jade-loader`，对`.css`文件使用[`style-loader`](https://www.webpackjs.com/loaders/style-loader)和[`css-loader`](https://www.webpackjs.com/loaders/css-loader)。

### loader特性

loader 通过（loader）预处理函数，为 JavaScript 生态系统提供了更多能力。 用户现在可以更加灵活地引入细粒度逻辑，例如压缩、打包、语言翻译和其他更多。

- loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。
- loader 可以是同步的，也可以是异步的。
- loader 运行在 Node.js 中，并且能够执行任何可能的操作。
- loader 接收查询参数。用于对 loader 传递配置。
- loader 也能够使用`options`对象进行配置。
- 除了使用`package.json`常见的`main`属性，还可以将普通的 npm 模块导出为 loader，做法是在`package.json`里定义一个`loader`字段。
- 插件(plugin)可以为 loader 带来更多特性。
- loader 能够产生额外的任意文件。

## 插件(plugins)

插件是 webpack 的支柱功能。webpack 自身也是构建于，你在 webpack 配置中用到的**相同的插件系统**之上！插件目的在于解决loader无法实现的**其他事**。

### 剖析

webpack**插件**是一个具有[`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)属性的 JavaScript 对象。`apply`属性会被 webpack compiler 调用，并且 compiler 对象可在**整个**编译生命周期访问。

**ConsoleLogOnBuildWebpackPlugin.js**

```
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
    apply(compiler) {
        compiler.hooks.run.tap(pluginName, compilation => {
            console.log("webpack 构建过程开始！");
        });
    }
}
```

compiler hook 的 tap 方法的第一个参数，应该是驼峰式命名的插件名称。建议为此使用一个常量，以便它可以在所有 hook 中复用。

### 用法

由于**插件**可以携带参数/选项，你必须在 webpack 配置中，向`plugins`属性传入`new`实例。

根据你的 webpack 用法，这里有多种方式使用插件。

### 配置

**webpack.config.js**

```
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

## 项目使用webpack

本项目使用yarn操作，mac下新建目录WebpackStudy目录

### 初始化项目

```
yarn init
```

选项一直`enter`，init操作之后生成`package.json`和`yarn.lock`文件

**package.json**

```
{
  "name": "WebpackStudy",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT"
}
```

### 项目下安装使用webpack

#### 安装配置

通过yarn全局安装webpack,这里我直接安装的是最新版的

```
yarn global add webpack
yarn global add webpack-cli
webpack -v //输出webpack版本号说明成功
```

**package.json**

```
"dependencies": {
    "webpack": "^4.30.0",
    "webpack-cli": "^3.3.0"
  }
```

#### 新建webpack.config.js

新建webpack.config.js文件并将下面的代码拷入文件中

```
const path = require("path");

module.exports = {
  entry: "./src/js/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js"
  }
};
```

`entry`表示入口文件，此处新建src目录，并在该目录下新建一个`index.js`文件，作为打包文件的入口文件

`output`表示出口文件，`path`表示输出文件的目录，`path.resolve`表示输出文件目录的解析,`__dirname`表示当前目录(注意是双_)，`dist`表示输出文件的目录，`filename`表示输出文件的名称

**当前文件目录结构**

![5cb4323402393](https://i.loli.net/2019/04/15/5cb4323402393.png)

**执行打包语句**

```
webpack//全局安装运行
node_modules/.bin/webpack//非全局安装下使用
```

如果全局安装的话直接执行`webpack`就可以，如果不是的话执行`node_modules`下的`.bin`下的webpack，执行完就发现目录多了dist文件目录和该目录下的index.js

**注意提示**(这里我在config文件中添加了mode:none再次运行没有该提示了)

![5cb4357f33afb](https://i.loli.net/2019/04/15/5cb4357f33afb.png)

```
localhost:WebpackStudy gm$ webpack
Hash: 99630336d25493823585
Version: webpack 4.30.0
Time: 61ms
Built at: 2019-04-15 15:41:00
   Asset      Size  Chunks             Chunk Names
index.js  3.57 KiB       0  [emitted]  main
Entrypoint main = index.js
[0] ./src/js/index.js 0 bytes {0} [built]
```

同样可以编辑index.js，运行webpack查看编译文件变化。

### 使用HtmlWebpackPlugin

该插件用来打包html文件，直接在官网搜索该插件就有教程，链接网址：[https://webpack.js.org/plugins/html-webpack-plugin/#root](https://webpack.js.org/plugins/html-webpack-plugin/#root)

github详细说明网址：[https://github.com/jantimon/html-webpack-plugin#options](https://github.com/jantimon/html-webpack-plugin#options)

#### 安装HtmlWebpackPlugin

```
yarn add html-webpack-plugin --dev
```

#### 配置webpack.config.js文件

**webpack.config.js**

```
const path = require("path");
var HtmlWebpackPlugin = require('html-webpack-plugin');//引入插件

module.exports = {
  mode: "none",
  entry: "./src/js/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js"
  },
  plugins: [new HtmlWebpackPlugin()]//引入插件
};
```

#### 打包

```
webpack
```

运行之后发现在`dist`目录下生成一个默认的index.html，并自动引入index.js文件，代码如下

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
  </head>
  <body>
  <script type="text/javascript" src="index.js"></script></body>
</html>
```

#### 配置属性(常用)

| 名称       | 类型       | 默认           | 描述                                                            |
| -------- | -------- |:------------ | ------------------------------------------------------------- |
| title    | {String} | Webpack App  | 用于生成的HTML文档的标题                                                |
| filename | {String} | 'index.html' | 要将HTML写入的文件。默认为index.html。您可以在这里指定一个子目录太（如：assets/admin.html） |
| template | {String} | ``           | webpack模板的相对或绝对路径。默认情况下，src/index.ejs如果它存在，它将使用               |

**webpack.config.js**

```
plugins: [new HtmlWebpackPlugin({
      title: 'MyApp',
      filename: 'index.html'
      template: "src/index.html"
  })]
```

此处我们将原来建好的index.html文件添加一些内容

**index.html**

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewPort" content="width=device-width,initial-scale=1">
        <meta name="author" content="gm">
        <title>WebpackStudy</title>
    </head>
    <body>
        <div>WebpackStudy</div>
    </body>
</html>
```

运行`webpack`进行文件打包，会发现`dist`下的`index.html`文档变成自己编写的文件

### 安装使用babel-loader

该loader用来打包脚本文件，官网链接网址：[https://webpack.js.org/loaders/babel-loader/#root](https://webpack.js.org/loaders/babel-loader/#root)

#### 安装babel-loader

```
yarn add babel-loader @babel/core @babel/preset-env --dev
```

安装完成之后`package.json`文件如下,增加`babel-loader`解析脚本文件

**webpack.config.js**

```
"devDependencies": {
    "@babel/core": "^7.4.3",
    "@babel/preset-env": "^7.4.3",
    "babel-loader": "^8.0.5",
    "html-webpack-plugin": "^3.2.0"
  }
```

#### 使用babel-loader

配置webpack.config.js

```
module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /(node_modules)/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"]
          }
        }
      }
    ]
  }
```

`rules`表示处理规则；`test`表示处理文件的正则匹配文件，这里表示.js文件；`exclude`表示对该目录下的文件不进行处理；`use`表示使用babel-loader进行处理。

#### 安装使用babel-present-react插件

使用babel-present-react来处理react文件

**安装**

```
yarn add babel-preset-react --dev
```

安装之后package.json文件修改为

**package.json**

```
"devDependencies": {
    "@babel/core": "^7.4.3",
    "@babel/preset-env": "^7.4.3",
    "babel-loader": "^8.0.5",
    "babel-preset-react": "^6.24.1",//增加这条
    "html-webpack-plugin": "^3.2.0"
  }
```

修改webpack.config.js，来处理react文件

**webpack.config.js**

```
module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /(node_modules)/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env","react"]//此处增加react选项
          }
        }
      }
    ]
  },
```

#### 将react添加到当前项目中

```
yarn init//刚才执行过，可以不执行
yarn add react react-dom
```

运行该命令之后`package.json`文件增加`react`和`react-dom`

**package.json**

```
"dependencies": {
    "react": "^16.8.6",
    "react-dom": "^16.8.6",
    "webpack": "^4.30.0",
    "webpack-cli": "^3.3.0"
  }
```

将`index.js`文件修改修改为`index.jsx`，并参照react官网网址：[https://react.docschina.org/](https://react.docschina.org/) 添加内容

**index.jsx**

```
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, Webpack!</h1>,
  document.getElementById('root')
);
```

并将index.html文件的div添加id="root"

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewPort" content="width=device-width,initial-scale=1">
        <meta name="author" content="gm">
        <title>WebpackStudy</title>
    </head>
    <body>
        <div id="root">WebpackStudy</div>
    </body>
</html>
```

修改`webpack.config.js`配置文件修改入口修改为`index.jsx`，并将`babel-loader`中的`rules`规则文件名改为`.jsx`

**webpack.config.js**

```
const path = require("path");
var HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "none",
  entry: "./src/js/index.jsx",//此处更改![5cb58851485eb](https://i.loli.net/2019/04/16/5cb58851485eb.png)
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js"
  },
  module: {
    rules: [
      {
        test: /\.m?jsx$/,//此处更改
        exclude: /(node_modules)/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env", "react"]
          }
        }
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: "MyApp",
      filename: "index.html",
      template: "src/index.html"
    })
  ]
};
```

点击运行webpack之后会发现报错(报错截图如下)，此处是由于`babel-loader`、`babel-core`、`babel-preset`版本问题导致

![5cb5886b939c2](https://i.loli.net/2019/04/16/5cb5886b939c2.png)

![5cb5887b734a8](https://i.loli.net/2019/04/16/5cb5887b734a8.png)

此处需要将版本使用6.x的版本，解决办法如下

```
yarn add babel-core@6.26.0 babel-loader@7.1.2 babel-preset-env@1.6.1 --dev
```

并将webpack.config.js文件，`rules`修改为以下内容

```
module: {
    rules: [
      {
        test: /\.m?jsx$/,
        exclude: /(node_modules)/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["env", "react"]//将它修改为env使用低版本
          }
        }
      }
    ]
  }
```

在运行就会发现已经打包成功，用浏览器打开就会发现内容修改为`Hello Webpack`

![5cb59a831c36b](https://i.loli.net/2019/04/16/5cb59a831c36b.png)

#### .babelsrc文件配置

可以参考网址：[https://excaliburhan.com/post/babel-preset-and-plugins.html](https://excaliburhan.com/post/babel-preset-and-plugins.html)

### 安装使用css-loader和style-loader

该loader用来解析样式文件，css-loader官网网址：[https://webpack.js.org/loaders/css-loader/#root](https://webpack.js.org/loaders/css-loader/#root)，style-loader官网网址：[https://webpack.js.org/loaders/style-loader/#root](https://webpack.js.org/loaders/style-loader/#root)

#### 安装

```
yarn add css-loader style-loader --dev
```

安装完成之后配置webpack.config.js文件

**webpack.config.js**

```
module: {
    rules: [
      {
        test: /\.m?jsx$/,
        exclude: /(node_modules)/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["env", "react"]
          }
        }
      },
      {//添加如下配置
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      }
    ]
  }
```

在目录添加`css`目录，并创建`index.css`文件

![5cb59e991690e](https://i.loli.net/2019/04/16/5cb59e991690e.png)

在`index.css`添加代码

```
#root{
    color: red;
}
```

**注意：此时运行webpack，发现样式并不会生效，此时需要在`.jsx`文件中引入样式文件**

运行webpack,查看样式变化，文字变红

**index.jsx**

```
import React from 'react';
import ReactDOM from 'react-dom';
import '../css/index.css';//加入这一行

ReactDOM.render(
  <h1>Hello, Webpack!</h1>,
  document.getElementById('root')
);
```

### css样式加载优化处理（使用ExtractTextWebpackPlugin）

如果这种方式引入，你会发现`dist`文件下，并没有样式文件，在`index.js`搜索发现如下代码

**index.js**

```
exports.push([module.i, "#root{\n    color: red;\n}", ""]);
```

根据html解析顺序的话，css需要等待所有的js代码加载完成才会进行样式加载，这样页面会有很大的一段白屏时间，接下来进行css样式加载优化处理。链接网址：[https://blog.csdn.net/qq_39793127/article/details/78900707](https://blog.csdn.net/qq_39793127/article/details/78900707)

#### 安装ExtractTextWebpackPlugin

```
yarn add extract-text-webpack-plugin --dev
```

#### 使用ExtractTextWebpackPlugin

在webpack.config.js配置文件中引入该插件，并将`css`文件的打包方式改为`ExtractTextWebpackPlugin`插件的方式

**webpack.config.js**

```
const path = require("path");
var HtmlWebpackPlugin = require("html-webpack-plugin");
const ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = {
  mode: "none",
  entry: "./src/js/index.jsx",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "index.js"
  },
  module: {
    rules: [
      ...//由于引入太多使用省略号隐藏
      {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: "css-loader"
        })
      }
    ]
  },
  plugins: [
    ...//由于引入太多使用省略号隐藏
    new ExtractTextPlugin("index.css")
  ]
};
```

运行webpack报错，错误如下，百度可知，插件当前版本不支持webpack4，使用beta版本可解决该问题![5cb5a5d47bbb3](https://i.loli.net/2019/04/16/5cb5a5d47bbb3.png)

解决方法，安装beta 版本

```
yarn add extract-text-webpack-plugin@next --dev
```

再次运行webpack，执行成功，`dist`下增加`index.css`文件

### 打包Sass样式文件(sass-loader)

现在进行sass文件进行打包，链接地址：[https://webpack.js.org/plugins/extract-text-webpack-plugin/#extracting-sass-or-less](https://webpack.js.org/plugins/extract-text-webpack-plugin/#extracting-sass-or-less)

#### 安装sass-loader

```
yarn add sass-loader node-sass --dev
```

**注意，sass-loader以来node-sass和webpack，使用sass-loader必须安装另外两个**

在webpack.config.js配置`rules`规则，依旧使用ExtractTextWebpackPlugin处理

**webpack.config.js**

```
{
        test: /\.scss$/,
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: ["css-loader", "sass-loader"]
        })
      }
```

新建`index.scss`文件，并更改样式文件，将背景更改为灰色，并将字体变成30px大小

**index.scss**

```
body{
    background: gray;
    #root{
        font-size: 30px;
    }
}
```

在`index.jsx`文件中引入`index.scss`样式文件

**index.jsx**

```
import React from 'react';
import ReactDOM from 'react-dom';
import '../css/index.css';
import '../css/index.scss';//添加这一行代码

ReactDOM.render(
  <h1>Hello, Webpack!</h1>,
  document.getElementById('root')
);
```

运行webpack之后发现`dist`目录下的`index.css`文件改变，在网页打开发现网页北京变成灰色，字体变大

index.css

```
#root{
    color: red;
}
body {
    background: gray; 
}
body #root {
    font-size: 30px; 
}
```

![5cb5abb1c85dc](https://i.loli.net/2019/04/16/5cb5abb1c85dc.jpg)

### 安装使用资源处理(file-loader和url-loader)

本文用file-loader的封装url-loader，`url-loader`类似于[`file-loader`](https://webpack.js.org/loaders/file-loader/)，但如果文件小于字节限制，则可以返回DataURL,file-loader官网链接：[https://webpack.js.org/loaders/file-loader/#root](https://webpack.js.org/loaders/file-loader/#root)，url_loader官网链接：[https://webpack.js.org/loaders/url-loader/#root](https://webpack.js.org/loaders/url-loader/#root)

#### 安装url-loader

```
yarn add file-loader url-loader --dev
//npm install url-loader --save-dev
```

**注意：url-loader依赖于file-loader,所以file-loader也需要安装**

#### 使用url-loader

配置webpack.config.js文件，在`rules`添加`url-loader`配置

**webpack.config.js**

```
{
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192
            }
          }
        ]
      }
```

`test`表示匹配的后缀名，`loader`表示使用`url-loader`，`option`中的`limit`表示大于8k生成图片，小于8k生成`DataURL`，这里自己照一张图片放入项目的`image`目录下，执行webpack会发现`dist`目录下多了一张打包的图片，修改index.scss的背景样式，在浏览器查看效果

```
body{
    background: url(../image/test.jpg);
    background-size: 100%;
    #root{
        font-size: 30px;
    }
}
```

![5cb5b169e13cb](https://i.loli.net/2019/04/16/5cb5b169e13cb.png)

### 使用字体文件font-awesome()

文字文件使用font-awesome，font-awesome官网地址：[http://www.fontawesome.com.cn/faicons/](http://www.fontawesome.com.cn/faicons/)

#### 安装font-awesome字体文件

```
yarn add font-awesome
```

#### 使用font-awesome

在`index.jsx`文件中编写，引入`font-awesome`里面的`css`文件，并在标签上加入`className`属性，放入`fontawesome`的样式

**index.jsx**

```
import React from "react";
import ReactDOM from "react-dom";
import "../css/index.css";
import "../css/index.scss";
import "font-awesome/css/font-awesome.min.css";//此处为添加代码

ReactDOM.render(
  <div>
    <i className="fa fa-address-book"/>//此处为添加代码
    <h1>Hello, Webpack!</h1>
  </div>,
  document.getElementById("root")
);
```

这时候运行webpack会如下错误，原因是打包配置里面没有配置这些类型文件的解析，接下来我们直接用url-loader在`rules`中来进行字体文件的配置

![5cb5bcec9faf5](https://i.loli.net/2019/04/16/5cb5bcec9faf5.png)

**webpack.config.js**

```
{
        //font-awesome字体文件处理
        test: /\.(eot|svg|ttf|woff|woff2)$/,
        use: [
          {
            loader: "url-loader",
            options: {
              limit: 8192
            }
          }
        ]
      }
```

这样运行之后会发现dist目录下增加了字体文件，在浏览器中打开就可以看到效果![5cb5bddfd2eca](https://i.loli.net/2019/04/16/5cb5bddfd2eca.png)

### 公共模块提出(CommonsChunkPlugin)

该插件是webpack内部的插架，直接在webpack.config.js直接配置就可以，直接配置

**webpack.config.js**

```
new webpack.optimize.CommonsChunkPlugin({
  name: 'commons',
  // (the commons chunk name)

  filename: 'commons.js',
  // (the filename of the commons chunk)

  // minChunks: 3,
  // (Modules must be shared between 3 entries)

  // chunks: ["pageA", "pageB"],
  // (Only use these entries)
});
//配置文件如果像上面这种配置的话，报异常：Error: webpack.optimize.CommonsChunkPlugin has //been removed, please use config.optimization.splitChunks instead.
//新版本配置在entry同目录配置为
optimization: {
    //抽取公共的dm
    splitChunks: {
      cacheGroups: {
        commons: {
          name: "commons",
          chunks: "initial",
          minChunks: 2
        }
      }
    }
  }
```

配置介绍

```
// optimization: {
  //   splitChunks: {
  //     chunks: "initial",         // 必须三选一： "initial" | "all"(默认就是all) | "async"
  //     minSize: 0,                // 最小尺寸，默认0
  //     minChunks: 1,              // 最小 chunk ，默认1
  //     maxAsyncRequests: 1,       // 最大异步请求数， 默认1
  //     maxInitialRequests: 1,    // 最大初始化请求书，默认1
  //     name: () => {},              // 名称，此选项课接收 function
  //     cacheGroups: {                 // 这里开始设置缓存的 chunks
  //       priority: "0",                // 缓存组优先级 false | object |
  //       vendor: {                   // key 为entry中定义的 入口名称
  //         chunks: "initial",        // 必须三选一： "initial" | "all" | "async"(默
  认就是异步)
  //         test: /react|lodash/,     // 正则规则验证，如果符合就提取 chunk
  //         name: "vendor",           // 要缓存的 分隔出来的 chunk 名称
  //         minSize: 0,
  //         minChunks: 1,
  //         enforce: true,
  //         maxAsyncRequests: 1,       // 最大异步请求数， 默认1
  //         maxInitialRequests: 1,    // 最大初始化请求书，默认1
  //         reuseExistingChunk: true   // 可设置是否重用该chunk（查看源码没有发现默认值）
  //       }
  //     }
  //   }
  // }
```

### 使用webpack-dev-server

`webpack-dev-server`为您提供了一个简单的Web服务器和使用实时重新加载的能力。

#### 安装webpack-dev-server

```
yarn global add webpack-dev-server --dev
//安装成功之后运行命令
webpack-dev-server//运行之后在浏览器打开localhost:8080查看
```

打开之后发现图片和字体文件找不到，在`output`中添加`pubicPath`属性，这个属性表示将index.html引入的css和资源文件都加上`dist`这层目录，相当于绝对路径引入，打包之后访问localhost:8080/dist/index.html,现在就可以直接访问该网页了，然后修改样式或者内容，网页会自动重新加载。

**webpack.config.js**

```
output: {
    path: path.resolve(__dirname, "dist"),
    publicPath: "/dist/",
    filename: "js/index.js"
  }
  ...
  devServer: {
      port:8086,
      contentBase: './dist'//这个跟publicPath属性是一样的作用
  }
```

此教程本人demo 地址(develop分支代码)：[https://github.com/coffeelife/WebpackStudy/tree/develop]
