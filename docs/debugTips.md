## 行断点 -- Line Breakpoints

使用最多的一种断点 , 代码执行到该行就启用断点



## 接口方法断点 -- Method Breakpoints

(**注意如果在方法上加断点 , 当方法过多时会导致服务启动时间大大延长**)

在接口的定义的`method()`方法上打断点 , IDEA 会自动在该接口当前所运行的实现类的`method()`方法上启用断点 , 源码解读时可更容易找到接口所调用的实现类



## 字段断点 -- Field Watchpoints

类中字段读写监控 , 字段中值发生修改时启用断点



## 异常断点 -- Exception Breakpoints

在`Breakpoints`中添加`Java Exception Breakpoints` , 可添 加指定的异常 , 出现指定异常时才启用断点



## 强制返回 -- Force Return

使用`Force Return`而不是`Drop Frame`来跳过不想执行的方法内剩余步骤



## Lambda表达式调用链追踪 -- Trace Current Stream Chain

在使用`stream`式编程`debug`时使用 `Trace Current Stream Chain` 来使数据转变可视化
