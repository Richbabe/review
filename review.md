#计算机网络
## HTTP
###HTTP METHOD
	OPTIONS: 查询支持的方法
	TRACE: 查询通信路径（经过多少代理），一般不用
	CONNECT: 使用 TLS/SSL 加密传输
###HTTP STATUS
	201，新建数据成功
	301/302，永久/临时转移
	304 Not Modified(资源未过期，如果过期直接200返回新的资源)
	401 未授权
	403 不允许访问
###HTTP2.0
二进制解析 头部压缩 多路复用（连接共享） SSL 二进制分祯层 服务器推送

## TCP
###和 UDP 的区别
	1.基于连接与无连接
  	2.TCP要求系统资源较多，UDP较少； 
  	3.UDP程序结构较简单 
  	4.流模式（TCP）与数据报模式(UDP); 
  	5.TCP保证数据正确性，UDP可能丢包 
  	6.TCP保证数据顺序，UDP不保证 
  	7.UDP 编程不需要 CONNECT 和 LISTEN
  	8.具体的处理方式需要用户自己在应用层编写
  
####udp应用场景
	1.面向数据报方式
  	2.网络数据大多为短消息 
  	3.拥有大量Client
  	4.对数据安全性无特殊要求
  	5.网络负担非常重，但对响应速度要求高

###流量控制（解决发送方和接收方速度不匹配的问题，接收方控制）

TCP的流量控制是利用滑动窗口机制实现的，接收方在返回的ACK中会包含自己的接收窗口的大小，以控制发送方的数据发送。

https://blog.csdn.net/seu_calvin/article/details/53198282
###拥塞控制（控制全局网络的速率）

防止过多的数据注入到网络中，这样可以使网络中的路由器或链路不致过载。

####过程：
慢开始-》到达ssthresh-》拥塞避免加法增大-》到达堵塞-》慢开始或快恢复（ssthresh 为当前的拥塞窗口的一半）

####快重传：
快重传是指，如果发送端接收到3个以上的重复ACK，不需要等到重传定时器溢出就重新传递，所以叫做快速重传，而快速重传以后，因为走的不是慢启动而是拥塞避免算法，所以这又叫做快速恢复算法。

如果没有快速重传和快速恢复，TCP将会使用定时器来要求传输暂停。在暂停这段时间内，没有新的数据包被发送。所以快速重传和快速恢复旨在快速恢复丢失的数据包。

####ZWP（Zero Window Probe）：
也就是说，发送端在窗口变成0后，会发ZWP的包给接收方，让接收方来ack他的Window尺寸，一般这个值会设置成3次，第次大约30-60秒（不同的实现可能会不一样）。如果3次过后还是0的话，有的TCP实现就会发RST把链接断了。



### notions
检验和 定时器（超时重传，持续计时器（但是当某个ACK报文丢失了，就会出现A等待B确认，并且B等待A发送数据的死锁状态，为了解决这种问题，TCP引入了持续计时器（Persistence timer），当A收到rwnd=0时，就启用该计时器，时间到了则发送一个1字节的探测报文，询问B是很忙还是上个ACK丢失了，然后B回应自身的接收窗口大小，返回仍为0（A重设持续计时器继续等待）或者会重发rwnd=x），保活，结束时间等待） 序号 确认 窗口 流水线 (回退 N 步、选择重传)  快速重传 流量控制 拥塞控制(慢启动，拥塞避免，快速恢复)
### status
client(active)
SYN_SENT->ESTABLISHED->FIN_WAIT_1->FIN_WAIT_2->TIME_WAIT->CLOSED
server(passive)
LISTEN->SYN_RCVD->ESTABLISHED->CLOSE_WAIT->LAST_ACK->CLOSED

为什么三次握手：如果两次握手，服务方发送 SYN/ACK 无法知道A是否已经接收到自己的同步信号，如果这个同步信号丢失了，A和B就B的初始序列号将无法达成一致。所以要客户端要再确定一次。如果服务方没有收到，会重传，一直到收到为止（SYN FLOOD？缩短SYN Timeout或者设置SYN Cookie）

补充：
第一个包，即A发给B的SYN 中途被丢，没有到达B
A会周期性超时重传，直到收到B的确认
第二个包，即B发给A的SYN +ACK 中途被丢，没有到达A
B会周期性超时重传，直到收到A的确认
第三个包，即A发给B的ACK 中途被丢，没有到达B
A发完ACK，单方面认为TCP为 Established状态，而B显然认为TCP为Active状态：
a. 假定此时双方都没有数据发送，B会周期性超时重传，直到收到A的确认，收到之后B的TCP 连接也为Established状态，双向可以发包。
b. 假定此时A有数据发送，B收到A的 Data + ACK，自然会切换为established 状态，并接受A的 Data。
c. 假定B有数据发送，数据发送不了，会一直周期性超时重传SYN + ACK，直到收到A的确认才可以发送数据

为什么四次挥手？FIN->（ACK->FIN）->ACK。等待数据发送完关闭。

为什么 TIME_WAIT？防止上一次连接中的包，迷路后重新出现，影响新连接。在进行关闭连接四路握手协议时，最后的ACK是由主动关闭端发出的，如果这个最终的ACK丢失，服务器将重发最终的FIN，因此客户端必须维护状态信息允许它重发最终的ACK。

tcp_tw_reuse，这个参数作用是当新的连接进来的时候，可以复用处于TIME_WAIT的socket。默认值是0。

##Restuful-API

动词
GET（SELECT）：从服务器取出资源（一项或多项）（提交长度有限制，POST 没有,TCP 发送一次）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
DELETE（DELETE）：从服务器删除资源。


#操作系统

进程通讯方式：文件映射，共享内存，管道，动态链接库， SOCKET。

进程（资源分配的最小单位）和线程（程序执行的最小单位）的区别：进程拥有自己的一套数据，线程则共享数据；进程是程序执行的实体，线程是进程的实体。

进程状态：就绪，执行，堵塞

（就绪 执行）（执行 堵塞）（堵塞 就绪）

CPU调度算法：

先来先服务调度算法

短作业(进程)优先调度算法

优先权调度算法的类型（非抢占式优先权算法，抢占式优先权算法）

高响应比优先调度算法

时间片轮转法


分页：空间小，不容易产生碎片，易于管理，缺点是长度与程序逻辑大小无关

分段：按照程序的自然分解划分的长度，可以动态改变的区域。

段页式存储：
用分段方法来分配和管理虚拟存储器。程序的地址空间按逻辑单位分成基本独立的段，而每一段有自己的段名，再把每段分成固定大小的若干页。

用分页方法来分配和管理实存。即把整个主存分成与上述页大小相等的存储块，可装入作业的任何一页。程序对内存的调入或调出是按页进行的。但它又可按段实现共享和保护。

分页和分段有许多相似之处,比如两者都不要求作业连续存放.但在概念上两者完全不同,主要表现在以下几个方面:

(1)页是信息的物理单位,分页是为了实现非连续分配,以便解决内存碎片问题,或者说分页是由于系统管理的需要.段是信息的逻辑单位,它含有一组意义相对完整的信息,分段的目的是为了更好地实现共享,满足用户的需要.

(2)页的大小固定,由系统确定,将逻辑地址划分为页号和页内地址是由机器硬件实现的.而段的长度却不固定,决定于用户所编写的程序,通常由编译程序在对源程序进行编译时根据信息的性质来划分.

(3)分页的作业地址空间是一维的.分段的地址空间是二维的.

死锁：

产生条件：

1）互斥条件：指进程对所分配到的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，则请求者只能等待，直至占有资源的进程用毕释放。
2）请求和保持条件：指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放。
3）不剥夺条件：指进程已获得的资源，在未使用完之前，不能被剥夺，只能在使用完时由自己释放。
4）环路等待条件：指在发生死锁时，必然存在一个进程——资源的环形链，即进程集合{P0，P1，P2，···，Pn}中的P0正在等待一个P1占用的资源；P1正在等待P2占用的资源，……，Pn正在等待已被P0占用的资源。

方法：
- 撤销所有的死锁进程 
- 进程回退（Roll Back）再启动 
- 按照某种原则逐一撤销死锁的进程直到。。。 
- 按照某种原则逐一抢占资源（资源被抢占的进程必须回退到之前对应的状态），直到。。。



存储器结构对于程序性能的影响

	空间局限性
	
	时间局限性
	
让最常见的情况运行的最快
让每个循环内部缓存命中的数量最多

内存置换算法
1.最佳置换算法(OPT)：最佳(Optimal, OPT)置换算法所选择的被淘汰页面将是以后永不使用的，或者是在最长时间内不再被访问的页面,这样可以保证获得最低的缺页率。但由于人们目前无法预知进程在内存下的若千页面中哪个是未来最长时间内不再被访问的，因而该算法无法实现。
2.先进先出(FIFO)页面置换算法：优先淘汰最早进入内存的页面，亦即在内存中驻留时间最久的页面。
3.最近最久未使用(LRU)置换算法：选择最近最长时间未访问过的页面予以淘汰，它认为过去一段时间内未访问过的页面，在最近的将来可能也不会被访问。该算法为每个页面设置一个访问字段，来记录页面自上次被访问以来所经历的时间，淘汰页面时选择现有页面中值最大的予以淘汰。

##LINUX系统
### 文件格式 EXT2/EXT3

读取上使用 INNODE 和 BLOCK 作为索引，日志系统
硬链接不能连接目录

###进程管理

某个进程杀不掉的原因（进入内核态，忽略kill信号）
kill 命令1表示重启，9表示强制退出，15表示正常结束。
nohup 永久执行（不随着用户挂起而退出）;&：在后台执行（用户挂起时，则自动退出）
kill和 killall

##硬链接和软连接区别
innode信息才是文件系统识别的。
若一个 innode 号对应多个文件名，则称这些文件为硬链接。换言之，硬链接就是同一个文件使用了多个别名。
INNODE不同，相当于快捷方式（ln 硬链接，ln -s 软连接）
硬链接不能连接文件夹，会产生死循环，复杂度提高

#数据结构与算法
堆排序：适合大数据，空间复杂度为 O（1），O（nlogn），不稳定

快速排序：大数据量时比堆排序快，对于数组，快速排序每下一次寻址都是紧挨当前地址的，而堆排序的下一次寻址和当前地址的距离比较长。空间复杂度为 O（n）或者 O（lgn），主要消耗在递归上。不稳定。

归并：稳定性排序 O（nlogn）

TOPK

红黑树

1.节点为红色或者黑色
2.根节点为黑色
3.叶子节点为黑色
4.红色的子节点必须是黑色
5.从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

约束了关键性质：从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。

HASH 树

#数据库
ACID：原子性（Atomicity）:整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
一致性（Consistency）:一个事务可以封装状态改变（除非它是一个只读的）。事务必须始终保持系统处于一致的状态，不管在任何给定的时间并发事务有多少。例如银行转账总额度不变
隔离性（Isolation）：隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。
持久性（Durability）：在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

索引缺点：占用物理空间，降低维护速度。经常查询的数据才需要索引

内连接，左连接，右连接

B 树和 B+树(树的所有叶结点构成一个有序链表，可以按照关键码排序的次序遍历全部记录;非叶结点仅具有索引作用，跟记录有关的信息均存放在叶结点中。)
（优点：1.由于B+树在内部节点上不包含数据信息，因此在内存页中能够存放更多的key。 数据存放的更加紧密，具有更好的空间局部性。因此访问叶子节点上关联的数据也具有更好的缓存命中率。
2.B+树的叶子结点都是相链的，因此对整棵树的便利只需要一次线性遍历叶子结点即可。而且由于数据顺序排列并且相连，所以便于区间查找和搜索。而B树则需要进行每一层的递归遍历。相邻的元素可能在内存中不相邻，所以缓存命中性没有B+树好。）

三范式：

第一范式：每一列都不可分割，原子性

第二范式：任意字段都只依赖于表中的同一个字段

第三范式：任何非主属性不依赖于其他非主属性（各字段之间相互独立）

反范式设计：适当冗余，缩短操作时间（空间换时间），对不可改变的信息无需范式化设计，例子：国籍信息可以跟用户信息放在一起，无需单独放在一张表中

读未提交：(Read Uncommitted)
读已提交（Read Committed） 大多数数据库默认的隔离级别
可重复读（Repeatable-Read) mysql数据库所默认的级别
序列化（Serializable）

脏读：在一个进程的事务当中，我更改了其中的一行数据，但是我修改完之后就释放了锁，这时候另一个进程读取了该数据，此时先前的事务是还未提交的，直到我回滚了数据 ，另一个进程读的数据就变成了无用的或者是错误的数据。我们通常把这种数据叫做脏数据，这种情况读出来的数据叫做賍读。

不可重复读：同事务中两次读取数据不一致（这是由于查询时系统中其他事务修改的提交而引起的。比如事务T1读取某一数据，事务T2读取并修改了该数据，T1为了对读取值进行检验而再次读取该数据，便得到了不同的结果。）

幻读：一个事务读取或更新了表里的所有行，接者又有另一个事务往该表里插入一个新行，在事务提交后。原来读取或更改过数据的事务又第二次读取了相同的数据，这时候这个事务中两次读取的结果集行数就不一样。原来更新了所有行，而现在读出来发现竟然还有一行没有更新。这就是所谓的幻读。

幻读和不可重复读（Read Committed）都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。

MYISAM：不支持事务，不支持外键，非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快；
支持全文索引。

INNODB：支持事务，支持外键，聚集索引（数据文件是和索引绑在一起的，必须要有索引，但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大），不保存表的具体行数，执行select count(*) from table时需要全表扫描。不支持全文索引。

如何选择：
1. 是否要支持事务，如果要请选择innodb，如果不需要可以考虑MyISAM；
2. 如果表中绝大多数都是只读查询，可以考虑MyISAM，如果既有读写也挺频繁，请使用InnoDB。
3. 系统奔溃后，MyISAM恢复起来更困难，能否接受； 
4. MySQL5.5版本开始Innodb已经成为Mysql的默认引擎(之前是MyISAM)，说明其优势是有目共睹的，如果你不知道用什么，那就用InnoDB，至少不会差。

索引使用情况：

1.查询like “str%”，select 为主键时有用（innodb）。
2.复合索引，如果索引列不是复合索引的第一部分，则不使用索引（即不符合最左前缀）
3.用or分割开的条件，如果or左右两个条件中有一个列没有索引，则不会使用索引。

意向锁

drop:删除整个表（包括结构）

delete:每次删除一行，并在事务中保存。表和索引不会删掉

truncate:全删，不会再事务中保存，执行速度快。表和索引会被删掉

explain:type(all全表扫描，index（使用了索引），range（范围操作），const（主键索引），ref，system（一般只有一行数据，直接返回） 都是极好的)

null 》system》const》range》index》all

什么样的字段适合建立索引？
在where条件后面作为查询的字段需要建立索引。
在（排序order 范围range 分组group）这些情况下也是可以使用索引的，所以在排序字段后面加上索引也是可以的。
注意：在where条件后也不是所有的字段都建立索引，因为索引本身也是有开销的。


#JAVA

##基础

IO（面向流）/NIO（Selector, Channel, Buffer，面向缓冲区）：NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂；IO: 一个典型的IO服务器设计- 一个连接通过一个线程处理.

instanceof：class可以是object对象的父类，自身类，不能是子类。在前两种情况下result的结果为true，最后一种为false。但是class为子类时编译不会报错。运行结果为false。result = object instanceof class 


new Integer(1) 和Integer a = 1不同，前者会创建对象，存储在堆中，而后者因为在-128到127的范围内，不会创建新的对象，而是从IntegerCache中获取的。那么Integer a = 128, 大于该范围的话才会直接通过new Integer（128）创建对象，进行装箱。(https://blog.csdn.net/wangyang1354/article/details/52623703)

为什么要同时重写 hashcode 和 equals，不同时重写会出现哪些问题？JAVA API 规定，必须相同，否则通过 MAP 操作，就无法获得应该的结果。同样的对象做 KEY 键，不重写 HASHCODE 的话无法返回同一个对象。


JAVA8 新特性：lambda表达式；默认方法；Stream API；

类型转换：int 到 float，long 到 float 和 double 会损失精度

字符串 length 返回的是代码单元的数量，codePointCount返回代码点

StringBuffer 多线程

StringBuilder 单线程

类之间的关系：is-a(继承), has-a(聚合),use-a(依赖)

final:

类不可继承（方法，不包括域）；方法不可重写（优化处理，内联）；修饰参数时不可改变值

static:

修饰内部类，不可修饰外部类

public>protected>default>private

接口：没有实例域的抽象类

重载是静态的（编译时确定的），重写是动态的（运行时确定的）

ERROR（发生时会选择线程终止） 和 Exception（运行时异常（运行的时候才发现有错）和非运行时异常（需要 try 和 catch））：前者很严重，不可恢复，后者可以通过捕捉后恢复。

集合：

Collection:

	List:
	
		ArrayList(Vector,支持同步)
		LinkedList
	
	Set:
	
		TreeSet（有序集，根据输入自动排序）
		HashSet(没有重复元素的集合)

Map:
	
	HashMap(HASHTABLE,支持同步,不支持 null,解决 hash 冲突：开放定址法（二次探测法），链地址法，再哈希法)
	LinkedHashMap(记录插入顺序) 
	TreeMap（键值有序排列）
	
优先级队列：REMOVE 时总是获得当前最小的元素

##并发

线程状态：

new

runnable（blocked，waiting，timed waiting）

terminated

创建线程四种方式：

继承 thread；实现接口 runnable；实现接口 callable；用线程池实现（不需要返回值时用 execute 效率高）

停止线程：interrupt，发一个中断信号，线程会在合适的时候停止。堵塞情况下调用会产生一个异常（Object.wait,Thread.sleep,Thread.join(主线程等待子线程执行完才继续)），不是所有堵塞都会响应（synchronized，reentrantLock.lock()不响应中断），IO 必须完成后才可以。interrupted（staitc）：测试当前线程是否已经中断。线程的中断状态 由该方法清除。换句话说，如果连续两次调用该方法，则第二次调用将返回 false。isInterrupted()：测试线程是否已经中断。线程的中断状态 不受该方法的影响。

在多线程中申明long 和 double是不对的，除非加上同步术语

ConcurrentHashMap:分段锁，弱一致性（size）

CopyOnWriteArrayList:迭代多与修改的情况下适合

Lock Condititon（Object 的这些方法是和同步方法捆绑使用，condition 是需要与锁捆绑使用；将 Object 方法分解到多个对象中去，以配合 lock 的使用；优势在于，同一个锁可以创建多个 condition 控制不同的条件）

和 Synochornized 的区别：finally 时必须释放锁，syn 不可中断，悲观锁，

读写锁：读锁可以共享，写锁需要等待；有写锁时，其他锁都不可获得。

countDownLatch，Barrier

悲观锁（独享锁），乐观锁（CAS）


Synochnoized(每一个对象都有一个与之关联的锁，称为内置锁，避免在成员函数上使用，这样会锁掉整个类，缩小同步块，锁中不要再有其他锁;synrhronized使用广泛。其应用层的语义是可以把任何一个非null对象作为"锁"，
当synchronized作用在方法上时，锁住的便是对象实例（this）；
当作用在静态方法时锁住的便是对象对应的Class实例，因为Class数据存在于永久带，因此静态方法锁相当于该类的一个全局锁；

当synchronized作用于某一个对象实例时，锁住的便是对应的代码块;升级（不可降级）：偏向锁（mark word 中记录线程 ID,CAS 竞争锁，失败的话升级），轻量级锁（线程在执行同步块之前，JVM会现在当前线程的栈帧中创建用于储存锁记录的空间（LockRecord），并将对象头的Mark Word信息复制到锁记录中。然后线程尝试使用CAS将对象头的MarkWord替换为指向锁记录的指针。如果成功，当前线程获得锁，并且对象的锁标志位转变为“00”，如果失败，表示其他线程竞争锁，当前线程便会尝试自旋获取锁。使用自旋，大于两个线程时膨胀），重量级锁) 

voliate(解决可见性，无法保证原子性，防止重排序)

concurrent 包的原理是 CAS

单例模式： Double Check Locking 双检查锁机制，此方案仍旧有问题
https://blog.csdn.net/kufeiyun/article/details/6166673
  
public static Singleton getInstance()
{
  if (instance == null)
  {
 synchronized(Singleton.class) {      //1
      Singleton inst = instance;         //2
      if (inst == null)
      {
        synchronized(Singleton.class) {  //3
          inst = new Singleton();        //4
        }
        instance = inst;                 //5
      }
    }
  }
  return instance;
}


##JAVA 虚拟机

JDK：开发的最小环境
JRE：程序运行的标准环境

方法区（运行时常量池）（又称持久带）：共享，存储被虚拟机加载的类信息，常量，静态常量，即时编译的代码等。

虚拟机栈（栈）：线程私有，生命周期与线程相同，局部变量表。线程请求的栈深度大于虚拟机最大深度，则 stackoverflowerror(递归)，动态扩展如果仍然无法得到足够的内存，就OutOfMemoryError（list.add(new Object())）

本地方法栈：调用 native 方法

堆：被共享，大部分存放对象实例以及数组

程序计数器：当前线程所执行的序号

内存泄漏：已使用的内存没有释放

内存溢出：已使用的内存已经到达最大值

Jvm 的参数:
	 UseSerialGC(虚拟机运行在Client 模式下的默认值，打开此开关后，使用Serial + Serial Old 收集器组合进行内存回收)
	 UseParallelGC(虚拟机运行在Server 模式下的默认值，打开此开关后，使用Parallel Scavenge + Serial Old（PS MarkSweep）的收集器)
	 UseParNewGC(打开此开关后，使用ParNew + Serial Old 收集器组合进行内存回收)
	 UseConcMarkSweepGC(打开此开关后，使用ParNew + CMS + Serial Old 收集器组合进行内存回收。)
	 
堆分区：
	1.年轻代
		EDEN
		from
		to
	2.老年代
	3.永久代

##GC

回收什么

	引用计数
	GC ROOTS（虚拟机栈中引用的对象，方法区静态属性引用的对象，方法区常量引用的对象，本地方法栈JNI 引用的对象）

何时回收
	Minor GC ，Full GC 触发条件
	Minor GC触发条件：当Eden区满时，触发Minor GC。
	
	Full GC触发条件：
	
	（1）调用System.gc时，系统建议执行Full GC，但是不必然执行
	
	（2）老年代空间不足
	
	（3）方法区空间不足
	
	（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存
	
	（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小
怎么回收

	标记清除（效率不高，太多碎片，老年代）
	复制（内存分成两块，清除时复制过去，内存缩小到原来的一半，一块较大的 eden 和两块较小的survivor，新生代）
	标记整理（老年代）

引用类型
	强引用，只要存在，就不会回收
	软引用，内存溢出之前，进行回收
	弱引用，不管是否够用，都要回收
	虚引用，被回收时能收到通知





对象的内存布局：对象头，实例数据，对齐填充

垃圾回收机制

Serial（old）收集器（单线程收集）：优点：简单高效；缺点：会 stop the world

ParNew（old）（多线程收集）：Serial 的多线程版（新生代）

Parallel Scavenge：控制吞吐量

CMS：目标：获得最短停留时间

		初始标记（STW，只记录直接关联GC ROOT 的数据）
		并发标记（GC tracing）
		重新标记（STW，修正标记产生的变动）
		并发清除
		
		优点：并发收集，低停顿
		缺点：对 CPU 资源敏感；无法处理浮动垃圾；清理后产生大量内存碎片

G1：划分区域，每次根据优先级来进行回收，保证最大效率。是为了缩短处理超大堆时产生的停顿，相对于 CMS的优势是内存碎片的产生率大大降低。

Eden 空间不足时，发起 minor GC（针对新生代）

大对象，长期存活的直接分配在老年代

内存监控工具：jmap（内存转储快照），jhat（分析 heapdump 文件，简历一个 http 服务器），jstack（线程快照），jps（显示正在运行的虚拟机进程），jstat（监控虚拟机的状态），jinfo（查看虚拟机配置参数）

可视化工具：jconsole，visualvm

类加载过程：加载，连接（验证，准备，解析），初始化，使用，卸载

加载器类型：启动，扩展，应用程序加载器

如果加载器不同的话，那么也不算是同一个类

#分布式架构（TODO）

负载均衡：HTTP重定向，DNS 解析，反向代理

§ CAP 原理和 BASE 理论。
§ Nosql 与 KV 存储(redis，hbase，mongodb，memcached 等)
§ 服务化理论(包括服务发现、治理等，zookeeper、etcd、springcloud 微服务、) § 负载均衡(原理、cdn、一致性 hash)
§ RPC 框架(包括整体的一些框架理论，通信的 netty，序列化协议 thrift，protobuff 等)
§ 消息队列(原理、kafka，activeMQ，rocketMQ)
§ 分布式存储系统(GFS、HDFS、fastDFS)、存储模型(skipList、LSM 等)
§ 分布式事务、分布式锁等