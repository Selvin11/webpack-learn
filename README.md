## Webpack4.0


## 概念

现代Javascript应用程序的静态模块打包器(module bundler)，当webpack处理应用程序时，会递归的构建一个依赖关系图，包含应用程序需要的每一个模块，然后将所有的这些模块打包成一个或多个bundle。



1. [webpack核心概念](#1)
2. [配置对象中的resolve](#resolve)
3. [webpack构建的代码类型](#codeType)
4. [模块热替换](#hmr)
5. [上下文模块-管理依赖](#require)

---
<a name="1">1. webpack核心概念</a>

将应用程序（各类资源文件）转化为模块的依赖图表，根据依赖关系进行自动化打包。

* `entry`：应用程序的入口，webpack的起点
* `output`：webpack将打包后的文件放置于何处
* `loader`：webpack需要对应用程序中的各类资源文件，`html/css/js/png`等进行相应的转化成模块，然后添加至依赖图表中，因此每种文件需要对应的loader，loader的主要使命是`test`（识别文件）以及`use`（使用对应的loader）。
  
  官方介绍：`loader`用于对模块的源代码进行转换。`loader`可以使你在`import`或加载模块时预处理文件。因此，`loader`类似于其他构建工具中的任务(task)，并提供了处理前端构建步骤的强大方法。

* `plugins`：loader是对应每一个文件进行转换，而plugins常用于webpack打包模块的`compilation`和`chunk`生命周期执行操作和自定义功能

  插件使用：想要使用一个插件，你只需要`require()`它，然后把它添加到`plugins`数组中。多数插件可以通过`选项(option)`自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用`new`来创建它的一个实例。

  插件调用原理：webpack中的`compile`对象会依次调用配置对象中的`plugins`的`apply属性`，因此每个插件对象的原型都有一个apply属性，传入的就是compile对象。

  ```javascript
  // 比如ConsoleLogOnBuildWebpackPlugin插件
  const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

  class ConsoleLogOnBuildWebpackPlugin {
    apply(compiler) {
      // 通过tapable提供的钩子，来执行插件
      compiler.hooks.run.tap(pluginName, compilation => {
          console.log("webpack 构建过程开始！");
      });
    }
  }
  ```



---
<a name="resolve">2. 配置对象中的resolve</a>

* 作用：模块解析，通过配置对象传递相关参数给webpack专用模块(enhanced-resolve)，如模块将在 resolve.modules 中指定的所有目录内搜索。 你可以替换初始模块路径，此替换路径通过使用 resolve.alias 配置选项来创建一个别名，可以在一定程度上对webpack进行优化。

---
<a name="codeType">3. webpack构建的代码类型</a>

在使用 webpack 构建的典型应用程序或站点中，有三种主要的代码类型：

* 你或你的团队编写的源码。
* 你的源码会依赖的任何第三方的 library 或 "vendor" 代码。
* webpack 的 runtime 和 manifest，管理所有模块的交互。
  runtime 包含：在模块交互时，连接模块所需的加载和解析逻辑。包括浏览器中的已加载模块的连接，以及懒加载模块的执行逻辑。
  manifest：当编译器(compiler)开始执行、解析和映射应用程序时，它会保留所有模块的详细要点。这个数据集合称为 "Manifest"，当完成打包并发送到浏览器时，会在运行时通过 Manifest 来解析和加载模块。无论你选择哪种模块语法，那些 import 或 require 语句现在都已经转换为 __webpack_require__ 方法，此方法指向模块标识符(module identifier)。通过使用 manifest 中的数据，runtime 将能够查询模块标识符，检索出背后对应的模块。

---
<a name="hmr">4. 模块热替换</a>
模块热替换(HMR - Hot Module Replacement)功能会在应用程序运行过程中替换、添加或删除模块，而无需重新加载整个页面。主要是通过以下几种方式，来显著加快开发速度：

* 保留在完全重新加载页面时丢失的应用程序状态。
* 只更新变更内容，以节省宝贵的开发时间。
* 调整样式更加快速 - 几乎相当于在浏览器调试器中更改样式。

---
<a name="require">5. 上下文模块-管理依赖</a>

* 如果你的请求含有表达式，会创建一个上下文(语境)，这样在编译的时候，并不知道精确匹配的模块。
  ```javascript
  require("./template/" + name + ".ejs");
  ```
* 你还可以使用 require.context() 方法来创建自己的上下文（模块）。 你可以给这个方法传3个参数：要搜索的文件夹目录，是否还应该搜索它的子目录，一个匹配文件的正则表达式。
  ```javascript
  require.context("./test", false, /\.test\.js$/);
  // （你创建了）一个test文件夹下面（不包含子目录），能被require请求到，所有文件名以 `.test.js` 结尾的文件形成的上下文（模块）。
  require.context("../", true, /\.stories\.js$/);
  // （你创建了）一个父级文件夹下面（包含子目录），所有文件名以 `.stories.js` 结尾的文件形成的上下文（模块）。
  ```

* 一个上下文模块导出一个（require）方法，这个方法可以接收一个参数：请求的对象。
  导出的方法有3个属性： resolve, keys, id。
  * resolve 是一个函数，它返回所请求的对象被解析后得到的模块id。
  * keys 也是一个函数，它返回一个数组，由所有可能被上下文模块处理的请求的对象（译者注：参考下面第二段代码中的key）组成。

  比如，在你想引入一个文件夹下面的所有文件，或者引入能匹配正则表达式的文件，你可以这样：

  ```javascript
  function importAll (r) {
    r.keys().forEach(r);
  }
  importAll(require.context('../components/', true, /\.js$/));
  
  // 或者
  var cache = {};
  function importAll (r) {
    r.keys().forEach(key => cache[key] = r(key));
  }
  importAll(require.context('../components/', true, /\.js$/));
  // 在构建时，所有被require的模块都会被存到（上面代码中的）cache里面。
  ```



## 问题及使用发现

1. rules中的loader执行顺序，数组从右往左一次执行

2. rules中的oneOf，匹配该数组中所有项，只执行匹配到的第一个满足文件后缀条件的，如果都没有匹配到，执行最后一项