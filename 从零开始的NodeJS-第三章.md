# Node的模块构成和实现

## Node的模块实现

在Node中，模块被分为两大类
- 核心模块(Node自身附带的模块，例如fs、Http、path等模块)
- 文件模块(用户自身编写的模块，业务模块或者功能模块)

**Node对于相同模块的二次加载都采用缓存优先的方式且核心模块优先于文件模块的缓存检查(核心模块一等公民)**


Node加载模块需要经历三个步骤
- 路径分析
- 文件定位
- 编译执行

### 路径分析

1. 模块标识符分析，Node采用`require()`函数接受一个标识符参数进行模块的查找。模块标识符主要分为三种类型
   - 核心模块 `const fs=require('fs')`
   - 路径形式的文件模块 `const file=require('./从零开始的NodeJS.xmind')`
    - 相对路径文件模块
    - 绝对路径文件模块 
   - 自定义模块 *从当前目录开始链式查找node_modules直到根目录*

```javascript
//module_path.js 
console.log(module.paths);
// 文件树
│  C:
│  ├─Examples
│  │  └─ModulePath
│  │          module_path.js       

// 结果集
[
  'C:\\Examples\\ModulePath\\node_modules',
  'C:\\Examples\\node_modules',
  'C:\\node_modules'
]
```
#### 目录分析(包分析)

`require()`在分析标识符的过程中，如果标识符本身不包含拓展名，则会进行`.js .json .node`的尝试性解析。如果`require()`函数在分析文件拓展名后，没有发现指定的文件，而是获取到一整个目录，此时Node会将目录当作一个包处理。Node会先读取目录下的`package.json`,然后通过JSON.parse()解析出包对象，然后取出main属性指定的文件名进行解析，如果`package.json`或main属性指定的文件不存在，则取index作为默认的文件名进行读取。


### 模块编译

在Node中，每个文件模块都是一个对象，它的定义如下：
```javascript
function Module(id,parent){
  this.id=id
  this.exports={}
  this.parent=parent
  if(parent&&parent.children){
    parent.children.push(this)
  }
  this.filename=null
  this.loaded=false
  this.children=[]
}
```

编译和执行是引入文件模块的最后一个。Node会新建一个模块对象，然后根据路径载入并编译。
  - `.js文件` 通过fs模块同步读取文件后编译
  - `.node文件` 采用`C/C++`编写的扩展文件，通过dlopen()方法加载最后编译生成的文件
  - `.json` 通过fs模块同步读取的文件，用JSON.parse()解析返回结果
  - `其余拓展名文件` 通过`.js`载入

#### JavaScript模块的编译



