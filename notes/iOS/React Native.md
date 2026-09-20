## JavaScript / TypeScript

### 闭包的理解

**什么是闭包（是什么：定义与本质）**

闭包就是函数 + 它创建时所能访问的外部环境变量（词法作用域）的组合。

换句话说就是：一个函数记住了它创建时的作用域，即使这个函数在别的地方执行，也能访问当时的变量。

```swift
function outer() {
  let count = 0;

  function inner() {
    count++;
    console.log(count);
  }

  return inner;
}

const fn = outer();

fn(); // 1
fn(); // 2
fn(); // 3

•	outer() 执行完后，本来应该销毁
•	但是 inner 被返回并赋值给 fn
•	inner 仍然引用了 count
•	所以 count 不会被 GC 回收

👉 这就是闭包
```

**闭包解决的核心问题**（为什么：背景，目的和重要性）

* 让变量活的更久（延长变量生命周期）
* 实现私有变量

```js
function createCounter() {
  let count = 0;

  return {
    inc() { count++; },
    get() { return count; }
  };
}
// 外部无法直接访问count
```

* 维持状态（核心）：**在多次渲染（render）之间，组件还能“记住”之前的数据**

```swift
React/RN本质就是：函数+状态
闭包是函数式编程的核心基础。

function Component() {
  const [count, setCount] = useState(0);

  function click() {
    console.log(count);
  }
}

// click是闭包，它捕获了count
// React 每次render，创建新的函数，新的闭包，新的作用域。闭包负责把state带进函数。UI = f(state)

React是自己存状态，而不是靠闭包。

React 内部（Fiber 节点）：
Component {
  hooks: [
    { state: 0 },   // 第一个 useState
    { state: xxx }, // 第二个 useState
  ]
}
// 当你写const [count, setCount] = useState(0);
// 实际发生：
1. React 找到当前组件的 hooks 数组
2. 取出第一个 state（count）
3. 返回给你
state是存在React内部，而不是函数。
```

**怎么来的**

闭包来自两个核心机制：

* 词法作用域（Lexical Scope）

  JS定义时决定作用域，而不是使用时。

  ```js
  function outer() {
    let a = 1;
  
    function inner() {
      console.log(a);
    }
  }
  // inner在定义时就绑定了a
  ```

* 执行上下文 + 作用域

  JS执行时，会创建

  * Execution Context（执行上下文）
  * Scope Chain（作用链）

闭包出现的条件：

```js
- 函数嵌套
- 函数内部引用外部变量
- 内部函数被带出作用域
```

**怎么工作的（原理、机制和过程）**

* 内存模型

  ```js
  const fn = outer();
  
  // js内部做了：
  outer 执行 → 创建执行上下文
    ↓
  生成变量对象（count）
    ↓
  inner 被返回
    ↓
  ⚠️ JS发现 inner 还引用 count
    ↓
  👉 outer 的变量对象不能释放
  ```

* 闭包结构本质

  ```js
  Closure = Function + [[Environment]]
  // [[Environment]] 指向创建时的作用域
  ```

* 垃圾回收

  ```js
  // 正常情况下，函数执行完毕被回收。
  // 闭包情况：不引用 -> 不回收，所以存在内存泄露风险。
  ```

**怎么用（应用场景与例子）**

* React Hooks本质就是闭包

* 闭包陷阱

  ```js
  useEffect(() => {
    setInterval(() => {
      console.log(count);
    }, 1000);
  }, []);
  // 永远打印旧址
  
  // 解决方案
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  
    return () => clearInterval(id);
  }, []);
  
  // 或者
  useEffect(() => {
    console.log(count);
  }, [count]);
  ```

* 模块化、封装

  ```js
  const createAPI = () => {
    const token = "xxx";
  
    return {
      getUser() {
        return fetch('/user', {
          headers: { Authorization: token }
        });
      }
    };
  };
  // token被闭包保护
  ```

* 防抖、节流

  ```js
  function debounce(fn, delay) {
    let timer;
  
    return function () {
      clearTimeout(timer);
      timer = setTimeout(() => fn(), delay);
    };
  }
  // timer被闭包持有
  ```

**有什么关系（联系、对比和局限）**

闭包和这些知识点强关联：

* 和作用域链：闭包 = 作用域链的延续

* 和执行上下文：闭包依赖执行上下文创建

* 和this对比

  | **对比** | **闭包** | **this** |
  | -------- | -------- | -------- |
  | 决定时机 | 定义时   | 调用时   |
  | 是否可变 | 不变     | 可变     |

* 和React关系： Hooks = 闭包 + 状态管理

* 和函数式编程：闭包是函数curry（柯里化）、高级函数的基础

**还能怎么扩展（创新、未来与批判）**

* 闭包 + 柯里化

  ```js
  function add(a) {
    return function(b) {
      return a + b;
    };
  }
  ```

* 闭包 + 函数组合

  ```js
  const compose = (f, g) => x => f(g(x));
  ```

* 闭包  + 状态机（高级设计）

  ```js
  function createStateMachine() {
    let state = "idle";
  
    return {
      dispatch(action) {
        if (action === "start") state = "running";
      },
      getState() {
        return state;
      }
    };
  }
  ```

* 闭包 + JSI/RN Bridge

  在RN中，闭包持有Native方法应用，本质也是闭包 + 引用保持

* 内存优化：

  ```js
  function bigClosure() {
    let bigData = new Array(1000000);
  
    return function() {
      console.log("use");
    };
  }
  // 即使 fn 不用 bigData, bigData 仍然不会释放
  // 手动释放
  bigData = null;
  ```



### 原型对象和原型链的理解

**是什么（定义与本质）**

原型：每个函数在创建时，都会自动拥有一个prototype对象。

```js
function Person() {}
console.log(Person.prototype);
// 输出
{
  constructor: Person
}
```

原型链：对象在查找属性或者方法时，沿着proto一层一层向上查找的链式结构。

```js
对象 obj
   ↓ __proto__
构造函数.prototype
   ↓ __proto__
Object.prototype
   ↓ __proto__
null（终点）
```

```js
function Person() {}
const p = new Person();

p.__proto__ === Person.prototype
Person.prototype.__proto__ === Object.prototype
```

**为什么（背景、目的和重要性）**

JS是基于原型的语言。（prototype-based）

解决问题：

* 实现继承

```js
Person.prototype.say = function() {
  console.log("hi");
};
// 示例共享
```

* 节省内存

```js
function Person() {
  this.name = "yx";
}
Person.prototype.say = function() {}
// 所有实例共享
```

* 动态扩展能力

```js
Array.prototype.myMap = function() {}
// 所有数据立即可用
```

**怎么来的（起源、发展与演化）**

* new做了什么

```js
const p = new Person();
// 等价于
const obj = {};
obj.__proto__ = Person.prototype;
Person.call(obj);
return obj;
// 对象链接到prototype
```

* 3个核心概念

```js
// prototype：函数的属性
Person.prototype

// proto: 对象的属性
p.__proto__

// constructor
Person.prototype.constructor = Person
```

**怎么工作的（原理、机制与过程）**

* 属性查找机制

```js
const p = new Person();
p.say();
// 查找顺序
1. p 自身有没有 say
2. p.__proto__（Person.prototype）有没有
3. 再往上 Object.prototype
4. 到 null 停止
// 这就是原型链查找
```

* 示例

```js
function Person() {}
Person.prototype.say = function() {
  console.log("hi");
};

const p = new Person();
p.say();

// 实际发生
p 没有 say
→ 找 Person.prototype
→ 找到 say
→ 执行
```

**怎么用（应用场景和例子）**

* 定义共享方法

```js
function Person(name) {
  this.name = name;
}

Person.prototype.say = function() {
  console.log(this.name);
};
```

* 实现继承

```js
function Animal() {}
Animal.prototype.eat = function() {};

function Dog() {}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
// 原型链：dog → Dog.prototype → Animal.prototype
```

* ES6 class本质

```js
class Person {
  say() {}
}
// 本质就是
Person.prototype.say = function() {}
```

**有什么关系（联系、对比和局限）**

* prototype vs proto

| **名称**  | **属于谁** |
| --------- | ---------- |
| prototype | 函数       |
| **proto** | 对象       |

`实例.__proto__ === 构造函数.prototype`

* 原型链 vs 作用域链

|      | **原型链**  | **作用域链** |
| ---- | ----------- | ------------ |
| 用途 | 属性查找    | 变量查找     |
| 方向 | 对象 → 原型 | 内 → 外      |
| 相关 | this        | 闭包         |

* 和this关系

```js
p.say();
this指向调用者p，即使方法来自prototype。
```

**还能怎么扩展（创新、未来与批判）**

* instanceof原理

```js
p instanceof Person
// 本质：检查 p.__proto__ 链上是否有 Person.prototype
```

* Object.create

```js
var obj = Object.create(proto) 
// 直接指定原型
```

* 原型污染

```js
Object.prototype.xxx = "bad";
// 所有对象受影响
```

* 性能优化

```js
链越长 → 性能越差
```

* React Native中的意义

RN虽然用函数组件为主，但JS引擎（Hermes、JSC）依然基于原型链，所有对象（Array/Object/Function）都依赖原型链。

> JavaScript 是基于原型的语言，每个对象都有一个内部属性指向其原型（**proto**），而函数有一个 prototype 属性用于构建这种关系。通过 new 创建对象时，会把实例的 **proto** 指向构造函数的 prototype。
>
> 当访问对象属性时，如果自身不存在，会沿着 **proto** 向上查找，直到 Object.prototype 或 null，这种链式查找结构就是原型链。
>
> 原型链的核心作用是实现对象之间的继承和方法共享，从而提高代码复用性并节省内存。



### this绑定call/apply/bind的理解

**是什么（定义与本质）**

this是函数执行时的调用上下文（调用者）。this不是在定义时决定的，而是在调用时决定的。

```js
function test() {
  console.log(this);
}
// this是谁看谁调用它
```

**为什么（背景、目的与重要性）**

this解决的问题：函数如何访问调用它的对象。

```js
const user = {
  name: "yx",
  say() {
    console.log(this.name);
  }
};
// this指向user，如果没有this，函数无法知道属于谁
```

**怎么来的（起源、发展与演化）**

this来自JS的调用机制：JS有4中绑定规则。

* 默认绑定

```js
function fn() {
  console.log(this);
}

fn();
// 浏览器：window
// 严格模式：undefined
```

* 隐式绑定

```js
const obj = {
  fn() {
    console.log(this);
  }
};

obj.fn();
// this = obj
```

* 显示绑定（call/apply/bind）

```js
fn.call(obj);
// 强制指定this
```

* new绑定

```js
function Person() {
  console.log(this);
}

new Person();
// this = 新对象
```

**怎么工作的（原理、机制和过程）**

this的本质：

```js
创建执行上下文
  ↓
确定 this 值
  ↓
执行函数
```

优先级

```js
new > bind > call/apply > 隐式绑定 > 默认绑定
```

* call/apply/bind

```js
fn.call(obj, arg1, arg2);
// 立即执行，参数一个个传

fn.apply(obj, [arg1, arg2]);
// 立即执行，参数是数组

const newFn = fn.bind(obj);
// 不执行，返回新函数，this永远绑定，参数一个个传
```

**怎么用（应用场景和例子）**

* 改变this指向

```js
function greet() {
  console.log(this.name);
}

greet.call({ name: "yx" });
```

* 借用方法

```js
Array.prototype.slice.call(arguments);
```

* React Native中常见场景

```js
class A {
  handle() {
    console.log(this);
  }
}

const fn = a.handle;
fn(); // this 丢失

// 解决方案
this.handle = this.handle.bind(this);
// 或者
handle = () => {
  console.log(this);
};

// 箭头函数没有自己的this，继承外层this
```

**有什么关系（联系、对比和局限）**

* this vs 闭包

|          | **this** | **闭包** |
| -------- | -------- | -------- |
| 决定时机 | 调用时   | 定义时   |
| 是否固定 | 不固定   | 固定     |
| 作用     | 指向对象 | 访问变量 |

* this vs 原型链

```js
p.say();
// 原型链查找方法，this指向调用者
```

* this vs 箭头函数

```js
const fn = () => {
  console.log(this);
};
// this来自外层作用域（闭包）
```

**还能怎么扩展（创新、未来与批判）**

* 手写call

```js
Function.prototype.myCall = function (context, ...args) {
  context.fn = this;
  const result = context.fn(...args);
  delete context.fn;
  return result;
};
```

* 手写bind

```js
Function.prototype.myBind = function (context) {
  const self = this;

  return function (...args) {
    return self.apply(context, args);
  };
};
```

* React Hooks中为什么不用this

函数组件无this，用闭包代替。

> this 是函数执行时的上下文对象，它的值是在函数调用时决定的，而不是定义时决定的。JavaScript 中主要有四种绑定规则：默认绑定、隐式绑定、显式绑定（call/apply/bind）以及 new 绑定，并且它们有明确的优先级。
>
> call 和 apply 会立即执行函数并指定 this，区别在于参数形式；而 bind 不会执行函数，而是返回一个永久绑定 this 的新函数。
>
> 在 React Native 中，由于函数组件没有 this，因此更多依赖闭包；而在类组件中，需要特别注意 this 丢失问题，通常通过 bind 或箭头函数解决。



### Promise / async await / 事件循环（宏任务、微任务）理解

**是什么（定义与本质）**

Promise是一个表示未来结果的对象。

三种状态：

```js
pending → fulfilled（成功）
        → rejected（失败）
```

async/await是Promise的语法糖，让异步代码看起来像同步代码。

```js
async function test() {
  const res = await fetchData();
}
// 本质还是promise
```

事件循环：JS用来处理异步任务的调度机制。

JS是：线程  + 非阻塞 + 事件循环

关键队列

```js
Call Stack（调用栈）
Microtask Queue（微任务）
Macrotask Queue（宏任务）
```

**为什么（背景、目的和重要性）**

JS为什么要异步？JS是单线程，如果同步 `while(true)`，页面卡死。

Promise解决：回调地狱（callback hell）

事件循环解决：如何调度异步任务执行顺序。

**怎么来的（起源、发展和演化）**

* Promise本质

  ```js
  new Promise((resolve, reject) => {})
  // 内部：维护状态 + 回调队列
  ```

* async/await本质

  ```js
  async function fn() {}
  // 本质
  function fn() {
    return Promise.resolve();
  }
  
  await x
  // 等价
  Promise.resolve(x).then(...)
  ```

* 事件循环结构

  ```js
  1. 执行同步代码（Call Stack）
  2. 清空微任务（Microtask）
  3. 执行一个宏任务（Macrotask）
  4. 再清空微任务
  5. 循环
  ```

**怎么工作的（原理、机制与过程）**

* 执行顺序规则

  ```js
  // 同步任务 > 微任务 > 宏任务
  console.log(1);
  
  setTimeout(() => console.log(2), 0);
  
  Promise.resolve().then(() => console.log(3));
  
  console.log(4);
  
  // 输出
  // 同步代码
  1
  4
  3 // 微任务
  0 // 宏任务
  ```

* async/await执行过程

  ```js
  async function test() {
    console.log(1);
    await Promise.resolve();
    console.log(2);
  }
  
  test();
  console.log(3);
  // 输出
  1， 3
  2
  
  // await xxx 本质
  暂停函数执行
  把后面的代码放入微任务执行
  ```

**怎么用的（使用场景与例子）**

* Promise链式调用

  ```js
  fetchData()
    .then(res => process(res))
    .then(data => console.log(data))
    .catch(err => console.log(err));
  ```

* async/await推荐

  ```js
  async function load() {
    try {
      const res = await fetchData();
      console.log(res);
    } catch (e) {
      console.log(e);
    }
  }
  ```

* 并发优化

  ```js
  ❌ 串行
  await a();
  await b();
  ✅ 并发
  await Promise.all([a(), b()]);
  ```

* RN中常见使用场景

  ```js
  •	网络请求
  •	AsyncStorage
  •	动画调度
  •	Bridge 通信
  ```

**有什么关系（联系、规避和局限）**

* Promise vs async/await

  |        | **Promise** | **async/await** |
  | ------ | ----------- | --------------- |
  | 本质   | 对象        | 语法糖          |
  | 写法   | then        | 同步风格        |
  | 可读性 | 一般        | 高              |

* Promise vs 事件循环

  ```js
  Promise的then属于微任务。
  ```

* setTimeout vs Promise

  ```js
  setTimeout → 宏任务
  Promise.then → 微任务
  // 所以
  Promise 一定比 setTimeout 先执行
  ```

* async/await vs 事件循环

  ```js
  await后，代码进入微任务队列
  ```

**还能怎么扩展（创新未来与批判）**

* 微任务有哪些

  ```js
  •	Promise.then
  •	queueMicrotask
  •	MutationObserver
  ```

* 宏任务有哪些

  ```js
  •	setTimeout
  •	setInterval
  •	setImmediate（Node）
  •	UI 渲染
  ```

* Node vs 浏览器事件循环

  Node 有多个阶段（timers、poll 等）

* RN中的事件循环

  ```js
  JS 线程（事件循环）
  ↓
  Bridge / JSI
  ↓
  Native 线程
  // Promise在JS线程执行，微任务优先级最高。
  ```

> JavaScript 是单线程的，通过事件循环机制实现异步。代码执行时会先执行同步任务，然后依次清空微任务队列（如 Promise.then），再执行宏任务（如 setTimeout），循环往复。
>
> Promise 是对异步操作的封装，内部维护状态并通过 then 注册回调，这些回调会进入微任务队列执行。
>
> async/await 是 Promise 的语法糖，本质上是将代码拆分成多个 Promise.then，其中 await 会暂停当前函数，并将后续逻辑放入微任务队列。
>
> 在执行顺序上，微任务优先于宏任务，这也是很多面试题考察的重点。



### TypeScript和JavaScript区别理解

* TypeScript是JavaScript的超集。

* 增加了：

  * 静态类型检查

  * 类型推导

  * 编译阶段类型检查

* 最终会编译为JavaScript执行

* TS是开发时工具，运行时依然是JS

* 可以提高大型项目的可维护性



### 什么是类型推导

```js
let a = 10 // 自动推导为number类型
```

TS会根据上下文自动推断类型，减少冗余声明。

当使用any，复杂泛型以及函数值不明确的情况会推导失败。



### interface vs type区别

interface适合定义对象结构，type更灵活。

|          | **interface**  | **type**               |
| -------- | -------------- | ---------------------- |
| 扩展     | extends        | &                      |
| 合并     | 支持声明合并 ✅ | 不支持 ❌               |
| 使用场景 | 对象结构       | 更灵活（联合、函数等） |

```js
interface A { name: string }
interface A { age: number } // 合并

type B = { name: string }
type C = B & { age: number }
```



### any/unknown/never区别

any：放弃类型检查（不安全）

unknown：安全版any（使用前必须先判断）

```js
let a: unknown;
a.toString(); ❌
```

never：永远不会发生的值

```js
function error(): never {
  throw new Error();
}
// never常用于穷尽检查
```



### extends在TS中的作用

**继承**

```js
interface B extends A {}
```

**泛型约束**

```js
function fn<T extends { name: string }>(obj: T) {}
```

**条件类型**

```js
type IsString<T> = T extends string ? true : false;
```



### TypeScript中泛型理解

**是什么**

泛型（Generics） = 类型参数化

```js
function add<T>(a: T, b: T): T
// T = 类型变量
// 本质：泛型让类型在使用时再决定。
```

**为什么**

* 解决类型复用问题

  ```js
  function identity<T>(val: T): T {
    return val;
  }
  ```

* 保持类型信息

  ```js
  function identity<T>(val: T): T {
    return val;
  }
  // 保持输入和输出一致
  ```

* 提高类型安全

  ```js
  identity<number>("abc"); // ❌ 报错
  // 编译期检查
  ```

**怎么来的**

* 泛型本质来源

  ```js
  类型系统 + 参数化思想
  // 类似其他语言
  •	Java：<T>
  •	Swift：<T>
  •	C++：template
  ```

* TS泛型的核心能力

  ```js
  1. 类型参数
  2. 类型推断
  3. 类型约束
  ```

**怎么工作的**

* 类型在编译时决定

  ```js
  静态类型系统（编译时），泛型会被擦除。
  function identity<T>(val: T): T {
    return val;
  }
  // 编译后JS
  function identity(val) {
    return val;
  }
  // 泛型只存在于类型层（不影响运行时）
  ```

* 类型推断

  ```js
  identify(123)
  TS会自动推断 T = number
  通常不需要手写 <T>
  ```

* 泛型约束

  ```js
  function logLength<T extends { length: number }>(val: T) {
    console.log(val.length);
  }
  // T 必须有 length 属性
  ```

**怎么用**

* 泛型函数

  ```js
  function getFirst<T>(arr: T[]): T {
    return arr[0];
  }
  ```

* 泛型接口

  ```js
  interface ApiResponse<T> {
    data: T;
    code: number;
  }
  // 使用
  const res: ApiResponse<User> = {
    data: { name: "yx" },
    code: 200
  };
  ```

* 泛型类

  ```js
  class Box<T> {
    value: T;
  
    constructor(val: T) {
      this.value = val;
    }
  }
  ```

* RN常见场景

  ```js
  // useState
  const [count, setCount] = useState<number>(0);
  // 本质
  function useState<T>(initial: T): [T, (v: T) => void]
                                     
  // API请求封装
  async function request<T>(url: string): Promise<T> {
    const res = await fetch(url);
    return res.json();
  }
  // 使用
  const user = await request<User>("/user");
  
  // 列表组件
  type ListProps<T> = {
    data: T[];
    renderItem: (item: T) => ReactNode;
  };
  ```

**有什么关系**

* 泛型 vs any

  |          | **泛型** | **any** |
  | -------- | -------- | ------- |
  | 类型安全 | ✅        | ❌       |
  | 类型保留 | ✅        | ❌       |

* 泛型 vs unknown

  unknown更安全

* 泛型 vs extends

  用于约束：<T extends Base>

* 泛型 vs keyof

  ```js
  function get<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
  }
  ```

**还能怎么扩展**

* 条件类型

  ```js
  type IsString<T> = T extends string ? true : false;
  ```

* 映射类型

  ```js
  type Readonly<T> = {
    readonly [K in keyof T]: T[K];
  };
  ```

* 工具类型（本质都是泛型）

  - Partial<T>
  - Pick<T, K>
  - Record<K, T>

* 泛型 + 函数式编程

  ```js
  function map<T, U>(arr: T[], fn: (x: T) => U): U[]
  ```

* 泛型设计能力

  ```ts
  type Result<T, E> =
    | { ok: true; data: T }
    | { ok: false; error: E };
  ```

> 泛型是 TypeScript 中对类型进行参数化的一种机制，它允许我们在定义函数、接口或类时不指定具体类型，而是在使用时再传入，从而实现类型复用和类型安全。
>
> 泛型的核心价值在于既避免了使用 any 带来的类型丢失问题，又能保持类型之间的关联关系，例如输入和输出类型一致。
>
> 在实际开发中，泛型广泛应用于函数封装、API 请求、React Hooks（如 useState）以及工具类型中，是构建可复用、可扩展类型系统的基础。



## React

### forwardRef和useImperativeHandle理解

在React里，数据是单向流动的：父 -> 子。但有些场景，必须反向控制组件。

- 父组件控制输入框 focus
- 打开/关闭子组件（Modal / BottomSheet）
- 调用子组件内部方法
- 获取子组件状态

**forwardRef**

允许父组件把ref传递给子组件。让函数组件可以接受Ref。帮你把ref传进去。

注意：React 的 `forwardRef` 默认不会自动暴露任何方法，需要显式用 `useImperativeHandle` 声明

```react
// 默认情况
function MyInput() {
  return <TextInput />;
}

const ref = useRef();
<MyInput ref={ref} />;
// 这种是拿不到ref的，函数组件默认没有ref
// 因为函数组件是没有实例，不像class component，React不希望父组件随便访问子组件内部状态

// 解决方案
import { forwardRef } from 'react';
import { TextInput } from 'react-native';

const MyInput = forwardRef((props, ref) => {
  return <TextInput ref={ref} {...props} />;
});

const inputRef = useRef(null);

<MyInput ref={inputRef} />;

// 可以直接操作
inputRef.current.focus();
```

**useImperativeHandle**

自定义暴露给父组件的ref内容。决定ref暴露什么能力。

```react
// 默认情况
forwardRef((props, ref) => {
  return <TextInput ref={ref} />;
});

// 父组件拿到的是ref.current === TextInput 实例，但有时候不想全部暴露，只想暴露部分方法。
import { forwardRef, useImperativeHandle, useRef } from 'react';
import { TextInput } from 'react-native';

const MyInput = forwardRef((props, ref) => {
  const innerRef = useRef<TextInput>(null);

  useImperativeHandle(ref, () => ({
    focus: () => {
      innerRef.current?.focus();
    },
    clear: () => {
      innerRef.current?.clear();
    }
  }));

  return <TextInput ref={innerRef} {...props} />;
});
// 父组件
const ref = useRef(null);

<MyInput ref={ref} />;

ref.current.focus();
ref.current.clear();
```

**使用场景**

* 控制Input

```react
// 子组件
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus()
  }));

  return <TextInput ref={inputRef} />;
});

// 父组件
const ref = useRef(null);

<CustomInput ref={ref} />

<Button onPress={() => ref.current.focus()} />
```

* 控制BottomSheet/Modal

```react
const MyBottomSheet = forwardRef((props, ref) => {
  const sheetRef = useRef(null);

  useImperativeHandle(ref, () => ({
    open: () => sheetRef.current?.present(),
    close: () => sheetRef.current?.dismiss(),
  }));

  return <BottomSheetModal ref={sheetRef} />;
});

// 父组件
const sheetRef = useRef(null);
<MyBottomSheet ref={sheetRef} />
sheetRef.current.open();
```

* 复杂组件（封装API）

```react
ref.current.submit();
ref.current.validate();
ref.current.reset();

很适合：
	•	表单组件
	•	聊天输入框
	•	上传组件
```

**React 的 `forwardRef` 默认不会自动暴露任何方法，需要显式用 `useImperativeHandle` 声明**

```tsx
// 子组件
const MyModal = forwardRef((props, ref) => {
  const open = () => {
    console.log('open');
  };

  return <View />;
});

// 父组件
const modalRef = useRef(null);

<MyModal ref={modalRef} />

// 然后
modalRef.current.open() // undefined is not a function

// 因为你没有告诉ref.current 应该暴露什么。
// useImperativeHandle的作用：
// 显示定义ref对外暴露的API
useImperativeHandle(ref, () => ({
  open,
  close,
}));
// 父组件只能拿到
- open
- close
```





## 其他

### 如何升级expo项目

**升级expo sdk**

```shell
npx expo install expo@latest
```

**升级依赖**

```shell
npx expo install --fix
# 自动修复依赖
```



### 初始化项目过程理解

**整理项目架构考虑**

* 常用架构模式

  对于Expo + React Native项目，主流推荐以下两种（或混合）。

  1. Feature-Sliced Design（功能切片/Feature First）:最适合中大型项目。按业务功能划分（auth、profile、feed等），每个feature自包含（components、hooks、services、types等），高度解耦、可复用、可测试。
  2. Clean Architecture/Layer Architecture： 分层（Presentation/Domain/Data），常与feature结合使用。UI层纯展示，业务逻辑抽到hooks/services，数据层用React Query / TanStack Query。
  3. 类型分组（Type-based）：小项目常用（components、screens、hooks统一放），但规模变大后容易混乱，不推荐长期使用。

  最新架构推荐：Feature-Sliced + Expo Router文件路由结合Clean Architecture思想。

* 项目文件组织推荐（强烈建议使用src包裹）

  Expo Router支持把路由文件放在根目录的app/，src/app（只需要移动文件夹并重启即可）

  官方和主流推荐：把所有源代码放在src/下，路由放在src/app。这样能清晰分离路由文件和其他文件，便于维护和AI理解项目规则。

  推荐完整结构（适合中大型项目）

  ```shell
  my-expo-app/
  ├── assets/                  # 图片、字体、svg 等静态资源
  ├── scripts/                 # 构建、生成等脚本（可选）
  ├── src/
  │   ├── app/                 # Expo Router 文件路由（不要随意改动这里的 _layout.tsx）
  │   │   ├── _layout.tsx      # 根布局（Stack / Tabs 等）
  │   │   ├── (tabs)/          # 分组路由（括号表示不影响 URL）
  │   │   │   ├── _layout.tsx
  │   │   │   ├── home.tsx
  │   │   │   └── profile.tsx
  │   │   ├── auth/
  │   │   │   ├── login.tsx
  │   │   │   └── register.tsx
  │   │   ├── index.tsx        # 首页（/）
  │   │   └── +not-found.tsx   # 404
  │   ├── features/            # 核心：按业务功能划分（推荐）
  │   │   ├── auth/
  │   │   │   ├── api/         # API 请求（React Query mutations/queries）
  │   │   │   ├── components/  # 该功能专用的 UI 组件
  │   │   │   ├── hooks/       # 业务 hooks（如 useLogin）
  │   │   │   ├── services/    # 纯业务逻辑（不含副作用）
  │   │   │   ├── store/       # 局部 Zustand / Jotai store（可选）
  │   │   │   └── types.ts
  │   │   ├── feed/
  │   │   └── profile/
  │   ├── entities/            # 共享业务实体（User、Post 等纯类型 + 工厂函数）
  │   ├── shared/              # 全局可复用
  │   │   ├── ui/              # 设计系统组件（Button、Card、Theme 等）
  │   │   ├── lib/             # 工具函数（date、format、validation）
  │   │   ├── api/             # 基础 API client（axios instance 等）
  │   │   ├── hooks/           # 通用 hooks（useDebounce、useTheme）
  │   │   └── constants.ts
  │   ├── widgets/             # 复杂组合组件（Header + Search + List）
  │   ├── processes/           # 跨 feature 的复杂流程（登录流程、支付流程）
  │   ├── hooks/               # 全局 hooks（如果不放 shared）
  │   ├── utils/               # 纯工具函数
  │   └── providers/           # 全局 Provider（QueryClient、Theme、Auth 等）
  ├── app.json                 # Expo 配置
  ├── eas.json                 # EAS Build 配置（可选）
  ├── package.json
  ├── tsconfig.json
  └── metro.config.js          # Metro 配置（可选）
  ```

  

### Tailwind CSS / Uniwind / Nativewind理解

这几个库的主要作用是把web上的Tailwind思维搬到React Native中。

**三个库分别做什么的**

* Tailwind CSS：本质是一套原子化CSS设计系统。

```js
// 在web中你会这样写
<div class="flex items-center justify-center bg-blue-500 p-4">
// 而不是
.container {
  display: flex;
  justify-content: center;
}
// 核心特点
- 用类名替代样式
- 原子化（一个class只做一件事）
- 快速开发UI
- 注意：Tailwind本身不能直接作用于React Native，因为RN没有CSS。
```

* Uniwind本质是Tailwind CSS在React Native中的运行时实现。可以简单理解把Tailwind className翻译为React Native中的style。

```js
<View className="flex-1 justify-center items-center bg-red-500" />
  
// Uniwind会在运行时帮你转成
<View style={{
  flex: 1,
  justifyContent: 'center',
  alignItems: 'center',
  backgroundColor: '#ef4444'
}} />

// 特点：
- 支持Tailwind v4
- 运行时解析（runtime）
- 更灵活（动态className）
- 性能略低于编译时方案
- 需要runtime解析
```

* Nativewind本质是Tailwind for React Native(编译时优化版本)

和Uniwind最大区别

| **方案**   | **原理**                |
| ---------- | ----------------------- |
| Uniwind    | 运行时解析              |
| NativeWind | **编译时转换（Babel）** |

```js
// NativeWind会把
<View className="flex-1 bg-red-500" />
// 编译成
<View style={styles.xxx} />
  
// 特点
- 性能更好
- 更接近RN原生
- 社区更成熟
- 配置稍复杂
- 动态class支持有限
```

**三者关系总结**

```js
Tailwind CSS（规范）
   ↓
NativeWind / Uniwind（实现）
```

| **角色**             | **类比**               |
| -------------------- | ---------------------- |
| Tailwind             | JavaScript 语法        |
| NativeWind / Uniwind | JS 引擎（V8 / Hermes） |

**uniwind配置步骤**

* 安装tailwind和uniwind

```shell
npm install uniwind tailwindcss
```

* 创建`global.css`文件

```css
@import 'tailwindcss';
@import 'uniwind';
```

* 导入`global.css`文件到 `app/_layout.tsx`

```tsx
import { Stack } from 'expo-router';

import '../global.css';

export default function RootLayout() {
  return <Stack />;
}
// 这里需要注意：不要把global.css文件从注册根组件的文件index.ts/index.js导入，因为任何改变都会使得不是热重载而是触发整个重载。
```

`npx skills add uni-stack/uniwind`作用

其本质就是安装一个AI Skill技能包，让AI更懂Uniwind、Tailwind RN项目。给 AI 安装 Uniwind 使用说明书。

* 什么是skill

Skill是给AI（Claude、Cursor、Agent）用的“提示工程  + 规则 + 知识包”。把团队经验注入AI。

| **类型** | **作用**       |
| -------- | -------------- |
| npm 包   | 给程序运行     |
| Skill    | 给 AI 理解项目 |

```shell
npx skills add uni-stack/uniwind
// 让AI
- 让AI更会写className
- 知道Uniwind的最佳实践
- 避免写错tailwind类
```

* 这个命令会生成一下文件

```shell
.agents/
.claude/
skills-lock.json

// .claude/skills/uniwind/skill.md 
给 Claude 系列 Agent（官方） 用的，
比如：
* Claude Code（Anthropic CLI）
* Claude Dev / Claude Agent
特点：
* 有 skill loader（自动加载）
* 支持 trigger（description 触发）
* 会较严格遵守规则
在 Claude 生态 → 这个是“真·规则系统”.
这是一个AI提示词模版
这是最重要的文件
里面通常包含：
•	Tailwind RN 使用规范
•	Uniwind 的写法约束
•	推荐写法
•	禁止写法
•	示例代码
当你在Cursor、Cluade里写代码时，AI会自动参照这些规则。

// .agent/skills/uniwind/skill.md 
给 通用 AI Agent / IDE 插件 用的.
比如：
* Cursor（部分支持）
* Windsurf
* OpenAI Agents
* 自定义 agent framework
特点：
* 没有统一标准
* 只是“约定俗成”
* 更像：📄“知识库文件”
可以理解为AI的执行脚本配置。
给Agent用的（更自动化的AI）
•	自动改代码
•	自动生成 UI
•	自动 refactor

// skills-lock.json
类似package-lock.json
作用：
•	锁定 skill 版本
•	保证团队一致
```

* Cursor会自动用这些规则吗

不会完全自动生效。

Cursor行为：

| **文件**       | **是否自动用**         |
| -------------- | ---------------------- |
| .claude/skills | ✅ 会参考（但不是强制） |
| .agents        | ❌ 需要触发 agent       |
| 普通代码       | ❌ 不一定遵守           |

也就是他只是增强AI，而不是控制AI。

* 真正的作用

```tsx
// 统一团队代码风格
// ❌ AI 乱写
<View style={{ padding: 10 }} />

// ✅ Skill 会引导成
<View className="p-2.5" />

// 减少AI写错代码
例如：
	•	不支持的 Tailwind 类
	•	RN 不支持的 CSS
  
// 提升AI输出质量
没有 skill：
AI 会：
	•	Web Tailwind 写法 ❌
	•	RN style 混用 ❌

有 skill：
AI 会：
	•	正确 className ✅
	•	符合 Uniwind ✅
```

* 如何使用

```shell
// 不需要手动修改skill，除非需要定制ai行为，写团队规范
// 正确用法：
在Cursor里面写：做一个登录按钮，使用 Uniwind 风格
AI 会：
	•	自动参考 skill.md
	•	按项目规范生成代码
// 如果AI写错
可以：Follow uniwind skill rules
```

**如何配置样式主题（theme）**

* 方式一

在`global.css`文件中定义

```css
@import 'tailwindcss';
@import 'uniwind';

@theme {
  /* Fonts — names must exactly match the font file name (no extension) */
  --font-sans: 'SourceSans3_400Regular';
  --font-sans-medium: 'SourceSans3_500Medium';
  --font-sans-semibold: 'SourceSans3_600SemiBold';
  --font-sans-bold: 'SourceSans3_700Bold';

  /* Colors */
  --color-primary: #242f65;
  --color-primary-200: #a1adce;
  --color-secondary: #455b9e;
  --color-error: #d71414;
  --color-neutral: #e8e8e8;
  --color-neutral-200: #e8e8e8;
  --color-neutral-400: #a5a5a5;
  --color-neutral-600: #575757;
  --color-gray: #333333;
}
```

```tsx
<Text className="text-primary text-xl md:text-2xl">{t('title')}</Text>
```



### Husky理解

Husky帮助你在`git commit/push`前自动执行检查任务。

* 代码格式化（Prettier）
* 代码规范检查（ESLint）
* 单元测试（Jest）
* 类型检查（TypeScript）
* 阻止不符合规范的代码提交

使用Husky后：

* 本地直接拦截错误（比CI更早）
* 自动格式化代码
* 保证提交质量
* 提升团队协作效率

**核心概念**

Git hooks有几个关键阶段：

| **hook**   | **触发时机** |
| ---------- | ------------ |
| pre-commit | commit 前    |
| commit-msg | 提交信息检查 |
| pre-push   | push 前      |

Husky就是帮你管理这些hooks的。

**Git Hook本质**

Git本身就支持hook机制。

```shell
在项目里有个隐藏目录：.git/hooks
里面会有这些文件
pre-commit
commit-msg
pre-push
post-merge
...

本质就是git在执行某些操作室，会自动这些脚本文件。
例如：git commit
Git 内部流程：
1.	执行 .git/hooks/pre-commit
2.	执行 commit
3.	执行 .git/hooks/commit-msg
```

**Husky做了什么**

Husky帮你更优雅的管理`.git/hooks`。

因为原生 hooks 有几个问题：

- 不好管理（每人本地不同）
- 不会跟代码一起提交
- 写起来麻烦

Husky做了两件事情：

* 把hooks放到项目里

```shell
.husky/
  ├── pre-commit
  ├── commit-msg
```

* 告诉Git，去执行.husky里的脚本

git里有一个配置：

```shell
git config core.hooksPath
// 默认是.git/hooks

Husky初始化时：npx husky init
执行：git config core.hooksPath .husky
结果：Git 不再看 .git/hooks，而是去 .husky/
也就是执行git commit 后， Git会执行 .husky/pre-commit
```

**npx husky作用**

```shell
{
  "name": "myapp",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "prepare": "husky",
  },
}
```

执行`npm install`或者`pnpm install`会自动执行prepare。等价于`npx husky`。然后会执行`husky install`

`husky install`做了两件事情：

* 设置hookPath

```shell
git config core.hooksPath .husky
```

* 创建内部执行脚本

```shell
.husky/_/husky.sh
```

**每个hook阶段做什么**

* pre-commit（最常用）

```shell
// 触发时机git commit
// 用途
	•	eslint
	•	prettier
	•	lint-staged
// 例如
pnpm lint-staged
```

* commit-msg

```shell
// 触发时机： git commit -m "xxx"
// 用途：校验 commit message（规范化）
// 示例
feat: add login
fix: crash bug
```

* pre-push

```shell
// 触发： git push
// 用途
	•	跑测试
	•	防止坏代码推上去
```

**配置流程**

* 安装依赖

```shell
npm install husky -D
```

* 初始化Husky

```shell
npx husky init

// 执行后会生成
.husky/
  └── pre-commit
// 并且package.json会多出
"scripts": {
  "prepare": "husky"
}
// 这个prepare很重要，install后会自动启用husky
```

* 配置pre-commit

```shell
// 编辑：.husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

pnpm lint
pnpm test

提交代码前自动执行：
	•	lint
	•	test
如果失败 👉 commit 会被阻止
```

* 接入ESLint + Prettier

```shell
pnpm add eslint prettier lint-staged -D
```

* 配置lint-staged（关键优化）

```shell
pnpm lint-staged
```

```shell
// package.json
"lint-staged": {
  "*.{js,ts,tsx}": [
    "eslint --fix",
    "prettier --write"
  ]
}
```

* 修改 `pre-commit`

```shell
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

pnpm lint-staged

// ESLint: 代码质量检查（逻辑层）bug + 规范
// Prettier: 代码格式化（样式层）统一代码风格
// lint-staged: 性能优化，不想每次commit都检查整个项目。

// 流程：
执行git commit
- 触发.husky/pre-commit
- 执行lint-staged：pnpm lint-staged
- lint-staged找到暂存文件
- 对这些文件执行
	- eslint --fix
	- prettier --write
```

**Prettier 和 ESLint配置**

通常使用`npx create-expo-app`后，eslint已经配置好了，这个时候只需要安装prettier就好。

**核心目标**

你接入 Prettier 的目的不是“能用就行”，而是：

1. ✅ 自动格式化代码（统一风格）
2. ✅ 和 ESLint 不冲突
3. ✅ 可以接入 Husky + lint-staged（提交自动格式化）

**配置步骤**

* 安装依赖

```shell
pnpm add -D prettier eslint-config-prettier
```

* 配置prettier

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 80,
  "tabWidth": 2
}

//.prettierignore
node_modules
.expo
dist
build
coverage

// 注意：这两个文件不配置也有默认的配置
```

* 修改ESLint配置

```js
module.exports = {
  extends: [
    'expo',
    'plugin:prettier/recommended' // 👈 加这一行
  ],
};

它做了三件事：
	1.	启用 prettier
	2.	关闭 ESLint 和 prettier 冲突的规则
	3.	把 prettier 错误当成 eslint 错误

// 实际上第3点现在一般不推荐，而且我们上一步也没有安装eslint-plugin-prettier
// eslint-plugin-prettier 的作用是:把 Prettier 当成 ESLint 规则来运行
  ESLint 会：
	•	检查语法问题 ✅
	•	同时检查格式问题（Prettier）✅

// 所以改为
module.exports = {
  extends: [
    'expo',
    'prettier' // 👈 只关闭冲突规则
  ]
};
```



### npx 和 npm的理解

npm：安装/管理包

npx：直接运行包（不用安装或用本地的）

**npm是什么**

nodejs的包管理工具。

* 安装依赖
* 管理依赖版本
* 执行脚本

**npx是什么**

一个执行工具，专门用来运行npm包里面的命令。

* 直接运行本地`node_modules`里的命令

```shell
//例如： npx eslint .
//等价于：./node_modules/.bin/eslint .
```

* 临时下载并执行（不用安装）

```shell
npx create-expo-app myApp

过程：
	1.	下载 create-expo-app
	2.	执行
	3.	用完就删
```

* 指定版本运行

```shell
npx eslint@8 .
```

**npm vs npx**

| **工具** | **作用** |
| -------- | -------- |
| npm      | 安装包   |
| npx      | 执行包   |

**npx使用场景**

* 创建项目（常见）

```shell
npx create-expo-app
npx create-react-app
```

* 执行工具（eslint、prettier）

```shell
npx eslint .
npx prettier --write .
```

* 初始化工具

```shell
npx husky init
npx tailwindcss init
```

* 一次性工具

```shell
npx cowsay hello
// 临时运行
```

**npm使用场景**

* 安装依赖

```shell
npm install axios
pnpm add react-query
```

* 运行scripts

```shell
npm run dev
pnpm run build
```



### 测试流程理解

**测试分类**

1. 单元测试

主要用于测试函数、逻辑。比如：

* utils方法
* 数据处理函数
* validation

核心思想：输入 → 输出是否符合预期

2. 组件测试

主要用于测UI + 交互；比如：

* 点击按钮是否触发函数
* 表单输入是否更新

* 错误提示是否显示

3. E2E测试

主要用于验证整个App流程（登录-下单-支付）

工具一般用Detox、Maestro

**React Native主流测试工具**

1. 测试框架： Jest（核心）

核心角色：测试运行器。

它负责：

- 执行测试文件（*.test.ts）
- 提供 API（describe / it / expect）
- mock 能力（jest.fn / jest.mock）
- 生成覆盖率

可以理解为

```shell
Jest = 测试引擎
```

2. jest-expo

核心角色：环境适配器（preset）

React-Native/Expo不能直接跑在Node里。

jest-expo做的事情：

* mock原生模块（Camera、Device、etc）
* 配置Babel转译RN代码 
* 让Jest能理解Expo项目

可以理解为：
```shell
jest-expo = 让 Jest 能跑 React Native
```

3. 组件测试库：React Native Testing Libray

核心角色：UI测试工具

它提供：

* render（渲染组件）
* fireEvent（模拟点击）
* query（查找元素）

重点理解：用用户视角测试UI，而不是测实现。

4. @testing-library/jest-native

核心角色：增强断言

```js
// 没有时
expect(text.props.children).toBe('Hello');

// 安装后
expect(getByText('Hello')).toBeTruthy();
expect(getByText('Hello')).toBeOnTheScreen();
```

它让语义变得

* 更语义化
* 更像人类语言

**安装配置**

* 安装

```shell
pnpm install -D jest jest-expo @testing-library/react-native @testing-library/jest-native @types/jest
// 注意：npm 对依赖间的冲突不友好。pnpm和yarn更方便。
// Yarn 的依赖解析算法对 peer dependency 冲突有更好的容错能力，很多情况下能自动解决这个问题
// pnpm 提供更严格的依赖隔离机制，能有效减少不同包之间的 peer dependency 冲突
```

* 创建jest配置文件

```shell
touch jest.config.js
# 配置如下
/** @type {import('jest').Config} */
module.exports = {
	// 使用 Expo 官方测试环境（必须）
  preset: 'jest-expo',

  // 每个测试前执行
  setupFilesAfterEnv: ['<rootDir>/src/test-utils/setup.ts'],

  // 支持路径别名（可选但推荐）
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^react-native-pager-view$': '<rootDir>/__mocks__/react-native-pager-view.tsx',
  },

  // RN 必备配置（避免报错）
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|native-base|react-native-svg)',
  ],

  // 覆盖率
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!**/node_modules/**',
  ],

  coverageReporters: ['text', 'lcov'],

  testPathIgnorePatterns: ['/node_modules/', '/android/', '/ios/'],
};
```

>`jest.config.js`
>
>**ModuleNameMapper**
>
>作用：
>
>当测试代码（不管是直接还是间接）`import ... from 'react-native-pager-view'` 时，**Jest 不去 `node_modules` 加载真实模块，改去加载 `__mocks__/react-native-pager-view.tsx`**。
>
>`^...$` 是正则锚定 —— 必须**精确等于** `react-native-pager-view`，避免把 `react-native-pager-view-extras` 之类的子包也劫持掉。
>
>流程：
>
>测试里 `import { ShiftImageCarousel } ...` → 组件文件 `import PagerView from 'react-native-pager-view'` → **Jest 命中 `moduleNameMapper`** → 拿到 `__mocks__/react-native-pager-view.tsx` 导出的假 `View` → 渲染时 `onPageSelected` 留在 props 里 → `fireEvent(..., 'pageSelected', ...)` 把它当回调直接调起来 → 业务逻辑被验证。
>
>不是所有三方模块都要加。`moduleNameMapper` 只是**最后兜底的强替换手段**，项目里不同的三方模块走的是不同档位的"在 Jest 下让它能跑"策略。
>
>1. 档位1
>
>   `jest.config.js` 里：
>
>   ```js
>   preset: 'jest-expo',
>   ```
>
>   `jest-expo` 本身就内置了一大堆 mock（`expo-modules-core`、`expo-font`、`expo-asset`、`expo-image`、`react-native` 自带的 native modules、`Animated` 等）。
>
>   这层免费送，所以你看不到这些包的映射也不用奇怪。
>
>2. 档位2：`transformIgnorePatterns` 放行 + 包本身能在 Node 里跑 → 也不用 mock
>
>   默认 Jest **不转译** `node_modules`。但 RN 生态大量包是 ESM 或带 Flow/TS 语法发布的，不转译就 `SyntaxError`。所以你看到这条很长的白名单：
>
>   ```js
>   transformIgnorePatterns: [
>     'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|native-base|react-native-svg)',
>   ],
>   ```
>
>   这条正则的意思是 **"括号里这些包要被转译，其他 node_modules 跳过"**。`react-navigation` / `@react-navigation/*` / `@sentry/react-native` 都在白名单里，被 Babel 转完之后是合法 JS，能直接在 Node 里 `require`，所以**不需要 mock**。
>
>   这是为什么很多 RN 包"既没在 `__mocks__/` 出现，也没在 `moduleNameMapper` 出现，但测试就是能跑"。
>
>3. 档位3：包自带 `jest-setup` 或文档要求一行 `jest.mock(...)` → 走 setup 文件
>
>   看 `src/test-utils/setup.ts`：
>
>   ```ts
>   jest.mock('@react-navigation/native');
>   jest.mock('expo-router');
>   jest.mock('uniwind');
>   jest.mock('react-native-keyboard-controller');
>   jest.mock('@expo/vector-icons');
>   jest.mock('react-native-mmkv');
>   ```
>
>   `jest.mock('xxx')` 不带工厂函数时，Jest 会去找：
>
>   1. **`__mocks__/xxx.ts(x)`**（项目根目录手写的 manual mock），或者
>   2. 包自身导出的 `__mocks__/xxx.js`（少数库自己提供）
>
>   所以这些**都已经在 `__mocks__/` 里有对应文件**：`__mocks__/uniwind.ts`、`__mocks__/react-native-mmkv.ts`、`__mocks__/@expo/vector-icons.tsx`、`__mocks__/react-native-keyboard-controller.tsx`…… 文件名跟包名对得上，Jest 自动找到。
>
>   `@react-navigation/native` 和 `expo-router` 是个例外——它们 npm 包里自带了 `jest-setup.js` 或 `jest/setup.js`，`jest.mock(...)` 能直接命中官方维护的替身，不用我们写文件。
>
>4. 档位4：包名不能直接做合法文件路径，或要劫持子模块 → 这才用 `moduleNameMapper`
>
>   `moduleNameMapper` 是**正则映射 → 任意路径**，能力比 manual mock 更强。它存在的理由通常是这几个：
>
>   | 场景                                                   | 例子                                                         | 为什么不能用前面的档位                                       |
>   | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
>   | 把整包替换成 ts/tsx 替身（不止 js）                    | `react-native-pager-view` → `__mocks__/react-native-pager-view.tsx` | 默认 manual mock 解析顺序对 ESM/TSX 边界容易出问题，显式映射最稳 |
>   | 处理 `transformIgnorePatterns` 之外、又含 ESM 语法的包 | `react-native-reanimated`、`react-native-worklets`           | 不映射的话默认会被 Babel 跳过转译，直接 `Unexpected token export` |
>   | 路径别名                                               | `^@/(.*)$` → `<rootDir>/src/$1`                              | 不是替换实现，是把别名翻译成真实路径                         |
>   | 把整个包替换成几行 stub                                | `react-native-country-flag`                                  | 真实组件依赖 native 资源，stub 一个 `<View />` 就够测试用    |
>
>   `pager-view` 用 `moduleNameMapper` 而不是放进 `setup.ts` + `__mocks__/`，主要是为了**TSX 替身的解析路径在所有 Jest 环境下都明确无歧义**——这条映射 = "凡是 import `react-native-pager-view`，立刻拿这个文件，别再走默认解析"。
>
>5. 决策树
>
>   1. 装好包，**先什么都不加直接跑测试**。
>   2. 报错是 `SyntaxError: Unexpected token` / `Cannot use import outside a module` → 把包名加进 `transformIgnorePatterns` 的白名单（档位 2）。
>   3. 报错是 `TurboModuleRegistry.getEnforcing(...)` / `null is not an object`（即真 native 调用）→ 写一个 `__mocks__/<包名>.ts`，在 `setup.ts` 里 `jest.mock('<包名>')`（档位 3）。
>   4. 上面两个都搞不定，或者你需要的是**TSX 替身 / 子路径劫持 / 路径别名** → 才上 `moduleNameMapper`（档位 4）。
>
>   简单说：**能不写就不写**，越靠后的档位侵入性越强、越容易让测试跟生产行为脱节。`sentry` / `react-navigation` 不出现在 `moduleNameMapper` 里，是因为它们在档位 1 或 2 已经解决了——不是漏配。

* 配置ts支持jest

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "types": ["jest"], // 让TypeScript认识Jest的全局函数（describe、it，test，expect）
    "paths": {
      "@/*": ["./src/*"],
      "@/assets/*": ["./assets/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts", "./uniwind-types.d.ts"]
}
```

* 创建初始化文件

```shell
mkdir -p src/test-utils
touch src/test-utils/setup.ts

# 建议先做这三类：
# afterEach 清理 timer / mock
# mock 你项目里不好测的原生模块
# mock 样式库 HOC（如需要）
```

```shell
// setup.ts内容
// 1 扩展 jest 断言（必须）
import '@testing-library/jest-native/extend-expect';

// 2 解决 RN Animated 报错
jest.mock('react-native/Libraries/Animated/NativeAnimatedHelper');

// 3 每次测试后清理
afterEach(() => {
  jest.clearAllMocks();
});
```

* 配置test-utils

```shell
touch src/test-utils/index.ts
# 封装统一 render / renderHook，把你业务所需 Provider（比如 React Query、i18n、导航等）一次性包进去。
# 这是让测试“可维护”的关键步骤。
```

```shell
# index.ts
import { render } from '@testing-library/react-native';
import React from 'react';

// 👉 如果你以后有 Provider（redux / navigation）可以加这里
const AllProviders = ({ children }: { children: React.ReactNode }) => {
  return children;
};

const customRender = (ui: React.ReactElement, options?: any) =>
  render(ui, { wrapper: AllProviders, ...options });

export * from '@testing-library/react-native';
export { customRender as render };

# 使用
import { render, fireEvent } from '@/test-utils';
```

* 创建根目录`__mocks__`

把你项目里会干扰测试的模块放入手动 mock：

1. 原生存储

2. 埋点 SDK
2. toast / animation / keyboard 相关库

`setup.ts`里面的模块级mock实现，这类代码放到`__mocks__`更符合Jest最佳实践。原因：

1. 职责更清晰
   - `setup.ts`：做测试生命周期初始化（如 clear timers）
   - `__mocks__`：放具体模块 mock 实现
2. 可复用、可维护
   模块 mock 能被全项目复用，后续某个 mock 需要调整时也更容易定位。

整体运行流程：

1. 启动Jest

Jest 读取 `jest.config.js` 里的：

- `setupFilesAfterEnv: ['./src/test-utils/setup.ts']`

所以每个测试文件执行前，都会先执行 `setup.ts`。

2. 执行`setup.ts`

`setup.ts` 里调用了：

- `jest.mock('uniwind')`
- `jest.mock('react-native-keyboard-controller')`

这两行的意思是：
“这两个模块不要用真实实现，改用 mock 实现”。

3. Jest去找对应的mock文件

当你写了 `jest.mock('xxx')`，Jest 会优先找：

- `__mocks__/xxx.ts`（或 js/tsx）

所以：

- `jest.mock('uniwind')` -> 使用 `__mocks__/uniwind.ts`
- `jest.mock('react-native-keyboard-controller')` -> 使用 `__mocks__/react-native-keyboard-controller.tsx`

4. 测试文件里import的就是mock

比如组件里 `import { KeyboardAwareScrollView } from 'react-native-keyboard-controller'`，测试时拿到的是你在 `__mocks__` 里写的简化版本，不会跑真实 native 行为。



### 测试内容理解

**核心能力**

1. 测试真正要掌握的能力是测什么，而不是怎么测。

应该优先测试：

核心逻辑

* 表单校验
* API调用逻辑
* 状态变化（loading、error、success）

用户行为：

* 点击按钮
* 输入内容
* 页面跳转

不建议测试：

* 样式（除非关键）
* 内部state实现细节
* 第三库内部行为

2. 测试思维

需要建立一个mindset：像用户一样使用你的组件。

不是：
调用函数 -> 看状态有没有变化

而是：

用户输入 -> 页面是否正确变化



**测试思维**

* 测试目标

测试的目标不是覆盖代码，而是覆盖风险。

* 判断模型

这个功能坏了，会不会出问题。

1. 必测内容（核心路径、高风险）

1.1 用户核心流程

原则：用户完不成 = 严重问题

比如你现在项目：

- 登录 / 注册
- 忘记密码
- 表单提交
- 支付 / 内购（你之前提到过）

1.2 复杂逻辑

这种最容易出bug

* 表单验证（react-hook-form）
* 条件渲染（不同状态UI）
* 多状态切换（loading、error、sucess）

1.3 和后端交互的逻辑

不测的话容易线上翻车

* API请求成功或失败
* 错误处理
* retry/loading



2. 建议测（中风险）

* 关键UI行为

  属于用户操作层

  * 点击按钮是否触发行为
  * 输入框是否更新
  * modal、bottom sheet是否打开

* 状态驱动UI

  ```tsx
  {loading && <Spinner />}
  {error && <Error />}
  // 这种特别适合测
  ```

  

3. 不建议测

* 样式

```tsx
<View style={{ marginTop: 10 }} />
// 不影响功能
```

* 第三方库行为

  - react-hook-form 内部逻辑

  - navigation 内部实现

你测不到，也不该测

* 实现细节

```tsx
const [count, setCount] = useState(0)
// 不关心state，本质关心UI
```



**拆解方法**

每个组件可以这样拆：

1. 列出用户能做什么？

   比如 Login：

   - 输入 email
   - 输入 password
   - 点击登录
   - 看到错误提示
   - 登录成功跳转

2. 列出可能出错的地方

   - email 不合法
   - password 为空
   - API 失败
   - loading 不消失
   - 没有跳转

3. 每个风险点一个测试

   这就是你测试用例来源



**真实RN场景案例**

登录页面应该测试内容：

1. 基础渲染

```ts
should render email and password input
```

2. 表单验证

```ts
should show error when email invalid
should show error when password empty
```

3. 用户交互

```ts
should update input value
should trigger submit on press
```

4. API成功

```ts
should navigate to home on success
```

5. API失败

```ts
should show error message
```

6. loading状态

```ts
should show loading indicator while submitting
```



**组件 vs 业务逻辑**

* 纯函数、工具函数（单元测试）

  `validateEmail(email)`测试输入输出即可。

* UI组件（组件测试）

  测试：渲染、用户行为、UI变化

* 复杂业务（hook、service）

  ```ts
  useLogin()
  ```

  测：状态变化、调用API是否正确



**工程机判断标砖**

* 这个功能用户会频繁使用吗？是 -> 测试
* 出bug会影响使用吗？ 是 -> 测试
* 逻辑复杂吗？ 是 -> 测试
* 只是展示UI吗? 是 -> 不测试或少测试



### Jest理解

**Jest是什么**

jest本质做4件事情：

* 运行测试文件
* 提供断言（expect）
* 提供mock能力
* 生成测试报告（通过、失败、覆盖率）

可以理解为：一个测试执行引擎 + 工具箱

**最小可运行示例**

```ts
// sum.ts
export function sum(a: number, b: number) {
  return a + b
}

// sum.test.ts
import { sum } from "./sum"

describe("sum function", () => {
  it("should return correct result", () => {
    expect(sum(1, 2)).toBe(3)
  })
})

// 运行后发生什么？
Jest 会：
1. 找到 *.test.ts
2. 执行里面的代码
3. 输出结果
PASS sum.test.ts
```

**Jest核心结构（必须掌握）**

* describe（测试分组）

```ts
describe("Login module", () => {
  // 测试写这里
})
// 作用：
- 分组测试
- 让结构更清晰
```

* it/test（测试用例）

```ts
it("should login success", () => {
  // 测试逻辑
})

// 本质：一个it = 一个测试场景
```

* expect（断言）

```ts
expect(result).toBe(3)
// 我期望这个值是3
```

**断言（最重要的基础）**

不用记全，只需掌握常用的。

* toBe（严格相等）

```ts
expect(1 + 1).toBe(2)
```

* toEqual(对象比较)

```ts
expect({ a: 1 }).toEqual({ a: 1 })
// 区别
expect({ a: 1 }).toBe({ a: 1 }) ❌ 
// 因为引用不同
```

* toBeTruthy/toBeFalsy

```ts
expect(true).toBeTruthy()
expect(false).toBeFalsy()
```

* toContain(数组、字符串)

```ts
expect([1, 2, 3]).toContain(2)
```

* toThrow(异常)

```ts
function errorFn() {
  throw new Error("fail")
}

expect(() => errorFn()).toThrow()
```

**函数调用测试（非常重要）**

* jest.fn()： 创建mock函数

```ts
const fn = jest.fn()
fn()
expect(fn).toHaveBeenCalled()
```

* 常用断言

```ts
expect(fn).toHaveBeenCalled()
expcet(fn).toHaveBeendCalledTimes(1)
expect(fn).toHaveBeenCalledWith('test')
```

* 实例

```ts
function callCallBack(cb) {
  cb('Hello')
}

it('should call callback', () => {
  const fn = jest.fn()
  
  callCallBack(fn)
  
  expect(fn).toHaveBeenCalledWith('Hello')
})
```

**Mock(核心中的核心)**

如果你只学一个技能，一定是mock。

为什么需要mock: 因为测试时不想依赖真实环境。

```shell
比如：
- API请求
- 数据库
- AsyncStorage
```

* mock函数返回值

```ts
const fn = jest.fn()
fn.mockReturnValue(10)
expect(fn()).toBe(10)
// mock返回值时，如果是同步函数使用：mockReturnValue
// 如果是异步函数，mockResolvedValue用于成功，mockRejectedValue用于失败
同步函数 → mockReturnValue
返回 Promise → mockResolvedValue / mockRejectedValue
```

* mock异步函数

```ts
const fn = jest.fn()
fn.mockResolvedValue('ok')
const result = await fn()
expect(result).toBe('ok')
```

* mock模块（RN重点）

```ts
import axios from "axios"

jest.mock('axios') // mock axios模块

it('mock api', async () => {
  axiso.get.mockResolvedValue({data: 'test'})
  
  const res = await axios.get('/user')
  expect(res.data).toBe('test')
})
```

```ts
jest.mock('../../authApi');
// 做了两件事情
- 整个模块被替换
- 所有导出函数都变成mock function： jest.fn

const mockRequestPasswordReset = authApi.requestPasswordReset as jest.MockedFunction<
  typeof authApi.requestPasswordReset>;
// 类型断言：只影响TypeScript，不影响运行时
// 让ts知道mockRequestPasswordReset.mockResolvedValue()
// 否则TS报：Property 'mockResolvedValue' does not exist
```

```ts
import { useRouter } from 'expo-router';
// 做了两件事：
// - 替换整个模块：原本import { useRouter } from 'expo-router';现在变成useRouter = jest.fn()，真实的useRouter被干掉了，变成一个 mock function
// - 你完全控制它的返回值：默认情况下，useRouter的值为undefined，因为jest.fn()默认啥也不返回
jest.mock('expo-router', () => ({
  useRouter: jest.fn(),
}));
// TypeScript断言类型：它的作用是mockRouter.mockReturnValue(...)不会报错。
const mockRouter = useRouter as jest.Mock; 
const mockPush = jest.fn();

beforeEach(() => {
  mockPush.mockClear();
  // 控制useRouter的返回值
  mockRouter.mockReturnValue({
    push: mockPush,
  });
});

// 交互流测试
it('should send request with entered email and navigate to successful page', () => {
  const { getByTestId } = render(<ForgotPasswordForm />);

  fireEvent.changeText(getByTestId('forgot-password-email-input'), 'test@example.com');
  fireEvent.press(getByTestId('forgot-password-send-button'));

  expect(mockSend).toHaveBeenCalledWith(
    { email: 'test@example.com' },
    { onSuccess: expect.any(Function) },
  );

  // 从mockSend第一次调用的参数里，拿出第2个参数（options），再拿出其中的onSuccess回调并手动执行
  // 测试里 mock 的 login 不会真的发请求、也不会自动触发成功回调，所以要手动 onSuccess?.() 来验证“成功后是否导航”。
  const onSuccess = mockSend.mock.calls[0]?.[1]?.onSuccess as (() => void) | undefined;
  onSuccess?.();

  expect(mockPush).toHaveBeenCalledWith('/forgot-password-success');
});

// 渲染状态测试，不是交互态测试
// 这段代码里先把hook mock成：error: 'Email is required'
// 组件一渲染就会读取 useRequestPasswordReset hook的值,看到error就返回错误的UI
it('should show error message when error is set', () => {
  mockUseRequestPasswordReset.mockReturnValue({
    requestPasswordReset: mockSend,
    isLoading: false,
    error: 'Email is required',
  });

  const { getByTestId } = render(<ForgotPasswordForm />);

  expect(getByTestId('send-error')).toBeTruthy();
});
```

> **jest.Mock vs jest.MockedFunction<T>**
>
> **`jest.Mock`** **= 泛用类型（不关心原函数）**
>  **`jest.MockedFunction`** **= 强类型版本（绑定原函数签名）**
>
> 1. `jest.Mock`（宽松 / 通用）
>
> ```ts
> const fn = jest.fn() as jest.Mock;
> ```
>
> 特点：
>
> - 不关心参数类型
> - 不关心返回值类型
> - 什么都能写
>
> ```ts
> fn.mockReturnValue(123);
> fn('abc', true, {});
> // TS 不会报错（因为它“啥都接受”）
> ```
>
> 2. `jest.MockedFunction`（强类型）
>
> ```ts
> const fn = someFunction as jest.MockedFunction<typeof someFunction>;
> ```
>
> 特点：
>
> - 自动继承原函数签名
> - 参数类型受约束
> - 返回值受约束
>
> ```ts
> function add(a: number, b: number): number {
>   return a + b;
> }
> 
> const mockAdd = add as jest.MockedFunction<typeof add>;
> 
> mockAdd(1, 2);      // ✅
> mockAdd('a', 'b');  // ❌ TS报错
> ```
>
> 3. 本质区别
>
> jest.Mock: 一个“无类型约束”的 mock 函数
>
> jest.MockedFunction: 一个“继承原函数类型”的 mock 函数
>
> ```
> MockedFunction = Mock + 类型安全
> ```

```ts
// mock hook通用模板
jest.mock('xxx', () => ({
  useSomething: jest.fn(),
}));

const mockUseSomething = useSomething as jest.Mock;

mockUseSomething.mockReturnValue(...)
// 适用于：
* navigation（useRouter）
* redux（useSelector）
* 自定义 hooks
```



**异步测试（必须掌握）**

* async/await

```ts
it('async test', async () => {
  const data = await Promise.resolve('ok')
  expect(data).toBe('ok')
})

// waitFor不是为了保险，而是为了等异步完成。
// 用waitFor的情况：
- API返回
- Promise
- setState
- hook更新
- UI变化
// 不用waitFor
- 常量
- 同步函数返回值
- 初始状态
```

* 错误处理

```ts
it('should throw error', () => {
  expect(Promise.reject('error')).rejects.toBe('error')
})
```

**生命周期**

* beforeEach/afterEach

```ts
beforeEach(() => {
  console.log('run before each test')
})

afterEach(() => {
  console.log('cleanup')
})

// 用途：
- 重置mock
- 初始化数据
```

* 清除mock

```ts
beforeEach(() => {
  mockRequestPasswordReset.mockClear(); // 只清调用记录，不清实现
  // 清调用记录 + 清实现: 避免“上个测试残留实现”，所以 mockReset() 更稳。
  mockRequestPasswordReset.mockReset(); 
})
// 作用：每次测试前，清空mock的调用记录，如果不清空，上一个测试的调用次数会影响下一个测试
afterMock(() => {
  jest.clearAllMocks()
})
```



### jest.fn理解

一个可记录行为的假函数。（mock function）

**基础功能**

```ts
const mockFn = jest.fn();
// 得到一个函数，可以的调用。
mockFn(); // 可以调用
// 关键是它可以记录所有行为
// 能记录什么？
mockFn('a', 123);
mockFn('b', 456);

expect(mockFn).toHaveBeenCalled();          // 是否调用过
expect(mockFn).toHaveBeenCalledTimes(2);   // 调用次数
expect(mockFn).toHaveBeenCalledWith('a', 123); // 参数
```

**核心能力**

* 记录调用信息

```ts
// 查看调用参数
mockFn('hello', 1);
mockFn('world', 2);

console.log(mockFn.mock.calls);
// 结果
[
  ['hello', 1],
  ['world', 2],
]

// 查看返回值
const fn = jest.fn((x) => x * 2);

fn(2);
fn(3);

console.log(fn.mock.results);
// 结果
[
  { value: 4 },
  { value: 6 }
]
```

* 自定义实现

```ts
// 1. 直接传实现
const sum = jest.fn((a, b) => a + b)
sum(1, 2)

// 2. mockImplementation
const fn = jest.fn()
fn.mockImplementation = ((x) => x * 10)
fn(2)

// 3. 临时覆盖一次: 非常适合测试多次调用的不同结果
fn.mockImplementationOnce(() => 'first')
fn.mockImplementationOnce(() => 'second')

fn() // 'first'
fn() // 'second'
fn() // undefined
```

* 控制返回值

```ts
// 1. mockReturnValue
const fn = jest.fn().mockReturnValue('hello')
fn(); // 'hello'

// 2. mockReturnValueOnce
const fn = jest.fn().mockReturnValueOnce('a').mockReturnValueOnce('b')
fn() // 'a'
fn() // 'b'
fn() // undefined

// 3. mockResolvedValue(Promise)
const fetchUser = jest.fn().mockResolvedValue({name: 'matt'})
featchUser() // {name: 'matt'}

// 4. mockRejectedValue(Promise)
const fn = jest.fn().mockRejectedValue(new Error('fail'))
await fn() // throw error
```

* 重置和清理

```ts
// 1. 清空调用记录
mockFn.mockClear() // 只清调用记录，不清实现

// 2. 重置（连实现一起清）
mockFn.mockReset()

// 3. 恢复原始（配合spyOn）
mockFn.mockRestore() // 只适用于spyOn

// 4. 全局场合用
beforeEach(() => {
  jest.clearAllMocks()
})
```

**React Native常见用法**

* 作为props callback

```ts
const onSubmit = jest.fn();

render(<Form onSubmit={onSubmit} />);

fireEvent.press(screen.getByText('Submit'));

expect(onSubmit).toHaveBeenCalled();
```

* Mock hook返回函数

```ts
useMyHook.mockReturnValue({
  submit: jest.fn(),
});
```

* mock storage/API

```ts
set: jest.fn((key, value) => store.set(key, value))
// 既执行逻辑，又可测试
```

**spyOn理解**

* spyOn是做什么的

监听一个已经存在的函数。

```ts
jest.spyOn(obj, 'method');
```

* 核心能力

```ts
- 记录调用情况
- 可以替换实现
- 可以恢复原始函数（很重要）
const api = {
  fetchUser: () => {
    return { name: 'real user' };
  },
};

// 使用spyOn
const spy = jest.spyOn(api, 'fetchUser');

api.fetchUser();

expect(spy).toHaveBeenCalled();

// 替换实现
jest.spyOn(api, 'fetchUser').mockReturnValue({ name: 'mock user' });

// 注意：
不是创建函数，而是劫持原函数
```

|                  | **jest.fn**      | **jest.spyOn**      |
| ---------------- | ---------------- | ------------------- |
| 是否需要已有函数 | ❌ 不需要         | ✅ 必须已有          |
| 用途             | 创建 mock        | 监听/替换真实方法   |
| 是否影响原函数   | 无               | 会（可恢复）        |
| 常见场景         | props / callback | API / module / util |

* 工程建议

```ts
// 优先用 spyOn 的场景

* API 请求
* util 方法
* storage（AsyncStorage）
* navigation 方法

// 优先用 fn 的场景

* props callback
* event handler
* hook return function
```

高级理解：

jest.fn: 我造一个假的演员

jest.spyOn: 我在真实演员身上装摄像头，甚至可以让他按我说的演









### React Native Test Library理解

一句话理解：用“用户视角”去操作和验证UI

**整体思维模式**

在RNTL中你做的事情永远是：

1. Render(渲染组件)
2. query（查找元素）
3. fireEvent（操作元素）
4. assert（验证UI）

**render（渲染组件）**

```tsx
// 基本用法
import { render } from '@testing-library/react-native';

const { getByText } = render(<Text>Hello</Text>);

// 返回值是什么？
const result = render(<MyComponent />);
// 你会拿到
result.getByText
result.getByPlaceholderText
result.queryByText
result.findByText
// 这些就是query工具
```

**query（查找元素）非常重要**

查询方式分类：

* getBy（同步，找不到就报错）

```tsx
getByText('Login');
// 用于元素必须存在
```

* queryBy（找不到返回null）

```tsx
queryByText('Error');
// 用于判断是否不存在
```

* findBy（异步）

```tsx
await findByText('Success');
// 用于API返回后才出现的UI
```

**查询优先级**

要按这个顺序用：

* 推荐

```tsx
getByText('Login')
getByPlaceholderText('Email')
```

* 最后才用

```tsx
getByTestId('submit-btn')
```

原因：testID 是“实现细节”，不是用户行为

**fireEvent（用户交互）**

* 点击

```tsx
fireEvent.press(button);
```

* 输入

```tsx
fireEvent.changeText(input, 'test@example.com');
```

* 示例

```tsx
const { getByText } = render(<Button title="Click" onPress={fn} />);
fireEvent.press(getByText('Click'));
```

**waitFor（处理异步）**

```tsx
// 为什么需要？
API / setState / effect 都是异步

// 用法
await waitFor(() => {
  expect(getByText('Success')).toBeTruthy();
});
// 它会不断重试，直到条件满足或超时
```

**完整案例**

```tsx
const Login = ({ onLogin }) => {
  const [email, setEmail] = useState('');

  return (
    <>
      <TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
      />
      <Button title="Login" onPress={() => onLogin(email)} />
    </>
  );
};
```

```tsx
it('should input and submit', () => {
  const onLogin = jest.fn();

  const { getByPlaceholderText, getByText } = render(
    <Login onLogin={onLogin} />
  );

  const input = getByPlaceholderText('Email');

  fireEvent.changeText(input, 'test@test.com');
  fireEvent.press(getByText('Login'));

  expect(onLogin).toHaveBeenCalledWith('test@test.com');
});
```

**真实组件测试**

* 场景1：登录页

```tsx
jest.mock('../api');

it('should login success', async () => {
  api.login.mockResolvedValue({ token: '123' });

  const { getByText, getByPlaceholderText } = render(<Login />);

  fireEvent.changeText(getByPlaceholderText('Email'), 'test@test.com');
  fireEvent.changeText(getByPlaceholderText('Password'), '123');

  fireEvent.press(getByText('Login'));

  await waitFor(() => {
    expect(getByText('Success')).toBeTruthy();
  });
});
```

* 场景2：表单验证

```tsx
it('should show error when email empty', async () => {
  const { getByText } = render(<Login />);

  fireEvent.press(getByText('Login'));

  expect(getByText('Email is required')).toBeTruthy();
});
```

* 错误提示

```tsx
api.login.mockRejectedValue(new Error('Invalid credentials'));
await waitFor(() => {
  expect(getByText('Invalid credentials')).toBeTruthy();
});
```



### Snapshot理解

**Snapshot（快照测试）在 React Native 项目中“可以用，但要非常克制”**
 很多团队后来都**减少甚至放弃 snapshot**，因为容易变成“无效测试”。

**Snapshot是什么**

在Jest里，Snapshot = 把组件渲染结果“拍一张结构快照”，下次对比有没有变化。

类比理解：
```shell
第一次运行测试 → 拍照保存
以后运行测试 → 拿当前结果和照片对比
```

**最简单例子**

```tsx
// 组件
const MyComponent = () => {
  return <Text>Hello World</Text>;
};

// 测试
import { render } from '@testing-library/react-native';

it('should match snapshot', () => {
  const { toJSON } = render(<MyComponent />);
  expect(toJSON()).toMatchSnapshot();
});
```

```shell
# 第一次运行会生成
__snapshots__/MyComponent.test.ts.snap
内容大致：
exports[`should match snapshot 1`] = `
<Text>
  Hello World
</Text>
`;
# 之后运行，jest会做：当前 render 结果 vs snapshot 文件
```

**当代码发生变化会发生什么**

```tsx
// 修改组件
<Text>Hello ChatGPT</Text>

// 再跑测试
❌ Snapshot mismatch
// Jest会提示
Expected: Hello World
Received: Hello ChatGPT
```

**Snapshot本质在做什么**

Snapshot测的是“UI结构是否发生变化”。

它不是测：

* 逻辑
* 交互
* 用户行为

它测试的是render输出结构（tree）

**优点**

* 写起来很快
* 可以快速发现UI变化
* 适合稳定组件

**最大问题**

* 很容易误报

```shell
改个文案：Login → Sign In
snapshot就会fail
```

* 很容易瞎更新

```shell
看到Press u to update snapshot就直接更新。
测试就失去意义
```

* 可读性差

```shell
一大坨 JSX 结构,很难看出问题在哪
```

**使用场景**

* UI结构稳定的纯展示组件

```tsx
const Divider = () => <View style={{ height: 1 }} />;
```

* 不太变化的UI组件

```tsx
Header / Footer / Icon
```

* 设计系统组件

```shell
比如：

* Button
* Card
```

不推荐使用场景：

```shell
表单 / 登录页

👉 因为：
* 状态多
* 变化多
* snapshot 很容易失效

有大量交互的组件
👉 应该用：fireEvent + expect

API相关组件
👉 snapshot 没意义
```















### patch-package理解

**核心思路**

直接改`node_modules`临时生效，生成patch文件（永久生效，可版本控制）。

**安装工具**

```shell
pnpm add patch-package postinstall-postinstall -D
```

postinstall-postinstall作用：确保 postinstall 脚本在所有包安装完成后“再执行一次”

```shell
# 正常配置
{
  "scripts": {
    "postinstall": "patch-package"
  }
}
# 你以为执行
install 所有依赖
→ 执行 postinstall（patch-package）

# 但现实是（尤其在 pnpm / yarn workspaces）：
某些依赖安装过程中
→ 提前触发 postinstall ❌
→ 此时 node_modules 还没完整
→ patch-package 执行失败或不完整

结果就是：
* patch 没完全应用
* CI 偶现失败
* 本地 OK，别人不 OK（最恶心）

# postinstall-postinstall 如何解决
第一次 postinstall（可能太早）→ 跳过
等全部 install 完成 → 再执行一次 postinstall
```



**配置package.json**

```json
{
  "scripts": {
    "postinstall": "patch-package"
  }
}
// 作用：每次 pnpm install / npm install 后自动打补丁
```

**修改node_modules**

```shell
node_modules/some-library/index.js
// 直接改
```

**生成patch**

```shell
npx patch-package some-library

// 会生成
patches/some-library+1.2.3.patch
注意：
- 1.2.3 是当前安装版本
- patch 是基于这个版本的
```

**提交代码并验证patch是否生效**

```shell
rm -rf node_modules
pnpm install
```

**常见坑**

* 改错包名

```shell
npx patch-package some-library
# 必须和 package.json 里的名字完全一致
```

* patch失效

原因：版本变了。

```shell
some-library+1.2.3.patch ❌
# 实际安装：1.2.4

# 解决：重新生成
npx patch-package some-library
```

* Expo/React Native特别注意

```shell
如果你是 Expo 或 RN：

👉 patch-package 是完全兼容的
👉 但要确保：
* 没有被 Metro cache 干扰
npx expo start -c
```

* iOS/android原生代码patch

```shell
node_modules/some-library/ios/xxx.m
node_modules/some-library/android/xxx.java

patch 也能生效,但要重新 build：
npx pod-install
npx react-native run-ios
```



### Layout-centered vs Container-centered理解

Layout-centered公共结构放在`_layout.tsx`里面，screen尽量薄。

Container-centered每一个screen都通过一个通用容器组件拿到header、background、back等能力。

**为什么现在更偏向Layout-centered**

因为Expo Router的设计哲学就是：路由层定义结构，页面层定义内容。

把group组公共UI、导航策略放`_layout.tsx`，天然和框架契合。

**两种方案优缺点**

1. _layout方案（当前更流行）

优点：

* 结构清晰：组级公共逻辑集中管理（header、标题、返回策略）
* 页面更薄：路由文件`login.tsx`/`signup.tsx`几乎只渲染业务组件
* 更符合框架心智
* 变更影响可控：改auth组壳子，一处改全组生效

缺点：

* 过多页面特例会让页面变复杂（需要screenOptions分流）
* 如果要跨多个route group复用同一壳子，可能会出现重复配置（需再抽小组件）

2. screenContainer方案

优点：

* 复用快：一个组件快速给多个页面套壳
* 跨组复用方便：不依赖某个具体的router group
* 对非路由上下文页面（如某些嵌套模块）也可以直接用

缺点：

* 容易长成万能组件：props越来越多，维护变重
* screen变接线层：页面到处传showHeader、backFallbackHref、onbackPress
* 与expo router的layout机制重叠，职责边界变模糊

**layout-centered（布局驱动）**

页面结构有layout控制。

典型在Expo Router

```tsx
// app/(auth)/_layout.tsx
export default function Layout() {
  return (
    <Stack
      screenOptions={{
        headerTitleAlign: 'center',
      }}
    />
  );
}
```

核心思想：页面长什么样，是layout决定的。

* header
* padding
* safe area
* 背景
* 对齐方式

都在route里统一控制。

**container-centered**

页面结构由页面自己控制。

```tsx
export default function Screen() {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Hello</Text>
    </View>
  );
}
```

核心思想：每个页面自己决定布局

**为什么现在主流是layout-centered**

因为container-centered在中大型项目中会失控。

* UI不一致

```shell
每个页面都写padding

* 16
* 20
* 24

结果：页面风格不统一
```

* header管理混乱

```shell
有的页面：
* 用 Stack header
* 有的自己写 header
* 有的没有 header

结果：

* 返回逻辑混乱
* UI 不统一
```

* SafeArea/状态栏问题

```shell
<SafeAreaView> // 有的页面有

有的页面没处理：

* iPhone 刘海遮住
* Android 状态栏重叠
```

* 重复代码严重

```shell
每个页面
<View style={{ flex: 1, backgroundColor: '#fff' }}>
写 100 次
```

**layout-centered优势**

* 统一UI（设计系统级别）

```tsx
// app/_layout.tsx
<Stack
  screenOptions={{
    headerStyle: { backgroundColor: '#fff' },
    contentStyle: { backgroundColor: '#F5F5F5' },
  }}
/>
// 所有页面自动统一
```

* 减少80%页面结构

```tsx
// 页面只写
export default function Screen() {
  return <Text>Hello</Text>;
}
//  不需要再关心：
* header
* safe area
* padding
```

* 支持分组布局

```shell
app/
 ├── (auth)/
 │    ├── _layout.tsx
 │    ├── login.tsx
 │    └── register.tsx
 ├── (main)/
 │    ├── _layout.tsx
 │    ├── home.tsx
 │    └── profile.tsx
// 每个分组一个 layout
```

```tsx
// auth layout（无 header）
<Stack screenOptions={{ headerShown: false }} />
// main layout（有 header）
<Stack screenOptions={{ headerShown: true }} />
```

页面自动继承

* 更适合团队协作: 职责清晰

```shell
团队开发时：

* layout = 架构层控制
* screen = 业务层实现
```

* 更容易做主题

```tsx
// 只改layout
contentStyle: {
  backgroundColor: isDark ? '#000' : '#fff'
}
// 所有页面自动切换
```

**对比**

```tsx
// container-centered（老项目常见）
export default function Profile() {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <CustomHeader title="Profile" />
      <View style={{ padding: 16 }}>
        <Text>User Info</Text>
      </View>
    </SafeAreaView>
  );
}
// 问题：
* header 每个页面写一遍
* padding 不统一
* SafeArea 容易漏

// layout-centered（推荐）
// layout.tsx
<Stack screenOptions={{ headerShown: true }} />

// page.tsx
export default function Profile() {
  return <Text>User Info</Text>;
}
// 页面极简
```

**_layout主要作用**

他是路由级别的UI容器 + 结构控制器

在 Expo Router 里：

- 每个目录都可以有一个 `_layout.tsx`
- 它控制这个目录下**所有页面的公共结构**

```shell
app/
 ├── (auth)/
 │    ├── _layout.tsx
 │    ├── login.tsx
 │    └── register.tsx
 
# _layout.tsx
 export default function Layout() {
  return <Stack screenOptions={{ headerShown: false }} />;
}
# 作用：
* login / register
* 全部自动没有 header
```

本质做了3件事：

* 定义导航结构（Stack、Tabs）

```tsx
<Stack />

<Tabs />
```

* 控制全局UI

```shell
* header
* 背景色
* 动画
* 手势
```

* 包括所有子页面

```shell
Layout(
  Screen1,
  Screen2,
  Screen3
)
```





### Expo Router理解

Expo Router基于文件系统的导航方案 + React Nativigation的封装增强版。

用结构思维设置App，而不是代码思维。

**Expo Router是什么**

Expo Router = 文件即路由（File-based routing）

它把写代码-> 配置路由这个过程变为：创建文件 = 创建页面 + 自动生成路由

**核心思想**

* 文件即路由（核心中的核心）

```tsx
// 传统React Navigation
<Stack.Navigator>
  <Stack.Screen name="Home" component={HomeScreen} />
  <Stack.Screen name="Profile" component={ProfileScreen} />
</Stack.Navigator>

// Expo Router
app/
  index.tsx        -> /
  profile.tsx      -> /profile
  user/[id].tsx    -> /user/:id
// 所有页面都是app目录下的文件。
- app/是路由唯一入口
- 每个文件 = 一个页面
- 不再定义路由，而是用文件结构表达App结构
```

不需要手动注册路由，目录结构就是路由结构

* 每个页面都有URL

```shell
app/profile.tsx → /profile
app/user/[id].tsx → /user/:id

# 统一web，iOS和Android，移动端像web路由模型靠拢
- Deep Linking 默认支持
- URL可分享
- 统一导航语义
```

* index.tsx决定初始页面

```shell
app/index.tsx -> /
app/(tabs)/index.tsx -> /
# 入口由路由决定，而不是代码
# 不像React Navigation
initialRouteName: 'Home'
```

* 路由是声明式结构，而不是配置代码

```ts
传统方式：
👉 你是在 JS 里“写配置”

Expo Router：
👉 你是在“设计文件结构”
```

* Layout(布局机制)机制（非常重要）

```tsx
app/
  _layout.tsx
  index.tsx
  profile/
    _layout.tsx
    index.tsx

// _layout.tsx = 当前目录的导航容器
// app/_layout.tsx
export default function Layout() {
  return <Stack />;
}

// app/profile/_layout.tsx
export default function Layout() {
  return <Tabs />;
}
结果：
* / 用 Stack
* /profile/* 用 Tabs

// _layout.tsx = App 入口（替代App.tsx）
// _layout.tsx = 导航架构层
- 放Provider（Theme、Redux）
- 定义导航（Stack/Tabs)
- 控制全局UI
```

本质用文件夹来表达导航层级，而不是用代码嵌套。

* URL驱动导航（类web思维）

```ts
// Expo Router引入了web的routing思维
import { Link } from 'expo-router';

<Link href="/profile/123" />
// 而不是
navigation.navigate('Profile', { id: 123 })
```

* Deep Linking天然支持

```ts
// 因为它是URL-based
myapp://user/123
// 自动匹配：不需要复杂配置linking
app/user/[id].tsx 
```

* Tabs/Platform-specific（平台差异处理）

```shell
# 官方默认使用不同平台的 tabs 实现
components/
  app-tabs.native.tsx   // iOS / Android
  app-tabs.tsx          // Web
# 一套路由，多端适配
```

* 非页面代码不能放在app目录

```shell
路由层 vs 业务层分离
```

* 与React Navigation关系

```ts
Expo Router ≠ 替代 React Navigation
👉 实际是：Expo Router = React Navigation + 文件路由 + 更高层抽象

底层仍然是：
* Stack
* Tabs
* Drawer
```



**Expo Router Notation**

Notation = 文件、目录命名规则。

本质：用特殊文件或目录格式来表达不同类型的路由行为。

* 核心Notation总览

```shell
# 1.普通路由
app/profile.tsx → /profile
最基础规则：
* 文件名 = 路径

# 2. 动态路由[param]
app/user/[id].tsx → /user/123
# 示例
app/products/[productId].tsx， 访问/products/100
# 使用
const { productId } = useLocalSearchParams();
# 路径参数 = URL驱动数据

# 3. 嵌套路由（文件夹）
app/profile/settings.tsx → /profile/settings
# 示例
app/
  profile/
    index.tsx →  /profile
    settings.tsx → /profile/settings
# 目录 = 路由层级

# 4. 布局导航容器
app/_layout.tsx
# 作用：
* 定义 Stack / Tabs / Drawer
* 包裹子页面

# 5. index.tsx(默认路由)
app/index.tsx → /
app/profile/index.tsx → /profile
# 默认页面 = 路由入口

# 6. route groups
app/(tabs)/home.tsx → /home
# 特点：
(xxx) 不会出现在 URL 中
# 示例 
app/
  (tabs)/
    home.tsx → /home
    settings.tsx → /settings
# 用途：做模块划分，而不是路径划分
app/
  (auth)/
    login.tsx
  (main)/
    home.tsx
# 解耦：业务模块 vs 路由路径

# 7. 特殊文件 + 前缀
# 404页面
app/+not-found.tsx
# 全局错误处理
+error.tsx
# +layout.tsx
已逐步被 _layout.tsx 替代
# +middleware.ts
服务端拦截 / 权限控制（高级）
# 这些是系统级hook

# 8. 平台文件（.native/.web)
tabs.native.tsx
tabs.web.tsx
# 作用：不同平台加载不同实现
# 统一路由，多端差异实现
```



**_layout.tsx**

* `_layout.tsx`是什么

每个目录都可以有一个`_layout.tsx`文件，用于定义该目录下页面布局和导航结构。

`_layout.tsx` = 当前目录所有页面的“导航容器 + UI包裹器”。

* 核心作用（本质）

`_layout.tsx`决定页面之间的关系。

它可以做三件事：

1. 定义导航栏结构（最重要）

```tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return <Stack />;
}
// 表示：当前目录所有页面 = Stack 关系
```

2. 包裹UI（Header、Footer）

```tsx
import { Slot } from 'expo-router';

export default function Layout() {
  return (
    <>
      <Header />
      <Slot />
      <Footer />
    </>
  );
}
// <Slot />：子页面渲染位置
```

3. 做初始化（只在Root Layout）

```tsx
// app/_layout.tsx
整个 App 的入口
// 可以做：
- 字体加载
- Splash Screen 控制
- 全局 Provider
// 相当于以前的App.tsx
```

* `_layout.tsx`的层级机制

```tsx
// 每个目录都可以有layout
app/
  _layout.tsx        (Root)
  index.tsx

  profile/
    _layout.tsx      (Profile Layout)
    index.tsx

// 渲染顺序（非常重要）
Root Layout
   ↓
Profile Layout
   ↓
Page
// 本质：Layout是逐层嵌套执行的。
```

* 三种最常见的嵌套类型

1. Stack Layout（最常用）

```tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return <Stack />;
}
// 行为：
* 页面 push / pop
* 自动 back 按钮

app/products/
  _layout.tsx
  index.tsx
  [id].tsx
```

2. Tabs Layout

```tsx
import { Tabs } from 'expo-router';

export default function Layout() {
  return <Tabs />;
}

//行为：
* 子页面 = tab

// 3个tab
app/(tabs)/
  _layout.tsx
  index.tsx
  feed.tsx
  profile.tsx
```

3. Slot Layout

```tsx
import { Slot } from 'expo-router';

export default function Layout() {
  return <Slot />;
}

// 行为：
* 不使用导航（无 stack / tabs）
* 只是 UI 包裹

// 典型场景：页面切换 = 直接替换，不会 push stack
<>
  <Header />
  <Slot />
  <Footer />
</>
```

* 完整例子

```shell
app/
  _layout.tsx          -> Root Stack

  (tabs)/
    _layout.tsx        -> Tabs
    index.tsx
    profile.tsx

  modal/
    _layout.tsx        -> Stack
    index.tsx
    
# 最终导航结构
Root Stack
 ├── Tabs
 │     ├── Home
 │     └── Profile
 └── Modal Stack
```

* 作为架构师必须理解

1. Layout = navigation tree

```shell
_layout.tsx → Stack / Tabs / Slot
```

2. 文件夹 = 路由层级

```shell
profile/settings → 嵌套导航
```

3. layout是层级嵌套的

```shell
Root → 子 layout → 页面
```



**Expo Router Navigation理解**

* 本质

Expo router提供两种导航方式：声明式（Link）+ 命令式（router）。

```ts
Web 思维：
<a href="/profile" />

React Native 传统：
navigation.navigate('Profile')

Expo Router：
Link + router.push('/profile')
```

* 两种导航方式

1. 声明式导航（Link）

这是expo router最推荐的方式。

```tsx
// 基本用法
import { Link } from 'expo-router';

<Link href="/about">Go to About</Link> // 点击自动跳转 /about


// React Native写法
import { Link } from 'expo-router';
import { Text } from 'react-native';

<Link href="/profile">
  <Text>Go to Profile</Text>
</Link>

// 自定义组件（很重要）
<Link href="/profile" asChild>
  <Pressable>
    <Text>Go</Text>
  </Pressable>
</Link>

// 核心特点

* 类似 <a> 标签
* 自动处理 navigation
* 支持 deep linking
* 更符合 React 思维
// 架构理解：Link = UI + Navigation 绑定
```

2. 命令式导航

```tsx
// 使用方式
import { useRouter } from 'expo-router';

const router = useRouter();

// 常用范式
router.push('/profile'); // 会加入历史栈
router.replace('/home'); // 替换当前页面（登录后常用）
router.back(); // 返回上一页
router.canGoBack()
router.setParams({ id: '123' });

// 机构理解：router = Navigation Controller
```

* URL是导航的核心

Expo router的核心理念：URL是导航的唯一来源。

```tsx
router.push('/user/123'); 对应 app/user/[id].tsx
// 本质变化：从“状态驱动” → “URL驱动”
```

* 路径规则

```tsx
// 绝对路径：从 root 开始
router.push('/profile');

// 相对路径: 类似文件路径
router.push('./settings');
router.push('../');

// 外部链接：打开浏览器
router.push('https://google.com');

// 返回上一页
router.push('..');
```

* 参数系统

```ts
// path参数
/user/123
const { id } = useLocalSearchParams();

// query参数
router.push('/search?q=react');
const { q } = useLocalSearchParams();

// 架构理解：URL = 路由 + 参数 + 状态
```

* Navigation hooks

```tsx
// useRouter
const router = useRouter();

// usePathname: 当前路径
const path = usePathname();

// useSegments: 路径拆分数组
const segments = useSegments();

// useLocalSearchParams: 当前页面参数
const params = useLocalSearchParams();

// 架构意义：用 hook 替代 navigation state
```

* Navigation Container

```tsx
传统React Navigation
<NavigationContainer>
  
Expo Router：不需要
NavigationContainer 由 Expo Router 自动管理

// 架构意义：你只关心路由，不关心容器
```

* 完整导航流程

```ts
用户点击 Link
   ↓
生成 URL (/profile/1)
   ↓
Expo Router 解析路径
   ↓
匹配文件（app/profile/[id].tsx）
   ↓
通过 React Navigation 渲染
```

* 和React Navigation本质区别

```ts
// 传统
action → navigation state → UI

// Expo Router
URL → route → navigation state → UI
```

Expo Router 的 Navigation 本质是：
 **以 URL 为核心，通过 Link（声明式）和 router（命令式）两种方式驱动页面跳转，并由文件系统自动映射到 React Navigation 的导航状态。**



**Common Navigation Pattern**

如何组织Stack、Tabs和Layout来解决真实业务场景。

```ts
Core Concepts → 是语法
Notation → 是规则
_layout → 是结构
Navigation → 是跳转

Common Patterns → 是“真实项目怎么设计”
```

* 核心模式总览

1. Tabs + Stack（最重要）

```ts
// 结构
app/
  (tabs)/
    _layout.tsx      -> Tabs
    index.tsx        -> 首页 tab

    feed/
      _layout.tsx    -> Stack
      index.tsx
      [postId].tsx

    settings.tsx
// 导航结构
Tabs
 ├── Home
 ├── Feed (Stack)
 │     ├── Feed List
 │     └── Post Detail
 └── Settings
 // 关键点：
 Tabs始终存在（底部栏不会消失）
 feed内部是Stack（可以push详情页）
 // 示例
 <Link href="/feed/123" />
 会：
* push /feed/[postId]
* Tab 仍然可见 
// 架构意义
Tabs一级导航
Stack二级导航
这是90%的App的标准结构
```

最常用的是：Stack + Tabs（Tabs 在 Stack 里面）
是否隐藏 Header / TabBar，本质不是“写样式”，而是由路由结构决定 + 少量 options 配置

```tsx
// 1. Stack + Tabs 更常用
// 结构
app/
  _layout.tsx        → Root Stack

  (tabs)/
    _layout.tsx      → Tabs
    home/index.tsx
    search/index.tsx

  product/[id].tsx   → 详情页（在 Tabs 外）
  
// 导航结构
Root Stack
 ├── Tabs
 │     ├── Home
 │     └── Search
 └── Product Detail（全屏）
 
// 优点（为什么最常用）
* ✅ 详情页自动隐藏 TabBar（因为不在 Tabs 内）
* ✅ 模态 / 登录 / 全屏页面好做
* ✅ 符合真实 App（电商 / 社交 / CMS）

// 实际行为
router.push('/product/123');
结果：
* TabBar ❌ 消失
* Header ✅ 由 Stack 控制

// 2. 隐藏Header、Tabbar的本质
核心原则（非常重要）
Header / TabBar 是否显示 ≠ UI 控制
本质 = 当前页面是否在对应 Layout 内

// 3. 如何隐藏TabBar
// 页面移除Tabs：结构决定显示/隐藏,是否显示 TabBar = 是否在 Tabs Layout 内
app/
  (tabs)/home/index.tsx
  product/[id].tsx

// 动态控制（不推荐）
navigation.getParent()?.setOptions({
  tabBarStyle: { display: 'none' },
});
// 问题：
* 生命周期复杂
* 返回时容易错乱
* 不优雅

// Modal/覆盖（次推荐）
<Stack.Screen
  name="modal/detail"
  options={{ presentation: 'modal' }}
/>

router.push('/modal/detail'); // TabBar 被覆盖（视觉上隐藏）

// 4. 如何隐藏Header
// 全局隐藏
<Stack screenOptions={{ headerShown: false }} />

// 单页面隐藏
<Stack.Screen
  name="product/[id]"
  options={{ headerShown: false }}
/>

// 动态控制
navigation.setOptions({ headerShown: false });
```

```ts
1. Tabs 是“容器”，不是页面
👉 只放：
* 首页
* 列表页
* 主入口

2. 详情页永远放 Tabs 外
👉 否则你一定会遇到：
* tabBar 难隐藏
* navigation 混乱

3. 不要用 hack 控制 UI
👉 用结构解决：结构 > 配置 > hack
```

```ts
Stack + Tabs 是主流架构；
TabBar 是否显示由页面是否属于 Tabs 决定，Header 是否显示由 Stack 配置控制。
```

2. 平台差异

```ts
// 不同平台使用不同tab实现
// 结构
components/
  app-tabs.native.tsx
  app-tabs.tsx
// 行为
* iOS / Android → 原生 Tabs
* Web → 自定义 Tabs
// 架构意义
一套路由，多端UI实现
// 非常适合：
* RN + Web 项目
* Design System 不同
```

3. 共享路由

```ts
// 多个Tab共享同一个页面
// 结构
app/(tabs)/
  (feed)/
    index.tsx

  (search)/
    search.tsx

  (feed,search)/
    users/
      [username].tsx
// 行为
/users/john
可以从：
* feed tab 进入
* search tab 进入
// 关键点
* (feed,search) = 共享 group
* URL 不变，但来源不同
// 架构意义
 同一个页面，不同入口，不同导航上下文
// 实际场景
* 用户详情页
* 商品详情页
// 深链注意
👉 如果直接打开：
/users/john
Expo Router 会：
👉 选择“字母顺序第一个 group”  
```

4. 权限路由

```ts
// 登录态控制页面访问
// 结构
app/
  (tabs)/        -> 需要登录
  sign-in.tsx
  create-account.tsx
  modal.tsx      -> 需要登录
// 核心写法
import { Stack } from 'expo-router';

<Stack>
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" />
    <Stack.Screen name="modal" />
  </Stack.Protected>

  <Stack.Protected guard={!isLoggedIn}>
    <Stack.Screen name="sign-in" />
    <Stack.Screen name="create-account" />
  </Stack.Protected>
</Stack>
// 行为
```

| **状态** | **访问**       |
| -------- | -------------- |
| 未登录   | 自动跳 sign-in |
| 已登录   | 进入 tabs      |

```ts
// 架构意义
路由 = 权限控制层
// 优势：
* 自动拦截 deep link
* 不需要手动 redirect
// Tabs权限控制
<Tabs.Protected guard={isVip}>
  <Tabs.Screen name="vip" />
</Tabs.Protected>
// 可以做到：
* 动态 tab
* feature flag
```

5. 不导航的模式

```ts
// 有时候不需要跳转
// 示例：登录弹窗
<>
  <Stack />
  <Modal visible={!isAuthenticated}>
    {/* 登录 UI */}
  </Modal>
</>
// 行为
* 页面不跳转
* 直接覆盖 UI
// 适合：
* 登录弹窗
* onboarding
* 权限提示
```

* 完整例子

```ts
app/
  _layout.tsx        -> Root Stack

  (auth)/
    sign-in.tsx
    register.tsx

  (tabs)/
    _layout.tsx      -> Tabs

    home/
      index.tsx
      detail.tsx

    search/
      index.tsx

    (home,search)/
      product/
        [id].tsx

  modal.tsx
// 导航结构
Root Stack
 ├── Auth
 ├── Tabs
 │     ├── Home (Stack)
 │     ├── Search (Stack)
 │     └── Shared Product Page
 └── Modal
```









### @tanstack/react-query理解

他是一个服务端状态管理库。（server state manager）

专门解决：

* API请求
* 缓存
* 同步
* 更新
* loading、error状态

```ts
// API 请求
自动执行 queryFn
// 缓存（核心能力）
['users'] → 对应一份缓存数据
下次再访问：
* 不重新请求（如果没过期）
* 或后台刷新
缓存只针对Query（读操作）
// 同步（重点）
多个组件：useQuery(['users'])，共享同一份数据。
// 更新
invalidateQueries(['users'])
自动重新请求 + 更新所有UI
// loading/error
const { isLoading, error } = useQuery(...)
不用自己维护状态
                                      
React Query = 请求 + 缓存 + 同步 + 状态 一体化
```

| **类型**        | **用途**                     |
| --------------- | ---------------------------- |
| Redux / Zustand | 本地状态（UI state）         |
| React Query     | **远程数据（Server state）** |

**什么是server state**

项目里通常由两种数据：

* Client State(本地状态)

```tsx
const [isOpen, setIsOpen] = useState(false);
// 特点：
* UI 控制
* 完全由你掌控
```

* Server State(React Query管的)

```tsx
const { data } = useQuery(['users'], fetchUsers);
// 特点：
* 来自服务器
* 会变化
* 需要缓存
* 可能过期
React Query就是专门管理这个
```

**核心思维**

```shell
传统思维：请求数据 -> 存State
React Query：声明我需要这份数据，剩下它帮你管
```

```ts
// 传统写法
useEffect(() => {
  setLoading(true);
  fetch('/users')
    .then(res => res.json())
    .then(setData)
    .finally(() => setLoading(false));
}, []);
// 问题：
* ❌ 没缓存
* ❌ 重复请求
* ❌ 页面切换丢数据
* ❌ 状态管理混乱
```

```ts
// React Query写法
const { data, isLoading, error } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
});
// 它自动帮你：
* 缓存数据
* 去重请求
* 后台刷新
* 错误处理
* loading 状态
```

**核心概念**

* Query查询

```ts
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers
})
// queryKey: 缓存的唯一标识，使用数组是为了表达参数化缓存。
['users']
['user', userId] // 这个user的缓存
['posts', { page: 1 }]
```

* Mutation（修改）

```ts
const mutation = useMutation({
  mutatationFn: createUser
})
// 用于：
* POST
* PUT
* DELETE
```

|              | **useQuery** | **useMutation** |
| ------------ | ------------ | --------------- |
| 是否自动执行 | ✅            | ❌               |
| 是否缓存     | ✅            | ❌               |
| 用途         | 获取数据     | 修改数据        |
| 是否需要 key | ✅            | ❌               |

* Cache（缓存）

```ts
// React Query会自动缓存
['users'] → 数据
// 下次再访问：
* 不请求（如果没过期）
* 或后台刷新（stale-while-revalidate）
```

* invalidation(失效机制)

```ts
queryClient.invalidateQueries(['users'])
//  数据更新后：→ 让缓存失效 → 自动重新请求

useMutation({
  onSuccess: () => {
    queryClient.invalidateQueries(['users']); // 标记数据过期，并触发重新获取。
  },
});
Mutation 不缓存，但会：
* 触发 Query 重新请求
* 或手动更新缓存

内部发生什么？
1. 标记 ['users'] 为 stale
2. 如果当前有组件在用：
    → 立即 refetch
3. UI 自动更新

React Query思路：不要直接改数据，而是让他重新获取
```

* Stale（过期概念）

```ts
useQuery({
  queryKek: ['users'],
  queryFn: fetchUsers,
  staleTime: 1000 * 60 * 5 // 5分钟不过期
})
```

**使用流程**

* 安装

```shell
npm install @tanstack/react-query
```

* 创建query client

```ts
// /lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query'
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes
      retry: 2,
    },
  },
});
// stale: 过期，数据是否需要重新请求
// gcTime: garbage collected, 数据是否被删除。数据多久不用就会被删除。
// 场景
A页面 → 用了 users
离开页面 → 没组件用它
10分钟后 → 被清除
```

* 在App中注入Provider

```tsx
import { QueryProvider } from '@tanstack/react-query'
import { queryClient } from './lib/queryClient'

export default function App() {
  return (
    <QueryProvider client={queryClient}>
      <Root />
    </QueryProvider>
  )
}
```

* 请求数据

```ts
const fetchUsers = async () => {
  const res = await fetch('https://api.com/users')
  return res.json()
}

const { data, error } = userQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
})
```

* 提交数据

```ts
const mutation = useMutation({
  mutationFn: (newUser) => {
    fetch('/users', {
      method: 'POST',
      body: JSON.stringify(newUser)
    })
  }
})
```

* 更新后刷新数据

```ts
const queryClient = useQueryClient()

const mutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    queryClient.invalidateQueries(['users'])
  }
})
```

**企业级用法**

* 自定义hook： 解耦 UI 和数据逻辑

```ts
export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });
};
// 页面
const { data } = useUsers();
```

* Query Key规范化

```ts
export const queryKeys = {
  users: ['users'],
  user: (id) => ['user', id],
};
```

* 错误处理统一

```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 1,
    },
  },
});
```

React Query本质是一个数据同步引擎，而不是简单的请求库。

它帮你解决的是：

```shell
- 缓存
- 同步
- 一致性
- 状态管理复杂度
```

**使用场景**

* 可以用（推荐用）

```shell
- 90%的业务API
- 用户信息
- 用户列表（分页、筛选）
- 详情页
- 表单提交（配合mutation）
```

* 不建议用

```shell
# 一次性请求
await fetch('/init-config');
特点：
* 只请求一次
* 不复用
* 不需要缓存
用 React Query 反而“重了”
# 强交互及时请求
onChangeText → 搜索 API
这种更适合：
* debounce + fetch
* 或专门搜索 hook
# 本地UI状态
isModalOpen ❌
```

* 判断公式

```shell
1.这个数据会被多个地方用吗？
👉 是 → 用 React Query
👉 否 → 不一定

2.这个数据需要缓存吗？
👉 是 → 用
👉 否 → 不用

3.数据会变化，需要同步 UI 吗？
👉 是 → 用
👉 否 → 不用
```



### svg图片处理

**项目中图片的使用**

React Native会根据不同屏幕密度的设备使用不同的图片，会自动匹配1倍、2倍和3倍图。不需要写任何逻辑。

* 不同类型图片的最佳实践

1. 小图标：使用1x，2x和3x：对清晰度高，拉伸会模糊

   推荐方式（更优解）：使用矢量图（svg、icon font）， 不失真，更省包体积

2. 普通UI图片（卡片图、插图）

   建议使用2x，3x（可以不提供1x）：现在几乎没有1x设备，减少资源数量。

3. 背景图、大图（banner、splash）

   不建议使用1x，2x和3x图

   推荐方案：用网络图

4. 全屏图、启动图

   特殊处理：iOS-Asset catalog，Android-drawable目录，Expo会帮你处理

* 最佳实践总结

  | **类型**    | **是否需要 1x/2x/3x** |
  | ----------- | --------------------- |
  | icon        | ✅ 必须（或用 SVG）    |
  | 小 UI 图    | ✅ 建议                |
  | 大图 / 背景 | ❌ 不需要              |
  | 网络图片    | ❌ 不需要              |

优先级：

1. **SVG（最优）**
2. PNG 多倍图
3. 网络图



**使用svg图片配置**

* react native中svg使用的两种方案

1. SVG文件 -> 组件

   - react-native-svg
   - react-native-svg-transformer

   ```tsx
   import Logo from '@/assets/icons/logo.svg'
   
   <Logo width={24} height={24} />
   ```

2. 手写SVG代码

   ```tsx
   import { Svg, Path } from 'react-native-svg'
   
   <Svg width={24} height={24}>
     <Path d="..." />
   </Svg>
   ```

   一般用于动态图、特殊动画。日常项目不用这个。

* 使用流程

  ```tsx
  // 1. 安装依赖
  pnpm add react-native-svg // 负责“渲染能力”
  pnpm add -D react-native-svg-transformer // 负责“告诉 Metro 如何把 svg 当源码模块处理”
  // react-native-svg,提供 React Native 渲染 SVG 的底层能力。React Native 默认不能像 Web 那样天然渲染 svg 标签。
  // react-native-svg 提供了像 Svg, Path, Circle 这些原生组件能力。如果没有它，即使你把 svg 文件转译成功了，App 也没法真正渲染。
  // react-native-svg-transformer: 让 Metro 在打包时，把 .svg 文件转换成 React 组件模块。
  // 比如你写：
  // import SwitchAgencyIcon from '@/assets/svg/switch-agency.svg';
  // 默认情况下，Metro 会把 .svg 当成一个普通 asset 文件，不会把它当组件处理。
  // 加了 react-native-svg-transformer 后，它会把 svg 内容转换成类似这样的组件结果：
  // const SvgComponent = props => <Svg ...>{/* paths */}</Svg>;
  // export default SvgComponent;
  // 这样你才能像组件一样用：
  // <SwitchAgencyIcon width={24} height={24} />
  
  // 2. 配置metro： metro.config.js
  const { getDefaultConfig } = require('expo/metro-config')
  
  const config = getDefaultConfig(__dirname)
  
  config.transformer = {
    ...config.transformer,
    babelTransformerPath: require.resolve('react-native-svg-transformer'),
  }
  
  config.resolver = {
    ...config.resolver,
    // 告诉metro：svg不再当普通静态资源处理
    assetExts: config.resolver.assetExts.filter(ext => ext !== 'svg'),
    // 告诉metro：svg要像ts、tsx、js一样，当源码模块去解析
    sourceExts: [...config.resolver.sourceExts, 'svg'],
  }
  
  module.exports = config
  
  // 3. TypeScript声明： declarations.d.ts
  // 所有 .svg 文件都被视为一个 React 组件，并且这个组件接收 SvgProps。
  // 负责“告诉 TypeScript：svg 模块默认导出的是一个可接收 SvgProps 的 React 组件”
  declare module '*.svg' {
    import React from 'react'
    import { SvgProps } from 'react-native-svg'
    const content: React.FC<SvgProps>
    export default content
  }
  
  // 4. 放svg文件
  assets/
    icons/
      home.svg
      profile.svg
  
  // 5. 直接使用
  import HomeIcon from '@/assets/icons/home.svg'
  
  <HomeIcon width={24} height={24} />
  ```

* 设计稿导出svg的正确方式

  不能直接拿设计稿导出的svg图片就用。因为其颜色和尺寸通常是写死的。

  通常使用SVGO或者SVGR进行SVG优化。用于：删除多余属性、转成currentColor、去掉inline style。

  ```tsx
  // 使用SVGO自动优化
  // 1. 安装
  pnpm add -D svgo
  // 2. 创建配置
  // svgo.config.js
  module.exports = {
    plugins: [
      'removeDimensions',
      'removeStyleElement',
      'removeAttrs',
    ],
  }
  
  // 3. 运行
  npx svgo assets/icons/*.svg
  它会帮你：
  * 删除 width/height
  * 清理垃圾属性
  * 压缩代码
  ```

  

**SVG图片使用的工程级写法**

不要在项目里到处直接用 SVG，建议统一封装：

```tsx
// 1. 封装一个Icon组件
import HomeIcon from '@/assets/icons/home.svg'

export const Icon = ({ name, size = 24, color = '#000' }) => {
  const icons = {
    home: HomeIcon,
  }

  const Component = icons[name]

  return <Component width={size} height={size} fill={color} />
}
// 2. 使用
<Icon name="home" size={24} color="blue" />
```























































