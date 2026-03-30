# Object-C

## **Objective-C 面向对象底层机制**

**Objective-C 对象和类的基本结构**

在 Objective-C 中，每个对象都是 **一个指向类的指针**。

- 当你调用一个对象的方法时，runtime 会通过对象的 **isa** **指针** 找到它所属的类（Class）。
- 类本身也是一个对象，它的 isa 指针指向 **元类（Meta-class）**。
- 元类里存储了类方法（+ 方法），类本身存储实例方法（- 方法）。

简单图示：

```shell
实例对象 -> isa -> 类 -> isa -> 元类 -> isa -> 根元类 (NSObject metaclass)
```

**核心概念**

| **名称**                      | **作用**         | **存储内容**                         |
| ----------------------------- | ---------------- | ------------------------------------ |
| **对象实例（instance）**      | 真正的对象       | 成员变量（ivars）                    |
| **Class（类）**               | 定义对象行为     | 实例方法列表、成员变量列表、属性列表 |
| **Meta-class（元类）**        | 定义类方法行为   | 类方法列表（+ 方法）                 |
| **isa 指针**                  | 指向对象所属类   | 对象 -> 类；类 -> 元类               |
| **方法列表（method list）**   | 存储方法信息     | SEL、IMP（函数实现指针）、类型编码   |
| **成员变量列表（ivar list）** | 存储成员变量信息 | 变量名、偏移量、                     |

**方法调用过程（消息发送）**

```shell
# 当你写：
[myObj doSomething];
```

```shell
# 背后实际发生的是：
objc_msgSend(myObj, @selector(doSomething));
```

**流程：**

1. runtime 通过对象 myObj->isa 找到类 MyClass。
2. 在 MyClass 的 **方法列表** 中查找 doSomething 的 **SEL**。
3. 如果找到，则取出对应的 **IMP**（函数指针），执行。
4. 如果找不到，则沿继承链向上查找父类。
5. 如果还是找不到，则调用 forwarding 机制或报错。

**元类和类方法调用**

```shell
[MyClass doSomethingClassMethod];
```

- MyClass 本身是一个对象（Class）。
- MyClass->isa 指向元类（Meta-class）。
- runtime 在 **元类方法列表** 中查找 doSomethingClassMethod。

**成员变量查找**

```shell
# 当你访问成员变量：
myObj->_name = @"Matt";
```

- runtime 会通过对象实例直接访问内存偏移量（ivar offset）。
- 偏移量由 **成员变量列表** 提供。

**图示说明**

```shell
+----------------------+
|  Person instance p   |  <-- 对象实例
|----------------------|
| _name                |  <-- 成员变量
| _age                 |
| isa ----------------> Person class
+----------------------+

         |
         v

+----------------------+
|      Person class    |  <-- 类对象
|----------------------|
| 方法列表: sayHello   |  <-- 实例方法
| 成员变量列表: _name,_age |
| isa ----------------> Person meta-class
+----------------------+

         |
         v

+----------------------+
|  Person meta-class   |  <-- 元类
|----------------------|
| 方法列表: classInfo  |  <-- 类方法
| isa ----------------> NSObject meta-class
+----------------------+
```

**总结**

- **对象调用实例方法** → isa 指向类 → 查找方法列表 → 执行 IMP
- **类调用类方法** → isa 指向元类 → 查找方法列表 → 执行 IMP
- **成员变量访问** → 直接通过对象实例 + 成员变量列表中的偏移量访问内存
- **方法列表 / 成员变量列表** 可以通过 runtime 函数查看
- **元类** 是类对象的类，用来存储类方法



## 	AppDelegate和SceneDelegate中应用的运行逻辑和流程？

**一、iOS 13 之前的逻辑（回顾）**

在 iOS 13 之前，应用只有一个主窗口 (UIWindow)。

生命周期方法都在 **AppDelegate** 中，比如：

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
- (void)applicationDidEnterBackground:(UIApplication *)application;
- (void)applicationWillEnterForeground:(UIApplication *)application;
- (void)applicationDidBecomeActive:(UIApplication *)application;
- (void)applicationWillTerminate:(UIApplication *)application;
```

在 didFinishLaunchingWithOptions: 中通常会：

- 创建 UIWindow；
- 设置根控制器；
- 调用 [window makeKeyAndVisible]。

**二、iOS 13 之后的新逻辑**

从 iOS 13 开始，苹果引入了 UIScene 和 UISceneDelegate 来支持**多窗口 (multi-scene)**，尤其是在 **iPadOS** 上（允许一个 App 同时开启多个独立窗口）。

因此：

- **AppDelegate** 负责全局生命周期（如启动、通知、后台任务等）；
- **SceneDelegate** 负责每个窗口场景的生命周期（UI 部分）。

**运行流程概览**

以下是一个应用从启动到展示界面的完整逻辑流程：

```shell
App Launch
│
├── AppDelegate: application:didFinishLaunchingWithOptions:
│       ↓
│       （系统决定要创建一个或多个 Scene）
│
├── AppDelegate: configurationForConnectingSceneSession:
│       ↓
│       返回一个 UISceneConfiguration（指定要使用的 SceneDelegate）
│
├── SceneDelegate: scene:willConnectToSession:options:
│       ↓
│       创建 UIWindow，并设置根视图控制器
│
└── SceneDelegate: sceneDidBecomeActive:
        ↓
        应用界面显示并开始响应用户交互
```

**AppDelegate 中主要职责（iOS13+）**

在 iOS13 之后，AppDelegate 主要负责非 UI 层面的全局管理：

````objective-c
// 应用完成启动
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    NSLog(@"App 启动完成");
    return YES;
}

// 创建新 Scene 时调用（多窗口支持）
- (UISceneConfiguration *)application:(UIApplication *)application
configurationForConnectingSceneSession:(UISceneSession *)connectingSceneSession
options:(UISceneConnectionOptions *)options {
    return [[UISceneConfiguration alloc] initWithName:@"Default Configuration"
                                          sessionRole:connectingSceneSession.role];
}

// 场景被丢弃时调用
- (void)application:(UIApplication *)application didDiscardSceneSessions:(NSSet<UISceneSession *> *)sceneSessions {
    NSLog(@"Scene 被丢弃");
}
````

**其他全局事件仍在 AppDelegate 中处理，例如：**

- 推送通知注册与回调；
- 后台任务处理；
- URL Scheme / Universal Links；
- App 生命周期（如后台恢复，但只针对整体 App）。

\**五、SceneDelegate 中主要职责（iOS13+）**

SceneDelegate 负责具体的 UI 场景（窗口）管理：

```objective-c
- (void)scene:(UIScene *)scene willConnectToSession:(UISceneSession *)session
      options:(UISceneConnectionOptions *)connectionOptions {
    UIWindowScene *windowScene = (UIWindowScene *)scene;
    self.window = [[UIWindow alloc] initWithWindowScene:windowScene];
    self.window.rootViewController = [ViewController new];
    [self.window makeKeyAndVisible];
}

// 场景进入前台
- (void)sceneWillEnterForeground:(UIScene *)scene {
    NSLog(@"Scene 将进入前台");
}

// 场景变为活跃状态
- (void)sceneDidBecomeActive:(UIScene *)scene {
    NSLog(@"Scene 已变为活跃");
}

// 场景进入后台
- (void)sceneDidEnterBackground:(UIScene *)scene {
    NSLog(@"Scene 已进入后台");
}
```

**总结职责：**

- 创建和管理 UIWindow；
- 管理场景的前后台切换；
- 管理与该窗口相关的 UI 生命周期；
- 每个 scene 对应一个窗口实例。

**六、运行示意图**

```shell
          ┌──────────────────────────────────────┐
          │              AppDelegate             │
          │--------------------------------------│
          │ didFinishLaunchingWithOptions        │
          │ configurationForConnectingSceneSession│
          │ didDiscardSceneSessions              │
          └──────────────┬───────────────────────┘
                         │
                         ▼
          ┌──────────────────────────────────────┐
          │              SceneDelegate            │
          │--------------------------------------│
          │ scene:willConnectToSession            │
          │ sceneWillEnterForeground              │
          │ sceneDidBecomeActive                  │
          │ sceneWillResignActive                 │
          │ sceneDidEnterBackground               │
          └──────────────────────────────────────┘
```



##  什么是自动变量？为什么block内部修改外部变量要加__block?

**自动变量**（Auto Variable）是指在函数或方法内部声明的局部变量，它们存储在栈内存中，生命周期与所在的作用域相同。

**自动变量的关键点：**

- **定义**：函数内部的局部变量
- **存储**：栈内存
- **生命周期**：与作用域相同
- **特点**：自动创建和销毁

**__block 修饰符的关键点：**

- **作用**：允许 Block 修改外部自动变量
- **原理**：将变量从栈移动到堆，通过指针访问
- **使用场景**：需要在 Block 内部修改变量值时

**例外**：静态变量、全局变量不需要 __block



## object中类的方法执行顺序，分类，类方法等执行顺序

**1. Objective-C 类加载与方法执行顺序总览**

在 Objective-C 中，类的加载与方法调用顺序主要分为以下阶段：

（从程序启动到类实例方法执行）

1. **+load 方法**
   - 程序启动时自动调用（无需手动触发）。
   - **调用时机早于 main()**。
   - **类与分类（Category）都可能实现 +load 方法**。
   - 执行顺序：
     1. 先调用类的 +load；
     2. 再调用该类所有分类（Category）的 +load；
     3. 如果存在继承关系，**父类的 +load 优先于子类调用**；
     4. **每个 +load 方法仅执行一次**；
   - **调用特征**：直接由运行时调用，**不参与消息发送机制**。

```objective-c
+ (void)load {
    NSLog(@"ClassName +load");
}
```

**2. +initialize 方法**

1. 当类**第一次接收到消息**时（例如创建实例或调用类方法）自动调用。

2. 只会调用一次，除非手动再次调用 super。

3. 执行顺序：

   - 先调用父类的 +initialize；
   - 再调用子类的 +initialize；
   - 如果分类实现了 +initialize，会覆盖原类的实现；

4. 与 +load 不同，**+initialize 是通过消息机制调用的**。

   即如果你在 +initialize 中调用 super，会执行父类的实现。

```objective-c
+ (void)initialize {
    NSLog(@"ClassName +initialize");
}
```

**3. 类加载与内存结构执行顺序总结**

| **阶段**   | **方法**             | **调用时机**   | **调用对象** | **调用机制**           |
| ---------- | -------------------- | -------------- | ------------ | ---------------------- |
| 程序启动   | +load                | 程序加载时     | 类与分类     | 直接调用（非消息机制） |
| 首次使用类 | +initialize          | 第一次发送消息 | 类（或子类） | 消息机制               |
| 创建实例   | -init / -initWithXXX | alloc 后       | 实例对象     | 消息机制               |

**4. +load 与 +initialize 对比**

| **对比项** | **+load**                            | **+initialize**          |
| ---------- | ------------------------------------ | ------------------------ |
| 调用时机   | 程序加载时（main 前）                | 第一次使用类时           |
| 调用方式   | 直接调用（无消息发送）               | 通过消息机制调用         |
| 调用顺序   | 父类 → 子类 → 分类                   | 父类 → 子类（可被覆盖）  |
| 调用次数   | 每类/分类各执行一次                  | 每类通常执行一次         |
| 常见用途   | 方法交换（Method Swizzling）、类注册 | 延迟初始化、静态变量设置 |

**5. 分类 (Category) 加载顺序**

1. 分类可以实现自己的 +load 方法；
2. **分类的 +load 在原类 +load 之后执行**；
3. 多个分类的加载顺序由编译顺序（Mach-O 文件中的 section 顺序）决定；
4. 如果分类中实现了同名方法（如 +initialize），会覆盖原类方法；
5. 分类中添加的方法会在运行时动态合并进类的 Method List。

**6. 类方法与实例方法执行顺序**

1. **类方法 (Class Method)**
   - 使用 + 定义，如 + (void)methodName;
   - 发送消息到类对象（[ClassName methodName]）；
   - 存储在 **Meta-Class**（元类）中；
   - 调用顺序：父类 → 子类 → 分类（覆盖同名方法）。
2. **实例方法 (Instance Method)**
   - 使用 - 定义，如 - (void)methodName;
   - 发送消息到实例对象（[object methodName]）；
   - 存储在 **类对象（Class Object）**中；
   - 调用顺序：当前类 → 父类查找 → 分类覆盖优先。

**7. 对象创建与方法调用整体顺序**

以创建一个对象为例，执行顺序如下：

1. **程序启动：**
   - 调用所有类与分类的 +load 方法（main 之前）。
2. **首次使用类：**
   - 调用该类的 +initialize 方法（如果尚未调用过）。
3. **分配内存：**
   - 调用 [Class alloc]，分配对象空间；
4. **初始化对象：**
   - 调用 -init（或其他初始化方法）；
5. **对象使用中：**
   - 调用实例方法；
6. **程序退出时：**
   - 若手动管理内存，调用 -dealloc 释放资源。

**8. 类与分类的加载顺序整体总结**

1. **父类 +load**
2. **子类 +load**
3. **父类分类 +load**
4. **子类分类 +load**
5. **main() 开始执行**
6. **首次使用类 → +initialize**
7. **alloc / init / 实例方法调用**

**9. 小结**

- **+load → 程序启动即执行，主要用于方法交换或运行时注册；**
- **+initialize → 类首次使用时执行，用于延迟初始化；**
- **分类的 +load 在类之后执行，但分类可以覆盖类的 +initialize；**
- **类方法存在于元类中，实例方法存在于类对象中；**
- **方法查找顺序始终是：分类 → 当前类 → 父类 → NSObject。**



## 	当前类遵循了某个协议，但是没有实现协议，而是在其分类中实现协议是常用的模式

**1. 这种模式的定义与背景**

在 Objective-C 中，一个 **类（Class）** 可以在主声明中声明遵循某个协议（@protocol），

但 **不一定在主类中实现**协议方法，而是选择在 **分类（Category）** 中去实现这些方法。

这种设计在实际开发中非常常见，被称为：

👉 **“通过 Category 实现协议（Protocol）的方法分离模式”** 或 **“协议解耦实现”**。

**2. 常见使用场景**

1. **解耦与代码组织**
   - 让主类只关心核心业务逻辑；
   - 将不同功能模块（如网络回调、UI代理、数据源）放入不同分类中；
   - 方便多人协作、模块化开发。
   - 示例：

```objc
@interface MyViewController : UIViewController <UITableViewDelegate, UITableViewDataSource>
@end

// 在 Category 中实现协议方法
@implementation MyViewController (TableView)
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return 10;
}
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    // ...
}
@end
```

**3. 运行机制与可行性**

1. 在编译阶段，编译器不会强制要求主类必须实现协议中的所有方法。
   - 只要类声明遵循协议，编译器会在分类中继续查找对应实现；
   - 只要最终类（包括分类）能提供协议要求的方法，运行时即认为协议被完整实现。
2. 在运行时，分类的方法会被**合并进原类的 method list**，因此在消息发送阶段是完全等效的。

```objc
// 协议定义
@protocol MyProtocol <NSObject>
- (void)doSomething;
@end

// 类声明遵循协议，但主类不实现
@interface MyClass : NSObject <MyProtocol>
@end

// 分类中实现协议
@implementation MyClass (ProtocolImpl)
- (void)doSomething {
    NSLog(@"Protocol implemented in Category");
}
@end
```

**4. 分类实现协议的优缺点**

| **优点**                   | **缺点**                                   |
| -------------------------- | ------------------------------------------ |
| 模块化清晰，将不同功能分区 | 逻辑分散，难以快速定位完整实现             |
| 减少主类臃肿，提高可读性   | 若多个分类实现同名方法，后编译的会覆盖前者 |
| 便于多人协作、代码隔离     | 编译器警告可能被隐藏（无法提示未实现协议） |

**5. 实际开发中常见的应用模式**

1. **按功能拆分控制器分类**
   - UIViewController+UITableView.m
   - UIViewController+CollectionView.m
   - 用于实现不同协议的代理回调。
2. **第三方库中的扩展模式**
   - 如 AFNetworking 中的 NSURLSessionTask (AFNetworking) 分类；
   - 协议方法通过分类实现，增强系统类行为。
3. **协议适配层**
   - 某个基础类声明遵循协议；
   - 实际实现分布在不同分类中，按模块加载；
   - 常用于插件化架构。

**6. 小结**

1. **类可以声明遵循协议，但不必在主类中实现；分类中实现同样有效。**
2. **这是 Objective-C 常见的模块化模式，用于分离功能与解耦代码。**
3. **核心原理：分类方法在运行时被合并进类的 method list，因此协议检查仍然通过。**
4. **推荐：主类仅声明遵循协议，在分类中按功能模块实现对应方法，提高可维护性与结构清晰度。**



## super和self的区别

```objc
@implementation Son : Father
- (id)init {
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

它考察的是 **Objective-C 中 self 与 super 的本质区别**，

以及**消息发送机制（objc_msgSend）**。

**1. self 和 super 的根本区别**

| **关键字** | **代表的含义**                       | **方法查找起点**             | **实际调用者**     |
| ---------- | ------------------------------------ | ---------------------------- | ------------------ |
| **self**   | 当前对象（实例本身）                 | 当前对象的类（isa 指向的类） | 当前对象           |
| **super**  | 当前对象的父类调用入口（编译器指令） | **父类的方法列表**           | **仍然是当前对象** |

注意：

- super 不是一个指针，也不是另一个对象。

- 它只是告诉编译器：**从父类的方法列表开始查找方法实现**，

  但**消息的接收者仍然是** **self**。

也就是说：

```objc
[super class];
```

在底层会被编译为：

```objc
objc_msgSendSuper({ self, [Son superclass] }, @selector(class))
```

而消息发送的目标对象仍然是 self（当前实例）。

**3. 为什么两个都是 Son？**

1. NSStringFromClass([self class])
   - [self class] → 发送消息给当前对象（Son 实例）；
   - 系统方法 -[NSObject class] 的实现是：返回 object_getClass(self)；
   - 所以返回 Son。
2. NSStringFromClass([super class])
   - [super class] 看似“从父类调用”，但实际上：
     - **消息仍然发给同一个对象（****self****）**；
     - 唯一区别是：**查找方法实现时，从父类开始找**；
   - 但 -[NSObject class] 没被重写（无论从哪里找，都是同一个实现）；
   - 该实现内部仍然根据 self 的真实类型返回类对象；
   - 所以依旧返回 Son。

**4.小结**

1. self 是对象自身，super 只是告诉编译器“去父类找方法”。
2. super 并不改变消息的接收者（仍然是 self）。
3. 因此 [self class] 与 [super class] 都会输出同样的结果。



## _objc_msgForward 是如何一步步让运行时“挽救”一个找不到的 selector 调用

**1. 背景：Objective-C 是基于消息发送机制的**

在 OC 中，方法调用：

```objc
[obj doSomething];
```

其实会被编译成：

```objc
objc_msgSend(obj, @selector(doSomething));
```

运行时流程是：



1. 在对象的 **类（Class）** 的 **方法缓存（cache）** 中找 doSomething；

2. 找不到就去方法列表（method list）；

3. 仍找不到，就沿着 **继承链** 向父类查找；

4. 如果整个继承链都没找到该方法实现，

   那么运行时就会进入 **消息转发机制**（Message Forwarding）。

这个时候就不是 objc_msgSend 继续处理了，

而是交给 _objc_msgForward 来进行“补救”。

**2. _objc_msgForward 的核心职责**

_objc_msgForward 是运行时的一个“兜底函数”，

当系统找不到某个 selector 的实现时，它会触发一个“消息转发”流程。

这整个流程依次会调用几个方法，让开发者有机会动态处理：

1. +resolveInstanceMethod: / +resolveClassMethod:
2. -forwardingTargetForSelector:
3. -methodSignatureForSelector:
4. -forwardInvocation:
5. -doesNotRecognizeSelector:

**3. 五个阶段的调用顺序与作用**

**（1）resolveInstanceMethod: / resolveClassMethod:**

> “动态方法解析阶段”

系统先问你：

“要不要现在动态添加这个方法的实现？”

```objc
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(doSomething)) {
        class_addMethod(self, sel, (IMP)dynamicMethodIMP, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

如果你返回 YES 并成功添加了方法，系统就会重新发送消息；

如果返回 NO，进入下一阶段。

**（2）forwardingTargetForSelector:**

> “快速转发阶段（Fast Forwarding）”

系统问：

“这个消息是不是该转发给其他对象？”

你可以直接指定一个新的对象来接收这个消息：

```objc
- (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(doSomething)) {
        return self.helper; // helper 对象有实现 doSomething
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

如果返回一个对象，系统会把消息直接发送给这个对象；

如果返回 nil 或 self，则继续下一步。

**（3）methodSignatureForSelector:**

> “生成方法签名阶段”

系统问：

“我准备打包消息了，请告诉我这个方法的参数和返回值结构（NSMethodSignature）。”

```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    if (aSelector == @selector(doSomething)) {
        // v@: 表示 void 返回值，参数 self 和 _cmd
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    return [super methodSignatureForSelector:aSelector];
}
```

如果返回 nil，表示完全无法处理该消息，系统会直接抛出异常：

unrecognized selector sent to instance ...

否则，进入下一步。

**（4）forwardInvocation:**

> “完整消息转发阶段（Normal Forwarding）”

系统把消息打包成一个 NSInvocation 对象传给你，你可以：

- 修改 selector；
- 修改参数；
- 转发给别的对象；
- 或者自己处理。

```objc
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([self.helper respondsToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:self.helper];
    } else {
        [super forwardInvocation:anInvocation];
    }
}
```

**（5）doesNotRecognizeSelector:**

> “最终兜底阶段（真正报错）”

如果前面所有阶段都没有处理该消息，系统就会执行：

```objc
- (void)doesNotRecognizeSelector:(SEL)aSelector {
    [super doesNotRecognizeSelector:aSelector];
}
```

此时程序会崩溃，抛出经典错误：

```shell
*** Terminating app due to uncaught exception 'NSInvalidArgumentException',
reason: '-[MyClass doSomething]: unrecognized selector sent to instance ...'
```

**4. 实际应用场景**

1. **动态方法解析（resolveInstanceMethod）**
   - 用于 @dynamic 属性；
   - 用于运行时添加方法（Method Swizzling 或动态代理）。
2. **快速转发（forwardingTargetForSelector）**
   - 常用于代理模式，例如把消息转发给内部 helper 对象；
   - 类似“多继承”的委托实现。
3. **完整消息转发（forwardInvocation）**
   - 用于实现类似 **消息中间层（message proxy）**；
   - 比如 NSProxy 的子类通常使用这一机制。

**6. 核心理解总结**

1. _objc_msgForward 是 Objective-C 的“找不到方法时的应急入口”；
2. 整个消息转发分为三层机制：
   - **动态解析（Dynamic Resolution）**
   - **快速转发（Fast Forwarding）**
   - **完整转发（Normal Forwarding）**
3. 你有 4 次机会（前四个方法）来“拦截”崩溃并自行处理消息；
4. 如果都没处理，系统最终会调用 doesNotRecognizeSelector: 导致崩溃。



## ios objc中runloop是做什么的？和县城什么关系，其集中模式的作用是什么？

**RunLoop（运行循环）** 是 iOS / macOS 底层非常核心的机制之一，

理解它就能掌握事件处理、线程保活、定时器、触摸响应、UI 更新等的本质。

**我们一步步讲清楚：**

**1. RunLoop 是什么？**

RunLoop 可以理解为：

> **一个让线程“活着”等待事件的循环机制。**

简单说，它的作用就是：

- 让线程在**有事做时工作**（处理事件）；
- 在**没事做时休眠**（节省 CPU 资源）；
- 从而让线程能长期存在而不退出。

类比：

> 就像一个“事件调度器”——不断循环地检测是否有任务、输入事件、定时器等需要处理。

**2. RunLoop 和线程的关系**

1. **每个线程都有一个 RunLoop，但默认只主线程自动开启。**

   - 主线程的 RunLoop 在 UIApplicationMain() 中自动启动；
   - 子线程的 RunLoop 默认是**不存在的**，需要手动创建并运行。

2. **RunLoop 与线程一一对应**：

   每条线程的 RunLoop 保存在一个全局字典中（线程指针为 key）。

3. **RunLoop 的生命周期 = 线程的生命周期**：

   - 当线程结束时，它的 RunLoop 也被销毁；
   - RunLoop 不会保持线程存活，反而是**RunLoop 让线程保持“等待事件”的活性状态**。

简单示意：

```objc
NSThread *thread = [[NSThread alloc] initWithBlock:^{
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    [runLoop run]; // 手动启动 RunLoop，线程开始循环等待事件
}];
[thread start];
```

**3. RunLoop 的核心作用**

| **功能**                  | **说明**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| **保持线程存活**          | 没有 RunLoop 的线程执行完任务就退出，有 RunLoop 的线程可长期等待事件 |
| **事件分发机制**          | 系统会将触摸事件、定时器回调、GCD 任务等加入 RunLoop 循环中处理 |
| **节省资源**              | 当没有事件时自动休眠，防止 CPU 空转                          |
| **管理定时器（NSTimer）** | NSTimer 必须依附在 RunLoop 上才能生效                        |
| **控制输入源**            | 处理输入源（触摸、网络、端口、PerformSelector等）            |
| **处理 UI 刷新**          | 主线程 RunLoop 负责 UI 刷新、动画、事件响应等                |

**4. RunLoop 的内部结构（核心组成）**

| **组成部分**                | **说明**                                          |
| --------------------------- | ------------------------------------------------- |
| **Input Sources（输入源）** | 异步事件（触摸、网络、Port 等）触发的回调         |
| **Timer Sources（定时源）** | NSTimer 等基于时间的事件                          |
| **Observer（观察者）**      | 可以监听 RunLoop 状态变化（即将睡眠、即将退出等） |
| **Mode（运行模式）**        | 运行循环的“配置集”，决定当前要监视的事件源集合    |

**5. RunLoop 的运行模式（Modes）**

RunLoop 是按“模式（Mode）”运行的。

每次只能运行在某一个 Mode 中。

常见系统内置的 Mode：

| **模式名称**              | **说明**    | **使用场景**                                                 |
| ------------------------- | ----------- | ------------------------------------------------------------ |
| **NSDefaultRunLoopMode**  | 默认模式    | 普通任务、定时器                                             |
| **UITrackingRunLoopMode** | UI 追踪模式 | 手指拖拽、滚动时（UIScrollView）                             |
| **NSRunLoopCommonModes**  | 模式集合    | 包含多个模式（如 Default + Tracking），用来同时监听多种模式事件 |

**重点理解：Mode 是用来隔离事件源的。**

也就是说：

- 当 RunLoop 运行在某个 Mode 下时，只会处理这个 Mode 中的事件源；
- 不同 Mode 之间相互独立，防止互相干扰。

**6. 举个实际例子**

问题：为什么 NSTimer 在滚动 UIScrollView 时会暂停？

```objc
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                                                  target:self
                                                selector:@selector(doTask)
                                                userInfo:nil
                                                 repeats:YES];
```

- 默认情况下，scheduledTimerWithTimeInterval: 会被添加到 NSDefaultRunLoopMode；
- 当你滚动 UIScrollView 时，RunLoop 进入 UITrackingRunLoopMode；
- 所以 Timer 在 Default 模式下不会被处理，看起来“暂停”了。

**解决办法：**

让 Timer 同时在多种模式下工作：

```objc
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

NSRunLoopCommonModes 是一个模式集合，包含常见模式：

- NSDefaultRunLoopMode
- UITrackingRunLoopMode

这样 Timer 在滚动时也能继续触发。

**7. RunLoop 的运行流程（简化版）**

```shell
while (线程未结束) {
    通知观察者：RunLoop 将进入
    检查定时器、输入源是否有任务
    如果没有任务 → 进入休眠（等待事件唤醒）
    如果有任务 → 执行任务回调
    通知观察者：RunLoop 将休眠 / 将退出
}
```

**8. RunLoop 在系统中的典型应用**

| **应用场景**                       | **说明**                                             |
| ---------------------------------- | ---------------------------------------------------- |
| **主线程事件循环**                 | 处理触摸、UI 绘制、定时器、动画、KVO 回调等          |
| **子线程保持活性**                 | 例如后台下载线程、音频播放线程                       |
| **自动释放池（@autoreleasepool）** | 每次 RunLoop 循环都会自动创建和释放一个池            |
| **网络请求、Socket 通信**          | CFRunLoopSource 监听端口事件                         |
| **GCD 与 RunLoop 协调**            | 主线程 GCD 回调实际上依附于 RunLoop 的事件循环中执行 |

**9. 小结对比**

| **概念**                                     | **说明**                                 |
| -------------------------------------------- | ---------------------------------------- |
| **RunLoop 是线程的“事件循环”机制**           | 让线程在空闲时休眠，有任务时被唤醒       |
| **每个线程可有一个 RunLoop（懒加载）**       | 主线程默认开启，子线程需手动创建         |
| **Mode 用于隔离事件源集合**                  | 一次循环只监听一个 Mode 的事件           |
| **CommonModes 是模式集合**                   | 可让事件在多种模式下被监听               |
| **RunLoop 常用于定时器、线程保活、事件响应** | 是 UIKit、Cocoa Touch 底层的重要支撑机制 |

> **RunLoop = 线程的“心脏”**

> 它维持线程的生命循环，让线程能持续运行、处理事件、节省资源；

> “模式（Mode）” 则像是“过滤器”，决定当前 RunLoop 要处理哪些类型的事件。



## ios objc 在 block 内为什么不能修改 block 外部变量

**1️⃣ 现象演示**

```objc
int count = 0;

void (^myBlock)(void) = ^{
    count = 10;   // ❌ 报错：Variable is not assignable (missing __block type specifier)
};
myBlock();
```

编译器报错：

> *Variable is not assignable (missing __block type specifier)*

也就是 —— **在 block 内无法修改外部的普通局部变量。**

**2️⃣ 原因：block 捕获（capture）机制**

当你在 block 内部访问外部变量时，

**block 会把这个变量的值“复制”一份到自己的内部内存中**，

而不是直接引用原变量。

也就是说：

```objc
int count = 0;
void (^myBlock)(void) = ^{
    NSLog(@"%d", count);
};
```

实际上是：

- 在创建 block 时，count 的当前值（=0）被**拷贝**进 block；
- block 内部访问的是**这份拷贝值**；
- 因此，即使在外部 later 改变 count，block 内部不会感知。

这叫做：

> **按值捕获（capture by value）**

**3️⃣ 捕获行为举例**

```objc
int count = 10;
void (^myBlock)(void) = ^{
    NSLog(@"count = %d", count);
};
count = 20;
myBlock();   // 输出：count = 10
```

解释：

- block 在创建时捕获了 count = 10；
- 后面 count 改为 20；
- 但 block 内访问的还是它捕获时的旧值（10）。

**4️⃣ 为什么不能修改？**

因为 block 捕获的是一个 **const 拷贝值**。

也就是说：

> block 内部的捕获变量默认是只读的。

换句话说，block 拷贝了当时的值，

但它不能直接修改外部的原变量（因为两者不是同一个存储位置）。

**5️⃣ 如何让 block 能修改外部变量？**

👉 使用 __block 修饰符。

```objc
__block int count = 0;

void (^myBlock)(void) = ^{
    count = 10;   // ✅ 可以修改
};

myBlock();
NSLog(@"count = %d", count);  // 输出 count = 10
```

**6️⃣ 为什么加了 __block 就能修改？**

__block 会改变编译器对该变量的捕获方式：

| **普通变量**             | __block **变量**                     |
| ------------------------ | ------------------------------------ |
| **按值捕获**（复制值）   | **按引用捕获**（复制变量的引用对象） |
| block 内访问的只是拷贝值 | block 内外访问的其实是同一个存储单元 |
| block 内不能修改         | block 内可以修改，修改外部也可见     |

编译器实际上会把 __block 修饰的变量包装成一个结构体，

结构体中保存指向变量的指针。

block 内外都操作这同一个结构体 → 所以修改能同步。

**7️⃣ 示例对比**

```objc
// 普通变量捕获
int a = 10;
void (^blockA)(void) = ^{ NSLog(@"a = %d", a); };
a = 20;
blockA(); // 输出 10

// __block 捕获
__block int b = 10;
void (^blockB)(void) = ^{ b = 20; NSLog(@"b = %d", b); };
blockB(); // 输出 20
NSLog(@"外部 b = %d", b); // 输出 20
```

**8️⃣ 总结表格**

| **修饰符** | **捕获方式** | **是否能修改** | **block 外是否能看到修改** | **原理**                             |
| ---------- | ------------ | -------------- | -------------------------- | ------------------------------------ |
| （无修饰） | 值捕获       | ❌ 不可修改     | ❌                          | 复制值到 block 内存                  |
| __block    | 引用捕获     | ✅ 可修改       | ✅                          | block 内外共享同一内存（结构体包装） |

**9️⃣ 特别说明：对象类型变量**

对对象（比如 NSString *str）也有类似现象：

```objc
NSString *name = @"Tom";
void (^myBlock)(void) = ^{
    NSLog(@"%@", name);
};
name = @"Jerry";
myBlock(); // 输出 Tom
```

如果想在 block 内修改对象的指向（不是对象内容，而是指针本身），也要加 __block

```objc
__block NSString *name = @"Tom";
void (^myBlock)(void) = ^{
    name = @"Jerry"; // ✅ 允许修改指针
};
myBlock();
NSLog(@"%@", name); // 输出 Jerry
```

不过，如果只是修改对象内容（如 NSMutableArray 添加元素），不用加 __block，因为对象地址没变。

**🔟 一句话总结**

> 在 Objective-C 中，block 捕获外部变量时默认是**按值复制（copy by value）**，

> 所以无法在 block 内修改原变量；

> 使用 __block 可以改为**按引用捕获（by reference）**，

> 使 block 与外部共享同一变量，从而支持修改。



































