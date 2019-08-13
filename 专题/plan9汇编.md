# plan9汇编

由于Go的开发者之一也是plan9汇编的开发者，自然而然会用到自己的东西，在Go中plan9作为中间汇编语言

https://golang.org/doc/asm#introduction
> The most important thing to know about Go's assembler is that it is not a direct representation of the underlying machine. Some of the details map precisely to the machine, but some do not. This is because the compiler suite (see this description) needs no assembler pass in the usual pipeline. Instead, the compiler operates on a kind of semi-abstract instruction set, and instruction selection occurs partly after code generation. The assembler works on the semi-abstract form, so when you see an instruction like MOV what the toolchain actually generates for that operation might not be a move instruction at all, perhaps a clear or load. Or it might correspond exactly to the machine instruction with that name. In general, machine-specific operations tend to appear as themselves, while more general concepts like memory move and subroutine call and return are more abstract. The details vary with architecture, and we apologize for the imprecision; the situation is not well-defined.

关于Go汇编器需要最先明确的一点是，汇编器产生的汇编（plan9）并不是底层机器对应的汇编。有些汇编（plan9）会精确的映射到机器对应的汇编，有的则不会。这是因为编译器套件在通常的管道中不需要汇编程序传递。相反，编译器在一种半抽象指令集上操作，并且在生成汇编代码后部分发生指令选择。汇编程序以半抽象形式工作，因此当您看到像mov这样的指令时，工具链实际为该操作生成的内容可能根本不是move指令，可能是clear或load；当然，也可能精确的映射为底层机器的mov指令。更常见的概念，如内存移动和子协程调用和返回则会更抽象，通常在特定机器上的这些操作底层实现是其特有的。实现细节因架构而不同，我们为不精确而道歉，这种情况无法明确定义。



## 为什么需要了解plan9汇编

- plan9是Go语言编译后产生的半抽象汇编，理解Go以及自己的写的代码的底层需要理解部分plan9汇编

不过，最终真正产生的可执行程序中的机器码又因底层机器不同，不一定和plan9汇编都是对应的。

## 汇编代码中程序的基本分段

- .data 有初始化值的全局变量或定义的常量
- .bss 没有初始化值的全局变量
- .text 代码段
- .rodata 只读数据段

代码在.text段中，Go语言用户程序的入口为main.main

## Go程序编译阶段

```
        编译      ->   汇编      ->             链接  
    /         \     /     \              /                 \
main.go  ->   main.s   ->   main.o   +        go.o              ->    main  
原文件    ->   汇编文件  ->  用户机器码 + 所依赖的Go语言本身的机器码   ->  可执行文件
```
各阶段说明：
- 编译：检查语法，生成汇编
- 汇编：汇编代码转换机器码
- 链接：将用户机器码与Go语言本身机器码文件链接在一起生成可执行程序

在上述的三个阶段中，实际上每个阶段执行后还会进行优化



## 生成Go汇编

（1）源文件main.go -> 汇编main.s
	- go build -gcflags -S main.go
	- go tool compile -N -l -S main.go
这两种方式生成的汇编基本一致（能显示为汇编指令的行一致），但不完全相同（似乎是一些地址？），是否额外参数影响了汇编结果？

（2）机器码main.o  -> 汇编main.s
	首先go tool compile -N -l main.go -> main.o
	然后go tool objdump main.o
 

 (3) 可执行文件main -> 汇编main.s + 所依赖的Go语言本身的汇编go.s 
 	首先go build -gcflags main.go  -> main
 	然后go tool objdump main


通常用go build -gcflags -S main.go即可


注意：
编译生成的汇编是plan9语法+go自定义的语法，而汇编时候应该把用户代码的plan9汇编转为对应机器的汇编、再转化为对应机器的机器码，而最终进行链接的时候，将用户代码对应的机器码与go对应机器下的机器码链接在一起，最后生成可执行文件

compile和objdump命令说明：
https://golang.org/cmd/compile/
https://golang.org/cmd/objdump/


## 解读一个简单的Go汇编

源码
```golang
package main

import "fmt"

func add(a, b int) int {
        return a + b
}

func main() {
        fmt.Println(add(2, 3))
}
```
go build -gcflags -S main.go

汇编代码 (go version go1.11.12 darwin/amd64)
```
# command-line-arguments
"".add STEXT nosplit size=19 args=0x18 locals=0x0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	TEXT	"".add(SB), NOSPLIT, $0-24
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	FUNCDATA	$0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	FUNCDATA	$3, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:6)	PCDATA	$2, $0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:6)	PCDATA	$0, $0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:6)	MOVQ	"".b+16(SP), AX
	0x0005 00005 (/Users/sagacioushugo/go/src/main.go:6)	MOVQ	"".a+8(SP), CX
	0x000a 00010 (/Users/sagacioushugo/go/src/main.go:6)	ADDQ	CX, AX
	0x000d 00013 (/Users/sagacioushugo/go/src/main.go:6)	MOVQ	AX, "".~r2+24(SP)
	0x0012 00018 (/Users/sagacioushugo/go/src/main.go:6)	RET
	0x0000 48 8b 44 24 10 48 8b 4c 24 08 48 01 c8 48 89 44  H.D$.H.L$.H..H.D
	0x0010 24 18 c3                                         $..
"".main STEXT size=134 args=0x0 locals=0x48
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:9)	TEXT	"".main(SB), $72-0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:9)	MOVQ	(TLS), CX
	0x0009 00009 (/Users/sagacioushugo/go/src/main.go:9)	CMPQ	SP, 16(CX)
	0x000d 00013 (/Users/sagacioushugo/go/src/main.go:9)	JLS	124
	0x000f 00015 (/Users/sagacioushugo/go/src/main.go:9)	SUBQ	$72, SP
	0x0013 00019 (/Users/sagacioushugo/go/src/main.go:9)	MOVQ	BP, 64(SP)
	0x0018 00024 (/Users/sagacioushugo/go/src/main.go:9)	LEAQ	64(SP), BP
	0x001d 00029 (/Users/sagacioushugo/go/src/main.go:9)	FUNCDATA	$0, gclocals·69c1753bd5f81501d95132d08af04464(SB)
	0x001d 00029 (/Users/sagacioushugo/go/src/main.go:9)	FUNCDATA	$1, gclocals·568470801006e5c0dc3947ea998fe279(SB)
	0x001d 00029 (/Users/sagacioushugo/go/src/main.go:9)	FUNCDATA	$3, gclocals·1cf923758aae2e428391d1783fe59973(SB)
	0x001d 00029 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $0
	0x001d 00029 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$0, $1
	0x001d 00029 (/Users/sagacioushugo/go/src/main.go:10)	XORPS	X0, X0
	0x0020 00032 (/Users/sagacioushugo/go/src/main.go:10)	MOVUPS	X0, ""..autotmp_3+48(SP)
	0x0025 00037 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $1
	0x0025 00037 (/Users/sagacioushugo/go/src/main.go:10)	LEAQ	type.int(SB), AX
	0x002c 00044 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $0
	0x002c 00044 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	AX, (SP)
	0x0030 00048 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	$5, 8(SP)
	0x0039 00057 (/Users/sagacioushugo/go/src/main.go:10)	CALL	runtime.convT2E64(SB)
	0x003e 00062 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	16(SP), AX
	0x0043 00067 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $2
	0x0043 00067 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	24(SP), CX
	0x0048 00072 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	AX, ""..autotmp_3+48(SP)
	0x004d 00077 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $0
	0x004d 00077 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	CX, ""..autotmp_3+56(SP)
	0x0052 00082 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $1
	0x0052 00082 (/Users/sagacioushugo/go/src/main.go:10)	LEAQ	""..autotmp_3+48(SP), AX
	0x0057 00087 (/Users/sagacioushugo/go/src/main.go:10)	PCDATA	$2, $0
	0x0057 00087 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	AX, (SP)
	0x005b 00091 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	$1, 8(SP)
	0x0064 00100 (/Users/sagacioushugo/go/src/main.go:10)	MOVQ	$1, 16(SP)
	0x006d 00109 (/Users/sagacioushugo/go/src/main.go:10)	CALL	fmt.Println(SB)
	0x0072 00114 (/Users/sagacioushugo/go/src/main.go:11)	PCDATA	$0, $0
	0x0072 00114 (/Users/sagacioushugo/go/src/main.go:11)	MOVQ	64(SP), BP
	0x0077 00119 (/Users/sagacioushugo/go/src/main.go:11)	ADDQ	$72, SP
	0x007b 00123 (/Users/sagacioushugo/go/src/main.go:11)	RET
	0x007c 00124 (/Users/sagacioushugo/go/src/main.go:11)	NOP
	0x007c 00124 (/Users/sagacioushugo/go/src/main.go:9)	PCDATA	$0, $-1
	0x007c 00124 (/Users/sagacioushugo/go/src/main.go:9)	PCDATA	$2, $-1
	0x007c 00124 (/Users/sagacioushugo/go/src/main.go:9)	CALL	runtime.morestack_noctxt(SB)
	0x0081 00129 (/Users/sagacioushugo/go/src/main.go:9)	JMP	0
	0x0000 65 48 8b 0c 25 00 00 00 00 48 3b 61 10 76 6d 48  eH..%....H;a.vmH
	0x0010 83 ec 48 48 89 6c 24 40 48 8d 6c 24 40 0f 57 c0  ..HH.l$@H.l$@.W.
	0x0020 0f 11 44 24 30 48 8d 05 00 00 00 00 48 89 04 24  ..D$0H......H..$
	0x0030 48 c7 44 24 08 05 00 00 00 e8 00 00 00 00 48 8b  H.D$..........H.
	0x0040 44 24 10 48 8b 4c 24 18 48 89 44 24 30 48 89 4c  D$.H.L$.H.D$0H.L
	0x0050 24 38 48 8d 44 24 30 48 89 04 24 48 c7 44 24 08  $8H.D$0H..$H.D$.
	0x0060 01 00 00 00 48 c7 44 24 10 01 00 00 00 e8 00 00  ....H.D$........
	0x0070 00 00 48 8b 6c 24 40 48 83 c4 48 c3 e8 00 00 00  ..H.l$@H..H.....
	0x0080 00 e9 7a ff ff ff                                ..z...
	rel 5+4 t=16 TLS+0
	rel 40+4 t=15 type.int+0
	rel 58+4 t=8 runtime.convT2E64+0
	rel 110+4 t=8 fmt.Println+0
	rel 125+4 t=8 runtime.morestack_noctxt+0
// 省略后续自动生成的汇编
```

其中
```golang
// 源码
func add(a, b int) int {
        return a + b
}
```
对应
```
// 汇编代码
// func add(a, b int) int
"".add STEXT nosplit size=19 args=0x18 locals=0x0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	TEXT	"".add(SB), NOSPLIT, $0-24
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	FUNCDATA	$0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:5)	FUNCDATA	$3, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:6)	PCDATA	$2, $0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:6)	PCDATA	$0, $0
	0x0000 00000 (/Users/sagacioushugo/go/src/main.go:6)	MOVQ	"".b+16(SP), AX
	0x0005 00005 (/Users/sagacioushugo/go/src/main.go:6)	MOVQ	"".a+8(SP), CX
	0x000a 00010 (/Users/sagacioushugo/go/src/main.go:6)	ADDQ	CX, AX
	0x000d 00013 (/Users/sagacioushugo/go/src/main.go:6)	MOVQ	AX, "".~r2+24(SP)
	0x0012 00018 (/Users/sagacioushugo/go/src/main.go:6)	RET
	0x0000 48 8b 44 24 10 48 8b 4c 24 08 48 01 c8 48 89 44  H.D$.H.L$.H..H.D
	0x0010 24 18 c3                                         $..
```

## 汇编函数结构
![image](images/asm_func.png)
```
                            参数+返回值的大小
                                  | 
 TEXT pkgname·add(SB),NOSPLIT,$32-32
       |        |              |
      包名     函数名         栈帧大小(局部变量+可能需要的额外调用函数的参数空间的总大小，但不包括调用其它函数时的 ret address 的大小)
```

需要注意的是方法(method)与函数(func不同)，汇编方法结构：
```
                            参数接收者+参数+返回值的大小
                                       | 
 TEXT pkgname·type·add(SB),NOSPLIT,$32-32
       |        |   |               |
      包名     类型 函数名     栈帧大小(局部变量+可能需要的额外调用函数的参数空间的总大小，但不包括调用其它函数时的 ret address 的大小)
```
Go 的汇编还引入了 4 个伪寄存器

>- `FP`: Frame pointer: arguments and locals.
>- `PC`: Program counter: jumps and branches.
>- `SB`: Static base pointer: global symbols.
>- `SP`: Stack pointer: top of stack.


>官方的描述稍微有一些问题，我们对这些说明进行一点扩充:
>- FP:  使用形如 `symbol+offset(FP)` 的方式，引用函数的输入参数。例如 `arg0+0(FP)`，`arg1+8(FP)`，使用 FP 不加 symbol 时，无法通过编译，在汇编层面来讲，symbol 并没有什么用，加 symbol 主要是为了提升代码可读性。另外，官方文档虽然将伪寄存器 FP 称之为 frame pointer，实际上它根本不是 frame pointer，按照传统的 x86 的习惯来讲，frame pointer 是指向整个 stack frame 底部的 BP 寄存器。假如当前的 callee 函数是 add，在 add 的代码中引用 FP，该 FP 指向的位置不在 callee 的 stack frame 之内，而是在 caller 的 stack frame 上。具体可参见之后的 **栈结构** 一章。
>- PC: 实际上就是在体系结构的知识中常见的 pc 寄存器，在 x86 平台下对应 ip 寄存器，amd64 上则是 rip。除了个别跳转之外，手写 plan9 代码与 PC 寄存器打交道的情况较少。
>- SB: 全局静态基指针，一般用来声明函数或全局变量，在之后的函数知识和示例部分会看到具体用法。
>- SP: plan9 的这个 SP 寄存器指向当前栈帧的局部变量的开始位置，使用形如 `symbol+offset(SP)` 的方式，引用函数的局部变量。offset 的合法取值是 [-framesize, 0)，注意是个左闭右开的区间。假如局部变量都是 8 字节，那么第一个局部变量就可以用 `localvar0-8(SP)` 来表示。这也是一个词不表意的寄存器。与硬件寄存器 SP 是两个不同的东西，在栈帧 size 为 0 的情况下，伪寄存器 SP 和硬件寄存器 SP 指向同一位置。手写汇编代码时，如果是 `symbol+offset(SP)` 形式，则表示伪寄存器 SP。如果是 `offset(SP)` 则表示硬件寄存器 SP。务必注意。对于编译输出(go tool compile -S / go tool objdump)的代码来讲，目前所有的 SP 都是硬件寄存器 SP，无论是否带 symbol。

>我们这里对容易混淆的几点简单进行说明：
>1. 伪 SP 和硬件 SP 不是一回事，在手写代码时，伪 SP 和硬件 SP 的区分方法是看该 SP 前是否有 symbol。如果有 symbol，那么即为伪寄存器，如果没有，那么说明是硬件 SP 寄存器。
>2. SP 和 FP 的相对位置是会变的，所以不应该尝试用伪 SP 寄存器去找那些用 FP + offset 来引用的值，例如函数的入参和返回值。
>3. 官方文档中说的伪 SP 指向 stack 的 top，是有问题的。其指向的局部变量位置实际上是整个栈的栈底(除 caller BP 之外)，所以说 bottom 更合适一些。
>4. 在 go tool objdump/go tool compile -S 输出的代码中，是没有伪 SP 和 FP 寄存器的，我们上面说的区分伪 SP 和硬件 SP 寄存器的方法，对于上述两个命令的输出结果是没法使用的。在编译和反汇编的结果中，只有真实的 SP 寄存器。
>5. FP 和 Go 的官方源代码里的 framepointer 不是一回事，源代码里的 framepointer 指的是 caller BP 寄存器的值，在这里和 caller 的伪 SP 是值是相等的。




## 参考资料
https://github.com/cch123/asmshare/blob/master/layout.md 第34期 Go夜读之《plan9汇编入门，带你打通应用和底层》
http://xargin.com/plan9-assembly/ Go 系列文章3 ：plan9 汇编入门
https://golang.org/doc/asm 官方文档
https://studygolang.com/articles/18124?fr=sidebar Go工具和调试详解

