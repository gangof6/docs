## 简洁、规范

为数不多规定了代码风格的语言，是的，你只能用一种规范来写代码。
```golang
// SayHello print hello world.
// 注释风格: 函数名 功能说明.(最后的.一定有)
func SayHello() { //函数开始括号一定在函数名后，不能换行
    fmt.Println("Hello World!") //无需;
}
```

为了能让golanger更好的写出格式清爽的代码，golang提供了工具：```gofmt```，可以快速将代码调整成正确的姿势。

规范的注释 + ```go oc```可以帮助你快速生成漂亮的文档。

使用git上的模块，正确的姿势是使用``go get``从指定的git服务器中拉去依赖的项目到GOPATH目录下。(当然也有第三方的工具，例如:glide)

编译：```go build```

运行：```./hello```

golang学习入门：https://golang.org/doc/ (没有比官方更好的文档了)

IDE：vim(plugin go-vim)/vscode...
## 协程


```
    graph LR
    PROCESS-->THREAD1(M)
    PROCESS-->THREAD2(M)
    PROCESS-->THREAD3(M)
    THREAD1(M)-->P
    P-->G0
    P-->G1
    P-->G2
```

    
协程可以有效减少线程切换的代价，GoRoutine是一个资源容器，保存任务的并发状态和栈空间。

每个线程绑定一个调度器(P)会持有一个本地队列(LocalGQueue)，同时还会有一个全局队列(GlobalGQueue)。

线程调度的优先级：本地队列->全局队列->抢别人的

调度的伪代码:

```golang
M.run() {
    P = GetP()
    gqueue = P.queue
    for (;;) {
        g = gqueue.pop
        g.routine()
    }
}
```

启动一个协程
```
var a string

func f() {
	print(a)
}

func hello() {
	a = "hello, world"
	go f()
}

```
 

## 内存管理

题外话，```ps -u --pid ${pid}```的时候看到的VSZ和RSS到底是什么？
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
xiaoju   10829  8.4  0.0 8965412 72752 ?       Sl    2018 32425:33 ./bin/mapdata_delivery_service -log_dir=./log -v=1
```

> from https://stackoverflow.com/questions/7880784/what-is-rss-and-vsz-in-linux-memory-management/21049737#21049737

**Virtual Memory Size(VSZ)** 所有进程可访问的内存(包括已被置换到磁盘的部分、已分配未使用部分、共享库使用部分)

**Resident Set Size(RSS)**
为进程已分配的内存大小。不包含已置换到磁盘的部分。包含调用的共享库使用部分、所有的栈和堆空间。

#### example

ProcessA: 

file | size
---|---
binary | 500K
shared libraries | 2500K

RAM | size
---|---
stack & heap (ALL) | 200K
stack & heap (USED) | 100K
shared libraries(USED) | 1000K
binary(USED) | 400K


RSS: 400K + 1000K + 100K = 1500K
VSZ: 500K + 2500K + 200K = 3200K

* 所有进程VSZ之和可能大于RAM
* RSS会随着进程运行时刻变化，VSZ可能不变
 
#### 进一步概念
* 物理内存
* 虚拟内存
* 分页
 
### Opearting System
编译后的ObjectFile包含：
* mechine code
* meetadata about application(OS architecture, debug information)
* application data(global variables or constants)

在Linux系统上，可执行程序格式为ELF([Executable and Linkable Format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format))
> 如何查看一个ELF结构？

```
size --format=sysv bin/main
bin/main  :
section                  size       addr
.text                 5401775    4198400
.plt                      544    9600192
.rodata               1603785    9601024
.typelink               16288   11204812
.itablink                3792   11221104
.gosymtab                   0   11224896
.gopclntab            2019463   11224896
.dynsym                   912   13246528
.rela                      24   13244368
.rela.plt                 792   13244416
.gnu.version               76   13245216
.gnu.version_r             80   13245312
.hash                     188   13245408
.dynstr                   540   13245984
.got.plt                  288   13250560
.dynamic                  304   13250848
.got                        8   13251152
.noptrdata             113728   13251168
.data                   33840   13364896
.bss                   126416   13398752
.noptrbss               23168   13525184
.tbss                       8          0
.debug_abbrev             255   13549568
.debug_line            783797   13549823
.debug_frame           480644   14333620
.debug_pubnames        590442   14814264
.debug_pubtypes        279097   15404706
.debug_aranges             48   15683803
.debug_gdb_scripts         44   15683851
.debug_info           2147315   15683895
.interp                    28    4198372
.note.go.buildid           56    4198316
Total                13627745
```

![image](http://note.youdao.com/yws/res/3833/1D4FA3C2FE7140F595B9F3829A25927B)

进程分配内存的方法:

进程加载时，创建虚拟存储空间，将程序地址映射到虚拟地址。

1. 静态分配(E.g. const)
2. 自动分配(E.g. 栈对象)
3. 动态分配
    1. mmap/munmap – allocates/deallocates fixed block memory page.
    2. brk/sbrk – changes/gets data segement size
    3. madvise – gives advise to Operating System how to manage memory
    4. set_thread_area/get_thread_area – works with thread local storage.

golang底层直接使用系统提供的上面4种方法进行内存分配，而不是使用libc提供的malloc和free。Go的内存分配器源于：[TCMalloc](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)
TCMalloc号称：分配速度更快、对多线程程序有更好的竞态控制。

基本思想：基于页(spans/mspan对象)，使用线程缓存，并将缓存按大小分组。Spans是由连续的8K的内存块组成。
![image](http://note.youdao.com/yws/res/3894/76E28C5E2C46420B9DE30414FD22A162)

> Tiny: <16k

> Small: < 32kB

> Large: >=32kB

#### golang的内存分配器工作流程：

allocating tiny object（0-16 bytes）:
1. 在P的mcache中寻找相关的tiny slot object.
2. 按长度截取成8, 4或2.
3. 如果足够大小，则放置到空间内.
```
0:
tinyObj := p.mcache.findTinySlotObject(size)
assert(tinyObj)
rest := tinyObj.rest - size
if rest > 0 {
    // 分配到tinyObj空闲的区域
} else {
    // 没有足够的空间
    mspanList := p.mcache.mspan
    mspan := mspanList.Get()
1:
    slot := mspan.bitmap.getFreeSlot()
    if slot != nil {
        slot.allocate()
        p.mcache.putTinySlotObject(slot)
        //重新分配
        goto 0
    } else {
        // no free slots
        mspan = mcentral.getMspan(size)
        if mspan != nil {
            goto 1
        } else {
            mheap := mcentral.mheap
            if mheap != nil {
                mspan = mheap.getSpan()
                goto 1
            } else {
                os.AcquireMorePages()
                goto 1
            }
        }
    }
}
```

#### 你的对象在栈上？还是在堆上?
```
package main
 
import (
 "fmt"
)
 
func getAge() *int {
 age := 34
 return &age 
 //返回一个局部变量，对编译器而言，这种情况称为：逃逸(escaped)
 //发生逃逸的对象会直接分配在堆上
}

func main() {
 fmt.Println(*getAge())
}
```

```
package main
 
import (
 "fmt"
)
 
const Width, Height = 640, 480
 
type Point struct {
 X, Y int
}
 
func MoveToCenter(c *Point) {
 c.X += Width / 2
 c.Y += Height / 2
}
 
func main() {
 p := new(Point) 
 //new和c++的new可不一样，这里的p并没有逃逸，
 //因此，编译器将p放在了栈上
 MoveToCenter(p)
 fmt.Println(p.X, p.Y)
}
```

关于栈和堆，指针相关的更多阅读：

[Playing with Pointers in Golang
](https://www.callicoder.com/golang-pointers/)

[Go Memory Management](https://povilasv.me/go-memory-management/)

[Getting to Go: The Journey of Go's Garbage Collector](https://blog.golang.org/ismmkeynote)

## 同步机制

* channel
```golang
c := make(chan int, 1)
c <- v
t := <-c
```
* WaitGroup
```
wg := sync.WaitGroup{}
wg.Add(3)

for i := 0; i < 3; i++ {
    go func() {
        time.Sleep(1 * time.Second)
        wg.Done()
    }()
}

wg.Wait()
fmt.Println("all sub routines done.")

```
* mutex
* atmoic
