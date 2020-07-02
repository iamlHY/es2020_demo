在引入 ECMAScript 2015（又称 ES6）之前，JavaScript 发展的非常缓慢。自 2015 年起，JS每年都有新特性添加进来。下面将介绍 ECMAScript 2020（ES2020）的一些最新特性。

##Optional Chaining 可选链式调用
可选链 可让我们在查询具有多层级的对象时，不再需要进行冗余的各种前置校验。

日常开发中，我们经常会遇到这种查询
```
const name = user && user.info && user.info.name;
```

又或是这种
```
const age = user && user.info && user.info.getAge && user.info.getAge();
```
这是一种繁琐但又不得不做的前置校验，否则很容易命中 Uncaught TypeError: Cannot read property... 这种错误，这极有可能让你整个应用挂掉。

用了 Optional Chaining ，上面代码会变成
```
const name = user?.info?.name;
const age = user?.info?.list?.[1];
const age = user?.info?.getAge?.();
```
可选链中的 ? 表示如果问号左边表达式有值, 就会继续查询问号后面的字段。根据上面可以看出，用可选链可以大量简化类似繁琐的前置校验操作，而且更安全。

##Nullish Coalescing 空值合并
当我们查询某个属性时，经常会遇到，如果没有该属性就会设置一个默认的值。它适用于很多情况，但不能应用在一些特殊的场景。例如，初始值是布尔值或数字的情况。举例说明，我们要把数字赋值给一个变量，当变量的初始值不是数字时，就默认其为 7 ：
```
let number = 1
let myNumber = number || 7
```
变量 myNumber 等于 1，因为左边的（number）是一个 真 值 1。但是，当变量 number 不是 1 而是 0 呢？
```
let number = 0
let myNumber = number || 7
```
0 是 假 值，所以即使 0 是数字。变量 myNumber 将会被赋值为右边的 7。但结果并不是我们想要的。而由两个问号组成：?? 的合并操作符就可以检查变量 number 是否是一个数字，而不用写额外的代码了。
```
let number = 0
let myNumber = number ?? 7
```
操作符右边的值仅在左边的值等于 null 或 undefined 时有效，因此，例子中的变量 myNumber 现在的值等于 0 了。

##Private Fields 私有字段
许多具有 classes 的编程语言允许定义类作为公共的，受保护的或私有的属性。Public 属性可以从类的外部或者子类访问，protected 属性只能被子类访问，private 属性只能被类内部访问。JavaScript 从 ES6 开始支持类语法，但直到现在才引入了私有字段。要定义私有属性，必须在其前面加上散列符号：#。
```
class Flower {
  #leaf_color = "green";
  constructor(name) {
    this.name = name;
  }

  get_color() {
    return this.#leaf_color;
  }
}

const rose = new Flower("rose");

console.log(rose.get_color()); // 输出：green
console.log(rose.#leaf_color) // 报错：SyntaxError: Private field '#leaf_color' must be declared in an enclosing class
```
如果我们从外部访问类的私有属性，就会报错。

##Promise.allSettled 方法
####Promise.all 缺陷

都知道 Promise.all 具有并发执行异步任务的能力。但它的最大问题就是如果其中某个任务出现异常(reject)，所有任务都会挂掉，Promise直接进入 reject 状态。

如果页面有三个区域，分别对应三个独立的接口数据，使用 Promise.all 来并发三个接口，如果其中一个接口服务异常，状态是reject，这会导致页面中该三个区域数据全都无法渲染出来，因为任何 reject 都会进入catch回调，如下：
```
//假设p1，p2是请求
const p1 = new Promise((res, rej) => setTimeout(() => res([1,2]), 1000));
const p2 = new Promise((res, rej) => setTimeout(() => rej('reject'), 1000));
Promise.all([p1,p2]). then (data => {
// 如果其中一个任务是 reject，则不会执行到这个回调。
	RenderContent (data);
})
. catch (error => {
// 本例中会执行到这个回调
	console.log(error);// reject
})
```
在ES2020中Promise.allSettled 就解决这问题，如果并发任务中，无论一个任务正常或者异常，都会返回对应的的状态（fulfilled 或者 rejected）与结果（业务value 或者 拒因 reason），在 then 里面通过 filter 来过滤出想要的业务逻辑结果，这就能最大限度的保障业务当前状态的可访问性。
```
Promise.allSettled([p1, p2])
.then(data => {
	console.log(data);//[ {status: "fulfilled", value: [1,2]},{status: "rejected", reason: reject }]
	//过滤掉status为rejected的元素，继续处理数据
});

```
##Dynamic Import 动态引入
动态 import 允许您将 JS 文件作为原生应用用程序中的模块动态导入。 在 ES2020之前，不管是否使用模块，都应该导入模块。
例如，假设我们需要添加一个下载功能。在动态导入之前，无论用户是否使用它，download.js仍然需要被导入。在动态导入之后，只有在需要模块时才延迟加载模块，从而减少开销和页面加载时间。
```
// download.js
export default {
    download() {
        // 代码
    }
}

// 动态导入前，全局导入
import { download } from "./download.js";
const downloadButton = document.querySelector(".downloadButton");
downloadButton.addEventListener("click", download);

// 动态导入后
const downloadButton = document.querySelector('.downloadButton');
downloadButton.addEventListener('click', () => {
  import('./download.js')
    .then(module => {
      module.download()
    })
    .catch(err => {
      // handle the error if the module fails to load
    })
})
```
##MatchAll 匹配所有项
如果你想要查找字符串中所有正则表达式的匹配项和它们的位置，MatchAll 非常有用。
```
const str = '<text>JS</text><text>正则</text>';
const reg = /<\w+>(.*?)<\/\w+>/g;

console.log(str.match(reg));//  ["<text>JS</text>", "<text>正则</text>"]
```
相比之下，matchAll 返回更多的信息，包括找到匹配项的索引。
```
const str = '<text>JS</text><text>正则</text>';
const allMatchs = str.matchAll(/<\w+>(.*?)<\/\w+>/g);

for(const match of allMatchs) {
  console.log(match);
}

// 输出
/*
[
    "<text>JS</text>",
    "JS",
    index: 0,
    input: "<text>JS</text><text>正则</text>",
    groups: undefined
]

[
    "<text>正则</text>",
    "正则",
    index: 15,
    input: "<text>JS</text><text>正则</text>",
    groups: undefined
]
*/
```
##globalThis 全局对象
Javascript 在不同的环境获取全局对象有不通的方式，node 中通过 global, web中通过 window, self 等，有些甚至通过 this 获取，但通过 this 是及其危险的，this 在 js 中异常复杂，它严重依赖当前的执行上下文，这些无疑增加了获取全局对象的复杂性。
```
// 过去获取全局对象，可通过一个全局函数
const beforeGlobalThis = (typeof window !== "undefined"
? window
: (typeof process === 'object' &&
   typeof require === 'function' &&
   typeof global === 'object')
    ? global
    : this);

beforeGlobalThis.name = 'xiaoming';

// ES2020，直接使用globalThis，不用去担心环境的问题
globalThis.name = 'xiaoming';
```
##BigInt
Js 中 Number类型只能安全的表示-(2^53-1)至 2^53-1 范的值，即Number.MINSAFEINTEGER 至Number.MAXSAFEINTEGER，超出这个范围的整数计算或者表示会丢失精度。
```
let num = Number .MAX_SAFE_INTEGER;   // 9007199254740991

num = num +  1 ;  //  9007199254740992

// 再次加 +1 后无法正常运算
num = num +  1 ;  //  9007199254740992

// 两个不同的值，却返回了true
9007199254740992 ===  9007199254740993 // true
```
为解决此问题，ES2020提供一种新的数据类型：BigInt。使用 BigInt 有两种方式：

####在整数字面量后面加n。

```
let  bigIntNum =  9007199254740993n ;
```
####使用 BigInt 函数。
```
let bigIntNum =  BigInt ( 9007199254740 );
let  anOtherBigIntNum =  BigInt ( '9007199254740993' );
```
通过 BigInt， 我们可以安全的进行大数整型计算。
```
let bigNumRet =  9007199254740993n +  9007199254740993n ;  // 18014398509481986n
bigNumRet.toString();  //  '18014398509481986'
```
