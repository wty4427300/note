zookeeper的介绍
1.分布式的高性能协调服务，它的特点就是数据是存储在内存中的，持久化实现在日志中。它的内存类似树状结构。可以帮助我们实现统一的配置中心，服务注册，分布式锁等。
2.每个目录成为Znode节点，但它不同于文件系统，它既可以视为文件夹，也可以视为文件存在数据，但是平时我们还是得叫它节点。
3.同一节点下子节点不能同名，且命名有规范的，它的路径是没有相对路径的概念的，都是绝对路径，任何都是以“/”开始。
4.zookeeper的数据分为四种类型：持久，顺序，临时，临时顺序。
5.zxid每次写请求的事务id顺序且唯一。version numbers版本号，对节点的写请求都会导致该节点的三种版本号增加。
6.watch的监听机制，可以监控znode的变化。watch的重要特性1.仅一次性：watch触发后会立即删除，要持续监听变化的话就要持续提供设置watch，这也是watch的注意事项。
2.有序性客户端先得到watch通知才可以查看变化结果。

1.顺序一致性，保证客户端操作是按顺序生效的。
2.原子性，更新成功或失败。没有部分结果。
3.单个系统映像，无论连接到那个服务器，客户端都将看到相同的内容。
4.可靠性，数据的变更不会丢失，除非被客户端都将看到相同的内容
5.及时性，保证系统的客户端读取到的数据是最新的。
