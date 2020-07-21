# JavaScript模块机制


## JS模块化

自我使用JS这门语言以来，JS在整个发展历史中不断的变迁和优化，随着使用者的增多和浏览器的支持规范的发展，JS的发展大致我划分了六个阶段。
表单校验 --> 工具类库 --> 组件库 --> 前端框架 --> 前端应用 --> 前端微服务

![avatar](/img/从零开始的NodeJS-模块化.png)

前端模块化的发展是把 **函数** 作为第一步的：模块的本质就是实现特定功能的一组方法，在JavaScript中，函数是创建局部作用域的唯一方式，因此采用了函数作为JS模块化的第一步。

```javascript
function cut(){
    //具体实现
}
```
`Cut` 方法可以作为模块被直接调用，这样的好处是实现了代码的复用，缺点是很容易造成命名冲突，并且污染了全局变量。当页面抽离出的函数模块较多时，看不出它们之间的组织关系。

把 **函数挂载到对象** 上是前端模块化迈出的第二步。把功能函数挂载到一个对象上，有需要的时候，直接通过调用对象属性的方式进行函数调用。
*这里隐隐有股Java类的感觉，学过Java的可能会感觉熟悉，本质上就是namespace模式的使用。C++的应该很了解。*
```javascript
const FileUnit= {
  path:undefined,
  read: function(){},
  write: function(){}
}
FileUnit.read(path);
```
`FileUnit`对象挂载了模块上的成员函数和变量。在进行使用的时候，就是调用这个对象的属性。这样的写法解决了第一步的问题，但是，由此衍生出了新的问题，对象会暴露出所有的成员变量和函数，并且挂载的函数和变量有可能被改写。

**立即执行函数(IIFE)** 是为了解决暴露对象而迈出的第三步。

```javascript
//什么是立即执行函数？-->就是声明完成立刻执行的函数
function x(){} //声明函数x
x();//调用函数x

//IIFE 声明完成立刻调用 (fn(){})()
const module= (function(){
    let _path = "./xxx.json";
    let cut = function(){
      console.log(_private)
      }
    return {
        cut: cut
    }
})()
module.cut();
module._path; // undefined
```

**立即执行函数(IIFE)** 允许在函数内部使用局部变量，而不会意外覆盖同名全局变量，但仍然能够访问到全局变量，而在模块外部无法修改未暴露的私有变量和函数。但缺点也很明显，很多时候，我们的模块都是架设在别的模块的依赖上进行封装，那当这个module模块依赖另一个模块怎么办?在进行探索之后，前端模块化迈出了第四步。

**引入依赖** 

```javascript
const module= (function(window,$){
    let cut = function(){
      console.log(window,$)
      }
    return {
        cut: cut
    }
})(window,jQuery)
```
将要引入的依赖作为匿名函数的参数进行传入，由此就可以在函数内部访问到依赖的下级模块进行逻辑的封装，由此开始，前端模块化正式开始了漫漫长路。也由此，前端模块化题提出了几个不得不解决的问题：

- 如何安全的包装一个模块的代码？（不污染模块外的任何代码）
- 如何唯一标识一个模块？
- 如何优雅的把模块的API暴漏出去？（不能增加全局变量）
- 如何方便的使用所依赖的模块？

前端的工程师对于上述问题给出的所有解决方案都有一个共同点：使用单个全局变量来把所有的代码包含在一个函数内，由此来创建私有的命名空间和闭包作用域。由此，前端模块规范诞生了。


## JS的模块化之路
- CommonJS 规范
- AMD
- CMD
- ES6模块

### CommonJS 规范

> CommonJS 规范为Javascript制定了一个美好的愿景 ——— Javascript能够在任何地方运行。

#### 出发点

在Web的发展过程中，前端的规范化在稳步推行，但后端的JS规范却远远落后，对后端的JS自身而言，还有很多缺陷
- 没有模块系统
- 标准版较少
- 缺乏包管理系统
- 没有标准化的接口

CommonJS 规范的提出，主要是为了弥补后端JS没有标准的缺陷，希望JS具有开发类似Java或者Python的大型应用的基础能力。

#### 模块规范

CommonJS 对模块的定义分为三个部分
- 模块引用
- 模块定义
- 模块标识

```javascript
//模块引用
const math=require('math')
```
`require`方法接收一个 **模块标识** ，以此引入一个模块的API到上下文环境中。

```javascript
//模块定义
exports.add =function(a,b){
  return a+b
}
//module.exports = value
//exports.xxx = value
```
 CommonJS规范规定，一个文件就是一个模块。每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。加载某个模块，其实是加载该模块的module.exports属性。

 ```javascript
//模块标识
模块标识就是传递给require()方法的参数。参数必须是符合小驼峰命名的字符串，或者以./ .. 开头的相对路径或绝对路径，可以省略js后缀。
```

CommonJS模块定义简单，接口简洁，**成功的将类聚的方法和变量等限定在私有作用域的中，同时支持导入和导出功能以顺畅的连接上下游依赖。**

#### 模块的加载机制

在服务器端，CommonJS模块的加载是同步的，并且模块加载的顺序，按照其在代码中出现的顺序加载，模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。和其他模块加载机制有重大不同的是CommonJS模块的加载输入的是被输出的值的拷贝。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。这点与ES6模块化有重大差异。

```javascript
// lib.js
let counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};

// main.js
const lib = require('./lib');
console.log(lib.counter);  // 3
lib.incCounter();
console.log(lib.counter); // 3
```
counter输出以后，lib.js模块内部的变化就影响不到counter了。这是因为counter是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。

### AMD

CommonJS模块规范的推出使得服务端Js开始模块化，但是CommonJS规范对浏览器端并不适用，CommonJS模块的加载是同步加载，并且按照其在代码中出现的顺序加载，浏览器端的模块都存放在服务器上，加载模块的等待时间取决于网速快慢，如果网速较差，会导致浏览器处于"假死"状态。因此，浏览器端的模块，不能采用"同步加载"（synchronous），只能采用"异步加载"（asynchronous）。这就是AMD规范诞生的背景。()

> AMD 即Asynchronous Module Definition，中文名是异步模块定义的意思。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。


#### 模块定义和使用

AMD一开始是CommonJS规范中的一个草案，全称是Asynchronous Module Definition，即异步模块加载机制。后来由该草案的作者以RequireJS实现了AMD规范，所以一般说AMD也是指RequireJS。AMD规范的实现源于想要一种比当今“编写一堆必须手动排序的具有隐式依赖项的脚本标签”更好的模块依赖解决方案而产生，AMD模块希望可以在浏览器中直接使用，并且具有良好的调试性且不依赖与特定化的服务器。AMD规范设计源于Dojo使用XHR + eval的真实经验。

RequireJS的基本思想是，通过define方法，将代码定义为模块；通过require方法，实现代码的模块加载。

```javascript
//定义没有依赖的模块
define(id?, dependencies?, factory);
//id ：可选参数，它指的是模块的名字。
//dependencies：可选参数，定义中模块所依赖模块的数组。
//factory：模块初始化要执行的函数或对象
//
define("alpha", ["require", "exports", "module"], function (require, exports, module) {  
      //三种暴露API的方式
      // exports.xxx=xxx
      // module.exports=xxx         
      // return xxx;               
        
});

//模块引入
require([module], callback);
//module：一个数组，里面的成员就是要加载的模块.
//callback：模块加载成功之后的回调函数。
require(["a","b","c"],function(a,b,c){
  //Code 
});
```
从总体上说，AMD规范修正了很多CMD规范的细节问题，并在模块化的路子上进行了并行的迈进。这里给出一个我觉得讲的不错的AMD规范的文章：[Why AMD ?](https://requirejs.org/docs/whyamd.html#amd) 有兴趣的可以读一读详细的了解AMD规范做了哪些改进。

### CMD

> CMD 即Common Module Definition, CMD是sea.js的作者在推广sea.js时提出的一种规范.SeaJS与RequireJS并称，SeaJS作者为阿里的玉伯。CMD规范专门用于浏览器端，模块的加载是异步的，模块使用时才会加载执行。CMD规范整合了CommonJS和AMD规范的特点。在 Sea.js 中，所有 JavaScript 模块都遵循 CMD模块定义规范。

在CMD规范中，一个模块就是一个文件。代码的书写格式如下：
```javascript
define(function(require, exports, module) {
    // 模块代码
    // 使用require获取依赖模块的接口
    // 使用exports或者module或者return来暴露该模块的对外接口
})
```

CMD规范采用全局的`define`函数定义模块, 无需罗列依赖数组，在`factory`函数中需传入形参`require,exports,module`.
`require`用来加载一个 js 文件模块和获取指定模块的接口对象`module.exports`。

`Sea.js`加载依赖的方式分为两个时期：

- 加载期：即在执行一个模块之前，将其直接或间接依赖的模块从服务器端同步到浏览器端；
- 执行期：在确认该模块直接或间接依赖的模块都加载完毕之后，执行该模块。

那作为异步加载模块的两种规范，AMD和CMD各有千秋。

1. AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块，CMD推崇就近依赖，只有在用到某个模块的时候再去require。
2. AMD和CMD最大的区别是对依赖模块的执行时机处理不同，同样都是异步加载模块，AMD在加载模块完成后就会执行改模块，所有模块都加载执行完后会进入require的回调函数，执行主逻辑。CMD加载完某个依赖模块后并不执行，只是下载而已，在所有依赖模块加载完成后进入主逻辑，遇到require语句的时候才执行对应的模块，这样模块的执行顺序和书写顺序是完全一致的。
因此AMD模块的使用者用户体验好，因为没有延迟，依赖模块提前执行了，CMD性能好，只有用户需要的时候才执行对应的模块。

### ES6

上述提到的模块规范的都不属于JS原生支持的, 在ECMAScript 6 (ES6)中，引入了模块功能, ES6 的模块功能汲取了CommonJS 和 AMD 的优点，拥有简洁的语法并支持异步加载，并且还有其他诸多更好的支持。

ES6 模块的设计思想就是：一个 JS 文件就代表一个 JS 模块。在模块中你可以使用 import 和 export 关键字来导入或导出模块中的东西。

ES6 模块主要具备以下几个基本特点：

- 自动开启严格模式，即使你没有写 use strict
- 每个模块都有自己的上下文，每一个模块内声明的变量都是局部变量，不会污染全局作用域
- 模块中可以导入和导出各种类型的变量，如函数，对象，字符串，数字，布尔值，类等
- 每一个模块只加载一次，每一个 JS 只执行一次， 如果下次再去加载同目录下同文件，直接从内存中读取。

```javascript
/** 定义模块 math.js **/
let basicNum = 0;
let add = function (a, b) {
    return a + b;
};
export { basicNum, add };
/** 引用模块 **/
import { basicNum, add } from './math';
function test(item) {
    item.textContent = add(99 + basicNum);
}
```

上述的例子采用了`math.js`定义了模块，通过`export`导出了一个挂在了`basicNum, add`两个函数的对象。在进行引用的时候，通过对对象进行析构得到导出的对应函数。这种加载方式，使得在采用`import`命令进行导入的时候，用户需要知道所要加载的变量名或函数名，否则无法加载出对应的函数。为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到`export default`命令，为模块指定默认输出。

```javascript
// export-default.js
export default function () {
  console.log('foo');
}
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```
这种导入方式使得用户可以为匿名函数指定任意名字进行使用。

#### ES6 模块与 CommonJS 模块的差异

① CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。

- commonJS模块一旦输出一个值，模块内部的变化就影响不到这个值。
- ES6模块如果使用import从一个模块加载变量，那些变量不会被缓存，而是成为一个指向被加载模块的引用，原始值变了，import加载的值也会跟着变。需要开发者自己保证，真正取值的时候能够取到值。


② CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

-运行时加载：commonJS 模块就是对象；即在输入时是先加载整个模块，生成一个对象，然后再从这个对象上读取方法，这种加载称为“运行时加载”。commonJS脚本代码在require的时候，就会全部执行。一旦出现某个模板被“循环加载”，就只能输出已经执行的部分，还未执行的部分不会输出。
- 编译时加载：ES6 模块不是对象，而是通过export命令显式指定输出的代码，import时指定加载某个输出值，而不是加载整个模块，这种加载称为“编译时加载”。

③ ES6 模块的运行机制与 CommonJS 不一样。ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

## 总结

本片简述了JS的一路发展以来，模块化的历程，从最开始模块化探索，到横空出世的CommonJS规范，从草案开始的AMD规范到Sea.js推广的CMD规范，最后官方的ES6模块化为模块化立下了一个阶段的里程碑。下篇将会回到NodejS谈谈Node中模块化和核心模块。
因为笔者水平有限，如果有错误或者遗漏，欢迎留言进行建议。


