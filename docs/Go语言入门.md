# 概述

Go是一门编译型语言（静态编译），Go会将源代码及其依赖直接转换为机器指令，然后执行。

Go的优点：语言层面支持并发，实现简单，高效利用多核；简洁高效；有垃圾回收机制，更安全

几个常用的命令行：

- go run xx.go，编译并运行
- go build xx.go，仅编译（1.17后推荐使用go install）
- go get xxx，获取并更新某个依赖，前提是安装好了git
- gofmt，它是用来格式化go源代码的

Go 语言不需要在语句或者声明的末尾添加分号，除非一行上有多条语句，编译器会自动将换行符转换为分号，但是有几个符号除外：(、{、[、.、+等，所以go语言在写大括号的时候，左括号不能单独起一行

函数的右小括弧也可以另起一行缩进，同时为了防止编译器在行尾自动插入分号而导致的编译错误，可以在末尾的参数变量后面显式插入逗号：

```Go
img.SetColorIndex(
    size+int(x*size+0.5), size+int(y*size+0.5),
    blackIndex, // 最后插入的逗号不会导致编译错误，这是Go编译器的一个特性
)               // 小括弧另起一行缩进，和大括弧的风格保存一致
```

# 程序结构

## 包

包出现的目的是为了支持模块化、封装、单独编译和代码重用。

一个包的源代码保存在一个或多个以.go为文件后缀名的源文件中，通常一个包所在目录路径的后缀是包的导入路径；例如包gopl.io/ch1/helloworld对应的目录路径是$GOPATH/src/gopl.io/ch1/helloworld。

按照惯例，一个包的名字和包的导入路径的最后一个字段相同，例如gopl.io/ch2/tempconv包的名字一般是tempconv。导入语句import将导入的包绑定到一个短小的名字，然后通过该短小的名字就可以引用包中导出的全部内容。

默认包名一般采用导入路径名的最后一段，这是一个约定，但是有下面几个例外情况：

- 包对应一个可执行程序，也就是main包，这时候main包本身的导入路径是无关紧要的。
- 包所在的目录中可能有一些文件名是以`_test.go`为后缀的Go源文件，这些源文件声明的包名也是以`_test`为后缀名的。所有以`_test`为后缀包名的测试外部扩展包都由go test命令独立编译，普通包和测试的外部扩展包是相互独立的。测试的外部扩展包一般用来避免测试代码中的循环导入依赖。
- 一些依赖版本号的管理工具会在导入路径后追加版本号信息，例如“gopkg.in/yaml.v2”。这种情况下包的名字并不包含版本号后缀，而是yaml

如果导入了一个包，但是又没有使用该包将被当作一个编译错误处理。这种强制规则可以有效减少不必要的依赖。

可以在 https://golang.org/pkg 和 https://godoc.org 中找到标准库和社区写的package。godoc这个工具可以让你直接在本地命令行阅读标准库的文档：

```Go
$ go doc http.ListenAndServe
package http // import "net/http"
func ListenAndServe(addr string, handler Handler) error
    ListenAndServe listens on the TCP network address addr and then
    calls Serve with handler to handle requests on incoming connections.
...
```

在Go中，禁止包的环状依赖，包的依赖关系形成一个有向无环图，每个包可以被独立编译，且可以被并发编译，所以Go的编译速度很快。

### 包导入

如果想同时导入两个名字相同的包，导入声明中必须至少为其中一个指定新的包名避免冲突，这就是导入包的重命名：

```Go
import (
    "crypto/rand"
    mrand "math/rand" // alternative name mrand avoids conflict
)
```

导入包的重命名不仅可以解决包名冲突，当包名很笨重想换一个简单的也可以用，或者包名和本地变量名冲突。

有时候我们只是想利用导入包而产生的副作用：它会计算包级变量的初始化表达式和执行导入包的init初始化函数，而不是直接使用包里面的内容，此时应该使用包的匿名导入：

```Go
import _ "image/png" // register PNG decoder
```

数据库包database/sql也是采用了类似的技术，让用户可以根据自己需要选择导入必要的数据库驱动，在使用其他库函数的时候，程序就能正常工作了。

### 包初始化

包初始化的时候，会根据依赖的顺序+声明的顺序进行包级变量的初始化：

```Go
var a = b + c // a 第三个初始化, 为 3
var b = f()   // b 第二个初始化, 为 2, 通过调用 f (依赖c)
var c = 1     // c 第一个初始化, 为 1

func f() int { return c + 1 }
```

如果包中含有多个go源文件，构建工具会将这些文件按照文件名排序，然后依次用编译器编译。

包级别的变量可以用表达式初始化，也可以用函数初始化：

```Go
// pc[i] is the population count of i.
var pc [256]byte = func() (pc [256]byte) {
    for i := range pc {
        pc[i] = pc[i/2] + byte(i&1)
    }
    return
}()
```

当初始化的工作比较复杂时，也可以用一个特殊的init初始化函数来简化初始化工作，每个文件都可以包含多个init初始化函数，按声明顺序调用：

```Go
func init() { /* ... */ }
```

该函数是不能被调用或引用的，它只会在程序开始时被调用。

当代码中导入了多个包的时候，会按照导入的顺序初始化它们，每个包只会被初始化一次。初始化工作是自下而上进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依赖的包都已经完成初始化工作了。

### 内部包

除了可导入和不可导入两个状态以外，还有第三种状态：标识符对于一小部分信任的包是可见的，但并不是对所有调用者都可见。

有时候会有这种场景：在重构的过程中，计划将一个大的包拆分为多个小的包，但是不想把所有子包都暴漏出去，同时还希望保留一些在内部子包之间共享一些通用的包。或者有时新包存在还并不稳定的接口，暂时只暴露给一些受限制的用户使用。

为了满足这些场景，Go语言的构建工具对包含internal名字的路径段的包导入路径做了特殊处理，这种包叫internal包。一个internal包只能被和internal目录有同一个父目录的包所导入。例如，net/http/internal/chunked内部包只能被net/http/httputil或net/http包导入，但是不能被net/url包导入。不过net/url包却可以导入net/http/httputil包。

## 输入与输出

最简单的输出函数：

```Go
fmt.Println(strings.Join(os.Args[1:], " "))
```

格式化输出：

```Go
fmt.Printf("%d\t%s\n", n, line)
```

常用的转换字符：

```Go
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

特别地，%#v代表用类似Go语言的语法打印值。

可以用fmt包来控制输出的进制：

```Go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

几个要说明的点：

- %之后的[1]代表再次使用第一个操作数，当参数少但是转换字符多的时候，就采用这种方式
- %后的#代表生成特殊进制的前缀，如0、0x或0X前缀

打印浮点数时，可以指定打印的宽度和控制打印精度：

```Go
for x := 0; x < 8; x++ {
    fmt.Printf("x = %d e^x = %8.3f\n", x, math.Exp(float64(x)))
}

x = 0       e^x =    1.000
x = 1       e^x =    2.718
x = 2       e^x =    7.389
x = 3       e^x =   20.086
x = 4       e^x =   54.598
x = 5       e^x =  148.413
x = 6       e^x =  403.429
x = 7       e^x = 1096.633
```

上面代码打印e的幂，打印精度是小数点后三个小数精度和8个字符宽度。

常用的转义字符：制表符`\t`和换行符`\n`

```Go
\a      响铃
\b      退格
\f      换页
\n      换行
\r      回车
\t      制表符
\v      垂直制表符
\'      单引号（只用在 '\'' 形式的rune符号面值中）
\"      双引号（只用在 "..." 形式的字符串面值中）
\\      反斜杠
```

f结尾的都是格式化函数，如`log.Printf` 和 `fmt.Errorf`；ln结尾的都是自带换行的，以跟 `%v` 差不多的方式格式化参数，并在最后添加一个换行符

输出函数也有多种，fmt.Printf负责将结果写到标准输出，fmt.Sprintf负责将结果以字符串的形式返回，fmt.Fprintf负责将结果写入文件中。

输入的方法：

```Go
input := bufio.NewScanner(os.Stdin)
for input.Scan() {
    counts[input.Text()]++
}
```

每次调用 `input.Scan()`，即读入下一行，并移除行末的换行符，读取的内容可以调用 `input.Text()` 得到。`Scan` 函数在读到一行时返回 `true`，不再有输入时返回 `false`

ioutil.Discard是一个特殊的输入流，可以吧不想要的内容输出到这里，只返回字节数：

```Go
nbytes, err := io.Copy(ioutil.Discard, resp.Body)
```

所有的输出流都是io.Writer

## 命令行参数

main方法是没有入参的，需要从os包获取命令行参数

`os.Args` 变量是一个字符串（string）的 *切片*（slice），它的第一个元素是命令名，其他元素是程序启动时传给它的参数

## 注释

注释语句以//开头。

如果要给一个包添加注释，一般加在package之前，文件的第一行。一个包通常只有一个源文件有包注释（如果有多个包注释，目前的文档工具会根据源文件名的先后顺序将它们链接为一个包注释）。如果包注释很大，通常会放到一个独立的doc.go文件中

给每一个函数写注释也是一个好习惯，这些内容都会被像godoc这样的工具检测到，并且在执行命令时显示这些注释。

多行注释可以用 `/* ... */` 来包裹，在文件一开头的注释一般都是这种形式

## 命名

Go中所有的命名都遵循一个规则：一个名字必须以一个字母（Unicode字母）或下划线开头，后面可以跟任意数量的字母、数字或下划线。

包本身的名字一般总是用小写字母。

Go语言的风格是尽量使用短小的名字，推荐使用 **驼峰式** 命名，当名字由几个单词组成时优先使用大小写分隔，而不是优先用下划线分隔。

可见性：

- 如果一个名字是在函数内部定义，那么它就只在函数内部有效。如果是在函数外部定义，那么将在当前包的所有文件中都可以访问。
- 名字的开头字母的大小写决定了名字在包外的可见性，大写代表导出的，即可以被外部的包访问

## 变量

### 声明

Go语言主要有四种类型的声明语句：var、const、type和func，分别对应变量、常量、类型和函数实体对象的声明。

`var` 声明可以声明多个变量。变量会在声明时直接初始化。如果变量没有显式初始化，则被隐式初始化其类型的零值，数值类型是 `0`，字符串类型是空字符串 `""`，布尔类型变量对应的零值是false，接口或引用类型（包括slice、指针、map、chan和函数）变量对应的零值是nil

```Go
var s, sep string
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
var f, err = os.Open(name) // os.Open returns a file and an error
```

符号 `:=` 是 *短变量声明*（short variable declaration），代表定义同时赋值，它只能在函数内部使用，它是最常用的，var形式的声明语句往往是用于需要显式指定变量类型的地方，或者是因为变量稍后会被重新赋值而初始值无关紧要的地方。短变量声明也可以用来声明和初始化一组变量：

```Bash
i, j := 0, 1
i, j = j, i // 交换 i 和 j 的值
```

上面这种声明常用于for语句的循环初始化语句部分

下面几种声明方式是等价的，实际使用中有一些区别：

```Go
s := ""    // 它最简洁，但只能用于函数内部，不能用于包变量
var s string    
var s = ""    // 同时声明多个变量时用这个
var s string = ""    // 当初始类型和变量值不是同一个类型的时候，用这种
```

简短变量声明语句也可以用函数的返回值来声明和初始化变量：

```Bash
f, err := os.Open(name)
```

允许短变量声明语句中一部分左边的变量已经声明过了，这种情况下对该变量只有赋值行为，但是必须必须至少要声明一个新的变量，否则代码不能编译通过。

简短变量声明语句只有对已经在同级词法域声明过的变量才和赋值操作语句等价，如果变量是在外部词法域声明的，那么简短变量声明语句将会在当前词法域重新声明一个新的变量。

值类型和引用类型：

- 值类型：整型、浮点型、布尔、string、数组和结构体
- 引用类型：指针、slice、map、管道chan、interface

### new函数

new函数可以创建变量，表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量地址，返回的指针类型为`*T`：

```Bash
p := new(int)   // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) // "0"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
```

每次调用new函数都是返回一个新的变量的地址，存在一种特殊情况：如果两个类型都是空的，也就是说类型的大小是0，例如`struct{}`和`[0]int`，有可能有相同的地址（依赖具体的语言实现）

### 生命周期

变量的生命周期：

- 对于在包一级声明的变量来说，它们的生命周期和整个程序的运行周期是一致的。
- 局部变量的生命周期则是动态的：每次从创建一个新变量的声明语句开始，直到该变量不再被引用为止，然后变量的存储空间可能被回收。

变量既可以分配在栈上，也可以分配在堆中。在栈中分配的变量，在函数调用完毕后就会被回收，在堆中分配的变量，由Go语言的GC机制决定回收还是不回收。

如果局部变量可以从函数中逃逸，那么它有可能暂时不会被回收，逃逸的变量需要额外分配内存，同时对性能的优化可能会产生细微的影响。

如果将指向短生命周期对象的指针保存到具有长生命周期的对象中，特别是保存到全局变量时，会阻止对短生命周期对象的垃圾回收（从而可能影响程序的性能）

### 赋值

元祖赋值允许同时更新多个变量的值，在赋值之前，赋值语句右边的所有表达式将会先进行求值，然后再统一更新左边对应变量的值：

```Bash
func gcd(x, y int) int {
    for y != 0 {
        x, y = y, x%y
    }
    return x
}
```

### 类型

可以用type关键字创建新的类型：type 类型名字 底层类型

下面的例子就将不同温度单位分别定义为不同的类型：

```Bash
// Package tempconv performs Celsius and Fahrenheit temperature computations.
package tempconv

import "fmt"

type Celsius float64    // 摄氏温度
type Fahrenheit float64 // 华氏温度

const (
    AbsoluteZeroC Celsius = -273.15 // 绝对零度
    FreezingC     Celsius = 0       // 结冰点温度
    BoilingC      Celsius = 100     // 沸水温度
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

它们虽然有着相同的底层类型float64，但是它们是不同的数据类型，因此它们不可以被相互比较或混在一个表达式运算。

对于每一个类型T，都有一个对应的类型转换操作T(x)，用于将x转为T类型。如果T是指针类型，可能会需要用小括弧包装T，比如`(*int)(0)`。只有当两个类型的底层基础类型相同时，才允许这种转型操作，或者是两者都是指向相同底层结构的指针类型。

一个类型的底层数据类型决定了内部结构和表达方式，之前定义的Celsius和Fahrenheit类型的算术运算行为和底层的float64类型是一样的：

```Bash
fmt.Printf("%g\n", BoilingC-FreezingC) // "100" °C
boilingF := CToF(BoilingC)
fmt.Printf("%g\n", boilingF-CToF(FreezingC)) // "180" °F
fmt.Printf("%g\n", boilingF-FreezingC)       // compile error: type mismatch
```

比较运算符可以用于相同类型之间的比较，或者与底层类型的未命名类型的值之间做比较，但是如果两个值有着不同的类型，则不能直接进行比较：

```Bash
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)          // "true"
fmt.Println(f >= 0)          // "true"
fmt.Println(c == f)          // compile error: type mismatch
fmt.Println(c == Celsius(f)) // "true"!
```

可以为类型定义一个string方法，当使用fmt包的打印方法时，将会优先使用该类型对应的String方法返回的结果打印：

```Bash
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }

c := FToC(212.0)
fmt.Println(c.String()) // "100°C"
fmt.Printf("%v\n", c)   // "100°C"; no need to call String explicitly
fmt.Printf("%s\n", c)   // "100°C"
fmt.Println(c)          // "100°C"
```

### 作用域

一个程序可能包含多个同名的声明，只要它们在不同的词法域就没有关系。例如可以声明一个局部变量，和包级的变量同名。当编译器遇到一个名字引用时，它会对其定义进行查找，查找过程从最内层的词法域向全局的作用域进行，如果该名字在内部和外部的块分别声明过，则内部块的声明首先被找到。在这种情况下，内部声明屏蔽了外部同名的声明，让外部的声明的名字无法被访问。

在函数中词法域可以深度嵌套，因此内部的一个声明可能屏蔽外部的声明，此时代码的可读性就会受到影响：

```Go
func main() {
    x := "hello!"
    for i := 0; i < len(x); i++ {
        x := x[i]
        if x != '!' {
            x := x + 'A' - 'a'
            fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
        }
    }
}
```

当出现同名的时候，很容易引发一个非常隐晦的bug。

并不是所有的词法域都显式地对应到由花括弧包含的语句，例如for语句创建了两个词法域：花括弧包含的是显式的部分，是for的循环体部分词法域，另外一个隐式的部分则是循环的初始化部分。下面的例子中，有三个不同的x变量，每个声明在不同的词法域：

```Go
func main() {
    x := "hello"
    for _, x := range x {
        x := x + 'A' - 'a'
        fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
    }
}
```

if和switch语句也会在条件部分创建隐式词法域，还有它们对应的执行体词法域，在if外是引用不到这些变量的：

```Go
if x := f(); x == 0 {
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```

在第二个if中，它嵌套在第一个if的内部，所以在第二个if中也可以访问到x

在包级别，声明的顺序并不会影响作用域范围，因此一个先声明的可以引用它自身或者是引用后面的一个声明，这可以让我们定义一些相互嵌套或递归的类型或函数。但是如果一个变量或常量递归引用了自身，则会产生编译错误。

## 常量

const代表声明常量，它通常在包一级范围，包级别声明的变量会在main入口函数执行前完成初始化，可以在整个包内被访问：

```Go
import "fmt"

const boilingF = 212.0
。。。
```

所有常量的运算都可以在编译期完成，因为它们的值是在编译期就确定的，因此常量可以是构成类型的一部分，例如用于指定数组类型的长度：

```Go
const IPv4Len = 4

// parseIPv4 parses an IPv4 address (d.d.d.d).
func parseIPv4(s string) IP {
    var p [IPv4Len]byte
    // ...
}
```

一个常量的声明也可以包含一个类型和一个值，但是如果没有显式指明类型，那么将从右边的表达式推断类型：

```Go
const noDelay time.Duration = 0
const timeout = 5 * time.Minute
fmt.Printf("%T %[1]v\n", noDelay)     // "time.Duration 0"
fmt.Printf("%T %[1]v\n", timeout)     // "time.Duration 5m0s"
fmt.Printf("%T %[1]v\n", time.Minute) // "time.Duration 1m0s"
```

批量声明常量的时候，初始化表达式是可以省略的，如果省略则代表使用前面常量的初始化表达式：

```Go
const (
    a = 1
    b
    c = 2
    d
)

fmt.Println(a, b, c, d) // "1 1 2 2"
```

常量声明可以使用iota常量生成器初始化，它可以用于简化初始化表达式。在一个const声明语句中，在第一个声明的常量所在的行，iota将会被置为0，然后在每一个有常量声明的行加一：

```Go
type Weekday int

const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)
```

周日将对应0，周一为1，如此等等。

下面是在复杂的常量表达式中使用iota：

```Go
const (
    _ = 1 << (10 * iota)
    KiB // 1024
    MiB // 1048576
    GiB // 1073741824
    TiB // 1099511627776             (exceeds 1 << 32)
    PiB // 1125899906842624
    EiB // 1152921504606846976
    ZiB // 1180591620717411303424    (exceeds 1 << 64)
    YiB // 1208925819614629174706176
)
```

Go中常量有个特别的地方：当它没有声明时指定类型时，它就是无类型的，这些无类型常量的精度更高。有六种未明确类型的常量类型，分别是无类型的布尔型、无类型的整数、无类型的字符、无类型的浮点数、无类型的复数、无类型的字符串。

当无类型常量用于表达式的时候，不需要显式的类型转换：

```Go
var x float32 = math.Pi
var y float64 = math.Pi
var z complex128 = math.Pi
```

## 判断

`if` 语句条件两边也不加括号，但是主体部分需要加：

```Go
if n > 1 {
    fmt.Printf("%d\t%s\n", n, line)
}
```

if里面也可以带一个简单的初始化语句，这样可以限制err的作用域：

```Go
if err := r.ParseForm(); err != nil {
    log.Print(err)
}
```

Go中的switch不需要显式地在每一个case后写break，语言默认执行完case后的逻辑语句会自动退出：

```Go
switch coinflip() {
case "heads":
    heads++
case "tails":
    tails++
default:
    fmt.Println("landed on edge!")
}
```

如果要相邻的几个case都执行同一逻辑的话，需要自己显式地写上一个fallthrough语句来覆盖这种默认行为。

Go语言里的switch还可以不带操作对象，不带操作对象时默认用true值代替，然后将每个case的表达式和true值进行比较：

```Go
func Signum(x int) int {
    switch {
    case x > 0:
        return +1
    default:
        return 0
    case x < 0:
        return -1
    }
}
```

## 循环

Go 语言只有 `for` 循环这一种循环语句。`for` 循环有多种形式，其中一种如下所示：

```Go
for initialization; condition; post {
    // zero or more statements
}
```

for、if、switch都可以紧跟一个简短的变量声明，一个自增表达式、赋值语句，或者一个函数调用。

for 循环的这三个部分每个都可以省略，如果省略 `initialization` 和 `post`，分号也可以省略：

```Go
// a traditional "while" loop
for condition {
    // ...
}
```

如果连 `condition` 也省略了，像下面这样：

```Go
// a traditional infinite loop
for {
    // ...
}
```

这个无限循环需要用break或者return才能停止

for range形式：

```Go
for _, arg := range os.Args[1:] {
    s += sep + arg
    sep = " "
}
```

下划线代表空标识符，表示丢弃不需要的数据

break和continue语句会改变控制流：

- break会中断当前的循环，并开始执行循环之后的内容
- ontinue会跳过当前循环，并开始执行下一次循环

这两个语句除了可以控制for循环，还可以用来控制switch和select语句。

continue会跳过内层的循环，如果我们想跳过的是更外层的循环的话，我们可以在相应的位置加上label，这样break和continue就可以根据我们的想法来continue和break任意循环，此时它们的作用类似goto语句。

Go中没有while循环，如果想实现的话需要结合无条件的for+break来实现。

Go语言提供goto语句，它可以无条件的转移到程序指定的行，语法是：

```Go
goto label
...
label: statement
```

# 数据类型

## 指针

Go语言提供了指针。指针是一种直接存储了变量的内存地址的数据类型。通过指针，我们可以直接读或更新对应变量的值，而不需要知道该变量的名字。

在其它语言中，比如C语言，指针操作是完全不受约束的。在另外一些语言中，指针一般被处理为“引用”，除了到处传递这些指针之外，并不能对这些指针做太多事情。

Go语言在这两种范围中取了一种平衡。指针是可见的内存地址，&操作符可以返回一个变量的内存地址，并且*操作符可以获取指针指向的变量内容，但是在Go语言里没有指针运算，也就是不能像c语言里可以对指针进行加或减操作。

如果用“var x int”声明语句声明一个x变量：

- &x表达式（取x变量的内存地址）将产生一个指向该整数变量的指针，指针对应的数据类型是`*int`，称为指向int类型的指针，如果指针名字为p，那么可以说“p指针指向变量x”，或者说“p指针保存了x变量的内存地址”。
- `*p`表达式对应p指针指向的变量的值，它可以出现在表达式的左边，代表更新指针所指向的变量的值

任何类型的指针的零值都是nil。

指针之间也是可以进行相等测试的，只有当它们指向同一个变量或全部是nil时才相等。

Go中有垃圾回收机制，但是如果在函数中返回变量的地址，此时函数调用完毕后变量依然不会被回收，因为指针依然引用这个变量：

```Bash
var p = f()

func f() *int {
    v := 1
    return &v
}
```

有了指针，在函数中也可以更新局部变量的值：

```Bash
func incr(p *int) int {
    *p++ // 非常重要：只是增加p指向的变量的值，并不改变p指针！！！
    return *p
}

v := 1
incr(&v)              // side effect: v is now 2
fmt.Println(incr(&v)) // "3" (and v is 3)
```

指针虽然提供了这样的便利，但是它也带来一些麻烦：那就是可能存在程序员不知道的地方，创建了指向某个变量的指针，它可能会修改变量的值。

## 整型

Go语言同时提供了有符号和无符号类型的整数运算：

- int8、int16、int32和int64四种截然不同大小的有符号整数类型，分别对应8、16、32、64bit大小的有符号整数，与此对应的是uint8、uint16、uint32和uint64四种无符号整数类型。
- 对应特定CPU平台机器字大小的有符号和无符号整数int和uint。这两种类型都有同样的大小，32或64bit，不同的编译器或者不同的硬件平台都可能产生不同的大小
- Unicode字符rune类型是和int32等价的类型，通常用于表示一个Unicode码点
- byte是uint8类型的等价类型，byte类型一般用于强调数值是一个原始的数据（可以直接当做字符类型使用，例如var c1 byte='a'）
- 无符号的整数类型uintptr，没有指定具体的bit大小但是足以容纳指针，只有在底层编程才需要

值域：

- 有符号整数采用2的补码形式表示，一个n-bit的有符号数的值域是从-2(n-1)到2(n-1)-1。
- 无符号整数的所有bit位都用于表示非负数，值域是0到2(n)-1

Go语言更推荐使用有符号数，因为有时可能会出现递减判断的场景，当有符号数的0减一之后变成负数可以退出循环，而无符号数的0减一则变成一个很大的数，可能与程序预期的不一致，所以无符号数往往只有在位运算或其它特殊的运算场景才会使用。

在Go语言中，%取模运算符的符号和被取模数的符号总是一致的，因此`-5%3`和`-5%-3`结果都是-2。

当出现计算结果溢出时，超出的高位的bit位部分将被丢弃。如果原始的数值是有符号类型，而且最左边的bit位是1的话，那么最终结果可能是负的：

```Go
var u uint8 = 255
fmt.Println(u, u+1, u*u) // "255 0 1"

var i int8 = 127
fmt.Println(i, i+1, i*i) // "127 -128 1"
```

几种位操作运算符：

```Go
&      位运算 AND
|      位运算 OR
^      位运算 XOR
&^     位清空（AND NOT）  z = x &^ y中如果y中对应位是1，则z对应位是0，否则z对应位=x对应位
<<     左移
>>     右移
```

0开头代表八进制，如0666；0x或者0X开头代表十六进制。

Go中的自增语句，只有i++和i--这种形式，不允许操作符放在变量前面

## 浮点数

Go语言提供了两种精度的浮点数，float32和float64。浮点数的范围极限值可以在math包找到。常量math.MaxFloat32表示float32能表示的最大数值。

在表示浮点数的时候，小数点前面或后面的数字都可以被省略：例如.707或1.

很小或很大的数最好用科学计数法书写，通过e或E来指定指数部分：

```Go
const Avogadro = 6.02214129e23  // 阿伏伽德罗常数
const Planck   = 6.62606957e-34 // 普朗克常数
```

Go中有几个特殊的值，代表无穷大和无效的结果：

```Go
var z float64
fmt.Println(z, -z, 1/z, -1/z, z/z) // "0 -0 +Inf -Inf NaN"
```

可以用math.NaN来表示一个非法的结果，需要注意的是NaN和任何数都是不相等的

## 复数

Go语言提供了两种精度的复数类型：complex64和complex128，分别对应float32和float64两种浮点数精度

内置的complex函数用于构建复数，内建的real和imag函数分别返回复数的实部和虚部：

```Go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

## 布尔型

一个布尔类型的值只有两种：true和false。

布尔值可以和&&（AND）和||（OR）操作符结合，并且有短路行为。

布尔值并不会隐式转换为数字值0或1，必须显式用代码逻辑转换。

## 字符串

### 基本使用

一个字符串是一个不可改变的字节序列。文本字符串通常被解释为采用UTF8编码的Unicode码点（rune）序列。

内置的len函数可以返回一个字符串中的字节数目（不是rune字符数目）

索引操作s[i]返回第i个字节的字节值，子字符串操作s[i:j]基于原始的s字符串的第i个字节开始到第j个字节（并不包含j本身）生成一个新字符串。

第i个字节并不一定是字符串的第i个字符，因为对于非ASCII字符的UTF8编码会要两个或多个字节。

string类型可以用加号连接起来，表示构造一个新字符串。

字符串可以用==和<进行比较；比较通过逐个字节比较完成的，因此比较的结果是字符串自然编码的顺序。

字符串的值是不可变的：一个字符串包含的字节序列永远不会被改变，因此尝试修改字符串内部数据的操作也是被禁止的：

```Go
s[0] = 'L' // compile error: cannot assign to s[0]
```

不变性意味着如果两个字符串共享相同的底层数据的话也是安全的，所以复制和剪切操作速度都非常快，它们都共享相同的底层数据，没有分配新的内存。

### 字符串面值和原生字符串

双引号表示的字符串面值中，可以用反斜杠插入各种转义符

原生字符串是用反引号来标注的，它没有转义操作，所有的内容都是字面的意思，程序中的原生字符串面值可能跨越多行。它一般用于编写正则表达式、HTML模版等包含反斜杠或者需要多行的场景

### 编码

可以通过十六进制或八进制转义在字符串面值中包含任意的字节，如\xhh（两个十六进制的数字）、\ooo（三个八进制的数字，但不能超过\377，对应一个字节的范围，十进制是255）。`\uhhhh`对应16bit的码点值，`\Uhhhhhhhh`对应32bit的码点值，其中h是一个十六进制数字。下列的字符串，它们都是相同的值：

```Go
"世界"
"\xe4\xb8\x96\xe7\x95\x8c"
"\u4e16\u754c"
"\U00004e16\U0000754c"
```

Go语言的源文件采用UTF8编码，UTF8编码使用1到4个字节来表示每个Unicode码点，ASCII部分字符只使用1个字节，常用字符部分使用2或3个字节表示。unicode包提供了诸多处理rune字符相关功能的函数（比如区分字母和数字，或者是字母的大写和小写转换等），unicode/utf8包则提供了用于rune字符序列的UTF8编码和解码的功能。

Unicode转义也可以使用在rune字符中。下面三个字符是等价的：

```Plaintext
'世' '\u4e16' '\U00004e16'
```

如果关心每个Unicode字符，而不是每个字节的话，则需要用utf8.RuneCountInString来获取字符数：

```Go
import "unicode/utf8"

s := "Hello, 世界"
fmt.Println(len(s))                    // "13"
fmt.Println(utf8.RuneCountInString(s)) // "9"
```

可以用解码器来遍历所有的字符，这里每次在循环中获取位置和字符长度：

```Go
for i := 0; i < len(s); {
    r, size := utf8.DecodeRuneInString(s[i:])
    fmt.Printf("%d\t%c\n", i, r)
    i += size
}
```

Go语言的range循环在处理字符串的时候，会自动隐式解码UTF8字符串，对于非ASCII，索引更新的步长将超过1个字节：

```Go
for i, r := range "Hello, 世界" {
    fmt.Printf("%d\t%q\t%d\n", i, r, r)
}
```

当解码字符串的时候，如果遇到一个错误的编码输入，则会返回一个特别的Unicode字符`\uFFFD`

Unicode字符rune类型是和int32等价的类型，通常用于表示一个Unicode码点。在程序中经常用rune序列来替代字符串，因为rune序列中每个rune大小是一样的。

字符串转rune序列：

```Go
// "program" in Japanese katakana
s := "プログラム"
fmt.Printf("% x\n", s) // "e3 83 97 e3 83 ad e3 82 b0 e3 83 a9 e3 83 a0"
r := []rune(s)
fmt.Printf("%x\n", r)  // "[30d7 30ed 30b0 30e9 30e0]"
```

rune序列转字符串，代表进行UTF-8编码：

```Go
fmt.Println(string(r)) // "プログラム"
```

### 字节与bytes.Buffer

字符串和字节slice之间可以相互转换：

```Go
s := "abc"
b := []byte(s)
s2 := string(b)
```

将字符串转为字节数组，其实是分配了一个新的字节数组用于保存字符串数据的拷贝，然后引用这个底层的字节数组。

因为字符串是只读的，因此逐步构建字符串会导致很多分配和复制。在这种情况下，使用bytes.Buffer类型将会更有效，一个Buffer开始是空的，但是随着string、byte或[]byte等类型数据的写入可以动态增长：

```Go
// intsToString is like fmt.Sprint(values) but adds commas.
func intsToString(values []int) string {
    var buf bytes.Buffer
    buf.WriteByte('[')
    for i, v := range values {
        if i > 0 {
            buf.WriteString(", ")
        }
        fmt.Fprintf(&buf, "%d", v)
    }
    buf.WriteByte(']')
    return buf.String()
}

func main() {
    fmt.Println(intsToString([]int{1, 2, 3})) // "[1, 2, 3]"
}
```

向bytes.Buffer添加任意字符的UTF8编码时，应该使用bytes.Buffer的WriteRune方法，如果仅仅是写入ASCII字符，则使用WriteByte就足够了。

### 转换

标准库中有四个包对字符串处理尤为重要：bytes、strings、strconv和unicode包：

- strings包提供了许多如字符串的查询、替换、比较、截断、拆分和合并等功能。
- bytes包也提供了很多类似功能的函数，但是针对和字符串有着相同结构的[]byte类型
- strconv包提供了布尔型、整型数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换。
- unicode包提供了IsDigit、IsLetter、IsUpper和IsLower等类似功能，它们都是遵循Unicode标准定义的字母、数字等分类规范

将一个整数转为字符串，一种方法是用fmt.Sprintf返回一个格式化的字符串；另一个方法是用strconv.Itoa(“整数到ASCII”)：

```Go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"
```

不同进制之间的转换：

```Go
fmt.Println(strconv.FormatInt(int64(x), 2)) // "1111011"
```

字符串转整数，可以使用strconv包的Atoi或ParseInt函数，还有用于解析无符号整数的ParseUint函数：

```Go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

ParseInt函数的第三个参数是用于指定整型数的大小；例如16表示int16，0则表示int。

## 数组

数组是一个由固定长度的特定类型元素组成的序列。因为数组的长度是固定的，因此在Go语言中很少直接使用数组。

默认情况下，数组的每个元素都被初始化为元素类型对应的零值：

```Go
var q [3]int = [3]int{1, 2, 3}
var r [3]int = [3]int{1, 2}
fmt.Println(r[2]) // "0"
```

如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算：

```Go
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```

数组的长度是数组类型的一个组成部分，因此[3]int和[4]int是两种不同的数组类型。数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。

初始化的时候可以直接初始化某个位置的元素，其余位置都使用默认初始化：

```Go
r := [...]int{99: -1}

type Currency int

const (
    USD Currency = iota // 美元
    EUR                 // 欧元
    GBP                 // 英镑
    RMB                 // 人民币
)

symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥"}

fmt.Println(RMB, symbol[RMB]) // "3 ￥"
```

数组是可以用等号比较的，两个数组相等的条件：同时满足数组长度相同，以及内部所有元素都相等

当调用一个函数的时候，函数的每个调用参数将会被赋值给函数内部的参数变量，此时就会有一个复制动作，所以用一个大的数组类型当做函数入参效率很低，而且在函数内部修改数组，也不会影响到原始数组，所以一般使用的时候都会传入一个数组指针作为函数入参，下面两个函数就在内部修改了数组的值：

```Go
func zero(ptr *[32]byte) {
    for i := range ptr {
        ptr[i] = 0
    }
}

func zero(ptr *[32]byte) {
    *ptr = [32]byte{}
}
```

## Slice

Slice（切片）代表变长的序列，一般写作[]T，其中T代表slice中元素的类型，slice的语法和数组很像，只是没有固定长度而已。

Slice的底层引用了一个数组对象，它由三部分组成：

- 指针：指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素
- 长度：slice中元素的数目。len函数会返回slice的长度
- 容量：从slice的开始位置到底层数据的结尾位置。cap函数会返回slice的容量

多个slice之间可以共享底层的数据，并且引用的数组部分区间可能重叠。对数组切分会产生一个slice：

```Go
Q2 := months[4:7]
summer := months[6:9]
fmt.Println(Q2)     // ["April" "May" "June"]
fmt.Println(summer) // ["June" "July" "August"]
```

将数组直接转换为slice，底层元素不变，可以直接使用months[:]：

```Go
a := [...]int{0, 1, 2, 3, 4, 5}
reverse(a[:])
fmt.Println(a) // "[5 4 3 2 1 0]"
```

字符串也可以进行类似的切片操作，返回一个字节序列。string的底层就是byte数组。

因为slice值包含指向第一个slice元素的指针，因此向函数传递slice将允许在函数内部修改底层数组的元素。下面的代码是将slice整体向左旋转n个元素，因为这些切片都共享同样的底层数据，所以可以这样操作：

```Go
s := []int{0, 1, 2, 3, 4, 5} // 定义切片时指定值
// Rotate s left by two positions.
reverse(s[:2])
reverse(s[2:])
reverse(s)
fmt.Println(s) // "[2 3 4 5 0 1]"
```

slice在声明的时候会隐式创建一个数组，然后slice的指针指向底层的数组。和数组类似，slice也可以按照顺序初始化序列，或者通过索引和元素指定。

slice之间不能通过等号比较，但是可以使用bytes.Equal函数来判断两个字节型slice是否相等。之所以slice没有提供判断相等的功能，是因为：

- slice可能包含自身，这种情况会很复杂，例如当slice声明为[]interface{}时，slice的元素可以是自身
- slice随时可能会被修改

一个nil值的slice的长度和容量都是0，但是也有非nil值的slice的长度和容量也是0的，例如[]int{}或make([]int, 3)[3:]。下面的几种初始化中，只有最后一种情况slice不为nil：

```Go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
```

当需要测试一个slice是否为空时，使用len(s) == 0来判断，而不应该用s == nil来判断。在官方接口的大部分地方，都是同等对待nil值的slice和0长度的slice的。

make函数也可以用来创建slice，若容量被忽略，则容量就等于长度：

```Go
make([]T, len)
make([]T, len, cap) // same as make([]T, cap)[:len]
```

当可以提前知道切片的最终大小时，使用上面第二种的分配方式会更好。

append函数用于向slice追加元素，它的逻辑是会先检查底层数组的大小是否足够添加新元素，如果足够则直接赋值，否则就申请一个新的数组然后赋值，并返回新的序列。每次调用append函数时，内部是否申请了新的内存是未知的，因此通常是将append返回的结果直接赋值给输入的slice变量：

```Go
runes = append(runes, r)
```

append函数可以用来追加多个元素，甚至是追加一个slice：

```Go
var x []int
x = append(x, 1)
x = append(x, 2, 3)
x = append(x, 4, 5, 6)
x = append(x, x...) // append the slice x
fmt.Println(x)      // "[1 2 3 4 5 6 1 2 3 4 5 6]"
```

内置的copy函数可以方便地将一个slice复制另一个相同类型的slice。

## Map

**map** 存储了键/值（key/value）的集合，对集合元素，提供常数时间的存、取或测试操作。键可以是任意类型，只要其值能用 `==` 运算符比较。

内置的make函数可以创建一个map：

```Go
ages := make(map[string]int) // mapping from strings to ints
```

另一种创建空的map的表达式是`map[string]int{}`

使用map字面值来创建map，可以指定一些初始值：

```Go
ages := map[string]int{
    "alice":   31,
    "charlie": 34,
}
```

使用内置的delete函数可以删除元素（没有删除全部的方法，可以直接make一个新的）：

```Go
delete(ages, "alice") // remove element ages["alice"]
```

无论是访问还是删除，即使元素不在map里面也不会报错。查找失败时将返回对应类型的零值。

禁止对map里面的元素取地址，因为map可能随着元素数量的增长而重新分配更大的内存空间，从而可能导致之前的地址无效：

```Go
_ = &ages["bob"] // compile error: cannot take address of map element
```

它是用hash实现的，当用for range遍历Map的时候，会发现顺序是随机的，而且每次运行都会变化，这就强制程序员在编程的时候，不依赖特定的遍历顺序。如果要按顺序遍历key/value对，我们必须显式地对key进行排序，可以使用sort包的Strings函数对字符串slice进行排序，再去遍历map：

```Go
import "sort"

var names []string
for name := range ages {
    names = append(names, name)
}
sort.Strings(names)
for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}
```

map上的大部分操作，包括查找、删除、len和range循环都可以安全工作在nil值的map上，它们的行为和一个空的map类似。但是向一个nil值的map存入元素将导致一个panic异常，在向map存数据前必须先创建map。

如果想要检查一个元素是否真的在map里面，应该这样写：

```Go
if age, ok := ages["bob"]; !ok { /* ... */ }
```

map的key必须是可比较的类型，所以map或者slice并不能作为key，但可以使用间接的方式，第一步先把slice转为string，第二步是用string作为key。下面这段代码就是统计相同的字符串列表的次数：

```Go
var m = make(map[string]int)

func k(list []string) string { return fmt.Sprintf("%q", list) }

func Add(list []string)       { m[k(list)]++ }
func Count(list []string) int { return m[k(list)] }
```

## 结构体

结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。每个值称为结构体的成员。下面就是一个结构体的定义：

```Go
type Employee struct {
    ID        int
    Name      string
    Address   string
    DoB       time.Time
    Position  string
    Salary    int
    ManagerID int
}

var dilbert Employee
```

结构体变量的成员可以通过点操作符访问，可以直接对每个成员赋值或者取地址。

指向结构体的指针也可以使用点操作符：

```Go
var employeeOfTheMonth *Employee = &dilbert
employeeOfTheMonth.Position += " (proactive team player)"

相当于
(*employeeOfTheMonth).Position += " (proactive team player)"
```

相邻的成员类型如果相同的话可以被合并到一行，就像下面的Name和Address成员：

```Go
type Employee struct {
    ID            int
    Name, Address string
    DoB           time.Time
    Position      string
    Salary        int
    ManagerID     int
}
```

一个命名为S的结构体类型将不能再包含S类型的成员：因为一个聚合的值不能包含它自身。（该限制同样适用于数组。）但是S类型的结构体可以包含`*S`指针类型的成员，这可以让我们创建递归的数据结构。

结构体类型的零值是每个成员都是零值。

可以用结构体成员定义顺序赋予初始值：

```Go
type Point struct{ X, Y int }

p := Point{1, 2}
```

它的可维护性较差，一般只在定义结构体的包内部使用，或者是在较小的结构体中使用。

更常用的写法是用成员名字和对应的值来初始化：

```Go
anim := gif.GIF{LoopCount: nframes}
```

如果成员被忽略的话将默认用零值。

结构体可以作为函数的参数和返回值，考虑到效率，较大的结构体通常会用指针的方式传入和返回。如果要在函数内部修改结构体成员的话，则必须用指针传入。

如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，可以使用等号来比较。可比较的结构体可以用于map的key类型。

一个结构体中可以嵌入另一个结构体：

```Go
type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius int
}

type Wheel struct {
    Circle Circle
    Spokes int
}
```

但是如果这样定义的话，那么初始化Wheel变量就会变得很麻烦。Go提供了一种特性：匿名成员，匿名成员可以是某个类型，也可以是指针，这种情况下不需要指定成员的名字：

```Go
type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
```

因为匿名成员的特性，可以直接访问叶子属性，而不需要给出路径：

```Go
var w Wheel
w.X = 8            // equivalent to w.Circle.Point.X = 8
w.Y = 8            // equivalent to w.Circle.Point.Y = 8
w.Radius = 5       // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```

在包内部，即使Point和Circle匿名成员都是不导出的，也可以用简短的方式访问它们，但是在包外部，就不能用这种方式访问了。

匿名成员Circle和Point都有自己的名字——就是命名的类型名字，用原始的方式访问变量也是可以的。

但是在初始化的时候，必须体现出完整的路径才行：

```Go
w = Wheel{Circle{Point{8, 8}, 5}, 20}

w = Wheel{
    Circle: Circle{
        Point:  Point{X: 8, Y: 8},
        Radius: 5,
    },
    Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
}

fmt.Printf("%#v\n", w)
// Output:
// Wheel{Circle:Circle{Point:Point{X:8, Y:8}, Radius:5}, Spokes:20}

w.X = 42

fmt.Printf("%#v\n", w)
// Output:
// Wheel{Circle:Circle{Point:Point{X:42, Y:8}, Radius:5}, Spokes:20}
```

因为匿名成员也有一个隐式的名字，因此不能同时包含两个类型相同的匿名成员，这会导致名字冲突。

实际上，外层的结构体不仅仅是获得了匿名成员类型的所有成员，而且也获得了该类型导出的全部的方法，这涉及到Go面向对象编程的核心：组合。

结构体中所有字段在内存中是连续的。

结构体之间也可以进行类型转换，但是字段的名字、个数、类型要完全一致。

## JSON

将结构体转为一个json字节slice：

```Go
data, err := json.Marshal(movies)
if err != nil {
    log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
```

为了生成便于阅读的格式，另一个json.MarshalIndent函数将产生整齐缩进的输出。该函数有两个额外的字符串参数用于表示每一行输出的前缀和每一个层级的缩进：

```Go
data, err := json.MarshalIndent(movies, "", "    ")
if err != nil {
    log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
```

在编码时，通过reflect反射技术，默认使用结构体的成员名来作为json对象名。只有导出的结构体成员才会被编码，所以要使用大写字母开头的成员名称。

定义结构体的时候，要附带Tag信息：

```Go
type Movie struct {
    Title  string
    Year   int  `json:"released"`
    Color  bool `json:"color,omitempty"`
    Actors []string
}
```

Tag是一个字符串面值，由一系列键值对组成，因为会带有双引号，所以一般用反引号来表示。json开头键名对应的值用于控制encoding/json包的编码和解码的行为，并且encoding/...下面其它的包也遵循这个约定。

成员Tag中json对应值的第一部分用于指定JSON对象的名字，omitempty选项表示当Go语言结构体成员为空或零值时不生成该JSON对象。

解码时可以通过定义结构体来只解码json中感兴趣的数据：

```Go
var titles []struct{ Title string }
if err := json.Unmarshal(data, &titles); err != nil {
    log.Fatalf("JSON unmarshaling failed: %s", err)
}
fmt.Println(titles) // "[{Casablanca} {Cool Hand Luke} {Bullitt}]"
```

也可以直接对流来解码json数据：

```Go
if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
    resp.Body.Close()
    return nil, err
}
```

# 函数和方法

## 函数基本用法

函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体：

```Go
func name(parameter-list) (result-list) {
    body
}
```

在调用的时候，实参会赋值给形参列表，作为局部变量使用。对形参进行修改不会影响实参。但是，如果实参包括引用类型，如指针，slice(切片)、map、function、channel等类型，实参可能会由于函数的间接引用被修改。

返回值也可以像形式参数一样被命名。在这种情况下，每个返回值被声明成一个局部变量，并根据该返回值的类型，将其初始化为该类型的零值。

如果一个函数在声明时，包含返回值列表，该函数必须以 return语句结尾，除非函数明显无法运行到结尾处。例如函数在结尾时调用了panic异常或函数中存在无限循环。

如果一组形参或返回值有相同的类型，那就不用给每个形参都指定类型：

```Go
func hypot(x, y float64) float64 {
    return math.Sqrt(x*x + y*y)
}
fmt.Println(hypot(3,4)) // "5"
```

函数的类型被称为函数的签名。如果两个函数形式参数列表和返回值列表中的变量类型一一对应，那么这两个函数被认为有相同的类型或签名。Go函数不支持重载。

偶尔会遇到没有函数体的函数声明，这表示该函数不是以Go实现的，这里只定义了函数签名：

```Go
package math

func Sin(x float64) float //implemented in assembly language
```

Go使用可变栈，不同于其他编程语言，它不会限制递归的深度，当深度增加的时候，栈的大小也逐渐按需增加。

在Go中，一个函数可以返回多个值。如果一个函数所有的返回值都有显式的变量名，那么该函数的return语句后面不用再跟其他内容了，这就是bare return。当一个函数有多处return语句以及许多返回值时，bare return 可以减少代码的重复，但是使得代码难以被理解，因为需要考虑默认初始化以及初始化的时机。

函数和其他值一样，拥有类型，可以赋值给变量。函数类型的零值是nil。

函数可以和nil值比较，但是函数值之间是不可比较的，也不能用函数值作为map的key。

可变参数函数代表可以接受0-n个参数：

```Go
func sum(vals ...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}

values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"
```

在函数内部，可变参数的行为和slice类似，但是它们的类型是不同的。

## 错误处理策略

panic是来自被调用函数的信号，表示发生了某个已知的bug。一个良好的程序永远不应该发生panic异常。

Go中有的函数永远不会发生错误（除了内存溢出等极端情况），如Contains；有的函数则保证入参合法，就不会发生错误，如time.Date函数；而大部分函数，都是有可能出错的，此时一般会把错误当做返回列表的最后一个值来返回。如果导致失败的原因只有一个，额外的返回值可以是一个布尔值，通常被命名为ok。

内置的error是接口类型，当函数返回非空的error时，其他返回值应该被忽略。有少部分函数在发生错误时，仍然会返回一些有用的返回值。比如，当读取文件发生错误时，Read函数会返回可以读取的字节数以及错误信息。对于这种情况，正确的处理方式应该是先处理这些不完整的数据，再处理错误。

Go设计的时候，把那些预期的错误当做返回值来处理，这样能让程序员更集中精力处理这些错误。

一些常用的错误处理策略：

- 传播错误：函数中某个子程序的失败，会变成该函数的失败：
  - ```Go
    resp, err := http.Get(url)
    if err != nil{
        return nil, err
    }
    ```

在向上层返回错误的时候，也可以组装一些重要的信息，并新建一个错误，这种方式可以让上层程序看到一系列的报错，便于定位问题：

```Go
doc, err := html.Parse(resp.Body)
resp.Body.Close()
if err != nil {
    return nil, fmt.Errorf("parsing %s as HTML: %v", url,err)
}
```

- 重试：限制重试的时间间隔或者重试次数
  - ```Go
    // WaitForServer attempts to contact the server of a URL.
    // It tries for one minute using exponential back-off.
    // It reports an error if all attempts fail.
    func WaitForServer(url string) error {
        const timeout = 1 * time.Minute
        deadline := time.Now().Add(timeout)
        for tries := 0; time.Now().Before(deadline); tries++ {
            _, err := http.Head(url)
            if err == nil {
                return nil // success
            }
            log.Printf("server not responding (%s);retrying…", err)
            time.Sleep(time.Second << uint(tries)) // exponential back-off
        }
        return fmt.Errorf("server %s failed to respond after %s", url, timeout)
    }
    ```
- 输出错误信息并结束程序：这种策略只应在main中执行。对库函数而言，应仅向上传播错误。
  - ```Go
    // (In function main.)
    if err := WaitForServer(url); err != nil {
        fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
        os.Exit(1)
    }
    ```

  -  或者使用log.Fatalf来替代，它可以先输出时间，再输出指定的值。
- 只输出错误信息，不需要中断程序的运行
- 忽略错误

遇到特定的错误后，可以根据错误类型决定程序的走向，如EOF错误：

```Go
in := bufio.NewReader(os.Stdin)
for {
    r, _, err := in.ReadRune()
    if err == io.EOF {
        break // finished reading
    }
    if err != nil {
        return fmt.Errorf("read failed:%v", err)
    }
    // ...use r…
}
```

error其实就是一个接口，它只有一个返回错误信息的方法：

```Go
type error interface {
    Error() string
}
```

创建一个error最简单的方法就是调用errors.New函数，它会根据传入的错误信息返回一个新的error：

```Go
package errors

func New(text string) error { return &errorString{text} }

type errorString struct { text string }

func (e *errorString) Error() string { return e.text }
```

因为它的底层是一个结构体，所以即使多次调用errors.New传入一个相同的值，也不会相等。

创建error还可以使用fmt的fmt.Errorf，它还会处理字符串格式化：

```Go
package fmt

import "errors"

func Errorf(format string, args ...interface{}) error {
    return errors.New(Sprintf(format, args...))
}
```

## 匿名函数和闭包

匿名函数允许在使用函数的时候。拥有函数名的函数只能在包级语法块中被声明，而匿名函数则可以在其他地方声明，当匿名函数用到外层的局部变量时，需要注意闭包的特性：

```Go
// squares返回一个匿名函数。
// 该匿名函数每次被调用时都会返回下一个数的平方。
func squares() func() int {
    var x int
    return func() int {
        x++
        return x * x
    }
}
func main() {
    f := squares()
    fmt.Println(f()) // "1"
    fmt.Println(f()) // "4"
    fmt.Println(f()) // "9"
    fmt.Println(f()) // "16"
}
```

在反复调用f函数的时候，它引用的局部变量也在不断更新，匿名函数不仅仅是一段代码逻辑，还附带了它引用的变量状态，此时变量的生命周期不由它的作用域决定，squares返回后，变量x仍然隐式的存在于f中。

闭包是简化的对象，对象是天然的闭包。闭包让无状态的函数有了状态，函数有了状态和行为，和对象一样，成为了完整的状态机。例如react的函数组件也得以实现，函数式编程也才能图灵完备。

如果想在匿名函数里面递归调用自己，那就必须把函数赋值给一个变量，例如下面的例子中，把函数赋值给了visitAll变量；在函数调用中不仅可以更新外部的局部变量，而且更新后的局部变量还可以在函数调用后继续使用，如下面的order变量：

```Go
func main() {
    for i, course := range topoSort(prereqs) {
        fmt.Printf("%d:\t%s\n", i+1, course)
    }
}

func topoSort(m map[string][]string) []string {
    var order []string
    seen := make(map[string]bool)
    var visitAll func(items []string)
    visitAll = func(items []string) {
        for _, item := range items {
            if !seen[item] {
                seen[item] = true
                visitAll(m[item])
                order = append(order, item)
            }
        }
    }
    var keys []string
    for key := range m {
        keys = append(keys, key)
    }
    sort.Strings(keys)
    visitAll(keys)
    return order
}
```

在使用迭代变量的时候要特别注意，下面一段程序是先创建一些目录，然后再删除它们：

```Go
var rmdirs []func()
for _, d := range tempDirs() {
    dir := d // NOTE: necessary!
    os.MkdirAll(dir, 0755) // creates parent directories too
    rmdirs = append(rmdirs, func() {
        os.RemoveAll(dir)
    })
}
// ...do some work…
for _, rmdir := range rmdirs {
    rmdir() // clean up
}
```

在循环体中，把循环变量d赋值给新的局部变量dir，这个操作是必须的，如果去掉这行的话程序会发生错误。因为循环变量d是一个确定的内存地址，它只是在循环中被赋值给一次又一次迭代的值，当删除操作执行时，for循环已完成，dir中存储的值等于最后一次迭代的值。这意味着，每次对os.RemoveAll的调用删除的都是相同的目录。

不仅循环中有这个问题，go语句、defer语句都会有类似的问题，因为go和defer后面的内容也是延迟执行的，程序无法预料延迟执行时之前那个地址现在保存的是什么。

## Deferred函数

有的函数经常需要关闭流，那就需要在所有执行路径下都执行同一段代码，逻辑越来越复杂。

一个简单的方式是使用defer，当包含该defer语句的函数执行完毕时，defer后的函数才会被执行，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。如果是在同一个函数中有多条defer语句，那么它们的执行顺序和声明顺序相反。

defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。

释放资源的defer应该直接跟在请求资源的语句后：

```Go
func title(url string) error {
    resp, err := http.Get(url)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    ct := resp.Header.Get("Content-Type")
    if ct != "text/html" && !strings.HasPrefix(ct,"text/html;") {
        return fmt.Errorf("%s has type %s, not text/html",url, ct)
    }
    doc, err := html.Parse(resp.Body)
    if err != nil {
        return fmt.Errorf("parsing %s as HTML: %v", url,err)
    }
    // ...print doc's title element…
    return nil
}
```

除了关闭资源，还有释放锁：

```Go
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}
```

defer后的函数会在return后执行，除了函数执行以外的其他部分，如参数表达式，或者返回函数的方法，这些都是在执行到defer时候就执行的。利用这一点可以记录一个函数的进入、退出、执行时间：

```Go
func bigSlowOperation() {
    defer trace("bigSlowOperation")() // don't forget the extra parentheses
    // ...lots of work…
    time.Sleep(10 * time.Second) // simulate slow operation by sleeping
}
func trace(msg string) func() {
    start := time.Now()
    log.Printf("enter %s", msg)
    return func() { 
        log.Printf("exit %s (%s)", msg,time.Since(start)) 
    }
}
```

利用匿名函数+defer，记录调用函数的入参和返回值：

```Go
func double(x int) (result int) {
    defer func() { fmt.Printf("double(%d) = %d\n", x,result) }()
    return x + x
}
_ = double(4)
// Output:
// "double(4) = 8"
```

被延迟执行的匿名函数甚至可以修改函数返回给调用者的返回值：

```Go
func triple(x int) (result int) {
    defer func() { result += x }()
    return double(x)
}
fmt.Println(triple(4)) // "12"
```

一般不要把defer放在循环体中，因为只有在函数执行完毕后，这些被延迟的函数才会执行，下面的代码可能有问题，因为在系统的文件描述符耗尽之前，没有文件会被关闭：

```Go
for _, filename := range filenames {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close() // NOTE: risky; could run out of file descriptors
    // ...process f…
}
```

更好的解决办法是单独抽出一个函数，处理每一个循环中的变量，这样就能保证defer的执行：

```Go
for _, filename := range filenames {
    if err := doFile(filename); err != nil {
        return err
    }
}
func doFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()
    // ...process f…
}
```

## panic异常与恢复

当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被延迟的函数（defer 机制），随后程序崩溃并输出日志信息。日志信息包括panic value和函数调用的堆栈跟踪信息。

```Go
panic: runtime error: integer divide by zero
main.f(0)
src/gopl.io/ch5/defer1/defer.go:14
main.f(1)
src/gopl.io/ch5/defer1/defer.go:16
main.f(2)
src/gopl.io/ch5/defer1/defer.go:16
main.f(3)
src/gopl.io/ch5/defer1/defer.go:16
main.main()
src/gopl.io/ch5/defer1/defer.go:10
```

直接调用内置的panic函数也会引发panic异常：

```Go
switch s := suit(drawCard()); s {
case "Spades":                                // ...
case "Hearts":                                // ...
case "Diamonds":                              // ...
case "Clubs":                                 // ...
default:
    panic(fmt.Sprintf("invalid suit %q", s)) // Joker?
}
```

对于严重错误才会出现panic，可以预料的错误都应该使用Go的错误机制处理。

有的函数虽然可能会导致错误，但是只要入参合法就一定不会返回error，如regexp.Compile，所以大多数情况下不用特别判断它返回的error。有的函数只要内部出现错误，就会引起panic，如regexp.MustCompile：

```Go
func MustCompile(expr string) *Regexp {
    re, err := Compile(expr)
    if err != nil {
        panic(err)
    }
    return re
}
```

这样做的好处是返回值变成1个了，这样就可以方便地给包级别的变量赋值：

```Go
var httpSchemeRE = regexp.MustCompile(`^https?:`) //"http:" or "https:"
```

函数名带Must前缀的函数都有类似的规则。

为了方便诊断问题，可以直接使用runtime包来输出堆栈信息（延迟函数的调用在释放堆栈信息之前）：

```Go
func main() {
    defer printStack()
    f(3)
}
func printStack() {
    var buf [4096]byte
    n := runtime.Stack(buf[:], false)
    os.Stdout.Write(buf[:n])
}

goroutine 1 [running]:
main.printStack()
src/gopl.io/ch5/defer2/defer.go:20
main.f(0)
src/gopl.io/ch5/defer2/defer.go:27
main.f(1)
src/gopl.io/ch5/defer2/defer.go:29
main.f(2)
src/gopl.io/ch5/defer2/defer.go:29
main.f(3)
src/gopl.io/ch5/defer2/defer.go:29
main.main()
src/gopl.io/ch5/defer2/defer.go:15
```

有时想让程序从panic中恢复运行，此时可以在deferred函数中使用内置函数recover，如果定义该defer的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil：

```Go
func Parse(input string) (s *Syntax, err error) {
    defer func() {
        if p := recover(); p != nil {
            err = fmt.Errorf("internal error: %v", p)
        }
    }()
    // ...parser...
}
```

恢复panic异常是有风险的，因为包级变量可能处于中间态、文件或者网络连接没有被关闭、获得的锁没有被释放等问题。

公有的API不应该直接返回panic，而是以error的形式返回。主程序也不要恢复其他程序的panic，这样同样是不安全的。

一个比较好的方式是在恢复时判断panic value的类型，如果它是某个特殊类型就尝试恢复，否则就直接终止程序：

```Go
// soleTitle returns the text of the first non-empty title element
// in doc, and an error if there was not exactly one.
func soleTitle(doc *html.Node) (title string, err error) {
    type bailout struct{}
    defer func() {
        switch p := recover(); p {
        case nil:       // no panic
        case bailout{}: // "expected" panic
            err = fmt.Errorf("multiple title elements")
        default:
            panic(p) // unexpected panic; carry on panicking
        }
    }()
    // Bail out of recursion if we find more than one nonempty title.
    forEachNode(doc, func(n *html.Node) {
        if n.Type == html.ElementNode && n.Data == "title" &&
            n.FirstChild != nil {
            if title != "" {
                panic(bailout{}) // multiple titleelements
            }
            title = n.FirstChild.Data
        }
    }, nil)
    if title == "" {
        return "", fmt.Errorf("no title element")
    }
    return title, nil
}
```

在上例中，deferred函数调用recover，并检查panic value：

- 当panic value是bailout{}类型时，deferred函数生成一个error返回给调用者。
- 当panic value是其他non-nil值时，表示发生了未知的panic异常，deferred函数将调用panic函数并将当前的panic value作为参数传入；此时，等同于recover没有做任何操作。

有些情况下的panic是无法恢复的，如内存不足。

## 时间与日期

一般会用到time包，time.Now()会返回一个time.Time类型，表示当前时间。该类型的Year、Month等方法可以直接返回年月日等信息。

格式化时间日期一般有两种方法：

第一：使用Printf或者SPringf

```Go
fmt.Printf("当前年月日 %d-%d-%d %d:%d:%d\n", now.Year
now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())

datestr := fmt.sprintf("当前年月日 %d-%d %d:%d:%d:%d\n", now.Year()
now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())

fmt.Printf("dateStr=%v\n", dateStr)
```

第二，使用time.Format完成：

```Go
fmt.Printf(now.Format"2006-01-02 15:04:05"))
fmt.Println()
fmt.Printf((now.Format("2006-01-02"))
fmt.Println()
fmt.Printf(now.Format("15:04:05"))
fmt.Println()
```

一些时间常量：

```Go
const(
    Nanosecond Duration =1    //纳秒
    Microsecond = 1000 * Nanosecond    //微秒
    Millisecond = 1000 * Microsecond    //毫秒
    Second = 1000 * Millisecond    //秒
    Minute = 60*Second     //分钟
    Hour = 60*Minute    //小时
)
```

睡眠：time.Sleep方法

time的unix方法会返回从1970年到现在经过的时间单位秒

## 方法基本用法

在函数声明的时候，在名字前面放上一个变量，就是一个方法：

```Go
// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

上面的代码里那个附加的参数p，叫做方法的接收器（receiver），早期面向对象将调用一个方法称为向一个对象发送消息。在Go中可以任意选择接收器的名字，一般是类型的第一个字母。

无论是方法还是字段都使用点来调用，所以如果两个同名的话，会有歧义。

在Go语言中，可以为一些简单的数值、字符串、slice、map来直接定义方法，就像下面例子中的Path，此时Path底层是一个slice，但是还是可以为它定义方法，但是如果底层是指针或者interface则不能：

```Go
// A Path is a journey connecting the points with straight lines.
type Path []Point
// Distance returns the distance traveled along the path.
func (path Path) Distance() float64 {
    sum := 0.0
    for i := range path {
        if i > 0 {
            sum += path[i-1].Distance(path[i])
        }
    }
    return sum
}
```

调用方法除了符合面向对象思想以外，还有一个好处是调用函数的时候必须带上冗长的包名，而调用方法的时候则不需要，只需要带上变量名。

方法的接收器通常是指针类型的，因为如果是非指针类型，每次调用方法都会产生一次拷贝，当对象比较大的时候会产生性能开销，而且拷贝可能会带来一些意外错误，因为内部的各字段底层还是共用一些数据。

不管method的receiver是指针类型还是非指针类型，都是可以通过指针/非指针类型进行调用的，编译器会自动做类型转换

可以把nil当做接收器，此时要在方法中对接收器为nil的情况进行处理：

```Go
// An IntList is a linked list of integers.
// A nil *IntList represents the empty list.
type IntList struct {
    Value int
    Tail  *IntList
}
// Sum returns the sum of the list elements.
func (list *IntList) Sum() int {
    if list == nil {
        return 0
    }
    return list.Value + list.Tail.Sum()
}
```

方法执行可以拆分成两步，第一步是将方法绑定到一个变量，第二步是直接调用那个变量：

```Go
p := Point{1, 2}
q := Point{4, 6}

distanceFromP := p.Distance        // method value
fmt.Println(distanceFromP(q))      // "5"
var origin Point                   // {0, 0}
fmt.Println(distanceFromP(origin)) // "2.23606797749979", sqrt(5)

scaleP := p.ScaleBy // method value
scaleP(2)           // p becomes (2, 4)
scaleP(3)           //      then (6, 12)
scaleP(10)          //      then (60, 120)
```

这样生成的函数第一个参数就是接收器，剩余参数就是原方法的入参。

函数和方法有一个重要的区别：

- 方法的接收器类型是指针或者值类型都行，调用的时候也都可以混用
- 函数的入参是指针类型还是值类型必须严格符合，否则编译就会报错

## 嵌套结构体

当结构体中有匿名成员时，结构体不仅可以直接访问叶子属性，还能直接调用叶子方法：

```Go
import "image/color"

type Point struct{ X, Y float64 }

type ColoredPoint struct {
    Point
    Color color.RGBA
}


red := color.RGBA{255, 0, 0, 255}
blue := color.RGBA{0, 0, 255, 255}
var p = ColoredPoint{Point{1, 1}, red}
var q = ColoredPoint{Point{5, 4}, blue}
fmt.Println(p.Distance(q.Point)) // "5"
p.ScaleBy(2)
q.ScaleBy(2)
fmt.Println(p.Distance(q.Point)) // "10"
```

这时Point类的方法被引入了ColoredPoint。此时的ColoredPoint和Point之间并不是父类和子类的关系，只是组合起来了而已。

一个struct可以拥有多个匿名字段，当调用一个方法的时候，编译器会首先在本类型里面找方法，然后再去内嵌字段对应的类型里面找，如果有冲突的话编译则会报错。

## 面向对象

Go支持面向对象编程，但是它不是纯粹的面向对象语言，它没有类，没有继承、重载、构造函数等。

Go语言只有一种控制可见性的手段：大写首字母的标识符会从定义它们的包中被导出，小写字母的则不会。如果想封装一个对象，可以将其放入struct中。

一个struct类型的字段对同一个包的所有代码都有可见性。

封装的好处：调用方只需要关注接口，不需要关注实现、隐藏细节、阻止外部随意修改内部状态。

在Go中继承是用匿名结构体来实现的。当使用匿名结构体时就是继承，如果是非匿名结构体则是组合。

在Go中多态则和其他语言类似，也有接口的概念。

# 接口

Go语言中接口类型的独特之处在于它是满足隐式实现的。这种设计的好处是：创建新的接口之后，并不需要修改旧代码的定义，就能建立实现关系。

## 接口基本用法

一个常见的接口是io.Writer类型：

```Go
package io

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    // Write writes len(p) bytes from p to the underlying data stream.
    // It returns the number of bytes written from p (0 <= n <= len(p))
    // and any error encountered that caused the write to stop early.
    // Write must return a non-nil error if it returns n < len(p).
    // Write must not modify the slice data, even temporarily.
    //
    // Implementations must not retain p.
    Write(p []byte) (n int, err error)
}
```

只要有新的类型实现了Write函数，就可以传入任何以io.Writer为参数的函数中。在Go中实现接口非常简洁，一个类型可以自由地被另一个满足相同接口的类型替换，非常简洁的实现了里氏替换。

io包中常用的类型还有Reader和Closer：

```Go
package io
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Closer interface {
    Close() error
}
```

有些接口类型通过组合已有的接口来定义：

```Go
type ReadWriter interface {
    Reader
    Writer
}
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

给一个类型定义String方法，可以让它满足最广泛使用之一的接口类型fmt.Stringer：

```Go
package fmt

// The String method is used to print values passed
// as an operand to any format that accepts a string
// or to an unformatted printer such as Print.
type Stringer interface {
    String() string
}
```

interface{}被称为空接口类型，可以将任意一个值赋给空接口类型。

一个具体的类型可能实现了很多不相关的接口。

一个接口内部其实包含了两个部分：类型和值。一个接口的零值就是nil。当把一个具体的变量赋值给接口时，这个时候实际上就是将接口的类型更新为对应的类型，值指向具体的变量。

接口值是可以用来比较的，不仅可以和nil比较，当它们的动态类型相同并且动态值也相同（动态值是可比较的，且相等），此时就说接口值是相等的。如果如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（比如切片），将它们进行比较就会失败并且panic。

使用fmt包的%T动作可以获取到接口的动态类型：

```Go
var w io.Writer
fmt.Printf("%T\n", w) // "<nil>"
w = os.Stdout
fmt.Printf("%T\n", w) // "*os.File"
w = new(bytes.Buffer)
fmt.Printf("%T\n", w) // "*bytes.Buffer"
```

一个接口可能只包含动态类型，动态值是nil，此时可能会有非常隐藏的bug。下面这段代码中，预期是debug变量为false时，禁止对输出的收集：

```Go
const debug = true

func main() {
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
    }
    f(buf) // NOTE: subtly incorrect!
    if debug {
        // ...use buf...
    }
}

// If out is non-nil, output will be written to it.
func f(out io.Writer) {
    // ...do something...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
```

但是实际上在debug为false时，下面这行代码发生了panic：

```Go
if out != nil {
    out.Write([]byte("done!\n")) // panic: nil pointer dereference
}
```

原因是main函数调用函数f的时候，自动给f的入参out赋值了一个类型为*bytes.Buffer的空指针，此时out的动态值是nil，动态类型是*bytes.Buffer，此时out是一个包含空指针值的非空接口，所以它不等于nil，但是调用方法时还是会出现空指针。

正确的方式是将实参和形参的类型统一起来，避免一开始就将一个不完整的值赋值给这个接口：

```Go
var buf io.Writer
if debug {
    buf = new(bytes.Buffer) // enable collection of output
}
f(buf) // OK
```

Go中当接口只有一个实现的时候，一般不需要定义接口。有一种特殊情况除外：具体类型和接口不在一个包的时候，此时这个接口是解耦两个包的好方式。

## 排序

sort包提供了排序功能。

一个内置的排序算法需要知道三个东西：序列的长度，表示两个元素比较的结果，一种交换两个元素的方式；这就是sort.Interface的三个方法：

```Go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}
```

下面是对一个字符串切片排序的例子：

```Go
type StringSlice []string
func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

sort.Sort(StringSlice(names))
```

sort包提供了StringSlice类型，直接调用sort.Strings(names)即可，int和float64也是一样。

将排序顺序转换为逆序：

```Go
sort.Sort(sort.Reverse(byArtist(tracks)))
```

sort.Reverse函数返回一个sort包不公开的类型reverse，它内嵌了一个sort.Interface，然后定义了Less方法去覆盖sort.Interface已经提供的Less方法，通过这样的方式让排序变成逆序：

```Go
package sort

type reverse struct{ Interface } // that is, sort.Interface

func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }

func Reverse(data Interface) Interface { return reverse{data} }
```

sort包提供的IsSorted函数会检查一个序列是否已经有序，它也使用sort.Interface进行抽象，但是它从不会调用Swap方法：这段代码示范了IntsAreSorted和Ints函数在IntSlice类型上的使用：

```Go
values := []int{3, 1, 4, 1}
fmt.Println(sort.IntsAreSorted(values)) // "false"
sort.Ints(values)
fmt.Println(values)                     // "[1 1 3 4]"
fmt.Println(sort.IntsAreSorted(values)) // "true"
sort.Sort(sort.Reverse(sort.IntSlice(values)))
fmt.Println(values)                     // "[4 3 1 1]"
fmt.Println(sort.IntsAreSorted(values)) // "false"
```

## 类型断言

类型断言判断接口的动态类型是否和断言的类型匹配。

基本语法是x.(T)，x就是接口，T是具体的类型，如果成功它会返回具体的接口值，如果失败会抛出panic：

```Go
var w io.Writer
w = os.Stdout
f := w.(*os.File)      // success: f == os.Stdout
c := w.(*bytes.Buffer) // panic: interface holds *os.File, not *bytes.Buffer
```

如果T是一个接口类型，那么断言就是在判断x的动态类型是否是T的子类型，如果成功则返回T，否则panic：

```Go
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // success: *os.File has both Read and Write
w = new(ByteCounter)
rw = w.(io.ReadWriter) // panic: *ByteCounter has no Read method
```

如果x是一个nil接口值，那么类型断言会直接失败。

类型断言也可以返回两个值，当返回两个值的时候，第二个返回值就代表是否检查成功，此时即使失败也不会panic：

```Go
var w io.Writer = os.Stdout
f, ok := w.(*os.File)      // success:  ok, f == os.Stdout
b, ok := w.(*bytes.Buffer) // failure: !ok, b == nil

if f, ok := w.(*os.File); ok {
    // ...use f...
}
```

类型断言常用于检查错误类型，os包提供了IsNotExist来检查是否是文件不存在的错误：

```Go
import (
    "errors"
    "syscall"
)

var ErrNotExist = errors.New("file does not exist")

// IsNotExist returns a boolean indicating whether the error is known to
// report that a file or directory does not exist. It is satisfied by
// ErrNotExist as well as some syscall errors.
func IsNotExist(err error) bool {
    if pe, ok := err.(*PathError); ok {
        err = pe.Err
    }
    return err == syscall.ENOENT || err == ErrNotExist
}

_, err := os.Open("/no/such/file")
fmt.Println(err) // "open /no/such/file: No such file or directory"
fmt.Printf("%#v\n", err)
// Output:
// &os.PathError{Op:"open", Path:"/no/such/file", Err:0x2}
```

在该函数内部就使用了断言，检查error是否是*PathError类型，再进一步检查它内部的error类型。

os提供了三种常用错误的检查：

```Go
package os

func IsExist(err error) bool
func IsNotExist(err error) bool
func IsPermission(err error) bool
```

类型断言另一个有用的应用场景是，在代码中判断变量是否是某个具体类型，如果是，就调用它更具体的方法。例如下面的程序中，WriteString比Write更高效，所以在程序中检查是否有这个方法，如果有就调用它：

```Go
// writeString writes s to w.
// If w has a WriteString method, it is invoked instead of w.Write.
func writeString(w io.Writer, s string) (n int, err error) {
    type stringWriter interface {
        WriteString(string) (n int, err error)
    }
    if sw, ok := w.(stringWriter); ok {
        return sw.WriteString(s) // avoid a copy
    }
    return w.Write([]byte(s)) // allocate temporary copy
}

func writeHeader(w io.Writer, contentType string) error {
    if _, err := writeString(w, "Content-Type: "); err != nil {
        return err
    }
    if _, err := writeString(w, contentType); err != nil {
        return err
    }
    // ...
}
```

fmt.Fprintf函数会区分传入的值到底是什么类型，然后根据不同的类型调用不同的方法：

```Go
package fmt

func formatOneValue(x interface{}) string {
    if err, ok := x.(error); ok {
        return err.Error()
    }
    if str, ok := x.(Stringer); ok {
        return str.String()
    }
    // ...all other types...
}
```

在查询数据库时，经常会使用这样的API：

```Go
import "database/sql"

func listTracks(db sql.DB, artist string, minYear, maxYear int) {
    result, err := db.Exec(
        "SELECT * FROM tracks WHERE artist = ? AND ? <= year AND year <= ?",
        artist, minYear, maxYear)
    // ...
}
```

在db.Exec的底层会有某个位置，将传入的参数转换为SQL字面量，会根据不同的参数类型进行不同的转换逻辑：

```Go
func sqlQuote(x interface{}) string {
    if x == nil {
        return "NULL"
    } else if _, ok := x.(int); ok {
        return fmt.Sprintf("%d", x)
    } else if _, ok := x.(uint); ok {
        return fmt.Sprintf("%d", x)
    } else if b, ok := x.(bool); ok {
        if b {
            return "TRUE"
        }
        return "FALSE"
    } else if s, ok := x.(string); ok {
        return sqlQuoteString(s) // (not shown)
    } else {
        panic(fmt.Sprintf("unexpected type %T: %v", x, x))
    }
}
```

switch也可以完成这个功能，可以针对不同的类型走不同的分支：

```Go
switch x.(type) {
case nil:       // ...
case int, uint: // ...
case bool:      // ...
case string:    // ...
default:        // ...
}
```

如果想要在不同的分支获取转换后的具体指，可以使用下面的格式：

```Go
func sqlQuote(x interface{}) string {
    switch x := x.(type) {
    case nil:
        return "NULL"
    case int, uint:
        return fmt.Sprintf("%d", x) // x has type interface{} here.
    case bool:
        if x {
            return "TRUE"
        }
        return "FALSE"
    case string:
        return sqlQuoteString(x) // (not shown)
    default:
        panic(fmt.Sprintf("unexpected type %T: %v", x, x))
    }
}
```

在不同的case中，x的类型都不同。

# 文件

## 读文件

os.File封装所有文件相关操作，File是一个结构体。

打开文件直接使用os.Open方法，返回一个File，在使用后文件要及时关闭：

```Go
file,errə:= os.Open("d:/test.txt")
if err != nil {
    fmt.Println("open file err=", err)
}
//输出下文件,看看文件是什么,看出file就是一个指针*File
fmt.Printf("file=%v", file)
//关闭文件
err = file.Close()
if err != nil {
    fmt.Println("close file err=", err)
}
```

读取文件时，可以根据file生成一个reader，然后用reader去逐行读取：

```Go
file,errə:= os.Open("d:/test.txt")
if err != nil {
    fmt.Println("open file err=", err)
}
//当函数退出时,要及时的关闭file
defer file.Close()//要及时关闭file句柄,否则会有内存泄漏.
//创建一个*Reader,是带缓冲的
/*
const (
    defaultBufSize=4096//默认的缓冲区为4096
)
*/
reader := bufio.NewReader(file)
//循环的读取文件的内容
for {
    str,err:=reader.ReadString('\n')//读到一个换行就结束
    if err=io.EOF {/ io.EOF表示文件的末尾
        break
    }
    fmt.Print(str)
}
```

读取文件时也可以用ioutil.ReadFile(file)这种方式一次性将整个文件读到内存中，它直接返回字节数组。

## 写文件

写入文件时，先用OpenFile方法以创建+写入的模式打开文件，然后创建一个writer，用writer写文件：

```Go
filePath := "d:/abc.txt"
file, err := os.OpenFile(filePath, os.O_WRONLY | os.oCREATE,0666)
if err != nil {
    fmt.Printf("open file err=%v\n", err)
    return
}
defer file.Close()//要及时关闭file句柄,否则会有内存泄漏.
//准备写入5句"hello,Gardon
str:="hello,Gardon\n"// \n表示换行
//写入时,使用带缓存的*Writer
writer := bufio.NewWriter(file)
for i := 0; i < 5; i++ {
    writer.WriteString(str)
}
//因为writer是带缓存,因此在调用Writerstring方法时,其实
//内容是先写入到缓存的,所以需要调用Flush方法,将缓冲的数据
//真正写入到文件中,否则文件中会没有数据!!!
writer.Flush()
```

选择不同的文件打开模式可以完成不同的功能，比如覆盖模式可以覆盖原文件。

拷贝文件主要通过io.Copy(writer, reader)函数来完成

## 其他

path和path/filepath包提供了关于文件路径名相关的函数操作。使用斜杠分隔路径可以在任何操作系统上工作。

虽然Go的垃圾回收机制会回收不被使用的内存，但是这不包括操作系统层面的资源，比如打开的文件、网络连接。因此我们必须显式的释放这些资源。

下面是一个将http响应信息写入本地文件的例子：

```Go
// Fetch downloads the URL and returns the
// name and length of the local file.
func fetch(url string) (filename string, n int64, err error) {
    resp, err := http.Get(url)
    if err != nil {
        return "", 0, err
    }
    defer resp.Body.Close()
    local := path.Base(resp.Request.URL.Path)
    if local == "/" {
        local = "index.html"
    }
    f, err := os.Create(local)
    if err != nil {
        return "", 0, err
    }
    n, err = io.Copy(f, resp.Body)
    // Close file, but prefer error from Copy, if any.
    if closeErr := f.Close(); err == nil {
        err = closeErr
    }
    return local, n, err
}
```

用defer去关闭http流是比较简单的方式，但是针对os.Create打开的文件，却没有使用defer去close，因为很多文件系统在写入文件时发生的错误会被延迟到文件关闭时反馈，此时必须显式的处理close可能带来的error。

判断文件是否存在：

```Go
func PathExists(path string) (bool, error) {
    _, err := os.Stat(path)
    // 返回的错误为nil，说明文件或者目录存在
    if err== nil {
        return true, nil
    }
    // 返回的错误类型如果用os.IsNotExist判断为true，说明文件或者目录不存在
    if os.IsNotExist(err) {
        return false, nil
    }
    return false, err
}
```

# 并发

一个并发发起网络请求的程序：

```Go
// Fetchall fetches URLs in parallel and reports their times and sizes.
package main

import (
    "fmt"
    "io"
    "io/ioutil"
    "net/http"
    "os"
    "time"
)

func main() {
    start := time.Now()
    ch := make(chan string)
    for _, url := range os.Args[1:] {
        go fetch(url, ch) // start a goroutine
    }
    for range os.Args[1:] {
        fmt.Println(<-ch) // receive from channel ch
    }
    fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) {
    start := time.Now()
    resp, err := http.Get(url)
    if err != nil {
        ch <- fmt.Sprint(err) // send to channel ch
        return
    }
    nbytes, err := io.Copy(ioutil.Discard, resp.Body)
    resp.Body.Close() // don't leak resources
    if err != nil {
        ch <- fmt.Sprintf("while reading %s: %v", url, err)
        return
    }
    secs := time.Since(start).Seconds()
    ch <- fmt.Sprintf("%.2fs  %7d  %s", secs, nbytes, url)
}
```

goroutine是一种函数的并发执行方式，可以简单的理解成线程，而channel是用来在goroutine之间进行参数传递。

main函数本身也运行在一个goroutine中，而go function则表示创建一个新的goroutine，并在这个新的goroutine中执行这个函数。

当一个goroutine尝试在一个channel上做send或者receive操作时，这个goroutine会阻塞在调用处，直到另一个goroutine从这个channel里接收或者写入值，这样两个goroutine才会继续执行channel操作之后的逻辑。

## Goroutines

当一个程序启动时，其主函数即在一个单独的goroutine中运行，我们叫它main goroutine。

新的goroutine会用go语句来创建，go语句会使其语句中的函数在一个新创建的goroutine中运行，go语句本身会快速返回：

```Go
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```

主函数返回时，所有的goroutine都会被直接打断，程序退出。

## Channels

一个channel是一个通信机制，它可以让一个goroutine通过它给另一个goroutine发送值信息。

每个channel都有一个特殊的类型，也就是channels可发送数据的类型，使用内置的make函数，我们可以创建一个channel：

```Go
ch := make(chan int) // ch has type 'chan int'
```

channel是引用类型，它的零值是nil。如果两个channel引用的是相同的对象，那么它们比较结果是相同的。一个channel也可以和nil进行比较。

用channel发送或者接收值：

```Go
ch <- x  // a send statement
x = <-ch // a receive expression in an assignment statement
<-ch     // a receive statement; result is discarded
```

Channel还支持close操作，用于关闭channel，随后对基于该channel的任何发送操作都将导致panic异常：

```Go
close(ch)
```

对一个已经被close过的channel进行接收操作依然可以接收到之前已经成功发送的数据；如果channel中已经没有数据的话将产生一个零值的数据。

以最简单方式调用make函数创建的是一个无缓存的channel，如果在make时指定第二个参数，对应channel的容量。如果channel的容量大于零，那么该channel就是带缓存的channel：

```Go
ch = make(chan int)    // unbuffered channel
ch = make(chan int, 0) // unbuffered channel
ch = make(chan int, 3) // buffered channel with capacity 3
```

### 不带缓存的Channels

一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作，反之亦然。

基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels。

下面这个程序使用了一个channel来同步两个goroutine：

```Go
func main() {
    conn, err := net.Dial("tcp", "localhost:8000")
    if err != nil {
        log.Fatal(err)
    }
    done := make(chan struct{})
    go func() {
        io.Copy(os.Stdout, conn) // NOTE: ignoring errors
        log.Println("done")
        done <- struct{}{} // signal the main goroutine
    }()
    mustCopy(conn, os.Stdin)
    conn.Close()
    <-done // wait for background goroutine to finish
}
```

程序在退出前总是可以输出done。在上面的例子里面，程序没有通过channel交换数据，它仅仅是用作两个goroutine之间的同步，这时候我们可以用`struct{}`空结构体作为channels元素的类型。

如果使用了无缓存的channel，当没有接收数据时，发送数据的goroutine将永远阻塞，这种情况被称为goroutines泄漏。和垃圾变量不同，泄漏的goroutines并不会被自动回收，这种情况无法自动恢复。

### close channel

下面的程序演示了如何将多个goroutine连接在一起，一个Channel的输出作为下一个Channel的输入。

三个goroutine的作用：

- 第一个goroutine是一个计数器，用于生成0、1、2、……形式的整数序列，然后通过channel将该整数序列发送给第二个goroutine
- 第二个goroutine是一个求平方的程序，对收到的每个整数求平方，然后将平方后的结果通过第二个channel发送给第三个goroutine
- 第三个goroutine是一个打印程序，打印收到的每个整数

```Go
func main() {
    naturals := make(chan int)
    squares := make(chan int)

    // Counter
    go func() {
        for x := 0; ; x++ {
            naturals <- x
        }
    }()

    // Squarer
    go func() {
        for {
            x := <-naturals
            squares <- x * x
        }
    }()

    // Printer (in main goroutine)
    for {
        fmt.Println(<-squares)
    }
}
```

因为第一个gouroutine不断的产生数据，所以在第三个goroutine会无限的打印数字。

一个场景是希望通过channel发送有限的数据。close函数可以用来关闭channel，关闭channel后，再向该channel发送数据将导致panic异常；当一个被关闭的channel中已经发送的数据都被成功接收后，后续的接收操作将不再阻塞，它们会立即返回一个零值。所以仅仅close并不行，而是在接收时判断channel是否已经关闭且没有值可以接收，接收操作可以返回两个值，第二个bool变量就能反应这一点：

```Go
// Squarer
go func() {
    for {
        x, ok := <-naturals
        if !ok {
            break // channel was closed and drained
        }
        squares <- x * x
    }
    close(squares)
}()
```

Go的range循环可以直接在channel上面迭代，当channel被关闭且没有值可以被接收时就会停止循环。改进版的代码如下，第一个goroutine只发送有限的值，之后channel就被关闭；第二个和第三个都用range来取channel中的值：

```Go
func main() {
    naturals := make(chan int)
    squares := make(chan int)

    // Counter
    go func() {
        for x := 0; x < 100; x++ {
            naturals <- x
        }
        close(naturals)
    }()

    // Squarer
    go func() {
        for x := range naturals {
            squares <- x * x
        }
        close(squares)
    }()

    // Printer (in main goroutine)
    for x := range squares {
        fmt.Println(x)
    }
}
```

channel和文件不同，并不是每个channel都必须关闭的。

不管一个channel是否被关闭，当它没有被引用时将会被Go语言的垃圾自动回收器回收。

试图重复关闭一个channel将导致panic异常，试图关闭一个nil值的channel也将导致panic异常。

关闭一个channels还会触发一个广播机制。

### 单方向的channel

有些时候，程序员希望一个channel只能接收数据，或者只能发送数据，加上这样的限制之后就能避免channel被下游程序滥用，提高程序的安全性。

类型`chan<- int`表示一个只发送int的channel，只能发送不能接收。相反，类型`<-chan int`表示一个只接收int的channel，只能接收不能发送。在编译期就会完成这种限制的检查。对一个只接受的channel调用close将直接导致编译错误。

下面的程序演示了单向channel的使用：

```Go
func counter(out chan<- int) {
    for x := 0; x < 100; x++ {
        out <- x
    }
    close(out)
}

func squarer(out chan<- int, in <-chan int) {
    for v := range in {
        out <- v * v
    }
    close(out)
}

func printer(in <-chan int) {
    for v := range in {
        fmt.Println(v)
    }
}

func main() {
    naturals := make(chan int)
    squares := make(chan int)
    go counter(naturals)
    go squarer(squares, naturals)
    printer(squares)
}
```

从chan int转换成chan<- int是一种隐式转换，但是反向转换不行。

### 带缓存的channel

带缓存的Channel内部持有一个元素队列。

下面的语句代表channel持有一个最大容量为3的队列，这代表可以无阻塞的情况下向该channel发送三个值：

```Go
ch = make(chan string, 3)
```

向缓存Channel的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。

两种可能阻塞的操作：

- 如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个goroutine执行接收操作而释放了新的队列空间。
- 如果channel是空的，接收操作将阻塞直到有另一个goroutine执行发送操作而向队列插入元素。

内置的cap函数可以返回队列的最大容量；内置的len则返回缓存队列中有效元素的个数：

```Go
fmt.Println(cap(ch)) // "3"
fmt.Println(len(ch)) // "2"
```

不能把带缓存的channel当做一个简单的单线程队列来使用，因为程序有可能面临永远阻塞的风险。

## 多线程同步

有时候会有这样的需求，在main goroutine中开启多个goroutine，每个goroutine独立运行，main goroutine等到其他任务都跑完之后再继续运行。

在goroutine个数确定的情况下，每完成一个任务可以向goroutine中发起一个事件，然后在main goroutine中取固定数量的事件：

```Go
// makeThumbnails3 makes thumbnails of the specified files in parallel.
func makeThumbnails3(filenames []string) {
    ch := make(chan struct{})
    for _, f := range filenames {
        go func(f string) {
            thumbnail.ImageFile(f) // NOTE: ignoring errors
            ch <- struct{}{}
        }(f)
    }
    // Wait for goroutines to complete.
    for range filenames {
        <-ch
    }
}
```

这里在循环中调用go的时候要注意，不能直接将循环变量在匿名函数中使用，而是传入作为入参。下面这种写法是错误的，因为循环变量的地址一直在变化：

```Go
for _, f := range filenames {
    go func() {
        thumbnail.ImageFile(f) // NOTE: incorrect!
        // ...
    }()
}
```

如果想在各goroutine中发送结果，然后在main goroutine中汇总，下面是一个典型的错误案例：

```Go
// makeThumbnails4 makes thumbnails for the specified files in parallel.
// It returns an error if any step failed.
func makeThumbnails4(filenames []string) error {
    errors := make(chan error)

    for _, f := range filenames {
        go func(f string) {
            _, err := thumbnail.ImageFile(f)
            errors <- err
        }(f)
    }

    for range filenames {
        if err := <-errors; err != nil {
            return err // NOTE: incorrect: goroutine leak!
        }
    }

    return nil
}
```

这个程序的错误在于在main goroutine中处理error的时候，一旦拿到一个非空的error之后，程序就返回了，不再从channel中取值之后，往channel发送值的goroutine都会永远阻塞下去。

解决该问题的办法有几种：

- 用一个具有合适大小的buffered channel，这样这些worker goroutine向channel中发送错误时就不会被阻塞
- 创建一个另外的goroutine，当main goroutine返回第一个错误的同时去排空channel

当goroutine的个数是未知的，无法简单的用一个循环去取结果的时候，这个时候情况比较复杂，这里使用了sync.WaitGroup来同步，它在每一个goroutine启动时加一，在goroutine退出时减一，当它减到0之前就会一直等待：

```Go
// makeThumbnails6 makes thumbnails for each file received from the channel.
// It returns the number of bytes occupied by the files it creates.
func makeThumbnails6(filenames <-chan string) int64 {
    sizes := make(chan int64)
    var wg sync.WaitGroup // number of working goroutines
    for f := range filenames {
        wg.Add(1)
        // worker
        go func(f string) {
            defer wg.Done()
            thumb, err := thumbnail.ImageFile(f)
            if err != nil {
                log.Println(err)
                return
            }
            info, _ := os.Stat(thumb) // OK to ignore error
            sizes <- info.Size()
        }(f)
    }

    // closer
    go func() {
        wg.Wait()
        close(sizes)
    }()

    var total int64
    for size := range sizes {
        total += size
    }
    return total
}
```

这个程序有几个要注意的地方：

- 用defer保证每个计数器即使是在出错的情况下依然能被减掉
- close的操作不能放到遍历sizes后面，如果那样的话，sizes一直不会关闭，所以就在range sizes阶段一直阻塞
- close的操作不能放到main goroutine里面，如果那样的话，往sizes里面放了一个事件之后，因为没有从sizes里面取，所以各goroutine也无法往里面放入值，所以会一直阻塞，计数器一直不会被减少

上面的场景依然可以使用buffered channel来控制。

## 限制并发数

下面是一个爬虫程序，该程序使用BFS，从一个初始url触发，不断通过crawl函数爬取其他url，然后再继续爬取：

```Go
func main() {
    worklist := make(chan []string)

    // Start with the command-line arguments.
    go func() { worklist <- os.Args[1:] }()

    // Crawl the web concurrently.
    seen := make(map[string]bool)
    for list := range worklist {
        for _, link := range list {
            if !seen[link] {
                seen[link] = true
                go func(link string) {
                    worklist <- crawl(link)
                }(link)
            }
        }
    }
}
```

这里的初始url是单独开启了一个goroutine，这是为了防止程序阻塞。上面这段程序实际上是把channel当做一个队列使用。

下面是这段代码的修改版本，当worklist为空时退出循环：

```Go
func main() {
    worklist := make(chan []string)
    var n int // number of pending sends to worklist

    // Start with the command-line arguments.
    n++
    go func() { worklist <- os.Args[1:] }()

    // Crawl the web concurrently.
    seen := make(map[string]bool)

    for ; n > 0; n-- {
        list := <-worklist
        for _, link := range list {
            if !seen[link] {
                seen[link] = true
                n++
                go func(link string) {
                    worklist <- crawl(link)
                }(link)
            }
        }
    }
}
```

这段程序没有对创建的goroutine数量做限制，这可能会导致开启非常多的goroutine，把系统拖垮。

为了限制并发数，常用的方法有两个：

1、使用有容量限制的buffered channel来控制并发，类似于信号量。每次向channel发送事件就相当于获取到一个通行证，获取通行证之后再去爬取，结束之后再将通行证归还。下面这段代码确保同一时间最多只有20个调用：

```Go
// tokens is a counting semaphore used to
// enforce a limit of 20 concurrent requests.
var tokens = make(chan struct{}, 20)

func crawl(url string) []string {
    fmt.Println(url)
    tokens <- struct{}{} // acquire a token
    list, err := links.Extract(url)
    <-tokens // release the token
    if err != nil {
        log.Print(err)
    }
    return list
}
```

2、直接使用20个常驻的goroutine来爬取，然后获取结果后统一放到一个channel里面：

```Go
func main() {
    worklist := make(chan []string)  // lists of URLs, may have duplicates
    unseenLinks := make(chan string) // de-duplicated URLs

    // Add command-line arguments to worklist.
    go func() { worklist <- os.Args[1:] }()

    // Create 20 crawler goroutines to fetch each unseen link.
    for i := 0; i < 20; i++ {
        go func() {
            for link := range unseenLinks {
                foundLinks := crawl(link)
                go func() { worklist <- foundLinks }()
            }
        }()
    }

    // The main goroutine de-duplicates worklist items
    // and sends the unseen ones to the crawlers.
    seen := make(map[string]bool)
    for list := range worklist {
        for _, link := range list {
            if !seen[link] {
                seen[link] = true
                unseenLinks <- link
            }
        }
    }
}
```

## Select

当同时需要等待多个channel的时候，可以用select语句，类似于多路复用：

```Go
select {
case <-ch1:
    // ...
case x := <-ch2:
    // ...use x...
case ch3 <- y:
    // ...
default:
    // ...
}
```

select会有多个case，每一个case代表一个通信操作（在某个channel上进行发送或者接收），当某个条件满足时，select才会去通信并执行case之后的语句；这时候其它通信是不会执行的。一个没有任何case的select语句写作select{}，会永远地等待下去。

```Go
func main() {
    // ...create abort channel...

    fmt.Println("Commencing countdown.  Press return to abort.")
    select {
    case <-time.After(10 * time.Second):
        // Do nothing.
    case <-abort:
        fmt.Println("Launch aborted!")
        return
    }
    launch()
}
```

上面这段代码里面time.After会10秒一次发送一个事件，同时abort是一个可以手动触发的事件，select语句会一直等待直到两个事件中的一个到达，无论是abort事件或者一个10秒经过的事件。如果10秒经过了还没有abort事件进入，那么就会执行launch方法。

select的case也可以都使用一个channel，这样就会让程序处于交替执行的状态：

```Go
ch := make(chan int, 1)
for i := 0; i < 10; i++ {
    select {
    case x := <-ch:
        fmt.Println(x) // "0" "2" "4" "6" "8"
    case ch <- i:
    }
}
```

time.Tick只有在程序整个生命周期都需要的时候才能使用，因为一旦任务结束了，但ticker这个goroutine还依然存活，继续徒劳地尝试向channel中发送值，然而这时候已经没有其它的goroutine会从该channel中接收值了，这会导致goroutine泄露。

如果只是想临时用一下，应该使用这种模式：

```Go
ticker := time.NewTicker(1 * time.Second)
<-ticker.C    // receive from the ticker's channel
ticker.Stop() // cause the ticker's goroutine to terminate
```

select的default可以避免使用channel的阻塞，当没有case准备好的时候，就会直接执行default，反复执行default被称为轮训channel。

channel的零值是nil，对一个nil的channel发送和接收操作会永远阻塞，这个特性可能会和select一起使用，来控制激活或者禁用某个case。

## goroutine退出

想要让某个goroutine优雅的退出，一个简单的方法是利用channel的关闭机制，相当于向所有使用channel的goroutine广播一个事件。

封装一个函数来判断是否要取消goroutine，当其他goroutine调用close(done)的时候，cancelled就会返回true：

```Go
var done = make(chan struct{})

func cancelled() bool {
    select {
    case <-done:
        return true
    default:
        return false
    }
}
```

在goroutine中轮训这个函数，判断是否有取消的信号，如果有则直接退出：

```Go
func walkDir(dir string, n *sync.WaitGroup, fileSizes chan<- int64) {
    defer n.Done()
    if cancelled() {
        return
    }
    for _, entry := range dirents(dir) {
        // ...
    }
}
```

在循环select的时候，也要加一个case，避免无限循环下去：

```Go
for {
    select {
    case <-done:
        // Drain fileSizes to allow existing goroutines to finish.
        for range fileSizes {
            // Do nothing.
        }
        return
    case size, ok := <-fileSizes:
        // ...
    }
}
```

综上，想要优雅的结束一个goroutine需要对程序逻辑进行侵入式的修改。有时如果发现有goroutine没有正常终止，可以在调试时故意发起一个panic，runtime会把每一个goroutine的栈dump下来，此时就可以顺利找到原因了。

## 互斥锁

避免并发问题的几种方法：

- 多个goroutine只是读某个共享变量，而不去写，例如map，此时就不存在并发问题
- 一个变量只能被一个goroutine访问，在Go中尽量使用channel去在goroutine之间通信
- 使用锁

下面是一个用channel实现的简单的互斥锁，这是一个容量只有1的channel：

```Go
var (
    sema    = make(chan struct{}, 1) // a binary semaphore guarding balance
    balance int
)

func Deposit(amount int) {
    sema <- struct{}{} // acquire token
    balance = balance + amount
    <-sema // release token
}

func Balance() int {
    sema <- struct{}{} // acquire token
    b := balance
    <-sema // release token
    return b
}
```

这个channel可以直接被换成sync包里的Mutex类型：

```Go
import "sync"

var (
    mu      sync.Mutex // guards balance
    balance int
)

func Deposit(amount int) {
    mu.Lock()
    balance = balance + amount
    mu.Unlock()
}

func Balance() int {
    mu.Lock()
    b := balance
    mu.Unlock()
    return b
}
```

这样就保证了存款和查余额这两个函数都并发安全。

在使用锁的时候，一般结合defer使用，即使函数错误也能保证解锁：

```Go
func Balance() int {
    mu.Lock()
    defer mu.Unlock()
    return balance
}
```

Go中的锁不支持可重入，设计者认为可重入的设计可能会导致bug

## 读写锁

在仅仅保证并发读是安全的情况下，使用互斥锁是没有必要的，例如在读取余额的时候，此时可以使用读写锁：

```Go
var mu sync.RWMutex
var balance int
func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
```

sync.RWMutex也提供了mu.Lock和mu.Unlock方法来获取互斥锁。

读锁允许多个只读操作并行执行，但写操作会完全互斥。

在读取变量的时候也要考虑并发安全，锁不仅可以解决goroutine先后调用的问题，还能阻止编译器指令重排序和并发下共享变量的可见性。

## 惰性初始化

有时会遇到这种场景：希望某个变量只初始化一次，然后后续它就可以供多个goroutine读取，为了保证在初始化的时候不出现并发问题，可以采用下面的解决方案：

```Go
var mu sync.Mutex // guards icons
var icons map[string]image.Image

// Concurrency-safe.
func Icon(name string) image.Image {
    mu.Lock()
    defer mu.Unlock()
    if icons == nil {
        loadIcons()
    }
    return icons[name]
}
```

要惰性初始化的变量是icons，每次获取值的时候都加锁，然后检查它是否初始化，如果没有的话再初始化。这种方式对性能影响较大，即使已经初始化完成了还要加锁，为了解决这个问题一个优化的版本如下：

```Go
var mu sync.RWMutex // guards icons
var icons map[string]image.Image
// Concurrency-safe.
func Icon(name string) image.Image {
    mu.RLock()
    if icons != nil {
        icon := icons[name]
        mu.RUnlock()
        return icon
    }
    mu.RUnlock()

    // acquire an exclusive lock
    mu.Lock()
    if icons == nil { // NOTE: must recheck for nil
        loadIcons()
    }
    icon := icons[name]
    mu.Unlock()
    return icon
}
```

这里引入了一个允许多读的锁，在已经初始化了的场景，它会加读锁后快速返回，可以允许多个goroutine同时读取。需要注意的是锁必须先释放共享锁，然后再加互斥锁。

上面这段代码虽然能实现功能但是比较繁琐，sync包提供了一个工具sync.Once来解决这个问题，它的入参是只需要执行一次的初始化函数：

```Go
var loadIconsOnce sync.Once
var icons map[string]image.Image
// Concurrency-safe.
func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

它的底层是用一个锁+一个是否已经初始化的boolean变量来实现的。

## goroutine和线程

goroutine和线程的区别：

- 线程占用的栈内存是固定的，一般是2M；goroutine一开始的时候栈只需要2KB，栈的大小会根据需要动态地伸缩
- 线程会被操作系统内核调度，每几毫秒都会调用中断处理器，挂起当前的线程，检查线程列表并决定下一次哪个线程可以被运行，经常需要保存一个线程的状态，然后恢复另外一个线程；Go会在n个操作系统线程上多工（调度）m个goroutine，每个goroutine根据Go语言本身的语义决定，只有代码中调用了睡眠或者阻塞时，goroutine才会休眠并开始执行另一个goroutine。goroutine调度不需要进入内核态，调度的开销很低
- 线程是有线程号的，goroutine则没有id的概念，这是为了防止程序与goroutine id耦合，避免线程的身份影响函数的行为

Go的调度器使用了一个叫做GOMAXPROCS的变量来决定会有多少个操作系统的线程同时执行Go的代码。其默认的值是运行机器上的CPU的核心数，所以在一个有8个核心的机器上时，调度器一次会在8个OS线程上去调度GO代码。在休眠中的或者在通信中被阻塞的goroutine是不需要一个对应的线程来做调度的。

可以用GOMAXPROCS的环境变量来显式地控制这个参数，或者也可以在运行时用runtime.GOMAXPROCS函数来修改它：

```Go
for {
    go fmt.Print(0)
    fmt.Print(1)
}

$ GOMAXPROCS=1 go run hacker-cliché.go
111111111111111111110000000000000000000011111...

$ GOMAXPROCS=2 go run hacker-cliché.go
010101010101010101011001100101011010010100110...
```

# 反射

Go语言提供了一种机制，能够在运行时更新变量和检查它们的值、调用它们的方法和它们支持的内在操作，而不需要在编译时就知道这些变量的具体类型。这种机制被称为反射。反射是一个复杂的内省技术，不应该随意使用。

fmt.Sprint可以接收任意类型的入参，它的实现中，不可能去在代码中穷尽区分所有的类型，因为有些类型可能是自定义的，所以需要反射来完成这项任务。

反射是一个强大的工具，但是它应该被谨慎使用，原因是：

- 反射的代码只有运行中才能发现问题，而不是编译时，它的风险更大，降低了程序的安全性
- 可读性差，要维护复杂的文档说明功能
- 反射的性能差，比正常的代码运行速度慢一到两个数量级

## reflect.Type和reflect.Value

reflect.Type代表一个Go类型，函数 reflect.TypeOf 接受任意的 interface{} 类型，并以 reflect.Type 形式返回其动态类型：

```Go
t := reflect.TypeOf(3)  // a reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t)          // "int"
```

reflect.TypeOf它返回的是一个动态类型的接口值，它总是返回具体的类型：

```Go
var w io.Writer = os.Stdout
fmt.Println(reflect.TypeOf(w)) // "*os.File"
```

fmt.Printf 提供了一个缩写 %T 参数，内部使用 reflect.TypeOf 来输出变量的动态类型：

```Go
fmt.Printf("%T\n", 3) // "int"
```

reflect.Value代表一个任意类型的值，函数 reflect.ValueOf 接受任意的 interface{} 类型，并返回一个装载着其动态值的 reflect.Value，它返回的也是具体的类型：

```Go
v := reflect.ValueOf(3) // a reflect.Value
fmt.Println(v)          // "3"
fmt.Printf("%v\n", v)   // "3"
fmt.Println(v.String()) // NOTE: "<int Value>"
```

除非 Value 持有的是字符串，否则 String 方法只返回其类型。

fmt.Printf 提供了一个缩写 %v 参数，内部使用 reflect.Value 来输出变量的值。

reflect.Type和reflect.Value可以互相转换，对 Value 调用 Type 方法将返回具体类型所对应的 reflect.Type：

```Go
t := v.Type()           // a reflect.Type
fmt.Println(t.String()) // "int"
```

reflect.ValueOf 的逆操作是 reflect.Value.Interface 方法，它会将reflect.Value还原到interface{} 类型：

```Go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface()      // an interface{}
i := x.(int)            // an int
fmt.Printf("%d\n", i)   // "3"
```

## reflect.Value的各种方法

reflect.Value比interface{}好的地方是，interface{} 隐藏了值的表示方式和所有方法，因此只有我们知道具体的动态类型才能使用类型断言来访问内部的值。而一个reflect.Value则有很多方法来检查其中的内容，例如Kind方法，它会返回类型，这些类型涵盖Go中所有的类型，包括表示空值的 Invalid 类型。Kind方法能替代类型断言实现任意的打印函数：

```Go
package format

import (
    "reflect"
    "strconv"
)

// Any formats any value as a string.
func Any(value interface{}) string {
    return formatAtom(reflect.ValueOf(value))
}

// formatAtom formats a value without inspecting its internal structure.
func formatAtom(v reflect.Value) string {
    switch v.Kind() {
    case reflect.Invalid:
        return "invalid"
    case reflect.Int, reflect.Int8, reflect.Int16,
        reflect.Int32, reflect.Int64:
        return strconv.FormatInt(v.Int(), 10)
    case reflect.Uint, reflect.Uint8, reflect.Uint16,
        reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        return strconv.FormatUint(v.Uint(), 10)
    // ...floating-point and complex cases omitted for brevity...
    case reflect.Bool:
        return strconv.FormatBool(v.Bool())
    case reflect.String:
        return strconv.Quote(v.String())
    case reflect.Chan, reflect.Func, reflect.Ptr, reflect.Slice, reflect.Map:
        return v.Type().String() + " 0x" +
            strconv.FormatUint(uint64(v.Pointer()), 16)
    default: // reflect.Array, reflect.Struct, reflect.Interface
        return v.Type().String() + " value"
    }
}
```

下面是一个打印一个值完整结构的函数：

```Go
func Display(name string, x interface{}) {
    fmt.Printf("Display %s (%T):\n", name, x)
    display(name, reflect.ValueOf(x))
}

func display(path string, v reflect.Value) {
    switch v.Kind() {
    case reflect.Invalid:
        fmt.Printf("%s = invalid\n", path)
    case reflect.Slice, reflect.Array:
        for i := 0; i < v.Len(); i++ {
            display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
        }
    case reflect.Struct:
        for i := 0; i < v.NumField(); i++ {
            fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
            display(fieldPath, v.Field(i))
        }
    case reflect.Map:
        for _, key := range v.MapKeys() {
            display(fmt.Sprintf("%s[%s]", path,
                formatAtom(key)), v.MapIndex(key))
        }
    case reflect.Ptr:
        if v.IsNil() {
            fmt.Printf("%s = nil\n", path)
        } else {
            display(fmt.Sprintf("(*%s)", path), v.Elem())
        }
    case reflect.Interface:
        if v.IsNil() {
            fmt.Printf("%s = nil\n", path)
        } else {
            fmt.Printf("%s.type = %s\n", path, v.Elem().Type())
            display(path+".value", v.Elem())
        }
    default: // basic types, channels, funcs
        fmt.Printf("%s = %s\n", path, formatAtom(v))
    }
}
```

在函数里面先使用Kind函数判断类型，然后根据不同的类型做不同的处理。比如如果发现是数组或者slice就遍历它的每一个值，然后进行递归。虽然reflect.Value类型带有很多方法，但是只有少数的方法能对任意值都安全调用。例如，Index方法只能对Slice、数组或字符串类型的值调用，如果对其它类型调用则会导致panic异常。

反射能够访问到结构体中未导出的成员，所以输出的结果并不稳定，可能在不同版本或者不同操作系统都会有变化，这也是将这些成员定义为私有成员的原因之一。

目前该函数是不支持对象内部带有回环的情况的，例如一个首位相连的链表，为了支持这种场景必须额外记录迄今访问的路径。fmt.Sprint入参是带环的数据结构是没有问题的，因为它针对指针的时候只会打印它的值，而不会做下一步的探索。

## 反射修改值

在Go中变量是可取地址的，但是表达式不能取地址。所有通过reflect.ValueOf(x)返回的reflect.Value都是不可取地址的。当它解引用的时候，才能取地址：

```Go
x := 2                   // value   type    variable?
a := reflect.ValueOf(2)  // 2       int     no
b := reflect.ValueOf(x)  // 2       int     no
c := reflect.ValueOf(&x) // &x      *int    no
d := c.Elem()            // 2       int     yes (x)
```

可以通过调用reflect.Value的CanAddr方法来判断其是否可以被取地址：

```Go
fmt.Println(a.CanAddr()) // "false"
fmt.Println(b.CanAddr()) // "false"
fmt.Println(c.CanAddr()) // "false"
fmt.Println(d.CanAddr()) // "true"
```

原始的reflect.ValueOf得到的结果都是不能取地址的，需要进一步转换，例如e是一个slice，reflect.ValueOf(e)不可取地址，但是reflect.ValueOf(e).Index(i)对应的值是可取地址的。

对于可取地址的reflect.Value，可以用它来访问变量，一般通过下面的方式，第一步是调用Addr()方法，它返回一个Value，里面保存了指向变量的指针。然后是在Value上调用Interface()方法，也就是返回一个interface{}，里面包含指向变量的指针。最后，如果我们知道变量的类型，我们可以使用类型的断言机制将得到的interface{}类型的接口强制转为普通的类型指针：

```Go
x := 2
d := reflect.ValueOf(&x).Elem()   // d refers to the variable x
px := d.Addr().Interface().(*int) // px := &x
*px = 3                           // x = 3
fmt.Println(x)                    // "3"
```

还有一种方式就是不使用指针，而是通过调用可取地址的reflect.Value的reflect.Value.Set方法来更新对应的值：

```Go
d.Set(reflect.ValueOf(4))
fmt.Println(x) // "4"
```

Set方法会在运行时对可赋值性进行检查，当类型不一致时会直接panic。对不可取地址的reflect.Value调用Set方法也会导致panic异常。

为了方便的设置值，reflect.Value提供了SetInt、SetUint、SetString和SetFloat等方法：

```Go
d := reflect.ValueOf(&x).Elem()
d.SetInt(3)
fmt.Println(x) // "3"
```

SetInt这类方法会尽可能的完成任务，即使它不是数字，是自己定义的类型，只要它底层是数字就可以。要注意的是：对于一个引用interface{}类型的reflect.Value调用SetInt会导致panic异常，即使那个interface{}变量对于整数类型也不行。

通过这些特性，可以修改结构体内部字段的值，但是不能修改未导出的成员：

```Go
stdout := reflect.ValueOf(os.Stdout).Elem() // *os.Stdout, an os.File var
fmt.Println(stdout.Type())                  // "os.File"
fd := stdout.FieldByName("fd")
fmt.Println(fd.Int()) // "1"
fd.SetInt(2)          // panic: unexported field
```

需要在修改值之前做一些必要的检查，防止出错：

```Go
fmt.Println(fd.CanAddr(), fd.CanSet()) // "true false"
```

## 结构体成员标签

reflect.Type的Field方法返回一个reflect.StructField，里面含有每个成员的名字、类型和可选的成员标签等信息。它的Tag成员就代表标签信息，然后用Get可以取到标签中某个具体的key。

下面是一个将http请求填充到一个结构体变量的过程：

```Go
import "gopl.io/ch12/params"

// search implements the /search URL endpoint.
func search(resp http.ResponseWriter, req *http.Request) {
    var data struct {
        Labels     []string `http:"l"`
        MaxResults int      `http:"max"`
        Exact      bool     `http:"x"`
    }
    data.MaxResults = 10 // set default
    if err := params.Unpack(req, &data); err != nil {
        http.Error(resp, err.Error(), http.StatusBadRequest) // 400
        return
    }

    // ...rest of handler...
    fmt.Fprintf(resp, "Search: %+v\n", data)
}

// Unpack populates the fields of the struct pointed to by ptr
// from the HTTP request parameters in req.
func Unpack(req *http.Request, ptr interface{}) error {
    if err := req.ParseForm(); err != nil {
        return err
    }

    // Build map of fields keyed by effective name.
    fields := make(map[string]reflect.Value)
    v := reflect.ValueOf(ptr).Elem() // the struct variable
    for i := 0; i < v.NumField(); i++ {
        fieldInfo := v.Type().Field(i) // a reflect.StructField
        tag := fieldInfo.Tag           // a reflect.StructTag
        name := tag.Get("http")
        if name == "" {
            name = strings.ToLower(fieldInfo.Name)
        }
        fields[name] = v.Field(i)
    }

    // Update struct field for each parameter in the request.
    for name, values := range req.Form {
        f := fields[name]
        if !f.IsValid() {
            continue // ignore unrecognized HTTP parameters
        }
        for _, value := range values {
            if f.Kind() == reflect.Slice {
                elem := reflect.New(f.Type().Elem()).Elem()
                if err := populate(elem, value); err != nil {
                    return fmt.Errorf("%s: %v", name, err)
                }
                f.Set(reflect.Append(f, elem))
            } else {
                if err := populate(f, value); err != nil {
                    return fmt.Errorf("%s: %v", name, err)
                }
            }
        }
    }
    return nil
}
```

## 调用方法

下面这个例子，获取了某个类型的所有方法：

```Go
// Print prints the method set of the value x.
func Print(x interface{}) {
    v := reflect.ValueOf(x)
    t := v.Type()
    fmt.Printf("type %s\n", t)

    for i := 0; i < v.NumMethod(); i++ {
        methType := v.Method(i).Type()
        fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name,
            strings.TrimPrefix(methType.String(), "func"))
    }
}
```

reflect.Type和reflect.Value都提供了一个Method方法。每次t.Method(i)调用将一个reflect.Method的实例，对应一个用于描述一个方法的名称和类型的结构体。

使用reflect.Value.Call方法，将可以调用一个Func类型的Value。

直接打印一个结构体的所有方法：

```Go
methods.Print(time.Hour)
// Output:
// type time.Duration
// func (time.Duration) Hours() float64
// func (time.Duration) Minutes() float64
// func (time.Duration) Nanoseconds() int64
// func (time.Duration) Seconds() float64
// func (time.Duration) String() string

methods.Print(new(strings.Replacer))
// Output:
// type *strings.Replacer
// func (*strings.Replacer) Replace(string) string
// func (*strings.Replacer) WriteString(io.Writer, string) (int, error)
```

# 底层编程

Go语言的视线隐藏了很多底层细节：

- 垃圾回收可以消除大部分野指针和内存泄漏相关的问题
- 无法获取变量真实的地址，因为垃圾回收器可能会根据需要移动变量的内存位置
- 无法知道当前的goroutine是运行在哪个操作系统线程之上，因为有时GO调度器会自动切换

这些特性可以让Go语言编写的程序具有高度的可移植性，语言的语义在很大程度上是独立于任何编译器实现、操作系统和CPU系统结构。

有的场景下可能要放弃这些语言特性，去完成一些性能更高的方法，例如需要与其他语言编写的库进行互操作，或者用纯Go语言无法实现的某些函数。这些方法不应该轻易使用，因为如果处理不好细节，很容易引发错误，而且这部分功能可能与未来的版本不兼容，涉及到很多底层实现细节。

unsafe包是一个采用特殊方式实现的包，虽然它可以和普通包一样的导入和使用，但它实际上是由编译器实现的。它提供了一些访问语言内部特性的方法，特别是内存布局相关的细节。

# 补充

## 网络

### web服务器

net包可以完成一系列网络操作，例如发起http请求：

```Go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
    "os"
)

func main() {
    for _, url := range os.Args[1:] {
        resp, err := http.Get(url)
        if err != nil {
            fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
            os.Exit(1)
        }
        b, err := io.ReadAll(resp.Body)
        resp.Body.Close()
        if err != nil {
            fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err)
            os.Exit(1)
        }
        fmt.Printf("%s", b)
    }
}
```

注意为了防止资源泄露，需要使用resp.Body.Close关闭resp的Body流

搭建一个简单的web服务器：

```Go
// Server1 is a minimal "echo" server.
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", handler) // each request calls handler
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

// handler echoes the Path component of the request URL r.
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "URL.Path = %q\n", r.URL.Path)
}
```

这个服务器的功能是返回当前用户正在访问的URL。比如用户访问的是 http://localhost:8000/hello ，那么响应是URL.Path = "hello"

main函数将所有发送到/路径下的请求和handler函数关联起来，/开头的请求其实就是所有发送到当前站点上的请求，服务监听8000端口。

还可以同时设置多个函数：

```Go
func main() {
    http.HandleFunc("/", handler)
    http.HandleFunc("/count", counter)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

此时对/count这个url的请求会调用到counter这个函数，其它的url都会调用默认的处理函数。

在这些代码的背后，服务器每一次接收请求处理时都会另起一个goroutine，这样服务器就可以同一时间处理多个请求，所以存在同时访问同一个方法的情况，此时要注意并发问题。

### http.Handler接口

在http服务器的实现中，调用了http包一个非常重要的方法ListenAndServe，它的第一个参数是要监听的地址，第二个参数是一个Handler实例，它用来执行进入服务器的请求：

```Go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

ListenAndServe会一直运行，直到直到这个服务因为一个错误而失败（或者启动失败），它的返回值一定是一个非空的错误。

用ListenAndServe方法直接定义一个服务器：

```Go
func main() {
    db := database{"shoes": 50, "socks": 5}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    switch req.URL.Path {
    case "/list":
        for item, price := range db {
            fmt.Fprintf(w, "%s: %s\n", item, price)
        }
    case "/price":
        item := req.URL.Query().Get("item")
        price, ok := db[item]
        if !ok {
            w.WriteHeader(http.StatusNotFound) // 404
            fmt.Fprintf(w, "no such item: %q\n", item)
            return
        }
        fmt.Fprintf(w, "%s\n", price)
    default:
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such page: %s\n", req.URL)
    }
}
```

在这段代码中，会根据不同的URL触发不同的行为。

当应用越来越复杂时，在一个方法中处理多个不同路径会非常繁琐，所以可以创建一个ServeMux并且使用它将URL和相应处理/list和/price操作的handler联系起来，这些操作逻辑都已经被分到不同的方法中。然后我们在调用ListenAndServe函数中使用ServeMux为主要的handler：

```Go
func main() {
    db := database{"shoes": 50, "socks": 5}
    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list))
    mux.Handle("/price", http.HandlerFunc(db.price))
    log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

type database map[string]dollars

func (db database) list(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
    item := req.URL.Query().Get("item")
    price, ok := db[item]
    if !ok {
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such item: %q\n", item)
        return
    }
    fmt.Fprintf(w, "%s\n", price)
}
```

在这一行代码中：

```Go
mux.Handle("/list", http.HandlerFunc(db.list))
```

Handle方法的第二个入参其实是函数值，它的类型是http.Handler接口，但是我们自定义的函数db.list并没有实现该接口，所以这里使用了http.HandlerFunc进行了一层转换（这里并不是调用http.HandlerFunc，而是从一个func变量转换为了另一个func变量）：

```Go
package http

type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

这种方式让多个方法都能传入对应的接口使用。

因为handler通过这种方式注册非常普遍，ServeMux有一个方便的HandleFunc方法，它帮我们简化handler注册代码成这样：

```Go
mux.HandleFunc("/list", db.list)
mux.HandleFunc("/price", db.price)
```

在大多数程序中，一个web服务器就足够了，所以net/http包提供了一个全局的ServeMux实例DefaultServerMux和包级别的http.Handle和http.HandleFunc函数，最终它就是一开始看到的例子那样：

```Go
func main() {
    db := database{"shoes": 50, "socks": 5}
    http.HandleFunc("/list", db.list)
    http.HandleFunc("/price", db.price)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

## 工具

### 竞争条件检测

Go的runtime和工具链为我们装备了一个复杂但好用的动态分析工具，竞争检查器（the race detector）。

只要在go build，go run或者go test命令后面加上-race的flag，就会让程序在运行或者编译的时候记录下所有可能出现并发问题的点，并打印一份报告。

### 常用命令行

go env：查看Go语言工具涉及的所有环境变量的值，其中重要的是GOPATH工作目录、GOROOT安装目录

go get：下载包

go get -u：保证每个包是最新版本，一般在第一次下载包的时候使用

go build：编译包，可以用来检测包是否正确。如果包是一个库，则忽略输出结果；如果包的名字是main，则会创建一个可执行程序，以导入路径的最后一段作为可执行程序的名字。后面可以跟一个目录名，或者默认指定当前目录对应的包

go run：编译并运行

go install：它也是编译，但是它会保存每个包的编译成果到$GOPATH/pkg目录下，目录路径和 src目录路径对应，可执行程序被保存到$GOPATH/bin目录。

go list：可用包查询

## 测试

在包目录内，所有以`_test.go`为后缀名的源文件在执行go build时不会被构建成包的一部分，它们是go test测试的一部分。

在`*_test.go`文件中，有三种类型的函数：

- 测试函数：以Test为函数名前缀的函数，用于测试程序的一些逻辑行为是否正确。go test命令会调用这些测试函数并报告测试结果是PASS或FAIL。
- 基准测试函数：以Benchmark为函数名前缀的函数，它们用于衡量一些函数的性能。go test命令会多次运行基准测试函数以计算一个平均的执行时间。
- 示例函数：以Example为函数名前缀的函数，提供一个由编译器保证正确性的示例文档。

每个测试函数必须导入testing包。测试函数有如下的签名：

```Go
func TestName(t *testing.T) {
    // ...
}
```

测试函数的名字必须以Test开头，可选的后缀名必须以大写字母开头：

```Go
func TestSin(t *testing.T) { /* ... */ }
func TestCos(t *testing.T) { /* ... */ }
func TestLog(t *testing.T) { /* ... */ }
```

t参数用于报告测试失败和附加的日志信息。下面是一个简单的测试函数：

```Go
package word

import "testing"

func TestPalindrome(t *testing.T) {
    if !IsPalindrome("detartrated") {
        t.Error(`IsPalindrome("detartrated") = false`)
    }
    if !IsPalindrome("kayak") {
        t.Error(`IsPalindrome("kayak") = false`)
    }
}
```

如果测试没有通过的话，使用t.Error报告失败信息。

go test命令后面可以跟一个目录，代表测试目录对应的包，也可以不指定参数，代表采用当前目录对应的包。

有时会出现这样的场景：包A使用包B的基础功能，此时包B的其中一个测试程序是引入包A来测试A使用B的场景，这时会导致包的循环依赖，会导致编译不通过。为了解决这个问题，可以在包B同目录声明一个xxx_test包，包名的`_test`后缀告诉go test工具它应该建立一个额外的包来运行测试。这个xxx_test包此时就相当于一个独立的包了，它同时可以使用包A和包B，所以可以导入他们进行测试。在设计层面，外部测试包是在所有它依赖的包的上层。