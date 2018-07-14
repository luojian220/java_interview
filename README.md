# java_interview 面试题整理 201805

    本面试题整理，主要根据本人或同事找工作过程中遇到的问题，同时参考了网友整理：https://mp.weixin.qq.com/s/4J-x6djj3EfBBiPoU1P0kA，后续还会持续更新。 适用于Java常见面试题。

## 1.中间件相关

### 1.1 Memcache和Redis区别？为什么单线程redis比多线程memcache效率更高？
    他们都是存储在内存中的nosql缓存数据库，最大的区别为：
        a.redis支持多种数据结构；而memcache只支持简单的k/v缓存；
        b.redis是单线程操作，所有命令串行执行；memcache是多线程操作，需要考虑数据并发执行问题，所以修改数据操作需加锁，同时读性能更优；
        c.redis支持pub/sub消息订阅机制，memcache不支持；
        d.redis支持数据持久化，而memcache不支持；
    redis效率高于memcache的原因是：
        memcache多线程需要考虑并发锁的消耗；
        memcache大量线程切换的开销；
        memcache每个线程会有自己的内存拷贝，而redis不需要这些操作；
        
### 1.2 redis有什么数据类型？ 都在哪些场景下使用？
    redis数据类型包括： string（字符串）,hash（哈希）,list（列表）,set（集合）,zset（有序集合）
    string ：存用字符串表示的二进制
    hash：存java对象，如用户对象;
    list: 是双向链表结构，存对象的集合;
    set: 存无序集合，元素不能重复;
    zset: 存有序集合，元素不能重复;
    
### 1.3 redis 持久化方式？各自优缺点？具体底层实现？
    持久化有两种方式：
    RDB:RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。原理是将Reids在内存中的数据库记录定时dump到磁盘上的RDB持久化。
    AOF:AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。原理是将Reids的操作日志以追加的方式写入文件。
    优缺点：
    1.RDB备份方式只有一个.rdb文件，利于备份和恢复；
    2.RDB备份方式，通过fork子进程来执行持久化操作，避免主进程执行IO操作；
    3.RDB相比于AOF机制，如果数据集很大，RDB的启动效率会更高
    4.RDB无法100%保证数据的高可用性，即可能出现数据丢失。因为在定时持久化之前宕机，没来得及写入磁盘的数据将会丢失
    5.RDB通过fork子进程来完成持久化工作，当数据集较大时，子进程消耗资源太大，可能导致整个服务器停止服务几百毫秒，甚至1秒钟。
    6.AOF备份方式，数据安全性更高，因为每次修改都会同步持久化到磁盘中；但是这样效率低
    7.AOF对日志文件的写入操作采用的是append模式，因此在写入过程中出现宕机，在下次启动之前要通过redis-check-aof工具解决数据一致性问题
    8.AOF备份方式，在日志过大时，可以通过rewrite机制压缩日志文件。
    9.AOF备份文件是一个格式清晰、易于理解的操作记录，可以作为历史操作记录。
    
### 1.4 说下dubbo的实现过程？注册中心挂了可以继续通信吗？
    实现过程
        1.解析服务，将配置的dubbo服务解析，转化成url格式，默认是按dubbo协议解析。
        2.暴露服务，向注册中心注册服务
        3.引用服务，通过registry://形式的协议，请求先到注册中心，注册中心选取调用一个提供者执行服务，包装返回结果返回。
    注册中心挂后服务可以继续调用，因为启动dubbo时，消费者会从zk拉取注册的生产者的地址接口等数据，缓存在本地。每次调用时，按照本地存储的地址进行调用
    
### 1.5 zk的实现原理？zk都可以做什么？
    ZooKeeper 是一个开源的分布式协调服务，由雅虎创建，是 Google Chubby 的开源实现。分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。
    在 ZooKeeper 中，一个客户端连接是指客户端和 ZooKeeper 服务器之间的TCP长连接。从第一次连接建立开始(客户端启动)，客户端会话的生命周期也开始了，通过这个连接，客户端能够通过心跳检测和服务器保持有效的会话，也能够向 ZooKeeper 服务器发送请求并接受响应，同时还能通过该连接接收来自服务器的 Watch 事件通知。
     
### 1.6 dubbo支持哪些序列化协议？hession?说下hession的数据结构？
    Dubbo支持dubbo、rmi、hessian、http、webservice、thrift、redis等多种协议。其中dubbo协议序列化方式为Hessian二进制
    hessian是由caucho提供的一个基于二进制-RPC实现的远程通讯类库。需要通过hessian本身提供的api 来发起请求，hessian通过其自定义的串行化机制将请求信息序列化，产生二进制流，hessian基于http协议进行传输。响应端根据hessian提供的API来接收请求。hessian根据其私有的串行化机制来讲请求信息反序列化，传递给使用者的已经是请求信息对象。处理完毕后直接返回，hessian将处理结果序列化后返回给调用端。
    
### 1.7 dubbo负载均衡策略和高可用策略都有哪些？动态代理策略呢？
    负载均衡策略：
        随机，按权重设置随机概率。
        轮循，按公约后的权重设置轮循比率。  
     高可用策略：
        权重调节；
        zk心跳服务检测，剔除无效服务；
 
### 1.8 知道netty吗？netty可以干嘛，nio ,bio ,aio 都是什么？有什么区别？ 

     
## 2.java语言相关

### 2.1 hashcode相等两个对象一定相等吗？equals呢？相反呢？
    hashcode相等，对象不一定相当，因为equesl方法不一定为true；相反equesl方法为 true，则一定hashcode一定相等。
    java一般约定我们比较两个对象是否相等，可以先比较hashcode，若相等再用equals比较。所以一般重写equals方法，同时也要重写hashcode
    
### 2.2 hashmap、hastable底层实现有什么区别？ConcurrentHashMap呢
    HashMap和Hashtable都实现了Map接口，主要的区别有：hashtable线程安全，另外HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。

### 2.3 hashmap和treemap有什么区别？底层数据结构是什么？
    HashMap：基于哈希表实现。使用HashMap要求添加的键类明确定义了hashCode()和equals()[可以重写hashCode()和equals()]，为了优化HashMap空间的使用，您可以调优初始容量和负载因子。底层结构：由数组+链表组成
    TreeMap：基于红黑树实现。TreeMap没有调优选项，因为该树总处于平衡状态。
    
### 2.4 hashmap其中参数负载因子 的实现原理是什么？


### 2.5 hashmap每次扩容，增加多少容量? 实现原理


### 2.6 有哪几种线程安全自增方式？
   AtomicInteger（轻量级） 、 加锁自增 、 redis自增等
   
### 2.7 线程池用过吗？都有什么参数？底层如何实现？


### 2.8 synchronized和lock什么区别？synchronized什么情况下是对象锁？什么时候是全局锁？为什么？
    synchronized和lock区别
        1.synchronized 是Java的关键字，是Java的内置特性，在JVM层面实现了对临界资源的同步互斥访问;lock是JDK层面实现的接口；
        2.lock有更多操作方法，可以灵活的选择是否阻塞，中断时间；
        3.lock支持读写锁
        4.lock需要代码显示释放锁，synchronized不需要代码释放
    synchronized对象锁
        修饰实例方法、synchronized (this)用法、synchronized (非this对象)
    synchronized类锁
        修饰静态方法、synchronized (类.class)
    实现原理：
    
### 2.9 ThreadLocal是什么？底层如何实现？ThreadLocal存储时，key是什么？
    ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量
    底层实现：
        线程副本都存储再ThreadLocalMap中，而ThreadLocalMap这个类型是ThreadLocal类的一个内部类；
        ThreadLocalMap的Entry继承了WeakReference，并且使用ThreadLocal作为键值
        首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）
        初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals
        　
        
        1、每个Thread对象内部都维护了一个ThreadLocalMap这样一个ThreadLocal的Map，可以存放若干个ThreadLocal。
        
        /* ThreadLocal values pertaining to this thread. This map is maintained
         * by the ThreadLocal class. */
        ThreadLocal.ThreadLocalMap threadLocals = null;
        2、当我们在调用get()方法的时候，先获取当前线程，然后获取到当前线程的ThreadLocalMap对象，如果非空，那么取出ThreadLocal的value，否则进行初始化，初始化就是将initialValue的值set到ThreadLocal中。
        
        public T get() {
            Thread t = Thread.currentThread();
            ThreadLocalMap map = getMap(t);
            if (map != null) {
                ThreadLocalMap.Entry e = map.getEntry(this);
                if (e != null)
                    return (T)e.value;
            }
            return setInitialValue();
        }
    　　3、当我们调用set()方法的时候，很常规，就是将值设置进ThreadLocal中。
    
    　　4、总结：当我们调用get方法的时候，其实每个当前线程中都有一个ThreadLocal。每次获取或者设置都是对该ThreadLocal进行的操作，是与其他线程分开的。
    
    　　5、应用场景：当很多线程需要多次使用同一个对象，并且需要该对象具有相同初始化值的时候最适合使用ThreadLocal。
    
    　　6、其实说再多也不如看一下源码来得清晰。如果要看源码，其中涉及到一个WeakReference和一个Map，这两个地方需要了解下，这两个东西分别是a.Java的弱引用，也就是GC的时候会销毁该引用所包裹(引用)的对象，这个threadLocal作为key可能被销毁，但是只要我们定义成他的类不卸载，tl这个强引用就始终引用着这个ThreadLocal的，永远不会被gc掉。b.和HashMap差不多。

### 2.10 volatile的工作原理？
    多线程并发执行时，每个线程都会主内存拷贝一份变量副本到线程工作内存 ，线程对变量的修改只会修改线程工作内存中的变量，后续再同步到主内存中，这种中间会导致一个线程修改了变量，而其他线程不知道，还是在旧值上修改，由此形成了线程安全性问题。
    上面描述的问题中主要说的只是是线程的可见性问题，线程安全性问题解决需要解决三个问题：原子性、可见性、有序性。
    volatile不能保证操作的原子性，可以保证可见性、和有序性
    使用volatile必须具备以下2个条件：
    1）对变量的写操作不依赖于当前值
    2）该变量没有包含在具有其他变量的不变式中
    
### 2.11 事务的特性
    ⑴ 原子性（Atomicity）
    　　原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响。
    ⑵ 一致性（Consistency）
    　　一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。
    　　拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。
    ⑶ 隔离性（Isolation）
    　　隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。
    　　即要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。
    　　关于事务的隔离性数据库提供了多种隔离级别，稍后会介绍到。
    ⑷ 持久性（Durability）
    　　持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。

### 2.12 事务的隔离级别
    ① Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。
    ② Repeatable read (可重复读)：可避免脏读、不可重复读的发生。
    ③ Read committed (读已提交)：可避免脏读的发生。
    ④ Read uncommitted (读未提交)：最低级别，任何情况都无法保证。
    
    1，脏读
    　　脏读是指在一个事务处理过程里读取了另一个未提交的事务中的数据。
    2，不可重复读
    　　不可重复读是指在对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询间隔，被另一个事务修改并提交了。
    　　例如事务T1在读取某一数据，而事务T2立马修改了这个数据并且提交事务给数据库，事务T1再次读取该数据就得到了不同的结果，发送了不可重复读。
    　　不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。
    3，虚读(幻读)
    　　幻读是事务非独立执行时发生的一种现象。例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这时事务T2又对这个表中插入了一行数据项，
    而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发现还有一行没有修改，其实这行是从事务T2中添加的，就好像产生幻觉一样，这就是发生了幻读。
    　　幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。

## 3.实际问题解决及算法问题
    
### 3.1 有一批100万的黑名单手机号数据（存放在表中，这个表后续可能每天还会有部分变动），需要针对数据对外提供一个接口，判断传入手机号是否在黑名单中？
    第一层思路，对数据分片、分组处理，提升查询速度
    第二层思路，数据如何存放，放入内存数据库redis（也可以在表中分片），提高效率，且redis的hash数据结构较适合存放。
    
    我的解决思路： 
    * 将数据灌入redis缓存中，用hash数据结构存放，按手机号前5位作为一个hash存放单元（最多可以分成10000个单元），每个hash单元中hkey(hash子key)为手机号，hval(hash子value)为黑名单记录，
    用HGET key field/ HEXISTS key field命令来判断是否在黑名单中；
    * 另外起一个线程，按黑名单记录更新时间，刷新变动的黑名记录到redis缓存中，记录中黑名单删除做逻辑删除标记，新增/修改都更新修改时间。
    
### 3.2 对大数据记录表做水平分表，是否有相关经验，处理思路是怎样的？
    * 对单表大记录做拆库拆表，只能做水平拆分，可以按主键取hash值，然后取n的模，即水平拆分为n个表，对n个表中具体分几个库也可以按一定规则决定。
    
### 3.3 对mq整个发送、消费情况做一个监控，包括发送成功记录数/失败数/消费失败数等
    * 思路1：在发送端/消费端对日志成功失败操作打日志，再收集日志存放到es中，做相关的分析报表。 Elasticsearch , Logstash, Kibana做为一个解决思路。
    
        

