## 前言

提起一个成熟的 vue 项目自然就离不开 webpack 之流的打包工具。众所周知，webpack 最终把你的所有依赖模块都打包成了一个 js 文件，那么本篇就来探索一下它到底做了什么。

示例项目由以下命令检出，或者可以翻[我的 github](https://github.com/everlose/learningInVue)，示例所使用的 webpack3 的版本为 3.9.1。

```bash
vue init webpack-simple learning-in-vue
```

## 一个最简单的例子

修改入口文件 `src/main.js`

```javascript
export default 'This is src/main.js'
```

并且我找到 `webpack.config.js` 注释掉了 `UglifyJsPlugin`

```javascript
// new webpack.optimize.UglifyJsPlugin({
//     sourceMap: true,
//     compress: {
//         warnings: false
//     }
// }),
```

最后在 `npm run build` 后产生了大概70行的代码。且看 `dist/build.js`，我写上了英文注释的翻译

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, {
/******/ 				configurable: false,
/******/ 				enumerable: true,
/******/ 				get: getter
/******/ 			});
/******/ 		}
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "/dist/";
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony default export */ __webpack_exports__["default"] = ('This is src/main.js');

/***/ })
/******/ ]);
//# sourceMappingURL=build.js.map
```

这么多星号看着好唬人，没想到区区一行 `console.log` 都能被编译成 70 多行的代码！来简化一下，这个结构无非就是立即调用的函数表达式罢了，并且把所有引用模块包装成一个数组作为参数传入，就如下面代码所展示的：

```javascript
(function(modules) {
    //...
})([
    (function(module, exports) {
        console.log('This is src/main.js');
    })
]);
```

而这个立即调用的函数表达式，前60多行的代码就是 webpack require 的核心。我大概翻译一下

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	// 缓存模块，所有被加载过的模块都会成为installedModules对象的属性，靠函数__webpack_require__做到。
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function 核心加载方法
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		// 检查模块是否已在缓存中，是则直接返回缓存中的模块不需要再次加载
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		// 创造一个新模块并放入缓存中，i是模块标识，l意为是否加载此模块完毕，exports是此模块执行后的输出对象
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		// 传入参数并执行模块函数
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded 标为true代表模块执行完成。
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module 返回此模块输出的对象
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	// webpack 私有变量，保存传入的modules，即所有的模块组成的数组
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	// 保存缓存中的模块数组
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	// 为 es6 exports 定义 getter
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 	    // 如果 exports 输出的对象本身不包含 name 属性时，定义一个。
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, {
/******/ 				configurable: false,
/******/ 				enumerable: true,
/******/ 				get: getter
/******/ 			});
/******/ 		}
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	// 解决 ES module 和 Common js module 的冲突，ES 则返回 module['default']
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	// 工具方法，判断是否object有property属性。
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	// 大概和 webpack.config.js 的 output 有关吧，webpack 的公共路径
/******/ 	__webpack_require__.p = "/dist/";
/******/
/******/ 	// Load entry module and return exports 执行第一个依赖模块并且返回它输出。
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony default export */ __webpack_exports__["default"] = ('This is src/main.js');

/***/ })
/******/ ]);
//# sourceMappingURL=build.js.map
```

注意关键的一步，在下面完成了模块注入最为关键的一步。

```javascript
// moduleId第一次执行时为0，modules[0]既是外部第一个依赖
// 执行时 this 作用域指向 module.exports 的那个对象
// 并传入参数 module, module.exports, __webpack_require__
modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

//...

// 具体模块执行收到参数 module, module.exports, __webpack_require__
// 并且修改了 __webpack_exports__ 给他加入了 default 属性对象。
// __esModule标记本模块是es6模块，__webpack_require__.n 将会处理它输出 module['default']
(function(module, __webpack_exports__, __webpack_require__) {
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony default export */ __webpack_exports__["default"] = ('This is src/main.js');
})
```

## 其他引入方式

就如 [webpack 3](https://doc.webpack-china.org/concepts/modules#-webpack-) 官网所说

> 对比 Node.js 模块，webpack 模块能够以各种方式表达它们的依赖关系，几个例子如下：

* ES2015 import 语句
* CommonJS require() 语句
* AMD define 和 require 语句
* css/sass/less 文件中的 @import 语句。
* 样式(url(...))或 HTML 文件中的图片链接(image url)

webpack 能处理的模块类型比较多，上面的例子里我们只演示 ES2015 import 语句。

### CommonJs require()

src/main.js 写成

```javascript
module.exports = 'This is src/main.js';
```

那么 dist/build.js 将会是

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// ...
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, exports) {

// 具体模块执行收到参数 module, module.exports（空对象）, __webpack_require__
// 这里正好修改 module.exports。
module.exports = 'This is src/main.js';

/***/ })
/******/ ]);
//# sourceMappingURL=build.js.map
```

### Amd 模块

src/main.js 写成

```javascript
define([], function () {
    return {
        amd: function () {
            console.log('AMD');
        }
    };
});
```

那么 dist/build.js 将会是

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// ...
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, exports, __webpack_require__) {

var __WEBPACK_AMD_DEFINE_ARRAY__, __WEBPACK_AMD_DEFINE_RESULT__;!(__WEBPACK_AMD_DEFINE_ARRAY__ = [], __WEBPACK_AMD_DEFINE_RESULT__ = (function () {
    return {
        amd: function amd() {
            console.log('AMD');
        }
    };
}).apply(exports, __WEBPACK_AMD_DEFINE_ARRAY__),
				__WEBPACK_AMD_DEFINE_RESULT__ !== undefined && (module.exports = __WEBPACK_AMD_DEFINE_RESULT__));

/***/ })
/******/ ]);
//# sourceMappingURL=build.js.map
```

生成了好啰嗦的的一段，简言之还是跳不脱 `module.exports = __WEBPACK_AMD_DEFINE_RESULT__` 这句话


### image

webpack 竟然能把这些静态资源作为模块，但其实这是通过loader实现的。 image 则可配置 url-loader 设置，被编译成base 64编码的串。

在 `webpack.config.js` 中配置 url-loader 并 舍弃 file-loader，配置 url-loader 若在 1Mb 大小内的图片则直接编译为base 64编码。

```javascript
//...

// {
//     test: /\.(png|jpg|gif|svg)$/,
//     loader: 'file-loader',
//     options: {
//         name: '[name].[ext]?[hash]'
//     }
// },
{
    test: /\.(png|jpg|gif|svg)$/,
    loader: 'url-loader',
    options: {
        limit: 10000,
        name: '[name].[ext]?[hash]'
    }
},
//...
```

修改 `src/main.js`，使其引入图片

```javascript
import image from './assets/images/logo.png'
export default image
```

编译后的 `dist/build.js` 则

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// ...
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
// 0号模块 从 1号模块导出中获得了图片，__webpack_require__ 实际上是所有的以来模块数组
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__assets_images_logo_png__ = __webpack_require__(1);
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__assets_images_logo_png___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0__assets_images_logo_png__);


/* harmony default export */ __webpack_exports__["default"] = (__WEBPACK_IMPORTED_MODULE_0__assets_images_logo_png___default.a);

/***/ }),
/* 1 */
/***/ (function(module, exports) {

// 省略真正的base 64图片编码，太长了
module.exports = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgA..........."

/***/ })
/******/ ]);
//# sourceMappingURL=build.js.map
```

### css

css 在本项目中则通过 vue-style-loader 与 css-loader 转化为js代码，调用时就是被插入。

在 `src/main.js` 下写入

```javascript
import style from './assets/styles/reset.css'
export default style;
```

编译后的 `dist/build.js` 将会多出一大堆的函数，首先0号依赖还是熟悉的面孔

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// ...
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

// 0号模块依旧是熟悉的代码，依赖于 1号模块输出的对象。
"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__assets_styles_reset_css__ = __webpack_require__(1);
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__assets_styles_reset_css___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0__assets_styles_reset_css__);

/* harmony default export */ __webpack_exports__["default"] = (__WEBPACK_IMPORTED_MODULE_0__assets_styles_reset_css___default.a);

/***/ }),
/* 1 */
/***/ (function(module, exports, __webpack_require__) {

// style-loader: Adds some css to the DOM by adding a <style> tag

// load the styles 调用2号模块获取css具体内容
var content = __webpack_require__(2);
if(typeof content === 'string') content = [[module.i, content, '']];
if(content.locals) module.exports = content.locals;
// add the styles to the DOM 调用4号模块去具体插入css内容
var update = __webpack_require__(4)("0b88176b", content, true);

/***/ }),
/* 2 */
/***/ (function(module, exports, __webpack_require__) {

exports = module.exports = __webpack_require__(3)(undefined);
// imports


// module 这里就是我们写的 reset.css 的具体内容了，
exports.push([module.i, "*{margin:0;padding:0}", ""]);

// exports


/***/ }),
/* 3 */
/***/ (function(module, exports) {

// css base code, injected by the css-loader
// 3号模块太长，不贴，调试后得知貌似只是暴露了两个方法出来
// ......

/***/ }),
/* 4 */
/***/ (function(module, exports, __webpack_require__) {

// ...太长只贴了关键步骤，总之关键的函数就是这些
var hasDocument = typeof document !== 'undefined'
var head = hasDocument && (document.head || document.getElementsByTagName('head')[0])

// ...
module.exports = function (parentId, list, _isProduction) {

    // ...
    var styles = listToStyles(parentId, list)
    addStylesToDom(styles)

    return function update (newList) {
        // ...
    }
}

function addStylesToDom (styles /* Array<StyleObject> */) {
    for (var i = 0; i < styles.length; i++) {
        // ...
        domStyle.parts.push(addStyle(item.parts[j]))
        // ....
    }
}

// 总之先调用了addStylesToDom，接着是addStyle，再是createStyleElement插入样式到head中。
function createStyleElement () {
  var styleElement = document.createElement('style')
  styleElement.type = 'text/css'
  head.appendChild(styleElement)
  return styleElement
}

function addStyle (obj /* StyleObjectPart */) {
    // ...
    styleElement = createStyleElement()
    // ...
}


/***/ }),
/* 5 */
// ...
/******/ ]);
//# sourceMappingURL=build.js.map
```


## 参考

[探索webpack模块以及webpack3新特性](https://juejin.im/post/59b9d2336fb9a00a636a3158)
[webpack把你的项目编译成了什么](https://segmentfault.com/a/1190000004144950)
