# LiveData  
- 基本结构，生命周期感知，数据设置与分发，粘性行为，订阅管理与线程安全  
1. 基本结构，LiveData 持有一个数据值 mData 和一个版本号 mVersion ，观察者记录其最后一次接受的版本号  
2. 生命周期感知，观察者通过 observe 注册时，会将 owner 的 Lifecycle 传入，创建 LifecycleBoundObserver ，监听生命周期时事件，只有只有在 Lifecycle 为 STARTED 或 RESUMED（即 ACTIVE）时，才会接收数据回调。  

# Room 
- 编译器代码生成，SQLite 映射，类型适配与转换，线程与事务管理，变更监听和自动刷新。  
1. 编译器注解处理与代码生成，注解处理器可以在编译的时候读取 room 相关的注解，进行模型校验并生成类型安全的代码，生成具体的 Impl 把注解转成预编译的 SQL，实现RoomDatabase，并检查数据库提前在编译器报错。  
2. 类型映射，@TypeConverter 将复杂类型序列化为可存储的基本类型  
3. 默认禁止在主线程做 I/O 操作  
4. 可以返回 Flow，LiveData 这样的数据

# ViewModel  
- VM的生命周期  
ViewModel 的生命周期与其持有者的作用域相绑定，而不是与界面视图绑定。只要作用域还在，ViewModel 就会存活，直到作用域被清理时才会被销毁并调用 onCleared()。  

# DataBinding  
- 原理  
    1. 编译器注解处理与代码生成，  
    xml文件会生成一个 Binding 类，继承自 ViewDataBinding ，包括布局中生命的字段 setter ，view的缓存引用，表达式求值，布局中的 @{} 表达式在编译期被解析，转成强类型的 Java/Kotlin 代码，避免运行时反射  
    2. 运行期绑定与脏标记机制  
    当调用binding.setVm或者给LiveData 设置新值时，DataBinding会标记相应的dirty flag,请求一次rebind,在下一个帧对相应的View做最小化更新，避免整树刷新  
    3. 可观察数据与监听接入  
    Observable 及 LiveData等都可以生成对应的监听代码，一旦变化会触发更新  
    4. 双向绑定，view的属性值可以和数据源互相同步。在这个过程中编译器生成两个方向的代码，  
    数据 -> View：像普通绑定一样在 executeBindings() 设置 text。  
    View -> 数据：为 EditText 等属性生成 Listener，监听用户输入变化再调用对应的 setter。   

# ViewBinding  
- 原理  
    ViewBinding 在编译期为每个布局生成一个包含强类型子视图引用的 Binding 类，运行时只在创建时进行一次 findViewById 并缓存，后续通过字段直接访问，做到类型安全、低开销、零反射的视图绑定。

# LifyCycle  
- Lifecycle只是定义了对观察者的存储操作。  状态的处理和最终的分发，是经过LifecycleRegistry遍历调用实现的。如果API的版本 大于等于 29，则使用LifecycleCallbacks这种方式。如果小于29，则通过添加一个透明的ReportFragment来监听生命周期。