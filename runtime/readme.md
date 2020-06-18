## 系统环境

```
[root@jordy tmp]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.6.1810 (Core) 
Release:	7.6.1810
Codename:	Core
[root@jordy tmp]# 
[root@jordy tmp]# uname -a 
Linux jordy 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
[root@jordy tmp]# 
[root@jordy tmp]# go version
go version go1.10.2 linux/amd64
```

## rumtime调试与跟踪
- 首先准备一个简单的源码文件hello.go
```go
package main
  
import (
    "fmt"
)

func main() {
    fmt.Println("hello world")
}
```

- go build编译源文件生成可执行文件
```
go build -gcflags "-N -l " -o hello hello.go
// -N 选项是禁止编译器优化(disable optimizations) 
// -l  选项是禁止内联(disable inlining )
//编译器更多选项，可参考帮助文档	(go tool compile --help)
```

- 试运行
```
./hello 
hello world
```
- 下面我们用gdb来逐步调试与分析可执行程序hello，以揭开golang 程序执行入口的本质与runtime精髓.
```
[root@jordy tmp]# gdb hello
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-114.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /tmp/hello...done.
warning: File "/usr/local/src/golang/go/src/runtime/runtime-gdb.py" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load:/usr/bin/mono-gdb.py".
To enable execution of this file add
	add-auto-load-safe-path /usr/local/src/golang/go/src/runtime/runtime-gdb.py
line to your configuration file "/root/.gdbinit".
To completely disable this security protection add
	set auto-load safe-path /
line to your configuration file "/root/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
	info "(gdb)Auto-loading safe path"
(gdb) info files
Symbols from "/tmp/hello".
Local exec file:
	`/tmp/hello', file type elf64-x86-64.
	Entry point: 0x44f4d0
	0x0000000000401000 - 0x0000000000482178 is .text
	0x0000000000483000 - 0x00000000004c4a7a is .rodata
	0x00000000004c4ba0 - 0x00000000004c56e8 is .typelink
	0x00000000004c56e8 - 0x00000000004c5728 is .itablink
	0x00000000004c5728 - 0x00000000004c5728 is .gosymtab
	0x00000000004c5740 - 0x0000000000513a3d is .gopclntab
	0x0000000000514000 - 0x0000000000520bdc is .noptrdata
	0x0000000000520be0 - 0x00000000005276f0 is .data
	0x0000000000527700 - 0x0000000000543d88 is .bss
	0x0000000000543da0 - 0x0000000000546438 is .noptrbss
	0x0000000000400f9c - 0x0000000000401000 is .note.go.buildid
(gdb) 

看到程序的入口地址为0x44f4d0，可以在此地址出打断点:
(gdb) b *0x44f4d0
Breakpoint 1 at 0x44f4d0: file /usr/local/src/golang/go/src/runtime/rt0_linux_amd64.s, line 8.
(gdb) run
Starting program: /tmp/hello 

Breakpoint 1, _rt0_amd64_linux () at /usr/local/src/golang/go/src/runtime/rt0_linux_amd64.s:8
8		JMP	_rt0_amd64(SB)

首先，go 可执行首先执行的是一个汇编程序rt0_linux_amd64.s，在源码子目录/usr/local/src/golang/go/src/runtime/中，
我们进一步看到，在该目录下go 支持了不同的操作系统，比如有linux,windows,darwin,solaris,plan9,android等等。
针对每种系统，对应一个汇编文件.s


可以在控制台直接运行 go tool dist list 来查看
```

- 查看入口文件并跟踪逻辑
```
在以上，我们初步定位并找到了当前系统(linux 64）所对应的入口文件rt0_linux_amd64.s，该文件源码见下：
  1 // Copyright 2009 The Go Authors. All rights reserved.
  2 // Use of this source code is governed by a BSD-style
  3 // license that can be found in the LICENSE file.
  4 
  5 #include "textflag.h"
  6 
  7 TEXT _rt0_amd64_linux(SB),NOSPLIT,$-8
  8     JMP _rt0_amd64(SB)//此行为第8行
  9 
 10 TEXT _rt0_amd64_linux_lib(SB),NOSPLIT,$0
 11     JMP _rt0_amd64_lib(SB)

我们看到，在第8行程序无条件跳转并调用了_rt0_amd64函数; 
JMP 指令是无条件跳转的意思，具体可参考go汇编的资料，如
https://xargin.com/plan9-assembly/

rt0_amd64函数在那？ 可以继续设置断点查看：
(gdb) b _rt0_amd64
Breakpoint 2 at 0x44be00: file /usr/local/src/golang/go/src/runtime/asm_amd64.s, line 15.
(gdb)

我们看到_rt0_amd64在asm_amd64.s 中定义，如
 1 // Copyright 2009 The Go Authors. All rights reserved.
   2 // Use of this source code is governed by a BSD-style
   3 // license that can be found in the LICENSE file.
   4 
   5 #include "go_asm.h"
   6 #include "go_tls.h"
   7 #include "funcdata.h"
   8 #include "textflag.h"
   9 
  10 // _rt0_amd64 is common startup code for most amd64 systems when using
  11 // internal linking. This is the entry point for the program from the
  12 // kernel for an ordinary -buildmode=exe program. The stack holds the
  13 // number of arguments and the C-style argv.
  14 TEXT _rt0_amd64(SB),NOSPLIT,$-8
  15     MOVQ    0(SP), DI   // argc
  16     LEAQ    8(SP), SI   // argv
  17     JMP runtime·rt0_go(SB)
  18 

看注释信息：当使用内部链接时，_rt0_amd64是大多数amd64系统的常用启动代码，
证明这是一个共用的封装，几乎大部分的64位系统都调用此封装；
看到在_rt0_amd64中，代码逻辑进一步无条件跳转到了 runtime·rt0_go函数， runtime·rt0_go在哪？继续打断点跟踪：

//注意要将·换成. 
(gdb) b runtime.rt0_go 
Breakpoint 3 at 0x44be10: file /usr/local/src/golang/go/src/runtime/asm_amd64.s, line 89.

看到rt0_go定义位于asm_amd64.s文件的约89行，好戏才刚刚开始，对于asm_amd64.s文件
我们逐步跟踪：rt0_go->nocpuinfo->ok
看到在ok 块中调用了几个核心步骤，正是在该块中，完成了初始化和运行时的核心步骤

231 ok:
 232     // set the per-goroutine and per-mach "registers"
 233     get_tls(BX)
 234     LEAQ    runtime·g0(SB), CX
 235     MOVQ    CX, g(BX)
 236     LEAQ    runtime·m0(SB), AX
 237 
 238     // save m->g0 = g0
 239     MOVQ    CX, m_g0(AX)
 240     // save m0 to g0->m
 241     MOVQ    AX, g_m(CX)
 242 
 243     CLD             // convention is D is always left cleared
 244     CALL    runtime·check(SB)
 245 
 246     MOVL    16(SP), AX      // copy argc
 247     MOVL    AX, 0(SP)
 248     MOVQ    24(SP), AX      // copy argv
 249     MOVQ    AX, 8(SP)
 250     CALL    runtime·args(SB)
 251     CALL    runtime·osinit(SB)
 252     CALL    runtime·schedinit(SB)
 253 
 254     // create a new goroutine to start program
 255     MOVQ    $runtime·mainPC(SB), AX     // entry
 256     PUSHQ   AX
 257     PUSHQ   $0          // arg size
 258     CALL    runtime·newproc(SB)
 259     POPQ    AX
 260     POPQ    AX
 261 
 262     // start this M
 263     CALL    runtime·mstart(SB)
 264 
 265     MOVL    $0xf1, 0xf1  // crash
 266     RET
 267 
 268 DATA    runtime·mainPC+0(SB)/8,$runtime·main(SB)
 269 GLOBL   runtime·mainPC(SB),RODATA,$8

初步看到，初始化过程陆续调用了几个关键的函数，如：
runtime·args(SB)，runtime·osinit(SB)，runtime·schedinit(SB)，runtime·mainPC(SB)，runtime·newproc(SB)，runtime·mstart(SB)，runtime·main(SB)

可以清洗地看到，在执行咱们的main.main之前，运行时进行了多个其他的步骤以完成初始化工作，即得出结论：
main.main并不是go 可执行的入口！
下面我们来专门的分析初始化的过程，了解golang runtime启动时的精髓.

```





