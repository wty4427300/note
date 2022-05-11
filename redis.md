redis的五种基础类型
string
list
hash
set
sorted set

redis缓存失效原因
1.有效期过短
2.内存满了（lru/lfu）
3.重启rdb全量持久化 aof增量持久化
(save 多少秒 多少个) () 

关于redis分布式锁的问题
1.上锁
set命令要用set key value px milliseconds nx
jedis.set(String key, String value, String nxxx, String expx, int time)
需要5个参数为了解决四个问题

1.**互斥性。**在任意时刻，只有一个客户端能持有锁。

2.**不会发生死锁。**即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。

3.**具有容错性。**只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。

4.**解铃还须系铃人。**加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。

第一个为key，我们使用key来当锁，因为key是唯一的。

第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？
原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，
我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成。

第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；

第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。

第五个为time，与第四个参数相呼应，代表key的过期时间。

2.解锁
String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
if (RELEASE_SUCCESS.equals(result)) {
    return true;
}

首先获取锁对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）

使用eval()可以保证原子性

3.缺陷
如果我们在master节点获取了锁，且锁还没有没有被同步到slave节点，此时如果master节点出现错误，slave节点升级为master节点就会导致锁丢失

4.redlock

1.获取当前Unix时间，以毫秒为单位。

2.依次尝试从5个实例，使用相同的key和具有唯一性的value（例如UUID）获取锁。
当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。
例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。
如果服务器端没有在规定时间内响应，客户端应该尽快尝试去另外一个Redis实例请求获取锁。

3.客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。
当且仅当从大多数（N/2+1，这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。

4.如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间）
，客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功，
防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）。

