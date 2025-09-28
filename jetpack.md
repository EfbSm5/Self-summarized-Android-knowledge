# ViewModel
- 生命周期  
    ViewModel并不直接绑定到UI组件的可见生命周期，而绑定在宿主的生命周期范围。在同一 Activity Fragment 的配置变化中复用同一个 ViewModel 实例，只有在彻底finish才会销毁。对于navigation，是绑定在NavBackStackEntry

# LifeCycle   
- 原理   
    在API < 29 之前，采用的是ReportFragment监听Activity生命周期，29之后，ComponentActivity继承了LifecycleOwner。ActivityLifecycleCallbacks生命周期回调。事件分发的处理逻辑在LifecycleRegistry类中，只有当宿主状态变更或者新增观察者两种情况，才会给观察者下发状态。  

# Room    
- 原理  
Room 是对 SQLite 的类型安全封装。底层仍是 SQLiteDatabase，但通过注解处理器在编译期生成样板代码，提供类型映射、SQL 校验与更友好的 API。   
- 优势  
1. 注解处理器在编译时解析 SQL，验证表/列是否存在、参数/返回类型是否匹配，提前发现语法或类型错误。  
2. 复杂类型通过 @TypeConverter 映射为 SQLite 支持的类型。  
3. 支持返回 Flow ，LiveData 等数据。
4. 主线程运行会直接报错，避免主线程 IO 操作

# ViewBinding    
- 原理  
    在编译器直接通过 XML 文件生成对应的 Java/Kotlin 类，比如ActivityMainBinding。生成的绑定类包含了对布局中所有视图的强类型引用，避免了 findViewById 的使用。
- 优势  
1.  类型安全：编译时检查，避免运行时错误
2.  空安全：自动处理可能为空的视图引用
3. 性能优化：避免了运行时的视图查找开销 
    
# LiveData  
- 原理
    通过observe方法注册观察者，会将 owner 的 Lifecycle 传入，将Observer与LifeCycleOwner组合成新的观察者包装类，进而实现绑定以及生命周期感知，只有合适的生命周期才会接收数据回调。
- 基本结构 
LiveData 持有一个数据值 mData 和一个版本号 mVersion，观察者记录其最后一次接受的版本号。
- 优势  
1. 实时数据刷新，当组件处于活跃状态或者从不活跃状态到活跃状态时总是能收到最新的数据。
2. 有生命周期，只有在活跃时才会更新，也不会出现内存泄漏

# DataBinding
- 原理  
  遍历 layout 生成对应的Binding类，其中间包括对应 View 的缓存，bind 静态方法，布局中字段的 setter，在布局中的@{}表达式会在编译期就被解析，避免运行时反射，当调用 binding.setVm(user)，databinding 会标记相应的 dirty flag,请求一次rebind,在更新中只更新脏标记位。
- 双向绑定   
    1. 数据 -> View：像普通绑定一样在 executeBindings() 设置 text。
    2. View -> 数据：为 EditText 等属性生成 InverseBindingListener，监听用户输入变化（TextWatcher），再调用对应的 setter（vm.setName(...) 或 ObservableField.set(...)）。
相较于 ViewBinding ，提供数据驱动 UI ，表达式，自动监听与最小更新，但是编译更重，性能要求更大
