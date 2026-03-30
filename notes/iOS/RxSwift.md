# RxSwift

## 核心思维

所有的数据变化都是一个“事件流”（stream），我们只需要监听（observe）它，并定义当他变化时该做什么。

* 核心模型

Observable->Observer->Subscription

| **角色**         | **含义**                   | **类比**           |
| ---------------- | -------------------------- | ------------------ |
| **Observable**   | 可被观察的对象，产生数据流 | 电台（发射信号）   |
| **Observer**     | 观察者，接收并响应事件     | 收音机（接收信号） |
| **Subscription** | 订阅关系，连接两者         | 电波连接           |

* 核心思维模式

1. 声明式

```swift
button.rx.tap
    .subscribe(onNext: { print("Button clicked") })
    .disposed(by: disposeBag)
```

2. 数据流动（Data Flow）而非命令流动

数据通过管道（stream）从源头流向观察者。

```swift
textField.rx.text
    .orEmpty
    .filter { $0.count > 3 }
    .map { $0.uppercased() }
    .bind(to: label.rx.text)
    .disposed(by: disposeBag)
```

3. 纯函数式思维

RxSwift 借鉴了 **函数式编程** 思维：

用 .map, .filter, .flatMap, .reduce 等操作符来组合事件流，而不是写 if/else 或回调。

操作符是 RxSwift 的核心，它让你能像操作数组一样去操作“事件流”。

4. 异步处理的统一化

在传统代码中：

- 网络请求 → completion handler
- 按钮点击 → target/action
- 输入监听 → delegate
- 通知 → NotificationCenter

这些事件源完全不同。

但在 RxSwift 中：全部都变成了 Observable。



## RxSwift核心构件

### Observable和Observer（可观察序列和观察者）

1. 核心概念

- Observable<T>：代表“随时间发出 T 类型事件的流”。
- Observer：订阅者，接收事件。事件有三类：.next(value)、.error(error)、.completed()。

2. 事件类型

- .next(value)：正常值事件（可多次）。
- .error(error)：错误事件，序列终止且不会再发 .completed。
- .completed()：正常完成，序列终止。

3. 创建方式

```swift
Observable.just(1)                  // 只发出一次 1，然后 completed
Observable.of(1,2,3)               // 依次发出 1,2,3，然后 completed
Observable.from([1,2,3])           // 从集合创建
Observable.create { observer in    // 自定义产生事件
    observer.onNext(1)
    observer.onCompleted()
    return Disposables.create()
}
```

4. 冷序列和热序列

- **冷（Cold）Observable**：每个订阅者会独立触发生产过程（例如 Observable.create 中的逻辑、网络请求）。订阅时“开始”。
  - 例：Observable.from([1,2,3])：每次订阅都会重放 1,2,3。
- **热（Hot）Observable**：事件在源头持续产生，订阅者只是“接收当前或之后的事件”。（例如 PublishSubject、UI 控件的 rx.tap）。
  - 例：let subject = PublishSubject<Int>()，subject.onNext(1) 在订阅前会丢失这个 1。

**实践建议**：大多数 API 默认冷序列（网络请求），当你需要多个订阅共享同一次执行结果时，考虑 share() 或 publish().refCount()。



### Disposable和DisposeBag（生命周期管理）

1. 核心概念

- Disposable：取消订阅的句柄（资源清理）。
- DisposeBag：用于自动管理 Disposable 的容器 —— 当 DisposeBag 被释放时，里面的订阅都会被 disposed。

```swift
let bag = DisposeBag()

Observable.of(1,2,3)
    .subscribe(onNext: { print($0) })
    .disposed(by: bag)
```

2. 手动释放

```swift
let disposable = Observable.interval(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe(onNext: { print("tick") })
// later
disposable.dispose()
```

3. 注意事项

- 在 UIViewController/View 中通常把 DisposeBag 作为生命周期属性；在 UITableViewCell 中需要在 prepareForReuse() 里重置 disposeBag，否则会导致重复订阅 / 内存泄漏。

```swift
class MyCell: UITableViewCell {
    var bag = DisposeBag()
    override func prepareForReuse() {
        super.prepareForReuse()
        bag = DisposeBag()
    }
}
```

> **UITableViewCell 的复用 &** prepareForReuse() 的精确时机（时间线）
>
> 假设 TableView 已经运行一段时间，用户在滚动，系统会重用 cell 以节省内存。
>
> 步骤（按时间顺序）：
>
> 1. TableView 需要显示 row A，于是调用 dequeueReusableCell(withIdentifier:for:)。
>    - 如果没有可复用 cell，系统会新建一个（init -> awakeFromNib -> layout…）。
>    - **若有可复用 cell（之前显示过别的 row）**，系统会 **先调用该 cell 的** **prepareForReuse()**，然后把该 cell 返回给 cellForRowAt。
> 2. cellForRowAt 得到这个（可能是重用的）cell，然后你在这里调用 cell.configure(with: viewModel) 或者执行绑定逻辑（subscribe / bind）。
> 3. 你为 cell 创建了一个或多个订阅（比如 viewModel 的 observable 绑定到 label、button 的 tap 等），这些订阅会被 .disposed(by: cell.disposeBag) 管理。
> 4. 当再次滚动，系统再次需要更多 cell，会重复第1步 —— **每次复用都会先调用 prepareForReuse()，再重新配置**。
>
> 关键：**prepareForReuse() 被调用的时机是“在你重新配置（bind）之前”** —— 所以这是清理旧订阅的正确时机。

> **为什么会出现“多个绑定 / 重复订阅”？**
>
> 因为 cell 并不是每次都销毁重建；它只是被**复用**。如果你在每次 cellForRowAt / cell.configure(...) 中做订阅，但又没有清理旧订阅，那么每次复用都会在原有订阅上**叠加**新的订阅。
>
> ```swift
> class MyCell: UITableViewCell {
>     // var disposeBag = DisposeBag()  // 假设你把它放在这里并且从未在 prepareForReuse 重置
>     var disposeBag = DisposeBag()
> 
>     func configure(with vm: ItemViewModel) {
>         // 每次 configure 都会创建一个新的订阅
>         vm.title
>            .bind(to: titleLabel.rx.text)
>            .disposed(by: disposeBag)
>     }
> }
> ```
>
> 流程导致的问题：
>
> - 第一次配置：订阅 A（绑定到 titleLabel）
> - cell 滚出并被复用 → prepareForReuse() 没有清理 disposeBag（所以 A 仍在）
> - 第二次配置（用于不同的 indexPath）：订阅 B（绑定到同一个 titleLabel）
> - 现在 vm 的每次更新会触发 A 和 B 两个回调 —— 导致两次更新或显示错误数据。
> - 订阅 A 持有对旧 viewModel 或某些对象的引用，可能造成内存无法释放（内存泄漏）。



### Subject系列（桥梁型-既是Observable，也是Observer）

Subject 既能被外部 onNext 推送事件，也能被订阅接收事件，常用作“跨层通信 / 事件分发 / 热流源”。

**主要类型与行为**

1. PublishSubject<T>
   - 仅将订阅后产生的事件发给订阅者。订阅前的事件丢失。
2. BehaviorSubject<T>
   - 需要一个初始值；新的订阅者会先收到最近一次的值（或初始值），然后接收后续事件。
3. ReplaySubject<T>
   - 缓存最近 n 个值（或全部），新的订阅者会收到这些历史值后再继续接收新值。
4. AsyncSubject<T>
   - 只在 .completed() 时发出最后一个值给订阅者（如果没有 .completed()，则不发任何 .next）。
5. BehaviorRelay<T>（来自 RxRelay）
   - 类似 BehaviorSubject，但没有 .error 和 .completed（不会终止），并且有 value getter/setter，适合做 ViewModel 状态。
   - 注意：Relay 不是 Subject 的子类，但在实践中经常替代 BehaviorSubject 用于状态管理。

```swift
let publish = PublishSubject<String>()
publish.onNext("a")         // 无人接收（若尚未订阅）
publish.subscribe { print($0) } // 订阅后
publish.onNext("b")         // 打印 b

let behavior = BehaviorSubject(value: "init")
behavior.onNext("x")
behavior.subscribe(onNext: { print($0) }) // 会先收到 "x"
```

**实践建议**：尽量少暴露 Subject 给外部（破坏封装）。在 ViewModel 中常用 private let relay = BehaviorRelay(value: ...)，对外暴露为 Observable 或 Driver。



### Scheduler（线程模型）

**核心概念**：Scheduler 决定“在哪个线程/队列上执行工作或分发事件”。Rx 把线程控制抽象为 SchedulerType。

**常见 Scheduler**

- MainScheduler.instance：主线程（UI 更新必须在这里）。
- ConcurrentDispatchQueueScheduler(qos:)：并发队列（背景计算）。
- SerialDispatchQueueScheduler：串行队列。
- OperationQueueScheduler：基于 OperationQueue 的调度。

**subscribe(on:) vs observe(on:)**

- subscribe(on:)：决定 **数据产生（订阅/上游）** 在哪个 scheduler 上执行（影响生产方）。常用于将耗时工作推到后台。

> 控制 **Observable 被订阅（subscribe）时** 的线程 —— 也就是“数据源开始生产事件”的线程。

- observe(on:)（RxSwift 6 以前叫 observeOn）：决定 **后续操作（下游）** 在哪个 scheduler 上执行（影响消费者/操作链后续）。常用于把结果切回主线程以更新 UI。

> 控制 **之后的所有操作符执行和订阅者接收事件的线程**。

```swift
someObservable
  .subscribe(on: ConcurrentDispatchQueueScheduler(qos: .background)) // 数据产生在后台
  .map { heavyCompute($0) }
  .observe(on: MainScheduler.instance) // UI 更新回到主线程
  .subscribe(onNext: { value in updateUI(value) })
  .disposed(by: bag)
```

**注意**：subscribe(on:) 影响源的执行位置，不等于把 map 等操作放到该线程——这些操作的执行线程由链中最近的 observe(on:) 决定。

>**map 的执行线程由上游最近的 observe(on:) 决定，而不是 subscribe(on:)。**
>
>当前链上最近的 observe(on:) 是——没有！
>
>所以默认它会在“当前线程”执行，也就是“上游的执行线程”。
>
>→ 因此，在这个例子里，map 会在后台线程执行。
>
>如果我们加一个新的 observe(on:) 在上面，比如：
>
>```swift
>someObservable
>  .subscribe(on: background)
>  .observe(on: MainScheduler.instance)
>  .map { heavyCompute($0) }  // ⚠️ 现在在主线程执行！
>```
>
>那么 .map 就会在主线程执行（因为它被 observe(on:) “接管”了）。



### Operator（操作符）

操作符是 Rx 的核心，通过它把流组合、转换、过滤、处理错误等。下面按类别说明并给出常见用法。

**创建类**

- just, of, from, range, interval, timer

```swift
Observable.interval(.seconds(1), scheduler: MainScheduler.instance)
```

用途：创建不同类型的源（一次性、序列、定时、周期等）。

> interval = 「每隔一段时间发事件」
>
> timer = 「延迟后发事件（可一次，也可周期）」

**转换类**

- map：值映射

```swift
.map { $0 * 2 }
```

- compactMap：去掉 nil 并映射

```swift
Observable.of("1", "2", "three", "4")
    .compactMap { Int($0) } // 尝试把字符串转 Int，失败则丢弃
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
// 输出1， 2， 4
```

> **使用场景**
>
> - 处理用户输入：过滤掉非法值。
> - 网络响应：只保留成功解析的数据。

- flatMap：把元素映射成一个 **新的 Observable**，并**合并它们的事件流**（可以并发订阅多个流）。

> **概念**
>
> - 上游每个元素都会触发一个新的 Observable；
> - 下游会**同时**订阅这些内部 Observable；
> - 所有内部流的事件**会交叉发送**。
>
> ```swift
> let names = PublishSubject<String>()
> 
> names
>     .flatMap { name in
>         return Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
>             .map { "\(name): \($0)" }
>             .take(3)
>     }
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> 
> names.onNext("A")
> DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
>     names.onNext("B")
> }
> // 交叉输出
> A: 0
> B: 0
> A: 1
> B: 1
> A: 2
> B: 2
> ```
>
> **使用场景**
>
> - 并发执行多个异步任务（例如多个网络请求并行发出，收集所有结果）。
> - 转换成新的流并合并结果（如按钮点击 → 启动多个操作）。

- flatMapLatest（switchMap）：只保留最新内部 Observable（常用于搜索），类似 flatMap，但只保留**最新的内部 Observable**，旧的会被取消订阅。

> **概念**
>
> - 上游每发出一个元素，就创建一个新的 Observable；
> - 会**自动 dispose 掉之前的旧流**；
> - 只监听最新的流事件。
>
> ```swift
> let query = PublishSubject<String>()
> 
> query
>     .flatMapLatest { text in
>         return Observable<String>.interval(.seconds(1), scheduler: MainScheduler.instance)
>             .map { "\(text) - \($0)" }
>             .take(3)
>     }
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> 
> query.onNext("apple")
> DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
>     query.onNext("banana")
> }
> // 输出
> apple - 0
> apple - 1
> banana - 0
> banana - 1
> banana - 2
> // 注意：apple 流被中断了，只保留最新的 banana。
> ```

- scan：类似累积的 reduce，每次发出累积结果,类似 reduce，但每次都会发出“中间累积结果”。

> **概念**
>
> - reduce：只在最后发出一个累积结果；
> - scan：每次累积后都发出当前结果（有点像 Swift 的 runningTotal）。
>
> ```swift
> Observable.of(1, 2, 3, 4)
>     .scan(0) { acc, value in
>         acc + value
>     }
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> // 输出
> 1
> 3
> 6
> 10
> 第1步：0 + 1 = 1
> 第2步：1 + 2 = 3
> 第3步：3 + 3 = 6
> 第4步：6 + 4 = 10
> ```
>
> **使用场景**
>
> - 计数、进度累加、分步求和；
> - 实时状态更新，比如计算当前总价或步数；
> - 在 MVVM 中用来维护 “状态流”（Redux-like 思维）。
>
> ```swift
> public func scan<ResultType>(
>     _ initialSeed: ResultType,
>     accumulator: @escaping (ResultType, Element) throws -> ResultType
> ) -> Observable<ResultType> {
>     return Scan(source: self.asObservable(), seed: initialSeed, accumulator: accumulator)
> }
> ```
>
> - initialSeed → 初始值（你传的 0）
> - accumulator → 闭包 (上次结果, 当前元素) -> 新结果
> - 返回新的 Observable，类型是 Observable<ResultType>

**过滤类**

- filter, take, skip, debounce, throttle, distinctUntilChanged
- debounce：防抖（等待一段静默期）
- throttle：节流（周期内只发一次）

> **filter**
>
> 只有满足条件的事件才会继续往下游传递。
>
> ```swift
> Observable.of(1, 2, 3, 4, 5)
>  .filter { $0 % 2 == 0 } // 只保留偶数
>  .subscribe(onNext: { print($0) })
> // 输出 2,4
> ```
>
> 使用场景
>
> - 表单输入校验：只处理合法输入
> - 过滤掉空字符串或错误数据
>
> **take**
>
> 从序列开头取前 N 个事件，然后完成（complete）。
>
> ```swift
> Observable.of(1, 2, 3, 4, 5)
>     .take(3)
>     .subscribe(onNext: { print($0) })
> // 输出：1,2,3
> ```
>
> 应用场景
>
> - 只取前几个结果；
> - 倒计时时只发特定次数；
> - 防止无限流造成资源占用。
>
> **skip**
>
> 丢弃前 N 个事件，从第 N+1 个开始接收。
>
> ```swift
> Observable.of(1, 2, 3, 4, 5)
>     .skip(2)
>     .subscribe(onNext: { print($0) })
> // 输出3,4,5
> ```
>
> 应用场景
>
> - 跳过初始的默认值；
> - 忽略初始化阶段的无效事件。
>
> **debounce**
>
> 在上游停止发事件后一段时间（静默期）才发出最后一个事件。
>
> 换句话说：
>
> - 如果事件连续快速触发（例如用户打字），
> - 只有停止输入超过指定时间才发出最后一次。
>
> ```swift
> let subject = PublishSubject<String>()
> 
> subject
>     .debounce(.milliseconds(500), scheduler: MainScheduler.instance)
>     .subscribe(onNext: { print("发出:", $0) })
>     .disposed(by: disposeBag)
> 
> subject.onNext("a")
> subject.onNext("ab")
> subject.onNext("abc")
> DispatchQueue.main.asyncAfter(deadline: .now() + 0.6) {
>     subject.onNext("abcd")
> }
> //发出: abc    // 0.5秒后
> //发出: abcd   // 又0.5秒后
> ```
>
> 应用场景
>
> - 搜索输入框：用户停止输入后再请求网络；
> - 表单校验：减少频繁验证；
> - 滑动结束后再触发加载。
>
>  **throttle**
>
> throttle：在指定时间间隔内，只允许发送第一个（或最新）事件。
>
> - 它能防止上游 Observable 太频繁发事件；
> - 常用语句是：“在 1 秒内我只关心一次事件”；
> - 类似于“冷却时间机制”。
>
> 时间线表示法（● 表示事件，↓ 表示下游收到）
>
> ```shell
> 上游: ●●●●●●●●●
> 时间: |------|------|------|
> 输出: ↓       ↓       ↓
> ```
>
> - 如果时间窗口是 1 秒，那么每 1 秒只会通过 1 次事件；
> - 期间的其他事件会被忽略或延迟发出（取决于 latest 参数）。
>
> ```swift
> let subject = PublishSubject<String>()
> 
> subject
>     .throttle(.seconds(1), scheduler: MainScheduler.instance)
>     .subscribe(onNext: { print("接收到:", $0) })
>     .disposed(by: disposeBag)
> 
> subject.onNext("A")          // t = 0s
> subject.onNext("B")          // t = 0.2s
> subject.onNext("C")          // t = 0.5s
> DispatchQueue.main.asyncAfter(deadline: .now() + 1.2) {
>     subject.onNext("D")      // t = 1.2s
>     subject.onNext("E")      // t = 1.4s
> }
> // 输出
> //接收到: A
> //接收到: D
> ```
>
> - 第一秒内（0~1s）只发出第一个 A；
> - 第二秒内只发出第一个 D；
> - 其余事件被忽略。
>
> 使用场景
>
> | **场景**         | **说明**             | **示例**                                  |
> | ---------------- | -------------------- | ----------------------------------------- |
> | **按钮防连点**   | 防止重复触发         | button.rx.tap.throttle(...)               |
> | **滚动事件**     | 控制高频滚动触发频率 | scrollView.rx.contentOffset.throttle(...) |
> | **状态更新限流** | 限制频繁状态变化     | viewModel.state.throttle(...)             |
> | **网络轮询控制** | 限制触发请求频率     | 适用于定时请求优化                        |
>
> **debounce和throttle的区别**
>
> | **特性**       | debounce**（防抖）**                       | throttle**（节流）**                   |
> | -------------- | ------------------------------------------ | -------------------------------------- |
> | 行为描述       | 等事件**静止一段时间后**再发出最后一个事件 | 在每个时间窗口内**只允许发出一次事件** |
> | 时间窗口触发点 | 静默结束时                                 | 窗口开始时                             |
> | 常用场景       | 输入框搜索、防抖动操作                     | 按钮防连点、滚动限流                   |
> | 发射时机       | 等待一段“空闲时间”                         | 固定周期（立即或窗口末）               |
> | 结果倾向       | 只发“最后一次”                             | 定期发（第一个或最新）                 |

**组合类**

- merge：并行合并多个 Observable（不等待）
- zip：按索引配对，等待各个流都有对应元素
- combineLatest：任何一个流有新值都用最新值组合
- withLatestFrom：当 A 触发时，取 B 的最新值（常用于“点击 + 当前表单值”）

**示例（表单）**

```swift
Observable.combineLatest(usernameObservable, passwordObservable) { u, p in
    return isValid(u, p)
}
.bind(to: loginButton.rx.isEnabled)
```

> **merge**
>
> merge**：把多个 Observable 的事件**合并到同一个序列中，所有上游 Observable 的事件都会“交叉”地发给下游。
>
> - 所有源 Observable **同时订阅、同时活跃**；
> - 下游会按它们各自的时间顺序接收事件；
> - 当所有上游完成 (.completed) 后，合并流才会完成。
>
> ```swift
> let a = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
>     .map { "A\($0)" }
>     .take(3)
> 
> let b = Observable<Int>.interval(.seconds(2), scheduler: MainScheduler.instance)
>     .map { "B\($0)" }
>     .take(2)
> 
> Observable.merge(a, b)
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> // 输出
> A0
> A1
> B0
> A2
> B1
> ```
>
> - a 每 1 秒发一次；
> - b 每 2 秒发一次；
> - merge 把两者交错合并成一个事件流；
> - 谁先发就谁先进入下游。
>
> 使用场景
>
> | **场景**                                       | **用法示例**                                     |
> | ---------------------------------------------- | ------------------------------------------------ |
> | **并发多个网络请求，等待所有响应**             | 多个 Observable<Response> 合并                   |
> | **同时监听多个输入源（按钮 / 输入框 / 事件）** | Observable.merge(button1.rx.tap, button2.rx.tap) |
> | **合并多个状态流**                             | 例如用户行为、系统事件同时驱动 UI                |
> | **统一处理多个错误流或日志流**                 | 把多个监控流汇总为一个输出流                     |
>
> **merge和flatmap区别**
>
> merge**静态并发合并**:“我手上已经有好几个 Observable，把它们合成一个即可。”
>
> ```swift
> Observable.merge(
>     buttonA.rx.tap.map { "A" },
>     buttonB.rx.tap.map { "B" },
>     buttonC.rx.tap.map { "C" }
> )
> .subscribe(onNext: { print("点击:", $0) })
> // 输出
> 点击: A
> 点击: B
> 点击: C
> // merge 的输入本身就是多个 Observable。
> ```
>
> **flatMap**动态创建并合并
>
> “上游每来一个元素，我再用它创建一个新的 Observable，并把这些内部流合并。”
>
> ```swift
> let userIDs = Observable.of(1, 2, 3)
> 
> userIDs
>     .flatMap { id in
>         api.getUserInfo(id: id) // 返回 Observable<User>
>     }
>     .subscribe(onNext: { user in
>         print("收到用户信息:", user)
>     })
> //flatMap 的输入是单个 Observable，输出是多个内部流的合并。
> ```
>
> **concat**
>
> 按顺序连接多个 Observable，**当前一个 Observable 完成 (onCompleted) 后**，才会订阅并开始下一个 Observable。
>
> ```swift
> let a = Observable.of("A1", "A2")
> let b = Observable.of("B1", "B2")
> 
> Observable.concat(a, b)
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> // 输出
> A1
> A2
> B1
> B2
> // 只有当 a 发出完成 (onCompleted) 后，
> // b 才会开始发事件。
> ```
>
> | **特性** | concat                           | merge                  |
> | -------- | -------------------------------- | ---------------------- |
> | 执行方式 | **串行**：上一个完成才开始下一个 | **并行**：所有同时执行 |
> | 顺序保证 | ✅ 按添加顺序                     | ❌ 谁先发谁先出         |
> | 适合场景 | 有严格先后顺序                   | 可以并发执行的任务     |
> | 完成条件 | 所有上游都完成                   | 所有上游都完成         |
>
> **zip**
>
> **zip** 会把多个 Observable **对应位置的元素** 取出，然后用你提供的函数“打包（zip）”成一个新元素，直到任意一个源序列发完为止。
>
> ```swift
> public static func zip<O1, O2, ResultType>(
>     _ source1: O1,
>     _ source2: O2,
>     resultSelector: @escaping (O1.Element, O2.Element) throws -> ResultType
> ) -> Observable<ResultType>
> ```
>
> | **参数**         | **含义**                              |
> | ---------------- | ------------------------------------- |
> | source1, source2 | 要合并的两个 Observable               |
> | resultSelector   | 配对时如何组合两个元素                |
> | 返回             | 一个新的 Observable，发出组合后的结果 |
>
> ```swift
> let numbers = Observable.of(1, 2, 3)
> let letters = Observable.of("A", "B", "C", "D")
> 
> Observable.zip(numbers, letters) { num, letter in
>     "\(num)-\(letter)"
> }
> .subscribe(onNext: { print($0) })
> .disposed(by: disposeBag)
> // 输出
> 1-A
> 2-B
> 3-C
> // 注意：letters 比 numbers 多一个元素 D，
> // 但 zip 会在较短的流（numbers）完成时结束。
> ```
>
> 工作机制：
>
> ```shell
> numbers: 1 ——— 2 ——— 3 ———
> letters: A —— B —— C —— D
> output : 1A —— 2B —— 3C —— [complete]
> // 当两个流都各发出一个值后，zip 才发出一对。
> // 任意一个流缺少配对项，事件不会发出。
> ```
>
> **combineLatest**
>
> **combineLatest** 会同时订阅多个 Observable。每当任意一个 Observable 发出新值时，它会取**所有 Observable 的最新值**，用 resultSelector 组合并发出一个新事件。
>
> ```swift
> public static func combineLatest<O1, O2, ResultType>(
>     _ source1: O1,
>     _ source2: O2,
>     resultSelector: @escaping (O1.Element, O2.Element) throws -> ResultType
> ) -> Observable<ResultType>
> ```
>
> | source1, source2 | 要合并的 Observable  |
> | ---------------- | -------------------- |
> | resultSelector   | 合并两个最新值的闭包 |
> | 返回             | 组合后的 Observable  |
>
> ```swift
> let first = BehaviorSubject(value: "A")
> let second = BehaviorSubject(value: "1")
> 
> Observable.combineLatest(first, second) { a, b in
>     "\(a)\(b)"
> }
> .subscribe(onNext: { print("输出:", $0) })
> .disposed(by: disposeBag)
> 
> first.onNext("B")
> second.onNext("2")
> second.onNext("3")
> first.onNext("C")
> // 输出
> 输出: A1   // 初始值组合
> 输出: B1   // first 改变
> 输出: B2   // second 改变
> 输出: B3
> 输出: C3   // first 改变，取 second 最新的 3
> // 只要任意一个 Observable 改变，就触发新事件；
> // 始终使用每个 Observable 的最新值。
> ```
>
> **withLatestForm**
>
> 当**上游 Observable（触发源）**发出事件时，**从另一个 Observable（数据源）**中取出最新值，并将该值发送给下游。
>
> ```swift
> let textInput = textField.rx.text.orEmpty.asObservable()
> let buttonTap = submitButton.rx.tap.asObservable()
> 
> buttonTap
>     .withLatestFrom(textInput)
>     .subscribe(onNext: { latestText in
>         print("按钮点击时输入框内容：\(latestText)")
>     })
>     .disposed(by: disposeBag)
> // 输出
> （假设用户输入 "RxSwift" 然后点击按钮）
> 按钮点击时输入框内容：RxSwift
> •	只有当 buttonTap 触发时，才取 textInput 的最新值；
> •	输入框变化不会主动触发输出；
> •	相当于“点击时拿最新输入”。
> ```
>
> ```swift
> let username = usernameTextField.rx.text.orEmpty.asObservable()
> let password = passwordTextField.rx.text.orEmpty.asObservable()
> 
> let credentials = Observable.combineLatest(username, password) {
>     (username: $0, password: $1)
> }
> 
> loginButton.rx.tap
>     .withLatestFrom(credentials)
>     .subscribe(onNext: { cred in
>         print("登录请求：", cred.username, cred.password)
>     })
>     .disposed(by: disposeBag)
> ```
>
> - 每次点击登录按钮时，拿到用户名和密码的最新值；
> - 输入变化不会触发请求，只有点击才触发；
> - 非常适合登录/注册/提交按钮。

**错误处理类**

- catchError, catchErrorJustReturn, retry, retryWhen, materialize/dematerialize
- materialize() 把事件（包括 error/completed）包装成 Event，便于把错误当作数据处理。

> **catchError**
>
> **捕获错误并返回新的 Observable**
>
> 当上游序列出错时，不让错误终止序列，而是**切换到另一个 Observable** 继续发事件。
>
> ```swift
> let source = Observable<Int>.create { observer in
>     observer.onNext(1)
>     observer.onNext(2)
>     observer.onError(MyError.sample)
>     return Disposables.create()
> }
> 
> source
>     .catchError { error in
>         print("捕获错误：", error)
>         return Observable.of(100, 200, 300)
>     }
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> // 输出
> 1
> 2
> 捕获错误： sample
> 100
> 200
> 300
> •	源序列出错；
> •	catchError 捕获并返回一个“替代序列”；
> •	程序不中断，继续发出替代数据。
> ```
>
> **常见用途**
>
> - 网络请求失败时返回缓存；
> - 错误回退（fallback）逻辑；
> - 防止流被错误中断。
>
> **catchErrorJustReturn**
>
> **捕获错误并返回一个默认值**
>
> 是 catchError 的简化版本。当出错时，不再切换 Observable，而是直接发出一个默认值并结束。
>
> ```swift
> let source = Observable.of(1, 2, 3)
>     .concat(Observable.error(MyError.sample))
> 
> source
>     .catchErrorJustReturn(999)
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> // 输出
> 1
> 2
> 3
> 999
> •	前面正常值照常发出；
> •	错误发生时发出 999；
> •	然后序列结束（不会再抛错）。
> ```
>
> **常见用途**
>
> - 网络错误时展示“默认数据”；
> - 数据加载失败时使用本地占位；
> - 简化错误处理逻辑。
>
> **retry**
>
> 发生错误时自动重新订阅（重试）
>
> 当发生错误时，**自动重新订阅上游**，重新执行整个序列。通常用于：网络请求失败自动重试。
>
> ```swift
> var attempt = 0
> 
> let source = Observable<Int>.create { observer in
>     attempt += 1
>     print("尝试第 \(attempt) 次")
>     
>     observer.onNext(1)
>     if attempt < 3 {
>         observer.onError(MyError.sample)
>     } else {
>         observer.onNext(2)
>         observer.onCompleted()
>     }
>     return Disposables.create()
> }
> 
> source
>     .retry(3)
>     .subscribe(
>         onNext: { print("收到:", $0) },
>         onError: { print("最终错误:", $0) },
>         onCompleted: { print("完成") }
>     )
>     .disposed(by: disposeBag)
> // 输出
> 尝试第 1 次
> 收到: 1
> 尝试第 2 次
> 收到: 1
> 尝试第 3 次
> 收到: 1
> 收到: 2
> 完成
> •	出错时重新开始；
> •	重试 3 次后仍失败才真正报错；
> •	若成功则正常完成。
> ```
>
> **常见用途**
>
> - 网络请求自动重试；
> - 设备连接重试；
> - 登录/鉴权请求容错。
>
> **retryWhen**
>
> **条件控制的重试（可延时、可选择是否重试）**
>
> 与 retry 类似，但提供**灵活的错误处理逻辑**。可以控制**何时、是否重试**。
>
> ```swift
> .retryWhen { errorObservable in ... }
> ```
>
> errorObservable 会发出上游的错误事件，
>
> 你可以决定：
>
> - 立即重试；
> - 延迟重试；
> - 或不重试
>
> ```swift
> var attempt = 0
> 
> let source = Observable<Int>.create { observer in
>     attempt += 1
>     print("尝试第 \(attempt) 次")
>     if attempt < 3 {
>         observer.onError(MyError.sample)
>     } else {
>         observer.onNext(999)
>         observer.onCompleted()
>     }
>     return Disposables.create()
> }
> 
> source
>     .retryWhen { errorObservable in
>         errorObservable.enumerated().flatMap { (index, error) -> Observable<Int> in
>             if index >= 1 {
>                 return Observable.error(error) // 重试两次后放弃
>             }
>             print("延迟 1 秒重试...")
>             return Observable<Int>.timer(.seconds(1), scheduler: MainScheduler.instance)
>         }
>     }
>     .subscribe(
>         onNext: { print("收到:", $0) },
>         onError: { print("最终错误:", $0) },
>         onCompleted: { print("完成") }
>     )
>     .disposed(by: disposeBag)
> // 输出
> 尝试第 1 次
> 延迟 1 秒重试...
> 尝试第 2 次
> 延迟 1 秒重试...
> 尝试第 3 次
> 收到: 999
> 完成
> •	出错 → 延迟 1 秒后重试；
> •	连续 2 次错误后仍出错 → 停止重试；
> •	成功后完成。
> ```
>
> **materialize / dematerialize**
>
> **把事件本身当作值流动**
>
> 在 Rx 中，事件（Event）本身包括：
>
> ```swift
> enum Event<Element> {
>     case next(Element)
>     case error(Error)
>     case completed
> }
> ```
>
> materialize 会把事件（next, error, completed）**转换为普通值发出**；
>
> dematerialize 则相反，把值重新“还原”为事件。
>
> ```swift
> let source = Observable<Int>.create { observer in
>     observer.onNext(1)
>     observer.onNext(2)
>     // .onError() 调用后，整个闭包虽然继续往下执行代码，
>     // 但Rx 内部会丢弃所有后续事件。                                 
>     observer.onError(MyError.sample)
>     observer.onNext(3)
>     return Disposables.create()
> }
> 
> source
>     .materialize() // 把事件封装为值
>     .subscribe(onNext: { event in
>         print("事件:", event)
>     })
>     .disposed(by: disposeBag)
> // 输出
> 事件: next(1)
> 事件: next(2)
> 事件: error(sample)
> •	错误不会终止序列；
> •	因为“错误”现在只是普通值。
> // materialize() 并不会“阻止错误”，它的作用是：把事件本身（next / error / completed）当作值发出。
> // 但它能捕获的事件，仅限在上游真正发出的事件范围内。
> ```
>
> ```swift
> source
>     .materialize()
>     .filter {
>         if case .error = $0 { return false } // 过滤错误事件
>         return true
>     }
>     .dematerialize()
>     .subscribe(onNext: { print($0) })
> // 输出
> 1
> 2
> •	过滤掉 .error；
> •	再用 dematerialize() 恢复为正常流；
> •	完美跳过错误。
> ```
>
> 

**实用类**

- share() / share(replay:scope:)：把冷 Observable 转为热并共享订阅（避免重复执行）。
- replay / publish / multicast：更强的多播控制（与 ConnectableObservable 相关）。
- startWith：在序列开始前发出一个初始值。
- delay：延迟事件。

```swift
// 示例（避免重复网络请求）
let shared = apiCall()
    .asObservable()
    .share(replay: 1, scope: .whileConnected)
```

**实践建议**：了解每个操作符的“语义”与“线程/取消行为”（例如 flatMapLatest 会取消旧请求，而 flatMap 不会）。

> - **Cold Observable（冷）**：每个订阅者都会**从头开始执行**，相互独立。
>
>   例：网络请求、文件读取、计算任务。
>
> - **Hot Observable（热）**：所有订阅者共享同一个数据源，**不会重新执行**。
>
>   例：按钮点击、通知流、传感器。
>
> **share**
>
> share() 将冷序列变为**热序列**，并**共享订阅结果**。所有订阅者共享同一份执行逻辑，避免重复副作用（如重复网络请求）。
>
> ```swift
> let source = Observable<Int>.create { observer in
>     print("🔵 执行了网络请求")
>     observer.onNext(Int.random(in: 0...100))
>     observer.onCompleted()
>     return Disposables.create()
> }
> 
> source.subscribe(onNext: { print("订阅1: \($0)") }).disposed(by: disposeBag)
> source.subscribe(onNext: { print("订阅2: \($0)") }).disposed(by: disposeBag)
> // 输出
> 🔵 执行了网络请求
> 订阅1: 32
> 🔵 执行了网络请求
> 订阅2: 78
> 两个订阅分别触发两次请求（冷）。
> ```
>
> ```swift
> let shared = source.share()
> 
> shared.subscribe(onNext: { print("订阅1: \($0)") }).disposed(by: disposeBag)
> shared.subscribe(onNext: { print("订阅2: \($0)") }).disposed(by: disposeBag)
> // 输出
> 🔵 执行了网络请求
> 订阅1: 57
> 订阅2: 57
> 两个订阅共享结果，只执行一次。
> ```
>
>  **share(replay:scope:)** **参数说明**
>
> ```swift
> .share(replay: Int, scope: SubjectLifetimeScope)
> ```
>
> | **参数** | **说明**                             |
> | -------- | ------------------------------------ |
> | replay   | 重放最近 N 个事件给新订阅者          |
> | scope    | 生命周期范围（默认 .whileConnected） |
>
> ```swift
> // 所有新订阅者都能立即拿到最近 1 个事件（类似缓存）。
> .share(replay: 1, scope: .forever)
> ```
>
> **使用场景**
>
> - 避免重复请求或重复副作用；
> - UI 多处监听同一个异步流；
> - 缓存最近结果供新订阅者立即使用。
>
> **replay / publish / multicast**
>
> 它们与 share() 类似，但更底层，属于 **ConnectableObservable** 系列，需要手动调用 .connect() 开始发射。
>
> **publish() ——** **变热，不重放**
>
> ```swift
> let source = Observable.of(1, 2, 3)
> let published = source.publish()
> 
> published.subscribe(onNext: { print("订阅1:", $0) }).disposed(by: disposeBag)
> 
> published.connect() // 手动启动发射
> 
> published.subscribe(onNext: { print("订阅2:", $0) }).disposed(by: disposeBag)
> // 输出
> 订阅1: 1
> 订阅1: 2
> 订阅1: 3
> // 订阅2在 connect 后才订阅，不会收到之前的值。
> ```
>
> **replay(_:) ——** **变热** **+** **重放最近** **N** **个事件**
>
> ```swift
> let replayed = source.replay(1)
> replayed.connect()
> 
> replayed.subscribe(onNext: { print("订阅:", $0) }).disposed(by: disposeBag)
> // 新订阅者会立刻收到最近的 1 个事件。
> ```
>
> **multicast(_:) ——** **自定义** **Subject** **作为共享中心**
>
> ```swift
> let subject = PublishSubject<Int>()
> let multicasted = source.multicast(subject)
> 
> multicasted.subscribe(onNext: { print("订阅1:", $0) }).disposed(by: disposeBag)
> multicasted.connect()
> multicasted.subscribe(onNext: { print("订阅2:", $0) }).disposed(by: disposeBag)
> // multicast 让多个订阅共享同一个 Subject，
> //所有事件从同一个 Subject 发出（可精细控制缓存/转发逻辑）。
> ```
>
> **startWith**
>
> 在序列开始前，**先发出一个（或多个）初始值**。常用于：初始化 UI 状态、设置默认值。
>
> ```swift
> Observable.of("B", "C")
>     .startWith("A")
>     .subscribe(onNext: { print($0) })
>     .disposed(by: disposeBag)
> // 输出
> A
> B
> C
> // 简单直观，就是给序列加个“头”。
> ```
>
> **应用场景**
>
> • 网络请求前先显示缓存或 Loading；
>
> • 状态流初始值（比如登录状态初始为 .idle）；
>
> • 默认 UI 值（如默认选项）。
>
> **delay**
>
> 延迟发出所有事件（包括完成事件）。常用于：动画延迟、延时显示、网络重试间隔。
>
> ```swift
> Observable.of("A", "B", "C")
>     .delay(.seconds(2), scheduler: MainScheduler.instance)
>     .subscribe(onNext: { print("收到:", $0, "时间:", Date()) })
>     .disposed(by: disposeBag)
> // 输出
> （2 秒后）
> 收到: A 时间: ...
> 收到: B 时间: ...
> 收到: C 时间: ...
> // 所有事件延迟 2 秒后发出。
> ```
>
> **注意**
>
> - delay 会**延迟整个事件流**；
> - 如果想只延迟错误，可以用 delaySubscription；
> - 对 UI 流要使用 MainScheduler.instance。

### 资源管理（多订阅共享 & 内存泄露防范）

**多订阅共享**

- 当同一个冷 Observable 被多个订阅者订阅时，会导致多次执行（例如发起多次网络请求）。为避免：
  - 使用 share(replay:1) 或 publish().refCount() 把它变为热流并缓存结果。

```swift
let request = URLSession.shared.rx.data(request: req)
    .map { parse($0) }
    .share(replay: 1, scope: .whileConnected)
```

- multicast（配合 ConnectableObservable）允许你控制何时连接源并开始发射（适用于复杂场景）。

**内存泄漏防范**

- 在闭包中引用 self 要小心，使用 [weak self] 或 Rx 的 withUnretained(self) / compactMap { [weak self] in self? }：

```swift
observable
  .withUnretained(self)
  .subscribe(onNext: { owner, value in
      owner.doSomething()
  })
  .disposed(by: bag)
```

- 使用 DisposeBag 按生命周期管理订阅（VC、Cell、ViewModel 各自管理）。
- 避免把 Subject 暴露为可外部写入（防止外部意外持有/发送）。对外暴露只读 Observable：

```swift
private let subject = PublishSubject<String>()
var events: Observable<String> { subject.asObservable() }
```

**Cell 重用特别提醒**：每个 cell 都要有独立的 DisposeBag 并在 prepareForReuse() 里重置。



### 使用注意

- **何时用 Relay（BehaviorRelay）**：用于表示可变的、不会 complete/error 的状态（ViewModel state）。因为 UI 状态一般不会想让流终止或出错。
- **Single / Completable / Maybe**：用于将请求语义更明确化（Single 表示要么成功要么失败）。把网络请求实现为 Single 是常见做法。
- **Driver / Signal（RxCocoa Traits）**：用于 UI 绑定的安全 wrapper：保证在主线程、不会 error、共享订阅（Driver 还会重放最新值）。推荐用在 ViewModel → View 的绑定中。



### 常见坑 & 调试建议

- **忘记 disposed(by:)** → 订阅泄漏。
- **在 cell 中未重置 bag** → 事件重复触发 / 内存泄漏。
- **误用 flatMap 而非 flatMapLatest** → 老请求回包覆盖新结果（搜索场景）。
- **滥用 Subject 替代结构化状态管理** → 代码难以维护/测试。
- **没有正确控制线程** → UI 更新在后台线程崩溃或卡顿。
- **调试手段**：debug()、在关键点 print()、使用 RxTest + TestScheduler 写时间相关单元测试。



## RxCocoa 构件（UI 层响应式封装）

### ControlProperty/ControlEvent

**核心概念**

- ControlProperty<Value>：表示**控件可读写的状态属性**（例如 UITextField.text）。语义上保证在主线程、不会产生 error、并且是“可绑定”的（支持双向绑定）。
- ControlEvent<Value>：表示**控件的事件流**（例如 UIButton.tap, UIScrollView.didScroll）。也是主线程、无错误、不会重放（通常不保存最新值，只表示事件）。

**区别要点**

- ControlProperty 是「值」语义（可读 + 可写），例如 textField.rx.text。它既可 bind(to:) 也可 asObservable()。
- ControlEvent 是「事件」语义（只发事件），例如 button.rx.tap。通常用于触发动作而不是保存状态。

```swift
// 读取输入并实时显示到 label（单向）
textField.rx.text
    .map { $0 ?? "" }
    .bind(to: label.rx.text)
    .disposed(by: bag)

// 按钮点击事件
button.rx.tap
    .subscribe(onNext: { print("tapped") })
    .disposed(by: bag)
```

**注意**

- 这两个类型都封装了线程与错误语义，开发者不用为 UI 线程或错误处理写额外代码（这是 RxCocoa 的舒适区）。

**ControlProperty/ControlEvent/Driver区别**

| **特性**     | **ControlProperty** | **ControlEvent**          | **Driver**          |
| ------------ | ------------------- | ------------------------- | ------------------- |
| 来自         | UIKit 控件属性      | UIKit 控件事件            | ViewModel 输出      |
| 是否可写     | ✅ 可双向绑定        | ❌ 只读                    | ❌ 只读              |
| 是否会错误   | ❌ 永不出错          | ❌ 永不出错                | ❌ 永不出错          |
| 是否重放     | ✅ 有最新值          | ❌ 不重放                  | ✅ 重放最新值        |
| 是否在主线程 | ✅ 是                | ✅ 是                      | ✅ 是                |
| 常用绑定方法 | .bind(to:)          | .bind(to:) / .subscribe() | .drive()            |
| 是否自动共享 | ✅ 是                | ✅ 是                      | ✅ 是                |
| 典型用途     | 输入框状态          | 点击事件                  | ViewModel 输出到 UI |

### Driver（UI 绑定的首选 Trait）

**核心语义（四项）**

- 在 **主线程**（MainScheduler）上发送事件
- **永不 error**（如果源会 error，你要在转换处降级）
- **共享副作用**（内部用了 share(replay:1)）——多个订阅不会重复触发上游工作
- **重放最新值**（订阅时会立即收到最近的值）

因此 Driver 非常适合 **ViewModel → View** 的输出（标签文本、表格数据、按钮是否可用等）。

**如何从 Observable 转成 Driver**

```swift
// BehaviorRelay -> Driver
let relay = BehaviorRelay(value: [String]())
let itemsDriver = relay.asDriver() // 不会 error

// Observable -> Driver（在可能 error 的地方需要降级）
let obs: Observable<[Model]> = api.fetchModels()
let safeDriver = obs.asDriver(onErrorJustReturn: [])
```

**示例：用 Driver 绑定 tableView**

```swift
viewModel.itemsDriver
    .drive(tableView.rx.items(cellIdentifier: "Cell")) { index, model, cell in
        cell.textLabel?.text = model.title
    }
    .disposed(by: bag)
```

**实践建议**

- UI 层输出优先用 Driver。如果某一步可能 error，用 asDriver(onErrorJustReturn:) 或 asDriver(onErrorDriveWith:) 降级处理。

**Diriver和Signal区别**

共同点

| **特性**       | **说明**                           |
| -------------- | ---------------------------------- |
| 🔸 主线程       | 所有事件都在 MainScheduler 上发送  |
| 🔸 无错误       | 永远不会发出 .error                |
| 🔸 自动共享     | 内部已实现 share()，不会重复副作用 |
| 🔸 常用于 UI 层 | ViewModel → View 数据或事件绑定    |
| 🔸 绑定函数     | 使用 .drive()                      |

区别

| **型**     | **是否重放最新值** | **常用于**                   |
| ---------- | ------------------ | ---------------------------- |
| **Driver** | ✅ 是（重放最新值） | 状态（UI 展示数据）          |
| **Signal** | ❌ 否（不重放）     | 事件（用户操作反馈、弹窗等） |



### Binder（自定义安全绑定）

**作用**：当你需要把一个流「安全地」绑定到 UI 的自定义属性/方法上时，用 Binder。Binder 保证：

- 在主线程执行绑定代码
- 不会传播 error（UI 绑定不该终止）

**创建方式**

```swift
extension Reactive where Base: UILabel {
    var attributedTextBinder: Binder<NSAttributedString> {
        return Binder(base) { label, attr in
            label.attributedText = attr
        }
    }
}

// 使用
someObservable
    .map { ... NSAttributedString ... }
    .bind(to: myLabel.rx.attributedTextBinder)
    .disposed(by: bag)
```

**为何不用 subscribe/onNext 直接设置？**

Binder 把「如何写 UI」的细节封装，确保主线程并且避免错误传播，还能在多个地方重用该绑定逻辑，语义更清晰。

**核心区别于 Subject**

- Relay（来自 RxRelay）**没有 error 和 completed**，不会终止流，适合 UI 状态（状态通常不应该“终止”）。
- BehaviorRelay：有当前值 (value)，订阅者会收到当前值然后继续接收新值 —— 很适合表示 ViewModel 的 state。
- PublishRelay：不会保留最近值，只发订阅后产生的事件，适合作为事件通道（例如“展示 toast”）。

**示例（ViewModel state）**

```swift
class VM {
    private let _text = BehaviorRelay<String>(value: "")
    var text: Driver<String> { _text.asDriver() }

    func updateInput(_ s: String) {
        _text.accept(s)
    }
}
```

**示例（事件）**

```swift
let toastRelay = PublishRelay<String>()
toastRelay.accept("Saved")
toastRelay.asSignal() // 可以转换为 Signal 在 UI 层订阅
```

**实践建议**

- 用 BehaviorRelay 保存 UI 状态（表格数据、表单当前值）。
- 用 PublishRelay 发短暂事件（navigation, toast, alert）。



### Reactive Extensions（.rx 命名空间）

**说明**

- .rx 命名空间是 RxCocoa 为 UIKit/Apple 框架提供的扩展集合。几乎每个常见控件都提供了 rx 的 property/event/ binder。
  - textField.rx.text、button.rx.tap、tableView.rx.items、tableView.rx.modelSelected、等等。

**TableView 绑定示例**

```swift
// ViewModel 提供 Driver<[Model]>
viewModel.itemsDriver
    .drive(tableView.rx.items(cellIdentifier: "Cell", cellType: UITableViewCell.self)) { index, model, cell in
        cell.textLabel?.text = model.title
    }
    .disposed(by: bag)

// 选择事件
tableView.rx.modelSelected(Model.self)
    .subscribe(onNext: { model in
        // handle selection
    }).disposed(by: bag)
```

**更多扩展**

- rx.isHidden, rx.isEnabled, rx.image（UIImageView）等常见属性也可以直接绑定，非常方便。



### 双向绑定机制（View ↔ ViewModel 自动同步）

**目标**：实现「控件的输入改变 ViewModel 状态」并且「ViewModel 的状态变更反映到控件上」，形成双向同步。

**常见做法（使用 BehaviorRelay + ControlProperty）**

```swift
// ViewModel
class VM {
    let username = BehaviorRelay<String>(value: "")
}

// ViewController
let vm = VM()
textField.rx.text.orEmpty
    .bind(to: vm.username)            // View -> VM
    .disposed(by: bag)

vm.username
    .asDriver()
    .drive(textField.rx.text)         // VM -> View
    .disposed(by: bag)
```

**注意：会不会造成循环？**

- Rx 的 bind/drive 本质上是订阅。因为 BehaviorRelay.accept 在内部会检查值是否改变（或你可以用 distinctUntilChanged()），通常不会导致无限循环。但要小心：
  - 在某些情况下（例如绑定会对值做不同的变换），可能产生重复赋值，影响性能或引发 UI 重绘。常用 distinctUntilChanged() 防止不必要的重复事件。

**双向绑定的助力函数（常见模式）**

```swift
// 一个常用的双向绑定辅助（示意）
func twoWayBind(_ relay: BehaviorRelay<String>, _ control: ControlProperty<String?>) -> Disposable {
    let d1 = relay.asObservable()
        .bind(to: control) // VM -> View
    let d2 = control.orEmpty
        .bind(to: relay)   // View -> VM
    return Disposables.create(d1, d2)
}
```

**实践建议**

- 双向绑定在表单场景非常好用。但在复杂逻辑（多个来源更新同一状态）要谨慎，尽量设计单一数据来源（single source of truth）。
- 对于需要防抖/校验的输入，最好在 View -> VM 路径上做 debounce 或校验，再决定是否 accept 到 state。



### 额外细节、常见坑与最佳实践（整合）

1. **UI 绑定尽量用 Driver / Signal / ControlProperty / ControlEvent**，这些 trait 已经帮你处理线程与错误。
2. **避免在闭包里强引用 view controller / view**，使用 [weak self] 或 withUnretained(self)。
3. **双向绑定注意 distinctUntilChanged**，避免不必要回流与 UI 重绘：

```swift
textField.rx.text.orEmpty
  .distinctUntilChanged()
  .bind(to: vm.username)
  .disposed(by: bag)
```

4. **Cell 里的 rx 绑定必须在 reuse 时清理**（reset disposeBag）。

5. **用 Relay 替代 Subject 作为 ViewModel 的状态容器**（Relay 无 error/completed，更适合长期状态）。
6. 事件（一次性）用 PublishRelay.asSignal()，信号语义更贴近 UI 事件流**。Signal 与 Driver 的区别在于：Signal 不会重放最新值（适用于事件），Driver 会（适用于状态）。**
7. **自定义 Binder** 可以把复杂 UI 更新逻辑封装在一个地方，保证安全（主线程且无错误传播）。
8. **尽量把 UI 更新的副作用放在 View 层，ViewModel 只提供 Driver/Signal/Observable**（清晰分层）。



## 响应式架构与模式（Reactive Architecture）

### MVVM + RxSwift（Input / Output 模式）

**Input / Output 模式**

在 Rx 架构中，常用一种明确的 ViewModel 接口约定：

```swift
protocol ViewModelType {
    associatedtype Input
    associatedtype Output

    func transform(_ input: Input) -> Output
}
```

- **Input**：View 层发出的事件流（例如点击、输入、滚动）。
- **Output**：ViewModel 输出的状态流（UI 显示用）。

**ViewController 不再直接操作属性，而是把流送入 ViewModel，订阅输出流。**

**示例：登录页 LoginViewModel**

```swift
struct LoginViewModel: ViewModelType {
    struct Input {
        let username: Observable<String>
        let password: Observable<String>
        let loginTap: Observable<Void>
    }

    struct Output {
        let isLoginEnabled: Driver<Bool>
        let loginResult: Signal<Result<User, Error>>
    }

    func transform(_ input: Input) -> Output {
        let credentialsValid = Observable
            .combineLatest(input.username, input.password)
            .map { !$0.isEmpty && !$1.isEmpty }

        let loginResult = input.loginTap
            .withLatestFrom(Observable.combineLatest(input.username, input.password))
            .flatMapLatest { username, password in
                API.login(username, password)
                    .asObservable()
                    .materialize()
            }
            .map { event -> Result<User, Error> in
                switch event {
                case .next(let user): return .success(user)
                case .error(let err): return .failure(err)
                default: return .failure(NSError(domain: "", code: -1))
                }
            }
            .asSignal(onErrorJustReturn: .failure(NSError(domain: "", code: -1)))

        return Output(
            isLoginEnabled: credentialsValid.asDriver(onErrorJustReturn: false),
            loginResult: loginResult
        )
    }
}
```

**ViewController 绑定**

```swift
let input = LoginViewModel.Input(
    username: usernameTextField.rx.text.orEmpty.asObservable(),
    password: passwordTextField.rx.text.orEmpty.asObservable(),
    loginTap: loginButton.rx.tap.asObservable()
)

let output = viewModel.transform(input)

output.isLoginEnabled
    .drive(loginButton.rx.isEnabled)
    .disposed(by: bag)

output.loginResult
    .emit(onNext: { result in
        switch result {
        case .success(let user): print("Welcome \(user)")
        case .failure(let error): print("Error: \(error)")
        }
    })
    .disposed(by: bag)
```



### 响应式网络层设计

**用 Observable / Single 封装 URLSession**

传统写法：

```swift
func fetchUser(id: String, completion: @escaping (User?, Error?) -> Void) {
    URLSession.shared.dataTask(with: url) { data, res, err in ... }.resume()
}
```

Rx 封装：

```swift
func fetchUser(id: String) -> Single<User> {
    return Single.create { single in
        let task = URLSession.shared.dataTask(with: url) { data, res, err in
            if let err = err {
                single(.failure(err))
                return
            }
            do {
                let user = try JSONDecoder().decode(User.self, from: data!)
                single(.success(user))
            } catch {
                single(.failure(error))
            }
        }
        task.resume()
        return Disposables.create { task.cancel() } // 支持取消
    }
}
```

**请求取消**

由于 Single.create 里返回 Disposables.create { task.cancel() }，只要订阅被 disposed（例如 ViewController 销毁），请求自动取消。

**错误处理与重试**

```swift
fetchUser(id: "1")
    .retry(3) // 自动重试
    .catchError { _ in Single.just(User.guest) }
    .subscribe(onSuccess: { user in ... })
    .disposed(by: bag)
```



### 响应式状态管理

**BehaviorRelay 管理状态**

BehaviorRelay 是状态的核心：持有当前值、可被订阅、更新时自动通知。

```swift
let items = BehaviorRelay<[Item]>(value: [])

func add(_ item: Item) {
    items.accept(items.value + [item])
}
```

任何订阅 items.asObservable() 的地方都会收到更新。

**combineLatest 合成全局状态**

当多个局部状态组合成更复杂 UI 状态时：

```swift
Observable
    .combineLatest(usernameRelay, passwordRelay, termsAcceptedRelay)
    .map { !$0.0.isEmpty && !$0.1.isEmpty && $0.2 }
    .bind(to: loginButton.rx.isEnabled)
    .disposed(by: bag)
```

**状态驱动 UI**，而非 UI 反向修改状态。



### 模块通信

**PublishSubject 广播事件**

当两个模块之间需要通信（例如通知刷新数据、页面关闭后回调）：

```swift
// 在 A 模块定义
let reloadSubject = PublishSubject<Void>()

// 在 B 模块监听
reloadSubject.subscribe(onNext: { [weak self] in
    self?.reloadData()
}).disposed(by: bag)
```

A → B 就像一个事件通道。

**替代 Notification**

传统的 NotificationCenter：

```swift
NotificationCenter.default.post(name: .didUpdate, object: nil)
```

用 Rx:

```swift
extension Reactive where Base: NotificationCenter {
    var didUpdate: Observable<Void> {
        return notification(.didUpdate).map { _ in () }
    }
}
```

**好处**：完全类型安全，可组合，可取消订阅。



### Coordinator + Rx（响应式导航）

**Coordinator 模式简介**

Coordinator 把 “页面跳转 / 导航逻辑” 从 ViewController 解耦。

**传统：**

ViewController 内部 push、present 新页面。

**Rx 架构：**

ViewModel 发出导航事件（Signal/PublishRelay），Coordinator 订阅并执行导航。

```swift
class LoginCoordinator {
    let showHome = PublishRelay<Void>()

    func start() {
        showHome.subscribe(onNext: { [weak self] in
            self?.navigateToHome()
        }).disposed(by: bag)
    }
}
```

ViewModel：

```swift
loginSuccess
    .emit(to: coordinator.showHome)
    .disposed(by: bag)
```

**生命周期绑定**

Coordinator 或 ViewModel 的生命周期和订阅关联：

当页面销毁时 dispose，自动停止事件流，避免内存泄漏。

> 核心概念
>
> | **概念**         | **来自** | **类型**                                            | **功能**                                       |
> | ---------------- | -------- | --------------------------------------------------- | ---------------------------------------------- |
> | **PublishRelay** | RxCocoa  | 类似 PublishSubject，但不会发送 .error / .completed | “事件中继器”，可以 .accept() 或被 .emit() 触发 |
> | **emit(to:)**    | RxCocoa  | Signal 的绑定方法                                   | 把 Signal 的事件发送到目标（如 Relay）         |
>
> 执行流程
>
> **(1) LoginCoordinator** **定义了一个中继器**
>
> ```swift
> let showHome = PublishRelay<Void>()
> //相当于一个“事件通道”，
> //任何地方往里发送事件（accept()），
> //start() 中的订阅者就会响应。
> ```
>
> **(2)** **在** **start()** **中订阅它**
>
> ```swift
> showHome
>     .subscribe(onNext: { [weak self] in
>         self?.navigateToHome()
>     })
> // 当 showHome 收到事件时，就调用 navigateToHome()。
> ```
>
> **(3) ViewModel** **的** **loginSuccess** **是一个** **Signal**
>
> ```swift
> loginSuccess
>     .emit(to: coordinator.showHome)
> // .emit(to:) 是 Signal 的绑定方法，
> // 会把 loginSuccess 的事件“发射”给目标（这里是 PublishRelay）。
> // 所以当 loginSuccess 发出事件时，
> // showHome 收到事件 → navigateToHome() 被调用。
> ```
>
> ```shell
> [ loginSuccess (Signal) ]
>           │
>           ▼
> [ coordinator.showHome (PublishRelay) ]
>           │
>           ▼
> [ navigateToHome() 执行 ]
> ```
>
> | **Trait** | **绑定方法** | **说明**           |
> | --------- | ------------ | ------------------ |
> | Driver    | .drive(to:)  | 适用于 UI 状态绑定 |
> | Signal    | .emit(to:)   | 适用于 UI 事件绑定 |
>
> emit作用：
>
> 把 Signal 的事件 **安全地转发到目标 Relay**，
>
> 并自动保证：
>
> - 在主线程执行；
> - 不会发送 .error；
> - 自动共享订阅。
>
> **.emit(to:) 是 Signal 的绑定方法，用于把「一次性 UI 事件流」发送到目标（如 Relay 或 UI）。**



### 测试（RxTest + RxBlocking）

**RxBlocking（同步等待）**

适合单步测试（阻塞等待结果）。

```swift
let result = try API.fetchUser(id: "1").toBlocking().single()
XCTAssertEqual(result.name, "Tom")
```

> 在 RxSwift 中：
>
> - 网络请求、异步逻辑等都返回 Observable<T>；
> - 但在单元测试中，我们需要**同步拿到结果**来断言。
>
> 这时就可以用 **RxBlocking**（RxSwift 自带的一个测试辅助库）。
>
> **toBlocking()** **是什么？**
>
> toBlocking() 会把一个异步的 Observable 转换为一个 **BlockingObservable**：
>
> ```swift
> let blocking = observable.toBlocking()
> // BlockingObservable 的特点是：会“阻塞”当前线程直到 Observable 发出结果或结束。
> // 也就是说：
> // •	它会等待 Observable 完成；
> // •	然后返回结果（或抛出错误）。
> ```
>
> **single()** **是什么意思？**
>
> .single() 是 BlockingObservable 的方法：
>
> ```swift
> let result = try observable.toBlocking().single()
> // 等待 Observable 发出 唯一一个值，并返回它。
> // 如果没有值或发出多个值，会抛出异常。
> ```
>
> **常见** **BlockingObservable** **方法**
>
> | **方法**      | **含义**                                       |
> | ------------- | ---------------------------------------------- |
> | single()      | 等待一个值返回                                 |
> | first()       | 等待第一个值                                   |
> | last()        | 等待最后一个值                                 |
> | toArray()     | 等待所有值返回                                 |
> | materialize() | 返回完整的事件序列（包含 .completed / .error） |

**RxTest（模拟时间流）**

RxTest 提供 TestScheduler，可以在虚拟时间下测试异步流。

```swift
let scheduler = TestScheduler(initialClock: 0)
let observer = scheduler.createObserver(String.self)

let obs = scheduler.createColdObservable([
    .next(10, "A"),
    .next(20, "B"),
    .completed(30)
])

obs.bind(to: observer).disposed(by: bag)
scheduler.start()

XCTAssertEqual(observer.events, [
    .next(10, "A"),
    .next(20, "B"),
    .completed(30)
])
```

这种“虚拟时间”测试可以精确验证 debounce、throttle、retry 等时间相关逻辑。

> 执行流程
>
> **(1)** **创建虚拟时间调度器**
>
> ```swift
> let scheduler = TestScheduler(initialClock: 0)
> // TestScheduler 是 RxTest 提供的虚拟时间调度器。
> // 它让你不用依赖真实时间（比如 sleep 1 秒），
> // 而是用「虚拟时间单位」来模拟事件的顺序与延迟。
> // 可以理解为一个“时间模拟器”。
> ```
>
> **(2)** **创建一个测试观察者**
>
> ```swift
> let observer = scheduler.createObserver(String.self)
> // 创建一个 TestableObserver，
> // 用来记录事件（包括 .next、.completed、.error）以及发生的“时间点”。
> // 它会把所有接收到的事件保存到一个数组 observer.events 里，供之后断言使用。
> ```
>
> **(3)** **创建一个「冷」可观测序列**
>
> ```swift
> let obs = scheduler.createColdObservable([
>     .next(10, "A"),
>     .next(20, "B"),
>     .completed(30)
> ])
> // createColdObservable 会创建一个 Cold Observable（冷序列）。
> 
> //“冷”意味着：
> //	•	每次订阅都会从头开始；
> //	•	时间是相对订阅时间的（不是绝对时间）。
> // 也就是说：
> //	•	订阅后 10 个时间单位 发 "A"；
> //	•	再过 10 个时间单位 发 "B"；
> //	•	再过 10 个时间单位 完成。
> ```
>
> **(4)** **绑定观察者并开始执行**
>
> ```swift
> obs.bind(to: observer).disposed(by: bag)
> scheduler.start()
> // 1️⃣ .bind(to: observer)
> // 让 observer 订阅 obs，开始监听事件。
> 
> // 2️⃣ scheduler.start()
> // 启动虚拟时间调度器，让所有事件按时间计划发出。
> ```
>
> **(5)** **验证结果**
>
> ```swift
> XCTAssertEqual(observer.events, [
>     .next(10, "A"),
>     .next(20, "B"),
>     .completed(30)
> ])
> // 测试框架会比较：
> //	•	实际收到的事件序列（observer.events）
> //	•	与预期序列是否一致。
> // 如果一致，测试通过。
> ```
>
> | **虚拟时间** | **事件**   | **说明**          |
> | ------------ | ---------- | ----------------- |
> | 0            | 订阅开始   | scheduler.start() |
> | 10           | .next("A") | 发送 A            |
> | 20           | .next("B") | 发送 B            |
> | 30           | .completed | 完成序列          |
> | >30          | 停止       | 测试结束          |
>
> **常用** **RxTest API**
>
> | **方法 / 类型**              | **功能**               |
> | ---------------------------- | ---------------------- |
> | TestScheduler(initialClock:) | 创建虚拟时间调度器     |
> | createColdObservable()       | 创建冷序列（相对时间） |
> | createHotObservable()        | 创建热序列（绝对时间） |
> | createObserver(Type.self)    | 创建测试观察者         |
> | scheduler.start()            | 启动事件流             |
> | .events                      | 记录事件（用于断言）   |
>
> | **名称**             | **用法**       | **记忆**     |
> | -------------------- | -------------- | ------------ |
> | TestScheduler        | 模拟时间       | 时间沙盒     |
> | createColdObservable | 构造测试输入流 | 虚拟事件表   |
> | createObserver       | 监听并记录结果 | 事件日志     |
> | scheduler.start()    | 启动执行       | 播放时间线   |
> | observer.events      | 实际输出       | 比对预期结果 |

总结：

**响应式架构不是在 UI 层用 Rx，而是让整个系统“以流为中心”，用流去管理事件、状态、请求、导航。**



## **进阶主题（Advanced Topics）**

### 错误恢复机制（Error Handling & Recovery）

在 Rx 中，**错误（error）会终止整个流**。

因此必须显式处理，否则 Observable 会直接停止，不再发送任何事件。

**catchError/catchErrorJustReturn**

作用：捕获错误并提供一个“替代流”继续执行。

```swift
API.fetchUser()
    .catchError { error in
        // 捕获错误，返回另一个流（比如备用请求）
        return API.fetchGuestUser()
    }
    .subscribe(onNext: { print($0) })
```

或者简单返回默认值：

```swift
API.fetchUser()
    .catchErrorJustReturn(User.guest)
```

优点：防止 error 让整个 UI 流中断（常用于 UI 层）。

**retry/retryWhen**

作用：自动重试失败的流（常用于网络断线重连）。

```swift
API.fetchUser()
    .retry(3) // 最多重试 3 次
    .subscribe(...)
```

retryWhen 更灵活，可基于错误类型或延迟策略控制重试。

```swift
API.fetchUser()
    .retryWhen { errorStream in
        errorStream
            .enumerated()
            .flatMap { attempt, error -> Observable<Int> in
                if attempt >= 3 { return Observable.error(error) } // 超过3次停止
                print("重试第 \(attempt+1) 次...")
                return Observable<Int>.timer(.seconds(2), scheduler: MainScheduler.instance)
            }
    }
```

> **errorStream**
>
> 这是 **上游 Observable 抛出的 error 组成的流**
>
> **返回的** **Observable** **决定行为**
>
> | **返回**   | **含义**                           |
> | ---------- | ---------------------------------- |
> | .next      | **触发 retry**                     |
> | .completed | **停止 retry，结束序列**           |
> | .error     | **停止 retry，并向下游传递 error** |
>
> retryWhen 的本质是**把 error 转换成一个“是否 retry 的信号流”**
>
> ```shell
> next → retry
> completed → stop
> error → stop + throw
> ```

**断线重连模型（Network Reconnect Pattern）**

```swift
networkStatusObservable // 网络状态流（online/offline）
    .filter { $0 == .online }
    .flatMapLatest { _ in API.fetchData().retryWhen(...) }
```

当网络恢复时自动重试请求。



### 背压与性能控制（Backpressure & Flow Control）

**背压（Backpressure）是什么？**

指**上游事件发送速度 > 下游消费速度** 时的性能与内存问题。

虽然 RxSwift 没有像 RxJava Flowable 那样的背压策略，但可以用**限流操作符**控制流速。

**throttle（节流）**

在一个时间窗口内，只取第一个事件（忽略后续）。

```swift
button.rx.tap
    .throttle(.milliseconds(500), scheduler: MainScheduler.instance)
    .subscribe(onNext: { print("点击") })
```

防止重复点击、重复请求。

**debounce（防抖）**

延迟一段时间后才发出最后一个事件（适合搜索框输入）。

```swift
textField.rx.text.orEmpty
    .debounce(.milliseconds(300), scheduler: MainScheduler.instance)
    .distinctUntilChanged()
    .flatMapLatest { query in API.search(query) }
```

用户停止输入 300ms 后再发请求，减少无用请求。

**控制并发任务数量（flatMap + 信号量）**

例如要同时并发最多 3 个请求：

```swift
let semaphore = DispatchSemaphore(value: 3)

Observable.from(urls)
    .flatMap { url -> Observable<Data> in
        return Observable.create { observer in
            semaphore.wait() // 占用名额
            API.fetch(url)
                .subscribe(
                    onNext: { observer.onNext($0) },
                    onError: { observer.onError($0) },
                    onCompleted: {
                        semaphore.signal() // 释放名额
                        observer.onCompleted()
                    })
        }
    }
```

控制上游发出的并发任务数，防止 CPU / 内存过载。



### 多线程与并发流（Concurrency & Scheduling）

- subscribe(on:)：指定“数据产生”线程（上游执行线程）
- observe(on:)：指定“数据消费”线程（下游执行线程）

```swift
Observable.just("heavy")
    .subscribe(on: ConcurrentDispatchQueueScheduler(qos: .background)) // 在后台生成
    .map { heavyCompute($0) }
    .observe(on: MainScheduler.instance) // 在主线程更新 UI
    .subscribe(onNext: { print($0) })
```

常见用法：网络请求/计算在后台 → UI 更新在主线程。



**并发控制（flatMap）**

默认 flatMap 是并行的，每个内部流都会并发执行。使用 concatMap 或 serialScheduler 可以强制串行执行。

```swift
Observable.of(task1, task2, task3)
    .concatMap { $0.run() } // 串行执行
```

 

### 副作用管理（Side Effects Management

**什么是副作用？**

任何会影响外部世界或状态的操作：网络请求、写文件、更新 UI、打印日志等。

**do(onNext:)**

在流中插入副作用，但不改变数据。

```swift
API.fetchUser()
    .do(onSubscribe: { print("开始请求") },
        onNext: { print("拿到结果 \($0)") },
        onDispose: { print("结束") })
    .subscribe(...)
```

**适合打印日志、埋点，不破坏数据流。**

> **do 用来“观察事件”并执行副作用（side effects），但不会改变事件流本身。**
>
> 它只是在事件经过时 **插入一些操作**（例如 log、loading、统计、埋点等）。
>
> do 可以监听 Observable 的 **完整生命周期**：
>
> ```swift
> .do(
>     onSubscribe: {},
>     onNext: {},
>     onError: {},
>     onCompleted: {},
>     onDispose: {}
> )
> ```
>
> | **回调**    | **触发时机**        |
> | ----------- | ------------------- |
> | onSubscribe | subscribe 时        |
> | onNext      | 每次 next           |
> | onError     | error               |
> | onCompleted | completed           |
> | onDispose   | subscription 被释放 |
>
> **使用场景**
>
> 显示loading
>
> ```swift
> API.fetchUser()
>     .do(
>         onSubscribe: { showLoading() },
>         onDispose: { hideLoading() }
>     )
>     .subscribe(
>         onNext: { user in
>             print(user)
>         }
>     )
> ```
>
> 打印网络日志
>
> ```swift
> API.fetchUser()
>     .do(onNext: { user in
>         print("API返回:", user)
>     })
>     .subscribe(...)
> ```
>
> 埋点统计
>
> ```swiift
> loginButton.tap
>     .do(onNext: {
>         Analytics.log("login_button_clicked")
>     })
>     .subscribe(...)
> ```
>
> 调试Rx链
>
> ```swift
> textField.rx.text.orEmpty
>     .do(onNext: { print("输入:", $0) })
>     .debounce(.milliseconds(300), scheduler: MainScheduler.instance)
>     .do(onNext: { print("debounce后:", $0) })
>     .subscribe(...)
> // 输入
> a
> ab
> abc
> // 输出
> 输入: a
> 输入: ab
> 输入: abc
> debounce后: abc
> ```
>
> **do和debug区别**
>
> do 是一个 **拦截事件但不改变事件流的操作符**。
>
> 特点：
>
> - 可以监听 Observable 生命周期
> - 可以执行自定义逻辑
> - 不会改变数据
> - 常用于 **loading、日志、埋点**
>
> debug 是 **专门用来调试 Rx 链的日志工具**。
>
> 它会自动打印：
>
> - subscribe
> - next
> - error
> - completed
> - dispose
>
> ```swift
> Observable.of(1,2)
>     .debug("Test")
>     .subscribe()
> // 输出
> Test -> subscribed
> Test -> Event next(1)
> Test -> Event next(2)
> Test -> Event completed
> Test -> isDisposed
> ```
>
> | **对比**             | **do**              | **debug** |
> | -------------------- | ------------------- | --------- |
> | 是否可执行自定义代码 | ✅                   | ❌         |
> | 是否自动打印事件     | ❌                   | ✅         |
> | 用途                 | 副作用              | 调试      |
> | 常见场景             | loading / analytics | Rx 链调试 |



**share() / replay()**

防止**重复订阅触发多次副作用**（如重复网络请求）。

```swift
let shared = API.fetchUser()
    .asObservable()
    .share(replay: 1, scope: .whileConnected)

shared.subscribe(onNext: { print("A: \($0)") }).disposed(by: bag)
shared.subscribe(onNext: { print("B: \($0)") }).disposed(by: bag)
```

实际上只请求一次 API（共享结果）。



**publish() + connect()**

更低层的控制（ConnectableObservable），允许你显式决定何时开始发送事件。

```swift
let connectable = API.fetchUser().publish()
connectable.connect() // 手动启动
```



### 内存优化与调试（Memory & Debugging）

**内存泄漏检测**

- 使用 **Instruments → Leaks / Allocations** 查看是否存在 Rx 对象未释放。

- 重点关注：

  - ViewController / ViewModel 是否互相引用；
  - 闭包中未使用 [weak self]；
  - TableViewCell 中是否在 prepareForReuse() 重置 disposeBag。

  

**调试流**

debug() 操作符可以打印事件的生命周期：

```swift
someObservable
    .debug("UserStream")
    .subscribe()
```

输出示例：

```shell
UserStream -> subscribed
UserStream -> Event next(John)
UserStream -> Event completed
UserStream -> disposed
```

快速确认事件是否发出、何时完成、是否被释放。



## 测试与调试（Testing & Debugging）

### RxTest — 响应式单元测试的核心工具

**作用：**

RxTest 让你能在**虚拟时间**下测试异步流，不需要真的等待。

**TestScheduler**

TestScheduler 是一个虚拟时间的调度器，允许你：

- 精确控制事件发出时机；
- 快速执行时间相关操作（debounce, delay, throttle 等）；
- 验证事件的顺序与内容。

基本用法

```swift
import RxSwift
import RxTest
import XCTest

class ExampleTests: XCTestCase {
    var scheduler: TestScheduler!
    var observer: TestableObserver<String>!
    var disposeBag: DisposeBag!
    
    override func setUp() {
        super.setUp()
        scheduler = TestScheduler(initialClock: 0)
        observer = scheduler.createObserver(String.self)
        disposeBag = DisposeBag()
    }

    func testExample() {
        // 创建一个冷 Observable（cold）
        let observable = scheduler.createColdObservable([
            .next(10, "A"),
            .next(20, "B"),
            .completed(30)
        ])

        observable.bind(to: observer).disposed(by: disposeBag)
        
        // 启动虚拟时间（相当于让流运行）
        scheduler.start()
        
        // 验证结果
        XCTAssertEqual(observer.events, [
            .next(10, "A"),
            .next(20, "B"),
            .completed(30)
        ])
    }
}
```



**Cold vs Hot Observable**

**Cold Observable**：每个订阅者都会独立收到完整事件序列（如 Observable.of(1,2,3)）。

**Hot Observable**：事件持续发出，订阅时只能接收当前之后的事件（如 UI 事件、PublishSubject）。

RxTest 提供：

- createColdObservable()
- createHotObservable()

```swift
let hot = scheduler.createHotObservable([
    .next(10, "X"),
    .next(20, "Y"),
    .completed(30)
])
```



**Marble Diagrams 思维（弹珠图）**

Marble Diagram 是 Rx 思维模型的图形化表达。

```shell
时间轴 →
--A--B----C--|
```

表示 Observable 按时间发出事件。

map、filter 等操作符的效果也能直观展示：

例如：

```swift
--A--B--C--|     (source)
map(+1)
--B--C--D--|     (output)
```

Marble 思维在 RxTest 中就是通过「时间戳」来验证这些弹珠何时出现。

**实战意义：**

- 可以精确测试复杂的时间依赖逻辑（防抖、节流、重试等）。
- 不再依赖 sleep() 或异步回调，测试执行迅速且确定性强。



### RxBlocking — 同步测试简化工具

**概念：**

RxBlocking 把异步流“阻塞”成同步调用，方便快速断言。

适合测试：

- 单次返回的流（Single、Maybe）
- 简单 Observable（发出有限个元素后结束）

```swift
import RxBlocking
import RxSwift
import XCTest

func testBlocking() throws {
    let observable = Observable.of(1, 2, 3)
    let result = try observable.toBlocking().toArray()
    XCTAssertEqual(result, [1, 2, 3])
}

func testSingleValue() throws {
    let single = Single.just("Hello")
    let value = try single.toBlocking().single()
    XCTAssertEqual(value, "Hello")
}
```

优点：

- 编写简单；
- 适合快速验证计算、转换、网络请求结果。

⚠️ 注意：

- 不适合无限流（例如 .interval），否则测试会卡死。



### 调试技巧（Debugging Techniques）

Rx 的流式代码调试困难，以下三种方式是“日常必备”。

**.debug("Tag")**

打印整个流的生命周期：

```swift
API.fetchUser()
    .debug("UserRequest")
    .subscribe()
```

输出示例：

```swift
UserRequest -> subscribed
UserRequest -> Event next(User)
UserRequest -> Event completed
UserRequest -> isDisposed
```

快速了解事件顺序与订阅释放情况。



**do()插入断点或打印**

在不改变数据的情况下插入副作用：

```swift
someObservable
    .do(onNext: { print("值: \($0)") },
        onDispose: { print("释放流") })
    .subscribe()
```

可以在 onNext 里下断点（breakpoint）调试流状态。



**.print()（简易版调试）**

```swift
someObservable
    .print("MyStream")
    .subscribe()
```

输出事件与线程信息，比 debug() 更简短。

**调试建议：**

- 在关键流（尤其是网络层、合并流）加 .debug()。
- 使用 do(onNext:) 打印中间数据。
- 结合 RxSwift.Resources.total 统计资源数量（内存泄漏排查）。



### 内存检测与生命周期调试

**weak self 防循环引用**

```swift
observable
    .subscribe(onNext: { [weak self] value in
        self?.updateUI(value)
    })
    .disposed(by: bag)
```

使用 [weak self] 或 .withUnretained(self)（RxSwift 6+）

```swift
observable
    .withUnretained(self)
    .subscribe(onNext: { owner, value in
        owner.updateUI(value)
    })
    .disposed(by: bag)
```



**DisposeBag 生命周期分析**

- 每个 VC/VM 都应有自己的 DisposeBag；
- Cell 重用时需要重置：

```swift
class MyCell: UITableViewCell {
    var bag = DisposeBag()
    override func prepareForReuse() {
        bag = DisposeBag()
    }
}
```

- 如果 DisposeBag 没有释放，说明该对象仍被持有（泄漏信号）。



**Instruments 检测**

- 打开 Xcode → Product → Profile → “Leaks”
- 在 Instruments 中查看 RxSwift._AnyObserver、Disposable 等对象是否一直存在。
- 结合 .debug() 输出判断订阅是否被释放。



### 持续集成（CI）环境下测试 Rx

**目标：**

让你的 Rx 测试可以在 CI（例如 GitHub Actions、Bitrise、Jenkins）中自动运行。

**实践建议：**

1. **测试独立、确定性**
   - 不依赖真实网络。使用 stub 或 mock Observable。

```swift
func mockUserAPI() -> Observable<User> {
    return Observable.just(User(id: "1", name: "Mock"))
}
```

1. **避免 sleep()**
   - 使用 TestScheduler 或 RxBlocking 等待确定事件。
2. **确保测试清理**
   - 每个测试用例 setUp() 时创建新 DisposeBag。
3. **可视化输出**
   - 在 CI 控制台中打印 .debug("Test") 输出，方便排查失败原因。
4. **结合 XCTestExpectation（可选）**
   - 对复杂异步流仍可用原生 XCTestExpectation 等待订阅完成。
5. **结合 XCTestExpectation（可选）**
   - 对复杂异步流仍可用原生 XCTestExpectation 等待订阅完成。

```swift
func testLoginFlow() {
    let expectation = XCTestExpectation(description: "login finished")

    viewModel.output.loginResult
        .emit(onNext: { _ in expectation.fulfill() })
        .disposed(by: bag)

    viewModel.input.loginTap.accept(())
    wait(for: [expectation], timeout: 2.0)
}
```

> **Stub = 用“假的实现”替代真实依赖，并返回固定结果。**
>
> 这样测试就不会依赖：
>
> - 网络
> - 数据库
> - 第三方 API
> - 不稳定环境
>
> 从而保证 **测试稳定、快速、可重复（deterministic）**。
>
> ```swift
> protocol UserAPI {
>     func fetchUser() -> Observable<User>
> }
> 
> class RealUserAPI: UserAPI {
>     func fetchUser() -> Observable<User> {
>         // 网络请求
>     }
> }
> 
> class UserViewModel {
>     private let api: UserAPI
> 
>     init(api: UserAPI) {
>         self.api = api
>     }
> }
> ```
>
> ```swift
> class StubUserAPI: UserAPI {
>     func fetchUser() -> Observable<User> {
>         Observable.just(User(id: "1", name: "StubUser"))
>     }
> }
> ```
>
> ```swift
> let vm = UserViewModel(api: StubUserAPI())
> ```
>
> **Stub vs Mock** **区别**
>
> | **类型** | **作用**           |
> | -------- | ------------------ |
> | Stub     | 返回固定数据       |
> | Mock     | 验证方法是否被调用 |
>
> ```swift
> class StubAPI: API {
>     func fetchUser() -> Observable<User> {
>         return Observable.just(User(id: "1", name: "Tom"))
>     }
> }
> // 只负责 返回数据。
> 
> class MockAPI: API {
>     var fetchCalled = false
>     func fetchUser() -> Observable<User> {
>         fetchCalled = true
>         return Observable.just(User(id: "1", name: "Tom"))
>     }
> }
> 
> XCTAssertTrue(api.fetchCalled)
> // Mock 用来 验证行为。
> ```
>
> XCTestExpectation 是 **XCTest 中用于测试异步代码的机制**。
>
> **XCTestExpectation = 告诉测试框架 “我要等待某个异步事件发生”**
>
> ```swift
> func testAsync() {
> 
>     let expectation = expectation(description: "等待异步任务")
> 
>     var result: Int?
> 
>     DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
>         result = 10
>         expectation.fulfill()
>     }
> 
>     wait(for: [expectation], timeout: 2)
> 
>     XCTAssertEqual(result, 10)
> }
> 
> // 执行流程
> 创建 expectation
> ↓
> 启动异步任务
> ↓
> wait()
> ↓
> 异步完成
> ↓
> fulfill()
> ↓
> 测试继续
> ```
>
> ```swift
> wait(for: expectations, timeout: T)
> // 最多等待 T 秒
> // 直到所有 expectation fulfill
> // 否则测试失败
> ```
