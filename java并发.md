# 导致并发问题的原因

1. 导致可见性的原因是缓存
2. 导致有序性的原因是编译优化

# java提供的三个关键字和六项 Happens-Before规则

1. volatile
   原始语义就是禁用cpu缓存。

# Happens-Before

1. 程序的顺序性规则
   即程序前面对某个变量的修改一定对后续操作是可见的
2. volatile变量规则
   对一个volatile变量的写操作Happens-Before于后续
   对这个变量的读操作
3. 传递性
   这条规则是指如果 A Happens-Before B，且 B Happens-Before C，
   那么 A Happens-Before C。
4. 管程中锁的规则
   这条规则是指对一个锁的解锁 Happens-Before 于后续对这个锁的加锁。
5. 线程 start() 规则
   即主线程a启动子线程b后，子线程b看以看到主线程a启动b之前的操作
6. 线程join的规则
   调用join方法并执行成功后，那么该线程一定执行完毕了。

# 原子性

源头是因为线程切换,所以才有原子性问题,而操作系统是依赖cpu中断进行线程切换的.

# synchronized

静态方法锁class
非静态锁this

# 并发的问题
1. 安全性
2. 活跃性
3. 性能

## 安全性
就是避免线程出现,原子性,可见性,有序性问题.
1. 场景
   对线程读写共享内存

