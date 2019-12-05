
 ### innodb引擎

	innoDb体系架构： 多个后台线程-》innoDb内存池-》磁盘文件 ，后台线程的主要作用是负责刷新内存池中的数据，保证缓冲池的数据是最近的，并且将修改数据刷新到磁盘。保证在数据库异常的时候innodb能恢复到正常
	
 #### 后台线程::
 
	1.Master Thread 主线程，负责将缓冲池的数据异步刷新到磁盘。包括脏页？的刷新。合并插入缓冲、undo回收等
	2.IO Thread 。1.0后支持设置innodb_read_io_threads innodb_write_io_threads 设置读写io线程。写线程要大于读线程
	3.purgeThread 用于回收事务提交后的undolog。innodb 1.1以后可以设置单独的线程来回收 innodb_purge_threads = 1 ,innodb 1.2后支持多个线程 1.1 只能1个
	4.page clear thread。在innodb 1.2后支持，作用是减轻master thread对于用户查询线程的阻塞。把脏页的刷新放进了单独的线程
	
 #### 内存::
 
	1.缓冲池，mysql是给予磁盘存储的数据库，为了弥补硬盘读写速度相对数据库慢的性能，划分一块内存来做缓冲。读数据，将数据fix到缓冲池后，
	在返回。写入的时候，先写入缓冲后，在入磁盘。不是实时是通过checkpoint机制。配置innodb_buffer_pool_size 设置缓冲池大小缓存
	的数据有索引页，数据页，undo页，插入缓冲，自适应哈希索引，innodb存储的锁信息，数据字典信息。
	
	2.从innodb 1.0后支持多个缓冲池实例 innodb_buffer_pool_instances 默认1，每个页根据哈希值平均分配到不通的缓冲池，减少数据库内部资源竞争，
	增加并发。 mysql5.6后支持 select POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES from INNODB_BUFFER_POOL_STATS 来查询实例的状态
	
	3.LRU 最近最少使用算法，当缓冲池不能存放新页的时候，先释放LRU中的尾端（头端是最频繁使用的页）。innodb缓冲池页大小默认16kb，
	innodb对LRU算法做了优化。在LRU list中加入了midpoint。存储的最页不是放在头端，而是放在midpoint位置。默认是在5/8的位置。
	可用innodb_old_blocks_pct控制ex. innodb_old_blocks_pct=37 代表新页插入在尾端的37%的位置。目的是为了把常见的可能是索引页或者数据页不会
	被刷出LRU list。这些页通常是查询操作需要，但又不是活跃的热点数据。 为了解决这个问题 innodb支持设置innodb_old_block_time 
	用于表示页到mid位置需要等待多久才会被加入到LRUlist 中。 
	
ex.
```
    set global innodb_old_blocks_time = 1000;
			
			#查询数据，或者索引扫码
    set global innodb_old_blocks_time = 0;
```		`
			
