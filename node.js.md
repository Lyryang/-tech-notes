



## 全局对象

在浏览器里，全局对象叫 `window`；在 Node.js 里，它叫 **`global`**（ES2020 后统一别名 **`globalThis`**）。

作用：**挂在它身上的东西，在任何文件、任何地方都能直接用，不需要 `require` / `import`。**



`setTimeout`、`setInterval`、`setImmediate`、`console`、`Buffer`、`process` 全都是它的属性。所以平时随手就能用它们，本质是

```js
setTimeout(...)      // 其实等价于
global.setTimeout(...)
globalThis.setTimeout(...)
```



总图

```
global / globalThis（全局对象，随处可用）
├── 定时器：setTimeout / setInterval / setImmediate
│      └── 底层：事件循环分阶段（timers → poll → check）
├── console  ── 底层写向 → process.stdout / stderr
├── Buffer   ── 一串字节(0~255)，继承 Uint8Array，十六进制显示
├── process  ── 代表当前进程
│      ├── cwd()      运行命令的目录（会变）
│      ├── argv       命令行参数，slice(2) 取用户传的
│      ├── platform   'win32'/'darwin'/'linux'
│      ├── env        环境变量，NODE_ENV 最常用
│      ├── exit(code) 粗暴退出，0 成功
│      └── kill(pid)  发信号，默认 SIGTERM
└──【非 global】__dirname / __filename
       └── 底层：模块包装函数的参数（每个模块私有）
```





### 事件循环

Node 处理这些异步任务，靠的是一个循环，分好几个**阶段**，按固定顺序转圈：

```
   ┌───────────────────────────┐
┌─>│   timers 阶段              │  ← setTimeout / setInterval 的回调在这里跑
│  ├───────────────────────────┤
│  │   poll 阶段（等 I/O）      │  ← 读文件、网络请求的回调
│  ├───────────────────────────┤
│  │   check 阶段               │  ← setImmediate 的回调在这里跑
│  ├───────────────────────────┤
│  │   close 阶段               │
│  └───────────────────────────┘
└──────────< 转下一圈 >──────────
```





### console

不只有log

`console` 是全局对象，用来向终端输出信息。常用方法：

```js
console.log('普通输出');     // 写到「标准输出」stdout
console.error('错误输出');   // 写到「标准错误」stderr（红色）
console.warn('警告');        // 也是 stderr
console.info('信息');        // 等同 log

console.table([{ a: 1, b: 2 }, { a: 3, b: 4 }]); // 表格形式，很直观
console.dir({ x: { y: 1 } }, { depth: null });    // 展开深层对象

console.time('计时');        // 计时开始
// ...一些代码...
console.timeEnd('计时');     // 打印耗时，如「计时: 3.2ms」
```

输出结果：

```
普通输出
错误输出
警告
信息
┌─────────┬───┬───┐
│ (index) │ a │ b │
├─────────┼───┼───┤
│ 0       │ 1 │ 2 │
│ 1       │ 3 │ 4 │
└─────────┴───┴───┘
{ x: { y: 1 } }
计时: 0.005ms
```





### `__dirname`/`__filename` 不是 global 属性

```js
console.log(__dirname);              // 当前文件所在【目录】的绝对路径
console.log(__filename);             // 当前文件本身的绝对路径（含文件名）
console.log(typeof global.__dirname);// 'undefined' ← 证明它真的不在 global 上！
```

能直接用，但不在global上

原因：Node 在运行你的每个 `.js` 文件前，会偷偷把代码包进一个函数里，这叫**模块包装函数（Module Wrapper）**

```js
(function (exports, require, module, __filename, __dirname) {
   // ⬆️ 写的所有代码，其实都被塞在这里运行
});
```

`__filename`、`__dirname`（还有 `require`、`module`、`exports`）本质是这个函数的**参数**





### `Buffer` — 「字节」

普通字符串、数字是给人看的；但网络传输、读写文件时，计算机底层流动的是**二进制原始数据**。`Buffer` 就是 Node 用来装这种「一串字节」的容器。



**字节（byte）**是计算机存储的基本单位，1 字节 = 8 位（bit），能表示的数值范围正好是 **0 ~ 255**。



**Buffer 是类型化数组**，继承自 `Uint8Array`（`U`nsigned `Int`eger `8`-bit = 无符号 8 位整数）。所以 Buffer 本质就是「一个每个元素都是 0~255 的数组」。



**为什么输出用十六进制**：1 字节 = 0~255，用十六进制表示正好是**两位**（`00`~`FF`），f 表示 15，紧凑又整齐。所以打印 Buffer 会看到 `<Buffer 48 65 ...>` 这种两位两位的形式。

```js
const buf = Buffer.from('Hello');

console.log(buf);               // <Buffer 48 65 6c 6c 6f>  ← 十六进制显示
```

 `<Buffer 48 65 6c 6c 6f>` 就是这 5 个字母的 ASCII 码的十六进制表示，**一个字母 = 一个字节 = 两位十六进制**。

| 字母 | ASCII（十进制） | ASCII（十六进制） |
| :--- | :-------------- | :---------------- |
| H    | 72              | **48**            |
| e    | 101             | **65**            |
| l    | 108             | **6c**            |
| l    | 108             | **6c**            |
| o    | 111             | **6f**            |



「字节 ≠ 字符」

在 UTF-8 编码下，**一个汉字占 3 个字节**，所以「你好」= 6 字节。

```js
const buf2 = Buffer.from('你好');
console.log(buf2);        // <Buffer e4 bd a0 e5 a5 bd>
console.log(buf2.length); // 6   ← 注意！两个汉字却是 6 个字节
```



创建buffer

```js
const buf = Buffer.from('abc');        // 从字符串（默认 UTF-8）
const buf1 = Buffer.from([72, 101, 108]);// 从字节数组，得到 <Buffer 48 65 6c>

```

```js
const buf1 = Buffer.alloc(5); // 分配 5 字节，且【自动清零】
console.log(buf1); // <Buffer 00 00 00 00 00>

Buffer.allocUnsafe(5);     // 分配更快，但【不清零】，可能残留旧数据
```







### `process` — 程序与操作系统的接口

`process` 是代表「**当前这个 Node 进程**」的全局对象，通过它能拿到运行环境、命令行参数、和操作系统交互。



`process.cwd()` —— 当前工作目录

返回**敲 `node` 命令时所在的目录**的绝对路径。

```
console.log(process.cwd());
```

 **和 `__dirname` 的区别**：

- `__dirname` = **文件**待的地方（写死的，跟你在哪运行无关）
- `process.cwd()` = **运行命令时人在的地方**（会变）



`process.argv` —— 命令行参数（String[]）

它是个字符串数组，约定：

- `argv[0]`：node 可执行文件的路径
- `argv[1]`：运行的脚本文件路径
- `argv[2]` 及之后：**开发者真正传的参数**

```js
// 假设文件叫 args.js
console.log(process.argv);
```

运行 `node args.js hello 123`，输出类似：

```powershell
[
  'C:\\...\\node.exe',   // [0]
  'C:\\...\\args.js',    // [1]
  'hello',               // [2]  ← 你传的
  '123'                  // [3]  ← 你传的（注意是字符串 '123' 不是数字）
]
```

取「用户真正传的参数」的标准写法是：

```js
const args = process.argv.slice(2); // ['hello', '123']
```



`process.platform` —— 当前操作系统

```js
console.log(process.platform);
// Windows → 'win32'（注意：即使 64 位也是 win32）
// macOS   → 'darwin'
// Linux   → 'linux'
```

常用来写跨平台代码，比如根据系统选不同的路径分隔符或命令。



`process.env` —— 环境变量对象

```js
console.log(process.env.PATH);      // 读系统 PATH 变量
console.log(process.env.NODE_ENV);  // 最常用：区分开发/生产环境

process.env.MY_KEY = 'hi';          // 也能设置，但只在【当前进程】有效，进程结束就没了
```

实战里 `process.env.NODE_ENV === 'production'` 是判断「生产环境」的行业惯例，用来决定是否压缩代码、是否打印调试日志等。



`process.exit([code])` —— 强制退出进程

```js
console.log('前');
process.exit(0);       // 立即退出。code 省略时默认 0
console.log('后');     // ❌ 永远不会执行
```

退出码约定：**`0` = 成功**，**非 0 = 出错**（比如 `1`）。这个码会交给操作系统/调用方判断程序是否正常结束。

> `process.exit()` 是「**粗暴**」退出：还没跑完的异步任务（比如没写完的文件、没发完的网络请求）会被直接掐断。所以正常业务里能自然结束就别强行 exit，通常只在遇到致命错误时用非 0 码退出。



`process.kill(pid[, signal])` —— 给进程发信号

根据**进程 ID（PID）** 向某个进程发送信号，默认发 `SIGTERM`（请求终止）。

```js
console.log(process.pid);   // 先看看「当前进程」自己的 ID

// process.kill(12345);     // 给 PID 为 12345 的进程发终止信号
// process.kill(process.pid); // 效果 = 终止自己
```







## node模块化



### 函数环境

在 `.js` 文件里写的代码，Node 不会原样执行，而是先把它塞进一个函数里再执行。

```js
(function (exports, require, module, __filename, __dirname) {
   // ↑↑↑ 这一行是 Node 加的
   
   // ……你写的所有代码，原封不动放在这中间……
   
});   // ← 这一行也是 Node 加的
```



Node 加载一个模块时，做的事是：把文件内容当字符串读进来 → 在头尾拼上上面那两段 → 编译成一个函数 → 调用它，并把 5 个实参传进去。

没 `const require = ...` 却能直接用它。



直接在文件顶层打印 `arguments`

```js
console.log('【现象1】arguments =', arguments);
```

简化结果：

```
'0': {}                    ← exports    （一个空对象，等你往里塞东西）
'1': [Function: require]   ← require    （找并加载别的模块的函数）
'2': { id, path, ... }     ← module     （描述"当前模块自己"的对象）
'3': 'C:\...\01-函数环境揭秘.js'  ← __filename （当前文件的绝对路径）
'4': 'C:\...\01-Node模块化'      ← __dirname  （当前文件所在文件夹）
```

五个参数作用：

- `__filename` / `__dirname`：当前文件、当前文件夹的**绝对路径**。做路径拼接全靠它俩。
- `require`：拿来加载别的模块。
- `module`：记录"我这个模块"的所有信息。
- `exports`：一个空对象 `{}`，你往它身上加属性，就是对外导出。



完整结果：

```powershell
【现象1】arguments = [Arguments] {
  '0': {},
  '1': [Function: require] {
    resolve: [Function: resolve] { paths: [Function: paths] },
    main: {
      id: '.',
      path: 'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化',
      exports: {},
      filename: 'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化\\01-函数环境揭秘.js',
      loaded: false,
      children: [],
      paths: [Array],
      [Symbol(kIsMainSymbol)]: true,
      [Symbol(kIsCachedByESMLoader)]: false,
      [Symbol(kURL)]: undefined,
      [Symbol(kFormat)]: undefined,
      [Symbol(kIsExecuting)]: true
    },
    extensions: [Object: null prototype] {
      '.js': [Function (anonymous)],
      '.json': [Function (anonymous)],
      '.node': [Function (anonymous)]
    },
    cache: [Object: null prototype] {
      'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化\\01-函数环境揭秘.js': [Object]
    }
  },
  '2': {
    id: '.',
    path: 'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化',
    exports: {},
    filename: 'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化\\01-函数环境揭秘.js',
    loaded: false,
    children: [],
    paths: [
      'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化\\node_modules',
      'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\node_modules',
      'C:\\Users\\19425\\Desktop\\笔记\\node_modules',
      'C:\\Users\\19425\\Desktop\\node_modules',
      'C:\\Users\\19425\\node_modules',
      'C:\\Users\\node_modules',
      'C:\\node_modules'
    ],
    [Symbol(kIsMainSymbol)]: true,
    [Symbol(kIsCachedByESMLoader)]: false,
    [Symbol(kURL)]: undefined,
    [Symbol(kFormat)]: undefined,
    [Symbol(kIsExecuting)]: true
  },
  '3': 'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化\\01-函数环境揭秘.js',
  '4': 'C:\\Users\\19425\\Desktop\\笔记\\agnet讲解师\\01-Node模块化'
}
```





### `module` 对象 与 `exports` 



```js
// module 就是"描述当前模块自己"的一张身份证
console.log('module.id       =', module.id);       // 主模块是 '.'，被 require 的是绝对路径
console.log('module.filename =', module.filename); // 当前文件绝对路径
console.log('module.path     =', module.path);     // 当前文件所在目录
console.log('module.loaded   =', module.loaded);   // false：此刻还在加载中，还没加载完
console.log('module.exports  =', module.exports);  // {}：真正对外导出的容器
console.log('module.children =', module.children);  // 我 require 过谁


// 一开始，exports 只是 module.exports 的一个"别名"（指向同一个对象）
console.log('exports === module.exports ?', exports === module.exports); // true
```

输出结果

```
module.id       = .              ← 主模块（直接被 node 运行的）id 就是 '.'
module.filename = ...\02-module对象.js   ← 绝对路径
module.path     = ...\01-Node模块化       ← 所在目录
module.loaded   = false          ← 此刻"正在加载中"，还没跑完
module.exports  = {}             ← 真正对外导出的容器
module.children = []             ← 我 require 过的模块会记在这
exports === module.exports ? true
```



`exports` 只是 `module.exports` 的"别名"

Node大致是这样调用环境函数的：

```js
module.exports = {};                 // 先准备一个空容器
const exports = module.exports;      // 再让形参 exports 指向同一个对象
wrapper(exports, require, module, __filename, __dirname);
```

会在加载前让`exports`和`module.exports`指向同一个对象



两个模块 `good.js` `bad.js` 导出不同

```js
// good.js —— ✅ 整体替换 module.exports
module.exports = function(){...}

// bad.js  —— ❌ 只是让"局部变量 exports"改指向，module.exports 没变，还是 {}
exports = function(){...}
```

导入结果不同：

```js
const good = require('./lib/good');
const bad = require('./lib/bad');

console.log('good 收到:', typeof good, '->', good());        // function -> 正常调用
console.log('bad  收到:', typeof bad, '->', bad);            // object   -> 加载前进行了同步，exports变成了{} 空对象！函数丢了
```





### `require` 怎么"找到"模块

`require(x)` 第一步永远是把 `x` 解析成一个唯一的绝对文件路径。

| 写法                    | 例子                                  | 查找规则                                                     |
| ----------------------- | ------------------------------------- | ------------------------------------------------------------ |
| **绝对路径**            | `require('C:/a/b.js')`                | 直接按该路径加载                                             |
| **相对路径** `./` `../` | `require('./lib/good')`               | 以**当前文件所在目录**为基准，拼成绝对路径再加载             |
| **裸模块名**            | `require('fs')` / `require('lodash')` | ① 先看是不是**内置模块**（`fs`/`path`/`http`…）→ 命中就用<br />② 否则从**当前目录的 `node_modules`** 开始找<br>③ 找不到就**逐级向上**到父目录的 `node_modules`<br>④ 一直找到磁盘根目录 |





### 后缀补全

不写后缀时，Node 按顺序试：.js  →  .json  →  .node   （.mjs 属于 ESM 场景）

```js
require('./data')

//  不写后缀：Node 按 .js -> .json -> .node 顺序自动补
//  这里没有 data.js，只有 data.json，于是命中 json
```





### 文件入口

只给文件夹时找入口：

```js
require('./demo-dir')
```

优先级：

1. 看该文件夹 package.json 的 "main" 字段  →  用它指定的文件
2. 没有 main / 没有 package.json          →  兜底找该文件夹的 index.js





### 模块缓存

同一个模块被 `require` 多次，模块体只执行一次，之后都拿缓存。 

counter.js

```js
// 这行是"副作用"：只要 counter.js 被真正执行，就会打印一次
console.log('>>> counter.js 的模块体被执行了！（注意它到底打印几次）');

module.exports = { count: 0 };
```

入口文件：

```js
console.log('第一次 require ↓');
const a = require('./counter');
console.log('第二次 require ↓（如果上面那句 ">>>" 没再打印，就证明被缓存了）');
const b = require('./counter');

a.count = 100;                       // 只改了 a
console.log('\nb.count =', b.count); // 却发现 b 也变了 -> a 和 b 是同一个对象
console.log('a === b ?', a === b);   // true
```

打印结果：

```
第一次 require ↓
>>> counter.js 的模块体被执行了！   ← 只打印了这一次！
第二次 require ↓

b.count = 100
a === b ? true
```

- 第二次 `require('./counter')` **没有**再打印 `>>>`，证明模块体没重跑。
- 缓存存放在 `require.cache`对象中，key 就是模块的绝对路径
- 副作用：模块天然是**单例**。这既是优点（共享状态、性能好），也是坑（改了 a 影响所有人）





### require完整流程

```mermaid
flowchart TD
  A["require('x')"] --> B["解析成绝对路径<br/>（三种路径 + 后缀/入口）"]
  B --> C{"require.cache<br/>里有吗？"}
  C -- 有 --> D["直接返回缓存的<br/>module.exports"]
  C -- 没有 --> E["新建 module 对象<br/>exports = {}"]
  E --> F["先塞进缓存<br/>（应对循环依赖）"]
  F --> G["读文件 → 套上包裹函数<br/>(exports,require,module,__filename,__dirname)"]
  G --> H["执行模块体<br/>你的代码往 module.exports 上挂东西"]
  H --> I["module.loaded = true"]
  I --> J["返回 module.exports"]
```



### 自测

1. 顶层为什么能用 `arguments`？包裹函数的 5 个参数分别是谁？
2. `exports = xxx` 为什么无效，`module.exports = xxx` 为什么有效？
3. `require('fs')`、`require('./a')`、`require('lodash')` 三者查找起点有何不同？
4. `require('./a')` 不写后缀，Node 按什么顺序补？给文件夹时按什么顺序找入口？
5. 一个模块 require 两次，模块体执行几次？缓存的 key 是什么？







## Node中ES模块化

早期浏览器里所有 `<script>` 共享**同一个全局作用域**。两个文件都写 `let name`，就冲突了；想复用别人的代码，只能靠“定义顺序 + 全局变量”硬凑。



**模块化解决两件事**：

1. **独立作用域** —— 每个文件是一个封闭盒子，内部变量不会泄漏出去污染别人。
2. **显式依赖** —— 我需要谁，就明确地“导入”谁；我能给别人用的，就明确地“导出”。



Node 从诞生起用的是 **CommonJS** 方案；后来 JS 语言官方（ECMAScript）定了 **ES Module** 标准，Node 也支持了。所以今天 Node 里**两套并存**。



### CommonJS：导入导出

导出

```js
// math.js
function add(a, b) { return a + b; }
const PI = 3.14;

// 方式一：整体赋值给 module.exports（最常用、最不容易错）
module.exports = { add, PI };

// 方式二：往 exports 上挂属性
exports.add = add;
exports.PI = PI;
```



导入

```js
// main.js
const math = require('./math.js');  // 传入相对路径，拿到导出的对象
console.log(math.add(1, 2)); // 3

// 也可以直接解构
const { add, PI } = require('./math.js');
```





### CommonJS 的三大运行特性

现有模块：计数器.cjs

```js
let count = 0;

function increment() {
  count++;
}

console.log('【计数器.cjs 被执行】此刻 count =', count);

module.exports = {
  count,                  // 导出的是“值的快照”(此刻是 0)，不是变量本身
  increment,
  getCount: () => count,  // 只有通过函数才能读到最新值
};
```

另外的模块：

```js
const a = require('./计数器.cjs');
const b = require('./计数器.cjs');

// 现象 1：缓存 —— 上面 require 了两次，
// 但“【计数器.cjs 被执行】”只会打印一次。为什么？
console.log('a === b ?', a === b); // 是同一个对象吗？

// 现象 2：值拷贝 —— 调用 increment 改变模块内部的 count
a.increment();
a.increment();
console.log('直接读导出的 a.count :', a.count);      
console.log('通过函数读 getCount() :', a.getCount()); 
```

控制台结果

```
【计数器.cjs 被执行】此刻 count = 0     ← 只打印了 1 次！
a === b ? true
直接读导出的 a.count : 0                ← 没变
通过函数读 getCount() : 2               ← 变了
```



1、同步加载

`require` 是**同步**的——执行到 `require('./x')` 就地停下、立刻把那个模块从头跑完、拿到结果再往下走。好处是简单直观；代价是不能天然处理异步（这也是后面 ESM 要改进的点）。



2、缓存（require 只执行一次）

`require` 了两次，但“计数器.cjs 被执行”**只打印一次**，而且 `a === b` 为 `true`。因为 Node 把每个模块第一次的运行结果**缓存**起来，第二次直接返回缓存的同一个对象。所以模块顶层代码**只会执行一次**——这是“单例”的天然来源。



3、值拷贝（导出的是快照，不是活引用）

`increment()` 明明让内部 `count` 变成了 2，为什么 `a.count` 还是 0，而 `getCount()` 是 2？

因为导出时 `module.exports = { count, ... }`，执行整个模块`计数器.cjs`，模块执行完毕后，这里的 `count` 是0，所以导出的`count`只是把当前的模块中、`count`的值 0， 复制了一份放进对象，从此和模块内部那个 `count` 变量**再无关系**。而 `getCount` 是个函数，每次调用都去读模块内部**最新的** `count`，所以能拿到 2。





### ES Module（ESM）

在Node里开启ESM：

1. 文件后缀用 **`.mjs`** → 强制当 ESM（推荐做实验用，简单明确）。
2. 最近的 **`package.json`** 里写 ["type": "module"](vscode-file://vscode-app/c:/Application/Microsoft VS Code/1b6a188127/resources/app/out/vs/code/electron-browser/workbench/workbench.html) → 该目录下所有 `.js` 都当 ESM。
3. 反过来，["type": "module"](vscode-file://vscode-app/c:/Application/Microsoft VS Code/1b6a188127/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 项目里想临时写一个 CommonJS 文件，用 **`.cjs`** 后缀。



常用语法：

导出

```js
// 具名导出（named export）
export const PI = 3.14;
export function add(a, b) { return a + b; }

// 默认导出（default export，一个模块最多一个）
export default function () { /* ... */ }

// 重导出（做“统一出口”文件时常用）
export { add } from './math.mjs';
```

导入

```js
// 导入
import { PI, add } from './math.mjs';      // 具名导入，名字要对上
import myDefault from './math.mjs';         // 默认导入，名字随便起
import * as math from './math.mjs';         // 命名空间导入，全打包成一个对象
import defaultFn, { PI } from './math.mjs'; // 混合


// 动态导入（返回 Promise，可按需/条件加载）
const mod = await import('./math.mjs');
```





### ESM 的核心特性



#### 1、静态结构

（编译期就确定依赖）

ESM 的 `import` 在代码**真正运行之前**就被引擎分析好了——谁依赖谁、导入了哪些名字，全是编译期静态确定的。好处：能做**摇树优化**（tree-shaking，打包时自动删掉没用到的导出）、能提前发现拼错的导入名。

所以`import` / `export` 只能写在**模块顶层**，并且不允许把 `import` 写进 `if` 。





#### 2、实时绑定（live binding）

—— 和 CommonJS 最大的区别

ESM 导入的**不是值的拷贝，而是对原模块变量的“活引用”**（live binding）。原模块里 `count` 一变，你这边 import 的 `count` 立刻同步。可以理解成：CommonJS 给你“照片”，ESM 给你“实时监控画面”。

例子：

计数器.mjs，和章节：CommonJS 的三大运行特性运行的例子一样

```js
// ES Module 版“计数器”模块，和 计数器.cjs 内容一样，只是换成 export 语法。
export let count = 0;

export function increment() {
  count++;
}

console.log('【计数器.mjs 被执行】此刻 count =', count);
```

另外的模块导入：

```js
import { count, increment } from './计数器.mjs';
import * as ns from './计数器.mjs';

console.log('初始 count :', count); // 0

increment();
increment();

// 现象：直接 import 进来的 count 竟然也跟着变了！
// 这就是“实时绑定”——导入的不是值的快照，而是对原变量的“活引用”。
console.log('两次 increment 后的 count :', count);
console.log('命名空间 ns.count          :', ns.count);
```

打印结果

```js
初始 count : 0
两次 increment 后的 count : 2   ← 直接 import 进来的 count 自己变了！
命名空间 ns.count          : 2
```

同样的代码逻辑，CommonJS 里 `a.count` 是 0，ESM 里 `count` 却变成了 2。





#### 3、顶层 await

es的设计是异步的：

```
阶段 1 —— 解析（Parse）
  → 静态分析所有 import/export，构建模块依赖图
  → 此时不执行任何代码

阶段 2 —— 实例化（Instantiate）
  → 为所有导出变量分配内存（但不赋值）
  → 建立模块间的"实时绑定"关系

阶段 3 —— 求值（Evaluate）  ← 顶层 await 在这里生效
  → 真正执行模块代码
  → 如果遇到顶层 await，整个模块求值暂停
  → 依赖此模块的父模块也会等待
  → 全部完成后才继续
```



例子：

```js
//顶层 await：不用包在 async 函数里，直接在模块顶层 await。
//这是 CommonJS 做不到的（require 是同步的）。
console.log('开始等待...');
await new Promise((resolve) => setTimeout(resolve, 500));
console.log('500ms 后继续执行（顶层 await 生效）');
```





#### 4、默认严格模式 

ESM 天生 `'use strict'`，而且**没有** CommonJS 那套包装参数（`require`、[module](vscode-file://vscode-app/c:/Application/Microsoft VS Code/1b6a188127/resources/app/out/vs/code/electron-browser/workbench/workbench.html)、[__filename](vscode-file://vscode-app/c:/Application/Microsoft VS Code/1b6a188127/resources/app/out/vs/code/electron-browser/workbench/workbench.html)、`__dirname` 都没有）。要拿当前文件路径，用 **`import.meta.url`**，再按需换算：

```js
import { fileURLToPath } from 'node:url';
import { dirname } from 'node:path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```





### 自测

| 维度        | CommonJS                      | ES Module                               |
| ----------- | ----------------------------- | --------------------------------------- |
| 开启方式    | 默认 / `.cjs`                 | `.mjs` / type:"module"                  |
| 导出        | `module.exports`/ `exports.x` | `export` / `export default`             |
| 导入        | `require()`                   | `import` / `import()`                   |
| 加载时机    | **运行时**（可写在 if 里）    | **编译期静态**（顶层）+ `import()` 动态 |
| 加载方式    | 同步                          | 异步、支持**顶层 await**                |
| 导入的是    | **值拷贝**（快照）            | **实时绑定**（活引用，只读）            |
| 顶层 `this` | `module.exports`              | `undefined`                             |
| 文件路径    | `__filename` / `__dirname`    | `import.meta.url`                       |
| 严格模式    | 默认非严格                    | **默认严格**                            |







## 基本内置模块



### 内置模块调用链

```
JS 代码			 ← 只需要调用语义清晰的 API
   ↓
lib/os.js            ← Node 源码仓库里 lib/os.js 文件，纯 JS 编写，人类可读。
   ↓
internalBinding('os') ← C++ 层 src/node_os.cc
   ↓
libuv (uv_cpu_info / uv_os_homedir / uv_os_tmpdir ...)
   ↓
操作系统 syscall（Linux: sysinfo/statfs；Windows: GlobalMemoryStatusEx ...）
```

C++层：

1. 查找已注册的 C++ 模块 `"os"`
2. 返回一个 JS 对象，但其方法的实现体是 C++ 函数



libuv 是 Node.js 的**跨平台基础库**，用纯 C 编写。

核心价值是：**屏蔽操作系统差异，提供统一接口**



**Syscall**：获取真实应用数据





### 永远用 `node:` 前缀

```js
const os = require('node:os');      // ✅
const os = require('os');           // ⚠️ 能用，但不推荐
```

原因（**安全相关**）：不带前缀时会先走模块解析流程，理论上可能被 `node_modules` 里同名包影响（供应链攻击的一种手法）；带 `node:` 前缀是硬编码走内置模块，**不可被劫持**，且跳过解析步骤更快。





### os —— 运行环境

用来拿到

代码跑在别人的机器上：是 Windows 还是 Linux？几核？内存还剩多少？临时文件该往哪写？用户主目录在哪？`os` 就是回答这些问题的。



#### `os.EOL` —— 换行符常量

```js
const os = require('node:os');
console.log(JSON.stringify(os.EOL));
// Windows → "\r\n"
// Linux / macOS      → "\n"
```

使用：

```js
const fs = require('node:fs');
const lines = ['id,name', '1,tom', '2,jerry'];

fs.writeFileSync('a.csv', lines.join('\n'));     // ❌ Windows 记事本打开会挤成一行
fs.writeFileSync('b.csv', lines.join(os.EOL));   // ✅ 各平台原生工具都正常
```

反向：

```js
// ❌ 在 Linux 上跑，遇到 Windows 传来的文件就废了
text.split(os.EOL);

// ✅ 兼容三种换行（\r\n, \n, 老 Mac 的 \r）
text.split(/\r\n|\r|\n/);
```





#### `os.arch()` —— CPU 指令集架构

```js
console.log(os.arch());     // 'x64'
console.log(process.arch);  // 'x64'  ← 完全等价，process.arch 更快（编译期常量）
```



**用途**：下载对应的预编译二进制。比如 esbuild、sharp、node-sass 这类包，安装时就是靠这个拼下载地址：

```js
const pkg = `esbuild-${os.platform()}-${os.arch()}`;  // esbuild-win32-x64
```





#### `os.cpus()` —— CPU 信息

```js
console.log(os.cpus().length);   // 32 → cpu线程数
```

Node 18.14+，推荐用`os.availableParallelism()`它替 `cpus().length`，`availableParallelism`会告知真正可用的核数，比如处在虚拟机环境中



```js
console.log(os.cpus()[0]);
```

输出：

```js
{
  model: '13th Gen Intel(R) Core(TM) i9-13980HX',
  speed: 2419,            
  times: {			// 单位：毫秒
    user: 135656,       // 执行用户态程序所花时间（开发者应用代码）
    nice: 0,              // 执行低优先级用户进程的时间（Unix 概念，Windows 下恒为 0）
    sys:  192359,        // 执行内核态（系统调用、驱动等）所花时间
    idle: 27541859,       // CPU 空闲时间
    irq:  41968            // 处理硬件中断所花时间
  }
}
```

基本信息：

| 字段    | 值                    | 含义                                        |
| :------ | :-------------------- | :------------------------------------------ |
| `model` | Intel Core i9-13980HX | CPU 型号，第13代英特尔酷睿 i9               |
| `speed` | 2419                  | 当前核心运行频率 **2419 MHz（约 2.4 GHz）** |





手写CPU使用率监视器，放到js文件中直接用终端执行：`node ./文件名.js`

```js
const os = require('node:os');

function snapshot() {
  // 把所有核心的时间加总
  return os.cpus().reduce((acc, cpu) => {
    for (const k in cpu.times) acc[k] = (acc[k] || 0) + cpu.times[k];
    return acc;
  }, {});
}

function usageBetween(a, b) {
  const totalA = Object.values(a).reduce((s, n) => s + n, 0);
  const totalB = Object.values(b).reduce((s, n) => s + n, 0);
  const idleDiff  = b.idle - a.idle;
  const totalDiff = totalB - totalA;
  return (1 - idleDiff / totalDiff) * 100;   // 非空闲占比
}

let prev = snapshot();
setInterval(() => {
  const cur = snapshot();
  console.log('CPU 使用率:', usageBetween(prev, cur).toFixed(1) + '%');
  prev = cur;
}, 1000);
```





#### `os.freemem()` / `os.totalmem()` —— 内存

```js
const os = require('node:os');
const toGB = b => (b / 1024 ** 3).toFixed(2) + ' GB';
console.log('总内存:', toGB(os.totalmem()));
console.log('空闲  :', toGB(os.freemem()));
```

方法返回的**单位一律是字节（Byte）**，返回的是数字。

输出：

```
总内存: 15.63 GB
空闲  : 3.78 GB
```



知道 Node 进程自己吃了多少内存，要用：

```js
console.log(process.memoryUsage());
 
// { rss, heapTotal, heapUsed, external, arrayBuffers }
// rss      = 进程占的物理内存总量
// heapTotal = V8 堆的总大小
// heapUsed = V8 堆里实际用掉的（排查内存泄漏就看它，周期性采样观察其是否上涨）
```





#### `os.homedir()` —— 用户主目录

```js
console.log(os.homedir());   // Windows: C:\Users\19425
                             // Linux:   /home/xxx
```

**用途**：CLI 工具的全局配置存放位置。

```js
const path = require('node:path');
const configDir = path.join(os.homedir(), '.myapp');   // C:\Users\19425\.myapp
```

（这就是 `.gitconfig`、`.npmrc`、`.ssh` 都在其目录下的原因。）





#### `os.hostname()` —— 主机名

```js
console.log(os.hostname());   // 'LAPTOP-PA60SI8J'
```

底层是 `gethostname()` syscall。

**用途**：日志里标记"这条日志是哪台机器产生的"。分布式系统排障必备。





#### `os.tmpdir()` —— 临时目录

```js
console.log(os.tmpdir());   // Windows: C:\Users\19425\AppData\Local\Temp
                            // Linux:   /tmp
```

临时目录：是操作系统提供的一个**专门用于存放临时文件**的目录

使用场景：文件上传中转，先写入临时目录，处理完再移动到目标位置





### path —— 纯字符串

path 不碰硬盘

```js
const path = require('node:path');
path.extname('/根本/不存在/的/文件.png');  // '.png'  ← 照样返回
```

"`path.resolve` 不会去找文件，只是按规则拼字符串。

所有 `path` 方法都是纯函数：相同输入必定相同输出，无副作用，无 IO。



不同心痛路径分隔符不一样，Windows是`\`（反斜杠），Linux / macOS，`/`（正斜杠）

```js
path        // 跟随当前平台（当前机器上 === path.win32）
path.win32  // 强制 Windows 规则
path.posix  // 强制 POSIX 规则

path.win32.join('C:\\a', 'b');   // 'C:\a\b'      ← 认识反斜杠
path.posix.join('C:\\a', 'b');   // 'C:\a/b'      ← 反斜杠只是普通字符！
```

**实践准则**：

- 处理**文件系统路径** → 用 `path`（跟随平台）。
- 处理 **URL 路径 / Git 路径 / 压缩包内路径 / 数据库里存的路径** → 用 `path.posix`（永远 `/`，跨平台一致）。
- 写**单元测试**时用 `path.posix` 或 `path.win32`





####  `path.sep` 与 `path.delimiter`

sep 分隔"目录"，delimiter 分隔"路径列表"

```js
path.sep;         // Windows: '\\'   POSIX: '/'   ← 路径内部的分隔符
path.delimiter;   // Windows: ';'    POSIX: ':'   ← 多个路径之间的分隔符
```

delimiter解析path环境变量

```js
process.env.PATH.split(path.delimiter).forEach(p => console.log(p));
// Windows 用 ; 是因为 : 被盘符占用了（C:）
```





#### `path.basename` / `dirname` / `extname` —— 切割三兄弟

```
/home/user/docs / note.md
└──── dirname ──┘ └basename┘
                    note  .md
                          └extname┘
```

```js
path.basename('/a/b/note.md');           // 'note.md'
path.basename('/a/b/note.md', '.md');    // 'note'        ← 第二参数：去掉指定后缀
path.dirname('/a/b/note.md');            // '/a/b'
path.extname('/a/b/note.md');            // '.md'         ← 注意带点
```





#### `path.normalize` —— 规范化

解析 `.` 和 `..`、合并重复分隔符、统一分隔符

```js
// Windows 机器上实测：
path.normalize('/a/b//c/../d/');   // '\a\b\d\'   ← 注意！斜杠被转成了反斜杠
path.posix.normalize('/a/b//c/../d/'); // '/a/b/d/'
```





####  `path.join` —— 拼接

`path.join(...片段)`  =  用 sep 连起来，再 normalize

```js
path.join('/a', 'b', 'c');          // '\a\b\c'（win）
path.join('/a', 'b', '..', 'c');    // '\a\c'      ← 自动消化 ..
path.join('a', '', 'b');            // 'a\b'       ← 空串被忽略
path.join();                        // '.'         ← 无参数返回当前目录
path.join('/a', '/b');              // '\a\b'      ← 后面的 / 只是普通分隔符！
```





#### `path.resolve` —— 解析成绝对路径

把参数当成一串 `cd` 命令，从左往右执行，最后打印 `pwd`

```js
const filePath = path.resolve('/a', 'b', 'c');   // cd /a; cd b; cd c   → '/a/b/c'
```

算法是**从右往左**扫描，遇到绝对路径就停止，若始终没遇到就在最前面拼上 `process.cwd()`

再比如：

```js
path.resolve('/a', 'b', '../c');   // 'C:\a\c'
```

| 步骤        | 当前操作                     | 当前路径                                  |
| :---------- | :--------------------------- | :---------------------------------------- |
| 初始        | 从 cwd 出发                  | `C:\Users\19425\Desktop\笔记\agnet讲解师` |
| 处理 `/a`   | 遇到绝对路径，**重置根目录** | `C:\a`                                    |
| 处理 `b`    | 相对路径，直接拼接           | `C:\a\b`                                  |
| 处理 `../c` | 先 `..` 退一级，再进 `c`     | `C:\a\c` ✅                                |





#### `path.relative(from, to)` —— 求相对路径

回答："站在 `from`，怎么走到 `to`？"

```js
path.posix.relative('/data/o/test/aaa', '/data/o/impl/bbb');
// '../../impl/bbb'    ← 机器实测通过
```

推导过程：公共前缀是 `/data/o`，从 `test/aaa` 退两级





#### 路径穿越攻击

服务器有一个文件接口，接收用户传来的文件名，然后把文件内容返回给用户：

```js
app.get('/file', (req, res) => {
  const p = path.join('/var/www/public', req.query.name);
  res.sendFile(p);
});
```

**本意：** 用户传 `?name=photo.jpg`，服务器返回 `/var/www/public/photo.jpg`



攻击者传入的不是正常文件名，而是 `?name=../../../etc/passwd`：

```js
path.join('/var/www/public', '../../../etc/passwd')
```

`path.join` 会**老老实实地处理 `..`（上级目录）**，计算结果：

```
/var/www/public
  ├── ../  →  /var/www
  ├── ../  →  /var
  └── ../  →  /
         + etc/passwd  →  /etc/passwd  💀
```

最终拿到的路径是 `/etc/passwd`（Linux 系统的账户密码文件），服务器就把这个敏感文件直接返回给攻击者了。



防御方式：

计算出用户想访问的最终绝对路径，判断它是否还在允许的根目录之内。

```js
const ROOT = path.resolve('/var/www/public');  // 安全根目录

function safeJoin(root, userInput) {
  // 第一步：算出最终绝对路径（resolve 会处理所有的 ..）
  const target = path.resolve(root, userInput);

  // 第二步：算出 target 相对于 root 的相对路径
  const rel = path.relative(root, target);

  // 第三步：判断是否逃逸
  if (rel.startsWith('..') || path.isAbsolute(rel)) {
    throw new Error('路径穿越攻击');
  }

  return target;
}
```





### `url` —— 按规范拆解地址

**`new URL(string)`** 是浏览器（和 Node.js）内置的 URL 解析器，传入一个 URL 字符串，返回一个包含各部分属性的对象。

```js
const u = new URL('https://user:pw@www.site.com:8080/a/b?x=1&y=2#top');

u.href;         // 'https://user:pw@www.site.com:8080/a/b?x=1&y=2#top'
u.protocol;     // 'https:'          ← 带冒号！
u.username;     // 'user'
u.password;     // 'pw'
u.host;         // 'www.site.com:8080'   ← 带端口
u.hostname;     // 'www.site.com'        ← 不带端口
u.port;         // '8080'                ← 字符串，不是数字
u.pathname;     // '/a/b'
u.search;       // '?x=1&y=2'        ← 带问号！
u.searchParams; // URLSearchParams 对象
u.hash;         // '#top'            ← 带井号！
u.origin;       // 'https://www.site.com:8080'  ← 只读
```



| 属性           | 中文含义       | 解释                                                         |
| :------------- | :------------- | :----------------------------------------------------------- |
| `href`         | 完整地址       | 整个 URL 字符串，原样返回全部内容                            |
| `protocol`     | 协议           | 通信所用的规则/方式，告诉浏览器"用什么方式去取资源"          |
| `username`     | 用户名         | 访问资源时的身份凭证（用户名部分）                           |
| `password`     | 密码           | 访问资源时的身份凭证（密码部分）                             |
| `host`         | 主机（含端口） | 服务器的完整地址，包含域名和端口号                           |
| `hostname`     | 主机名         | 服务器的域名或 IP，**不含**端口号                            |
| `port`         | 端口号         | 服务器上具体"哪扇门"，区分同一台服务器上的不同服务           |
| `pathname`     | 路径           | 服务器上资源的具体位置，类似文件系统的目录路径               |
| `search`       | 查询字符串     | `?` 之后的键值对，用于向服务器传递额外参数                   |
| `searchParams` | 查询参数对象   | `search` 的结构化封装，可直接用 `.get()` 读取参数            |
| `hash`         | 哈希/锚点      | `#` 之后的内容，指向页面内某个具体位置，**不会发送给服务器** |
| `origin`       | 源             | 协议 + 主机 + 端口的组合，是浏览器**同源策略**的判断基准     |





#### `URLSearchParams` —— 查询参数

```js
const sp = new URLSearchParams('a=1&b=2&a=3');

sp.get('a');       // '1'          ← 只拿第一个
sp.getAll('a');    // ['1', '3']   ← 拿全部（重复 key 是合法的！）
sp.has('b');       // true
sp.append('c', 4); // 追加（允许重复）
sp.set('a', 9);    // 覆盖：删掉所有 a，只留一个,set的第二个参数9表示的含义是将键 'a' 的值设置为 '9'，如果 'a' 已经存在，则会删除所有现有的 'a' 键值对，并只保留一个新的 'a=9' 键值对。
sp.delete('b');
sp.toString();     // 'a=9&c=4'
sp.sort();         // 按 key 排序（做缓存 key 时用来归一化）
```





#### `fileURLToPath` —— 查询参数

将 URL 对象或 URL 字符串转换为文件系统路径字符串

```js
import { fileURLToPath } from 'node:url';
import { dirname } from 'node:path';
const __filename = fileURLToPath(import.meta.url);
```



在 ES Modules（ESM）中，Node.js **不再提供** CommonJS 的 `__filename` 和 `__dirname` 全局变量。
但模块本身的 URL 可以通过 `import.meta.url` 获取，格式类似：

```js
import.meta.url
// → "file:///Users/lyryang/project/src/index.js"
```

`fileURLToPath` 的转换效果

```js
fileURLToPath(import.meta.url)
// → "/Users/lyryang/project/src/index.js"  (Unix)
// → "C:\\Users\\lyryang\\project\\src\\index.js"  (Windows)
```

它会处理：

- 移除 `file://` 协议前缀
- 对 URL 编码字符进行**解码**（如 `%20` → 空格）
- 在 **Windows** 上正确转换为反斜杠路径



### `util` —— 工具箱



#### `util.promisify` —— 回调 → Promise



Node 的"错误优先回调"约定

```js
fn(参数..., (err, result) => {})
//            ↑ 第一个参数是错误   ↑ 回调必须是最后一个参数
```

只有满足这两条的函数才能被 `promisify`。



`promisify`可以避免形成回调地狱

Node.js 传统 API（如 `fs.readFile`）采用"错误优先回调"风格：

```js
fs.readFile('a.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err); // 第一个参数是错误
  } else {
    console.log(data);  // 第二个参数是结果
  }
});
```

promisify 转换后

```js
const readFile = util.promisify(fs.readFile);
// readFile 现在返回一个 Promise

readFile('a.txt', 'utf8')
  .then(text => console.log(text))   // 成功时拿到结果
  .catch(err => console.error(err)); // 失败时捕获错误
```



原理：

```js
function myPromisify(original) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      original.call(this, ...args, (err, value) => {
        if (err) reject(err);
        else resolve(value);
      });
    });
  };
}
```

1. 用call保留 `this`（所以对象方法也能用）。
2. 将回调函数中的回调中的`err`和`value`传入`Promise`的`resolve`和`reject`中





#### `util.callbackify` —— Promise → 回调（反方向）

`util.callbackify` 是 `util.promisify` 的**逆操作**：将一个返回 Promise 的 `async` 函数，转换为遵循 Node.js **错误优先回调**风格的函数。

```js
const util = require('node:util');

async function fetchData() {
  return 'Hello, World!';
}

const fetchDataWithCallback = util.callbackify(fetchData);

fetchDataWithCallback((err, result) => {
  if (err) throw err;
  console.log(result); // 'Hello, World!'
});
```





#### `util.inherits` —— 上古的继承方式

`util.inherits` 用于**在构造函数之间建立原型继承关系**，让子类继承父类的方法。



源码

```js
function inherits(ctor, superCtor) {
  ObjectDefineProperty(ctor, 'super_', {
    value: superCtor, writable: true, configurable: true,
  });
  ObjectSetPrototypeOf(ctor.prototype, superCtor.prototype);
}
```

1. 把 `子.prototype` 的原型指向 `父.prototype`（打通实例方法查找链）。
2. 挂一个 `子.super_` 指向父构造函数（方便调用父类方法）。



用法

```js
const util = require('node:util');

// 导入 EventEmitter 类
const EventEmitter = require('node:events'); 

// 创建MyStream类
function MyStream() {
  EventEmitter.call(this);      // ⚠️ 必须手动调用父构造函数！
}

// inherits参数1：子类构造函数
// inherits参数2：父类构造函数
// 作用：让子类继承父类的原型方法
util.inherits(MyStream, EventEmitter);

```

es6之后直接用extends

```js
class MyStream extends EventEmitter {
  constructor() { super(); }     // super() 自动调父构造函数
  write(d) { this.emit('data', d); }
}
```





#### `util.isDeepStrictEqual` —— 深度严格比较

```js
const util = require('node:util');
util.isDeepStrictEqual({a:[1,{b:2}]}, {a:[1,{b:2}]});   // true
```





### 自测

- A1. `os.cpus()[0].times.idle` 突然变小了，可能吗？为什么？
- A2. 为什么建议用 `os.availableParallelism()` 而不是 `os.cpus().length`？
- A3. 写文件用 `os.EOL`，读文件为什么不能用？



- B1. `path.join('/a', 'b', '/c')` 和 `path.resolve('/a', 'b', '/c')` 分别是什么？
- B2. `path.extname('.env.local')` 是什么？
- B3. 一个脚本 `C:\proj\src\run.js`，用户在 下执行 `node proj\src\run.js`。`path.resolve('./cfg.json')` 和 `path.join(__dirname,'cfg.json')` 分别指向哪？
- B4. 为什么 `target.startsWith(root)` 不足以防路径穿越？



- C1. `new URL('a.html', 'https://s.com/dir/page.html').href` = ?
- C2. `new URL('a.html', 'https://s.com/dir/').href` = ?
- C3. `new URL('http://a.com:80/x').href` = ?
- C4. `new URLSearchParams({t:'1 2'}).toString()` = ?



- D1. `util.callbackify(async () => { throw 'oops'; })` 的回调收到的 `err` 是什么？为什么？
- D2. `util.inherits` 和 `class extends` 在静态方法上有什么区别？
- D3. `util.isDeepStrictEqual([1,2], [1,2])`、`(NaN, NaN)`、`(0, -0)` 分别是什么？
- D4. `util.promisify` 要求被包装的函数满足什么两个条件？







#### 自测答案：

**A1.** 不可能（除非机器重启或 CPU 热插拔）。`times` 是开机以来的**单调递增**累计毫秒数。这正是必须做两次采样求差的原因。

**A2.** `cpus().length` 返回物理机总核数，无视 cgroup/CPU affinity 限制；容器里会严重高估，导致开过多 worker 反而变慢。`availableParallelism()` 返回**进程实际可用**的并行度。

**A3.** 因为 `os.EOL` 反映的是**当前运行平台**，而文件的换行符取决于**它被创建的平台**。跨平台传输的文件用 `os.EOL` 分割会失败。读用 `/\r\n|\r|\n/`。

**B1.** `path.join('/a','b','/c')` → `'/a/b/c'`（`/c` 只是普通片段）；`path.resolve('/a','b','/c')` → `'/c'`（在 Windows 上是 `C:\c`，因为遇到绝对路径就丢弃左侧全部）。

**B2.** `'.local'`。`extname` 只看**最后**一个点，且忽略开头的点。

**B3.** `path.resolve('./cfg.json')` → `C:\cfg.json`（基于 cwd，也就是用户敲命令的 ）；`path.join(__dirname,'cfg.json')` → `C:\proj\src\cfg.json`（基于脚本位置）。**后者才是你想要的**。

**B4.** 前缀绕过：`/var/www/public-evil/x` 满足 `startsWith('/var/www/public')`，但它不在目标目录内。必须用 `path.relative` 判断结果是否以 `..` 开头 / 是否为绝对路径。

**C1.** `'https://s.com/dir/a.html'`（base 的最后一段 `page.html` 被替换掉）。

**C2.** `'https://s.com/dir/a.html'`（base 以 `/` 结尾，直接在其下追加）。两题结果相同，但推导路径不同——这是最容易搞混的地方。

**C3.** `'http://a.com/x'`。80 是 http 的默认端口，被规范要求省略；同时空路径补成 `/x` 保持不变。

**C4.** `'t=1+2'`。`URLSearchParams` 使用表单编码规则，空格编码为 `+`。

**C5.** `.pathname` 会带多余的前导 `/`、保留百分号编码、且用 `/` 而非 `\`。实测 `new URL('file:///C:/x y.txt').pathname` → `'/C:/x%20y.txt'`，完全不能当路径用。必须用 `fileURLToPath()`。

**D1.** 收到一个 `Error` 实例，`err.code === 'ERR_FALSY_VALUE_REJECTION'`，`err.reason === 'oops'`。等等——`'oops'` 是**truthy** 字符串，所以其实会**原样**传给回调（`err === 'oops'`）。只有 reject 值为 falsy（`null`/`undefined`/`0`/`''`/`false`/`NaN`）时才会被包装成 `ERR_FALSY_VALUE_REJECTION`。**这道题是陷阱题，仔细看 4.2 节的规则。**

**D2.** `util.inherits` 只设置 `子.prototype` 的原型，**不继承静态方法**；`class extends` 额外设置了 `Object.setPrototypeOf(子, 父)`，静态方法会被继承。

**D3.** `true` / `true` / `false`。第二、三个和 `===` 的行为**正好相反**，因为用的是 `Object.is` 语义。

**D4.** ① 回调必须是**最后一个**参数；② 回调必须是 `(err, value)` 错误优先形式。不满足时需要挂 `util.promisify.custom`。







## 文件I/O

**I/O** 是 **Input/Output（输入/输出）** 的缩写，指程序与外部系统之间的数据交换操作。



### I/O 到底慢在哪

Node 进程跑在 **用户态（user mode）**，CPU 在这个模式下**没有权限**直接给磁盘控制器发指令。想读文件，只能"求"操作系统：

```
S 代码
   ↓ fs.readFile('a.txt')
Node 的 C++ 层 (node_file.cc)
   ↓
libuv
   ↓ 系统调用 syscall  ← 这里发生【用户态 → 内核态】切换
操作系统内核 (Linux: open/read/close  Windows: CreateFile/ReadFile)
   ↓ 驱动程序
磁盘控制器 → 物理磁盘
```

整个过程非常消耗时间



速度差距数量级：

| 操作             | 耗时    | 放大到"人类时间"（×10 亿） |
| ---------------- | ------- | -------------------------- |
| CPU 执行一条指令 | ~0.3 ns | 1 秒                       |
| L1 缓存读取      | ~1 ns   | 3 秒                       |
| 内存读取         | ~100 ns | 5 分钟                     |
| SSD 随机读       | ~100 μs | 3.5 天                     |
| 机械硬盘寻道     | ~10 ms  | 1 年                       |
| 同城网络往返     | ~1 ms   | 1 个月                     |
| 跨洲网络往返     | ~150 ms | 15 年                      |

 SSD（固态硬盘）

内存读取（程序运行时数据存放的地方）比 SSD 快约 1000 倍，比机械硬盘快约 10 万倍。



因为 I/O 耗时是 CPU 执行指令耗时的**几十万倍**，如果 CPU 同步等待 I/O，就等于把几十万次计算机会白白扔掉——所以必须用**异步非阻塞**的方式，让 CPU 在等待 I/O 期间去做其他事情。





###  Node "异步非阻塞"

让 CPU 在等待 I/O 期间去做其他事情

```
阻塞模型：
  发起I/O → 😴等待100μs → 收到数据 → 继续执行
             ↑
         浪费33万条指令的时间

非阻塞模型（Node.js）：
  发起I/O → 继续执行其他代码 → 收到I/O完成通知 → 处理数据
             ↑
         这段时间CPU没有空转，一直在干活
```

文件I/O的异步和浏览器事件循环有所不同：

```
网络 I/O  →  走操作系统的事件通知机制，真·异步，不占线程

文件 I/O  →  走 libuv 的【线程池】，默认只有 4 个线程！
             因为 Linux 上文件的原生异步 API 历史上很残缺，libuv 只能用
             "开个线程去同步地读，读完通知主线程" 来模拟异步
```





### fs 模块



#### 三副面孔：callback / promise / sync

`fs` 模块的每个 API 基本都有三个版本：

```js
const fs = require('fs');
const fsp = require('fs/promises');   // 等价于 require('fs').promises


// ① 回调版（error-first 约定）
fs.readFile('a.txt', 'utf8', (err, data) => {
  if (err) return console.error('出错了', err);
  console.log(data);
});


// ② Promise 版（推荐，配合 async/await）
async function main() {
  try {
    const data = await fsp.readFile('a.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('出错了', err);
  }
}


// ③ 同步版（会阻塞！方法名一律以 Sync 结尾）
try {
  const data = fs.readFileSync('a.txt', 'utf8');
  console.log(data);
} catch (err) {
  console.error('出错了', err);
}
```

**error-first callback 约定**：Node 回调的第一个参数永远是错误对象。没出错时是 `null`。这是整个 Node 生态的铁律



新建新建 `lab1.js`：

```js
const fs = require('fs');

console.log('1. 开始');

fs.readFile(__filename, 'utf8', (err, data) => {
  console.log('3. 文件读完了，长度 =', data.length);
}); // __filename的意思是当前文件的路径，这行代码的意思是读取当前文件的内容，读取完成后会执行回调函数。

console.log('2. 结束');
```

运行 `node lab1.js`，输出顺序是 **1 → 2 → 3**。

`readFile` 那一行执行时，只是"下了个单"就立刻返回了，同步代码继续往下跑。等线程池把文件读完，回调才被塞进事件循环执行。



同步版本读一次

```js
const fs = require('fs');

console.log('1. 开始');
const data = fs.readFileSync(__filename, 'utf8');
console.log('3. 文件读完了，长度 =', data.length);
console.log('2. 结束');
```

顺序变成 1 → 3 → 2。中间那行**卡住**了后面所有代码。3步骤输出了文件长度，说明文件读完了才执行后面代码





#### Buffer：读文件默认不给字符串

```js
fs.readFile('a.txt', (err, data) => {
  console.log(data);          // <Buffer 68 65 6c 6c 6f>
  console.log(typeof data);   // object
});
```

因为 **Node 不知道这个文件是什么**。它可能是 `.txt`（UTF-8）、可能是 `.png`（二进制）、可能是 GBK 编码的老文件。磁盘上躺着的永远只是**一串字节**，"字符"是人强加的解释。



所以node原样给字节，由开发者决定怎么解释。

```js
// 方式一：告诉 readFile 用什么编码解释
fs.readFile('a.txt', 'utf8', (err, data) => { /* data 是 string */ });
fs.readFile('a.txt', { encoding: 'utf8' }, ...);   // 完整写法

// 方式二：拿到 Buffer 后自己转
fs.readFile('a.txt', (err, buf) => {
  const str = buf.toString('utf8');
});
```





#### Buffer 是什么

**Buffer 是类型化数组**，继承自 `Uint8Array`（`U`nsigned `Int`eger `8`-bit = 无符号 8 位整数）。所以 Buffer 本质就是「一个每个元素都是 0~255 的数组」。

```js
const buf = Buffer.from('hello');
console.log(buf);              // <Buffer 68 65 6c 6c 6f> 16进制
```



**分配在 V8 堆外（off-heap）**。这点很重要：Buffer 不占 V8 的堆内存配额，也不参与常规 GC 扫描 —— 因为 I/O 缓冲区往往很大，放堆里会拖垮 GC。



截断乱码

```js
const buf = Buffer.from('你好世界', 'utf8');   // 12 字节

console.log(buf.toString('utf8'));            // 你好世界
console.log(buf.slice(0, 4).toString('utf8')); // 你�   ← 乱码！
```

`slice(0,4)` 切下了 `你`的 3 字节 + `好`的第 1 个字节，那半个字节无法组成合法 UTF-8 序列，于是变成替换字符 `�`。





#### 路径

 fs 的相对路径，相对的是「运行 node 的目录」，不是「js 文件所在目录」

```js
// C:\project\src\app.js
fs.readFileSync('./data.txt');
```

这行代码找的**不是** `C:\project\src\data.txt`，而是 `path.resolve(process.cwd(), './data.txt')`。



读文件最好永远拼绝对路径

```js
const path = require('path');
const fs = require('fs');

// CommonJS
const p = path.resolve(__dirname, 'data.txt');

// ESM (Node 20.11+)
// const p = path.resolve(import.meta.dirname, 'data.txt');

fs.readFileSync(p, 'utf8');
```





#### `fs.readFile` —— 读取文件

```js
fs.readFile语法
fs.readFile(path[, options], callback)
参数：
  path: 文件路径
  options: 可选参数，指定编码格式和文件标志,例如：['utf8', 'r']
  callback: 回调函数，传入两个参数err和data
```

```js
const fs = require('fs');
const fsp = require('fs/promises');
const path = require('path');

const file = path.resolve(__dirname, 'demo.txt');

// 回调
fs.readFile(file, 'utf8', (err, data) => {
  if (err) {
    if (err.code === 'ENOENT') return console.log('文件不存在');
    throw err;
  }
  console.log(data);
});

// await
const data = await fsp.readFile(file, 'utf8');
```



**"一次性读取大文件会撑爆内存"** 的问题

`fs.readFile` 的工作方式是：**把整个文件完整读入内存，然后才交给开发者用**

```
文件 (300MB) ──读取──▶ Buffer (300MB 内存) ──▶ 你的回调
```

所以：

- 1 个用户下载 → 占用 300MB 内存
- 10 个用户同时下载 → 占用 **3000MB (3GB)** 内存
- 内存耗尽 → **进程直接崩溃 (OOM)**



硬件上限

| 读取方式       | 上限                | 超出后果                   |
| :------------- | :------------------ | :------------------------- |
| 读成**字符串** | ~512MB (V8 限制)    | 抛出 `ERR_STRING_TOO_LONG` |
| 读成**Buffer** | ~4GB (Node.js 限制) | 超出直接报错               |





#### `fs.writeFile` —— 写入文件

fs.writeFile —— 写入文件
语法：

```js
fs.writeFile(file, data[, options], callback)
// options: { encoding: 'utf8', mode: 0o666, flag: 'w' }
```

参数：

-   file 是要写入的文件名或文件描述符。

-   data 是要写入的内容，可以是字符串或 Buffer。

-   options 是一个可选的对象，包含以下属性：

  - ​    encoding：指定文件的字符编码，默认值为 'utf8'。

  - ​    mode：指定文件的权限，默认值为 0o666。

  - ​    flag：指定文件的打开方式，默认值为 'w'。

-   callback 是一个回调函数，当写入操作完成时调用。它接受一个参数 err



```js
await fsp.writeFile(file, 'hello 世界');            // string 默认按 utf8 编码
await fsp.writeFile(file, Buffer.from([1,2,3]));    // Buffer 原样写，编码参数被忽略
```



`flag` 决定写入行为

默认 `flag: 'w'` 会做三件事：不存在则创建、存在则清空（truncate）、然后从头写。

| flag   | 含义     | 文件不存在  | 文件已存在      |
| ------ | -------- | ----------- | --------------- |
| `'r'`  | 只读     | 报错 ENOENT | 从头读          |
| `'w'`  | 只写     | 创建        | **清空**        |
| `'w+'` | 读写     | 创建        | **清空**        |
| `'a'`  | 追加     | 创建        | 从末尾追加      |
| `'a+'` | 读+追加  | 创建        | 从末尾追加      |
| `'wx'` | 只写独占 | 创建        | **报错 EEXIST** |
| `'ax'` | 追加独占 | 创建        | 报错 EEXIST     |

```js
// 追加写日志
await fsp.writeFile('log.txt', '一行日志\n', { flag: 'a' });
```



该API的底层

```js
const fs = require('fs');

const fd = fs.openSync('x.txt', 'w+');   // ① 打开，拿文件描述符
fs.writeSync(fd, 'hello world');         // ② 写
const buf = Buffer.alloc(5);
fs.readSync(fd, buf, 0, 5, 6);           // ③ 从偏移量 6 读 5 字节 → 'world'
console.log(buf.toString());
fs.closeSync(fd);                        // ④ 必须关闭，否则 fd 泄漏
```

fd（file descriptor）是一个小整数，是内核为当前进程维护的"已打开文件表"的索引。进程能打开的 fd 数量有上限，忘记 `close` 会导致 `EMFILE: too many open files`。





#### `fs.stat` —— 文件/目录信息

```js
const fsp = require('fs/promises');
const st = await fsp.stat(file);
```

- file: 文件路径
- st: 文件/目录信息对象



```js
const st = fs.statSync(__filename); // 同步获取当前文件信息

console.log(st.size);          // 字节数（目录的 size 无意义，别用）
console.log(st.atime);         // Date  上次【访问/读取】
console.log(st.mtime);         // Date  上次【内容修改】       modify
console.log(st.ctime);         // Date  上次【元数据变更】     change ⚠️
console.log(st.birthtime);     // Date  【创建】时间

console.log(st.atimeMs, st.mtimeMs);   // 同样的时间，毫秒数字版

console.log(st.isFile());          // 是否普通文件
console.log(st.isDirectory());     // 是否目录
console.log(st.isSymbolicLink());  // 是否符号链接（stat 恒为 false，见 6.4）

console.log(st.mode.toString(8));  // 权限位，八进制
console.log(st.ino, st.dev);       // inode 号 + 设备号 → 唯一标识一个文件
```

打印结果：

```powershell
1057
2026-07-29T22:58:54.184Z
2026-07-29T22:58:51.521Z
2026-07-29T22:58:51.521Z
2026-07-17T03:41:59.179Z
1785365934184.3677 1785365931521.1377
true
false
false
100666
19984723346621868 947368919
```



**`ctime` = change time = inode（元数据）变更时间**。

触发 `ctime` 更新的操作：

- 修改文件内容（同时也更新 mtime）
- **改权限 `chmod`、改属主 `chown`**（只动 ctime，不动 mtime）
- **重命名 `rename`**
- 创建/删除硬链接

所以 **`ctime` 只会前进，永远不能被程序回拨** —— 这正是它的设计目的：防篡改审计



`size` 与"占用空间"不是一回事

`st.size` 是**逻辑长度**。真正的磁盘占用是 `st.blocks * 512`，因为文件系统按块（通常 4KB）分配。1 字节的文件也占 4KB。稀疏文件甚至可能 size 很大而 blocks 很小。





#### `fs.readdir` —— 列出目录内容

```js
import fsp from 'fs/promises';
const dir = './'; // 指定要读取的目录
const names = await fsp.readdir(dir);
console.log(names);   // ['a.txt', 'sub', 'b.png']
```





#### `fs.mkdir` —— 创建目录

```
// ❌ 父目录不存在会报 ENOENT；目录已存在会报 EEXIST
await fsp.mkdir('a/b/c');

// ✅ 99% 的场景都该这样写
await fsp.mkdir('a/b/c', { recursive: true });
//   ① 自动创建所有缺失的父目录（等价于 mkdir -p）
//   ② 已存在时【不报错】，直接返回 undefined
//   ③ 返回值是【第一个被创建的目录】的路径，全都已存在则返回 undefined
```

配套：

```js
await fsp.rmdir(dir);                              // 只能删空目录
await fsp.rm(dir, { recursive: true, force: true }); // 递归删除（Node 14.14+，推荐）
await fsp.unlink(file);                            // 删文件
```





#### `fs.existsSync` —— 目录是否存在

```js
const fs = require('fs');
const dir = './';         // 指定要读取的目录
const c = fs.existsSync(dir);
if (c) console.log('目录存在');
```

```js
// ② 异步版：fsp.access
const fs = require('fs');
const fsp = require('fs/promises');
try {
  await fsp.access(p, fs.constants.F_OK);   // F_OK 存在 / R_OK 可读 / W_OK 可写
  console.log('存在');
} catch {
  console.log('不存在');
}
```







## 文件流

```mermaid
graph LR
    A[磁盘文件<br/>源头] -->|_read 每次搬 hwm 字节| B[可读流缓冲区<br/>readableLength]
    B -->|data 事件 / read| C[你的代码]
    C -->|ws.write| D[可写流缓冲区<br/>writableLength]
    D -->|_write 逐块落盘| E[磁盘文件<br/>目的地]
```





### 什么是流

> 数据从一个地方**缓缓地**流动到另一个地方。

关键词是"缓缓"——不是一次性搬完，而是**分批、有节奏**。



`Readable` 可读流，数据从外部源头流进开发者的程序

`Writable` 可写流，数据从开发者的程序流向外部目的地

`Duplex` 双工流，一个对象同时是可读流和可写流，读写两端互不相干





### 为什么需要流

1、数据规模不一致

磁盘那个方块很大，内存那个方块很小。一个 4GB 的日志文件，`fs.readFile` 会直接给你一个 4GB 的 Buffer——但 Node 的 Buffer 有上限，V8 堆也有上限，进程直接 OOM 崩溃。

`readFile` 的内存占用是 O(文件大小)，流是 O(1)。文件涨到 3GB，`readFile` 直接死，流的内存曲线纹丝不动。代价是慢了 35%（多了一堆事件调度）——这是标准的时间换空间。

<img src="./Typora图片/image-20260730075506480.png" alt="image-20260730075506480" style="zoom:50%;" />



2、处理能力不一致

内存里的数据"快"，磁盘里的数据"慢"，中间那根管子是细的。

这意味着**即使内存装得下，也不能一次性把数据糊到目的地**。如果你按内存的速度往磁盘/网络里灌，多出来的数据只会堆在中间的队列里——这就是**背压（backpressure）**

<img src="./Typora图片/image-20260730075716589.png" alt="image-20260730075716589" style="zoom:50%;" />





### 可读流 Readable



#### 创建

```js
// 创建可读流 Readable
const readStream = fs.createReadStream(path, {
  encoding: null,        // 默认 null → 给你 Buffer；给 'utf8' → 给你 string
  start: undefined,      // 起始字节（含）
  end: undefined,        // 结束字节（含！闭区间）
  highWaterMark: 65536,  // 每次从磁盘搬多少字节，默认 64KB
  autoClose: true,       // 读完/出错后自动关 fd
  flags: 'r',
})
```





#### 完整示例

demo.txt

```txt
abcdefghijklmnopqrstuvwxyz
```

准备文件路径

```js
import fs from "node:fs"; // 导入文件系统模块
import path from "node:path"; // 导入路径模块
import { fileURLToPath } from "node:url"; // 导入 URL 模块，fileURLToPath 用于将 file URL 转换为文件路径

const __dirname = path.dirname(fileURLToPath(import.meta.url)); // 获取当前文件所在目录的绝对路径 C:\UsereadStream\19425\Desktop\笔记\agnet讲解师
const FILE = path.resolve(__dirname, "demo.txt"); // 解析文件路径 C:\UsereadStream\19425\Desktop\笔记\agnet讲解师\文件流\demo.txt

const t0 = Date.now(); // 记录当前时间戳
const log = (...a) => console.log(`[${String(Date.now() - t0).padStart(4)}ms]`, ...a); // 定义日志函数，输出时间戳和日志内容
```

可读文件流代码

```js
const readStream = fs.createReadStream(FILE, { highWaterMark: 10 }); // 创建一个可读流，读取 demo.txt 文件，每次从磁盘般 10 字节数据到缓冲区

readStream.on("open", (fd) => log("open  文件已打开，fd =", fd)); // 监听 open 事件，文件打开时触发，输出文件描述符 fd
readStream.on("ready", () => log("ready 流已就绪")); // 监听 ready 事件，流已就绪时触发
readStream.on("close", () => log("close 文件已关闭（autoClose 默认 true）")); // 监听 close 事件，文件关闭时触发
readStream.on("error", (err) => log("error", err.message));
readStream.on("end", () => log("end   数据已全部读完（此时文件还没关）"));

log("—— 已注册 open/end/close，但还没注册 data ——");
log("现在同步睡 100ms，观察这期间有没有 data 冒出来");

const until = Date.now() + 100;
while (Date.now() < until) {} // 阻塞主线程 100ms

readStream.on("data", (chunk) => {
  log(`data  收到 ${chunk.length} 字节:`, JSON.stringify(chunk.toString()));
}); // 监听 data 事件，读取到数据块时触发，输出数据块长度和内容
```

输出结果

```js
[   0ms] —— 已注册 open/end/close，但还没注册 data ——
[   2ms] 现在同步睡 100ms，观察这期间有没有 data 冒出来
[ 104ms] open  文件已打开，fd = 3
[ 104ms] ready 流已就绪
[ 106ms] data  收到 10 字节: "abcdefghij"
[ 106ms] data  收到 10 字节: "klmnopqrst"
[ 106ms] data  收到 6 字节: "uvwxyz"
[ 106ms] end   数据已全部读完（此时文件还没关）
[ 106ms] close 文件已关闭（autoClose 默认 true）
```

执行时序拆解：

```
JS主线程（单线程）                     					libuv I/O线程池
─────────────────────────────────────  				──────────────────────
createReadStream() → 向libuv提交"打开文件"请求
注册 open/ready/close/error/end 监听
打印日志 [0ms] [2ms]
while(busy-wait 100ms) ←── 占住主线程    				←── 文件很快打开完毕
                                              				open回调 → 放入事件队列
                                              				（队列中等待，无法执行）
while 结束
注册 data 监听
↓ 同步代码全部执行完毕，主线程归还给事件循环
事件循环: 取出 open 回调 → 触发 [104ms] open
                         → 触发 [104ms] ready
此时 data 已注册 → 流进入 flowing 模式 → 连续触发 data/end/close
```





#### 消费数据方式

流动模式（推）——`on('data')`，数据推给你，你被动接：

```js
const readStream = fs.createReadStream(FILE, { highWaterMark: 10 }); // 创建一个可读流，读取 demo.txt 文件，每次从磁盘般 10 字节数据到缓冲区
readStream.on('data', chunk => { /* ... */ });
```

暂停模式（拉）——`on('readable')` + `rs.read(n)`，你主动去缓冲区取：

```js
readStream.on('readable', () => {
  let chunk;
  while ((chunk = rs.read(3)) !== null) {   // 精确取 3 个单位，取不到返回 null
    console.log(chunk);
  }
});
```

异步迭代（最推荐）——`for await`，自带背压：

```js
const readStream = fs.createReadStream(FILE, { highWaterMark: 5, encoding: "utf8" });

// 异步迭代：每次循环自动拿一个 chunk（大小由 highWaterMark 决定）
for await (const chunk of readStream) {
  await 慢处理(chunk);   // 这行没跑完，就不会读下一块
}
```



`readStream.on('data', fn)` 内部干了这件事：

```js
// Node 源码 Readable.prototype.on 的简化逻辑
if (ev === 'data') {
  state.readableListening = true;
  state.flowing = true;
  this.resume();   // ← 这才是真正启动引擎的地方
}
```

| flowing | 含义                                    |
| ------- | --------------------------------------- |
| `null`  | 初始状态，**一个字节都不读**            |
| `true`  | 流动模式，拼命读，读到就 `emit('data')` |
| `false` | 暂停模式，读到就存进缓冲区，等你来取    |

消费模式不同就是flowing不同



暂停模式完整示例

```js
const readStream = fs.createReadStream(FILE, { highWaterMark: 5, encoding: "utf8" }); // 创建可读流，设置高水位线为 5 字节，编码为 utf8
readStream.on("readable", () => { // 监听 readable 事件，当流可读时触发
  let chunk;
  // read(n) 精确取 n 个；read() 取光缓冲区；取不到返回 null
  while ((chunk = readStream.read(3)) !== null) { // 循环读取数据，每次读取 3 字节，直到没有数据可读
    log("readable:", JSON.stringify(chunk)); // 输出读取到的数据
  }
});
```

输出

```
[   1ms] readable: "abc"
[   6ms] readable: "def"
[   7ms] readable: "ghi"
[   7ms] readable: "jkl"
[   7ms] readable: "mno"
[   7ms] readable: "pqr"
[   8ms] readable: "stu"
[   8ms] readable: "vwx"
[   8ms] readable: "yz"
```





#### `pause()` / `resume()`

这两个方法用于手动控制流的暂停与恢复

```js
const rs = fs.createReadStream(FILE, { highWaterMark: 5, encoding: "utf8" });

// 监听 data 事件 → 流自动进入 flowing 模式，开始持续推数据
rs.on("data", (chunk) => {
  rs.pause();                          // 叫停：flowing = false
  setTimeout(() => rs.resume(), 500);  // 恢复：flowing = true
});
```

比如从叫停到恢复期间的500ms就可以进行数据库操作

`pause()` 是"尽力而为"的刹车，不是立即生效。调用后，**缓冲区里已经攒好的数据可能还会再 emit 一两块**（因为 `_read` 是异步的，飞在半路的数据收不回来）。





### 可写流 Writable



#### 创建

```js
const ws = fs.createWriteStream(path, {
  flags: 'w',            // 'w' 覆盖(不存在则建、存在则清空) | 'a' 追加 | 'r+' 读写不清空
  encoding: 'utf8',      // 写入字符串时按什么编码转 Buffer
  start: undefined,      // 从第几字节开始写（配合 'r+' 可做随机写）
  highWaterMark: 16384,  // 队列容量阈值，默认 16KB（注意：可写流是 16KB，可读流是 64KB）
  autoClose: true,
})
```





#### 完整示例

创建空文件`out-05.txt`

路径处理

```js
import fs from "node:fs"; // 导入文件系统模块
import path from "node:path"; // 导入路径模块
import { fileURLToPath } from "node:url"; // 导入 URL 模块，fileURLToPath 用于将 file URL 转换为文件路径

const __dirname = path.dirname(fileURLToPath(import.meta.url)); // 获取当前文件所在目录的绝对路径 C:\UsereadStream\19425\Desktop\笔记\agnet讲解师
const OUT = path.resolve(__dirname, "out-05.txt"); // 解析文件路径 C:\UsereadStream\19425\Desktop\笔记\agnet讲解师\文件流\out-05.txt

const t0 = Date.now(); // 记录当前时间戳
const log = (...a) => console.log(`[${String(Date.now() - t0).padStart(4)}ms]`, ...a); // 定义日志函数，输出时间戳和日志内容
```

创建可读流

```js
// 创建可读流
const ws = fs.createWriteStream(OUT, { flags: "w", encoding: "utf8" }); // 创建一个可写流，写入到 OUT 文件中，'w'覆盖模式，编码为 utf8

ws.on("open", (fd) => log("open   fd =", fd)); // 监听 open 事件，输出文件描述符
ws.on("ready", () => log("ready"));
ws.on("finish", () => log("finish 所有数据已交给底层（end() 之后触发）")); // 监听 finish 事件，表示所有数据已交给底层
ws.on("close", () => log("close  文件句柄已关闭（autoClose 默认 true）"));
ws.on("error", (e) => log("error", e.code));

log("write 返回:", ws.write("第一行\n")); // 字符串会按 encoding 转成 Buffer
log("write 返回:", ws.write(Buffer.from("第二行\n"))); // 也可以直接给 Buffer

// write 的第二/三参数：编码 + 该块真正落盘后的回调
ws.write("第三行\n", "utf8", () => log("  ↳ 第三行已落盘"));

// end 可以带最后一块数据；end 之后流进入 ended 状态
ws.end("第四行(由 end 写入)\n", () => {
  log("end 回调：写完了");
  log("文件内容：\n" + fs.readFileSync(OUT, "utf8"));
});
```

终端输出：

```
[   1ms] write 返回: true
[   3ms] write 返回: true
[   4ms] open   fd = 3
[   4ms] ready
[   4ms]   ↳ 第三行已落盘
[   5ms] end 回调：写完了
[   6ms] 文件内容：
第一行
第二行
第三行
第四行(由 end 写入)

[   6ms] finish 所有数据已交给底层（end() 之后触发）
[   6ms] close  文件句柄已关闭（autoClose 默认 true）
```

`out-05.txt`文件内容

```
第一行
第二行
第三行
第四行(由 end 写入)
```





#### ws.write

```js
ws.write(chunk, [encoding], [callback])
```

参数：

- chunk，string | Buffer | Uint8Array，要写入的数据
- encoding，chunk 是**字符串**时才生效，指定编码，默认用创建流时的 `encoding`，若未指定则为 `'utf8'`
- callback，**该块数据真正落盘后**触发，可在此确认写入成功或捕获错误

```js
// 只传 chunk（字符串）
ws.write("第一行\n");

// chunk 是 Buffer，encoding 无意义，可省略
ws.write(Buffer.from("第二行\n"));

// 三个参数都传
ws.write("第三行\n", "utf8", () => {
  log("第三行已落盘"); // ← 这块数据写完才执行
});
```





#### `write()` 的返回值含义

返回值的判定公式就一行：
$$
return=(writableLength<writableHighWaterMark)
$$


其中 `writableLength` 是**当前积压在内存队列里、还没落盘的字节数**。



示例

创建空文件`out-06.txt`

路径处理

```js
import fs from "node:fs";
import path from "node:path";
import { fileURLToPath } from "node:url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const OUT = path.resolve(__dirname, "out-06.txt");

// 日志信息函数
const info = (tag, ret) =>
  console.log(
    `${tag} → 返回 ${String(ret).padEnd(5)} | writableLength=${ws.writableLength} / hwm=${ws.writableHighWaterMark}`
  ); 
```

hwm 故意设成 4 字节例子：

```js
/**
 * 心智模型：
 *   ws.write(data) 不是"写到磁盘"，而是"把 data 交给写入通道"。
 *   写入通道 = 内存缓冲区
 *   通道里已积压的字节数 = ws.writableLength，容量 = ws.writableHighWaterMark。
 *   积压 < 容量 → 返回 true （还能塞，随便写）
 *   积压 >= 容量 → 返回 false（塞爆了，你该停手，等 drain）
 *   注意：返回 false 时这次数据依然被接收了，只是进了内存队列。
 */
const ws = fs.createWriteStream(OUT, { highWaterMark: 4 });

// 判定公式：写完这块之后， writableLength < highWaterMark ? true : false
info("write('abc')  3字节", ws.write("abc")); // 3 < 4 → true，还能塞
info("write('d')    1字节", ws.write("d")); // 4 >= 4 → false，满了
info("write('efgh') 4字节", ws.write("efgh")); // 已经满了，继续 false
info("write('ijk')  3字节", ws.write("ijk")); // 数据照收不误，只是排进内存队列

// 注意上面 writableLength 一路涨到 11：因为文件还没 open，一个字节都没真正下盘
console.log("\n等待文件打开并把队列刷下去…");

ws.on("drain", () => {
  console.log(`drain! 队列已排空 writableLength=${ws.writableLength}`);
  info("排空后 write('lmn')", ws.write("lmn"));
  ws.end(() => console.log("\n文件内容:", fs.readFileSync(OUT, "utf8")));
});
```

终端输出：

```
write('abc')  3字节 → 返回 true  | writableLength=3 / hwm=4
write('d')    1字节 → 返回 false | writableLength=4 / hwm=4
write('efgh') 4字节 → 返回 false | writableLength=8 / hwm=4
write('ijk')  3字节 → 返回 false | writableLength=11 / hwm=4

等待文件打开并把队列刷下去…
drain! 队列已排空 writableLength=0
排空后 write('lmn') → 返回 true  | writableLength=3 / hwm=4

文件内容: abcdefghijklmn
```

`out-06.txt`文件内容

```
abcdefghijklmn
```

全流程数据堆积直观展示

```
同步代码（立刻执行）          异步（稍后执行）
─────────────────────        ──────────────────
ws.write("abc")   →堆积
ws.write("d")     →堆积
ws.write("efgh")  →堆积      文件 open 完成
ws.write("ijk")   →堆积  ──→ 开始刷盘 →→→→→→→ 磁盘
                             队列清空
                             触发 drain 事件
```

结论：

1. **返回 `false` ≠ 写入失败。** 数据照收，一个字节不丢，只是排进了内存队列。
2. **`false` 是一句劝告：「输入数据到达highWaterMark警戒线了，你该停手了」。** 你可以无视它，代价是内存爆炸



`drain` 事件的作用

```js
ws.on("drain", () => {
  // 队列已经排空，writableLength === 0
  // 现在可以安全地继续写了
  console.log(`drain! 队列已排空 writableLength=${ws.writableLength}`);
  
  ws.write("lmn"); // 重新开始写
  ws.end(...);
});
```





#### ws.end

```js
ws.end([chunk], [callback])
```

| 参数       | 类型              | 是否必填 | 说明                                                         |
| :--------- | :---------------- | :------- | :----------------------------------------------------------- |
| `chunk`    | `string | Buffer` | ❌ 可选   | 结束前最后写入的一块数据，等价于先 `write(chunk)` 再 `end()` |
| `callback` | `function`        | ❌ 可选   | 等价于监听一次 `finish` 事件，所有数据写完后触发             |

```js
ws.end();                   // 结束
ws.end('最后一块数据');       // 结束前顺便写一块
ws.end('data', () => {});   // 带完成回调
```





### 背压 Backpressure

核心矛盾：读快写慢

读取速度太快，写入速度太慢，导致缓冲区爆炸



模拟一个网盘

```js
// 模拟一个"慢速目的地"：网络 / U 盘 / 数据库。每块要 10ms 才吞得下
class SlowSink extends Writable {
  _write(chunk, enc, cb) {
    setTimeout(cb, 10); // cb() 就是在说"这块我消化完了，可以给我下一块"
  }
}
const sink = new SlowSink({ highWaterMark: 64 * 1024 });
```



❌ bad 模式 —— 无脑灌水

```js
rs.on("data", (chunk) => {
  read += chunk.length;
  sink.write(chunk);   // write() 返回 false 了也不管！
});
```

数据情况

```
时间轴：
t=0ms     读取：[chunk1][chunk2][chunk3]...[chunk100]  全部塞给 sink
          sink：[chunk1] 正在消化（要 10ms）...
                ^^^^^^^^ 后面 99 块全堆在 writableLength 队列里！

t=300ms   写入队列积压 = 10MB↑
t=600ms   写入队列积压 = 20MB↑
t=1000ms  写入队列积压 = 30MB↑  RSS 内存一路飙升
          ...
```



✅ good 模式 —— 手动刹车/松油门

```js
rs.on("data", (chunk) => {
  read += chunk.length;
  const ok = sink.write(chunk);
  if (!ok) {           // sink 说"我撑不住了"
    rs.pause();        // 立刻踩刹车：停止读取
    sink.once("drain", () => rs.resume());  // sink 消化完了再松油门
  }
});
```

发生了什么：

```
时间轴：
t=0ms     读取 chunk1 → sink.write() 返回 true  → 继续读
          读取 chunk2 → sink.write() 返回 false → rs.pause() ← 踩刹车！

          sink 慢慢消化队列中的块...

t=10ms    sink 触发 "drain" → rs.resume() ← 松油门，继续读
t=10ms    读取 chunk3 → sink.write() 返回 false → rs.pause()
          ...循环往复

结果：写入队列积压始终维持在 ~64KB 附近，内存平稳
```



✅ pipe 模式 —— 内置好了上述逻辑

```js
rs.pipe(sink);
// pipe 内部源码本质上就是：
// rs.on('data', chunk => { if(!dest.write(chunk)) rs.pause() })
// dest.on('drain', () => rs.resume())
// 还额外处理了 end 传播、error 传播
```





### pipe 和 pipeline 



#### pipeline

原生 `pipe` 有个著名缺陷：**上游出错时，它只会 `unpipe`，不会关闭下游。**

```js
fs.createReadStream('不存在.txt').pipe(fs.createWriteStream('out.txt'));
// 结果：out.txt 被创建成 0 字节，fd 泄漏，进程可能挂着不退出
```

生产代码一律用 `pipeline`，它保证**任何一环出错，整条链路全部 destroy**：

```js
import { pipeline } from 'node:stream';
import { pipeline as pipelineAsync } from 'node:stream/promises';

// 回调版
pipeline(rs, transform, ws, err => { if (err) ... });

// Promise 版（推荐）
await pipelineAsync(rs, transform, ws);
```

例子

```js
import { pipeline } from 'node:stream/promises';

await pipeline(
  fs.createReadStream('不存在.txt'),
  fs.createWriteStream('out.txt')
);
// ✅ rs 出错 → ws 立刻被 destroy → fd 释放 → 进程正常退出
```

参数

```js
pipeline(
  rs,         // 第1个：必须是 Readable（数据源）
  transform1, // 中间：Transform 流，0个或多个
  transform2, // 中间：Transform 流，0个或多个
  ws,         // 最后一个流：必须是 Writable（数据终点）
  (err) => {  // 最后一个参数：回调
    if (err) console.error('出错了', err);
    else     console.log('成功写完');
  }
);
```





#### Transform

关于其中的参数transform，之前有可读流 (Readable)，只能从它"读"数据；可写流 (Writable)，只能往它"写"数据；

Transform 是**既可读又可写**的流，既能写进去，又能读出来，写进去的数据会被"加工"后才能读出来

通常是引入pipeline时，同时引入Transform

```js
import { Transform, pipeline } from "node:stream";
```





#### 完整使用

准备数据源文件：demo.txt

```
abcdefghijklmnopqrstuvwxyz
```

引入相关模块，处理数据源文件路径

```js
import fs from "node:fs"; // 导入文件系统模块 
import path from "node:path"; // 导入路径模块
import zlib from "node:zlib"; // 导入 zlib 模块，用于压缩和解压缩
import { Transform, pipeline } from "node:stream"; // 导入 Transform 和 pipeline 类
import { pipeline as pipelineAsync } from "node:stream/promises"; // 导入异步版本的 pipeline
import { fileURLToPath } from "node:url"; // 导入 fileURLToPath 函数，用于将 URL 转换为文件路径

const __dirname = path.dirname(fileURLToPath(import.meta.url)); // 获取当前文件的目录路径
const SRC = path.resolve(__dirname, "demo.txt"); // 解析数据源文件路径

class UpperCase extends Transform { // 定义一个 UpperCase 类，继承自 Transform
  _transform(chunk, enc, push) { // 重写 _transform 方法
    push(null, chunk.toString().toUpperCase()); // 将数据块转换为大写并推送到下游
  }
}
```

数据流处理

场景：读文件 → 转大写 → gzip 压缩 → 写文件

pipeline 会：自动处理背压 + 任何一环出错就销毁全链路 + 统一回调

```js
pipeline(
  fs.createReadStream(SRC), // 创建可读流，从源文件读取数据
  new UpperCase(), // 创建 UpperCase 转换流，将数据转换为大写
  zlib.createGzip(), // 创建 gzip 压缩流，将数据压缩
  fs.createWriteStream(path.resolve(__dirname, "out-09.txt.gz")), // 创建可写流，将压缩后的数据写入目标文件
  (err) => {
    if (err) return console.error("链路失败:", err.message); // 如果发生错误，输出错误信息
    console.log("① callback 版 pipeline 完成 → out-09.txt.gz"); // 输出完成信息
  }
)
```

得到压缩文件`out-09.txt.gz`

可以在最后的err回调中添加反向解压逻辑查看

```js
(err) => {
  if (err) return console.error("链路失败:", err.message);
  console.log("① callback 版 pipeline 完成 → out-09.txt.gz");

  // 反向验证：解压回来看看
  pipelineAsync(
    fs.createReadStream(path.resolve(__dirname, "out-09.txt.gz")),
    zlib.createGunzip(),
    fs.createWriteStream(path.resolve(__dirname, "out-09-解压.txt"))
  ).then(() => {
    console.log(
      "② Promise 版 pipeline 完成，解压内容 =",
      fs.readFileSync(path.resolve(__dirname, "out-09-解压.txt"), "utf8")
    );
  });
}
```

得到文件`out-09-解压.txt`，均为demo.txt的大写形式

```
ABCDEFGHIJKLMNOPQRSTUVWXYZ
```



要引入一个Transform并继承，因为**`pipeline` 只接受流对象**，所以必须把"加工逻辑"包装成一个流，才能串进管道里。

```js
// ❌ 这样不行，pipeline 不接受普通函数
pipeline(
  fs.createReadStream(SRC),
  (data) => data.toString().toUpperCase(),  // 普通函数，不是流
  zlib.createGzip(),
  fs.createWriteStream("out.gz"),
)
```

必须

```js
// ✅ UpperCase 是个 Transform 流，pipeline 认识它
pipeline(
  fs.createReadStream(SRC),
  new UpperCase(),   // 流对象 ✓
  zlib.createGzip(),
  fs.createWriteStream("out.gz"),
)
```







## net模块（重学）



### 是什么

`net` 模块是 Node.js 对 TCP 协议的封装，让开发者可以用 JavaScript 直接进行 TCP 网络编程

```mermaid
graph TD
    A["应用层<br/>HTTP / WebSocket / Redis协议 / MySQL协议"] --> B["传输层 TCP<br/>👉 net 模块在这里"]
    B --> C["网络层 IP"]
    C --> D["链路层 网卡"]
```

`http` 模块 = `net` 模块 + 一套文本格式约定



`net` 能做两件事：

| 场景            | listen 的参数       | 走不走网卡             |
| --------------- | ------------------- | ---------------------- |
| 网络通信 TCP/IP | 端口号 `9000`       | 走                     |
| 进程间通信 IPC  | 路径 `\\.\pipe\xxx` | 不走，内核内存直接拷贝 |

**API 一模一样**，只是 `listen()` 传的东西不同。这是 `net` 设计上最优雅的地方。





### http长短连接



短连接（HTTP/1.0 默认）

```
三次握手 → 请求 → 响应 → 四次挥手
```

代价：每要一张图片就要重新握手一次。一个网页 100 个资源 = 100 次握手 + 100 次挥手，RTT 全浪费在建连上。



长连接（HTTP/1.1 默认，`Connection: keep-alive`）

```
三次握手 → 数据 数据 数据 数据 ... → 四次挥手
```





### 创建服务器（语法）

```js
import net from "node:net";

const server = net.createServer();   // ① 造对象:纯 JS 层面,只是 new 了一个 Server 实例

server.listen(9000);                 // ② 系统调用:真正 socket() + bind() + listen()
```

`createServer()` —— 只在 V8 堆里创建了一个 `Server` 对象(它是个 `EventEmitter`)。操作系统内核对此一无所知,没有任何文件描述符、没有占用任何端口。

`listen(9000)`—— 这一步才让内核干活:创建 socket、`bind()` 到 9000 端口、`listen()` 开始监听。从此刻起端口才被占用,内核才会为你维护"连接队列"。

一个能工作的服务器 = **对象 + 监听**,两步缺一不可。





#### 服务器事件

**`Server` 是一个 `EventEmitter`**。它自己不"返回"结果，而是在生命周期的不同阶段**发事件**通知，通过 `server.on(...)` 订阅。这四个事件正好覆盖了服务器的**完整生命周期**:

```
创建 ──listen()──► [listening] ──►── [connection]×N ──►── [close]
                        │                                    ▲
                        └──────────── [error] ───────────────┘(任何阶段都可能出错)
```



逐个说明

**`listening`** —— 当服务器成功绑定并开始监听端口时触发 

`listen()` 是异步的,所以"绑定成功"这个结果要通过事件告诉开发者。

等价写法:

```js
server.listen(9000, () => console.log("ready")); // 第二个参数就是 listening 回调
// 等价写法
server.on("listening", () => console.log("ready"));
```



**`connection`** —— 每当有新客户端成功建立连接时触发。

两个对象:

- `server` = 表示服务器整体
- `socket`= 和客户端的单个连接，`socket` 参数本质是一个 **Duplex 流**（可读可写）

```js
server.on("connection", (socket) => {
  socket.write("你好\n");                 // 给这个客户端发数据
  socket.on("data", (buf) => { /* 收 */ });
});
```

来 100 个客户端,就触发 100 次 `connection`,给开发者 100 个不同的 `socket`。



**`close`** —— 分为server和socket

`server` 上的 `close` 事件

**触发时机**：调用 `server.close()` 后，服务器停止接受新连接，并且**等到所有现存连接都断开**后才触发。

`.on`监听和直接调用`close`方法都能直接触发事件

```js
server.close(() => {
  console.log("服务器已完全关闭，所有连接已释放");
});
// 等价于
server.on("close", () => { /* ... */ });
```

`socket` 上的 `close` 事件

**触发时机**：当**这一个连接**完全关闭时触发。

```js
socket.on("close", (hadError) => {
  console.log("某个客户端断开了", hadError ? "(因错误)" : "");
});
```



**`error`** —— 端口被占用(`EADDRINUSE`)是最典型场景。**一定要监听**,因为 `EventEmitter` 的规则是:没人监听 `error` 事件时,错误会被直接 `throw` 出来导致进程退出。







#### 完整示例

```js
import net from "node:net";

// 创建一个 TCP 服务器实例（此时还未开始监听端口）
const server = net.createServer();

// "listening" 事件：
server.on("listening", () => console.log("✅ 监听中 :9000"));

// "connection" 事件
server.on("connection", (socket) => {
  console.log("🔗 新客户端:", socket.remoteAddress, socket.remotePort); // socket.remoteAddress / remotePort：客户端的 IP 地址和端口号
  socket.write("welcome\n"); // 连接建立后主动向客户端发送一段欢迎消息

  // "data" 事件：收到客户端发来的数据时触发，buf 是一个 Buffer（二进制数据）；这里把收到的内容原样回传（echo 回显）
  socket.on("data", (buf) => socket.write(`echo: ${buf}`));

  // "close" 事件：该客户端连接关闭时触发
  socket.on("close", () => console.log("👋 客户端断开"));
});

// "error" 事件：服务器发生错误时触发（必须监听，否则错误会被抛出导致进程崩溃）
server.on("error", (err) => {
  if (err.code === "EADDRINUSE") console.error("❌ 端口被占用");
  else console.error(err);
});

// "close" 事件：整个服务器关闭（不再接受新连接且所有连接结束）时触发
server.on("close", () => console.log("🛑 服务器已完全关闭"));

// 开始监听 9000 端口，启动服务器；成功后会触发上面的 "listening" 事件
server.listen(9000);
```





内核两个队列：

```mermaid
graph LR
    A[客户端 SYN] --> B["半连接队列<br/>SYN queue<br/>(握手进行中)"]
    B -->|三次握手完成| C["全连接队列<br/>accept queue<br/>(等着被取走)"]
    C -->|"accept()"| D["触发 connection 事件"]
```





### server和socket



`server` 是那个"守在端口上、负责接待新连接"的对象。

职责有两块：

1. 干活：`server.listen(port)` 占住端口、接受新连接（这是它的**动作**）。
2. 报告状态：通过事件告诉你它现在怎么样了——`listening`（我开工了）、`connection`（来客人了）、`close`（我收工了）、`error`（出事了，比如端口被占）。



`socket` 代表"服务器和某一个客户端之间的那一条具体连接"——即一条双向的数据通道。

它的特点：

- 一个客户端 = 一个 socket。10 个客户端连进来，就有 10 个独立的 socket，但**始终只有 1 个 server**。
- 真正的数据收发都发生在 socket 上：`socket.write()` 发、`socket.on("data")` 收。
- 它有自己的生命周期：建立 → 收发数据 → 关闭（`end` / `close`）。



`.on` 监听区别

| 对象                       | `.on()` 监听什么       | 常见事件                                                     |
| :------------------------- | :--------------------- | :----------------------------------------------------------- |
| **server**（`net.Server`） | 服务器**整体**的状态   | `listening`（开始监听）、`connection`（有人连进来）、`close`（服务器关了）、`error`（如端口被占用） |
| **socket**（`net.Socket`） | **某一条连接**上的动静 | `data`（**收到数据**）、`end`、`close`、`error`、`timeout`   |





### 创建客户端

三种创建客户端的方式

```js
import net from "node:net";

// 写法 A：传一个 options 对象
const socket = net.createConnection({ host: "127.0.0.1", port: 9000 });

// 写法 B：传 (port, host) 参数
const socket = net.createConnection(9000, "127.0.0.1");

// 写法 C：用 connect（它就是 createConnection 的别名）
const socket = net.connect(9000, "127.0.0.1");
```



`net.createConnection(...)` 本质上就是把下面这两步**打包成一步**

```js
const socket = new net.Socket();   // ① 先创建一个空的 socket 对象
socket.connect(9000, "127.0.0.1"); // ② 再显式发起连接
```



最简示例

```js
const net = require("net");

// 连接服务器；连接成功（三次握手完成）后触发回调
const socket = net.createConnection(9000, "127.0.0.1", () => {
  console.log("已连接到服务器");
  socket.write("你好，服务器");   // 发一条数据
});

socket.on("data", (chunk) => {
  console.log("服务器回复:", chunk.toString());
  socket.end();                   // 收到回复后优雅关闭
});

socket.on("close", () => console.log("连接已关闭"));
socket.on("error", (err) => console.error("出错:", err.message));
```





### socket 的事件与方法

（客户端服务端通用）

```js
// ===== 监听事件 =====
socket.on("connect", () => {});   // 三次握手完成（仅客户端）
socket.on("data",  chunk => {});  // 收到数据，chunk 永远是 Buffer
socket.on("end",   () => {});     // 对方发来 FIN，"我不发了"
socket.on("close", hadErr => {}); // 连接彻底销毁
socket.on("error", err => {});    // 必须监听！否则进程崩溃
socket.on("timeout", () => {});   // 配合 setTimeout 使用，不会自动断开

// ===== socket方法 =====
socket.write(数据);          // 发送
socket.end([数据]);          // 发完剩余数据 → 发 FIN → 优雅关闭
socket.destroy();            // 立即销毁，缓冲区数据直接丢弃
socket.setEncoding("utf8");  // 让 data 直接给字符串（慎用，会切坏中文/协议）
socket.setNoDelay(true);     // 关闭 Nagle 算法，小包立刻发，降低延迟
socket.setKeepAlive(true);   // 开启 TCP 层心跳，探测半死连接
socket.setTimeout(30000);    // 30s 没有任何数据往来则触发 timeout

// ===== 属性 =====
socket.remoteAddress / remotePort   // 对端
socket.localAddress  / localPort    // 本机（客户端端口是内核随机分配的）
socket.bytesRead / bytesWritten
```





### 生命周期



客户端的 `"connect"` ≈ 服务端的 `"connection"`（都代表"这条连接建好了"），只是一个在 socket 上、一个在 server 上。

建好之后，两端**共用同一套 socket 事件/方法**。



客户端 socket 生命周期

```
 net.createConnection(port, host)
          │
          ▼
   [发起 TCP 三次握手]
          │
          ▼
   ┌──────────────┐
   │  "connect"   │  ← 握手完成，可以开始通信
   └──────────────┘
          │
   ┌──────┴───────────────────────┐
   │  socket.write(data)  发数据   │  ← 收发可反复多次
   │  "data"  收到对端数据         │
   └──────┬───────────────────────┘
          │
   我方要关：socket.end()  → 发 FIN
   对方要关：收到对方 FIN → 触发 "end"
          │
          ▼
   ┌──────────────┐
   │   "close"    │  ← 连接彻底销毁（hadError 表示是否因错误关闭）
   └──────────────┘

   ⚡ "error"   任何时刻都可能触发（必须监听，否则进程崩溃）
   ⏱ "timeout" 若 setTimeout 且长时间无数据往来时触发（不会自动断开）
```





服务端生命周期

```
 net.createServer((socket) => { ... })   ← 每来一个连接就 new 一个 socket
          │
   server.listen(port)
          │
          ▼
   ┌──────────────┐
   │ "listening"  │   （server 事件）服务器开始监听端口
   └──────────────┘
          │
   有客户端连进来
          │
          ▼
   ┌──────────────────┐
   │ "connection"     │  （server 事件）回调参数就是这条连接的 socket
   │   → socket       │
   └────────┬─────────┘
            │  从这里往下，就和上面客户端的 socket 生命周期一模一样：
            ▼
     ┌──────────────────────────────┐
     │ socket "data"  收客户端数据    │
     │ socket.write() 回数据          │
     │ socket "end"   客户端发了 FIN  │
     │ socket "close" 这条连接销毁    │
     │ socket "error" 这条连接出错    │
     └──────────────────────────────┘

   （注意：服务端 socket 没有 "connect" 事件，握手完成体现在 "connection" 上）

   ⚡ server "error"  端口被占用等 → 服务器级错误
   🛑 server.close()  停止接收新连接（已有连接不受影响）
```





### TCP



#### TCP 是"字节流"，不是"消息流"

TCP 是**字节流协议**，它只保证数据按顺序到达，**不保证消息边界**

`write` 了 N 次 ≠ 对方触发 N 次 `data` 事件



粘包（Sticky Packet）：多条消息挤在一起

```js
for (let i = 1; i <= 5; i++) {
  client.write(`{"id":${i}}`);
}
```

连续快速写入 5 条小消息（`{"id":1}` ~ `{"id":5}`），由于：

- 消息很小，发送间隔为 0
- 底层 **Nagle 算法**会将小数据合并成一个 TCP 段

服务端很可能只触发 **1 次** `data` 事件，收到的是：

```json
{"id":1}{"id":2}{"id":3}{"id":4}{"id":5}
```



半包（Half Packet）：一条大消息被系统自动拆开

```js
client.write("A".repeat(500 * 1024)); // 写入 500
```

写入一条 500KB 的大消息，由于：

- 超过了 TCP 的 **MSS（最大报文段大小，约 1.5KB）**
- 底层会自动**拆分**成多个 TCP 段分批传输

服务端会触发**多次** `data` 事件，每次只收到一部分 → **半包**





#### 字节流演示代码

```js
// 运行：node net模块/粘包与半包.js
// TCP 是"字节流"，不是"消息流"。write 多少次 ≠ data 触发多少次
import net from "node:net";

const PORT = 9003;

const server = net.createServer((socket) => {
  let n = 0;
  socket.on("data", (chunk) => {
    n++;
    const preview = chunk.toString("utf-8").slice(0, 50);
    console.log(`[server] 第 ${n} 次 data 事件 | 长度 ${chunk.length} | 内容 ${JSON.stringify(preview)}...`);
  });
  socket.on("close", () => {
    console.log(`\n[server] 客户端一共 write 了 6 次，服务器却只触发了 ${n} 次 data 事件`);
    console.log("[server] 说明：TCP 只保证【字节顺序】，不保证【消息边界】");
    server.close();
  });
});

server.listen(PORT, () => {
  const client = net.createConnection(PORT, "127.0.0.1", () => {
    console.log("=== 场景一：连续 write 5 条小消息 → 大概率被【粘】成 1 次 data ===");
    for (let i = 1; i <= 5; i++) {
      client.write(`{"id":${i}}`);
    }

    setTimeout(() => {
      console.log("\n=== 场景二：write 1 条 500KB 大消息 → 被【拆】成很多次 data ===");
      client.write("A".repeat(500 * 1024));
      setTimeout(() => client.end(), 800);
    }, 500);
  });
});
```

输出结果：

```
[server] 第 1 次 data 事件 | 长度 40 | 内容 "{\"id\":1}{\"id\":2}{\"id\":3}{\"id\":4}{\"id\":5}"...

=== 场景二：write 1 条 500KB 大消息 → 被【拆】成很多次 data ===
[server] 第 2 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 3 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 4 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 5 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 6 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 7 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 8 次 data 事件 | 长度 65536 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...
[server] 第 9 次 data 事件 | 长度 53248 | 内容 "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"...

[server] 客户端一共 write 了 6 次，服务器却只触发了 9 次 data 事件
[server] 说明：TCP 只保证【字节顺序】，不保证【消息边界】
```





#### 长度前缀协议

然 TCP 不给边界，就自己在字节流上划边界



约定每条消息的格式为：

```
┌───────────────┬─────────────────────┐
│  4 字节长度头  │   JSON 负载 (body)  │
│  (大端 UInt32) │   长度 = 长度头的值  │
└───────────────┴─────────────────────┘
```

只要**先读长度头，就知道 body 有多长**，从而精确切分每条消息的边界。



封包 `encode`

```js
function encode(obj) {
  const body = Buffer.from(JSON.stringify(obj), "utf-8");
  const head = Buffer.alloc(4);
  head.writeUInt32BE(body.length, 0); // 写入 body 字节长度
  return Buffer.concat([head, body]);
}
```

- `body`：把对象序列化成 JSON 字节
- `head`：分配 4 字节，写入 **body 的字节长度**
- `writeUInt32BE`：**BE = Big Endian（大端）**，即"网络字节序"，跨平台通信的统一标准
- 最终返回：`[长度头][body]` 拼接的完整数据包

> UInt32 最大可表示约 4GB，用 4 字节足够描述任意消息长度。



解包 `createDecoder`

通过**内部缓冲区 + 循环**同时处理粘包和半包。

```js
function createDecoder(onMessage) {
  let cache = Buffer.alloc(0);  // 累积缓冲区（闭包保存）

  return function (chunk) {
    cache = Buffer.concat([cache, chunk]);  // 新数据追加到缓冲区

    while (true) {
      if (cache.length < 4) return;          // ① 半包：长度头都没收全
      const len = cache.readUInt32BE(0);
      if (cache.length < 4 + len) return;    // ② 半包：body 没收全

      const body = cache.subarray(4, 4 + len);
      cache = cache.subarray(4 + len);       // ③ 移除已处理部分，保留剩余
      onMessage(JSON.parse(body.toString("utf-8")));
      // 循环继续 → 处理粘在后面的下一条消息
    }
  };
}
```



`cache.readUInt32BE(0)`从 Buffer 的**第 0 个字节开始，读取 4 个字节，解释成一个无符号整数**。

- `read`：读
- `UInt32`：4 字节无符号整数（对应写入时的 `writeUInt32`）
- `BE`：大端（对应写入时的 `BE`）
- `(0)`：从下标 0 开始读



 `cache.subarray(4 + len)` —— 切掉已处理的数据

```js
cache = cache.subarray(4 + len);
```

- `4`：长度头占 4 字节
- `len`：body 占 len 字节
- `4 + len`：一条完整消息的总长度
- `subarray(4 + len)`：从这条消息**结束的位置**开始，取到末尾





##### 解包逻辑直观展示

假设要发两条消息（为了简单，body 用假想的短内容）：

- 消息 A：长度头 = `3`，body = `"aaa"`（3 字节）→ 完整包 7 字节
- 消息 B：长度头 = `2`，body = `"bb"`（2 字节）→ 完整包 6 字节

用 `[3]` 表示"值为3的4字节长度头"来简化画图。



**情况一：粘包**（两条消息一次性到达）

`data` 事件收到的 `chunk`，拼进 cache 后长这样（共 13 字节）

```
cache: [3]aaa[2]bb
下标:   012 345 6789...
```

进入 `while(true)`：

**第 1 轮循环**

```js
cache.length < 4 ?  →  13 < 4 ? 否，继续
len = cache.readUInt32BE(0)  →  读出 3
cache.length < 4 + len ?  →  13 < 7 ? 否，说明 body 齐了
body = cache.subarray(4, 7)  →  取出 "aaa"
cache = cache.subarray(7)  →  cache 变成 [2]bb（剩 6 字节）
onMessage("aaa")  ✅ 解析出消息 A
```

此时 `cache` 只剩：`[2]bb`，**循环不退出，继续下一轮**。

**第 2 轮循环**

```js
cache.length < 4 ?  →  6 < 4 ? 否
len = cache.readUInt32BE(0)  →  读出 2
cache.length < 4 + len ?  →  6 < 6 ? 否，body 齐了
body = cache.subarray(4, 6)  →  取出 "bb"
cache = cache.subarray(6)  →  cache 变成空（0 字节）
onMessage("bb")  ✅ 解析出消息 B
```

此时 `cache` 长度为 0。

**第 3 轮循环**

```js
cache.length < 4 ?  →  0 < 4 ? 是，return 退出
```



**情况二：半包**（一条消息分两次到达）

现在消息 A 被 TCP 拆开了，分两次 `data` 事件到达。

**第一次 data**，只到了一半（4 字节）：

```js
cache: [3]a          ← 长度头齐了，但 body 只到了 "a"
```

进入 `while`：

```js
cache.length < 4 ?  →  4 < 4 ? 否
len = cache.readUInt32BE(0)  →  读出 3
cache.length < 4 + len ?  →  4 < 7 ? 是！body 不齐
return  ← 退出循环，什么都不解析
```

> **cache 没有被清空**，`[3]a` 完整地保留在缓冲区里，等下一次数据。

**第二次 data**，剩余部分到了（`aa`）：

```js
cache = Buffer.concat([cache, chunk])
// cache 从 [3]a 变成 [3]aaa  ← 现在齐了！
```

进入 `while`：

```js
cache.length < 4 ?  →  7 < 4 ? 否
len = readUInt32BE(0)  →  读出 3
cache.length < 4 + len ?  →  7 < 7 ? 否，body 齐了！
body = subarray(4, 7)  →  "aaa"
cache = subarray(7)  →  空
onMessage("aaa")  ✅ 解析出消息 A
```



总结

```
每收到一批新数据，就反复尝试从缓冲区头部"啃"出完整消息：

  能啃出一条完整消息  →  解析它，从缓冲区删掉，继续啃下一条  （治粘包）
  数据不够啃一条完整的 →  return 退出，把半截留在缓冲区等下次  （治半包）
```









## http模块



### net缺陷

net模块有net.createServer` / `net.connect` / `socket.write` / `'data'` 事件等方式创建服务器，并从客户端接收数据

但是TCP能提供的能力用一句话概括：**在两台机器之间可靠地、按顺序地搬运一串字节。**



TCP提供不了的东西：

1. 确保消息边界，TCP是字节流，没法准确的把每个消息单独拆开来归为一个数据包
2. 定不了消息格式， 字节里哪段是"我要 /user 这个资源"，哪段是数据？没人规定
3. 连接的生命周期，这个 socket 用完关掉？还是留着复用？复用的话下一个请求怎么和上一个区分？





### http模块是什么

HTTP就是在TCP之上补的一套文本协议，规定了消息格式和边界。

Node 的 http模块 = net 模块 + HTTP协议的编解码器



编解码器 = 编码器（Encoder）+ 解码器（Decoder），负责在两种数据格式之间互相转换。

```
JS 对象/字符串
        ↕  ← 编解码器做这个转换
原始字节流（TCP 传输）
```





### 手写http



#### net构建http服务器

```js
import net from 'node:net';

const server = net.createServer((socket) => {
  socket.on('data', (chunk) => {
    console.log(JSON.stringify(chunk.toString()));  

    const body = 'hello from raw net';

    // 手动拼接 HTTP 响应报文

    socket.write(
      'HTTP/1.1 200 OK\r\n' +
      'Content-Type: text/plain; charset=utf-8\r\n' +
      `Content-Length: ${Buffer.byteLength(body)}\r\n` +
      'Connection: close\r\n' +
      '\r\n' +
      body
    );
    socket.end();
  });
});

server.listen(3000, () => console.log('net server → http://127.0.0.1:3000'));
```

解释

```js
const server = net.createServer((socket) => { ... });
server.listen(3000, ...);
```

`net.createServer` 监听 TCP 端口，每当有客户端连接进来，就触发回调，拿到一个 `socket` 对象。此时**完全不知道 HTTP 的存在**，只是一个 TCP 连接。

接收原始字节并打印

```js
socket.on('data', (chunk) => {
  console.log(JSON.stringify(chunk.toString()));
});
```

手动拼接HTTP响应报文

```js
socket.write(
  'HTTP/1.1 200 OK\r\n' +                              // 状态行
  'Content-Type: text/plain; charset=utf-8\r\n' +      // 响应头
  `Content-Length: ${Buffer.byteLength(body)}\r\n` +   // 响应头
  'Connection: close\r\n' +                            // 响应头
  '\r\n' +                                             // 空行：头部结束
  body                                                 // 响应体
);
socket.end(); // 主动关闭连接
```

这就是 HTTP 的真身 —— **一段纯文本**。浏览器也能打开 `http://127.0.0.1:3000`，因为手拼的响应完全合法。

`Buffer.byteLength(body)`方法：返回字符串在使用指定编码进行编码后的字节长度





#### 客户端发送http请求

用**原始 TCP socket** 手动构造并发送一个 HTTP/1.1 请求

```js
import net from 'node:net';

// 1. 建立 TCP 连接
const socket = net.connect(3000, '127.0.0.1', () => {
  // 2. 连接建立后，手动写入原始 HTTP 报文
  socket.write(
    'GET /hello HTTP/1.1\r\n' +   // 请求行
    'Host: 127.0.0.1:3000\r\n' +  // 请求头
    'Connection: close\r\n' +      // 请求头：告诉服务端响应后关闭连接
    '\r\n'                         // 空行：头部结束标志（关键！）
  );
});

// 3. 接收响应数据，拼接到 raw
let raw = '';
socket.on('data', (c) => (raw += c));

// 4. 连接关闭后，打印完整原始响应
socket.on('end', () => {
  console.log(JSON.stringify(raw));
});
```







### HTTP报文

HTTP报文，就是HTTP通信中传输的数据块，本质上是一段按照固定格式组织的纯文本字符串

请求报文：

```
GET /hello HTTP/1.1\r\n        ← 请求行（方法 + 路径 + 版本）
Host: 127.0.0.1:3000\r\n       ← Header
Connection: close\r\n           ← Header
\r\n                            ← 空行（Header 结束标志）
                                ← Body（GET 请求通常没有）
```

响应报文：

```
HTTP/1.1 200 OK\r\n             ← 状态行（版本 + 状态码 + 描述）
Content-Type: text/plain\r\n    ← Header
Content-Length: 18\r\n          ← Header
Connection: close\r\n           ← Header
\r\n                            ← 空行（Header 结束标志）
hello from raw net              ← Body
```





### http确认消息边界的方式

http确认body边界的方式



方式一：`Content-Length`

```
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 11          ← 告诉你：body 正好 11 字节

Hello World                 ← 正好 11 字节（含换行的话要算进去）
```

Node.js 示例：

```js
const http = require('http');

http.createServer((req, res) => {
  const body = 'Hello World';           // 11 字节
  res.writeHead(200, {
    'Content-Type': 'text/plain',
    'Content-Length': Buffer.byteLength(body)  // 必须精确！
  });
  res.end(body);
}).listen(3000);
```



方式二：`Transfer-Encoding: chunked`（分块传输）

原理：发送方事先不知道总长度，把body拆成若干块，每块前面用十六进制数自保文本块长度，最后发一个长度为0的块表示结束

```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

6\r\n          ← 第一块：6 字节
Hello \r\n
5\r\n          ← 第二块：5 字节
World\r\n
0\r\n          ← 结束标志：长度为 0
\r\n
```

Node.js 示例：

```js
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  // 不设置 Content-Length，Node 会自动使用 chunked

  res.write('Hello ');   // 第一块
  
  setTimeout(() => {
    res.write('World');  // 第二块（模拟异步数据，比如数据库查询结果）
    res.end();           // 发送 0 长度块，结束
  }, 1000);
}).listen(3000);
```

用于式响应（比如边查数据库边输出、SSE、大文件），因为这些操作在发头的时候还不知道总长度。

这也是`res.write()` 和 `res.end()` 分开存在的原因。





### 创建http服务器



#### 完整示例

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // req：读取请求信息
  console.log(req.method, req.url);

  // res：构造并发送响应
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World');
});

// 绑定3000端口，进入监听状态
server.listen(3000, () => {
  console.log('服务器运行在 http://localhost:3000');
});
```

`server.listen(3000, callback)`：将服务器绑定到本机的 **3000 端口**，服务器进入**监听状态**，等待客户端（浏览器、curl 等）发起 HTTP 请求。一旦有请求进来，就触发 `createServer` 中定义的回调函数 `(req, res) => { ... }`。

callback仅仅在服务器成功启动并开始监听后执行一次





#### 语法

```js
import http from 'node:http';

const server = http.createServer((req, res) => { /* 每来一个请求跑一次 */ });
server.listen(3000, () => console.log('ok'));
```



`http.createServer()` 

语法：

```js
const server = http.createServer([options][, requestListener])
```

作用：创建一个 HTTP 服务器实例（`http.Server` 对象）。服务器创建后需调用 `.listen()` 才会开始监听端口、接收请求。



`options`（可选，对象）

用于配置服务器行为：

| 字段               | 类型    | 说明                                            |
| :----------------- | :------ | :---------------------------------------------- |
| `IncomingMessage`  | Class   | 自定义请求对象的类，默认 `http.IncomingMessage` |
| `ServerResponse`   | Class   | 自定义响应对象的类，默认 `http.ServerResponse`  |
| `keepAlive`        | Boolean | 是否保持长连接，默认 `false`                    |
| `keepAliveTimeout` | Number  | 长连接空闲超时时间（毫秒），默认 `5000`         |
| `requestTimeout`   | Number  | 请求超时时间（毫秒），默认 `300000`（5分钟）    |



`requestListener`（可选，函数）

这是核心参数，每当有新请求进来的时候就会自动调用，该函数接收两个参数：

```js
function requestListener(req, res) { ... }
```

**`req`（`http.IncomingMessage` 实例）** — 代表客户端发来的请求：

```js
req.method      // 请求方法：'GET' / 'POST' ...
req.url         // 请求路径：'/index.html?a=1'
req.headers     // 请求头对象：{ 'content-type': 'application/json', ... }
req.socket      // 底层 TCP socket
// body 数据需要监听 data 事件读取（req 是可读流）
```

读取req信息示例：

```js
// 06-req-anatomy.js
import http from 'node:http';

http.createServer((req, res) => {
  console.log('method      :', req.method);        // 'POST'
  console.log('url         :', req.url);           // '/api/user?id=1'  ← 只有路径+查询，没有域名！
  console.log('httpVersion :', req.httpVersion);   // '1.1'
  console.log('headers     :', req.headers);       // 键全部小写
  console.log('rawHeaders  :', req.rawHeaders);    // ['Host','...','User-Agent','...'] 原样、扁平数组

  // 要完整 URL 得自己拼
  const full = new URL(req.url, `http://${req.headers.host}`);
  console.log('pathname    :', full.pathname);
  console.log('query id    :', full.searchParams.get('id'));

  res.end('ok');
}).listen(3000);
```



**`res`（`http.ServerResponse` 实例）** — 代表要返回给客户端的响应：

```js
res.statusCode = 200          // 设置状态码
res.setHeader(name, value)    // 设置响应头
res.writeHead(statusCode, headers)  // 一次性设置状态码 + 响应头
res.write(chunk)              // 写入响应体（可多次调用）
res.end([data])               // 结束响应，可顺带写最后一段数据
```

res构建示例

```js
import http from 'node:http';

http.createServer((req, res) => {
  // 方式一：分步设置（可反复改，直到头被发出）
  res.statusCode = 201;
  res.statusMessage = 'Created';
  res.setHeader('Content-Type', 'application/json; charset=utf-8');
  res.setHeader('X-Powered-By', 'me');
  res.removeHeader('X-Powered-By');
  console.log(res.getHeader('content-type'));  // 发出前可读回
  console.log(res.headersSent);                // false

  // 方式二：一次性写头（等价，且更快）
  // res.writeHead(201, { 'Content-Type': 'application/json' });

  res.write('{"a":1');   // ← 第一次 write：头部在此刻真正被发送到 socket
  console.log(res.headersSent);  // true

  // res.setHeader('X-Late', '1');  // ❌ 此刻会抛 ERR_HTTP_HEADERS_SENT

  res.end('}');          // 必须调用！否则客户端一直挂着等
}).listen(3000);
```





#### 一个请求进来时发生了什么

```mermaid
sequenceDiagram
    participant C as 客户端
    participant N as net.Server（http.Server 继承它）
    participant P as llhttp 解析器（C++）
    participant J as 你的 requestListener

    C->>N: TCP 三次握手
    N->>N: emit 'connection'(socket)
    N->>P: 给这个 socket 挂一个解析器
    C->>N: 发字节 "GET /a HTTP/1.1\r\nHost:...\r\n\r\n"
    N->>P: socket 'data' → parser.execute(chunk)
    P->>P: 解析出方法/路径/头
    P->>N: onHeadersComplete 回调
    N->>N: 创建 IncomingMessage(req) + ServerResponse(res)
    N->>J: emit 'request'(req, res)  ← 你的函数在这里被调用
    P->>N: onBody（body 分块到达）
    N->>J: req.emit('data', chunk)
    P->>N: onMessageComplete
    N->>J: req.emit('end')
```



1、TCP 三次握手 + `emit 'connection'`

- 客户端和服务器线完成 TCP 三次握手，建立一条连接
- 这时 HTTP 层还完全没参与。`net.Server` 接受连接后，创建一个 `socket`（`net.Socket` 对象），并触发 `'connection'` 事件。



2、给 socket 挂一个解析器

- `http.Server` 在 connection 事件里，为这条 socket 关联一个 **llhttp 解析器实例**。
- llhttp 是用 C 写的 HTTP 解析器，性能高。它本身**不读 socket**，只负责「喂给socket字节，它吐出解析事件」



3、字节到达 → `parser.execute(chunk)`

- 客户端把 HTTP 请求文本当作普通 TCP 字节发过来
- socket 收到数据触发底层的数据回调，Node 把这块 `chunk` 交给 `parser.execute(chunk)`



4、解析出方法/路径/头

```
P->>P: 解析出方法/路径/头
P->>N: onHeadersComplete 回调
```

- llhttp 从字节里识别出请求行（`GET /a HTTP/1.1`）和请求头（`Host: ...`）。
- 当解析到头部结束的空行（`\r\n\r\n`）时，触发 **`onHeadersComplete`** 回调，通知 JS 层：「头已经齐了」。



5、创建 req/res 并 emit 'request'

```
N->>N: 创建 IncomingMessage(req) + ServerResponse(res)
N->>J: emit 'request'(req, res)  ← 你的函数在这里被调用
```

- 有了完整的头信息，Node 就能构造两个对象：

  - **`IncomingMessage`（req）**：可读流，代表进来的请求（含 method、url、headers，body 稍后以流的形式给到）。
  - **`ServerResponse`（res）**：可写流，往它写就是往 socket 写响应。

- 然后触发`'request'`事件，也就是你写的：

  ```js
  http.createServer((req, res) => { /* 代码在这里跑 */ })
  ```



6、 body 分块到达

```
P->>N: onBody（body 分块到达）
N->>J: req.emit('data', chunk)
```

- 如果请求有 body（比如 POST），llhttp 每解析出一段 body，就通过 `onBody` 交给 JS。

- Node 把它作为 `req` 这个可读流的 `'data'` 事件推到`requestListener`函数这。在`createServer`中如此读取 body ：

  ```js
  let body = '';
  req.on('data', chunk => body += chunk);
  req.on('end', () => { /* body 收齐了 */ });
  ```



7、请求结束

```
P->>N: onMessageComplete
N->>J: req.emit('end')
```

- llhttp 判断整个消息读完（根据 `Content-Length` 或 chunked 编码），触发 `onMessageComplete`。
- Node 在 `req` 上触发 `'end'` 事件，通知「这个请求的 body 全部收完了」。





### 创建http客户端



#### 完整示例

```js
// 11-client.js
import http from 'node:http';

const req = http.request(
  'http://127.0.0.1:3000/api/user?id=1',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json; charset=utf-8' },
  },
  (res) => {                                  // 读取响应
    console.log('status :', res.statusCode);  // 200
    console.log('headers:', res.headers);

    res.setEncoding('utf8');                  // 把 Buffer 按 utf8 解码成字符串,避免多字节字符被截断
    let body = '';
    res.on('data', (c) => (body += c));       // body 分块到达,逐块拼接
    res.on('end', () => console.log('body   :', body));  // // 收齐后打印
  }
);

// 监听请求层面的错误
req.on('error', (err) => console.error('请求失败:', err.message));

req.write(JSON.stringify({ name: 'node' }));  // 往请求体body里写数据
req.end();                                    // 声明 body 写完,真正发出请求
```

整体时序

```
http.request() 建对象 → req.write() 写 body → req.end() 发送
        ↓
服务器处理并返回
        ↓
'response' 事件 → 回调拿到 res → res 'data'(分块) → res 'end'(读完)
```



#### 语法

```js
http.get(url, res => { res.on('data', ...); res.on('end', ...) });   // 无 body
const r = http.request(opts, res => {...}); r.write(body); r.end();  // 有 body
```



```js
http.request(url, options, callback)
```

参数:

- **url**:`'http://127.0.0.1:3000/api/user?id=1'` — 目标地址,带查询参数 `id=1`。
- **options**:请求配置 — `method: 'POST'`,以及请求头 `Content-Type: application/json`。
- **callback `(res) => {...}`**:这是 **`'response'` 事件的监听器**,当服务器返回响应、**响应头到达时**被调用(注意此刻 body 还没读完)。





### 流事件

| 事件     | 哪类流   | 触发时机                     | 现实现象                                                   |
| -------- | -------- | ---------------------------- | ---------------------------------------------------------- |
| `data`   | Readable | 有新数据块可读               | body 大时触发多次，小时触发一次                            |
| `end`    | Readable | 数据已读完                   | body 收完的信号                                            |
| `error`  | 两者     | 出错                         | 不监听则进程崩溃                                           |
| `close`  | 两者     | 底层连接关闭                 | 浏览器关掉 tab 时触发；`server.js` 里的流 timer 用它做清理 |
| `drain`  | Writable | 缓冲区排空，可以继续写       | `res.write()` 返回 `false` 时等这个信号，背压控制          |
| `finish` | Writable | `end()` 调用后，数据全部写出 | `res.end()` 之后触发                                       |



`data` — Readable：有数据块到了

```js
req.on('data', (chunk) => {
  // chunk 是 Buffer，不是字符串。中文按 utf8 占 3 字节，
  // 直接 chunk.toString() 可能把一个汉字劈成两半 → 乱码
  chunks.push(chunk);
});
```

**运行现象**：body 1KB 时触发 1 次；body 10MB 时触发几十~几百次，每次约 64KB。

一旦注册了 `data` 监听器，流立刻从**暂停模式**切到**流动模式**开始送数据——所以监听器要在同步代码里注册



`end` — Readable：数据读完了

```js
req.on('end', () => {
  // 此刻才有完整 body。这是唯一能安全解析 JSON 的时机
  const body = Buffer.concat(chunks).toString('utf8');
  const data = JSON.parse(body);
});
```

**运行现象**：GET 请求没有 body，`data` 一次都不触发，但 `end` **照样触发**。
**判定依据**：HTTP 层看 `Content-Length` 读够了，或分块编码收到 `0\r\n\r\n` 终止块。



`error` — 两者都有：出错了

```js
req.on('error', (err) => {
  // 不监听 = 未捕获异常 = 整个 Node 进程直接挂掉
  // 常见原因：客户端上传到一半关掉浏览器
  console.error('请求流出错', err.code);
});

res.on('error', (err) => {
  // 常见 err.code：EPIPE（对端已断开你还在写）
});
```



`close` — 两者都有：底层连接彻底关了

```js
res.on('close', () => {
  // 无论正常结束还是客户端提前断开，都会触发 → 天然的清理钩子
  clearInterval(timer);       // 停掉定时器，否则往死掉的连接里 write
  db.cancelQuery();           // 取消还在跑的数据库查询，省资源
});
```



`drain` — Writable：缓冲区排空了，可以继续写

```js
function writeLots() {
  let i = 0;
  function step() {
    while (i < 100000) {
      const ok = res.write(`第 ${i++} 行\n`);
      // write() 返回 false = 内核发送缓冲区满了（客户端网速跟不上）
      // 此时数据全堆在 Node 内存里 → 继续硬写会内存暴涨
      if (!ok) {
        res.once('drain', step);  // 等排空信号，再从断点继续
        return;                   // 必须 return，让出执行权
      }
    }
    res.end();
  }
  step();
}
```



`finish` — Writable：`end()` 的数据全部交给内核了

```
res.on('finish', () => {
  // 响应完整写出。适合记访问日志、埋点、统计耗时
  console.log(`${req.method} ${req.url} → ${res.statusCode} 耗时 ${Date.now() - t0}ms`);
});
```

- `finish` = **我这边写完了**（业务成功）
- 只有 `finish` 触发过，才说明响应是完整的。





### req / res 参数背后类

|        | ** 我写出去的（可写流）**                         | **我读进来的（可读流）**                         |
| ------ | ------------------------------------------------- | ------------------------------------------------ |
| 客户端 | `ClientRequest`<br>`http.request()` 的返回值`req` | `IncomingMessage`<br>callback 里的 `res`         |
| 服务器 | `ServerResponse`<br>`requestListener`里的 `res`   | `IncomingMessage`<br>`requestListener`里的 `req` |



`OutgoingMessage` 是"我要发的报文"的共同父类，`IncomingMessage` 是"我收到的报文"，底下永远是同一条 `net.Socket`。

类继承图

```mermaid
graph TD
    S[stream.Readable] --> IM["http.IncomingMessage<br/>服务端的 req / 客户端的 res"]
    ST[stream.Stream] --> OM["http.OutgoingMessage<br/>实现 write/end/cork"]
    OM --> CR["http.ClientRequest<br/>客户端的 req"]
    OM --> SR["http.ServerResponse<br/>服务端的 res"]
    NS[net.Server] --> HS[http.Server]
    HS -."每个连接持有".-> SOCK[net.Socket]
    IM -."req.socket".-> SOCK
    SR -."res.socket".-> SOCK
```

虚线表示了，一条socket承载了多对req和res的传输







## https加密原理

全程由三位角色出演：

| 角色   | 身份                          | 动机                           |
| :----- | :---------------------------- | :----------------------------- |
| 邓哥   | 客户端 / 浏览器               | 想跟摸尼卡安全地说话           |
| 摸尼卡 | 服务器 / 网站                 | 想安全地接待邓哥               |
| 成哥   | 中间人（*Man-in-the-Middle*） | 想**看到**并**改写**他俩的对话 |





### 一、问题的起点：明文在裸奔

邓哥给摸尼卡发了一句「我爱你」，中间人成哥把它改成了「我恨你」。

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥（客户端）
    participant C as 成哥（中间人）
    participant M as 摸尼卡（服务器）

    D->>C: 我爱你（明文）
    Note over C: 全程可读、可改、可冒充
    C->>M: 我恨你（明文）
```

由此引出安全通信的三个目标，后面所有的技术都是在逐个解决它们：

| 目标         | 含义                       | 解决手段    |
| :----------- | :------------------------- | :---------- |
| **保密性**   | 别人看不懂                 | 加密        |
| **完整性**   | 别人改不了（改了能被发现） | 签名 / 摘要 |
| **身份认证** | 对面确实是摸尼卡本人       | 证书 / *CA* |





### 二、对称加密

> **对称加密**：产生一个密钥，可以用其加密，也可以用其解密。

```mermaid
flowchart LR
    A["明文：我爱你"] -- "加密 + key" --> B["密文：一堆乱码"]
    B -- "解密 + 同一把 key" --> C["明文：我爱你"]
```



致命伤：密钥怎么送过去？

邓哥和摸尼卡素未谋面，那把 `key1` 本身也得走网络。而这条网络上坐着成哥。

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥
    participant C as 成哥（中间人）
    participant M as 摸尼卡

    M->>C: 我们使用 key1 通信
    Note over C: key1 到手
    C->>D: 我们使用 key1 通信（原样转发，不露痕迹）

    D->>C: key1 加密( 我爱你 )
    Note over C: 用 key1 解密 → 篡改 → 用 key1 重新加密
    C->>M: key1 加密( 我恨你 )
```

**结论：对称加密解决了「怎么加密」，没解决「怎么把钥匙安全地交出去」。**





### 三、非对称加密

> **非对称加密**：产生一对密钥，一个用于加密（公钥），一个用于解密（私钥）。

```mermaid
flowchart LR
    K["生成密钥对"] --> P["公钥 public key<br/>随便公开"]
    K --> S["私钥 private key<br/>死也不给别人"]
    M1["明文"] -- "用 公钥 加密" --> CT["密文"]
    CT -- "只能用 私钥 解密" --> M2["明文"]
```

关键性质：**公钥加密的东西，只有配对的私钥能解开**。所以公钥被成哥看见完全没关系 —— 他能加密，但解不开。



于是有了实际方案 —— 用非对称密钥去传递对称密钥：**把对称密钥 `key2` 安全地送过去**。之后全部用 `key2` 通信。

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥（客户端）
    participant M as 摸尼卡（服务器）

    Note over M: 生成密钥对：公钥key1 / 私钥key1
    M->>D: 公钥key1（明文传输，被看到也无所谓）

    Note over D: 随机生成一把对称密钥 key2
    D->>M: 公钥key1 加密( key2 )
    Note over M: 用 私钥key1 解密，取出 key2

    Note over D,M: 双方现在都持有 key2，且只有他俩有
    D->>M: key2 加密( 我爱你 )
    M->>D: key2 加密( 你是个好人 )
```





### 四、还是被攻破：中间人换公钥

问题出在第 2 步 —— 邓哥收到一把公钥，**他凭什么相信这把公钥是摸尼卡的？**

成哥自己也生成一对 `公钥key3 / 私钥key3`，然后做了个「掉包」：

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥
    participant C as 成哥（中间人）
    participant M as 摸尼卡

    M->>C: 公钥key1
    Note over C: 扣下 key1，塞进自己兜里<br/>换成自己的 公钥key3 发出去
    C->>D: 公钥key3（冒充摸尼卡）

    D->>C: 公钥key3 加密( key2 )
    Note over C: 用 私钥key3 解密 → 拿到 key2
    C->>M: 公钥key1 加密( key2 )

    Note over D,M: 两边都以为握手成功了<br/>实际上 key2 在成哥手里

    D->>C: key2 加密( 我爱你 )
    Note over C: 解密 → 篡改 → 重新加密
    C->>M: key2 加密( 我恨你 )
```

邓哥这边只跟成哥建了一条连接，摸尼卡那边也只跟成哥建了一条连接，成哥在中间做了个「二房东」。两端**毫无感知**。

**结论：加密算法本身没有问题，问题是「公钥不能自证身份」。**

缺的不是更强的加密，而是一个**可信的第三方**来担保「这把公钥确实属于 www.monica.com」。







### 五、CA：Certificate Authority 证书颁发机构

思路很像现实世界的身份证：邓哥不认识摸尼卡，但邓哥**信任公安局**；公安局给摸尼卡发了张身份证，邓哥就信了。

*CA* 就是网络世界的公安局，它自己也有一对 `CA公钥 / CA私钥`。



证书颁发流程

```mermaid
sequenceDiagram
    autonumber
    participant M as 摸尼卡（网站）
    participant CA as CA 证书颁发机构

    Note over M: 自己生成 公钥key1 / 私钥key1<br/>（私钥永不外传，CA 也拿不到）
    M->>CA: 提交 域名 www.monica.com + 公钥key1 + 费用
    Note over CA: 核验：这个域名确实归你所有吗？<br/>核验通过后，用 CA私钥 对证书内容签名
    CA->>M: 颁发数字证书 Digital Certificate
```



证书里有什么

```mermaid
flowchart TB
    subgraph DC["数字证书 Digital Certificate (DC)"]
        direction TB
        A["域名：www.monica.com"]
        B["颁发机构：某 CA"]
        C["服务器公钥：公钥key1"]
        D["证书签名 Signature<br/>由 CA私钥 生成"]
    end
```



 证书签名 Signature

签名的两个性质是整个信任体系的地基：

1. **算法公开** —— 任何人都能把证书内容重新算一遍，得到一个签名值；
2. **只有 CA 能生成合法签名** —— 因为生成签名的最后一步要用 **CA私钥** 加密，而 *CA私钥* 只有 *CA* 有。

于是形成了一个漂亮的不对称局面：**人人可验证，唯 CA 可签发。**

补充：

- 签名 = `CA私钥` 对「证书内容的**哈希摘要**」进行加密，而不是直接加密整段内容；
- 证书里的**域名、服务器公钥都是明文**，任何人可直接读取，被加密的只有那段签名；
- 验证过程是：`CA公钥` 解开签名得到摘要 A，本地对证书内容重新做哈希得到摘要 B，比对 `A == B`。

证书里的任何东西在中途发生了改变，都无法完成验证





### 六、浏览器获取并验证证书

现在把握手的第一步换成「发证书」而不是「发裸公钥」。

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥（浏览器）
    participant M as 摸尼卡（服务器）

    D->>M: 我要访问 www.monica.com
    M->>D: 数字证书（域名 + 颁发机构 + 公钥key1 + 证书签名）

    Note over D: 操作系统/浏览器内置了受信任 CA 的公钥列表
    Note over D: ① 用 CA公钥 解开证书签名<br/>② 用公开算法把证书内容重算一遍签名<br/>③ 比对两者是否一致

    alt 一致，且域名与访问地址相符、证书未过期
        D->>D: 验证通过：公钥key1 确实属于 www.monica.com
    else 任一项不符
        D->>D: 拦截并告警：您的连接不是私密连接
    end
```





#### 为什么中间人改不动

成哥想把证书里的公钥换成自己的 `公钥key3`，或者把域名改成 `www.abc.com`：

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥（浏览器）
    participant C as 成哥（中间人）
    participant M as 摸尼卡

    M->>C: 证书（www.monica.com + 公钥key1 + 签名）
    Note over C: 想改内容，就必须重算签名<br/>重算签名必须用 CA私钥<br/>成哥没有 CA私钥
    C->>D: 篡改后的证书（www.abc.com + 公钥key3 + 旧签名）

    Note over D: 重算出的签名 ≠ 证书里附带的签名
    D->>D: 验证失败，连接终止
```

成哥面前只剩两条死路：

- 改内容不改签名 → 签名对不上，验证失败；
- 改内容并重算签名 → 需要 *CA私钥*，他没有。

那他直接**原样转发**摸尼卡的真证书行不行？可以，但那样他手里就没有 `私钥key1`，后面邓哥用 `公钥key1` 加密的 `key2` 他根本解不开 —— 攻击链在这里断掉了。





### 七、后续：回到混合加密

证书验证通过后，`公钥key1` 的归属已经可信了，剩下的流程和第三章完全一样：

```mermaid
sequenceDiagram
    autonumber
    participant D as 邓哥（浏览器）
    participant C as 成哥（中间人）
    participant M as 摸尼卡（服务器）

    Note over D,M: 证书已验证通过，公钥key1 可信

    D->>C: 公钥key1 加密( key2 )
    Note over C: 只有 公钥key1，没有 私钥key1<br/>解不开，看不见 key2
    C->>M: 原样转发（改了也只会解密失败）
    Note over M: 用 私钥key1 解出 key2

    D->>C: key2 加密( 我爱你 )
    C->>M: 看不懂，也改不了
    M->>D: key2 加密( 你是个好人 )
```



至此三个目标全部达成：

| 目标     | 由谁保证                                       |
| :------- | :--------------------------------------------- |
| 保密性   | 对称密钥 `key2`（`key2` 由非对称加密安全送达） |
| 身份认证 | *CA* 签发的数字证书                            |
| 完整性   | 证书签名 + 传输阶段的消息认证码                |





### 八、https 协议

*https* 并不是一个全新的协议，就是http的基础上，配上了「证书 + 混合加密」

HTTPS = HTTP + TLS 层，TLS 用证书完成身份认证、用（EC）DHE 协商出会话密钥、用对称加密传输数据。

TLS 是 Transport Layer Security（传输层安全协议）的缩写

```mermaid
flowchart TB
    subgraph HTTP["http 协议"]
        direction TB
        h1["http"] --> h2["TCP/IP"]
    end
    subgraph HTTPS["https 协议"]
        direction TB
        s1["https"] --> s2["SSL / TLS"] --> s3["TCP/IP"]
    end
```

上层的 *http* 报文写法一模一样，只是交给下层之前先被 *SSL* 层加了密。所以 *http* 的一切知识（方法、状态码、请求头）在 *https* 中完全通用。



TLS 是 Transport Layer Security（传输层安全协议）的缩写,是 SSL（Secure Sockets Layer，安全套接层）的继任者







## https模块实践



### 环境准备

安装OpenSSL软件，用于制作服务器私人证书

比如*OpenSSL* 装在 `C:\Application\OpenSSL-Win64`

追加到了**用户级** `PATH`：

```
# 已执行（追加，不覆盖原有内容）
$old = [Environment]::GetEnvironmentVariable('Path','User')
[Environment]::SetEnvironmentVariable('Path', $old.TrimEnd(';') + ';C:\Application\OpenSSL-Win64\bin', 'User')
```

**新开一个终端**后验证：

```powershell
openssl version
# OpenSSL 4.0.1 9 Jun 2026
```

> 想撤销：`系统属性 → 环境变量 → 用户变量 Path`，删掉那一条即可。





### 服务器结构

做 demo 时的结构

```mermaid
flowchart LR
    B["Chrome"] <--> N["node<br/>https<br/>443"]
    subgraph S["服务器"]
        N
    end
```

*node* 自己端着 *443* 端口，自己持有私钥，自己做 *TLS* 握手，自己跑业务。一个进程干完所有事。



生产环境的结构

```mermaid
flowchart LR
    B["Chrome"] <-->|"https"| X["nginx<br/>https<br/>443"]
    X <-->|"http"| N["node<br/>http<br/>9527"]
    subgraph S["服务器"]
        X
        N
    end
```

*nginx* 在最外面接 *443*，**证书和私钥只配在 nginx 上**；它把 *TLS* 解开后，用**明文 http** 转发给内网的 *node:9527*。

这个动作有个名字：**TLS 终止（TLS Termination）**。



为什么生产环境要多一层

| 理由               | 说明                                                         |
| :----------------- | :----------------------------------------------------------- |
| 性能               | *nginx* 是 *C* 写的、事件驱动，*TLS* 握手（*RSA* 私钥运算）比 *Node* 快得多，还能开硬件加速 |
| 一处配置，多处复用 | 一台机器上跑5个 node 服务，证书只需要在 nginx 配一次         |
| 换证书不重启业务   | 换证书 `nginx -s reload` 即可，node进程完全无感              |
| 静态资源           | 图片/JS/CSS 直接由 nginx 吐出，根本不进 node                 |





### 证书来源

方式一：向权威 CA 购买（生产用）



方式二：本地自己当 CA（学习/内网用）

既然浏览器只信任它认识的CA，那我就自己造一个CA，再让浏览器认识

```mermaid
flowchart TB
    A["① 造一个 CA<br/>（CA私钥 + CA自签根证书）"] --> B["② 用这个 CA 给 localhost 签一张服务器证书"]
    B --> C["③ 把 CA 根证书装进操作系统的<br/>「受信任的根证书颁发机构」"]
    C --> D["④ 浏览器验签通过，小锁变绿"]
```





### OpenSSL构建证书



#### 命令总览

先建目录：

```powershell
cd 'c:\Users\19425\Desktop\笔记\agnet讲解师\https-lab\cert'
```

整体分两半：

```mermaid
flowchart LR
    subgraph CA["前三条：造 CA（公安局）"]
        direction TB
        c1["1. CA 私钥"] --> c2["2. CA 证书请求 CSR"] --> c3["3. CA 自签根证书"]
    end
    subgraph SV["后三条：造服务器证书（办身份证）"]
        direction TB
        s1["4. 服务器私钥"] --> s2["5. 服务器证书请求 CSR"] --> s3["6. CA 签发服务器证书"]
    end
    CA -->|"用 CA私钥 签"| SV
```



```
【上半：造公安局】                    【下半：办身份证】

命令1 → ca-pri-key.pem               命令4 → server-key.pem
命令2 → ca-pub-key.pem（CSR）         命令5 → server-scr.pem（CSR）
命令3 → ca-cert.crt（根证书）◄─────── 命令6 → server-cert.crt
              │ 用 CA私钥 签发 ┘
```



上半：造 CA（公安局）

| 命令 | 动作                                                         | 输出文件         | 文件是什么                                                   |
| ---- | ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| 1    | 生成 RSA 密钥对，用 3DES 加密保存                            | `ca-pri-key.pem` | **CA 私钥**，用密码锁着，签发证书时才解开用                  |
| 2    | 从私钥导出公钥，加上身份信息（国家/省/公司…），打包成一个待审文件 | `ca-pub-key.pem` | **CSR**（证书签名请求），不是公钥文件，名字有误导            |
| 3    | 用自己的私钥给自己的 CSR 盖章                                | `ca-cert.crt`    | **CA 根证书**，issuer = subject（自己签自己），要发给所有客户端 |



下半：造服务器证书（办身份证）

| 命令 | 动作                                                  | 输出文件          | 文件是什么                                                   |
| ---- | ----------------------------------------------------- | ----------------- | ------------------------------------------------------------ |
| 4    | 生成 RSA 密钥对，**不加密**                           | `server-key.pem`  | **服务器私钥**，Node 启动时直接读，不加密是为了免输密码      |
| 5    | 从私钥导出公钥，加上身份信息（CN 填域名 `localhost`） | `server-scr.pem`  | **服务器 CSR**，拿去交给 CA 审核盖章                         |
| 6    | CA 用自己的私钥给服务器的 CSR 盖章                    | `server-cert.crt` | **服务器证书**，握手时明文发给所有浏览器，里面含公钥+域名+CA签名 |





| 缩写 | 全称                        | 含义                                                   |
| ---- | --------------------------- | ------------------------------------------------------ |
| CA   | Certificate Authority       | 证书颁发机构，负责签发和管理数字证书的可信机构         |
| CSR  | Certificate Signing Request | 证书签名请求，申请证书时生成的文件，包含公钥和身份信息 |





#### 命令 1：生成 CA 私钥

动作：生成 RSA 密钥对，用 3DES 加密保存

```powershell
openssl genrsa -des3 -out ca-pri-key.pem 2048
# Enter PEM pass phrase:        ← 输入 123456，屏幕上不会有任何显示，正常的，输完直接回车
# Verifying - Enter PEM pass phrase:  ← 再输一遍 123456
```

这里的`123456` 就是 **3DES 的对称密钥**



| 片段                  | 含义                                                         |
| :-------------------- | :----------------------------------------------------------- |
| `genrsa`              | 用 *RSA* 算法生成一个**密钥对**。注意：*RSA* 私钥文件里其实**同时含公钥**，公钥可以随时从私钥推出来 |
| `-des3`               | 用对称算法 *3DES* 把这个私钥文件**再加一层密**。运行时它会问你要密码，这个密码就是 *3DES* 的 *key* |
| `-out ca-pri-key.pem` | 输出到文件。`.pem` = *Privacy-Enhanced Mail*，一种 *Base64* 文本封装格式 |
| `2048`                | **位数（bit），不是字节数**。大纲写的 `1024` 现在已被认为不安全，2048 起步 |



生成文件： `ca-pri-key.pem` ，**CA 私钥**，用密码锁着，签发证书时才解开用

```
-----BEGIN ENCRYPTED PRIVATE KEY-----   ← 注意 ENCRYPTED
MIIFJDBWBgkqhkiG9w0BBQ0wSTAxBgkqhkiG9w0BBQwwJAQQhA2YMavAnmNWhp/Y
RpzcuAICCAAwDAYIKoZIhvcNAgkFADAUBggqhkiG9w0DBwQI/0uraooc5KYEggTI
Lu+qz6i+ttmdOyxkSLfAm00elsJdJJt/U65SmTqI35N7QUayiQLElKN01gKYiiiX
bnl8prsrGyBiEf5q0+CGVxZLi0ymYfpN9GE1LUj1P9K1qMXba0dZrEbdopj7lia3
LrQF765UVusFCbpMN4qMMUlwzQbFdhwRxk9GPE/5iSHOZYK/umo7WqVx6Fm7l10p
kYc+G9cO/bB7VRNbJ3YGQeRAT9rZ7GpbjgOVqAnSkuZVIw1kuxco60AYkr0aSiL8
28Z+b8FDd4XvEqukJXTu7/ZTIZaUUW6nCn1jSbvD+vEjukQuVs/aaEck2VxSP1aX
4ufW8grfgYZ1t18WR68rAw3WIISpDIyC6zCnJMtTuaEneOyfeLnbkde2HPwhxXGP
g7h1/1qvI+yQnX08mam9Mda4ZpYtGf45qxmN3rvhU3DoMcBICpE53Wm2kHu8G3MI
7c9wiYWHgWk004ImzoUVrSOs2OZ3C9ro5PMiuCQV152UVW690QfAMIgNQxB5r6du
gor8BNyQwVg1m0L7u7V4Ccq+sd1j35O4LZSGikUgYTtKWNb9Gg8Bjvu68M/Y8NAQ
Hy81I4R5Sw9bD54FjDSJe0Mh4mqzqinUo+bOo97TyApu8WTf+heMyD6uJDnU5pFn
Oq3GiOEVebfp04IreDn06PlzSfZSsqnK0KZ6G8h1jamgZ1p46/rebkTMQ7tuGLsz
Sje13Zz9ZLyETIiIAqvTwlE9vYhX7tpjJjWuevzs09aYC/McnvdLvTK2lGmv3ift
wThqUs7wNq629pWStZpd+IGDGNghLtA8ZMrFy41EoR/Cl+XcgOeJvo1ux49KJ0Ya
5iDQhNdPTtGBPE8T35Nx0c3ZwdCIGXh77MrLQUCl0Wx4eJbCK0iubrVL3z81KKAc
hAXkaE4LM0PPcZueYKdrde4hNRZ0FsDfIf2xBFfcWroWcJ1l226u0gyC9ZJVFqJ+
YMhaCvdMKd2hTYdZ/YPOQ9p7B3X8owF8vSQOxFLcgqpTKIA2Sr9K3IiL8YYpeA5y
m7whrbtGvf4FxufckpqXAXayr0m8S5yzgAAebLTvjKhzgwXaiP+IjnkHNTNPakHs
BWNG6zm4F85pTBSD7yitj8c1szewXUs+S1bSKGNHL6IiIxoIvwX+cNtOPN7HAO7N
F1L4+M2Tnyj4xZcj51S9Kexvc5kzV1Qt3K2V90n5axCyAwY9ubf33h5XeUGVnlWG
qZkeVP8LIghiwiji2+HQsOvsqBArRCWYrrvC/JH1lnD/L39F5D9y2rXruocxB4oB
0aeqvWnkZBttF47c/JiZXYISuh0yuw+EzFOG+VxGnnZqwE+FA2gfvsvnyf8wZxvG
wEAVahoYcpkdMgZNBGhju3lsi871gRrTNHVJwCCydwaD+f69+DtbjlhdFYoSDBam
SuugPChAFqywGztevAuJLO0NAp6nyaLl1wGhQUCGtXZvj3Dy5Y4T/iHehFAHwu+P
6cHFwgVmw8AAmASbidOfL3haJ9oXsonRzi+sN3UonBrUla7l1Bn7MRCY4lfXmvzp
jm2R1TV26rvGCQcDZQLpTuEBzgPZFvEoaUmD7IZXtPBlx/y4l0yaYqx/BsyO6/EN
WDfq743xI0E5gRF2qxHhH5bHLoACKS6X
-----END ENCRYPTED PRIVATE KEY-----
```

去掉 `-des3` 再生成一次对比，头部会变成 `-----BEGIN PRIVATE KEY-----`。**这就是「加了密」肉眼可见的证据。**





#### 命令 2：生成 CA 证书请求

动作：从私钥导出公钥，加上身份信息（国家/省/公司…），打包成一个待审文件

```powershell
openssl req -new -key ca-pri-key.pem -out ca-pub-key.pem

Enter pass phrase for ca-pri-key.pem:   ← 输 123456，回车
```

`-key ca-pri-key.pem` 表示「从这把私钥里导出公钥」，因为私钥被 *3DES* 加密过，所以会先让你输密码。



运行时它会问一串问题：

| 提示                                        | 填什么             | 示例                                                     |
| :------------------------------------------ | :----------------- | :------------------------------------------------------- |
| *Country Name (2 letter code)*              | 国家，只能两个字母 | `CN`                                                     |
| *State or Province Name*                    | 省                 | `Sichuan`                                                |
| *Locality Name*                             | 市                 | `Chengdu`                                                |
| *Organization Name*                         | 组织/公司          | `DengGe-CA`                                              |
| *Organizational Unit Name*                  | 部门               | `Security`                                               |
| **Common Name (CN)**                        | **最关键的一项**   | *CA* 填机构名 `DengGe Root CA`；**服务器证书必须填域名** |
| *Email / challenge password / company name* | 直接回车跳过       |                                                          |



生成文件：`ca-pub-key.pem` CSR（证书签名请求），不是公钥文件

```j
-----BEGIN CERTIFICATE REQUEST-----
MIICtjCCAZ4CAQAwcTELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB1NpY2h1YW4xEDAO
BgNVBAcMB0NoZW5nZHUxEjAQBgNVBAoMCURlbmdHZS1DQTERMA8GA1UECwwIU2Vj
dXJpdHkxFzAVBgNVBAMMDkRlbmdHZSBSb290IENBMIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEAv8PGD+5RSewkqB4jLA4QejKxCez519i5gUsCJzwghPoo
KIWpqFl81yTVKpLBfJ2wK47pNwHOlek/tfypKB1R1VsDJlfNmjDGcwy7EfiPrxEZ
4tTkTwoa+9AanprVit/+t5AMJhOh0nyucFQ0ra99Xv1q2Nu9jy/SLZNi063U+bu2
u+Dzx4fnpRf7B9lKYF8JJ6FRF/TcaWyeY5SDjQ6ZQQJ3ROoeHTDPvyZIsd+n3nfr
llE59mtUNjBJzHcF+dHecSVFSzLdxeYr7iTg6p7j6Kax6dzw6KhGioPFq/0BEyjv
rM+JOpqIVkOVpqw9SJvtphy+j+TGaQbwrja5wrbejQIDAQABoAAwDQYJKoZIhvcN
AQELBQADggEBAIjPUvnbIkb1dEW8tK3oZXU5F+IAejFDIti9EHZA+SXWQ/auYh08
FRTOHPIwkEELwkp/lkrhU3/6BBj7yD/7WqE7dAhjQG+7ziyQgUrKUer0+yeTgecC
zyG8TJz3QVEvlJdeZKVxmKH2KxqE6BgaO/4xD3e+NA4yZpX3I1GyB5LKnlLWpQ1B
Qyu2v+m/4WB/vdm12nfKbdu5lFxl42zwjRMgkAGVmQe4tyvTRIEY+iXmCrBovotD
gBPeixwCiEJxOgwb3PAEl1lotJENxG5Ca3bXzufltEqc++rKC4kHNFFFTurcWa/n
XeXLAanNom7K1xMpbh1iIopYSlxASREW6f4=
-----END CERTIFICATE REQUEST-----
```





#### 命令 3：CA 自签根证书

动作：用自己的私钥给自己的 CSR 盖章

```powershell
openssl x509 -req -in ca-pub-key.pem -signkey ca-pri-key.pem -days 3650 -out ca-cert.crt

Enter pass phrase for ca-pri-key.pem: // 输入密码 123456
```

| 片段                      | 含义                                                   |
| :------------------------ | :----------------------------------------------------- |
| `x509`                    | 使用 *X.509* 证书标准（就是全世界通用的那个证书格式）  |
| `-req`                    | 告诉它「输入的是一个 *CSR*，不是证书」                 |
| `-in ca-pub-key.pem`      | 输入的 *CSR*                                           |
| `-signkey ca-pri-key.pem` | **用这把私钥签名**。签名者 = 自己 ⇒ **自签名证书**     |
| `-days 3650`              | 有效期 10 年（大纲没写，默认只有 30 天，容易过期踩坑） |
| `-out ca-cert.crt`        | 输出证书                                               |



生成文件：`ca-cert.crt`，**CA 根证书**，issuer = subject（自己签自己），要发给所有客户端

```
-----BEGIN CERTIFICATE-----
MIIDkTCCAnmgAwIBAgIUOuJSrIfCGhtzohJQXpmi/IAaaVIwDQYJKoZIhvcNAQEL
BQAwcTELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB1NpY2h1YW4xEDAOBgNVBAcMB0No
ZW5nZHUxEjAQBgNVBAoMCURlbmdHZS1DQTERMA8GA1UECwwIU2VjdXJpdHkxFzAV
BgNVBAMMDkRlbmdHZSBSb290IENBMB4XDTI2MDgwNDA2NDU0OVoXDTM2MDgwMTA2
NDU0OVowcTELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB1NpY2h1YW4xEDAOBgNVBAcM
B0NoZW5nZHUxEjAQBgNVBAoMCURlbmdHZS1DQTERMA8GA1UECwwIU2VjdXJpdHkx
FzAVBgNVBAMMDkRlbmdHZSBSb290IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAqOZG1fp1HE8vaIIIO9JIo4yNCfd8HDSxVS36B+XN6waxJ5DxLvdM
f13mK9F4U2tLh3n2M01p4i5Pr77fmoVGe46ClxLYGVaihWfCq3aOD2+w4qPHIPlv
E/zYY8h1r2xmvFVDGER4r8QREKtl7M+bshe2TagOZhwBHNxDgczEwPt2LlFZU5bY
00QaDrUyqHbr+pLzLkp2fz0NXK9J176Z7KJmomE6YkzAzQH/OkTCXllGgfw9YZrr
Iy0GdDxr+K+Jj5yblK/2mSIaVYr/nXSxapa1gA+LdWg5PqRU+pyfOYdxC4HymZm8
LzPA36CvS/ZAJNFX6SuUy6xO1Mjsg/MtNwIDAQABoyEwHzAdBgNVHQ4EFgQUYVZh
7Zji8GydjUsHELN6IcpcvwkwDQYJKoZIhvcNAQELBQADggEBAAwHUtRdavkLoiTh
4kWbjuckouP6vzGznuWhoZJiGEbUXgluBELEOEh70pldVYZA5RfSFwvt8Iy19704
8/bPdNmqKiDuvpGoxUlHlTAX/2NNcMylZEbrTBhJ1EwouN8PTgKrxAB3Vbw+5oiU
yBZUG8/fi8XtY4AWF7Vzk6J6oIflXCdHWIjfL8uaWvXY821nrJlj28pQnmyj1qex
gCki2h6ZlemK13cmmdo6DkxuNxxARCS7CDxoIhk+uvw6P0A53JybDVQYKjXLixLw
3DiyFuHolHqUpOE8oj1vlzFdP2Nx5044EGYyaTe33xti3gXpV36XZuovz6rnurGX
kbNzroQ=
-----END CERTIFICATE-----
```



读取并显示证书的主体（subject）和颁发者（issuer）信息

```
openssl x509 -in ca-cert.crt -noout -subject -issuer
```

```
subject=... CN=DengGe Root CA
issuer= ... CN=DengGe Root CA     ← subject 和 issuer 完全一样 = 自己给自己签
```



| 片段              | 含义                                                         |
| :---------------- | :----------------------------------------------------------- |
| `openssl x509`    | 操作 X.509 格式的证书                                        |
| `-in ca-cert.crt` | 输入文件：要查看的证书                                       |
| `-noout`          | **不输出**证书本身的编码内容（没有它会把整个证书 Base64 内容打印出来，很乱） |
| `-subject`        | 打印证书的**主体**（这张证书是颁给谁的）                     |
| `-issuer`         | 打印证书的**颁发者**（这张证书是谁签发的）                   |





#### ------ 华丽的分割线 ------

上面在造公安局，下面在办身份证。





#### 命令 4：生成服务器私钥

动作：生成 RSA 密钥对，**不加密**

```powershell
openssl genrsa -out server-key.pem 2048
```

**注意：这里没有 `-des3`。** 为什么？

因为 *Node* 启动时要读这个私钥文件。如果加了密，你就得在代码里写：

```js
{ key: fs.readFileSync('server-key.pem'), passphrase: '123456' }
```

密码硬编码在代码里 = 白加。所以工程上的通行做法是：**服务器私钥不加密，改用文件权限保护**（*Linux* 下 `chmod 400`，属主 *root*）。



生成文件：`server-key.pem`，**服务器私钥**，Node 启动时直接读，不加密是为了免输密码

```
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDn0g2ArPe6L3N0
yAAbxOW1/U58CXzIvtDmJroka5d5PrX8WutSEA55soHxesJyxGT0hpi8AeJdm4hM
BiYZuD6NUxZJ3o9K3V3aa4IU9uul+pFBaMJOODbOb1Myo+BroavJefWe28ZIaaNl
3vTmlFvviOY7gR+9eE6tN9uVug03DIP3OZttL6d8/4IE1nAuYBMSpAExDTE+WCR9
xBnki4AN3n4JSW183DhjMacAhLZkTMXQEZz/21Y7J4+lm62DdMwaIDHiucxRno2f
OTkUy26jrWFl6owjpgrImJiUf5gHw8Nw7RfNJsflgL5sr4S54Zpvpd8oNtzyB4By
mtGAbmnHAgMBAAECggEADSvjpYNcGWGOY08uFByxo7RFQvehmnxVmKF+Pr/vFIlq
H4NKF/1Vau6Vwsm/BeItex76XZ4BgRdZIYOINahTDPQnhB1gxCo/k2Q7psySxzAb
E/2v/ktRy5R4aOVjxLLtPbn5BJ/eh8AIPOIqoL757EW5ox29aUrgkF6Mm8Z0E+h0
hb0AjyCc8Vbtp97tERqvbx1sOLA1cyKaS+tamQTx3oi22X8OEJQIOyU4+z8+UEfY
FIavtFZ6Gowrq4Dy2Jcv3kCk4r0u3euUxjYEIVi761o98nyf1XNDe4JjwZyqUChi
O7kNkKvcyu4uta611yUkv1bqbBPVEmok38C3tgRPBQKBgQD2XpWaZJ7p6DwTu9td
Q4CEjABpIQHsHyf96YJk4v/5mB+ykH7d7C0NUNIcFha7AAylR99mQL0MRxHYP50/
nbcxuB4jCr7SgOE9gxTKBjR3M3JK1SVtVCVqOVY7dy6irO598Qsj8cFjLuJfU+rA
J2gu/WbFbHmFGWYriPgKtSV82wKBgQDw4eCRJNla8WmolELbo/cJf7NCtPwU3sGK
r/ATQZAelxQNe1dwmVNJjheA8bYiv89BFs8scngoqD1hRHNUQsbO/ZBJfY4IzPi+
X92UDcntInu2SkWbSDDqlF2Gtoac71vMRX1IbvP5KxyzFsQBUBPnNkLFPX9SC19W
JbyQrT1khQKBgDxdrJZp9S4nAyFqtDND6OeRTkB0H/51G91xK8wAeuvzREECerNx
vRA4L2NGXkdvmamd1LU/Q5MQjuTHoYYG5O4/yNKZPWxiu83QonVl09EpP+XqGAOT
zJ8ccW6co0akzML7Htmt0M62nCzzaN+wSCzLVjUlgRkFwi0EoJIvFIJlAoGBAKM9
n88c9sOfyKe62dg5NMfWJ6h0OET4dJbFBmA2VTG11pwQU0B2V7AIQDtdEr5X2F9W
0s1fQbjj0rzhA3s9ybcqtSZolOG37nYtR173PTmb3dS6XpOOB8BhDUSm23Ds8Pvs
t6tnraTVJ5ziN7BgESzmY7eBJpB2LoSWDETLckh1AoGBAKZ/xRgix3meFYabGAzG
5vlUHb0ogm+OuEIzZ3tZZShMwUcJNKDh9YNk171i8jOCgmfO/Rfc4VF4hKD9is/E
EiDbKvE1xDE8n2jcHyhuLvdBKmWL6wbBcJuRW6MctBbSEm3Wc4Ia96bqUBn9ej4f
9vJxHvCnElZL+xpJfWl0yj9p
-----END PRIVATE KEY-----
```





#### 命令 5：生成服务器证书请求

动作：从私钥导出公钥，加上身份信息（CN 填域名 `localhost`）

```powershell
openssl req -new -key server-key.pem -out server-scr.pem
```

跟命令 2 一样，但 **Common Name 这次必须填你要访问的域名**，比如 `localhost` 或 `www.monica.com`。



 生成文件：`server-scr.pem`，**服务器 CSR**，拿去交给 CA 审核盖章

```
-----BEGIN CERTIFICATE REQUEST-----
MIICrTCCAZUCAQAwaDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB1NpY2h1YW4xEDAO
BgNVBAcMB0NoZW5nZHUxEzARBgNVBAoMCk1vbmljYS1JbmMxDDAKBgNVBAsMA1dl
YjESMBAGA1UEAwwJbG9jYWxob3N0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
CgKCAQEA59INgKz3ui9zdMgAG8Tltf1OfAl8yL7Q5ia6JGuXeT61/FrrUhAOebKB
8XrCcsRk9IaYvAHiXZuITAYmGbg+jVMWSd6PSt1d2muCFPbrpfqRQWjCTjg2zm9T
MqPga6GryXn1ntvGSGmjZd705pRb74jmO4EfvXhOrTfblboNNwyD9zmbbS+nfP+C
BNZwLmATEqQBMQ0xPlgkfcQZ5IuADd5+CUltfNw4YzGnAIS2ZEzF0BGc/9tWOyeP
pZutg3TMGiAx4rnMUZ6Nnzk5FMtuo61hZeqMI6YKyJiYlH+YB8PDcO0XzSbH5YC+
bK+EueGab6XfKDbc8geAcprRgG5pxwIDAQABoAAwDQYJKoZIhvcNAQELBQADggEB
AJdPo3ljA+IBM/njVYL2DydMw+kY7vSjymK7jDCh2GW7njFcT0WBYEmd2Szwoqw0
YW5JU77rzkA83R4rniuRMuLBV29Wms5zVN9dzCU7TPkqwPT0kbfixWcQTKFJsKv3
p2buBefIh3Yr/Fi3Hd8l3yk4ewV7vRNyDgyA4v+3AZDVgXFDBMB8VyguwJH+O9PD
3Pw967ORg6wBMQTx8t04gSa/hKDRLsRcr9FUuVS1LO5UvpW4oEHoB+y5wLcMdoDY
vYBOHoN89UfH/WWTyJ3TvLdinspTJmj7P3iIjoQPat6Qskx4EbJIj1Bn3jObt2+P
xEfiV0UpboH9E7r9lBuglGA=
-----END CERTIFICATE REQUEST-----
```





#### 命令 6：CA 签发服务器证书

动作：CA 用自己的私钥给服务器的 CSR 盖章

```powershell
openssl x509 -req -CA ca-cert.crt -CAkey ca-pri-key.pem -CAcreateserial -in server-scr.pem -days 825 -out server-cert.crt

Enter pass phrase for ca-pri-key.pem:  // 输入密码 123456
```

| 片段                    | 含义                                                         |
| :---------------------- | :----------------------------------------------------------- |
| `-CA ca-cert.crt`       | 签发者的证书（用来把 *issuer* 信息填进新证书）               |
| `-CAkey ca-pri-key.pem` | **签发者的私钥 —— 真正用来算签名的东西**，会让你输密码       |
| `-CAcreateserial`       | 生成 `ca-cert.srl` 序列号文件。*CA* 签发的每张证书都要有唯一序列号，这是吊销机制（*CRL*/*OCSP*）的依据 |
| `-in server-scr.pem`    | 待签发的 *CSR*                                               |
| `-days 825`             | 有效期。浏览器规定服务器证书最长 *398* 天，本地实验随便填    |



生成文件：`server-cert.crt`，**服务器证书**，握手时明文发给所有浏览器，里面含公钥+域名+CA签名

```
-----BEGIN CERTIFICATE-----
MIIDqTCCApGgAwIBAgIUAeafSX0g781uJPe88qwPv6rJnUowDQYJKoZIhvcNAQEL
BQAwcTELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB1NpY2h1YW4xEDAOBgNVBAcMB0No
ZW5nZHUxEjAQBgNVBAoMCURlbmdHZS1DQTERMA8GA1UECwwIU2VjdXJpdHkxFzAV
BgNVBAMMDkRlbmdHZSBSb290IENBMB4XDTI2MDgwNDA3MDAyOVoXDTI4MTEwNjA3
MDAyOVowaDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB1NpY2h1YW4xEDAOBgNVBAcM
B0NoZW5nZHUxEzARBgNVBAoMCk1vbmljYS1JbmMxDDAKBgNVBAsMA1dlYjESMBAG
A1UEAwwJbG9jYWxob3N0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA
59INgKz3ui9zdMgAG8Tltf1OfAl8yL7Q5ia6JGuXeT61/FrrUhAOebKB8XrCcsRk
9IaYvAHiXZuITAYmGbg+jVMWSd6PSt1d2muCFPbrpfqRQWjCTjg2zm9TMqPga6Gr
yXn1ntvGSGmjZd705pRb74jmO4EfvXhOrTfblboNNwyD9zmbbS+nfP+CBNZwLmAT
EqQBMQ0xPlgkfcQZ5IuADd5+CUltfNw4YzGnAIS2ZEzF0BGc/9tWOyePpZutg3TM
GiAx4rnMUZ6Nnzk5FMtuo61hZeqMI6YKyJiYlH+YB8PDcO0XzSbH5YC+bK+EueGa
b6XfKDbc8geAcprRgG5pxwIDAQABo0IwQDAdBgNVHQ4EFgQUC2DTyw7MhWhSpIj/
2mvxuDkUjTMwHwYDVR0jBBgwFoAUYVZh7Zji8GydjUsHELN6IcpcvwkwDQYJKoZI
hvcNAQELBQADggEBAIGQyn3NEJVPopkM1Lz48E8DhDsTvvIeuq7AgD2eGe7INc6T
PQFvHP9ZXe8c5OxQGOQyjnPUU+h9MuGCN1JrU3HhPPH+tJQBuKAnok/5NHWnhD0a
Q1ll9r7udRiCIw/eiK0AhdzZ+WMIvKfS25D5QP6sHgcCMpMhe1A+4nIH0eKJd7ab
SRNAJfoMU7Uma4Omu9Dwl6TnQjJFuuSOjorVWLe9IuzcmTKSqcdBeYClU0wQQyq6
O0Q0cQlmAkBuYXFZUUKae2zRhQZOf1JzvHOdU3bHe6wBTSgNqOYrBcijZhBxGeRV
MSJMt9PvCc6quUZFIPIF0vk/V+o7WEv4nQiTIWc=
-----END CERTIFICATE-----
```





#### 每一步在干什么

上半：造 CA（扮演公安局）

| 步骤                  | 本质                                           | 类比                                               |
| :-------------------- | :--------------------------------------------- | :------------------------------------------------- |
| 命令1：生成 CA 私钥   | 造一把只有 CA 自己有的"专用印章钥匙"，加密锁住 | 公安局刻了一个防伪章，锁进保险柜                   |
| 命令2：生成 CA 的 CSR | 把 CA 的公钥 + 身份信息打包成申请文件          | 公安局写了一份自我介绍（"我是谁、我的公章长啥样"） |
| 命令3：CA 自签根证书  | 用自己的私钥给自己的申请盖章                   | 公安局给自己的自我介绍**盖了自己的章**             |

> ⚠️ 命令3之后，CA 根证书（`ca-cert.crt`）需要**手动安装进浏览器**，浏览器才会信任它。这就是"自签"的代价——没人给你担保，只能自己说"相信我"。

------

下半：造服务器证书（办身份证）

| 步骤                     | 本质                                              | 类比                                           |
| :----------------------- | :------------------------------------------------ | :--------------------------------------------- |
| 命令4：生成服务器私钥    | 造服务器自己的私钥，**不加密**（Node 要自动读取） | 服务器自己刻了一把私章，不加锁                 |
| 命令5：生成服务器的 CSR  | 把服务器公钥 + 域名（`localhost`）打包成申请文件  | 服务器写申请书："我叫 localhost，这是我的公钥" |
| 命令6：CA 签发服务器证书 | CA 用**自己的私钥**给服务器的 CSR 盖章            | 公安局审核申请书，盖上公安局的章 → 身份证诞生  |





### https 模块本体



#### 和http的差别

```js
// 01-http-server.js
import http from 'node:http';
const server = http.createServer((req, res) => { ... });
server.listen(80);

// 02-https-server.js
import https from 'node:https';
const options = {
  key:  fs.readFileSync('cert/server-key.pem'),   // 私钥
  cert: fs.readFileSync('cert/server-cert.crt'),  // 证书
};
const server = https.createServer(options, (req, res) => { ... });  // ← 多一个 options
server.listen(443);
```

| 差别                | http         | https                |
| :------------------ | :----------- | :------------------- |
| 模块                | `node:http`  | `node:https`         |
| `createServer` 参数 | `(handler)`  | `(options, handler)` |
| 默认端口            | 80           | 443                  |
| **回调里的写法**    | **完全一样** | **完全一样**         |





#### 最小可运行https

利用上面的证书构建的最小可运行https服务器

node执行，文件代码在后面

```
cd 'c:\Users\19425\Desktop\笔记\agnet讲解师\https-lab'
node 02-https-server.js      # 终端 A
```

访问可以看到，确实是https协议

<img src="./Typora图片/image-20260804151315791.png" alt="image-20260804151315791" style="zoom:50%;" />

`02-https-server.js`

```js
// 最小可用的 https 服务器
// 跟 01-http-server.js 逐行对比：只有 3 处不同（模块名、options、端口）
import https from 'node:https';
import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const certDir = path.resolve(__dirname, 'cert');

// 差异点 ①：https.createServer 多接收一个 options，里面装的就是「私钥 + 证书」
const options = {
  // key：服务器私钥。用来解开客户端发来的密钥材料 —— 绝不外传
  key: fs.readFileSync(path.join(certDir, 'server-key.pem')),
  // cert：服务器证书。里面含公钥 + 域名 + CA 签名 —— 握手时明文发给所有人
  cert: fs.readFileSync(path.join(certDir, 'server-cert.crt')),
};

// 差异点 ②：https.createServer(options, handler)
const server = https.createServer(options, (req, res) => {
  // 这里开始，跟 http 的写法 100% 一致
  const socket = req.socket;
  res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
  res.end(`
    <h1>我是加密的 https</h1>
    <p>socket.encrypted = ${socket.encrypted}</p>
    <p>TLS 版本：${socket.getProtocol?.()}</p>
    <p>协商出的密码套件：${socket.getCipher?.().name}</p>
    <p>路径：${req.url}</p>
  `);
});

// 握手失败时服务端不会崩，但会静默丢弃连接。挂上这个监听才能看到真实原因
server.on('tlsClientError', (err, socket) => {
  console.log('[握手失败]', err.code, '-', err.message);
});

// 差异点 ③：https 的默认端口是 443
server.listen(443, () => {
  console.log('https 服务已启动 → https://localhost');
});
```





####  https 当客户端

用 Node 当客户端，用同一个服务器、**改变一个变量**，跑四次请求，对比结果。代码在章节末尾



场景1：默认行为 → 用内置CA列表验服务器证书 → 找不到 DengGe CA ❌ 失败

```
不告诉 Node 我们的 CA 是谁
→ Node 去内置权威CA列表里找 → 找不到 DengGe Root CA
→ 报错：证书由未知机构签发
```



场景2：传入之前造的CA验服务器证书 `ca: caCert` → ✅ 成功

```
把 ca-cert.crt 告诉 Node
→ 相当于"把根证书安装进浏览器"的编程实现
→ 验签通过，连接建立
```

> 说明：这就是浏览器里"安装根证书"这个操作的本质



场景3：CA 验证通过，但域名不匹配 → ❌ 失败

```
ca 传对了，但 servername 填了 www.hacker.com
→ 证书里写的 CN=localhost，对不上
→ 报错：hostname/IP does not match certificate
```

> 说明：证书验证有两关——**CA签名** 和 **域名匹配**，缺一不可



场景4：`rejectUnauthorized: false` → 跳过对服务器证书的验证 → ✅ 成功（但有代价）

```
客户端直接不验证服务端的证书
→ 连接成功，但 socket.authorized = false
```

> 说明：这等于"加密了但不认证身份"——数据是密文，但你不知道对面是不是真服务器，中间人攻击依然有效



文件内容如下：

`03-https-client.js`

```js
// Node 当客户端，用三种姿势访问自签服务器，直观看到「验证」到底在验什么
// 先跑 02-https-server.js，再跑本文件：node 03-https-client.js
import https from 'node:https';
import fs from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const caCert = fs.readFileSync(path.resolve(__dirname, 'cert/ca-cert.crt'));

function request(title, options) {
  return new Promise((resolve) => {
    console.log(`\n===== ${title} =====`);
    const req = https.request({ hostname: 'localhost', port: 443, path: '/', ...options }, (res) => {
      const cert = res.socket.getPeerCertificate();
      console.log('✅ 成功  状态码:', res.statusCode);
      console.log('   对方证书 subject :', cert.subject?.CN);
      console.log('   对方证书 issuer  :', cert.issuer?.CN);
      console.log('   SAN              :', cert.subjectaltname);
      console.log('   本次连接是否被信任:', res.socket.authorized, res.socket.authorizationError ?? '');
      res.resume();
      res.on('end', resolve);
    });
    req.on('error', (err) => {
      console.log('❌ 失败  错误码:', err.code);
      console.log('   ', err.message);
      resolve();
    });
    req.end();
  });
}

// 场景 1：默认行为。Node 只信任内置的权威 CA 列表，我们的野鸡 CA 不在里面
await request('场景1 默认（不告诉 Node 我们的 CA）', {});

// 场景 2：把我们的 CA 根证书告诉 Node —— 相当于「浏览器安装根证书」的编程版
await request('场景2 传入 ca: ca-cert.crt', { ca: caCert });

// 场景 3：认了 CA，但故意用一个不在 SAN 里的域名去访问
await request('场景3 CA 正确，但域名不在 SAN 里', { ca: caCert, servername: 'www.hacker.com' });

// 场景 4：直接关掉校验（只能用于本地调试，线上写这行等于把 https 退化成 http+加密无认证）
await request('场景4 rejectUnauthorized: false', { rejectUnauthorized: false });

```





#### https 客户端访问公告站点

```js
import https from 'node:https';

https.get('https://www.baidu.com', (res) => {
  console.log(res.statusCode);
  res.on('data', (c) => process.stdout.write(c));
});
```

访问公网站点不需要配任何证书 —— *Node* 内置了一份权威 *CA* 根证书列表（来自 *Mozilla*），`tls.rootCertificates` 可以打印出来看：

```js
import tls from 'node:tls';
console.log(tls.rootCertificates.length);  // 一百多张
```







## Node.js 事件循环



### 一句话总结

Node主线程跑一遍同步代码后，进入一个【死循环】，这个循环按照固定六个顺序巡视六个队列，每个队列里装着一类回调；每执行完一个回调，都要先把 nextTick 和 Promise 两个“插队队列”清空，才会执行下一个

![image-20260805152046661](./Typora图片/image-20260805152046661.png)





### Node 架构

```
┌───────────────────────────────────────────┐
│  JS 代码                              	  │
├───────────────────────────────────────────┤
│  Node API（fs / net / http / crypto …）   │
├────────────────┬──────────────────────────┤
│   V8 引擎      │      libuv (C 语言)       │
│  （执行 JS）   │  事件循环 + 线程池 + epoll │
└────────────────┴──────────────────────────┘
```

当写 `fs.readFile('big.txt', cb)`：

1. JS 主线程把这个任务**扔给 libuv**，然后**立刻返回**，继续跑下一行
2. libuv 派一个**线程池里的工人线程**去真正读磁盘
3. 读完了，工人线程把 `cb` 塞进 **poll 队列**
4. 主线程巡逻到 poll 阶段时，取出 `cb` 来执行





### 一轮循环六个阶段

```js
   ┌─────────────────────────────┐
┌─>│           timers            │  setTimeout / setInterval 的回调
│  └──────────────┬──────────────┘
│  ┌──────────────┴──────────────┐
│  │      pending callbacks      │  上一轮遗留的系统级回调（如 TCP 错误）
│  └──────────────┬──────────────┘
│  ┌──────────────┴──────────────┐
│  │      idle, prepare          │  Node 内部自用，你碰不到
│  └──────────────┬──────────────┘      ┌───────────────┐
│  ┌──────────────┴──────────────┐      │   incoming:   │
│  │            poll             │<─────┤  连接、数据、  │
│  └──────────────┬──────────────┘      │   文件读完…    │
│  ┌──────────────┴──────────────┐      └───────────────┘
│  │            check            │  setImmediate 的回调
│  └──────────────┬──────────────┘
│  ┌──────────────┴──────────────┐
└──┤       close callbacks       │  socket.on('close') 之类
   └─────────────────────────────┘
```

`nextTick`和`Promise`不属于任何阶段，这两者是每个阶段之间的插队通道，甚至是每个回调之间的插队通道

只需要真正关心 **timers / poll / check** 三个，另外三个日常几乎遇不到。





#### timers —— 存放计时器的回调

```js
setTimeout(() => console.log('hi'), 1000);
```

setTimeout不是一秒后执行，而是一秒后才有资格被执行

- 计时由libuv在底层记时间戳，不占 JS 线程
- 到期后回调进 timers 队列
- 必须等主线程巡逻到 timers 阶段，才会正真跑

所以：

```js
setTimeout(() => console.log('timer'), 100);
const s = Date.now();
while (Date.now() - s < 500) {}   // 同步卡 500ms
// 实际输出时间：500ms 后，不是 100ms
```





#### pending callbacks

上一轮循环中被推迟的**系统级** I/O 回调，最典型的是 TCP 层的错误（比如 `ECONNREFUSED`）。写业务基本碰不到，知道有这么个格子就行。





#### idle / prepare

libuv 内部使用。





#### poll —— 全场最重要的阶段

`fs.readFile`、`http`服务器收到请求、`socket`收到数据、数据库查询返回…… 99% 的业务回调都在这里



poll运作方式

```
进入 poll 阶段：
│
├─ poll 队列里有回调？
│   └─ 有 → 依次执行，直到队列清空（或达到系统上限，防止饿死后面的阶段）
└─ poll 队列是空的？
    ├─ check 队列里有 setImmediate 待执行？
    │   └─ 有 → 立刻结束 poll，进入 check 阶段
    │
    ├─ timers 里有已到期的定时器？
    │   └─ 有 → 立刻结束 poll，回到循环开头去跑 timers
    │
    └─ 都没有 → **阻塞在这里睡觉**，等操作系统通知有新 I/O 完成
        （但最多睡到"最近一个定时器到期"的那一刻）
```

最后的等待阻塞是 Node 高性能的核心：它不是在空转轮询，是真的在 `epoll_wait` 上睡着了，CPU 占用 0%。



```js
const start = Date.now();
console.log('主线程结束，此刻没有任何 I/O 回调可执行');
setTimeout(() => console.log(`定时器触发，poll 白等了约 ${Date.now() - start}ms`), 800);
```

```
主线程结束，此刻没有任何 I/O 回调可执行
定时器触发，poll 白等了约 802ms      ← 这 800ms 里 CPU 是 0%
```





#### check —— setImmediate 的专属车道

使用 `setImmediate` 的回调会直接进入这个队列。

`setImmediate` 的语义就是："等 poll 阶段忙完，立刻执行我"。

它的名字起得极烂——`setImmediate` 并不比`setTimeout(fn, 0)`更"immediate"。真正准确的翻译是 **`setInPollDone`**。





####  close callbacks

`socket.on('close', cb)`、`server.close(cb)` 之类的关闭事件回调





### 微任务：nextTick 与 Promise



规则一：两个队列有优先级，nextTick > Promise

`process.nextTick` 队列**整个清空**之后，才轮到 Promise 的 ``.then`

```js
Promise.resolve().then(() => console.log('promise 1'));
process.nextTick(() => console.log('nextTick 1'));
Promise.resolve().then(() => console.log('promise 2'));
process.nextTick(() => {
  console.log('nextTick 2');
  process.nextTick(() => console.log('nextTick 3'));   // 清空过程中新加的
});
```

```
nextTick 1
nextTick 2
nextTick 3      ← 注意：清空过程中新塞进来的，也会在本次一起清掉
promise 1
promise 2
```



规则二：清空的时机是"每个回调之间"，不是"每个阶段之间"（Node 11+ 的行为）

```js
setTimeout(() => {
  console.log('timer 1');
  Promise.resolve().then(() => console.log('   └─ timer 1 的微任务'));
  process.nextTick(() => console.log('   └─ timer 1 的 nextTick'));
}, 0);
setTimeout(() => {
  console.log('timer 2');
  Promise.resolve().then(() => console.log('   └─ timer 2 的微任务'));
}, 0);
```

```
timer 1
   └─ timer 1 的 nextTick
   └─ timer 1 的微任务
timer 2
   └─ timer 2 的微任务
```

不是 `timer 1 → timer 2 → 微任务们`。这一点和浏览器现在的行为已经对齐了（Node 10 及以前是按阶段清的）。



规则三：微任务能饿死事件循环

```js
setTimeout(() => console.log('timer 终于跑上了'), 0);
let n = 0;
(function loop() {
  if (++n < 2_000_000) process.nextTick(loop);
  else console.log('nextTick 递归结束');
})();
```

```
nextTick 递归结束
timer 终于跑上了，被拖延了 800+ms
```





####  ESM 陷阱

在 `.mjs` 或 `"type": "module"` 的项目里，**顶层代码**中的 promise 会跑到 nextTick 前面：

```js
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
```

原因：ESM 的模块求值本身就跑在一个 microtask 里，此时遵循浏览器的事件循环，顶层代码结束时 V8 会先把当前微任务队列续完，才轮到 Node 的 nextTick 队列。







## Node 事件中枢：EventEmitter



### 解决了什么问题

假设写了一个下载器：

```js
function download(url) {
  // 下载完成后要做什么？
  // 1. 保存文件  2. 打印日志  3. 通知用户  4. 更新进度条...
}
```

如果把这些功能都写在download中，那么每加一个需求就要修改一次download源码

往后面需求叠加

```js
// 又来了：要处理错误、要更新进度条、要通知用户
function download(url, onDone, onError, onProgress, onNotify) {
  //                  ↑         ↑         ↑           ↑
  //               第2个       第3个      第4个        第5个
  //           顺序记不住，随时传错位

  。。。
}

// ===== 调用方：根本数不清在传什么 =====
download(
  "https://example.com/file.zip",
  (data) => saveFile(data),   // onDone    ← 第几个？
  (err)  => alert(err),       // onError   ← 第几个？
  (pct)  => updateBar(pct),   // onProgress← 第几个？
  ()     => toast("完成!")     // onNotify  ← 第几个？
);

// 三个月后同事改代码，把 onError 和 onProgress 传反了：
download(
  url,
  saveFile,
  updateBar,  // ← 他以为这是 onProgress，其实是 onError 的位置
  alert,      // ← 默默地传错了，没有任何报错
  toast
);
```



EventEmitter的思路是发布-订阅

**EventEmitter的功能就是，可以在执行完一部分代码后，往全局广播事件**

**EventEmitter对象可以通过监听这些事件，实现功能插入**



新模式的解决方式是让download仅负责下载，并在其中添加事件，如果想让download做额外的事情，就在外部订阅这个事件，并把想在事件发生时所做的事情传进去

```js
const d = download(url);
d.on('done', saveFile);
d.on('done', log);        // 同一个事件可以挂多个监听
d.on('progress', updateBar);
```

download内部伪代码

```js
function download(url) {
  const emitter = new EventEmitter();

  fetch(url)                         // 1. 发起网络请求，返回 Promise
    .then(response => response.blob()) // 2. 请求成功 → 把响应体转成二进制数据，再返回一个 Promise
    .then(data => {                    // 3. 转换成功 → data 就是最终的文件内容
      emitter.emit('done', data);      //    数据准备好了，把事件“done"广播出去
    })
    .catch(err => {                    // 4. 上面任何一步出错 → 统一在这里处理
      emitter.emit('error', err);
    });

  // ↓ 把 emitter 还给调用方，让他自己决定怎么响应
  return emitter;
}

// download 源码从此封笔，永远不需要再改
```

注册在事件done上的的saveFile和log都会被调用

```
┌─────────────────────────────────────────────────────────┐
│                    发布-订阅模型                          │
│                                                         │
│   d.on('done', saveFile)   ← saveFile 订阅了 'done'      │
│   d.on('done', log)        ← log      也订阅了 'done'    │
│   d.on('progress', updateBar) ← updateBar 订阅 'progress'│
│                                         │               │
│         download 内部完成时：            │               │
│         emit('done', data)  ────────────┤               │
│                                         ↓               │
│                              saveFile(data)             │
│                              log(data)      ← 都会被调用 │
└─────────────────────────────────────────────────────────┘
```







### 内部结构

EventEmitter 实例内部就一个对象 `_events`：

```js
emitter._events = {
  'done':     [fn1, fn2],
  'progress': [fn3],
  'error':    [fn4]
}
```

`key`是事件名，`value`是该事件的监听器数组（队列）。所以“内部维护多个事件队列” = 一张`事件名→函数数组`的哈希表

- `on` = 往对应数组 `push`
- `emit` = 取出对应数组，**同步地、按注册顺序**依次调用
- `off` = 从数组里 `splice` 掉

剩下的 API 都是这三件事的变体。





### 基本用法

```js
const EventEmitter = require('events');   // 注意：模块导出的就是类本身

const bus = new EventEmitter();

// 注册
bus.on('msg', (a, b) => console.log('收到', a, b));

// 触发，emit 的参数会原样传给监听器
bus.emit('msg', 'hello', 123);   // 收到 hello 123

// emit 返回值：有没有监听器
console.log(bus.emit('nobody'));  // false
```

实际项目里常通过继承

```js
class Downloader extends EventEmitter { // 创建一个Downloader类，继承自EventEmitter
  start(url) {
    this.emit('start', url); // 触发start事件，并传递url参数
    setTimeout(() => this.emit('done', '文件内容'), 1000); // 模拟下载完成后，触发done事件，并传递文件内容参数
  }
}

const d = new Downloader(); // 创建Downloader实例

// 订阅，也叫做注册监听器
d.on('start', u => console.log('开始', u)); // 监听start事件，当start事件被触发时，执行回调函数，打印开始和url参数
d.on('done', c => console.log('完成', c)); // 监听done事件，当done事件被触发时，执行回调函数，打印完成和文件内容参数

d.start('http://x.com'); // 调用start方法，传递url参数
```

















