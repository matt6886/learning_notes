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

闭包是自包含的代码块，它可以在函数中传递和使用。它能捕获并保存它作用域外的常量或变量。

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



### Swift Concurrency vs CGD理解

Swift Concurrency是Swift提供的一套结构化并发模型。用于更安全、更清晰的编写并发代码。

它主要包含以下几个核心组件：

* async/await
* Task
* TaskGroup
* Actor

GCD存在问题

* 回调地狱：嵌套严重，可读性差，难以维护

```swift
DispatchQueue.global().async {
    fetchA {
        fetchB {
            fetchC {
                ...
            }
        }
    }
}
```

* 状态管理混乱：多线程容易产生数据竞态（race condition)， 需要加锁。GCD没有提供数据安全模型。

```swift
var count = 0

DispatchQueue.global().async {
    count += 1
}
```

* 线程管理复杂：需要手动切换线程，关注执行上下文

```swift
DispatchQueue.main.async { }
DispatchQueue.global().async { }
```

* 错误处理困难：分散在回调中，不统一。

```swift
fetch { result in
    switch result {
    case .success:
    case .failure:
    }
}
```

* 生命周期不可控：无父子任务，无自动取消，容易泄露。

**Swift Concurrency核心思想**

* 结构化并发：任务必须有清晰的生命周期和层级关系

```swift
Task {
    await taskA()
    await taskB()
}
// 父任务控制子任务
// 自动取消
// 生命周期清晰
```

* async/await：同步写法表达异步，替代callback和completion hanlder，代码更直观，容易维护。

```swift
let result = await fetch()
```

* Actor: 数据隔离中心。Actor是Swift Concurrency的最大亮点。保证同一时间只有一个任务访问数据。GCD需要手动加锁。

```swift
actor Counter {
    var value = 0
    
    func increment() {
        value += 1
    }
}
```

* Task: 轻量（类似协程），自动调度，支持取消。

```swift
Task {
    await fetch()
}
```

* TaskGroup：用于管理一组并发子任务的结构化工具。解决：并发执行多个任务 + 收集结果。

```swift
func fetchAll() async -> [Int] {
    await withTaskGroup(of: Int.self) { group in
        
        group.addTask { await fetchA() }
        group.addTask { await fetchB() }
        group.addTask { await fetchC() }
        
        var results: [Int] = []
        
        for await result in group {
            results.append(result)
        }
        
        return results
    }
}
// 所有任务一起跑，动态添加任务，按完成顺序返回，生命周期绑定（group结束，任务结束），自动取消，所有任务自动取消。

// async let 适用于少量固定并发任务
async let a = fetchA()
async let b = fetchB()
```

**Swift Concurrency常见坑**

* Actor重入问题
* 忘记await
* 主线程问题
* Task泄露
* 强引用问题
* Cancellation不会自动终止，必须手动检查。
* 线程不固定

### Actor vs @MainActor理解

**Actor**

是用于保护共享数据的并发隔离模型。

* 内部数据不能被外部直接访问
* 同一时间只允许一个任务访问。
* 避免数据竞态。

```swift
actor Counter {
    var value = 0
}
```

**@MainActor**

是一个全局Actor，用于保证代码在主线程执行。

```swift
// 传统写法
DispatchQueue.main.async {
  label.text = "Hello"
}

// Swift Concurrency
await MainActor.run {
  label.text = "Hello"
}

// 或者
@MainActor
func updateUI() {
  label.text = "Hello"
}
```

MainActor = 数据隔离 + 线程约束



### Runloop理解

Runloop是一个事件循环机制，用于让线程持续运行，并在有任务时处理任务，没有任务时休息。本质就是让线程活着，有事做事，没事睡觉。

**核心思维**

Runloop = while(true)循环。

```swift
while (true) {
    if 有任务 {
        处理任务
    } else {
        休眠
    }
}
```

Runloop本质是一个事件驱动循环。负责监听事件（Event）、分发事件（Diapatch)、控制线程休眠/唤醒。

**核心结构**

Runloop有几个关键组件：

* Source（事件源）：输入事件

  * source0：手动触发，不依赖系统

  ```swift
  performSelector
  ```

  * source1: 系统触发，基于内核（Mac Port），系统自动唤醒，例如触摸事件、网络事件

* Timer： 定时器，定时触发任务

```swift
Timer.scheduledTimer
```

* Observer: 观察者，用于监听Runloop的状态变化，例如即将进入循环、即将休眠、被唤醒。

* Mode： Runloop的运行模式。Runloop同一时间只能运行在一个模式下。

  * NSDefaultRunloopMode：默认模式，用于普通UI、timer、网络事件
  * UITrackingRunloopMode：滑动模式，scrollview滚动，手势跟踪，优先处理滑动事件。
  * NSRunloopCommonModes：不是一个mode，是标记集合。（集合标签）本质：Common = Default + Tracking。加入common mode的任务，会同时在多个mode下执行。

  Mode本质：每个mode都有对应的source、timer和observer。Runloop切换Mode时，只处理当前Mode的内容。

滑动时timer停止的原因：

滑动时mode会切换到UItrackingRunloopMode，而timer默认是在NSDefaultRunllopMode运行。解决方案将timer标记为common mode，也就是Timer 被加入到 “CommonModes 标记对应的所有 Mode”。

**Runloop工作流程**： 监听 -> 执行 -> 休眠 -> 唤醒 

```swift
1. 进入 RunLoop
2. 通知 observers（即将处理）
3. 处理 timers
4. 处理 sources
5. 如果没有任务 → 休眠
6. 被唤醒（事件到来）
7. 继续处理
```

**Runloop与线程的关系**

每个线程都有Runloop，主线程默认开启，子线程默认不开启。



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

不占用内存，在使用时通过计算获取。其本质是一个函数。

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
在super.init()前，不能使用self
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

带associatedtype的procotol不是完整的类型。不能直接作为变量类型或者函数参数使用。因为编译器无法确定具体的关联类型（类型不完整、不确定）。

```swift
protocol Container {
    associatedtype Item
    
    func add(_ item: Item)
    func getItem(at index: Int) -> Item
}
```

必须借助泛型（Generics）、some（不透明类型）、any（存在类型/类型擦除）来解决。

| **方式** | **用途**         |
| -------- | ---------------- |
| 泛型     | 最常见           |
| some     | 返回某种具体类型 |
| any      | 类型擦除         |

* 解决方案1：使用泛型，泛型约束

```swift
func process<C: Container>(_ container: C) {
    let item = container.getItem(at: 0)
    print(item)
}
// C.Item类型是确定的
// 如果写var container: Container会报错
// Protocol 'Container' can only be used as a generic constraint
// because it has Self or associated type requirements
// 因为 协议不知道 Item 是什么类型。

func test<T: Container>(c: T) {
    // T.Item 是确定的
}

struct Box<T>: Container {
    typealias Item = T
    func add(_ item: T) {}
}
// 优点：类型安全（编译期确定），无性能损耗（静态派发）。
test(Box<Int>())
test(Box<String>())
```

* 使用Any

```swift
var container: any Container
// 注意any Container只是类型擦除后的协议类型，能调用的方法受限。
// any Container 只能调用不依赖 Item 的方法
```

* 解决方案3：使用some（不透明类型）

```swift
func makeContainer() -> some Container {
    IntContainer()
}
// some表示：返回某一种具体类型，但不告诉你是哪种
// 编译器知道具体类型，但调用方不知道

// 使用场景
var body: some View
```

在Swift中，带有associatedtype的协议不能直接作为变量或者参数类型使用，因为它本身不是一个完整类型，编译器无法确定关联类型。

为了解决这种问题，通常由三种方式：

* 使用泛型，在编译器确定类型，是最推荐的方式。
* 使用some，表示返回某个具体但隐藏的类型，常用于返回值。
* 使用any，表示可以存储任意符合协议的类型，但需要配合类型擦除才能使用其方法。

本质区别在于：

* 泛型和some都是编译器就确定类型
* any是运行时动态分发，存在性能损耗。

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



### 类型擦除（Type Erasure）理解

```swift
// 问题
protocol Container {
    associatedtype Item
    func add(_ item: Item)
}

var c: any Container
c.add(1) // ❌ 报错
// 因为c的真实类型不确定->Item也不确定->无法调用方法
```

**类型擦除到底是什么**

本质：把泛型信息、关联类型信息隐藏掉，用一个统一的壳包装起来，对外暴露固定接口。

**核心思想**

类型擦除做了三件事情：

* 捕获具体类型

```swift
init<C: Container>(_ container: C)
// 把具体类型抓进来
```

* 保存行为（函数闭包）

```swift
let _add: (T) -> Void
// 不在关心类型，只关系能不能做这个操作
```

* 对外统一接口

```swift
func add(_ item: T) {
    _add(item)
}
// 所有类型统一调用方式
```

**完整实现**

* 定义类型擦除结构

```swift
class AnyContainer<T>: Container {
    private let _add: (T) -> Void

    init<C: Container>(_ container: C) where C.Item == T {
        _add = container.add
    }

    func add(_ item: T) {
        _add(item)
    }
}
```

* 具体实现

```swift
struct IntBox: Container {
    func add(_ item: Int) {
        print("IntBox:", item)
    }
}

struct StringBox: Container {
    func add(_ item: String) {
        print("StringBox:", item)
    }
}
```

* 统一使用

```swift
let intBox = IntBox()
let anyBox = AnyContainer(intBox)

anyBox.add(10) // ✅
```

类型擦除的本质：把协议里面的associatedtype转换为泛型参数。

| **方式**      | **是否保留类型信息** | **是否能调用方法** |
| ------------- | -------------------- | ------------------ |
| any Container | ❌ 不知道             | ❌                  |
| 泛型          | ✅ 知道               | ✅                  |
| 类型擦除      | ❌ 外部不知道         | ✅ 内部知道         |

```swift
类型擦除是将带有 associatedtype 的协议包装成一个具体类型，通过泛型固定住关联类型，并通过 closure 保存原始对象的方法实现，从而实现统一调用接口。

它解决的问题是：协议本身因为关联类型不确定无法作为变量使用，而类型擦除通过“把关联类型转成泛型参数”让类型在编译期确定，同时对外隐藏具体实现。
```



### 静态派发和动态派发的理解

静态派发（static dispatch）：编译时就知道调用哪个方法

动态派发（dynamic dispatch）：运行时才决定调用哪个方法

以打电话形象的理解就是：静态派发知道具体要打电话给谁，直接拨号，而动态派发你只知道公司，需要客服给你转接到具体的某个人。

**例子**

```swift
// 静态派发
struct Dog {
    func speak() {
        print("Woof")
    }
}

let d = Dog()
d.speak()
// 编译时就已经确定类型，Dog.speak(), 运行时不用再查找

// 动态派发
protocol Animal {
    func speak()
}

struct Dog: Animal {
    func speak() {
        print("Woof")
    }
}

struct Cat: Animal {
    func speak() {
        print("Meow")
    }
}

let a: Animal = Dog()
a.speak()
// 编译器只知道a是Animal，但不知道是Dog还是Cat，必须在实际运行时查找实际类型-> 调用对应的方法。
```

**Swift中哪些是静态派发**

* struct/enum

```swift
struct A {
    func test() {}
}
```

* 泛型

```swift
func test<T: Animal>(_ a: T) {
    a.speak()
}
// 编译期展开,所以是静态派发
test(Dog)
test(Cat)
```

* some(不透明类型)

```swift
func getAnimal() -> some Animal
// 实际是类型是固定的，编译器知道-> 静态派发
```

**动态派发**

* any（存在类型）

```swift
let a: any Animal = Dog()
a.speak() 
// 运行时确定
```

* class + override

```swift
class Animal {
    func speak() {
        print("Animal")
    }
}

class Dog: Animal {
    override func speak() {
        print("Dog")
    }
}

let a: Animal = Dog()
a.speak() // Dog

// runtime决定
```

* @objc/dynamic

```swift
class A {
    @objc dynamic func test() {}
}
// 强制走runtime
```

* 类型擦除

```swift
class AnyContainer<T> {
    private let _add: (T) -> Void
}
// _add是closure，相当于函数指针，运行时调用
```

* AnyView

```swift
AnyView(Text("Hello"))
// 内部存closure，存render函数，渲染时再调用
```

**优缺点**

静态派发更快（没有查找过程），编译器可以优化（inline、清除调用），类型安全更强。

缺点不灵活，类型必须固定。



**动态派发使用场景**

* 多态

```swift
let animals: [any Animal]
```

* 插件系统

```swift
let plugins: [any Plugin]
```

* SwiftUI

```swift
AnyView
// UI类型不固定
```

在Swift中，静态派发是编译器就确定调用目标，常见于struct，泛型和some类型，性能更好。而动态派发是在运行时根据实际类型查找方法实现，常见于any、class和类型擦除。像AnyView和AnyPublisher本质上就是通过类型擦除引入动态派发，以换取更强的抽象能力和灵活性。

**class是否动态派发**

class默认是动态派发（vtable)，但是某些情况下（final，private，编译器可推断不可重写），可以被优化为静态派发。是否发生动态派发，不完全取决于有没有继承，而是取决于方法是否可被override。

一句话：是否动态派发，看这个方法在编译期是否确定不会被override。

```swift
// 情况1: 普通class，即使没有写子类，编译器也必须假设，所以必须使用动态派发
class Animal {
    func speak() {
        print("Animal")
    }
}
```

```swift
// 静态派发情况：
// final修饰
final class Animal {
    func speak() {
        print("Animal")
    }
}
// 不允许继承，编译器确定不会被override，可以静态派发

class Animal {
    final func speak() {
        print("Animal")
    }
}
// 即使class可被继承，但是这个方法不能被override，静态派发。

// private/fileprivate 修饰
class Animal {
    private func speak() {}
}
// 外部无法override，编译器确定不会被重写，静态派发。

// class调用流程：动态派发
对象 → isa 指针 → vtable → 方法地址 → 调用
// final优化后
直接函数地址调用（类似 struct）
```

| **情况**        | **是否动态派发** |
| --------------- | ---------------- |
| 普通 class 方法 | ✅ 动态           |
| override 方法   | ✅ 动态           |
| final class     | ❌ 静态           |
| final func      | ❌ 静态           |
| private func    | ❌ 静态           |
| struct 方法     | ❌ 静态           |

> Swift 中 class 默认使用动态派发，因为它支持继承和方法重写，编译器必须在运行时通过 vtable 查找具体实现。即使当前没有子类，编译器也必须假设未来可能被继承。只有在编译器可以确定方法不会被 override 的情况下，比如使用 final、private，或者 final class，才可以优化为静态派发。



### struct和class方法调用区别理解

**直观区别**

struct调用方法操作的是拷贝的值，而class调用方法操作的是同一块内存。

```swift
// struct值类型
struct Counter {
    var value = 0
    
    mutating func increment() {
        value += 1
    }
}

var c1 = Counter()
var c2 = c1

c2.increment()

print(c1.value) // 0
print(c2.value) // 1

// class引用类型
class Counter {
    var value = 0
    
    func increment() {
        value += 1
    }
}

let c1 = Counter()
let c2 = c1

c2.increment()

print(c1.value) // 1
print(c2.value) // 1
```

**方法调用本身的区别**

struct静态派发，class动态派发。

struct没有继承，方法不被重写，而class有继承和多态，方法可能被重写，必须运行时决定。

**内存层面的区别**

struct：栈内存、值拷贝，直接操作的当前值。

class：堆内存+引用，通过引用找到对象 -> 再调用方法

**struct和class使用场景**

struct：

* 数据模型（model）
* 不需要共享状态
* 希望高性能
* SwiftUI View（必须是struct）

class：

* 需要共享状态
* 需要继承、多态
* UIKit等

> struct 和 class 在方法调用上的核心区别在于派发机制和内存语义。struct 是值类型，方法调用采用静态派发，编译期就能确定调用目标，性能更好；而 class 是引用类型，支持继承和方法重写，因此方法调用通常采用动态派发，需要在运行时通过 vtable 查找具体实现。此外，struct 的方法操作的是值拷贝，而 class 操作的是共享的引用对象。



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



### AppDelegate Vs SceneDelegate理解

`AppDelegate`管理应用级生命周期，`SceneDelegate`管理界面级生命周期（窗口）。

| **角色**      | **管什么**             | **生命周期粒度** |
| ------------- | ---------------------- | ---------------- |
| AppDelegate   | 整个 App               | 全局             |
| SceneDelegate | 一个 UI 场景（window） | 局部             |

iOS13引入`SceneDelegate`多窗口，一个App可以有多个界面实例。每个界面有独立的生命周期。

**生命周期**

* AppDelegate生命周期

1. 启动阶段

```swift
func application(
  _ application: UIApplication,
  didFinishLaunchingWithOptions launchOptions: ...
) -> Bool
// app启动完成（最重要）
```

2. 状态变化

```swift
applicationWillResignActive
applicationDidEnterBackground
applicationWillEnterForeground
applicationDidBecomeActive
```

3. 终止

```swift
applicationWillTerminate
```

* SceneDelegate生命周期

1. 创建Scene

```swift
scene(_:willConnectTo:options:)
// 初始化UIWindow（核心入口）
```

2. 前后台切换

```swift
sceneWillEnterForeground
sceneDidBecomeActive
sceneWillResignActive
sceneDidEnterBackground
```

3. 销毁

```swift
sceneDidDisconnect
```

* 调用顺序

1. app冷启动流程

```swift
1. AppDelegate.application(didFinishLaunching)
2. AppDelegate.configurationForConnectingSceneSession
3. SceneDelegate.scene(willConnectTo)
4. SceneDelegate.sceneWillEnterForeground
5. SceneDelegate.sceneDidBecomeActive
```

2. 进入后台

```swift
1. SceneDelegate.sceneWillResignActive
2. SceneDelegate.sceneDidEnterBackground
3. AppDelegate.applicationDidEnterBackground
```

3. 回到前台

```swift
1. AppDelegate.applicationWillEnterForeground
2. SceneDelegate.sceneWillEnterForeground
3. SceneDelegate.sceneDidBecomeActive
```

4. 冷启动完整运行流程

```swift
App 启动
 ↓
AppDelegate.didFinishLaunching
 ↓
创建 Scene Session
 ↓
SceneDelegate.willConnectTo
 ↓
创建 UIWindow
 ↓
设置 rootViewController
 ↓
window.makeKeyAndVisible()
 ↓
sceneDidBecomeActive
```

* 职责划分

1. AppDelegate负责全局、一次性与UI无关。

```swift
•	SDK 初始化（Firebase、Analytics）
•	Push 注册
•	App 配置
•	Deep Link（全局处理）
•	App 生命周期监听
```

2. SceneDelegate: UI和窗口相关

```swift
•	创建 UIWindow
•	设置 rootViewController
•	页面恢复（state restoration）
•	Scene 级 Deep Link 处理
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
  // 这个是UI更新的关键入口
    var objectWillChange: ObservableObjectPublisher { get }
}
```

这个 objectWillChange 是一个 **Combine 发布者（Publisher）**，

当对象中任何被 @Published 修饰的属性发生变化时，它就会发出通知。



**@StateObject**

用来持有一个`ObservableObject`，生命周期是View管理（只初始化一次），会订阅对象的变化。



**@Published是什么**

用在`ObservableObject`理的属性上，@Published是一个属性包裹器（property wrapper），用来声明当这个属性发生变化时，要通知所有订阅者（包括SwiftUI视图）。

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



**完整更新流程**

1. View初始化

```swift
@StateObject var vm = ViewModel()
```

* 创建viewModel实例
* 订阅vm.objectWillChange
* 把这个订阅绑定到View的刷新机制

```swift
vm.objectWillChange
   .sink { _ in
       triggerViewUpdate()
   }
```

2. @Published属性被修改

```swift
vm.count += 1
```

@Published背后做了什么，其等价于（简化版）

```swift
var count: Int {
    willSet {
        objectWillChange.send()
    }
}
```

3. 发送变更通知

```swift
objectWillChange.send()
```

这个Publisher会通知所有订阅者（也就是View）

4. SwiftUI收到通知

```swift
收到 objectWillChange
    ↓
标记 View 为 dirty（需要刷新）
    ↓
进入下一轮 runloop
    ↓
重新计算 body
```

5. 重新计算body

```swift
旧：Text("0")
新：Text("1")
```

只更新Text，不重建整个View。

```swift
1.	@StateObject 持有 ObservableObject，并订阅 objectWillChange
2.	@Published 在属性变化时自动调用 objectWillChange.send()
3.	SwiftUI 收到通知后：
    •	标记 View 需要刷新
    •	在下一轮 runloop 重新执行 body
4.	SwiftUI 对新旧 View 做 diff
5.	只更新变化的 UI 部分
```

**@Published 负责发通知，@StateObject 负责订阅通知，SwiftUI 收到通知后重新计算 body 并 diff 更新 UI。**



### SwiftUI状态系统@State，@StateObject以及@Environment理解

这些属性包裹器本质在做两件事：

* 存储状态（state storage，不在View struct本身）
* 建立依赖关系（依赖变化 -> 触发body重新计算）

View是struct（值类型），状态不能直接存储在view里。

**@State：View内部状态**

```swift
@State var count = 0
// 编译后大致编程
private var _count: State<Int>
var count: Int {
  get { _count.wrappedValue }
  nonmutating set { _count.wrappedValue = newValue }
}
// _count真正存储数据（存在SwiftUI的State Storage中）
// count 只是访问入口

// 底层机制
State<T>
   ↓
SwiftUI 内部存储（类似一个全局状态表）
   ↓
通过 View identity 找到对应状态

struct CounterView: View {
    @State var count = 0

    var body: some View {
        VStack {
            Text("\(count)")
            Button("Add") {
                count += 1
            }
        }
    }
}

// 触发更新流程
count 修改
   ↓
State storage 更新
   ↓
标记 View dirty
   ↓
重新计算 body
```

通常用于view内部的简单状态。



**@StateObject：持有引用类型（ViewModel）**

```swift
@StateObject var vm = ViewModel()
// 编译后类似
private var _vm: StateObject<ViewModel>
private var vm {
  _vm.wrappedValue
}

// StateObject内部做了两件事
持有ViewModel（保证生命周期）
订阅ObjectWillChange

// 内部结构
StateObject
   ↓
ObservedObjectBox
   ↓
订阅 objectWillChange
   ↓
触发 View 刷新

// 示例
class CounterVM: ObservableObject {
    @Published var count = 0
}

struct ContentView: View {
    @StateObject var vm = CounterVM()

    var body: some View {
        VStack {
            Text("\(vm.count)")
            Button("Add") {
                vm.count += 1
            }
        }
    }
}
```



**@Environment：从环境中读取值**

```swift
@Environment(.\colorScheme) var scheme

// 编译后类似
private var _scheme: Enviroment<ColorScheme>
private var scheme {
  _scheme.wrappedValue
}

// 本质
EnvironmentKey + EnvironmentValues(类似字典)

// SwiftUI环境系统
EnvironmentValues（一个 Key-Value 容器）
        ↓
从父 View 逐层传递
        ↓
子 View 读取

// 使用示例
struct ThemeView: View {
    @Environment(\.colorScheme) var scheme

    var body: some View {
        Text(scheme == .dark ? "Dark" : "Light")
    }
}

// 自定义Environment
struct MyKey: EnvironmentKey {
    static let defaultValue: String = "Default"
}

extension EnvironmentValues {
    var myValue: String {
        get { self[MyKey.self] }
        set { self[MyKey.self] = newValue }
    }
}

// 使用
// 注入
MyView()
    .environment(\.myValue, "Hello")

// 读取
@Environment(\.myValue) var value

// 更新机制
Environment 改变
   ↓
所有依赖该 key 的 View
   ↓
重新计算 body

// 使用场景
全局/跨层级共享数据
•	主题
•	语言
•	用户信息（轻量）
```



### Property Wrapper属性包裹器理解

```swift
@proprtyWrapper
struct MyWrapper {
  var wrappedValue: Int
}

// 使用
@MyWrapper var value = 10

// 编译器会帮你做什么，其会展开为
private var _value = MyWrapper(wrappedValue: 10)
var value: Int {
  get: { _value.wrappedValue }
  set: { _value.wrappedValue = newValue }
}
// @Wrapper var x
// _x -> 真正存储（Wrapper类型）
// x -> wrappedValue(你平时使用的值)

// $是怎么来的
// 如果Wrapper多写一个属性
@propertyWrapper
struct MyWrapper {
  var wrappedValue: Int
  var projectedValue: String {
    return "current value: \(wrappedValue)"
  }
}

// 编译器会再生成一个
var $value: String {
  _value.projectedValue
}
// $是语法糖，本质是
wrapper.projectedValue
```

**统一模型**

任何property wrapper都是

```swift
@Wrapper var x
// 展开为
_x -> Wrapper类型（真正存储）
x -> wrappedValue(值)
$x -> projectedValue(扩展能力)
```

由于每个wrapper都可以自定义projectedValue类型，所有$每种都不一样。



**SwiftUI常见Wrapper**

* @State

```swift
@State var count = 0
// 编译后
_count: State<Int>
count: Int
$count: Binding<Int>

// SwiftUI的设计目标是允许子View修改父View状态。所以$count是可读写的引用。
// Binding本质
struct Binding<Value> {
  let get: () -> Value
  let set: (Value) -> Void
}

$count其实是
Binding {
  get: { count }
  set: { count = $0 }
}
```

> $vm 不是ViewModel
>
> 而是
>
> ```swift
> ObservedObject<UserVM>.Wrapper
> ```
>
> SwiftUI 内部专门定义的一个 Wrapper 类型。
>
> 类似
>
> ```swift
> struct Wrapper {
>     subscript<Subject>(
>         dynamicMember keyPath: ReferenceWritableKeyPath<ObjectType, Subject>
>     ) -> Binding<Subject>
> }
> ```
>
> 核心作用：**自动把对象属性转化为Binding**

* @Binding

```swift
@Binding var count: Int
// 本质只是接收一个Binding，所以$count还是Binding（继续往下传）
```

* @StateObject

```swift
@StateObject var vm = ViewModel()
// 编译后
_vm: StateObject<ViewModel>
vm: ViewModel
$vm: ObservedObject<ViewModel>.Wrapper

// vm是引用类型，SwiftUI不需要Binding整个对象，而是Binding到其内部的@Published对象，所以$vm.name才是Binding
TextField("name", text: $vm.name)

vm.name   → 值
vm.$name  → Publisher（数据流）
但SwiftUI需要的是Binding<String>，所以需要把 vm.name → 转换成 Binding

这里的$vm: ObservedObject<ViewModel>.Wrapper具体做了什么？
当你写$vm.name
1.访问$vm
$vm -> Wrapper
2.访问name
$vm.name
这里不是普通属性访问
3.动态生成Binding
Binding(
    get: { vm.name },
    set: { vm.name = $0 }
)
4.最终结果
$vm.name → Binding<String>

完整数据流：
TextField 输入
   ↓
Binding.set
   ↓
vm.name = 新值
   ↓
@Published 触发
   ↓
objectWillChange
   ↓
View 刷新
```

* @Published

```swift
@Published var count = 0
// 编译后
count: Int
$count: Published<Int>.Publisher // $count是数据流

vm.$count
    .sink { print($0) }
```

| **Wrapper**     | $xxx **类型** | **本质作用**   |
| --------------- | ------------- | -------------- |
| @State          | Binding       | 双向数据       |
| @Binding        | Binding       | 传递引用       |
| @StateObject    | Wrapper       | 访问子 Binding |
| @ObservedObject | Wrapper       | 提供 Binding   |
| @Published      | Publisher     | 数据流         |

```swift
$ 放在谁前面，就取谁的 projectedValue。
wrappedValue（x） → 当前值
projectedValue（$x） → “额外能力”
```







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



### iOS架构理解

**核心目标**

iOS架构设计的核心目标：解耦、可维护、可测试、可扩展、可复用。

* 解耦（Decoupling）：模块之间互不依赖。

例如：View不直接依赖网络层，Controller不处理业务逻辑。

* 可维护（Maintainability）：修改一个模块，不影响其他模块，代码容易理解。
* 可测试（Testability）：可独立测试ViewModel、UserCase和Service
* 可扩展（Scalability）：新需求不需要大改旧代码。
* 可复用（Resusability）：组件可复用，网络层和业务逻辑。

好的架构不是为了复杂，而是为了让复杂系统变得简单可控。

**常见架构**

* MVC

```swift
// 各部分职责
Model（数据）：用户数据、接口数据
View（界面）：UILabel、UITextfield、UIButton
Controller（控制器）：处理所有业务逻辑

// 实力流程
View（点击按钮）
   ↓
Controller（处理点击）
   ↓
调用 API（请求数据）
   ↓
更新 Model
   ↓
Controller 更新 View

// 优缺点
优点：简单、上手快
缺点：Controller编程上帝类，难维护、难测试
```

* MVP

```swift
MVP = Model + View + Presenter
核心思想，Controller不写逻辑，用Presenter
View：只负责显示
Presenter：处理所有逻辑
Model：数据

// 流程
View（点击按钮）
   ↓
Presenter（处理逻辑）
   ↓
调用 Model / API
   ↓
Presenter 更新 View

// 示例
protocol LoginView {
    func showError()
    func goToHome()
}

class LoginPresenter {
    weak var view: LoginView?

    func login(username: String, password: String) {
        API.login(username, password) { result in
            if result.success {
                self.view?.goToHome()
            } else {
                self.view?.showError()
            }
        }
    }
}

class LoginVC: UIViewController, LoginView {
    let presenter = LoginPresenter()

    func loginTapped() {
        presenter.login(username: "a", password: "b")
    }
}

// 优缺点：
优点：逻辑从VC分离，更好测试
缺点： View 和 Presenter耦合（通过接口），Presenter可能变很大
```

* MVVM

```swift
MVVM = Model + View + ViewModel
核心思想：数据驱动UI（自动绑定）

// 各部分职责
View: UI
ViewModel: 数据 + 逻辑
Model： 数据
ViewModel不直接操作View，而是绑定数据

// 流程
View（点击）
   ↓
ViewModel（处理逻辑）
   ↓
更新数据（state）
   ↓
View 自动更新（绑定）

// 示例
class LoginViewModel: ObservableObject {
    @Published var isLoginSuccess = false

    func login() {
        API.login { result in
            self.isLoginSuccess = result.success
        }
    }
}

// View
@StateObject var vm = LoginViewModel()

Button("Login") {
    vm.login()
}

if vm.isLoginSuccess {
    Text("Success")
}

// 优缺点：
解耦很好，支持响应式（RxSwift、Combine），已测试。
学习成本高，可能过度设计。
```

* VIPER

| VIPER      | 类比     | 作用             |
| ---------- | -------- | ---------------- |
| View       | 用户界面 | 显示UI、接收点击 |
| Presenter  | 客服     | 控制流程         |
| Interactor | 厨房     | 处理业务逻辑     |
| Entity     | 菜       | 数据模型         |
| Router     | 配送     | 页面跳转         |

```swift
VIPER = View + Insteractor + Presenter + Entiry + Router
核心思想：极致解耦（每个职责拆分到最细）。
把一个页面拆成 5 个“只做一件事”的角色

// 比喻
你点外卖：
1.	你（View） → 下单
2.	客服（Presenter） → 帮你转达
3.	厨房（Interactor） → 做饭
4.	菜品（Entity） → 数据
5.	配送员（Router） → 把你送到“下一个页面”

// 完整流程
View
 ↓
Presenter
 ↓
Interactor
 ↓
API

API 返回
 ↑
Interactor
 ↑
Presenter
 ↓         ↓
View     Router

// 示例
// View(Controller)
class LoginViewController: UIViewController {

    var presenter: LoginPresenter!

    @IBAction func loginTapped() {
        presenter.login(username: "a", password: "b")
    }

    func showError() {
        print("error")
    }
}

// Presenter: 核心调度
class LoginPresenter {

    var view: LoginView?
    var interactor: LoginInteractor?
    var router: LoginRouter?

    func login(username: String, password: String) {
        interactor?.login(username, password)
    }

    func loginSuccess() {
        view?.showSuccess()
        router?.goToHome()
    }

    func loginFailed() {
        view?.showError()
    }
}

// Interactor: 业务逻辑
class LoginInteractor {

    var presenter: LoginPresenter?

    func login(_ username: String, _ password: String) {
        API.login(username, password) { success in
            if success {
                self.presenter?.loginSuccess()
            } else {
                self.presenter?.loginFailed()
            }
        }
    }
}

// Entity
struct User {
    let name: String
}

// Router: 跳转
class LoginRouter {

    func goToHome() {
        // push / present
    }
}

// 优缺点
文件爆炸，开发成本高，学习成本高。
适合大型项目、多人协助、复杂业务。不适合小项目、快速开发。
```



### 策略模式理解

**什么是策略模式**（是什么）

策略模式是将一组可互换的算法（策略）封装起来，并且可以再运行时自由切换。

* 定义一系列算法（策略）
* 把他们一个个封装起来
* 让他们可以相互替换
* 客户端无需担心具体替换

**核心思想（为什么）**

策略思想要解决的问题是：避免大量if/else、switch分支。

策略模式的思路：把算法变化抽离出来。变成

* 每种支付方式 = 一个策略
* 外部只负责选择策略，不负责实现

```swift
func pay(type: String) {
    if type == "alipay" {
        // 支付宝逻辑
    } else if type == "wechat" {
        // 微信逻辑
    } else if type == "apple" {
        // Apple Pay
    }
}
// 问题
•	代码膨胀
•	难扩展（新增支付方式要改原代码）
•	不符合开闭原则（OCP）
```

**结构设计（怎么用）**

策略模式一般包含3个角色：

* Strategy（策略协议）

```swift
protocol PaymentStrategy {
    func pay(amount: Double)
}
```

* Concrete Strategy(具体策略)

```swift
class AlipayStrategy: PaymentStrategy {
    func pay(amount: Double) {
        print("使用支付宝支付 \(amount)")
    }
}

class WechatStrategy: PaymentStrategy {
    func pay(amount: Double) {
        print("使用微信支付 \(amount)")
    }
}
```

* Context(上下文)

```swift
class PaymentContext {
    private var strategy: PaymentStrategy
    
    init(strategy: PaymentStrategy) {
        self.strategy = strategy
    }
    
    func setStrategy(_ strategy: PaymentStrategy) {
        self.strategy = strategy
    }
    
    func executePay(amount: Double) {
        strategy.pay(amount: amount)
    }
}
```

* 使用

```swift
let context = PaymentContext(strategy: AlipayStrategy())
context.executePay(amount: 100)

context.setStrategy(WechatStrategy())
context.executePay(amount: 200)
```

**使用场景**

* 支付方式

1. 支付宝 / 微信 / Apple Pay

2. 不同策略 = 不同支付实现

* 网络请求策略

1. 缓存策略（Cache/Netowrk First）
2. 重试策略
3. 限流策略

```swift
// 场景：缓存 + 网络策略
比如：
	•	只走缓存
	•	只走网络
	•	先缓存再网络（最常见）
	•	网络失败回退缓存

// step1: 变化点 -> 请求数据的获取方式

// step2: 定义策略
protocol RequestStrategy {
    func request(
        cache: () -> Data?,
        network: () async throws -> Data
    ) async throws -> Data
}

// step3: 具体策略
// Cache First
class CacheFirstStrategy: RequestStrategy {
    func request(
        cache: () -> Data?,
        network: () async throws -> Data
    ) async throws -> Data {
        
        if let data = cache() {
            return data
        }
        
        return try await network()
    }
}
// Network First
class NetworkFirstStrategy: RequestStrategy {
    func request(
        cache: () -> Data?,
        network: () async throws -> Data
    ) async throws -> Data {
        do {
            return try await network()
        } catch {
            if let data = cache() {
                return data
            }
            throw error
        }
    }
}

// step4: Context
class APIClient {
    var strategy: RequestStrategy
    
    init(strategy: RequestStrategy) {
        self.strategy = strategy
    }
}

•	类似 URLCache / Alamofire RequestInterceptor
•	可以动态切换（弱网 / 强网环境）
```

* 图片加载策略（类似SDWebImage）

1. 内存缓存
2. 磁盘缓存
3. 网络加载

* 排序过滤逻辑

```swift
// 场景
•	商品排序（价格 / 销量 / 时间）
•	数据过滤（有效 / 无效 / 权限）

// step1: 变化点-> 排序规则、过滤规则

// step2: 策略协议
protocol SortStrategy {
    func sort(_ items: [Item]) -> [Item]
}

// step3: 实现
struct Item {
    let price: Double
    let sales: Int
}
// 按价格排序
class PriceSortStrategy: SortStrategy {
    func sort(_ items: [Item]) -> [Item] {
        items.sorted { $0.price < $1.price }
    }
}
// 按销量排序
class SalesSortStrategy: SortStrategy {
    func sort(_ items: [Item]) -> [Item] {
        items.sorted { $0.sales > $1.sales }
    }
}

// step3: Context
class SortContext {
    var strategy: SortStrategy
    
    func execute(items: [Item]) -> [Item] {
        strategy.sort(items)
    }
}

•	比 if-else 更易扩展
•	在电商 / 列表页非常常见
```

* 动画策略（UIKit/SwiftUI）

1. 不同动画曲线
2. 不同转场方式

```swift
// 场景：
•	不同动画效果
•	不同转场方式

// step1:变化点 -> 动画实现方式

// step2: 策略协议
protocol AnimationStrategy {
    func animate(view: UIView)
}

// step3: 实现
// Fade 动画
class FadeAnimation: AnimationStrategy {
    func animate(view: UIView) {
        view.alpha = 0
        UIView.animate(withDuration: 0.3) {
            view.alpha = 1
        }
    }
}
// Scale动画
class ScaleAnimation: AnimationStrategy {
    func animate(view: UIView) {
        view.transform = CGAffineTransform(scaleX: 0.5, y: 0.5)
        UIView.animate(withDuration: 0.3) {
            view.transform = .identity
        }
    }
}
```



* 表单验证策略

```swift
// 场景
•	手机号
•	邮箱
•	密码强度

// step1: 变化点，验证规则

// step2: 策略协议
protocol ValidationStrategy {
    func validate(_ text: String) -> Bool
}

// step3: 实现
// 手机号
class PhoneValidation: ValidationStrategy {
    func validate(_ text: String) -> Bool {
        return text.count == 11
    }
}

// 邮箱
class EmailValidation: ValidationStrategy {
    func validate(_ text: String) -> Bool {
        return text.contains("@")
    }
}

// step4: Context
class Validator {
    private var strategies: [ValidationStrategy] = []
    
    func add(_ strategy: ValidationStrategy) {
        strategies.append(strategy)
    }
    
    func validate(_ text: String) -> Bool {
        return strategies.allSatisfy { $0.validate(text) }
    }
}
// 支持组合策略：必填 + 格式 + 长度
validator.add(RequiredValidation())
validator.add(EmailValidation())
在 React Native 项目中，我把表单验证抽象为策略模式，使验证逻辑可复用，并支持组合验证规则，减少了大量重复代码。
```

**优缺点**

优点：

* 符合开闭合原则（OCP）：新增策略不用改代码
* 消除if-else：结构更清晰
* 提高扩展性：可以动态切换策略
* 解耦：使用者不关心具体实现

缺点：

* 类数量变多
* 需要理解成本
* 客户端需要策略的存在

> 策略模式的核心是“分离变化”。在实际开发中，我会优先识别哪些逻辑会变化，比如请求策略、排序规则、动画效果、验证规则，然后通过 protocol 抽象行为，将不同实现封装为独立策略。
>
> 在 Swift 中，我也会结合 protocol + 泛型 + 闭包优化策略模式，使其更加轻量化，而不是一味增加类数量。



























































