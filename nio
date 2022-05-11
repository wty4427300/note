客户端 到 buffer 到channel 到selector 到线程 开始执行
selector.open获取一个选择器

1.当客户端连接时，会通过serversocketchannel得到对应的socketchannel
2.selector进行监听 select()方法（可使用阻塞或者非阻塞方法），返回有事件发生的通道的个数。
3.将得到的channel注册到selector
4.注册后得到一个selectionkey,会被selector管理，以集合的方式
5.四种事件 读事件 写事件 连接成功事件 新连接事件
6.获取selectionkey
7.通过key获取channel
7.通过channel完成业务的处理
