## pprof笔记

### 一、如何使用
#### 【单个本地程序】
1.如果想收集该程序运行时cpu 耗时的数据，则需要在程序中加入收集cpu profile 的代码，下面是一个示例hello-cpu-profile.go：
比如我的示例程序是:
```
package main
import (
	"fmt"
)
func main(){
	fmt.Println("hello")
}
```
注:整个仅仅是一个示例，仅打印一个hello字符，其实没有太大的分析价值，这里仅仅是为了举例如何使用
如果要手机cpu profile，则需要在上述demo程序中嵌入手机cpu profile数据的代码:
首先引入runtime/pprof包，然后嵌入收集cpu profile的必要代码：
```

  1 package main
  2
  3 import (
  4     "flag"
  5     "fmt"
  6     "log"
  7     "os"
  8     "runtime/pprof"
  9 )
 10
 11 var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to file")
 12
 13 func main() {
 14     flag.Parse()
 15     if *cpuprofile != "" {
 16         f, err := os.Create(*cpuprofile)
 17         if err != nil {
 18             log.Fatal(err)
 19         }
 20         pprof.StartCPUProfile(f)
 21         defer pprof.StopCPUProfile()
 22     }
 23     fmt.Println("hello")
 24 }
```
然后编译该程序：
```
go build -o hello-cpu-profile hello-cpu-profile.go
```
生成可执行文件./hello-cpu-profile
下面就可以在当前目录下执行该可执行的文件，并手机cpu profile数据了，方法就是在执行的时候，加上--cpuprofile项，
将cpu profile 保存到类 .prof 的文件中，比如
```
./hello-cpu-profile  --cpuprofile=/tmp/hello-cpu-profile.prof
./hello-cpu-profile  --cpuprofile=/tmp/hello-cpu-profile.prof
hello
```
程序成功执行，输出hello，并生成了cpu profile 的文件/tmp/hello-cpu-profile.prof
整个时候，cpu profile 数据的收集工作就完成了，拿接下来就是分析了。如何分析，就用go tool pprof 工具：
```
go tool pprof /tmp/hello-cpu-profile.prof
Type: cpu
Time: Apr 9, 2020 at 9:50am (CST)
Duration: 204.53ms, Total samples = 0
No samples were found with the default sample value type.
Try "sample_index" command to analyze different sample values.
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```
看到已经进入pprof命令行，当前Type是cpu
具体如何分析，就可以学学pprof 工具具体的使用了.
比如我们看一下top：
(pprof) top
Showing nodes accounting for 0, 0% of 0 total
      flat  flat%   sum%        cum   cum%
(pprof)
注：因为我们当前的demo程序过于简单，仅仅打印一个hello而已。刚才说了，仅是作为示例，并无太大分析价值；
所以，这里我们看top时，并未打印任何信息.

下面我们换一个稍微复杂一点的程序：斐波那契数列
Xxx


以上是针对cpu profile的收集与分析，下面我们介绍一些对单个程序的内存分析：
过程跟cpu分析非常类似，首先引入runtime/pprof包，然后嵌入收集mem profile的必要代码：
```
 1 package main
  2
  3 import (
  4     "flag"
  5     "fmt"
  6     "log"
  7     "os"
  8     "runtime/pprof"
  9 )
 10
 11 var memprofile = flag.String("memprofile", "", "write memory profile to this file")
 12
 13 func main() {
 14     var s = "hello"
 15     fmt.Println(s)
 16     if *memprofile != "" {
 17         f, err := os.Create(*memprofile)
 18         if err != nil {
 19             log.Fatal(err)
 20         }
 21         pprof.WriteHeapProfile(f)
 22         f.Close()
 23         return
 24     }
 25 }
 ```
然后编译该程序：
go build -o hello-mem-profile hello-mem-profile.go
然后执行该程序并收集mem profile；收集方法就是在后面加上-memprofile 项，如：
./hello-mem-profile -memprofile=/tmp/hello-mem-profile.mprof
hello
这个时候，memory profile文件就收集好了，见/tmp/hello-mem-profile.mprof
然后我们跟分析cpu profile一样，也用go tool pprof 工具来分析men profile 的数据：

go tool pprof  /tmp/hello-mem-profile.mprof

https://blog.golang.org/pprof



单个程序的某个函数，比如针对一个函数的单元测试

#### 【web服务】
当正在执行压测的时候，再启动profile收集才有意义.有两种收集方式：
##### 1.如果使用的是浏览器
则可以访问pprof对于的地址，如：http://localhost:9909/debug/pprof/，看到如下html页面：
```
Types of profiles available:
Count	Profile
2	allocs
0	block
0	cmdline
5	goroutine
2	heap
0	mutex
0	profile
10	threadcreate
0	trace
full goroutine stack dump
Profile Descriptions:
* allocs: A sampling of all past memory allocations
* block: Stack traces that led to blocking on synchronization primitives
* cmdline: The command line invocation of the current program
* goroutine: Stack traces of all current goroutines
* heap: A sampling of memory allocations of live objects. You can specify the gc GET parameter to run GC before taking the heap sample.
* mutex: Stack traces of holders of contended mutexes
* profile: CPU profile. You can specify the duration in the seconds GET parameter. After you get the profile file, use the go tool pprof command to investigate the profile.
* threadcreate: Stack traces that led to the creation of new OS threads
* trace: A trace of execution of the current program. You can specify the duration in the seconds GET parameter. After you get the trace file, use the go tool trace command to investigate the trace.

/debug/pprof/
```
如果是想收集cpu信息，直接点击profile链接，就会开始收集（默认是收集30秒时间段的），然后会自动下载收集好的profile，以供分析；
将浏览器下载的profile文件导入服务器上，然后用prof go tool 分析(go tool pprof 帮助文档go tool pprof  --help)，如，我把profile重新命名了下，叫profile_ab_10000_runing，则：
```
➜  pprof go tool pprof profile_ab_10000_runing
Type: cpu
Time: Apr 10, 2020 at 2:00pm (CST)
Duration: 30s, Total samples = 5.11s (17.03%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```
进入后就可以执行pprof的常用命令进行分析了，比如top，list之类的。

也可以将收集结果以http web的方式在浏览器中将profile展示出来，以便查看分析，这样看起来也很方便，比如：
```
➜  go tool pprof -http=:9092 profile_ab_10000_runing
Serving web UI on http://localhost:9092
```
这种方式更直观，很方便，可以查看top，也支持list ，web（.svg图片的方式展示调用关系），还可以查看调用关系图(graph)，火焰图(flame graph)等



##### 2.如果使用的是命令行
则可以直接用pprof 工具来“实时”收集cpu、内存等profile数据，这样的好初就是免去了刚才上面那种方式的“下载”与“导入”的过程；

因为我们的web服务是监听在http://localhost:9909/debug/pprof/ ，所以直接用pprof工具收集即可，如cpu profile 数据。
首先我们开始启动压测，给服务一定的压力：
ab -n 10000 -c 150 -k "http://localhost:9909/test"

然后用go tool pprof 工具“实时”收集：
```
➜  pprof go tool pprof  http://localhost:9909/debug/pprof/profile
Fetching profile over HTTP from http://localhost:9909/debug/pprof/profile
Saved profile in /Users/bawenmao/pprof/pprof.samples.cpu.008.pb.gz
Type: cpu
Time: Apr 10, 2020 at 2:30pm (CST)
Duration: 30s, Total samples = 1.45s ( 4.83%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 1200ms, 82.76% of 1450ms total
Showing top 10 nodes out of 123
      flat  flat%   sum%        cum   cum%
     600ms 41.38% 41.38%      600ms 41.38%  syscall.syscall
     170ms 11.72% 53.10%      170ms 11.72%  runtime.pthread_cond_wait
     100ms  6.90% 60.00%      100ms  6.90%  runtime.pthread_cond_signal
      70ms  4.83% 64.83%       70ms  4.83%  runtime.usleep
      60ms  4.14% 68.97%       60ms  4.14%  runtime.kevent
      50ms  3.45% 72.41%       50ms  3.45%  runtime.(*semaRoot).queue
      40ms  2.76% 75.17%      120ms  8.28%  bufio.(*Writer).Flush
      40ms  2.76% 77.93%       40ms  2.76%  runtime.nanotime
      40ms  2.76% 80.69%      100ms  6.90%  runtime.netpoll
      30ms  2.07% 82.76%       30ms  2.07%  internal/poll.(*pollDesc).prepare
(pprof)
```
当然了，直接在命令行抓取收集，但是您看的时候，也可以把像上面一样，把数据打到浏览器里面显示，方法很简单，也像上面一样，加个-http 然后在某个端口中启动即可，如
```
go tool pprof -http=:9092  http://localhost:9909/debug/pprof/profile
➜  pprof go tool pprof -http=:9092  http://localhost:9909/debug/pprof/profile
Fetching profile over HTTP from http://localhost:9909/debug/pprof/profile
Saved profile in /Users/bawenmao/pprof/pprof.samples.cpu.013.pb.gz
Serving web UI on http://localhost:9092
```
见图片(by-http)



记住关键点：不管哪种方式收集，一定是在服务正在被压或者业务逻辑正在执行的时候启动收集；
刚开始不熟悉流程，可能在服务都没有任何请求的时候就去收集，这是毫无意义的，也不会收集到明显的profile信息，可能就会像下面这样，啥都看不到：
```
➜  pprof go tool pprof  http://localhost:9909/debug/pprof/profile
Fetching profile over HTTP from http://localhost:9909/debug/pprof/profile
Saved profile in /Users/bawenmao/pprof/pprof.samples.cpu.009.pb.gz
Type: cpu
Time: Apr 10, 2020 at 2:31pm (CST)
Duration: 30s, Total samples = 0
No samples were found with the default sample value type.
Try "sample_index" command to analyze different sample values.
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 0, 0% of 0 total
      flat  flat%   sum%        cum   cum%
(pprof)
(pprof)
```

### 嵌入web框架 
如果是想嵌入的web框架中， 比如类似fasthttp
可以直接import fasthttp封装好的包，如
```
import "github.com/valyala/fasthttp/pprofhandler"
```
然后手动注册路由
```
router.Any("/debug/pprof/*", func(ctx *routing.Context) error {
    pprofhandler.PprofHandler(ctx.RequestCtx)
    return nil
})
```


### 一个内存分析的例子
```
➜  ~ go tool pprof   http://xxx.com/debug/pprof/heap
Fetching profile over HTTP from http://xxx.com/debug/pprof/heap
Saved profile in /Users/jordy/pprof/pprof.ime_server.alloc_objects.alloc_space.inuse_objects.inuse_space.011.pb.gz
File: ime_server
Type: inuse_space
Time: Apr 27, 2020 at 1:46pm (CST)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 3874.55kB, 100% of 3874.55kB total
Showing top 10 nodes out of 36
      flat  flat%   sum%        cum   cum%
 1056.33kB 27.26% 27.26%  1056.33kB 27.26%  bufio.NewReaderSize
  896.99kB 23.15% 50.41%   896.99kB 23.15%  git.sogou-inc.com/iweb/go-util/logserver.GetLogServer.func1
  896.99kB 23.15% 73.57%   896.99kB 23.15%  github.com/valyala/fasthttp/stackless.NewFunc
  512.19kB 13.22% 86.78%   512.19kB 13.22%  runtime.malg
  512.05kB 13.22%   100%   512.05kB 13.22%  github.com/valyala/fasthttp.(*URI).parse
         0     0%   100%   512.05kB 13.22%  git.sogou-inc.com/iweb/go-util.HttpGet
         0     0%   100%   512.05kB 13.22%  git.sogou-inc.com/iweb/go-util.WriteMicroServerAccessLog
         0     0%   100%   896.99kB 23.15%  git.sogou-inc.com/iweb/go-util.init
         0     0%   100%   896.99kB 23.15%  git.sogou-inc.com/iweb/go-util/logserver.GetLogServer
         0     0%   100%   512.05kB 13.22%  git.sogou-inc.com/iweb/ime-dict/internal/srv/controllers/v1.(*DictController).Lbsdict
(pprof)
(pprof)
```


二、如何分析


