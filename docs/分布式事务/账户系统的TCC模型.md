## 账户系统TCC模型

### try 阶段

​	业务检查  资源预留

​	检查库存是否足够 , 检查卡里钱是否足够 , 如果不够要失败

​	冻结本次事务的库存 , 冻结本次转账的余额

### confirm 阶段

​	冻结消费

​	执行业务  处理 **try** 阶段冻结资源 

### cancel 阶段

​	回滚业务 , 释放 **try** 阶段预留的资源

---

高效模型高效在于 **confirm** 阶段扣钱服务无数据库操作

## 高效模型

​                  张三--------------转账20元-------------->李四

​                  扣钱服务--------转账20元------------->加钱服务

try :           扣张三20元            > >              什么都不做

confirm:        什么都不做            > >                 加20元

cancel:         回退20元            > >               什么都不做

 

## 标准模型

​                  张三--------------转账20元-------------->李四

​                  扣钱服务--------转账20元------------->加钱服务

try :           冻结张三20元    > >              什么都不做

confirm:        扣冻结20元      > >          加20元

cancel:      回退冻结的20元     > >            什么都不做



## 隔离级别推演

事务安全的并发执行

​                  张三 100 --------------转账20元-------------->李四 100

​                  张三 100 --------------转账20元-------------->李四 100

​            

try :         冻结20       冻结20      -->  冻结40 余额60    > >     什么都不做  什么都不做 -->  100元 

confirm: 扣冻结20   扣冻结20  -->   冻结0 余额 60     > >   加20           加20      -->  140元

cancel :   还冻结20   还冻结20  -->   冻结0 余额 100   > >   什么都不做  什么都不做 -->  100元 

