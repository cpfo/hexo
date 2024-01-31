---
title: go语言结构和基础语法
categories: [技术]
tags: [go]
date: 2024-01-31 16:13:04
---

简单介绍下go语言的程序结构，基础语法和数据类型。

<!-- more -->

# 结构示例

go语言的基础组成包含以下部分

* 包声明
* 引入包 
* 函数
* 变量
* 语句和表达式
* 注释

以下面代码为例

```go
package main

import "fmt"

func main() {
    /*输出hello world*/
    fmt.Println("hello, world")
}

```

1. 第一行代码 `package main` 定义了包名。必须在源文件中非注释的第一行指明这个文件属于哪个包，如：`package main`。`package main`表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 `main` 的包。
2. 下一行 `import "fmt"` 告诉 Go 编译器这个程序需要使用 `fmt` 包，`fmt` 包实现了格式化 IO（输入/输出）的函数。
3. 下一行 `func main()` 是程序开始执行的函数。`main` 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 `init()` 函数则会先执行该函数）
4. `/**/` 是注释。可以在任何地方使用以 `//` 开头的单行注释。多行注释也叫块注释，均已以 `/*` 开头，并以 `*/` 结尾
5. `fmt.Println()` 将字符串输出到控制台
6. 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如`Print`，那么使用这种形式的标识符的对象就可以被外部包的代码所使用(需要先引入)，这被称为导出(像java中的public)，标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像java的`protected` ）


# 运行代码

运行go文件

> go run hello.go

也可以生成二进制文件

> go build hello.go

`{` 不能单独放在一行， 和java不同。


# 基础语法

* 行分隔符

语句结束是由分号（;）标志，但是代码不需要添加，Go编译器会自动在每行结束的地方插入分号。一行代表一个语句结束。

* 注释

```
// 单行注释

/*
多行注释
*/
```

* 标识符

标识符用来命名变量、类型等程序实体。

必须是字母数字下划线组成，但是第一个字符必须是字母或者下划线，不能是数字。

* 字符串连接

字符串连接使用 `+` 操作

> fmt.Println("hello, " + "world")

* 关键字

有25个关键字

```shell
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

36个预定义标识符

```
append       bool         byte         cap          close        complex      
complex64    complex128   uint16       copy         false        float32      
float64      imag         int          int8         int16        uint32       
int32        int64        iota         len          make         new          
nil          panic        print        println      real         recover      
string       true         uint         uint8        uint64       uintptr      
```

程序一般由关键字、常量、变量、运算符、类型和函数组成。

* 空格

空格通常用于分隔标识符、关键字、运算符和表达式，以提高代码的可读性。

1. 变量的声明必须使用空格隔开
2. 关键字和表达式之间要使用空格

* 格式化字符串

Go 语言中使用 `fmt.Sprintf` 或 `fmt.Printf` 格式化字符串并赋值给新串

`Sprintf` 根据格式化参数生成格式化的字符串并返回该字符串
`Printf`  根据格式化参数生成格式化的字符串并写入标准输出


```go
package main

import "fmt"

func main() {
	/*输出hello world*/
	fmt.Println("hello, world")
	fmt.Println("hello, " + "world")

	var url = "name=%s&num=%d"
	var num = 1
	var name = "张三"
	var result = fmt.Sprintf(url, name, num)
	fmt.Println(result)
	fmt.Printf(url, name, num)
}

```
# 数据类型

数据类型用于声明函数和变量。数据类型是为了把数据分成内存大小不同的数据，编程的时候需要用大数据的时候才需要申请大内存，就可以充分利用内存。


1. 布尔型

布尔型只能是true或者false， 例如 `var b bool = true`

2. 数字类型

整型 int 和浮点型 float32、float64，Go 语言支持整型和浮点型数字，并且支持复数，其中位的运算采用补码。

还区分有符号和无符号的

|  类型 |  描述 |
| ----- | --------- | 
| uint8  |  无符号 8 位整型 (0 到 255) | 
| uint16 |  无符号 16 位整型 (0 到 65535) |
| uint32 |  无符号 32 位整型 (0 到 4294967295) |
| uint64 | 无符号 64 位整型  |
| int8   |  有符号 8 位整型 (-128 到 127) |
| int16  |  有符号 16 位整型 (-32768 到 32767) |
| int32  |  有符号 32 位整型 (-2147483648 到 2147483647) |
| int64  |  有符号 64 位整型 |


3. 字符串类型

字符串就是一串固定长度的字符连接起来的字符序列。Go 的字符串是由单个字节连接起来的。使用 `UTF-8` 编码

4. 派生类型

   * 指针类型（Pointer）
   * 数组类型
   * 结构化类型(struct)
   * Channel 类型
   * 函数类型
   * 切片类型
   * 接口类型（interface）
   * Map 类型

派生类型(也称为复合类型)
    
5. 其他数字类型


| 类型 | 描述 |
| --- | ----|
|byte| 类似 uint8 |
|rune  |类似 int32  |
|uint |32 或 64 位  |
|int  |与 uint 一样大小  |
|uintptr  |无符号整型，用于存放一个指针  |