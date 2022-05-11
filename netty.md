# 开始
EventLoop 就是一个单例的线程池，里面含有一个死循环的线程不断的做着3件事情：监听端口，处理端口事件，处理队列事件。
每个 EventLoop 都可以绑定多个 Channel，而每个 Channel 始终只能由一个 EventLoop 来处理。

然后进来的所有的任务都会，先判断当前线程是不是这个单例线程，是的话就放进一个多生产者，单消费者的无界队列

nioeventloop的的父类singleThreadEventExecutor的startThread方法
这个方法先判断线程是否启动过了，保证eventloop只有一个线程，如果没有启动过，则用cas将state状态改为st_started
也就是已经启动然后调用doStartThread方法。

然后开始执行

# 一些需要注意的点(改写的还是要写一写的)
channelPipeline就是一个存储channelHandler的双向链表

# Bootstrap
主要干了三件事
1.配置线程池
2.channel初始化（服务端还是客户端）



