
### go 协程，从语言层面实现了go自己的协程。用于自己的调度器和工作方式（channel），非抢占式的协程

1.go chanel 在输入数据 ch<- 1 和读取数据 value := <-ch 之前都是阻塞，同时 make(chan int , 1024) 第二参数不传默认是不带缓存的，意思是之前提到的每一个chan是阻塞的，传入代表带缓存，并且是大小（1024可容纳1024个元素，并且先入先出）len(chan) 当前已存个数，cap(chan)为chanel容纳数量，也就是上文提到的1024在一个已关闭 channel 上执行接收操作(<-ch)总是能够立即返回，返回值是对应类型的零值。
2.协程中输出未必能正常输出，协程和主进程交互可以通过chanel或者通过sync.mutex Lock(),在主进程中判断lock()后的标识来确认协程是否执行完其原理是共用相同内存并且带锁。
3. 可以声明单向chanel。只读，只取。
	```
	var ch1 chan int  　　　　// 普通channel
	var ch2 chan <- int 　　 // 只用于写int数据
	var ch3 <-chan int 　　 // 只用于读int数据
	也可以用个类型动态转化某个普通chanel变成当前通道
	ch4 := make(chan int)
	ch5 := <-chan int(ch4)   // 单向读
	ch6 := chan<- int(ch4)  //单向写
	单向channel的作用有点类似于c++中的const关键字，用于遵循代码“最小权限原则”。
	```
4.select 用于监听chanel的io，类似于swtich但只有一个case能执行，如果所有case都不能满足，可以执行default。default是可选的。不存在default的时候，会阻塞。等待某个case成立后执行一次。如果有满足多个case，会伪随机的执行其中一个case。在写的时候，只有带缓存的chanel才能写，不带的永远都不会满足case
5. go chanel不会超时，但可以通过声明一个类似以下的chanel实现超时
```
	timeout := make(chan bool, 1)
	go func() {
		time.Sleep(5*time.Second)
		timeout <- true
	}()
	switch {
	case <- ch:
	// 从ch中读取到数据

	case <- timeout:
	// 没有从ch中读取到数据，但从timeout中读取到了数据
	}
	```
6. go runtime包中有几个处理goroutine的函数
	Goexit 退出当前执行的goroutine，但是defer函数还会继续调用

	Gosched 让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行

	NumCPU 返回 CPU 核数量

	NumGoroutine 返回正在执行和排队的任务总数

	GOMAXPROCS 用来设置可以并行计算的CPU核数的最大值，并返回之前的值
	
### go 小知识

指针变量 p 、*p、 &p的区别：*p用于取值，&p用于取出指针变量在内存地址（非值）
	单引号，双引号，反引号的区别：
	双引号用来创建 可解析的字符串字面量 (支持转义，但不能用来引用多行)；
	反引号用来创建 原生的字符串字面量 ，这些字符串可能由多行组成(不支持任何转义序列)，原生的字符串字面量多用于书写多行消息、HTML以及正则表达式。
	单引号则用于表示Golang的一个特殊类型：rune，类似其他语言的byte但又不完全一样，是指：码点字面量（Unicode code point），不做任何转义的原始容

拥有底层相同类型的才能转化，用类似T(x)。如果是指针类型加上括号（*T)(x) 
#string到int  
int,err := strconv.Atoi(string)  

#string到int64  
int64, err := strconv.ParseInt(string, 10, 64)  
//第二个参数为基数（2~36），
//第三个参数位大小表示期望转换的结果类型，其值可以为0, 8, 16, 32和64，
//分别对应 int, int8, int16, int32和int64

#int到string  
string := strconv.Itoa(int) 
//等价于
string := strconv.FormatInt(int64(int),10)
 
#int64到string  
string := strconv.FormatInt(int64,10)  
//第二个参数为基数，可选2~36
//对于无符号整形，可以使用FormatUint(i uint64, base int)

#float到string
string := strconv.FormatFloat(float32,'E',-1,32)
string := strconv.FormatFloat(float64,'E',-1,64)
// 'b' (-ddddp±ddd，二进制指数)
// 'e' (-d.dddde±dd，十进制指数)
// 'E' (-d.ddddE±dd，十进制指数)
// 'f' (-ddd.dddd，没有指数)
// 'g' ('e':大指数，'f':其它情况)
// 'G' ('E':大指数，'f':其它情况)

#string到float64
float,err := strconv.ParseFloat(string,64)

#string到float32
float,err := strconv.ParseFloat(string,32)

#int到int64
int64_ := int64(1234)

