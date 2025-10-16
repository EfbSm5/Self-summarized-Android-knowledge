# Flow   
# 协程  
-  是什么    
    “协程是运行在用户态的轻量执行单元，suspend 函数在挂起点让出线程、异步完成后原地恢复，结合结构化并发实现高并发又易管理。”
- 协程的上下文  
    CoroutineContext 就是协程的配置与控制面板，包含 Job 和 Dispatcher 等信息，并在挂起调用链里隐式传递。  
- Continuation  
    Continuation 表示‘如何从挂起点继续’，持有状态机的帧与 CoroutineContext。异步完成时调用 resumeWith 恢复执行，是把回调式 API 封装为挂起函数的桥梁