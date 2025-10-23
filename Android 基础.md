# 四大组件  
- Activity  
  管理 UI 和用户交互。生命周期由 onCreate（初次加载） → onStart（可见，但不能交互） → onResume（可以交互） → onPause → onStop → onDestroy 控制。底层由 ActivityManagerService (AMS) 管理，通过 Binder 与应用通信。若后台重新启动需要onRestart，接着走完 onStart 和 onResume。  
- Service  
  后台任务执行组件，没有界面。分为 启动型服务（startService） 和 绑定型服务（bindService）。常用于播放音乐、下载等。  
- BroadcastReceiver  
  消息广播机制。系统或应用发送广播（如电量变化），由接收器监听并响应。分为 静态注册（manifest，常用于开机启动）和 动态注册（代码里注册）。  
- ContentProvider  
  数据共享组件，跨进程访问数据的主要方式，底层用 Binder IPC。常用于访问系统数据（通讯录、媒体库）或自定义数据。

# Fragment  
Fragment 是 Activity 的“子界面”，可以独立管理生命周期与 UI。  
- 生命周期  
onAttach->onCreate->onCreateView（返回View对象）->onViewCreated(初始化交互逻辑)->onStart->onResume->onPause->onStop->onDestoryView->onDestory->onDetach()  

# Service 启动机制
写好Intent之后  
1. startService,这样的service是和activity没有强绑定关系的，即使activity退出了service也仍然继续运行
2. bindService,这service可以进行消息互通，有绑定关系，退出activity,service也会关闭
3. 又startService也bind，这样没有绑定，即使activity退出也会运行，也可以传输数据
4. 前台服务，startForegroundService，服务里startForeground，适合需要强提醒的
- 假如是StartService生成的service,onCreate->onStartCommand()->onDestory() onStartCommand()返回整数值，描述系统杀死服务怎么重启    

# 为什么用Service,不用thread
Thread 解决的是“并发问题”，
Service 解决的是“生命周期问题”。
线程只是一个执行单元，而 Service 是系统级组件，有生命周期管理、进程隔离、前后台控制、持久运行能力。Service的生命周期独立于Activity,系统会尽力保活

# View的渲染流程
- 测量，布局，绘制
- 测量主要是确定每个view和viewgroup的大小，这是一个从顶到下的过程，从decorview开始，每个viewgroup根据父容器传递的测量规格，将自己的约束传递给子view,询问子view想要多大，这个过程中，整个视图树可能被遍历多次，如果一个子view的测量结果影响了其他视图的尺寸（比如weight属性），可能会需要更多次测量
- 布局确定具体位置，与测量相似
- 绘制
  将view的实际内容绘制到屏幕上的canvas，这也是自顶向下的递归过程，但是绘制顺序可以由onDraw方法影响
- 硬件加速
  在现代android为了提升性能会启用硬件加速，不开启的话，指令直接在cpu执行，结果写入bitmap，开启硬件加速后，onDraw的绘制命令不会被立刻执行，而是记录到显示列表中，最终这个显示列表会被同步到一个renderthread,这个线程使用GPU将命令转换为实际的OpenGL或者Vulkan指令，进行光栅化，最终提交给系统进行合成显示

 # 自定义View的执行流程和注意事项。
- 还是测量布局绘制
- 重写测量的 onMeasure,可以测量计算view应该有多大，需要根据传入的spec模式和自己的内容得到一个希望的大小，当是at_most处理wrap_content，必须调用setMeasuredDimension保存结果,假如是viewGroup还需要重新写onLayout来安排子view,普通view不需要
- 重写onDraw,通过canvas和paint进行自定义绘制，避免分配对象，需要支持padding
- invalidate（）触发重绘，requestLayout（）触发重新测量布局

   
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
    
# Binder    
- 高效：基于内核驱动 binder，采用内存映射减少数据拷贝只需一次 copy。
- 安全：内置 UID/PID 验证，进程身份天然可信。
- 一致性：提供面向对象的调用方式（代理模式）。

# AIDL  
- Android Interface Definition Language，用于定义进程间通信接口。定义一个 .aidl 文件声明接口，编译时生成  Stub binder。Client 实现接口，最终通过 Binder 驱动调到 Service 端的 Service。  支持基本类型、序列化对象、List/Map，但数据结构必须可序列化。  

# Messenger  
使用 Handler + Message 封装 Binder，也可以实现 ipc

# Handler
- 干什么用：线程之间传递消息执行任务  
- 组成：Message、MessageQueue、Looper、Handler。
- 流程：
Handler 发送 Message → 入队 MessageQueue->Looper 不断循环，取出 Message->分发给对应 Handler 处理。    
- MessageQueue 的机制  
MessageQueue 以 Message.when 升序维护一个有序链表；同一时间的消息按插入顺序执行。
- Handler 运行在哪个线程，取决于它绑定的 Looper 所在线程。
- post {} 适合执行简单逻辑，sendMessage() 适合传递结构化数据（如任务类型、参数等）

# Compose渲染原理
- Composition，layout,draw
- composition是决定要显示什么，compose运行时会构建并维护一个ui描述的数据结构 Slot Table,这个数据结构描述了UI的逻辑结构与状态，而不是直接的绘制指令，当state变化时，compose调度器会安排重组，重组过程中，compose会比较新的ui描述与旧的slot table,找到什么需要重组，从而生成新的ui数
- 布局，决定ui位置与大小，其需要测量子项，和决定自身大小和位置，而compose比view好在只用单次测量，避免了view系统多次测量的性能问题
- 绘制，与view的硬件加速类似

# skia
跨平台的 2D 图形渲染引擎,把上层的绘图命令（如 drawRect、drawCircle、drawText、drawBitmap）转换为底层的 GPU / CPU 渲染操作。

# RecyclerView   
- RecyclerView 的复用是“按 viewType 分类的多级缓存 + 统一的 ViewHolder 框架”。当某个 item 滑出屏幕时，其 ViewHolder 被回收到缓存/池中；当新 item 需要显示时，从这些缓存优先取可复用的 ViewHolder，尽量避免新建与全量绑定，从而提升性能与流畅度。
- 缓存机制。  
  1. Scrap 缓存（屏幕内还在使用的复用） 
  2. CachedViews（屏幕外缓存）
  3. ViewCacheExtension（开发者自定义）
  4. RecycledViewPool（全局缓存池）
- ViewHolder  
ViewHolder 是 RecyclerView 高效复用的“最小工作单元”：承载 itemView 的视图引用与状态，解耦“视图结构创建”和“数据绑定”，显著降低创建/查找视图的成本，同时为缓存与回收机制提供载体。

# OOM  
- 常见原因：
  1. 大量 Bitmap 占用内存。
  2. Context 泄漏（持有 Activity 的引用）。
  3. 静态集合保存大量对象。
  4. 未关闭 Cursor、IO。
- 解决办法：
  1. 使用 弱引用（WeakReference）。
  2. 使用 ApplicationContext 而不是 ActivityContext。
  3. Bitmap 使用压缩、缓存（LruCache）。
  4. 检查内存泄漏工具（LeakCanary）。  

# 内存泄漏
- 对象没有被正常回收
- 匿名类和内部类没有被处理，导致外部类不能被正确回收
- 单例模式的对象没有被释放或者及时清理，导致对象中一直存在于内存中
- 使用Handler,未正确处理消息队列和外部类弱引用，可能导致外部类无法被回收
- 长时间后台任务，运行完成不清空
- context被长生命周期对象引用
- 使用缓存导致内存泄漏
- 未关闭的资源，使用一些资源完成后没有显式关闭这些资源，导致资源泄漏

# ANR
- 触发原因
  复杂计算，网络请求，IO操作，线程死锁，主线程等待子线程，binder调用
  ANR会生成traces.txt 其中包含了anr的调用栈信息，可以定位问题
  避免 strictmode
- 触发条件  
  主线程 5s 内未响应输入事件。
  BroadcastReceiver 10s 内未完成。
  Service 20s 内未完成。  

# 四种启动模式  
- standard：默认模式，每次启动都会新建实例。
- singleTop：栈顶已有实例则复用，否则新建。
- singleTask：栈内复用，复用时会调用 onNewIntent。
- singleInstance：独占一个任务栈。

# AMS  
1. 管理四大组件的生命周期。
2. 负责进程调度、任务栈管理。
3. 通过 Binder 与应用交互。  

# PMS  
1. 管理 APK 安装、卸载、解析。
2. 记录包信息、签名验证。
3. 提供 PackageManager 接口给应用调用。  

# WMS  
1. 管理窗口的添加、删除、层级。
2. 与 SurfaceFlinger 协作完成界面显示。
3. 负责输入事件的分发。  

# XML 布局的“加载”
- XML 布局的“加载”就是 LayoutInflater 读取 XML，按标签创建 View，解析并应用属性与样式，递归组装成 View 树，并根据需要附加到父容器的过程。你在 XML 写的每个节点，最终都会走到“构造函数 + 属性解析 + LayoutParams 生成 + 递归添加”的这条链路。
- XML布局的加载就是LayoutInflater读取XML,按标准创建View，解析并应用属性与样式，递归组装成View树，根据需要附加到父容器。

# 单activity
- 性能好，不需要新开acitivity
- 简化状态管理与数据共享

# 性能优化
- 衡量哪里不好
- 利用工具找到性能瓶颈 android studio自带
  1. 流畅 布局避免过度绘制，减少布局层级
  2. 列表优化，recycleview,异步加载，分页加载
  3. 避免gc,避免在频繁调用的方法中创建对象，防止造成内存波动
  4. 懒加载，避免启动时密集io操作
  5. 防止内存泄漏，使用合适的图片尺寸
  6. 网络优化


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

# WebView
View组件，可以用来渲染HTML界面，可以显示远程网页，本地HTML文件，显示字符串拼接的HTML内容。
- 与Android交互  
可以在 Android 端注册一个jsinterface对象，让网页端的 JS 调用，也可以让Android调用网页中的js函数，监听页面事件，

# 程序遇到异常退出，怎么知道（客户的手机上crash，怎么捕获并传到服务端）  
- 在 Application 安装 UncaughtExceptionHandler
- 崩溃时只写本地文件（原子写入），不要发网络
- 冷启动后扫描 crash 目录，异步 gzip 上传，成功后删除
  
# 热修复  
- 在应用已经安装了以后，开发者还可以动态加载补丁包，在运行时替换出问题的方法，从而修bug不重发版本。
- 原理：把新Dex放在加载顺序最前面
  1. 加载补丁 dex（修复包） →
  2. 用反射拿到 PathClassLoader.pathList.dexElements →
  3. 将“补丁 dex 的 element”插入到原先 elements 的最前面 →
  4. 这样当类查找时，优先会在“补丁 dex”中找到修复后的类。
    
# Android内存占用大的场景  
1. 内存泄漏  
2. 图片太多  解决：减少图片尺寸，换用更空间更小的格式  
3. 视频/相机实时画面 原因：显示至少需要双缓冲，解码器帧缓冲也常驻 解决：降低分辨率，需要时开启高分辨率预览，减少中间缓冲尺寸。  
4. WebView  避免：尽量复用，开启懒加载  。   
5. 加载超大数据

# 如何绘制canvas，初始化canvas  
- 自定义控件绘制：onDraw 的 Canvas，最简单、最稳定。
- 需要离屏合成/重复使用绘制结果：Bitmap + Canvas（离屏）。
- 需要主动渲染、与 UI 解耦：SurfaceView/TextureView。
- 想在已有 Texture 上偶尔涂写：TextureView.lockCanvas。  
- compose, 直接使用Canvas函数，或者modifier.drawBehind就可以。  

# 隐式启动  
当你调用 startActivity(intent) 或 startService(intent) 时，AMS（ActivityManagerService）会：
1. 在系统已安装应用的 AndroidManifest.xml 里查找所有声明了 <intent-filter> 的组件
2. 用“匹配规则”逐一对比你的 Intent 与每个 filter
3. 找到“最匹配”的一个或多个目标。如果多个同级匹配，弹出选择器让用户选；若无匹配则抛出 ActivityNotFoundException