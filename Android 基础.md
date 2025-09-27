# View的渲染流程
- 测量，布局，绘制
- 测量主要是确定每个view和viewgroup的大小，这是一个从顶到下的过程，从decorview开始，每个viewgroup根据父容器传递的测量规格，将自己的约束传递给子view,询问子view想要多大，这个过程中，整个视图树可能被遍历多次，如果一个子view的测量结果影响了其他视图的尺寸（比如weight属性），可能会需要更多次测量
- 布局确定具体位置，与测量相似
- 绘制
  - 将view的实际内容绘制到屏幕上的canvas,这也是自顶向下的递归过程，但是绘制顺序可以由onDraw方法影响，
- 硬件加速
  - 在现代android为了提升性能会启用硬件加速，不开启的话，指令直接在cpu执行，结果写入bitmap，开启硬件加速后，onDraw的绘制命令不会被立刻执行，而是记录到显示列表中，最终这个显示列表会被同步到一个renderthread,这个线程使用GPU将命令转换为实际的OpenGL或者Vulkan指令，进行光栅化，最终提交给系统进行合成显示

# Compose渲染原理
- Composition，layout,draw
- composition是决定要显示什么，compose运行时会构建并维护一个ui描述的数据结构 Slot Table,这个数据结构描述了UI的逻辑结构与状态，而不是直接的绘制指令，当state变化时，compose调度器会安排重组，重组过程中，compose会比较新的ui描述与旧的slot table,找到什么需要重组，从而生成新的ui数
- 布局，决定ui位置与大小，其需要测量子项，和决定自身大小和位置，而compose比view好在只用单次测量，避免了view系统多次测量的性能问题
- 绘制，与view的硬件加速类似

# XML 布局的“加载”
- XML 布局的“加载”就是 LayoutInflater 读取 XML，按标签创建 View，解析并应用属性与样式，递归组装成 View 树，并根据需要附加到父容器的过程。你在 XML 写的每个节点，最终都会走到“构造函数 + 属性解析 + LayoutParams 生成 + 递归添加”的这条链路。
- XML布局的加载就是LayoutInflater读取XML,按标准创建View，解析并应用属性与样式，递归组装成View树，根据需要附加到父容器。

# Retrifit
- 接口抽象：用 Java Interface 定义 API，高度抽象且类型安全。
- 动态代理：核心魔法。无需编写重复样板代码，在运行时动态处理所有方法调用。
- 注解解析：用注解配置请求，声明式编程，代码极其清晰。
- 责任链模式：通过“转换器”和“调用适配器”工厂链，功能高度可扩展和解耦。你可以轻松插入自定义逻辑。
- 依赖 OkHttp：站在巨人的肩膀上，专注于高层抽象，而让最好的 HTTP 客户端处理底层复杂网络问题。

# 单activity
- 性能好，不需要新开acitivity
- 简化状态管理与数据共享

# 性能优化
- 衡量哪里不好
- 利用工具找到性能瓶颈 android studio自带
  1。流畅 布局避免过度绘制，减少布局层级
  2.列表优化，recycleview,异步加载，分页加载
  3.避免gc,避免在频繁调用的方法中创建对象，防止造成内存波动
  4.懒加载，避免启动时密集io操作
  5.防止内存泄漏，使用合适的图片尺寸
  6.网络优化

# 内存泄漏
- 对象没有被正常回收
- 匿名类和内部类没有被处理，导致外部类不能被正确回收
- 单例模式的对象没有被释放或者及时清理，导致对象中一直存在于内存中
- 使用Handler,未正确处理消息队列和外部类弱引用，可能导致外部类无法被回收
- 长时间后台任务，运行完成不清空
- context被长生命周期对象引用
- 使用缓存导致内存泄漏
- 未关闭的资源，使用一些资源完成后没有显式关闭这些资源，导致资源泄漏

# Handler消息机制

# Service 启动机制
1. 写好Intent
2. startService,这样的service是和activity没有强绑定关系的，即使activity退出了service也仍然继续运行
3. bindService,这service可以进行消息互通，有绑定关系，退出activity,service也会关闭
4. 又startService也bind，这样没有绑定，即使activity退出也会运行，也可以传输数据
5. 前台服务，startForegroundService，服务里startForeground，适合需要强提醒的
- 假如是StartService生成的service,onCreate->onStartCommand()->onDestory() onStartCommand()返回整数值，描述系统杀死服务怎么重启

# 双亲委托机制
- 当类加载器收到加载类的请求时，它首先不会尝试自己加载这个类，而是把这个请求委托给自己的父类加载器去实现，只有父类加载器反馈自己无法加载，子类才会加载
- （顶层）BootClassLoader (加载Framework，核心类库)
  - ↑
- （中间）PathClassLoader (加载已安装的APK，自己的代码和第三方库)
  - ↑
- （底层）DexClassLoader (加载外部Dex/Jar)
- 避免自己的类覆盖了系统的类，避免重复加载

# ANR
- 复杂计算，网络请求，IO操作，线程死锁，主线程等待子线程，binder调用
- ANR会生成traces.txt 其中包含了anr的调用栈信息，可以定位问题
- 避免 strictmode

# 自定义View的执行流程和注意事项。
- 还是测量布局绘制
- 重写测量的 onMeasure,可以测量计算view应该有多大，需要根据传入的spec模式和自己的内容得到一个希望的大小，当是at_most处理wrap_content，必须调用setMeasuredDimension保存结果,假如是viewGroup还需要重新写onLayout来安排子view,普通view不需要
- 重写onDraw,通过canvas和paint进行自定义绘制，避免分配对象，需要支持padding
- invalidate（）触发重绘，requestLayout（）触发重新测量布局

# 常见布局
- LinearLayout weight权重
- RelativeLayout 相对于
- ConstraintLayout 加强版相对于
- FrameLayout 全部都默认在左上角，可以切换位置

# 尺寸单位
- dp显示布局
- sp显示字体
- px底层像素运算

# 申请权限
- 检查是否有权限ContextCompat.checkSelfPermission（context,permission），没有的话ActivityCompat.shouldShowRequestPermissionRationale，如果第一次就打开权限窗口，requestpermission

# 事件分发
- 核心思想：协作与责任传递
- dispatchTouchEvent 返回是继续往下传达处理还是返回上级
- onInterceptTouchEvent viewgroup的方法，判断是自己处理还是向下分发
- onTouchEvent 真正处理逻辑所在，
- 顺序性：事件总是先到达最外层的 ViewGroup，然后依次向内分发。
- 拦截的不可逆性：一旦父容器 (ViewGroup) 决定拦截（onInterceptTouchEvent 返回 true），当前手势序列的所有后续事件都会直接交给它的 onTouchEvent 处理，而不会再询问 onInterceptTouchEvent，也不会再分发给子 View。
- 消费的决定性：如果一个 View 消费了 ACTION_DOWN 事件，它就成了这个手势序列的“主角”，后续事件都归它处理。
- 责任回溯：如果事件一直向下分发，直到最内层的子 View 的 onTouchEvent 也没有被消费，那么事件会依次向上回溯，交给父容器的 onTouchEvent 处理。如果所有父容器都不处理，最终这个事件会被 Activity 的 onTouchEvent 接收。
- requestDisallowInterceptTouchEvent()：这是一个非常重要的方法。子 View 可以通过调用父容器的这个方法，请求父容器不要拦截当前及后续的事件。这在处理滑动冲突时非常有用（例如，子 View 是横向滑动的 ViewPager，父容器是纵向滑动的 ScrollView）。