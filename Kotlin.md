# Flow   
# 协程  
-  是什么    
    “协程是运行在用户态的轻量执行单元，suspend 函数在挂起点让出线程、异步完成后原地恢复，结合结构化并发实现高并发又易管理。”
- 协程的上下文  
    CoroutineContext 就是协程的配置与控制面板，包含 Job 和 Dispatcher 等信息，并在挂起调用链里隐式传递。  
- Continuation  
    Continuation 表示‘如何从挂起点继续’，持有状态机的帧与 CoroutineContext。异步完成时调用 resumeWith 恢复执行，是把回调式 API 封装为挂起函数的桥梁  

# 拓展函数  
扩展函数是静态函数的语法糖，kotlin 编译时建立一个工具类，声明形如 fun ext(receiver: T, ...) 的静态方法。冲突时类自己的成员函数优先。因为扩展只是静态帮助函数，不改变类本身的 API。

# 数据类  
数据类自动生成equals/hashCode/toString/componentN/copy

# crossinline/noinline 
noinline 禁止某参数内联；crossinline 禁止非局部返回（如直接 return 跳出外层函数），保证可在不同上下文安全调用，inline直接进行内联。

# apply 的底层原理
 inline + receiver lambda：inline fun <T> T.apply(block: T.() -> Unit): T。

 # kotlin的apply和also有什么区别
 - 相同点：apply 和 also 都是作用域函数，默认 inline，返回原对象本身，常用于链式调用与初始化时的副作用。
 - 区别  
    apply：把对象作为接收者 this 传入；在 lambda 内直接访问成员；返回对象本身。适合“配置该对象”的初始化场景。  
    also：把对象作为参数 it 传入；更强调“对该对象做点额外的事（副作用）”；返回对象本身。适合调试、日志、校验、统计等不改变业务主线的旁路操作。     
要“改这个对象的属性/调用它的方法并继续返回它”：用 apply  
要“顺带做点事，但不改变对象本身（例如日志/校验/注册）”：用 also

# by委托  
“by 委托”是 Kotlin 提供的委托语法糖，用来把某些职责转交给“委托对象”来实现。主要有两大类场景：
1. 类委托（委托实现接口/类的功能）：编译期生成“转发方法”
2. 属性委托（把属性的读写逻辑交给代理对象，如 by lazy、by map、可观察/可校验属性等）：把属性 p 的 get/set 行为交给 delegate 对象。编译器会把 p 的访问重写为调用 delegate.getValue/ setValue。  

# Kotlin空安全实现机制  
编译器在字节码层通过「可空类型标记 + 自动空值检查」  

# Kotlin泛型和Java泛型的区别
Java：泛型在编译期类型检查，运行时类型擦除（type erasure），没有 reified。通用用法依赖通配符 ? extends/ super 做 use-site variance。

Kotlin：同样编译到 JVM，会产生和 Java 类似的擦除，但语言层支持 声明站位变型（declaration-site variance）（out/in）、可实化泛型（reified） + inline（仅在 inline 函数里），并且对可空性、星投影（star-projection）、泛型约束的语法更友好。  
## reified
reified 只能用于 inline 函数，允许在编译期把类型实化到调用点，从而在函数体内能获得实际类型。

# 回调实现  
Kotlin 的函数类型（() -> Unit、(Int) -> String 等）是一个实现了 FunctionN 接口的匿名对象，编译器自动帮你生成这个对象。  

# lazy  
lazy用于实现延迟初始化。第一次访问的时候才运行，之后每次访问都返回缓存的结果。
- 原理  
它通过实现 Lazy<T> 接口（默认使用 SynchronizedLazyImpl）完成，内部用 synchronized 或原子变量保证线程安全。by lazy 编译后会生成一个代理对象，访问属性时调用其 value。

# Kotlin Native
可以编译成原生机器码，不需要JVM，自带一套轻量的运行时，提供协程支持、内存管理（GC）、线程与并发原语、异常处理、标准库底层支持等
- 其可以与c互操作，但是不能直接调用 java 库
- 协程同样可用，但调度器实现不同；某些情况下对主线程约束更严格
- 