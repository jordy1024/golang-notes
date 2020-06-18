## 查看.go源码对应的汇编代码
GOOS=linux GOARCH=386 go tool compile -S xxx.go > xx.S

## 反汇编工具
go tool objdump -s "runtime\.init\b" hello


## 编译
go build -gcflags "-N -l " -o hello hello.go

```
➜  src cat main.go
package main

import (
	"helloworld"
)

func main() {
	helloworld.PrintMe()
}
➜  src
➜  src tree helloworld
helloworld
├── helloworld.go
└── helloworld.s

0 directories, 2 files
```
