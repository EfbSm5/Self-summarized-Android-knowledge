# Java 基础整理

## 死锁（Deadlock）
- 四个必要条件
  - 互斥（Mutual Exclusion）：资源同一时刻只能被一个线程占有
  - 请求并保持（Hold and Wait）：持有部分资源同时请求新资源且不释放已占有
  - 不可剥夺（No Preemption）：已获得资源不可被强制剥夺
  - 循环等待（Circular Wait）：存在进程/线程形成的环形资源等待链

## synchronized 与 Lock 的区别
- 实现层面
  - synchronized：JVM 层面，基于对象头与监视器；自动释放
  - Lock（如 ReentrantLock）：API 层面，需手动释放
- 特性对比
  - 可中断：synchronized 不可中断；Lock 可中断
  - 获取方式：synchronized 阻塞等待；Lock 支持非阻塞 tryLock、超时获取
  - 公平性：synchronized 非公平；Lock 可配置公平/非公平
  - 条件队列：Lock 提供 Condition 多条件队列；synchronized 只有一个 wait/notify 集合
- 使用建议
  - 优先 synchronized：简单、易用、错误率低
  - 需要非阻塞/可中断/超时/公平锁/多条件时选 Lock
- JVM 锁优化
  - 对象头 Mark Word；偏向锁 → 轻量级锁 → 重量级锁的升级过程

## JNI 调用 C 代码
- 两个方向：Java 调 C；C 回调 Java/访问字段
- 基本流程
  1) Java 声明 native 方法；System.loadLibrary("hello")
  2) javac -h 生成头文件
  3) C/C++ 实现头文件中的函数
  4) 编译为动态库并随进程加载
- 常见类型处理
  - 原始类型可直接用
  - 引用类型需转换，如 GetStringUTFChars 获取 UTF-8 字符串  
- 上面的是静态注册，假如动态注册？手动进行Java方法和native方法关联，通过会必须被调用的JNI_OnLoad函数完成注册。系统在创建完虚拟机后就会注册一大堆JNI函数，通过类名，我们可以找到常见类的JNI函数实现位置。  
- 对于JNI中的对象，同样可以设置不同引用来设置其生命周期和释放方式。

## 类加载过程
- 加载：读入字节码，创建 Class 对象
- 验证：字节码合法性校验，保障 JVM 安全
- 准备：为类静态变量分配内存并设默认值 null
- 解析：符号引用 → 直接引用
- 初始化：执行 <clinit>，为 static 赋值/执行静态代码块

## Exception vs Error
- 共同点：均继承 Throwable
- Error：JVM 内部严重错误，通常不可恢复，不建议捕获
- Exception：
  - RuntimeException：由程序错误引发（空指针、越界、类型转换等）
  - 非运行时异常：如 I/O、SQL、反射等受检异常

## String / StringBuilder / StringBuffer
- String：不可变，线程安全；频繁拼接性能差
- StringBuilder：可变，非线程安全，性能高（单线程推荐）
- StringBuffer：可变，线程安全，同步开销导致相对慢

## final / finally / finalize
- final：修饰类（不可继承）、方法（不可重写）、变量（不可再赋值）
- finally：try-catch 结束后必执行的块（除非进程终止等极端情况）
- finalize：对象回收前回调，不可靠，已被废弃语义，不应使用

## int vs Integer
- int：基本类型，高性能，不能为 null
- Integer：包装类，可为 null，可用于泛型/集合/反射等，存在装箱拆箱开销

## ==、equals、hashCode
- ==：基本类型比较值；引用类型比较是否同一对象
- equals：默认等同于 ==；不少类重写为“逻辑相等”
- hashCode 关系
  - equals 为 true → hashCode 必须相等
  - 反之不要求
  - 影响 HashMap/HashSet 的定位与去重
  - 实现建议：将定义逻辑身份的不可变字段纳入 equals/hashCode；重写二者保持一致；与 compareTo 语义尽量一致

## static 关键点
- 静态变量：类级共享，内存中仅一份
- 静态方法：与实例无关的操作
- 静态代码块：类加载时一次性初始化
- 接口中的静态方法：不会被实现类继承
- 静态上下文不能直接访问实例成员

## 重写与重载
- 重写（override）：父子类方法签名相同，体现多态
- 重载（overload）：同名不同参，编译期分派

## Java 集合概览
- Collection
  - List：有序可重复（ArrayList、LinkedList、Vector）
  - Set：不重复（HashSet、LinkedHashSet、TreeSet）
  - Queue/Deque：队列（LinkedList、ArrayDeque、PriorityQueue）
- Map
  - HashMap、LinkedHashMap、TreeMap、ConcurrentHashMap、SortedMap/NavigableMap

## Runnable 与 Thread
- Runnable：定义任务“做什么”（接口）
- Thread：定义“在哪个执行流里跑”（类）
- 解耦优势：任务可提交到不同执行环境（线程、线程池、定时器）；同一 Runnable 可被多线程复用

## 静态方法与静态内部类
- 静态方法/静态内部类：不能直接访问外部类的非静态成员
- 非静态内部类：持有外部类实例引用，可访问其成员

## ThreadPoolExecutor 核心机制
- 提交流程：先核心线程 → 队列 → 非核心线程 → 拒绝策略
- 关键参数：corePoolSize、maximumPoolSize、keepAliveTime、workQueue、threadFactory、handler
- 回收：非核心受 keepAliveTime 影响；allowCoreThreadTimeOut 可让核心也回收
- 最佳实践：显式使用 ThreadPoolExecutor 参数化创建，避免 Executors 工厂的无界风险

## ArrayList 扩容
- 基于数组，默认（JDK8）扩容为旧容量的 1.5 倍
- 扩容触发：添加时容量不足
- 成本：新数组分配 + 元素拷贝，频繁扩容性能差（可预估容量）

## HashMap 存储机制
- 底层：数组 + 链表/红黑树（JDK8 起）
- 冲突：同桶元素拉链，链长超过阈值且表容量足够时树化
- 负载因子：默认 0.75；达到阈值时扩容为 2 倍并重哈希
- hash 扰动：减少碰撞均匀分布

## LRU（Least Recently Used）
- 最近最少使用策略，优先淘汰最久未使用的元素
- 常见实现：LinkedHashMap（accessOrder=true），或双向链表 + HashMap
- 在缓存中提升命中率，支持 O(1) 访问/更新/淘汰

## Java 内存模型（JMM）要点
- 可见性：volatile、锁、final 语义保证线程间可见
- 有序性：happens-before 规则（锁解锁、volatile 读写、线程启动/终止/中断、传递性）
- 原子性：基本读写具备；复合操作需同步
- 指令重排：JMM 允许重排但保持 HB 语义；volatile 禁止特定重排
- 常见并发安全手段：不可变对象、CAS + 原子类、锁、并发容器