
### go 协程

从语言层面实现了go自己的协程。用于自己的调度器和工作方式（channel），基于通信来共享内存（CSP），而非共享内存来通信


#### PMG模型
P：程序运行相关的上下文，可简单理解成逻辑解释器，可代表并发数量（非并行）

M: 对应内核线程，与内核线程一对一绑定

G: 实际运行的go协程
运行go协程时，G会加入的P的local队列，P去寻找M进行执行。若P的G队列都执行完，P将会从G的global队列中拿50%进行执行，若global队列中无G，将尝试从别的P上获取。
go协程阻塞分2中情况：
1. 用户态的阻塞，程序上chan被阻塞，或者等待对应chan的信号时，G会计入wait队列，M继续执行下一个G1。当G被唤醒时，将加入P的local队列
2. 系统侧的阻塞，类似硬盘IO，网络请求时，P将解绑G和M，去寻找新的M1.若G被唤醒，会阐述加入空闲的P2.如果无空闲，加入P的global队列

1.go chanel 在输入数据 ch<- 1 和读取数据 value := <-ch 之前都是阻塞，同时 make(chan int , 1024) 第二参数不传默认是不带缓存的，意思是之前提到的每一个chan是阻塞的，传入代表带缓存，并且是大小（1024可容纳1024个元素，并且先入先出）len(chan) 当前已存个数，cap(chan)为chanel容纳数量，也就是上文提到的1024在一个已关闭 channel 上执行接收操作(<-ch)总是能够立即返回，返回值是对应类型的零值。

2.协程中输出未必能正常输出，协程和主进程交互可以通过chanel或者通过sync.mutex Lock(),在主进程中判断lock()后的标识来确认协程是否执行完其原理是共用相同内存并且带锁。

3.可以声明单向chanel。只读，只取。

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

5.go chanel不会超时，但可以通过声明一个类似以下的chanel实现超时

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
```

6.go runtime包中有几个处理goroutine的函数

	Goexit 退出当前执行的goroutine，但是defer函数还会继续调用

	Gosched 让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行

	NumCPU 返回 CPU 核数量

	NumGoroutine 返回正在执行和排队的任务总数

	GOMAXPROCS 用来设置可以并行计算的CPU核数的最大值，并返回之前的值
#### GC回收
常见的回收机制有，计数回收、标记回收、分代回收、标记整理回收，GO是三色标记回收
go的内存分配算法是决定了基本不会产生不连续的内容，自然整理也无意义
1. 所有对象均为白色
2. 从根对象开始扫描，扫描的对象被置为灰色，并加入待处理队列
3. 从待处理队列中取出对象，在进行扫描，被扫描的对象被置为灰色，并加入待处理队列，同时对象本身为黑色
4. 递归完成后，对所以白色对象进行回收

#### 泛型（go 1.18引入）
所有类型定义都可以使用类型形参，所以结构体、接口、chan均支持泛型
```go
type myStruct[T int|string] struct{
	Name string
	Data T
}
type MyInterface[T int|float32] interface{
	print(data T)
}
type MyChan[T int|string] chan T
```

基础类型不能只有类型形参
```go
// 错误示例，定义泛型时不能只有类型形参
type CommonType[T int|string|float32] T

```
当类型形参是指针时，需要用interface{}包裹
```go
// 错误示例
type myType[T *int|*string] []T
// 正常示例
type myType[T interface{*int|*string}] []T
```
支持泛型接收者
```go
type MQ[T int|string] []T
func (m *MQ[T]) add(value T){
 //do some code
}

```
支持泛型函数
```go
func add[T int|float32](a,b T) T {
	return a + b
}
```
匿名函数不能定类型实参，但可以用定义好的类型实参
```go
// 错误示例
fn := func[T int| string] (a,b T) T{
	return a + b
}

// 可以利用定义好的类型实参

func MyFn[T int| string](a,b T) T{
	
	fn2 := func(i,j T) T{
		return i -j
    }
	return fn2(a,b)
}
```
不支持泛型方法
```go
type myStruct struct{
	
}
// 错误示例,猜测是因为接受者都不是泛型，非泛型的方法就算处理了泛型也无法存储
func(m myStruct) Add[T int| string](a,b T) T {
	return a + b 
}
```
inteface变化，过往interface是方法约束，现在演变成类型约束
```go
type myInterface interfece{
	~int
}

type myFn interface{
	add(data []string) bool
}
```

### go 小知识

指针变量 p 、*p、 &p的区别：*p用于取值，&p用于取出指针变量在内存地址（非值）

	单引号，双引号，反引号的区别：
	
	双引号用来创建 可解析的字符串字面量 (支持转义，但不能用来引用多行)；
	
	反引号用来创建 原生的字符串字面量 ，这些字符串可能由多行组成(不支持任何转义序列)，原生的字符串字面量多用于书写多行消息、HTML以及正则表达式。
	
	单引号则用于表示Golang的一个特殊类型：rune，类似其他语言的byte但又不完全一样，是指：码点字面量（Unicode code point），不做任何转义的原始容

拥有底层相同类型的才能转化，用类似T(x)。如果是指针类型加上括号（*T)(x) 

```
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
```
