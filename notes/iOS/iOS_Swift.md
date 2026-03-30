## Swift

### Codable和CodingKeys的理解

```swift
struct BannerImage: Codable {
    let en: String
    let zh: String
    let zhTw: String
    let ms: String?
    
    enum CodingKeys: String, CodingKey {
        case en = "english"
        case zh = "chinese"
        case zhTw = "traditional_chinese"
        case ms = "malay"
    }
}
```

Codable是swift中的协议，包括Decodable和Encodable两个协议。

其中Decodable用于将JSON格式（或者其他格式）的数据转换为swift对象，而Encodable协议用于将swift对象转换为JSON格式（或者其他格式）的数据。

```swift
let json = """
{
  "en": "banner_en.png",
  "zh": "banner_zh.png",
  "zhTw": "banner_zhTw.png",
  "ms": "banner_ms.png"
}
""".data(using: .utf8)!

do {
    let banner = try JSONDecoder().decode(BannerImage.self, from: json)
    print(banner.zh)  // 输出: banner_zh.png
} catch {
    print(error)
}
```

CodingKeys是用来指定JSON字段名和结构体属性之间的映射关系。

如果JSON字段名和结构体属性之间的名字完全一样，则不需要指定CodingKeys，Swift会自动映射。

如果JSON字段名和结构体属性名字不完全一样，则需要指定CodingKeys告诉swift如何映射。

```swift
enum CodingKeys: String, CodingKey {
    case en = "english"
    case zh = "chinese"
    case zhTw = "traditional_chinese"
    case ms = "malay"
}
//	•	english → 对应属性 en
//	•	chinese → 对应属性 zh
//	•	chinese_traditional → 对应属性 zhTw
//	•	malay → 对应属性 ms
```



### Identifiable理解

Identifiable是Swift标准库里面的一个协议。

```swift
protocol Identifiable {
    associatedtype ID: Hashable
    var id: ID { get }
}
```

也就是结构体里面有一个属性id并且可以被哈希，就可以遵循`Idenfitifiable`协议。

作用：

在swiftui中用于标识元素的身份。比如在List或者foreash中，或者绑定的视图里面使用了一个数组，swiftui必须知道每一个元素的身份。以便追踪哪个元素被修改了，高效的更新或者删除，避免重复或者渲染错误。

如果没有遵循`Identifiable`，则需要手动指定

```swift
List(banners, id: \.id) { banner in
    Text(banner.label.title)
}
```

> **Hashable**
>
> Hashable是swift标准库的一个协议。如果一个对象可以生成一个唯一的哈希值，就是可以Hashable的。
>
> 用于让一个类型能被唯一标识并且高效比较。



### Error协议理解

```swift
enum NetworkError: Error {
    case invalidURL
    case serverError(String)
}
```

Error协议是swift标准库的一个协议，是一个空协议。用于标识可以抛出的错误类型。

任何遵循了`Error`协议的类型（通常是enum类型）都可以被throw出去。然后在调用处使用do-catch被捕获。

```swift
func fetchData(from urlString: String) throws {
    guard let url = URL(string: urlString) else {
        throw NetworkError.invalidURL
    }

    // 模拟：服务器返回错误信息
    let success = false
    if !success {
        throw NetworkError.serverError("Internal Server Error 500")
    }

    print("✅ Data fetched successfully")
}
```

```swift
do {
    try fetchData(from: "invalid-url")
} catch let error as NetworkError {
    switch error {
    case .invalidURL:
        print("❌ URL 无效")
    case .serverError(let message):
        print("❌ 服务器错误：\(message)")
    }
} catch {
    print("❌ 其他未知错误: \(error)")
}
```

`case serverError(String)`理解：

表示这是一个带关联值的枚举类型，表示这个错误类型不仅发生了服务器错误，还携带一段额外的信息。

* .invalidURL就像一个固定的标签
* .serverError(String)就像一个带参数的标签



### Swift闭包理解

**什么是闭包**

闭包是自包含的代码块，它可以在函数中传递和使用。它能捕获并保存它作用于外的常量或变量。

简单讲：闭包是可以向变量一样传递的函数。并且能记住创建时周围环境的值。



**闭包和函数的关系**

其实函数也是一种特殊的闭包。Swift把以下三种情况都称为闭包表达式。

| **类型**   | **示例**            | **是否具名** | **是否能捕获外部变量** |
| ---------- | ------------------- | ------------ | ---------------------- |
| 全局函数   | func add(a\:b\:)    | ✅ 有名字     | ❌ 不捕获               |
| 嵌套函数   | 函数内部定义函数    | ✅ 有名字     | ✅ 可以捕获             |
| 闭包表达式 | { (x, y) in x + y } | ❌ 匿名       | ✅ 可以捕获             |



**闭包的基本语法结构**

```swift
{ (参数列表) -> 返回类型 in
    // 闭包体（代码）
}
```

```swift
// 求和闭包
let sum = { (a: Int, b: Int) -> Int in
    return a + b
}
```

```swift
// 闭包的调用
print(sum(3, 5)) // 输出 8
```



**Swift中对闭包语法的简化**

* 完整写法

```swift
let sum = { (a: Int, b: Int) -> Int in
    return a + b
}
```

* 根据上下文推断类型

（比如函数参数要求 (Int, Int) -> Int）

```swift
let sum: (Int, Int) -> Int = { a, b in
    return a + b
}
```

* 单行闭包可以省略return

```swift
let sum: (Int, Int) -> Int = { a, b in a + b }
```

* 使用位置参数

```swift
let sum: (Int, Int) -> Int = { $0 + $1 }
```

* swift闭包省略规则总结

| **场景**       | **可以省略的部分**      | **原因**                         |
| -------------- | ----------------------- | -------------------------------- |
| 参数为空       | 可省略 () 和 in         | 没有参数，不需要标记输入部分     |
| 返回类型可推断 | 可省略 -> Type          | 编译器能根据 return 语句推断类型 |
| 单行表达式     | 可省略 return           | 自动返回单行表达式的结果         |
| 有参数但不命名 | 可省略参数名，用 $0, $1 | 隐式参数名                       |



**闭包的高级特性：捕获外部变量**

这是闭包名字的来源：**Closure = 闭合包裹外部变量**。

闭包可以“记住”它定义时作用域外的变量。

```swift
func makeCounter() -> () -> Int {
    var count = 0
    return {
        count += 1
        return count
    }
}

let counter = makeCounter()
print(counter()) // 1
print(counter()) // 2
print(counter()) // 3
```

- count 是 makeCounter() 内部的局部变量；
- 但是闭包捕获了它；
- 即使函数返回后，count 仍然存在于闭包中；
- 所以每次调用 counter()，count 都会递增。



**闭包使用场景**

| **场景**                            | **示例**                                   |
| ----------------------------------- | ------------------------------------------ |
| **回调 (completion handlers)**      | 网络请求完成时执行某个操作                 |
| **高阶函数**（map, filter, reduce） | 用函数式方式处理数组                       |
| **动画、异步代码**                  | UIView.animate 或 async 回调               |
| **立即执行闭包（IIFE）**            | 用于初始化默认值（你刚才看到的 { ... }()） |
| **事件处理、按钮点击**              | SwiftUI 的 .onTapGesture { ... }           |



### Result泛型类型理解

Result是Swift标准库中的一个非常实用的泛型类型，用于优雅的表示**成功或失败**两种结果。

是Swift错误处理和异步编程中经常会用到的核心概念。

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

- Result 是一个 **泛型枚举类型**
- 有两个可能的状态：
  - .success(value)：表示操作成功，携带一个成功结果（Success 类型）
  - .failure(error)：表示操作失败，携带一个错误（Failure 类型，必须遵守 Error 协议）

```swift
enum NetworkError: Error {
    case invalidURL
    case noData
}

func fetchUser() -> Result<String, NetworkError> {
    let success = Bool.random() // 随机成功或失败

    if success {
        return .success("User: Matt")
    } else {
        return .failure(.noData)
    }
}

let result = fetchUser()

switch result {
case .success(let user):
    print("✅ 获取成功：\(user)")
case .failure(let error):
    print("❌ 失败：\(error)")
}
```

**Result常用方法**

| **方法**         | **含义**                     | **示例**                                               |
| ---------------- | ---------------------------- | ------------------------------------------------------ |
| get()            | 获取成功值或抛出错误         | try result.get()                                       |
| map(_:)          | 转换成功值                   | .success("Matt").map { $0.uppercased() }               |
| mapError(_:)     | 转换错误类型                 | .failure(.noData).mapError { MyError($0) }             |
| flatMap(_:)      | 链式连接下一个可能失败的操作 | .success(2).flatMap { .success($0 * 2) }               |
| flatMapError(_:) | 处理错误并返回新的 Result    | .failure(.noData).flatMapError { .success("Default") } |

* get方法

```swift
let result = fetchUser()

do {
    let user = try result.get()
    print("✅ 用户: \(user)")
} catch {
    print("❌ 错误: \(error)")
}
```

* map方法

```swift
let result: Result<Int, Error> = .success(10)
let squared = result.map { $0 * $0 }
print(squared) // success(100)
```

* mapError

```swift
enum MyError: Error { case custom }

let result: Result<Int, NetworkError> = .failure(.noData)
let newResult = result.mapError { _ in MyError.custom }
```

**Result + async/await（现代用法）**

在 Swift 的新并发模型中，Result 仍然非常有用。

```swift
func fetchUserData() async -> Result<String, Error> {
    do {
        let data = try await someAsyncNetworkCall()
        return .success(data)
    } catch {
        return .failure(error)
    }
}
```



### Swift Concurrency（并发编程模型）理解

**什么是Swift Concurrency**

Concurrency（并发）指程序可以同时执行多个任务，比如同时下载图片，解析JSON，更新UI。

Swift 以前的异步写法是基于：

- 回调闭包（completion handlers）
- GCD (DispatchQueue)
- Combine 等响应式框架

这些方式虽然可行，但**代码层级嵌套多、错误难处理、可读性差**。

于是 Swift 5.5 引入了新的语言级语法：

> ✅ async / await / Task / Actor

这套机制与编译器、运行时深度集成，自动帮你管理线程和并发执行。



**核心概念：async/await**

* async: 声明异步函数

```swift
func fetchData() async -> String {
    return "Hello from server"
}
```

> * async表示函数可能会暂停执行（等待异步任务完成）
>
> * 调用异步函数时，必须使用await

* await：等待异步结果

```swift
func loadData() async {
    let message = await fetchData()
    print(message)
}
```

> * await表示等待fetchData执行完毕后再继续往下执行
> * 编译器会自动让出线程资源，不会阻塞主线程。



**与传统写法对比**

* 旧写法

```swift
func fetchUser(completion: @escaping (String?, Error?) -> Void) {
    DispatchQueue.global().async {
        completion("Matt", nil)
    }
}

fetchUser { user, error in
    if let user = user {
        print("User: \(user)")
    }
}
```

* 新写法

```swift
func fetchUser() async throws -> String {
    return "Matt"
}

Task {
    do {
        let user = try await fetchUser()
        print("User: \(user)")
    } catch {
        print("Error: \(error)")
    }
}
```

| **旧写法**     | **新写法**                   |
| -------------- | ---------------------------- |
| 使用闭包       | 使用 await                   |
| 嵌套回调       | 顺序执行（看起来像同步代码） |
| 线程切换难控制 | 自动切换、非阻塞             |
| 错误用参数返回 | 直接用 try / throw           |



**结构化并发**

Swift 的并发模型是**结构化的**。

也就是说，**子任务的生命周期由父任务管理**，确保不会产生“悬挂的后台任务”。

并发模型核心原则：

> **每一个异步任务都应该有明确的生命周期，由父任务管理。**
>
> 子任务不会“悬挂”在外面，保证代码可预测、易管理、不会泄露资源。

核心特点：

| **特点**     | **说明**                                                 |
| ------------ | -------------------------------------------------------- |
| **父子关系** | 子任务的生命周期依赖父任务，父任务结束前会等待子任务完成 |
| **自动取消** | 父任务取消时，所有子任务会自动被取消                     |
| **并行执行** | 使用 async let 可以在父任务中并行启动子任务              |
| **可组合**   | 可以通过 await 等待子任务结果，代码像同步一样顺序执行    |

举例：并行执行多个任务

```swift
func fetchUserName() async -> String {
    print("Fetching name...")
    try? await Task.sleep(nanoseconds: 1_000_000_000) // 1 秒
    return "Matt"
}

func fetchUserAge() async -> Int {
    print("Fetching age...")
    try? await Task.sleep(nanoseconds: 2_000_000_000) // 2 秒
    return 25
}

func fetchUserProfile() async {
    async let name = fetchUserName()  // 并行启动
    async let age = fetchUserAge()    // 并行启动

    // 等待结果
    let userName = await name
    let userAge = await age

    print("User: \(userName), age: \(userAge)")
}

Task {
    await fetchUserProfile()
}
```

**特点**：

1. async let 创建子任务，但这些子任务在父任务结束前会自动等待完成。
2. 不会出现“悬挂任务”或未管理的线程。
3. 父任务可以通过 await 获取结果，顺序代码风格清晰。



**Task： 创建并发任务**

Task 是 Swift 并发的最小执行单元，可以在任意上下文启动异步任务。

1. 创建任务

```swift
Task {
    let message = await fetchData()
    print(message)
}
```

- **立即启动**任务
- 任务的生命周期由创建它的上下文（通常是父任务或全局作用域）管理

2. task返回值和错误

```swift
func fetchData() async throws -> String {
    if Bool.random() { return "Success" }
    throw NSError(domain: "NetworkError", code: 500)
}

Task {
    do {
        let result = try await fetchData()
        print("✅ Result:", result)
    } catch {
        print("❌ Error:", error)
    }
}
```

- Task 可以执行 async throws 的函数
- 可以用 await 等待结果
- 错误用 try/catch 捕获

3. Task.detached: 创建独立任务

```swift
Task.detached {
    let message = await fetchUserName()
    print("Detached Task: \(message)")
}
```

- detached 任务**独立于父任务**：
  - 不受父任务取消影响
  - 不继承父任务的执行上下文（例如优先级、Actor 隔离）
- 用途：
  - 完全独立的后台任务
  - 与父任务逻辑无关的任务
- ⚠️ 注意：不要滥用，否则可能引起数据竞争或未管理的任务泄漏

4. Task.sleep: 暂停任务

```swift
print("Start")
try? await Task.sleep(nanoseconds: 2_000_000_000) // 2 秒
print("End")
```

- Task.sleep(nanoseconds:) 是异步暂停任务
- **非阻塞主线程**：
  - 不会卡 UI 或线程
- 返回 throws，因为任务可能被取消
- 单位是**纳秒**：
  - 1_000_000_000 ns = 1 秒

5. Task和async let的对比

| **特性**   | **Task**                                            | **async let**            |
| ---------- | --------------------------------------------------- | ------------------------ |
| 生命周期   | 父任务管理或独立                                    | 由父任务管理             |
| 并行       | 可完全独立                                          | 并行，但父任务必须等待   |
| 继承上下文 | Task -> 默认继承父任务上下文；Task.detached -> 独立 | 自动继承父任务上下文     |
| 用途       | 启动任意异步任务                                    | 在结构化并发中创建子任务 |



**Actor: 并发中的数据安全守卫者**

在多线程访问共享资源时，可以出现数据竞争（Data Race）。

Swift提供了Actor来保证同一时间只有一个任务访问共享状态。

```swift
actor Counter {
    var value = 0
    
    func increment() {
        value += 1
    }
}

let counter = Counter()
Task {
    await counter.increment()
    print(await counter.value)
}
// actor 自动保证线程安全（相当于一个轻量级锁）。
```



**async/await + throw组合**

异步函数可以像同步函数一样抛错：

```swift
func fetchData() async throws -> String {
    throw NetworkError.serverError("Internal Server Error")
}

Task {
    do {
        let result = try await fetchData()
        print(result)
    } catch {
        print("❌ 错误:", error)
    }
}
```

async throws 同时支持异步等待与错误传播，语法统一、自然。



**总结**

| **概念**       | **含义**                   | **示例**                         |
| -------------- | -------------------------- | -------------------------------- |
| async          | 声明异步函数，可能暂停执行 | func fetchData() async -> String |
| await          | 等待异步函数完成           | let data = await fetchData()     |
| Task           | 启动新的异步任务           | Task { await fetchData() }       |
| async let      | 创建并行异步任务           | async let name = fetchName()     |
| throws + async | 异步函数可抛错             | func fetchData() async throws    |
| actor          | 保证并发安全的引用类型     | actor Counter { var value = 0 }  |

Swift Concurrency 通过 async/await、Task 和 Actor让异步代码像同步代码一样简单、可读、类型安全，同时自动处理线程调度与数据安全。



### 计算属性、存储属性以及lazy属性理解

**存储属性**

占用内存，真实存储值。

```swift
struct Person {
    var name: String
    var age: Int
}
```

**计算属性**

不占用内容，在使用时通过计算获取。其本质是一个函数。

```swift
struct Square {
    var side: Double
    var area: Double {
        get {
            return side * side
        }
        
        set {
            side = sqrt(newValue)
        }
    }
}
```

**lazy属性**

是一种的特殊的存储属性，只有在第一次访问时才初始化。lazy属性必须使用`var`，因为let修饰的是常量，必须在初始化时赋值，而lazy需要延迟初始化，必须得使用`var`修饰。

```swift
class DataLoader {
    lazy var data: String = {
        print("Loading data...")
        return "Large Data"
    }()
}

let loader = DataLoader()

print("Before access")
print(loader.data)
print(loader.data)

// 输出
Before access
Loading data...
Large Data
Large Data
```



### init初始化方法理解

在swift中，init方法用于创建对象并初始化属性。

swift的初始化机制比很多语言严格，要求所有属性在对象使用前必须初始化完成。

swift中常见的init方法主要由：

```swift
1. Designated Initializer（指定初始化器）
2. Convenience Initializer（便利初始化器）
3. Required Initializer（必须初始化器）
4. Failable Initializer（可失败初始化器）
5. Default Initializer（默认初始化器）
6. Memberwise Initializer（成员初始化器 - struct）
7. Overriding Initializer（重写初始化器）
```

**Designated Initializer(指定初始化器)**

这是类的主要初始化方法。

作用：

* 负责初始化所有属性
* 调用父类的初始化方法

```swift
class Person {

    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}
```

初始化流程

```swift
Person.init
↓
属性初始化
↓
对象创建完成
```

使用场景：

对象必须提供的数据。

```swift
class ProductViewController: UIViewController {

    let productId: String

    init(productId: String) {
        self.productId = productId
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

**Convenience Initializer(便利初始化器)**

便利初始化器是辅助初始化方法。

特点：

* 必须调用同类的指定初始化器
* 不能直接调用父类的init方法

```swift
class Person {

    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    convenience init(name: String) {
        self.init(name: name, age: 0)
    }
}
```

流程：

```swift
convenience init
↓
调用指定 init
↓
对象创建
```

使用场景：

* 提供简化创建方式

**Required Initialzer（必须初始化器）**

所有子类必须实现这个方法。

```swift
class Animal {

    var name: String

    required init(name: String) {
        self.name = name
    }
}

// 子类
class Dog: Animal {

    required init(name: String) {
        super.init(name: name)
    }
}
```

使用场景：

* 框架设计
* 协议约束

**Failable Initializer（可失败初始化器）**

有时候对象可能无法创建。

例如：
```swift
URL
JSON
配置文件
```

```swift
struct User {

    var age: Int

    init?(age: Int) {

        if age < 0 {
            return nil
        }

        self.age = age
    }
}

let user = User(age: -1)

print(user)   // nil
```

使用场景：

* 数据校验
* 解析失败
* 非法参数

```swift
URL(string: "invalid url")
// 返回 URL?
```

**Default Initializer（默认初始化器）**

如果所有属性都有默认值，swift会自动生成init方法。

```swift
class User {

    var name = "Unknown"
    var age = 0
}

let user = User()
```

使用场景：

* 简单数据模型

**Memberwise Initializer（成员初始化器）**

这个只存在于struct，swift会自动生成。

```swift
struct User {

    var name: String
    var age: Int
}
// swift自动生成
init(name: String, age: Int)
```

如果自己写了init方法，则swift不会自动生成。

**Overriding Initializer（重写初始化器）**

子类可以重写父类init。

```swift
class Animal {

    var name: String

    init(name: String) {
        self.name = name
    }
}

// 子类
class Dog: Animal {

    var breed: String

    init(name: String, breed: String) {
        self.breed = breed
        super.init(name: name)
    }
}
```

**UIKit中常见init方法**

```swift
// xib调用
init(nibName:bundle:)
// Storyboard 会调用：
init(coder:)
```

| **类型**         | **作用**        | **使用场景** |
| ---------------- | --------------- | ------------ |
| Designated init  | 主要初始化方法  | 类初始化     |
| Convenience init | 辅助初始化      | 简化创建     |
| Required init    | 子类必须实现    | 框架设计     |
| Failable init    | 可能创建失败    | 数据校验     |
| Default init     | 自动生成        | 属性有默认值 |
| Memberwise init  | struct 自动生成 | 结构体       |
| Override init    | 子类初始化      | 继承体系     |



### swift的两阶段初始化理解

swift为了保证对象安全初始化，规定初始化必须分为两个阶段。

```swift
Phase 1：初始化自己的属性
Phase 2：父类初始化完成后才能使用 self
```

流程

```swift
子类初始化
↓
Phase 1
子类初始化自己的属性
↓
调用 super.init()
↓
父类初始化自己的属性
↓
Phase 2
对象完全初始化完成
↓
可以使用 self
```

**为什么必须先self.xxx = value**

```swift
class Animal {

    var name: String

    init(name: String) {
        self.name = name
    }
}

// 子类
class Dog: Animal {

    var age: Int

    init(name: String, age: Int) {

        self.age = age
        super.init(name: name)
    }
}

// Swift 要求：
// 在调用 super.init 之前
// 子类必须初始化自己的所有属性
// 父类可能访问self.age,就会 读取未初始化内存。
```

**swift初始化规则**

swift有4个安全规则，其中两个最重要。

* 规则1

```swift
子类必须先初始化自己的属性，再调用super.init()方法
```

* 规则2

```swift
再super.init()前，不能使用self
```

**swift对使用self的定义**

* 初始化属性

```swift
self.age = age
//这是 给属性赋初始值。
//Swift 允许。
//因为对象正在构建。
```

* 访问或调用self

```swift
print(self.age)
self.doSomething()
self.view
//这是 使用对象。
//Swift 不允许。
//因为对象还没构建完成。
```



### extension的用法理解

extension允许你再不修改原始代码的情况下，为已有类型添加新功能。

```swift
extension = 给已有类型添加能力
```

**基本用法**

```swift
extension TypeName {
    // 新功能
}

extension String {
    func greet() {
        print("Hello \(self)")
    }
}

"Matt".greet()
```

**常见用法**

* 添加计算属性
* 添加方法
* 添加构造函数
* 添加下标
* 遵循协议
* 添加嵌套类型

**添加计算属性**

extension可以添加计算属性，但是不能添加存储属性。

```swift
extension Int {
    var squared: Int {
        return self * self
    }
}
```

**添加方法**

```swift
extension Int {
    func repeatAction(_ action: () -> Void) {
        for _ in 0..<self {
            action()
        }
    }
}

3.repeatAction {
    print("Hello")
}
```

修改值的方法：

对于值类型（struct、enum），如果要修改自身，需要使用mutating

```swift
extension Int {
    mutating func square() {
        self = self * self
    }
}

var num = 4
num.square()
print(num)
```

**添加初始化方法**

extension可以增加新的 **init** 方法

```swift
struct Person {
    var name: String
    var age: Int
}

extension Person {
    init(name: String) {
        self.name = name
        self.age = 0
    }
}

let p = Person(name: "Tom")
```

**协议扩展**

swift中extension最强大的地方是protocol extension。

```swift
// 定义协议
protocol Greetable {
    func greet()
}

// 扩展协议
extension Greetable {
    func greet() {
        print("Hello")
    }
}

// 实现
struct Person: Greetable {}

// 使用
let p = Person()
p.greet()

// 输出
Hello

// 协议提供默认实现
```

**添加subscript**

extension也可以添加下标访问。

```swift
extension Array {
    subscript(safe index: Int) -> Element? {
        if index >= 0 && index < count {
            return self[index]
        }
        return nil
    }
}

let arr = [10, 20, 30]

print(arr[safe: 1])  // Optional(20)
print(arr[safe: 5])  // nil
```

**extension的组织代码能力**

很多iOS项目都会使用extension拆分代码结构。

```swift
class LoginViewController: UIViewController {

}

// UI
extension LoginViewController {
    func setupUI() {
        // UI code
    }
}

// Actions
extension LoginViewController {
    @objc func loginTapped() {
        // button action
    }
}

// Network
extension LoginViewController {
    func requestLogin() {
        // network request
    }
}
```

**extension + 协议**

```swift
protocol Loadable {
    func showLoading()
    func hideLoading()
}

// 默认实现
extension Loadable where Self: UIViewController {
    func showLoading() {
        print("show loading")
    }

    func hideLoading() {
        print("hide loading")
    }
}
```

**extension限制**

* 不能添加存储属性
* 不能重写方法

**extension添加嵌套类型**

```swift
extension SomeType {
    enum State {
        case loading
        case success
        case failed
    }
}
```

```swift
struct NetworkManager {

}

extension NetworkManager {
    enum APIError: Error {
        case networkError
        case invalidResponse
        case decodingError
    }
}
// 形成命名空间, 避免命名冲突
```

extension可以添加类型：

| **类型**  | **是否允许** |
| --------- | ------------ |
| enum      | ✅            |
| struct    | ✅            |
| class     | ✅            |
| typealias | ✅            |

```swift
extension NetworkManager {
    struct Request {
        let url: String
        let method: String
    }
}

extension NetworkManager {
    typealias Completion = (Result<Data, Error>) -> Void
}
```

extension添加类型的架构意义：

* 状态管理

```swift
extension ViewModel {
    enum State { }
}
```

* Action定义

```swift
extension LoginViewModel {
    enum Action {
        case login
        case logout
    }
}
```

* 路由定义

```swift
extension Router {
    enum Route {
        case home
        case profile
        case settings
    }
}
```

好处：

* 命名空间管理

```swift
NetworkManager.APIError
```

* 代码逻辑归属清晰

```swift
LoginViewModel.State
```

* 避免全局类型污染

```swift
// 项目中会有很多
State
Action
Error
Response
```



### associatedtype理解

**associatedtype是协议中定义的一个占位类型，具体类型由实现协议的类型确定。可以理解为协议里的泛型。**

```swift
protocol Container {
    associatedtype Item
    
    func add(_ item: Item)
    func getItem(at index: Int) -> Item
}
// Item 是一个占位类型,但协议 不知道 Item 是什么类型。真正的类型由 实现这个协议的类型决定。
```

```swift
struct IntContainer: Container {

    typealias Item = Int
    
    var items: [Int] = []

    mutating func add(_ item: Int) {
        items.append(item)
    }

    func getItem(at index: Int) -> Int {
        return items[index]
    }
}
// Item = Int

struct StringContainer: Container {

    typealias Item = String

    var items: [String] = []

    mutating func add(_ item: String) {
        items.append(item)
    }

    func getItem(at index: Int) -> String {
        return items[index]
    }
}
// Item = String
```

标准库

```swift
protocol Collection {

    associatedtype Element
    associatedtype Index

}
```

**associatedtype vs 泛型**

```swift
struct Stack<T> {
    var items: [T]
}
// 类型在 使用时确定：
Stack<Int>()

protocol Stack {
    associatedtype Element
}
// 类型在 实现协议时确定：
struct IntStack: Stack {
    Element = Int
}
```

**associatedtype限制**

带associatedtype的procotol不是完整的类型。不能直接作为变量类型。

```swift
protocol Container {
    associatedtype Item
    
    func add(_ item: Item)
    func getItem(at index: Int) -> Item
}
```

必须使用：

| **方式** | **用途**         |
| -------- | ---------------- |
| 泛型     | 最常见           |
| some     | 返回某种具体类型 |
| any      | 类型擦除         |

* 使用泛型：泛型约束

```swift
func process<C: Container>(_ container: C) {
    let item = container.getItem(at: 0)
    print(item)
}
// 如果写var container: Container会报错
// Protocol 'Container' can only be used as a generic constraint
// because it has Self or associated type requirements
// 因为 协议不知道 Item 是什么类型。
```

* 使用Any

```swift
var container: any Container
// 注意any Container只是类型擦除后的协议类型，能调用的方法受限。
// any Container 只能调用不依赖 Item 的方法
```

* 使用some

```swift
func makeContainer() -> some Container {
    IntContainer()
}
// some表示：返回某一种具体类型，但不告诉你是哪种
```

**associatedtype类型约束**

associatedtype还可以添加限制。

```swift
protocol Cache {
    associatedtype Key: Hashable
    associatedtype Value
}
// Key 必须遵循 Hashable

struct UserCache: Cache {
    typealias Key = String
    typealias Value = User
}
```

**where约束**

```swift
protocol Container {
    associatedtype Item
    
    func add(_ item: Item)
}

protocol ComparableContainer: Container
where Item: Comparable {
    
}
// Item 必须是 Comparable
struct NumberContainer: ComparableContainer {
    typealias Item = Int
}
```







## UIKit

### UIViewController生命周期理解

**生命周期流程图**

```swift
init
↓
loadView
↓
viewDidLoad
↓
viewWillAppear
↓
viewDidAppear
↓
(用户操作 / 页面交互)
↓
viewWillDisappear
↓
viewDidDisappear
↓
deinit
```

分为三个阶段：

* 页面创建
* 页面显示
* 页面消失

**生命周期详解**

1. init初始化

当VC被创建时执行。

```swift
class MyViewController: UIViewController {

    init() {
        super.init(nibName: nil, bundle: nil)
        print("init")
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

let vc = MyViewController()
```

使用场景：

* 依赖注入
* 初始化变量

2. loadView

创建view，如果没有使用storyboard或者xib，系统会自动创建view。你也可以自己重写。

```swift
override func loadView() {

    let view = UIView()
    view.backgroundColor = .white

    self.view = view
}
// 一般开发中很少重写
```

3. viewDidLoad（最常用）

当view已经加载到内存，但是没有显示时调用。只会调用一次。

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    print("viewDidLoad")

    setupUI()
    fetchData()
}
```

使用场景：

* 初始化UI
* 注册tableView
* 网络请求
* 绑定viewModel

```swfit
func setupUI() {
    view.backgroundColor = .white
}
```

4. viewWillAppear

页面即将显示时调用，每次显示时都会调用。

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    print("viewWillAppear")
}
```

使用场景

* 刷新数据
* 隐藏/显示导航栏

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    navigationController?.setNavigationBarHidden(false, animated: true)
}
```

5. viewDidAppear

页面已经完全显示时调用

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)

    print("viewDidAppear")
}
```

使用场景：

* 开始动画
* 开始视频
* 开始统计

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)

    startAnimation()
}
```

6. viewWillDisappear

页面即将消失时调用。例如：

* push新页面
* pop当前页面
* dismiss

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)

    print("viewWillDisappear")
}
```

使用场景：

* 停止任务
* 保存数据
* 暂停动画

7. viewDidAppear

页面已经消失时调用。

```swift
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)

    print("viewDidDisappear")
}
```

使用场景：

* 释放资源
* 停止播放
* 取消订阅

8. deinit（销毁）

当VC被销毁时调用。

```swift
deinit {
    print("ViewController released")
}
```

使用场景：

* 确认内存是否释放
* 取消通知

```swift
deinit {
    NotificationCenter.default.removeObserver(self)
}
```

































## SwiftUI

### ObservableObject，@Published， @StateObject理解

**SwiftUI核心思想**

SwiftUI的核心思想：数据驱动界面。

SwiftUI的哲学：

* UI是数据变化的函数
* 当数据发生变化时，UI自动更新，而不是你手动刷新。

要实现这个机制，SwiftUI就需要知道数据变化了，就需要：

* 一个可以被观察的对象（ObservableObject）
* 和能触发变化通知的属性（@Published）



**ObservableObject是什么**

ObservableObject是一个协议，表示：这个对象可以被SwiftUI视图观察，当对象内部数据发生变化时，视图自动刷新。

```swift
public protocol ObservableObject: AnyObject {
    var objectWillChange: ObservableObjectPublisher { get }
}
```

这个 objectWillChange 是一个 **Combine 发布者（Publisher）**，

当对象中任何被 @Published 修饰的属性发生变化时，它就会发出通知。



**@Published是什么**

@Published是一个属性包裹器（property wrapper），用来声明当这个属性发生变化时，要通知所有订阅者（包括SwiftUI视图）。

```swift
@Published var name = "Matt"
```

它的作用是：

* 自动生成一个Publisher（发布者）
* 当属性改变时，调用objectWillChange.send()
* 触发SwiftUI界面刷新

```swift
let user = UserViewModel()

user.$name.sink { newName in
    print("名字变了：\(newName)")
}

user.name = "Jack"
// 输出：名字变了：Jack
```

user.$name 是 @Published 自动生成的 Publisher。当 name 改变时，它会发出新的值，Combine 订阅者会收到通知。



**SwiftUI中使用**

```swift
struct UserView: View {
    @StateObject var viewModel = UserViewModel()

    var body: some View {
        VStack {
            Text("👤 Name: \(viewModel.name)")
            Text("🎂 Age: \(viewModel.age)")

            Button("Change Name") {
                viewModel.name = "Alice"
            }
        }
        .padding()
    }
}
```

- @StateObject 告诉 SwiftUI：我拥有并观察这个 ObservableObject。
- 当 viewModel.name 或 viewModel.age 改变时，视图会自动重绘。



**@StateObject vs @ObservableObject**

| **属性包装器**     | **说明**                   | **生命周期**                  |
| ------------------ | -------------------------- | ----------------------------- |
| @StateObject       | SwiftUI 创建并持有这个对象 | 一般用于视图的“源头”          |
| @ObservedObject    | 外部传入的可观察对象       | 一般用于子视图接收            |
| @EnvironmentObject | 从全局环境中获取共享对象   | 全局状态共享（比如 AppState） |

```swift
struct ParentView: View {
    @StateObject var viewModel = UserViewModel()

    var body: some View {
        ChildView(viewModel: viewModel)
    }
}

struct ChildView: View {
    @ObservedObject var viewModel: UserViewModel

    var body: some View {
        Text("Child Name: \(viewModel.name)")
    }
}
```

- ParentView 负责创建并持有状态（@StateObject）
- ChildView 只观察它（@ObservedObject）
- 当 viewModel.name 变化，父子视图都会更新。



**内部工作机制**

```swift
class UserViewModel: ObservableObject {
    @Published var name = "Matt"
}
```

编译器其实会生成类似这样的代码：

```swift
class UserViewModel: ObservableObject {
    var objectWillChange = ObservableObjectPublisher()

    var name = "Matt" {
        willSet {
            objectWillChange.send()
        }
    }
}
```

SwiftUI 监听 objectWillChange，当 send() 被调用时，重新渲染相关视图。



### GeometryReader理解

**什么是GeometryReader是什么？**

GeometryReader是一个容器视图，它会将父视图的几何信息（尺寸，坐标）作为参数提供给闭包中的内容。

> 它可以“读出（read）”父布局提供的几何信息，比如：
>
> - 宽度（geometry.size.width）
> - 高度（geometry.size.height）
> - 在全局坐标系中的位置（geometry.frame(in:)）

```swift
GeometryReader { geometry in
    // geometry 是 GeometryProxy 类型
    // 提供 size、frame 等几何信息
}
```



**Geometry的核心类型：GeometryProxy**

闭包中的参数 geometry 是一个 **GeometryProxy** 对象，

它包含了关于当前容器的空间信息。

| **属性**       | **类型**   | **说明**                         |
| -------------- | ---------- | -------------------------------- |
| size           | CGSize     | 当前容器的宽度和高度             |
| safeAreaInsets | EdgeInsets | 安全区域（刘海、底部栏）内边距   |
| frame(in:)     | CGRect     | 返回在某个坐标空间内的位置和尺寸 |

```swift
GeometryReader { geometry in
    Text("Width: \(geometry.size.width)")
}
```



**基础示例**

```swift
struct GeometryExampleView: View {
    var body: some View {
        GeometryReader { geometry in
            VStack {
                Text("Width: \(geometry.size.width)")
                Text("Height: \(geometry.size.height)")
            }
            .frame(width: geometry.size.width, height: geometry.size.height)
            .background(Color.blue.opacity(0.2))
        }
        .background(Color.gray.opacity(0.3))
    }
}
```

> * GeometryReader 会占满父容器提供的所有空间（默认行为）。
>
> * 你可以读取它的宽高，动态调整内部布局。



**使用场景**

* 自适应布局

```swift
// 根据父视图宽度动态调整子视图的大小：
GeometryReader { geometry in
    Circle()
        .fill(Color.purple)
        .frame(width: geometry.size.width * 0.5,
               height: geometry.size.width * 0.5)
        .position(x: geometry.size.width / 2,
                  y: geometry.size.height / 2)
}
// 无论父容器多大，圆形始终居中并占宽度一半。
```

* 元素相对定位

```swift
// 利用 geometry.frame(in:) 获取全局坐标：
GeometryReader { geometry in
    Text("Hello, World!")
        .background(Color.yellow)
        .onAppear {
            let globalFrame = geometry.frame(in: .global)
            print("Global position: \(globalFrame.origin)")
        }
}
// 输出类似：Global position: (x: 20, y: 120)
// .frame(in: .global) 表示在屏幕坐标系中的位置，
// .local 表示在父视图内部坐标系中的位置。
```

* 实现响应式布局

```swift
GeometryReader { geometry in
    if geometry.size.width > geometry.size.height {
        // 横屏布局
        HStack {
            Text("Landscape")
            Spacer()
        }
        .padding()
    } else {
        // 竖屏布局
        VStack {
            Text("Portrait")
            Spacer()
        }
        .padding()
    }
}
```

* 自定义动画或者响应效果

```swift
// 例如实现“视差滚动（Parallax Scroll）”：
struct ParallaxHeader: View {
    var body: some View {
        GeometryReader { geometry in
            let offset = geometry.frame(in: .global).minY
            
            Image("header")
                .resizable()
                .scaledToFill()
                .frame(height: 200 + (offset > 0 ? offset : 0))
                .clipped()
                .offset(y: offset > 0 ? -offset : 0)
        }
        .frame(height: 200)
    }
}
// 滚动时通过 geometry.frame(in: .global) 读取图片的滚动偏移量，实现动态放大或移动。
```



### .task和.onAppear的理解

在swiftui中，.task和.onAppear都是view出现在屏幕上时触发代码的修饰器，他们的设计目标不同：

```swift
.onAppear → 生命周期回调（类似 UIKit 的 viewWillAppear）
.task     → 用于执行 async/await 的异步任务
```

**.onAppear是普通生命周期事件，.task是swift Concurrency的任务管理机制**

| **特性**             | .onAppear          | .task                    |
| -------------------- | ------------------ | ------------------------ |
| 主要用途             | 生命周期事件       | 执行异步任务             |
| 是否支持 async/await | ❌ 不直接支持       | ✅ 原生支持               |
| 任务取消             | ❌ 不会自动取消     | ✅ View 消失会自动 cancel |
| 调用时机             | View 出现          | View 出现                |
| 推荐场景             | UI更新 / analytics | 网络请求 / async任务     |

**核心区别：async支持**

.onAppear不支持async，而.task原生支持async.

**任务取消机制**

.task有一个非常重要的特性，当view消失时，任务会自动取消。

```swift
.task {
    await loadData()
}
```

**.task(id:)**

.task还有一个非常强大的功能，当id发生变化时会重新执行task。

```swift
struct UserView: View {

    var userId: String

    var body: some View {

        Text("User")

        .task(id: userId) {
            await loadUser(userId)
        }
    }
}

userId 改变
↓
旧任务取消
↓
新任务启动
```

**使用场景**

* 页面数据加载推荐使用`.task`

```swift
struct ProfileView: View {

    @State private var user: User?

    var body: some View {

        Text(user?.name ?? "Loading...")

        .task {
            user = await fetchUser()
        }
    }
}
```

* 监听数据参数变化加载数据推荐使用`.task`

```swift
.task(id: searchText) {

    results = await search(searchText)
}
```

* 埋点、Analytics推荐`.onAppear`

```swift
.onAppear {
    Analytics.track("ProfileView")
}
```











## Others

### 函数式编程

函数式编程（Functional Programming，FP）是一种编程范式，它把函数视为**第一等公民**，强调**不可变性**（immutability）和无副作用（pure functions）。

**函数式编程的核心概念**

| **概念**                             | **含义**                                                     | **Swift 示例**                                               |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **纯函数 (Pure Function)**           | 同样输入 → 永远同样输出，且没有副作用（不修改外部状态、不依赖外部变量） | func add(_ a: Int, _ b: Int) -> Int { return a + b }         |
| **不可变性 (Immutability)**          | 数据不可修改，只能创建新值                                   | let numbers = [1, 2, 3] let doubled = numbers.map { $0 * 2 } |
| **高阶函数 (Higher-order function)** | 函数可以作为参数或返回值                                     | func operate(_ x: Int, _ y: Int, using op: (Int, Int) -> Int) -> Int { return op(x, y) } |
| **函数组合 (Function Composition)**  | 将小函数组合成更复杂的函数                                   | let increment = { $0 + 1 } let double = { $0 * 2 } let incThenDouble = { double(increment($0)) } |



**Swift中的函数式编程特性**

函数式思想在Swift中大量使用，尤其是在集合操作，响应式编程（如RxSwift，Combine中）

1. 使用高阶函数操作集合

```swift
let numbers = [1, 2, 3, 4, 5]

// map: 映射为新数组
let squared = numbers.map { $0 * $0 }  // [1, 4, 9, 16, 25]

// filter: 筛选出偶数
let even = numbers.filter { $0 % 2 == 0 }  // [2, 4]

// reduce: 归约为单个值
let sum = numbers.reduce(0) { $0 + $1 }  // 15
```

这些函数式方法没有改变原始数组，而是**返回新值**。



2. 链式组合函数

```swift
let result = numbers
    .filter { $0 % 2 != 0 }   // 奇数 [1,3,5]
    .map { $0 * $0 }          // 平方 [1,9,25]
    .reduce(0, +)             // 求和 35
```

这段代码是纯函数式写法，**没有可变状态，也没有副作用**。



3. 使用闭包表达函数式逻辑

```swift
let greet: (String) -> String = { name in
    "Hello, \(name)!"
}

print(greet("Matt")) // 输出: Hello, Matt!
```

闭包在 Swift 中是 FP 的核心工具。



4. 组合函数

可以把多个函数拼接成一个更复杂的函数：

```swift
func increment(_ x: Int) -> Int { x + 1 }
func double(_ x: Int) -> Int { x * 2 }

func compose<A, B, C>(
    _ f: @escaping (B) -> C,
    _ g: @escaping (A) -> B
) -> (A) -> C {
    return { f(g($0)) }
}

let incThenDouble = compose(double, increment)
print(incThenDouble(3)) // 输出 8
```



### Coordinator，Router和Navigator理解

**项目结构**

```swift
App
│
├── Router
│     └── AppRouter.swift
│
├── Modules
│
│   ├── Product
│   │      ├── ProductCoordinator.swift
│   │      ├── ProductNavigator.swift
│   │      ├── ProductListVC.swift
│   │      └── ProductDetailVC.swift
│   │
│   ├── Checkout
│   │      ├── CheckoutCoordinator.swift
│   │      ├── CheckoutNavigator.swift
│   │      ├── CheckoutVC.swift
│   │      └── PaymentVC.swift
│   │
│   └── Order
│          ├── OrderCoordinator.swift
│          ├── OrderNavigator.swift
│          └── OrderHistoryVC.swift
// 每个模块都有
// Coordinator
// Navigator
// ViewController
// Router 在全局。
```

**Navigator的作用（场景内页面跳转）**

Navigator只做一件事：管理当前模块内部页面的跳转。

```tex
ProductList
 ↓
ProductDetail
```

```swift
class ProductNavigator {

    private let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func showProductList() {
        let vc = ProductListViewController()
        navigationController.pushViewController(vc, animated: true)
    }

    func showProductDetail(productId: String) {
        let vc = ProductDetailViewController(productId: productId)
        navigationController.pushViewController(vc, animated: true)
    }
}
```

作用：封装 push / pop

使用场景：**同一个模块内部页面跳转**

```tex
ProductList → ProductDetail
Checkout → Payment
Payment → Result
```

**Coordinator作用（模块流程控制）**

Coordinator是模块的控制中心。

负责：

1 业务流程
2 页面跳转逻辑
3 状态管理
4 复杂判断

```swift
class ProductCoordinator {

    private let navigator: ProductNavigator
    private let router: AppRouter

    init(navigator: ProductNavigator, router: AppRouter) {
        self.navigator = navigator
        self.router = router
    }

    func start() {
        navigator.showProductList()
    }

    func didSelectProduct(productId: String) {
        navigator.showProductDetail(productId: productId)
    }

    func buyProduct(productId: String) {
        router.openCheckout(productId: productId)
    }
}
```

作用：管理 Product 模块流程

使用场景：控制模块流程

```tex
用户点击商品
 ↓
显示详情
 ↓
点击购买
 ↓
跳转 Checkout 模块
```

**Router的作用（跨模块导航）**

负责模块之间跳转。

```text
Product → Checkout
Checkout → OrderHistory
```

```swift
class AppRouter {

    private let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func openCheckout(productId: String) {

        let navigator = CheckoutNavigator(navigationController: navigationController)

        let coordinator = CheckoutCoordinator(
            navigator: navigator,
            router: self,
            productId: productId
        )

        coordinator.start()
    }

    func openOrderHistory() {

        let navigator = OrderNavigator(navigationController: navigationController)

        let coordinator = OrderCoordinator(
            navigator: navigator,
            router: self
        )

        coordinator.start()
    }
}
```

负责：

创建模块

创建Coordinator

启动模块

**完整示例**

假设用户点击：

```tex
ProductList 页面
点击商品
```

流程

```tex
ProductListVC
   ↓
ProductCoordinator.didSelectProduct()
   ↓
ProductNavigator.showProductDetail()
   ↓
push ProductDetailVC
```

用户点击：

```tet
Buy Button
```

流程

```tet
ProductDetailVC
   ↓
ProductCoordinator.buyProduct()
   ↓
Router.openCheckout()
   ↓
CheckoutCoordinator.start()
   ↓
CheckoutNavigator.showCheckout()
   ↓
push CheckoutVC
```

用户支付成功：

```tet
PaymentVC
   ↓
CheckoutCoordinator.paymentSuccess()
   ↓
Router.openOrderHistory()
   ↓
OrderCoordinator.start()
   ↓
OrderNavigator.showOrderHistory()
```

| **组件**       | **负责什么**     |
| -------------- | ---------------- |
| ViewController | UI + 用户事件    |
| Navigator      | 当前模块页面跳转 |
| Coordinator    | 模块流程控制     |
| Router         | 跨模块导航       |

总结

```tet
Navigator
= 页面跳转工具

Coordinator
= 模块流程管理者

Router
= 模块路由器
```



### Stevia用法理解

Stevia是一个Swift DSL（Domain Specific Language）布局库，用于简化AutoLayout代码。

核心目标：用更少、更直观的代码描述布局。

**Stevia核心思想**

* DSL布局语言

DSL：用接近自然语言的方式写布局。

```swift
view1.top(10).left(10)
view2.centerInContainer()
```

* 链式API

```swift
view.top(10).left(20).width(100)
```

* Stack布局

类似SwiftUI和UIStackView

```swift
stack(
    view1,
    view2,
    view3
)
```

**基本用法**

* 添加view

```swift
let titleLabel = UILabel()
let loginButton = UIButton()

subviews {
  titleLabel,
  loginButton
}
```

**基础布局**

* Top/Left/Right/Bottom

```swift
titleLabel.top(100).left(20).right(20)
// top = 100
// left = 20
// right = 20
```

* 设置宽高

```swift
button.width(200).height(50)
```

* Center

```swift
button.centerInContainer()
// 等价于
// centerX
// centerY
```

**布局相对关系**

```swift
titleLabel
↓
textField
↓
button

textField.Top == titleLabel.Bottom + 20
button.Top == textField.Bottom + 20

// 注意：
Top/Bottom/Left/Right 都是Stevia提供的布局对象。
```

```swift
Title
TextField
Password
Button
```

```swift
let title = UILabel()
let email = UITextField()
let password = UITextField()
let login = UIButton()

view.subviews(title, email, password, login)

title.top(100).centerHorizontally()

email.Top == title.Bottom + 30
email.left(20).right(20).height(40)

password.Top == email.Bottom + 20
password.left(20).right(20).height(40)

login.Top == password.Bottom + 30
login.centerHorizontally().width(200).height(50)
```

**Stack布局（Stevia强项）**

Stevia支持stack layout。

类似：

```swift
UIStackView
SwiftUI VStack
```

* Vertical Stack

```swift
stack(
    20,
    title,
    email,
    password,
    login
)

// vertical stack
// spacing = 20
```

* 设置边距

```swift
stack(
    20,
    title,
    email,
    password,
    login
).left(20).right(20)
```

* 设置高度

```swift
stack(
    20,
    title.height(40),
    email.height(40),
    password.height(40),
    login.height(50)
)
```

* Horizontal Stack

```swift
hstack(
    10,
    cancelButton,
    confirmButton
)

// 效果：cancelButton | confirmButton
```

**填满父视图**

```swift
child.fillContainer()
// 等价于
top = 0
bottom = 0
left = 0
right = 0
```

**比例布局**

```swift
leftView.Width == 0.3 * view.Width
rightView.Width == 0.7 * view.Width
```

**size**

```swift
view.size(100)
// 等价
width = 100
height = 100
```

**常见API**

| **API**              | **作用** |
| -------------------- | -------- |
| top()                | 顶部约束 |
| bottom()             | 底部约束 |
| left()               | 左边     |
| right()              | 右边     |
| width()              | 宽       |
| height()             | 高       |
| centerHorizontally() | 水平居中 |
| centerVertically()   | 垂直居中 |
| centerInContainer()  | 居中     |
| fillContainer()      | 填满     |



**StackView理解**

在iOS中，UIStackView是一个布局容器。用于自动把子view按顺序排列。

例如：

```swift
Label
TextField
Button
```

StackView 会自动帮你布局：

```swift
Label
   ↓
TextField
   ↓
Button
```

你不需要给每个 view 写：

```swift
top constraint
bottom constraint
```

* 核心思想： 一个容器按顺序排列子view
* axis： axis 决定 **排列方向**。

```swift
let stack = UIStackView()

stack.axis = .vertical
stack.spacing = 20

stack.addArrangedSubview(title)
stack.addArrangedSubview(email)
stack.addArrangedSubview(password)
stack.addArrangedSubview(login)
```

* stevia中的stack

其实是 **UIStackView 的语法糖**。

```swift
stack(
    20,
    title,
    email,
    password,
    login
)
// 等价于
let stack = UIStackView()

stack.axis = .vertical
stack.spacing = 20

stack.addArrangedSubview(title)
stack.addArrangedSubview(email)
stack.addArrangedSubview(password)
stack.addArrangedSubview(login)
```



### iOS布局系统理解

**translatesAutoresizingMaskIntoConstraints理解**

其作用是：**是否把 frame/autoresizingMask 转换成 constraint**。

* iOS布局方式

| **系统**         | **时代**      |
| ---------------- | ------------- |
| AutoresizingMask | iOS 2 ~ iOS 6 |
| AutoLayout       | iOS 6+        |

* AutoresizingMask

早期iOS通过`view.autoresizingMask`布局。

```swift
view.autoresizingMask = [
    .flexibleWidth,
    .flexibleHeight
]
// 父view变大
// 子view自动调整
```

* TranslatesAutoresizingMaskIntoConstraints

这个属性含义：是否把 autoresizingMask 转换成 AutoLayout constraint

```swift
view.translatesAutoresizingMaskIntoConstraints = true
// 系统自动根据 frame / autoresizingMask,生成 constraint 默认值为true
// 用于兼容老代码：
view.frame = CGRect(...)
// 系统会自动生成constraints
```

* autoLayout 

如果自己写constraints

```swift
NSLayoutConstraint.activate([
    view.topAnchor.constraint(equalTo: superview.topAnchor)
])

+ 

translatesAutoresizingMaskIntoConstraints = true

// 系统会
自动constraint
+
你写的constraint

// 导致冲突：
Unable to simultaneously satisfy constraints
```

正确写法

```swift
let button = UIButton()

button.translatesAutoresizingMaskIntoConstraints = false

view.addSubview(button)

NSLayoutConstraint.activate([
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])
```

* Stevia和Snapkit

其在添加view时都会内部自动设置

```swift
translatesAutoresizingMaskIntoConstraints = false
```

* 总结

true → 系统帮你生成constraint

false → 只使用你写的constraint



**iOS布局系统整体流程**

UIKit每一帧UI更新时，会执行：

```swift
1 更新约束 (updateConstraints)
2 计算布局 (layout)
3 设置 frame
4 调用 layoutSubviews
5 渲染到屏幕
```

完整流程

```swift
setNeedsUpdateConstraints
        ↓
updateConstraints
        ↓
AutoLayout 求解 constraint
        ↓
计算 frame
        ↓
layoutSubviews
        ↓
draw
```

**frame和AutoLayout关系**

AutoLayout的最终结果就是frame。

```swift
Constraint
     ↓
AutoLayout Engine
     ↓
计算 frame
     ↓
view.frame

button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
// 系统会计算
button.frame = CGRect(...)
```

**为什么有时候frame是0**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    print(view.frame)
}

// 输出： (0,0,0,0) 原因是布局还没有发生
// 真正布局发生在 viewDidLayoutSubviews

override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    
    print(view.frame)
}

// UIKit 布局完成后调用的方法,你可以在这里调整子view的frame
override func layoutSubviews() {
    super.layoutSubviews()
    
    imageView.frame = CGRect(
        x: 0,
        y: 0,
        width: bounds.width,
        height: bounds.height
    )
}
// layoutSubviews = frame布局入口
```

**setNeedsLayout**

```swift
// 当UI改变时：
view.backgroundColor = .red
// 系统不会立即重新布局，如果想触发布局
view.setNeedsLayout() 
// 标记需要重新布局，但不会立即执行，而是下一次Runloop再执行。
```

**layoutIfNeeded**

```swift
// 如果你希望立即布局：
view.layoutIfNeeded()
// 作用：立即执行layout pass
constraint
↓
计算 frame
↓
layoutSubviews
```

```swift
buttonWidth.constant = 200

UIView.animate(withDuration: 0.3) {
    self.view.layoutIfNeeded()
}
// 动画会发生
// layoutIfNeeded, 触发 constraint → frame 变化,所以 UI 可以动画。
```

**setNeedsLayout vs layoutIfNeeded**

| **方法**       | **作用**     |
| -------------- | ------------ |
| setNeedsLayout | 标记需要布局 |
| layoutIfNeeded | 立即布局     |

**setNeedsUpdateConstraints**

```swift
// AutoLayout还有一个
setNeedsUpdateConstraints()
// 作用：标记需要更新的约束
// 触发 updateConstraints()
override func updateConstraints() {
    super.updateConstraints()
}
```

**完整layout cycle**

```swift
// UIKit每次局部流程
setNeedsUpdateConstraints
      ↓
updateConstraints
      ↓
AutoLayout Engine
      ↓
计算 frame
      ↓
layoutSubviews
      ↓
draw
```

**什么时候触发布局**

```swift
1 view.frame改变
2 constraint改变
3 setNeedsLayout
4 layoutIfNeeded
5 view加入父view
6 屏幕旋转
```

**AutoLayout为什么必须layoutIfNeeded**

```swift
constraint.constant = 200
UIView.animate(withDuration: 0.3) {
    self.view.layoutIfNeeded()
}

// AutoLayout → frame变化 → 动画
```



### RxDataSource理解

**RxDataSource是什么**

```swift
RxDataSource = RxSwfit的扩展库

作用：用Rx的方式绑定 UITableView 和 UICollectionView的数据。

如果只用RxCocoa，则只能这样绑定：
items.bind(to: collectionView.rx.items(cellIdentifier: "Cell")) { index, item, cell in
}
这个方式只支持单section。

实际App：
首页
 ├ section 1 banner
 ├ section 2 menu
 ├ section 3 list

这种情况就需要使用RxDataSource。
```

**RxDataSource和RxSwift关系**

```swift
RxSwift
   │
   ├ RxCocoa
   │
   └ RxDataSources (第三方库)

RxDataSources不是RxSwift的官方库，是社区维护的。需要单独安装。
```

**RxDataSource解决什么**

```swift
普通RxCocoa只能绑定一个列表。
Observable<[Item]>

复杂UI需要 Observable<[Section]>
[
 section1,
 section2,
 section3
]
所以引入了SecionModel
```

**SectionModel是什么**

```swift
SectionModel表示一个section。
结构：
public struct SectionModel<Section, Item> {
    public var model: Section
    public var items: [Item]
}

SectionModel
     |
     |-- section 信息
     |-- items

例如：
section title
section items
```

```swift
// 例子：购物首页

// 结构
section1
  商品A
  商品B

section2
  商品C
  商品D

// 数据结构
let sections = [
    SectionModel(
        model: "Hot",
        items: ["iPhone", "Mac"]
    ),

    SectionModel(
        model: "New",
        items: ["Vision Pro", "iPad"]
    )
]
```

**RxCollectionViewSectionedReloadDataSource**

```swift
这是CollectionView的dataSource。

作用：把section数据绑定到CollectionView。

定义：
RxCollectionViewSectionedReloadDataSource<SectionModel>
即CollectionView DataSource支持 section reload
```

```swift
// 简单例子
// 定义数据
let sections = Observable.just([
    SectionModel(model: "A", items: ["1","2","3"]),
    SectionModel(model: "B", items: ["4","5","6"])
])

// 创建dataSource
let dataSource = RxCollectionViewSectionedReloadDataSource<
    SectionModel<String, String>
>(
    configureCell: { dataSource, collectionView, indexPath, item in
        let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: "Cell",
            for: indexPath
        )
        return cell
    }
)

// 绑定数据
sections
.bind(to: collectionView.rx.items(dataSource: dataSource)
.disposed(by: disposeBag)
```

整体流程

```swift
Observable<[SectionModel]>
        │
        ▼
RxCollectionViewSectionedReloadDataSource
        │
        ▼
UICollectionView
```

复杂项目: 电商首页

```swift
section1 banner
section2 menu
section3 product

// 数据
enum HomeSection {
    case banner
    case menu
    case product
}

// SectionModel
let sections = [
    SectionModel(
        model: HomeSection.banner,
        items: banners
    ),

    SectionModel(
        model: HomeSection.menu,
        items: menus
    ),

    SectionModel(
        model: HomeSection.product,
        items: products
    )
]

// cell根据section渲染
configureCell: { dataSource, collectionView, indexPath, item in
    let section = dataSource.sectionModels[indexPath.section]

    switch section.model {

    case .banner:
        let cell = collectionView.dequeueReusableCell(...)
        return cell

    case .menu:
        let cell = collectionView.dequeueReusableCell(...)
        return cell

    case .product:
        let cell = collectionView.dequeueReusableCell(...)
        return cell
    }
}
```

**CollectionView中的DataSource**

在UIKit中，CollectionView不会自己知道数据，它需要一个数据提供者（DataSource）。

所以必须实现：UICollectionViewDataSource

```swift
class ViewController: UIViewController, UICollectionViewDataSource {

    var items = ["A","B","C"]

    func collectionView(
        _ collectionView: UICollectionView,
        numberOfItemsInSection section: Int
    ) -> Int {
        return items.count
    }

    func collectionView(
        _ collectionView: UICollectionView,
        cellForItemAt indexPath: IndexPath
    ) -> UICollectionViewCell {

        let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: "Cell",
            for: indexPath
        )

        return cell
    }
}

// 然后
collectionView.dataSource = self

// 流程
UICollectionView
       │
       ▼
collectionView.dataSource
       │
       ▼
问你两个问题：

1 多少个cell
2 每个cell长什么样

// 本质
DataSource = UI数据提供者
```

**传统方式 vs Rx方式**

```swift
// 传统方式更新时：
items.append("D")
collectionView.reloadData()

// 手动管理
reload
insert
delete
```

```swift
// Rx方式

// 核心思想
UI = 数据流
数据变化UI自动更新

let items = Observable.just(["A","B","C"])
items
.bind(to: collectionView.rx.items(
    cellIdentifier: "Cell"
)) { index, item, cell in

}
.disposed(by: disposeBag)

// 流程
Observable
     │
     ▼
Rx binding
     │
     ▼
UICollectionView

Rx方式 = 用 Observable 数据流驱动UI，而不是手动 reloadData
```

**为什么需要RxDataSource**

```swift
RxCocoa只支持单section，对于多section，需要使用RxDataSources。例如：

banner
menu
product

RxDataSource提供SectionModel：
SectionModel
    │
    ├ section info
    └ items

而RxCollectionViewSectionedReloadDataSource实际上就是UICollectionViewDataSource的Rx版本实现。
不用再写
numberOfItemsInSection
cellForItemAt
而是使用closure提供

let dataSource = RxCollectionViewSectionedReloadDataSource<
    SectionModel<String, String>
>(
    configureCell: { dataSource, collectionView, indexPath, item in

        let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: "Cell",
            for: indexPath
        )

        return cell
    }
)
// 理解
RxCollectionViewSectionedReloadDataSource<Section>
// 泛型
SectionModel<String,String>
// 意思
sectionModel = String
itemModel = String
// 结构
SectionModel
   │
   ├ model = String
   └ items = [String]
```

| **参数**       | **含义**       |
| -------------- | -------------- |
| dataSource     | 当前数据源     |
| collectionView | collectionView |
| indexPath      | 位置           |
| item           | 当前item数据   |



### UICollectionView布局系统

**UICollectionView初始化理解**

iOS的CollectionView有两个完全不同的系统：

* 数据系统 -> DataSource
* 布局系统 -> Layout

```swift
// 创建CollectionView并指定它的布局方式。
UICollectionView(
    frame: CGRect,
    collectionViewLayout: UICollectionViewLayout
)

// 结构
UICollectionView
     │
     ├ DataSource (数据)
     │
     └ Layout (布局) // layout 决定 cell如何排列
```

**UICollectionViewLayout是什么**

UICollectionViewLayout是CollectionView的布局引擎。

决定：

* cell大小
* cell位置
* cell排列方式
* section间距
* 滚动方式

UIKit有3中布局方式

| **布局**                            | **说明**     |
| ----------------------------------- | ------------ |
| UICollectionViewFlowLayout          | 最传统布局   |
| UICollectionViewCompositionalLayout | 现代复杂布局 |
| 自定义Layout                        | 完全自定义   |

**UICollectionViewFlowLayout（旧布局）**

```swift
let layout = UICollectionViewFlowLayout()

layout.itemSize = CGSize(width: 100, height: 100)
layout.scrollDirection = .vertical
layout.minimumLineSpacing = 10

// 效果： grid布局
// 例如：
[1][2][3]
[4][5][6]
// 缺点：
复杂布局很难实现
```

**UICollectionViewCompositionalLayout**

```swift
iOS13引入的现代CollectionView布局系统。
// 特点：
像搭积木一样组合布局，所以叫compositional（组合式）
// 布局结构
Item
 ↓
Group
 ↓
Section
 ↓
Layout
// 核心结构
Layout
 └ Section
      └ Group
           └ Item
```

| **概念** | **含义**    |
| -------- | ----------- |
| Item     | 一个cell    |
| Group    | 一组cell    |
| Section  | 一个section |
| Layout   | 整个布局    |

```swift
func createLayout() -> UICollectionViewLayout {

    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .fractionalHeight(1.0)
    )

    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .absolute(100)
    )

    let group = NSCollectionLayoutGroup.horizontal(
        layoutSize: groupSize,
        subitem: item,
        count: 2
    )

    let section = NSCollectionLayoutSection(group: group)

    return UICollectionViewCompositionalLayout(section: section)
}

// 效果
[1][2]
[3][4]
```

```swift
// App Store 首页
Section1 横向滚动banner

Section2 grid

Section3 列表

let layout = UICollectionViewCompositionalLayout { sectionIndex, environment in

    switch sectionIndex {

    case 0:
        return bannerSection()

    case 1:
        return gridSection()

    default:
        return listSection()
    }

}

// 横向滚动卡片
func bannerSection() -> NSCollectionLayoutSection {

    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1),
        heightDimension: .fractionalHeight(1)
    )

    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(0.8),
        heightDimension: .absolute(200)
    )

    let group = NSCollectionLayoutGroup.horizontal(
        layoutSize: groupSize,
        subitems: [item]
    )

    let section = NSCollectionLayoutSection(group: group)

    section.orthogonalScrollingBehavior = .groupPaging

    return section
}
```

**NSCollectionLayoutSize**

 传统布局（FlowLayout）

```swift
layout.itemSize = CGSize(width: 100, height: 100)

// 问题：不同屏幕，不同布局，很难适配
```

CompositionalLayout引入：比例尺寸，自适应尺寸。而`NSCollectionLayoutSize`就是描述宽高如何计算。

**NSCollectionLayoutSize结构**

```swift
NSCollectionLayoutSize(
    widthDimension: NSCollectionLayoutDimension,
    heightDimension: NSCollectionLayoutDimension
)
// 结构
NSCollectionLayoutSize
     │
     ├ widthDimension
     └ heightDimension
// 每个dimenssion都是：NSCollectionLayoutDimension
```

**NSCollectionViewLayoutDimenssion有三种类型**

| **类型**          | **含义**       |
| ----------------- | -------------- |
| .fractionalWidth  | 父容器宽度比例 |
| .fractionalHeight | 父容器高度比例 |
| .absolute         | 固定尺寸       |
| .estimated        | 预估尺寸       |

* absoulute：固定尺寸

```swift
let size = NSCollectionLayoutSize(
    widthDimension: .absolute(100),
    heightDimension: .absolute(100)
)
// 意思：
宽 = 100
高 = 100
```

* fractionalWidth：宽度比例

```swift
widthDimension: .fractionalWidth(0.5)
// 意思：宽 = 父容器宽度 × 0.5
let itemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(0.5),
    heightDimension: .absolute(100)
)
// 效果
[ 50% ][ 50% ]
```

* fractionalHeight：高度比例

```swift
heightDimension: .fractionalHeight(1.0)
// 意思：高度 = 父容器高度 × 1.0
let itemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1),
    heightDimension: .fractionalHeight(1)
)
// 表示item填满group
```

* estimated：动态高度

这是自适应cell的关键。

```swift
heightDimension: .estimated(100)
// 意思：
预估高度 = 100
真实高度 = AutoLayout计算

// 常见于
聊天
评论
文本列表

let itemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1),
    heightDimension: .estimated(100)
)
// 高度自动增长
```

**常见Item Size写法**

* 两列grid

```swift
let itemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(0.5),
    heightDimension: .absolute(150)
)
// 效果：
[ item ][ item ]
```

* 三列grid

```swift
widthDimension: .fractionalWidth(1.0)
Group count = 3

// 效果
[1][2][3]
```

```swift
let itemSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1),
    heightDimension: .fractionalHeight(1)
)

let item = NSCollectionLayoutItem(layoutSize: itemSize)

let groupSize = NSCollectionLayoutSize(
    widthDimension: .fractionalWidth(1),
    heightDimension: .absolute(120)
)

let group = NSCollectionLayoutGroup.horizontal(
    layoutSize: groupSize,
    subitem: item,
    count: 2
)
// 最终布局
Group width = screen width

Item width = group width / 2

结果：

[ item ][ item ]
```



### UITableView/UICollectionView中ContentOffset理解

**四个属性关系图**

```swift
                 系统UI环境
      (NavigationBar / TabBar / SafeArea)
                       │
                       ▼
           contentInsetAdjustmentBehavior
             (是否允许系统自动调整)
                       │
                       ▼
开发者设置 contentInset  +  系统自动Inset
                       │
                       ▼
             adjustedContentInset
                       │
                       ▼
           contentOffset 控制滚动位置

// 核心公式
adjustedContentInset = contentInset + 系统自动添加的Inset
```

**ContentInset（开发者控制）**

```swift
contentInset = 内容区域的内边距
类型UIEdgeInsets

UICollectionView
┌───────────────────────┐
│        top inset      │
│   ┌───────────────┐   │
│   │               │   │
│L  │   content     │ R │
│   │               │   │
│   └───────────────┘   │
│      bottom inset     │
└───────────────────────┘

// 示例
collectionView.contentInset = UIEdgeInsets(
    top: 20,
    left: 16,
    bottom: 40,
    right: 16
)
```

常见使用场景：

* 卡片布局

```swift
| margin | card | margin |

collectionView.contentInset.left = 16
collectionView.contentInset.right = 16
```

* 底部按钮

```swift
列表
列表
列表
---------
[按钮]

collectionView.contentInset.bottom = 80
```

**contentInsetAdjustmentBehavior**

这是决定系统是否自动添加inset的开关。

```swift
UIScrollView.ContentInsetAdjustmentBehavior
```

| **模式**       | **含义**           |
| -------------- | ------------------ |
| automatic      | 系统自动调整       |
| scrollableAxes | 只在可滚动方向调整 |
| never          | 永远不自动调整     |
| always         | 总是调整           |

* automatic（默认）

```swift
系统自动根据 SafeArea / NavigationBar 调整

NavigationBar
─────────────
CollectionView

// 系统自动调整
contentInset.top = navBarHeight
```

* never

```swift
scrollView.contentInsetAdjustmentBehavior = .never

系统不再自动调整

NavigationBar
─────────────
Cell1 (被挡住)
Cell2
```

* scrollableAxes

```swift
只在可以滚动方向调整

水平collectionView

系统只调整：left/right
```

**系统自动添加的inset怎么来的**

系统会根据safeArea计算。

```swift
// 主要来源有：
NavigationBar
StatusBar
TabBar
HomeIndicator

// 示例1：NavigationBar
NavigationBar：44
StatusBar：44
SafeAreaTop：88
系统自动：contentInset.top = 88

// 示例2：底部Tabbar
TabBar = 49
HomeIndicator = 34
SafeAreaBottom：83
系统自动：contentInset.bottom = 83

┌─────────────────────┐
│ StatusBar           │
│ NavigationBar       │
├─────────────────────┤
│                     │
│     SafeArea        │
│                     │
├─────────────────────┤
│ TabBar              │
│ HomeIndicator       │
└─────────────────────┘
```

**adjustedContentInset**

```swift
这是系统最终使用的contentInset。
公式：adjustedContentInset = contentInset + safeAreaInset

collectionView.contentInset.top = 10
safeAreaTop = 88
adjustedContentInset.top = 98
```

**contentOffset**

```swift
当前滚动位置。

scrollView 初始位置：contentOffset.y = -adjustedContentInset.top

// 示例
adjustedContentInset.top = 88
contentOffset.y = -88 // 初始
// 图示
contentOffset = -88
 ↓
刚好显示第一个cell

// 判断顶部的正确写法
if scrollView.contentOffset.y <= -scrollView.adjustedContentInset.top
```

**四个属性使用场景**

* 带导航栏的列表

```swift
collectionView.contentInset = UIEdgeInsets(
    top: 0,
    left: 16,
    bottom: 20,
    right: 16
)

collectionView.contentInsetAdjustmentBehavior = .automatic

// 系统
NavigationBar = 88

// 最终
adjustedContentInset.top = 88
```

* 沉浸式header

```swift
// UI
图片header
NavigationBar透明

collectionView.contentInsetAdjustmentBehavior = .never
```

* 自定义导航栏

```swift
collectionView.contentInsetAdjustmentBehavior = .never
scrollView.contentInset.top = 88
```

**完整关系列表**

```swift
           NavigationBar
           StatusBar
           TabBar
           SafeArea
                │
                ▼
 contentInsetAdjustmentBehavior
                │
                ▼
      系统自动添加 contentInset
                │
                ▼
      contentInset (开发者)
                │
                ▼
         adjustedContentInset
                │
                ▼
           contentOffset
```



































































