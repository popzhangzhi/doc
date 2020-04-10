
 ### innodb引擎
 
  innoDb体系架构： 多个后台线程-》innoDb内存池-》磁盘文件 ，后台线程的主要作用是负责刷新内存池中的数据，保证缓冲池的数据是最近的，并且将修改数据刷新到磁盘。保证在数据库异常的时候innodb能恢复到正常
  
  内存架构：重做日志缓冲 额外内存池 缓冲池。三者并列
  	
```	
额外内存池：每个缓冲池的帧缓冲 （frame buffer） 还有对应的缓冲控制对象（buffer control block）。这些对象记录了LRU 锁 等待等信息需要从额外内存池中申请
缓冲池： 数据页 插入缓冲 锁信息 索引页 自适应哈希索引 数据字典信息
```  
	
 #### 后台线程::
 
 1.Master Thread 主线程，负责将缓冲池的数据异步刷新到磁盘。包括脏页的刷新。合并插入缓冲、undo回收等

 2.IO Thread 。1.0后支持设置innodb_read_io_threads innodb_write_io_threads 设置读写io线程。写线程要大于读线程

 3.purgeThread 用于回收事务提交后的undolog。innodb 1.1以后可以设置单独的线程来回收 innodb_purge_threads = 1 ,innodb 1.2后支持多个线程 1.1 只能1个

 4.page clear thread。在innodb 1.2后支持，作用是减轻master thread对于用户查询线程的阻塞。把脏页的刷新放进了单独的线程
	
 ### 内存::
 
 1.缓冲池，mysql是给予磁盘存储的数据库，为了弥补硬盘读写速度相对数据库慢的性能，划分一块内存来做缓冲。读数据，将数据fix到缓冲池后，
	在返回。写入的时候，先写入缓冲后，再入磁盘。不是实时是通过checkpoint机制。配置innodb_buffer_pool_size 设置缓冲池大小缓存
	的数据有索引页，数据页，undo页，插入缓冲，自适应哈希索引，innodb存储的锁信息，数据字典信息。
	
 2.从innodb 1.0后支持多个缓冲池实例 innodb_buffer_pool_instances 默认1，每个页根据哈希值平均分配到不通的缓冲池，减少数据库内部资源竞争，
	增加并发。 mysql5.6后支持 select POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES from INNODB_BUFFER_POOL_STATS 来查询实例的状态
	
 3.LRU 最近最少使用算法，当缓冲池不能存放新页的时候，先释放LRU中的尾端（头端是最频繁使用的页）。innodb缓冲池页大小默认16kb，
	innodb对LRU算法做了优化。在LRU list中加入了midpoint。存储的最页不是放在头端，而是放在midpoint位置。默认是在5/8的位置。
	可用innodb_old_blocks_pct控制ex. innodb_old_blocks_pct=37 代表新页插入在尾端的37%的位置。目的是为了把常见的可能是索引页或者数据页不会
	被刷出LRU list。这些页通常是查询操作需要，但又不是活跃的热点数据。 为了解决这个问题 innodb支持设置innodb_old_block_time 
	用于表示页到mid位置需要等待多久才会被加入到LRUlist 中。 
```	
set global innodb_old_blocks_time = 1000;

#查询数据，或者索引扫码
set global innodb_old_blocks_time = 0;
#如果预估自己的活跃热点数据不止63%，在执行之前可以通过 
set GLOBAL innodb_old_blocks_pct=20 
#来减少刷出的概率
```

  4.LRU 列表用来管理已经读取的页，当数据库刚刚启动，LRU是空的。这个时候页都存在Free列表中。当需要从缓冲池分页时，首先从Free列表中找到是否有可用的空闲页，若有，则从Free列表中删除，放入LRU列表中。若无。根据LRU算法淘汰LRU尾部的页，分配给新页。当页从LRU列表old加入new部分，被称之为page made young，而因为innodb_old_blocks_time设置而导致页没有从old转移到new。称之为page no made young。
  
  5.通过命令show engine innodb status 可以查看到相关的信息。buffer pool size 代表缓冲池的页数 * 16k 可以算出缓冲池大小。通过buffer pool hit rate 可以看到缓冲池的命中率，通过该值不应该小于95%。如果出现要排查下是否由于全表扫描导致LRU列表污染。
  
  6.redo log buffer。 把重做日志缓冲刷新到重做日志文件，可由 innodb_log_buffer_size控制，默认8M。一般无需修改
  
 ```
 以下3种情况会触发
 	 master thread 每秒刷入重做日志文件
	 每个事务提交时会刷入重做日志文件
	 当重做日志缓冲池剩余空间小于二分之一时，会输入重做日志文件
```
7.checkpoint 将脏页数据输入磁盘的作用：1.缩短数据库恢复时间 2 缓冲池不够用，将脏页刷入磁盘 3 重做日志不可用时，刷新脏页
	
``` 
有2中checkpoint
	
	sharp checkpoint 默认方式。发在数据库关闭的时候将所有脏页都刷入。也就是参数 innodb_fast_shutdwon =1
	
	fuzzy checkpoint 部分刷新到磁盘
		1.主线程以每秒和每十秒从缓冲池的脏页刷新到磁盘
		2.innodb 1.1会保证LRU有差不多100个空闲页可使用，如果不足会将LRU列表尾端的页移除，如果这些页有脏页，则会checkpoint，innodb 1.2后会开异步线程，放在单独的page cleaner进行，用户可以设置innodb_lru_sacn_depth设置可用的页数量。默认1024
		3.重做日志不可用时，会刷新脏页。如total_redo_log_file_size 设置重做日志大小。当脏页占整个重做日志文件的75%以下，不刷新，以上触发刷新。
		4.脏页太多会触发checkpoint。innodb_max_dirty_pages_pct =75 .表示缓冲池脏页数量占据75%的时候强制触发checkpoint
```

#### Master thread工作方式
	
##### innodb1.0之前的master thread

```
每秒操作包含 
1.日志缓冲刷新到磁盘（总是）即使这个事务还没有提交。所以再大的事务提交的时间也很快
2.合并插入缓冲 （可能）：判断io次数是否小于5，小于认为io压力小，执行合并插入缓冲操作
3 最多刷新100个脏页到磁盘（可能） 判断缓冲池的脏页比是否超过了配置文件中的 innodb_max_dirty_pages_pct 默认90.则进行同步操作
4 没有活动，切换到后台loop（可能）

每10秒操作：
1.刷新100个脏页 （可能） 判断过去10秒io操作小于200，则认为有足够的io能力刷新100个脏页
2.最多合并5个插入缓冲（总是）同上条件下
3.将日志刷新到磁盘（总是）同上条件下
4.删除无用的undo页（总是）同上条件下，如对表update、delete这类操作，原先的行被标记为删除。但因为一致性读的关系，需要保留这些信息，在full purge会自动判断是否可以删除
5.刷新100个或者10个脏页到磁盘（总是）会判断缓冲池中脏页比例（buf_get_modified_ratio_pct）是否大于70% 。是刷新100个脏页，否刷新10个

background loop ： 删除无用的undo页 合并20个插入缓冲 跳回主循环
```
##### innodb 1.2之前的master thread

```

对io是有限制的，在缓冲池刷新磁盘的时候，做了一定的赢编码。与如今ssd硬盘有高速的io能力的时候，限制了写入性能。
即就是主线程最多刷新100个脏页和20个插入缓冲，因此从1.0以后，提供了innodb_io_capactiy 用来表示磁盘的吞吐量。默认200.对于刷新磁盘页的数量会按照该值的百分比来控制：
1.合并缓冲时，合并缓冲的数量为innodb_io_capacity 的5%
2.缓冲池刷新脏页，刷新数量为innodb_io_capacity

同时修复了innodb_max_dirty_pages_pct 默认值得问题，之前90%台大了。如果数据库有很大内存，刷新脏页的速度会降低，在数据库恢复阶段需要更多的时间。默认值修改成75.google测试值80比较接近
同时增加了参数 innodb_adaptive_flushing 来影响每秒刷新脏页的数量。通过判断产生redo log的速度来决定刷新脏页数量，当脏页比例小于innodb_max_dirty_pages_pct的时候也会刷新脏页
之前最多删除20个undo页，现在可以通过参数innodb_purge_batch_size来控制删除undo页的数量，默认20.并且可以动态修改。set global innodb_purge_batch_size = 50 

```

#### innodb 1.2版本的master thread

把刷新脏页的 page cleaner thread 单独分离到一个线程（undo log 脏页）。从而减轻了master thread的工作

### innodb 关键特性

#### 插入缓冲

是针对不是唯一的非聚集索引做的。因为聚集索引的插入是顺序的，速度很快，所以不用，而非聚集索引，因为b+tree的特性，插入是随机的，离散的。
对于这类非聚集索引的插入和更新不是每一次直接插入到索引页，而是先判断对应的非聚集索引页是否在缓冲池，如在，就插入，不在则先放到一个插入缓冲的对象，欺骗似的告诉数据库已经插入，而实际没有。然后以一定频率进行insert buffer和辅助索引页子节点的合并操作。通常多个插入到一个操作，提交非聚集索引的性能

插入过多会导致占据很大的缓冲池，影响其他操作。默认是25%冲池，可以设置innodb_change_buffer_max_size=25 控制change buffer的最大使用内存。最大50%



	
	
	

			
