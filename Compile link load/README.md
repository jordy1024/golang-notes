## Golang 可执行文件组成与大小分析

###背景
不经意间听说，Go语言的可执行文件比其他语言的要大很多，于是想亲自做个试验对比一下。


###Go vs C 
```
#include<stdio.h>
int main(void){
        printf("hello world\n");
}

[root@jordy ~]# gcc -o hello hello.c

[root@jordy ~]# ./hello
hello world

[root@jordy ~]# du -sh hello
12K	hello

```

```
[root@jordy ~]# vim hello.go
package main
  
import "fmt"

func main() {
    fmt.Println("hello world")
}

[root@jordy ~]# go build -o hello-go hello.go

[root@jordy ~]# 
[root@jordy ~]# 
[root@jordy ~]# go build -o hello-go hello.go
[root@jordy ~]# 
[root@jordy ~]# ./hello-go
hello world
[root@jordy ~]# du -sh hello-go
2.0M	hello-go
```








