## 逃逸分析
https://juejin.im/post/5c7920f3e51d457120759b77   
```
vim /tmp/t.go
package main

import "fmt"

func main() {
    test()
}
func test() {
    a := new(int)
    *a = 10
    fmt.Println(*a)
}


go build -gcflags '-m' /tmp/t.go
# command-line-arguments
/tmp/t.go:11:13: inlining call to fmt.Println
/tmp/t.go:5:6: can inline main
/tmp/t.go:9:10: test new(int) does not escape
/tmp/t.go:11:14: *a escapes to heap
/tmp/t.go:11:13: test []interface {} literal does not escape
/tmp/t.go:11:13: io.Writer(os.Stdout) escapes to heap
<autogenerated>:1: (*File).close .this does not escape

```



