# GO基础

## 基础包

### time包

获取时间对象

```go
// 1. 获取当前的时间对象
func Now() Time

// 2. 获取指定时间的时间对象
// 其中loc是时区，常用的时区有 local
func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time

```

根据时间对象，获取时间明细

```go
// 年、月、日、时、分、秒
func (t Time) Year() int
func (t Time) Month() Month
func (t Time) Day() int
func (t Time) Hour() int
func (t Time) Minute() int
func (t Time) Second() int

// unix时间戳
func (t Time) Unix() int64
func (t Time) UnixNano() int64
```

时间格式化

```go
// 格式化的模板对应的时间必须是 2006-01-02 15:04:05
// 格式化时间
f1 := t1.Format("2006年01月02日 15点04分05秒")
func (t Time) Format(layout string) string

// 解析时间并生成时间对象
func Parse(layout, value string) (Time, error)
```



###  file操作

**1. fileInfo文件信息**

```go
// 获取文件信息，其中name为文件名
func Stat(name string) (FileInfo, error)

// 查看文件信息明细
type FileInfo interface {
    Name() string       // base name of the file
    Size() int64        // length in bytes for regular files; system-dependent for others
    Mode() FileMode     // file mode bits
    ModTime() time.Time // modification time
    IsDir() bool        // abbreviation for Mode().IsDir()
    Sys() any           // underlying data source (can return nil)
}
```

**2. filepath文件路径**

```go
// 获取绝对路径(path可传入相对路径)
func Abs(path string) (string, error)

// 获取path的目录
func Dir(path string) string

// 获取path的文件名（不包含目录）
func Base(path string) string

// 合并路径后，计算出最终的目录（即不会包括 ".", ".."这类目录）
func Join(elem ...string) string

// 将path分解为dir(目录名)和file(文件名)
func Split(path string) (dir, file string)
```

**3. os**

目录相关

```go
// 创建目录（如果父目录不存在会报错）
func Mkdir(name string, perm FileMode) error
// 递归创建目录（如果父目录不存在会先创建父目录，然后再创建子目录）
func MkdirAll(path string, perm FileMode) error
```

文件相关

```go
// 创建文件 
func Create(name string) (*File, error)

// 打开文件
func Open(name string) (*File, error)
// 打开文件： 可选的flag
// os.O_CREATE: 不存在文件则创建
// os.O_RDONLY: 只读
// os.O_WRONLY: 只写
// os.O_RDWR: 读写(相当于 os.O_RDONLY | os.O_WRONLY) 
// os.O_APPEND: 追加到文件的尾部
// os.O_TRUNC: 截断文件
func OpenFile(name string, flag int, perm FileMode) (*File, error)

// 删除文件
func Remove(path string) error
func RemoveAll(path string) error

// 读取文件
func (f *File) Read(b []byte) (n int, err error)

// 写入文件
func (f *File) Write(b []byte) (n int, err error)
func (f *File) WriteString(s string) (n int, err error)
```

**4. io**

```go
func Copy(dst Writer, src Reader) (written int64, err error)
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error)
```

**5. ioutil**

```go
func ReadFile(filename string) ([]byte, error)
func WriteFile(filename string, data []byte, perm fs.FileMode) error
```

## 并发

### runtime包

```go
//  调度相关
// 让出当前的时间片，让其他goroutine先执行
func Gosched()
// 退出goroutine。除了已经注册的defer，在goroutine里的其他代码都不会执行。
func Goexit()

// 基础信息
func GOROOT() string	// 显示go的安装目录GOROOT
func NumCPU() int		// 获取CPU数量
GOOS					// 获取当前操作系统的名称
func GOMAXPROCS(n int) int		// 设置最大CPU数量
```

### sync包

**WaitGroup**

```go
func (wg *WaitGroup) Add(delta int)		// 增加计数器的值
func (wg *WaitGroup) Done()				// 将计数器的值减1
func (wg *WaitGroup) Wait()				// 阻塞当前goroutine，直到计数器的值为0
```

**Mutex **

```go
func (m *Mutex) Lock()				// 上锁
func (m *Mutex) TryLock() bool		// 尝试上锁
func (m *Mutex) Unlock()			// 解锁
```

**RWMutex**

```go
// 读锁
func (rw *RWMutex) RLock()
func (rw *RWMutex) RUnlock()

// 写锁
func (rw *RWMutex) Lock()
func (rw *RWMutex) Unlock()
```

### channel

通道是可以让go routine相互通信的工具。

每个通道都有与之相关的类型，该类型是通道允许传送数据的类型。（声明通道时，通道的类型是nil，它不能用于读写数据。通道必须通过make初始化之后才能读写数据。）

```go
// 声明并初始化通过，通道里传送的数据类型为int
ch := make(chan int)

// 向通道写入数据
ch <- data

// 从通道读取数据
data <- data

// 读取通道ch的数据，直到通道关闭。
for val := range ch {
    // do something
}

// 关闭通道
close(ch)

// 缓冲通道，capacity为缓冲区的容量(通道也可以理解为队列)
// 发送：缓冲区满了才会阻塞
// 接收：缓冲区空了才会阻塞
ch := make(chan type, capacity)
```

定向通道

定向通道的应用：在函数的入参中限制通道的只写或者只读，对参数进行更多的约束。类似C++对入参加const。

```go
// 只写通道
var ch chan <- int

// 只读通道
var ch <- chan int
```

**timer**

```go
// 创建一个timer
func NewTimer(d Duration) *Timer

// 停止一个timer
func (t *Timer) Stop() bool

func After(d Duration) <-chan Time
```

### CSP模型

go语言利用了CSP模型的process和channel的概念，具体到go语句就是goroutine和channel。

其他语言使用的是线程和共享内存来进行并发和通信。

go并发编程的一个思想

**不要通过共享内存来通信，而是通过通信来共享内存**

不建议使用sync来并发编程

强烈建议使用channel来并发编程



## 反射

反射建立在类型基础之上。

go语言的类型：

- 变量包括(value, type)两部分
- type包括static type和concrete type。简单来说static type是指编码时定义的类型（如int，string, io.Reader)，concrete type是runtime类型。
- 类型断言能否成功，取决于变量的concrete type，而不是static type。因此，一个reader变量如果它的concrete type也实现了write方法的话，它也可以被类型断言为writer。
- 编译时就知道变量类型的是静态类型; 运行时才知道一个变量类型的叫做动态类型。

在go语言只有接口才能使用反射。每个interface变量都有一个对应pair，pair中记录了实际变量的值和类型

```go
(value, type)
```

value是实际变量值，type是实际变量的类型。一个interface{}类型的变量包含了2个指针，一个指针指向值的类型【对应concrete type】，另外一个指针指向实际的值【对应value】

```go
// tty的类型是 *os.File
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)

// r的static type是io.Reader
var r io.Reader
// 接口r的pair信息是(tty, *os.File)，所以它的concrete type是 *os.File
r = tty

// w的static type是io.Writer
var w io.Writer
// r的类型断言能成功是因为它的concrete type为 *os.File,它实现了 io.Writer接口。
// 最后w的pair信息是(tty, *os.File)，它与r的pair信息是一样的。
w = r.(io.Writer)
```

interface及其pair的存在，是Golang中实现反射的前提。

**反射的作用**：用来检测存储在接口变量内部（值value，类型concrete type）pair对的一种机制。

**使用反射的步骤**:

1. 先有需要反射的变量（一般是interface）
2. 把它转化为reflect对象（reflect.Type或者reflect.Value）
3. 根据实际情况调用不同的反射函数

```go
t := reflect.TypeOf(i)			// 得到类型的元数据
v := reflect.ValueOf(i)			// 得到实际的值
```

