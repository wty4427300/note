# 部分资料
https://mp.weixin.qq.com/s/kvZBbENeLoEjZUblb3ckaA

# go build

go build的时候默认不会安装目标代码包所依赖的代码包。当然，如果被依赖的代码包归档文件不存在，或者源码文件有了变化，那么他还是会被编译的。
强制编译的命令是go build -a。另外，如果不但要编译依赖包,还要安装他们的规定那个文件，可以使用可以加上-i。

可以用select，default配合使用判断通道有没有关闭

# 通道关闭

通道的关闭原则不要从接收端关闭通道，因为你不知道发送端还会不会继续发送。所以在1对1的情况下应该在发送端关闭通道
在多对1的情况下应该在最后一个活跃的发送端关闭通道。

# slice

就是动态数组，底层是一个数组，slice是数组的引用，slice的长度和容量都是可变的。
元素数量小于1024，两倍扩容。大于1024，每次扩容为原来的1.25倍。

# map

map的key是不可变的，所以key的类型必须是string，int，uint，uintptr，bool，complex64，complex128，float32，float64，unsafe.Pointer，time.Time，以及这些类型的指针。

map如何实现有序性，先使用slice排序key，然后根据slice里key顺序去取之。

# channel

内部是两个goroutine队列，一个goroutine负责发送，一个goroutine负责接收。
每当channel的读写操作超过了可缓冲的值，就会阻塞。那么当前goroutine就会被挂到
对应的队列上，直到有其他goroutine执行了相反的channel的读写操作。才会被唤醒。


