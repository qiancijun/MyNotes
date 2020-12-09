# 一、包、变量和函数



## 1.1 包

每个 Go 程序都是由包构成的。

程序从 `main` 包开始运行。

本程序通过导入路径 `"fmt"` 和 `"math/rand"` 来使用这两个包。

按照约定，包名与导入路径的最后一个元素一致。例如，`"math/rand"` 包中的源码均以 `package rand` 语句开始。



``` go
import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("hello world " , rand.Intn(10))
}
```



## 1.2 导入

此代码用圆括号组合了导入，这是“分组”形式的导入语句。

``` go
import "fmt"
import "math"
```



## 1.2.1 导出名

在 Go 中，如果一个名字以大写字母开头，那么它就是已导出的。例如，`Pizza` 就是个已导出名，`Pi` 也同样，它导出自 `math` 包。

`pizza` 和 `pi` 并未以大写字母开头，所以它们是未导出的。

在导入一个包时，你只能引用其中已导出的名字。任何“未导出”的名字在该包外均无法访问。



## 1.3 函数

函数可以没有参数或接受多个参数。

在本例中，`add` 接受两个 `int` 类型的参数。

注意类型在变量名 **之后**。



``` go
import "fmt"

func add(x, y int)  int {
	return x + y
}

func main() {
	fmt.Println(add(1 ,2))
}
```

> 当连续两个或多个函数的已命名形参类型相同时，除最后一个类型以外，其它都可以省略。
>
> x, y int => x int, y int



### 1.3.1 多值返回

函数可以返回任意数量的返回值。

``` go
import "fmt"

func swap(x, y int) (int, int) {
	return y, x
}

func main() {
	a, b := swap(2,1)
	fmt.Println(a, b)
}
```

### 1.3.2 命名返回值

Go 的返回值可被命名，它们会被视作定义在函数顶部的变量。

返回值的名称应当具有一定的意义，它可以作为文档使用。

没有参数的 `return` 语句返回已命名的返回值。也就是 `直接` 返回。

直接返回语句应当仅用在下面这样的短函数中。在长的函数中它们会影响代码的可读性。

``` go
import "fmt"

func fun1() (x, y int) {
	x = 1
	y = 2
	return
}

func main() {
	a, b := fun1()
	fmt.Println(a, b)
}
```



### 1.3.3 函数值

函数也是值。它们可以像其它值一样传递。

函数值可以用作函数的参数或返回值。



## 1.4 变量

`var` 语句用于声明一个变量列表，跟函数的参数列表一样，类型在最后。

就像在这个例子中看到的一样，`var` 语句可以出现在包或函数级别。

``` go
import "fmt"

var i int
var ok bool

func main() {
	var name string = "hello"
	fmt.Println(i, ok, name)
}
```



### 1.4.1 变量的初始化

变量声明可以包含初始值，每个变量对应一个。

如果初始化值已存在，则可以省略类型；变量会从初始值中获得类型。

``` go
import "fmt"

var i, j, k int = 1, 2, 3

func main() {
	var a, b, c bool = true, false, true
	fmt.Println(i, j, k, a, b, c)
}
```



### 1.4.2 短变量声明

在函数中，简洁赋值语句 `:=` 可在类型明确的地方代替 `var` 声明。

函数外的每个语句都必须以关键字开始（`var`, `func` 等等），因此 `:=` 结构不能在函数外使用。

``` go
import "fmt"

// j := 2 Error

func main() {
	i := 1
	ok := true
	fmt.Println(i, ok)
}
```



## 1.5 基本类型

Go 的基本类型有

1. bool
2. string
3. int  int8  int16  int32  int64
4. uint uint8 uint16 uint32 uint64 uintptr
5. byte // uint8 的别名
6. rune // int32 的别名   // 表示一个 Unicode 码点
7. float32 float64
8. complex64 complex128

同导入语句一样，变量声明也可以“分组”成一个语法块。

`int`, `uint` 和 `uintptr` 在 32 位系统上通常为 32 位宽，在 64 位系统上则为 64 位宽。 当你需要一个整数值时应使用 `int` 类型，除非你有特殊的理由使用固定大小或无符号的整数类型。

``` go
import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```



### 1.5.1 零值

没有明确初始值的变量声明会被赋予它们的 **零值**。

1. 数值类型为 `0`，
2. 布尔类型为 `false`，
3. 字符串为 `""`（空字符串）。



``` go
import (
	"fmt"
)

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
}
```



### 1.5.2 类型转换

表达式 `T(v)` 将值 `v` 转换为类型 `T`。

一些关于数值的转换：

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

或者，更加简单的形式：

```go
i := 42
f := float64(i)
u := uint(f)
```

与 C 不同的是，Go 在不同类型的项之间赋值时需要显式转换。试着移除例子中 `float64` 或 `uint` 的转换看看会发生什么。



### 1.5.3 类型推导

在声明一个变量而不指定其类型时（即使用不带类型的 `:=` 语法或 `var =` 表达式语法），变量的类型由右值推导得出。

当右值声明了类型时，新变量的类型与其相同：

```
var i int
j := i // j 也是一个 int
```

不过当右边包含未指明类型的数值常量时，新变量的类型就可能是 `int`, `float64` 或 `complex128` 了，这取决于常量的精度：

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

``` go
import (
	"fmt"
)

func main() {
	i := 1
	ok := false
	f := 3.14
	x := 1 + 2i
	fmt.Printf("i的类型为 %T\n", i)
	fmt.Printf("ok的类型为 %T\n", ok)
	fmt.Printf("f的类型为 %T\n", f)
	fmt.Printf("x的类型为 %T\n", x)
}

// i的类型为 int
// ok的类型为 bool
// f的类型为 float64
// x的类型为 complex128
```



## 1.6 常量

常量的声明与变量类似，只不过是使用 `const` 关键字。

常量可以是字符、字符串、布尔值或数值。

常量不能用 `:=` 语法声明。



``` go
import "fmt"

const PI = 3.14
const name = "Qianci"

func main() {
	fmt.Println(PI, name)
}
```



### 1.6.1 数值常量

数值常量是高精度的 **值**。

一个未指定类型的常量由上下文来决定其类型。

再尝试一下输出 `needInt(Big)` 吧。

（`int` 类型最大可以存储一个 64 位的整数，有时会更小。）

（`int` 可以存放最大64位的整数，根据平台不同有时会更少。）



``` go
import "fmt"

const i = 1 << 62

func main() {
	fmt.Println(i)
}
```





# 二、 流程控制

## 2.1 for

Go 只有一种循环结构：`for` 循环。

基本的 `for` 循环由三部分组成，它们用分号隔开：

- 初始化语句：在第一次迭代前执行
- 条件表达式：在每次迭代前求值
- 后置语句：在每次迭代的结尾执行

初始化语句通常为一句短变量声明，该变量声明仅在 `for` 语句的作用域中可见。

一旦条件表达式的布尔值为 `false`，循环迭代就会终止。

**注意**：和 C、Java、JavaScript 之类的语言不同，Go 的 for 语句后面的三个构成部分外没有小括号， 大括号 `{ }` 则是必须的。

``` go
import "fmt"

func main() {
	sum := 0
	for i := 0; i <= 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

> 初始化语句和后置语句是可选的。



### 2.1.1 Go中的while

``` go
import "fmt"

func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```



### 2.1.2 死循环

如果省略循环条件，该循环就不会结束，因此无限循环可以写得很紧凑。

``` go
func main() {
	for {
	}
}
```



### 2.1.3 range

`for` 循环的 `range` 形式可遍历切片或映射。

当使用 `for` 循环遍历切片时，每次迭代都会返回两个值。第一个值为当前元素的下标，第二个值为该下标所对应元素的一份副本。

``` go
import "fmt"

func main() {
	a := [5]int {1, 2, 3, 4, 5}
	for i, v := range a {
		fmt.Println("下标：", i, " ", "值：", v)
	}
}
/*
下标： 0   值： 1
下标： 1   值： 2
下标： 2   值： 3
下标： 3   值： 4
下标： 4   值： 5
*/
```

可以将下标或值赋予 `_` 来忽略它。

```
for i, _ := range a
for _, value := range a
```

若你只需要索引，忽略第二个变量即可。

```
for i := range a
```



## 2.2 if

Go 的 `if` 语句与 `for` 循环类似，表达式外无需小括号 `( )` ，而大括号 `{ }` 则是必须的。

``` go
import "fmt"

func main() {
	i := 1
	if i > 0 {
		fmt.Println("i大于0")
	}
}
```



### 2.2.1 if的简短形式

同 `for` 一样， `if` 语句可以在条件表达式前执行一个简单的语句。

该语句声明的变量作用域仅在 `if` 之内。

``` go
import "fmt"

func main() {
	if i := 1; i > 0 {
		fmt.Println("i大于0")
	}
}
```

### 2.2.2 if和else

在 `if` 的简短语句中声明的变量同样可以在任何对应的 `else` 块中使用。

```go
import "fmt"

func main() {
	if i := 0; i > 0 {
		fmt.Println("i大于0")
	} else {
		fmt.Println("i小于等于0")
	}
}
```

``` go
import "fmt"

func main() {
	i := -1
	if i > 0 {
		fmt.Println("i大于0")
	} else if i < 0 {
		fmt.Println("i小于0")
	} else {
		fmt.Println("i等于0")
	}
}
```



## 2.3 switch

`switch` 是编写一连串 `if - else` 语句的简便方法。它运行第一个值等于条件表达式的 case 语句。

Go 的 switch 语句类似于 C、C++、Java、JavaScript 和 PHP 中的，不过 Go 只运行选定的 case，而非之后所有的 case。 实际上，Go 自动提供了在这些语言中每个 case 后面所需的 `break` 语句。 除非以 `fallthrough` 语句结束，否则分支会自动终止。 Go 的另一点重要的不同在于 switch 的 case 无需为常量，且取值不必为整数。



``` go
import "fmt"

func main() {
	i := 1
	switch i {
	case 0:
		fmt.Println(0)
	case 1:
		fmt.Println(1)
	default:
		fmt.Println(-1)
	}
}
```

> switch 的 case 语句从上到下顺次执行，直到匹配成功时停止。



## 2.4 defer

defer 语句会将函数推迟到外层函数返回之后执行。

推迟调用的函数其参数会立即求值，但直到外层函数返回前该函数都不会被调用。



``` go
import "fmt"

func main() {
	for i := 1; i <= 10; i++ {
		defer fmt.Println(i)
	}
	fmt.Println("hello")
}

/*
hello
10
9
8
7
6
5
4
3
2
1

Process finished with exit code 0
*/
```





# 三、更多数据类型



## 3.1 指针

Go 拥有指针。指针保存了值的内存地址。

类型 `*T` 是指向 `T` 类型值的指针。其零值为 `nil`。

```go
var p *int
```

`&` 操作符会生成一个指向其操作数的指针。

```go
i := 42
p = &i
```

`*` 操作符表示指针指向的底层值。

```go
fmt.Println(*p) // 通过指针 p 读取 i
*p = 21         // 通过指针 p 设置 i
```

这也就是通常所说的“间接引用”或“重定向”。

与 C 不同，Go 没有指针运算。



``` go
import "fmt"

func main() {
	i := 1
	var p *int = &i
	fmt.Println(&i)
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(i)
}

/*
0xc00000a0e0
0xc00000a0e0
1
1
*/
```



## 3.2 结构体

一个结构体（`struct`）就是一组字段（field）。



``` go
import "fmt"

type Point struct {
	x, y int
}

func main() {
	fmt.Println(Point{1, 2})
}
```



结构体字段使用点号来访问。

``` go
import "fmt"

type Point struct {
	x, y int
}

func main() {
	point := Point{1, 2}
	fmt.Println(point)
	fmt.Println(point.x)
	fmt.Println(point.y)
}
```

结构体字段可以通过结构体指针来访问。

如果我们有一个指向结构体的指针 `p`，那么可以通过 `(*p).X` 来访问其字段 `X`。不过这么写太啰嗦了，所以语言也允许我们使用隐式间接引用，直接写 `p.X` 就可以。

``` go
import "fmt"

type Point struct {
	x, y int
}

func main() {
	point := Point{1, 2}
	p := &point

	fmt.Println((*p).x)
	fmt.Println((*p).y)
	fmt.Println("============")
	fmt.Println(p.x)
	fmt.Println(p.y)
}
```

### 3.2.1 结构体文法

结构体文法通过直接列出字段的值来新分配一个结构体。

使用 `Name:` 语法可以仅列出部分字段。（字段名的顺序无关。）

特殊的前缀 `&` 返回一个指向结构体的指针。



``` go
import "fmt"

type Point struct {
	x, y int
}

var (
	point1 = Point{1, 2}
	point2 = Point{x: 1}
	point3 = Point{}
	p = &Point{2,2}
)

func main() {
	fmt.Println(point1)
	fmt.Println(point2)
	fmt.Println(point3)
	fmt.Println(p)
}

/*
{1 2}
{1 0}
{0 0}
&{2 2}
*/
```



## 3.3 数组

类型 `[n]T` 表示拥有 `n` 个 `T` 类型的值的数组。

表达式

```go
var a [10]int
```

会将变量 `a` 声明为拥有 10 个整数的数组。

数组的长度是其类型的一部分，因此数组不能改变大小。这看起来是个限制，不过没关系，Go 提供了更加便利的方式来使用数组。



``` go
import "fmt"

func main() {
	var a [10]int

	for i := 0; i < 10; i++ {
		a[i] = i
	}

	for i := 0; i < 10; i++ {
		fmt.Println(a[i])
	}
}
```



### 3.3.1 数组的初始化

``` go
import "fmt"

func main() {
	var a = [5]int {1, 3, 5, 7, 9}
	b := [5]int {2, 4, 6, 8, 10}

	for i := 0; i < len(a); i++ {
		fmt.Println(a[i], " " , b[i])
	}
}
```

> len()：求数组的长度



## 3.4 切片

每个数组的大小都是固定的。而切片则为数组元素提供动态大小的、灵活的视角。在实践中，切片比数组更常用。

类型 `[]T` 表示一个元素类型为 `T` 的切片。

切片通过两个下标来界定，即一个上界和一个下界，二者以冒号分隔：

```
a[low : high]
```

它会选择一个半开区间，包括第一个元素，但排除最后一个元素。

以下表达式创建了一个切片，它包含 `a` 中下标从 1 到 3 的元素：

```
a[1:4]
```

``` go
import "fmt"

func main() {
	var a = [5]int {1, 2, 3, 4, 5}
	var b = a[0 : 2] // 两个元素

	for i := 0; i < len(b); i++ {
		fmt.Println(b[i])
	}
}
```

切片并不存储任何数据，它只是描述了底层数组中的一段。

更改切片的元素会修改其底层数组中对应的元素。

与它共享底层数组的切片都会观测到这些修改。



``` go
import "fmt"

func main() {
	var a = [5]int {1, 2, 3, 4, 5}
	var b = a[:] // 数组a的引用

	b[0] = 0
	fmt.Println("切片b")
	for i := 0; i < len(b); i++ {
		fmt.Print(b[i], " ")
	}
	fmt.Println("\n数组a")
	for i := 0; i < len(a); i++ {
		fmt.Print(a[i], " ")
	}
}

/*
切片b
0 2 3 4 5 
数组a
0 2 3 4 5 
*/
```



### 3.4.1 切片文法

切片文法类似于没有长度的数组文法。

这是一个数组文法：

```go
[3]bool{true, true, false}
```

下面这样则会创建一个和上面相同的数组，然后构建一个引用了它的切片：

```go
[]bool{true, true, false}
```



### 3.4.2 切片的默认行为

在进行切片时，你可以利用它的默认行为来忽略上下界。

切片下界的默认值为 `0`，上界则是该切片的长度。

对于数组

```
var a [10]int
```

来说，以下切片是等价的：

```
a[0:10]
a[:10]
a[0:]
a[:]
```



``` go
import "fmt"

func main() {
	var a = [5]int {1, 2, 3, 4, 5}
	var b = a[:] // 数组a的引用
	fmt.Println("切片b")
	for i := 0; i < len(b); i++ {
		fmt.Print(b[i], " ")
	}
	fmt.Println("\n数组a")
	for i := 0; i < len(a); i++ {
		fmt.Print(a[i], " ")
	}
}
```



### 3.4.3 切片的容量和长度

切片拥有 **长度** 和 **容量**。

切片的长度就是它所包含的元素个数。

切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。

切片 `s` 的长度和容量可通过表达式 `len(s)` 和 `cap(s)` 来获取。

只要具有足够的容量，你就可以通过重新切片来扩展一个切片。

``` go
import "fmt"

func main() {
	var a = [5]int {1, 2, 3, 4, 5}
	var b = a[0 : 3]
	fmt.Printf("b: len = %d cap = %d  b: %v \n", len(b), cap(b), b)

	b = a[0 : 2]
	fmt.Printf("b: len = %d cap = %d  b: %v \n", len(b), cap(b), b)

	b = a[2 : ]
	fmt.Printf("b: len = %d cap = %d  b: %v \n", len(b), cap(b), b)
}

/*
b: len = 3 cap = 5  b: [1 2 3] 
b: len = 2 cap = 5  b: [1 2] 
b: len = 3 cap = 3  b: [3 4 5] 
*/
```



### 3.4.4 nil切片

切片的零值是 `nil`。

nil 切片的长度和容量为 0 且没有底层数组。



```go
import "fmt"

func main() {
	var a []int

	if a == nil {
		fmt.Println("nil")
	}
}
```



### 3.4.5 用make创建切片

切片可以用内建函数 `make` 来创建，这也是你创建动态数组的方式。

`make` 函数会分配一个元素为零值的数组并返回一个引用了它的切片：

```
a := make([]int, 5)  // len(a)=5
```

要指定它的容量，需向 `make` 传入第三个参数：

```
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

​	

### 3.4.6 切片中的切片

切片可包含任何类型，甚至包括其它的切片。

``` go
import "fmt"

func main() {
	a := make([][]int, 4)
	for i := 0; i < len(a); i++ {
		a[i] = make([]int, 4)
		for j := 0; j < len(a[i]); j++ {
			a[i][j] = j
		}
	}

	for i := 0; i < len(a); i++ {
		for j := 0; j < len(a[i]); j++ {
			fmt.Print(j, " ")
		}
		fmt.Println()
	}
}

/*
0 1 2 3 
0 1 2 3 
0 1 2 3 
0 1 2 3
*/
```



### 3.4.7 向切片追加元素

为切片追加新的元素是种常用的操作，为此 Go 提供了内建的 `append` 函数。内建函数的[文档](https://go-zh.org/pkg/builtin/#append)对此函数有详细的介绍。

```
func append(s []T, vs ...T) []T
```

`append` 的第一个参数 `s` 是一个元素类型为 `T` 的切片，其余类型为 `T` 的值将会追加到该切片的末尾。

`append` 的结果是一个包含原切片所有元素加上新添加元素的切片。

当 `s` 的底层数组太小，不足以容纳所有给定的值时，它就会分配一个更大的数组。返回的切片会指向这个新分配的数组。

（要了解关于切片的更多内容，请阅读文章 [Go 切片：用法和本质](https://blog.go-zh.org/go-slices-usage-and-internals)。）



``` go
import "fmt"

func main() {
	a := make([]int, 0, 4)
	a = append(a, 1)
	a = append(a, 2)
	a = append(a, 3)
	a = append(a, 4)
	for i := 0; i < len(a); i++ {
		fmt.Println(a[i])
	}
}
```



## 3.5 映射

映射将键映射到值。

映射的零值为 `nil` 。`nil` 映射既没有键，也不能添加键。

`make` 函数会返回给定类型的映射，并将其初始化备用。



``` go
import "fmt"

func main() {
	m := make(map[int]int) // 中括号里的是键，后面的是值
	m[1] = 1
	m[0] = 0
	fmt.Println(m[1])
	fmt.Println(m[0])
}
```

映射的文法与结构体相似，不过必须有键名。

``` go
import "fmt"

func main() {
	var m = map[int]int {
		1 : 1,
		0 : 0, // 必须有逗号
	}
	fmt.Println(m[1])
	fmt.Println(m[0])
}
```

若顶级类型只是一个类型名，你可以在文法的元素中省略它。

``` go
import "fmt"

type Point struct {
	x,y int
}

func main() {
	var m = map[int]Point {
        1 : {1, 1}, // 1 : Point{1, 1},
        2 : {2, 2}, // 2 : Point{2, 2},
	}
	fmt.Println(m[1])
	fmt.Println(m[2])
}

```



### 3.5.1 修改映射

在映射 `m` 中插入或修改元素：

```
m[key] = elem
```

获取元素：

```
elem = m[key]
```

删除元素：

```
delete(m, key)
```

通过双赋值检测某个键是否存在：

```
elem, ok = m[key]
```

若 `key` 在 `m` 中，`ok` 为 `true` ；否则，`ok` 为 `false`。

若 `key` 不在映射中，那么 `elem` 是该映射元素类型的零值。

同样的，当从映射中读取某个不存在的键时，结果是映射的元素类型的零值。

**注** ：若 `elem` 或 `ok` 还未声明，你可以使用短变量声明：

```
elem, ok := m[key]
```

``` go
import "fmt"

type Point struct {
	x,y int
}

func main() {
	var m = map[int]Point {
		1 : {1, 1},
		2 : {2, 2},
	}

	m[1] = Point{3, 3} // 修改映射
	p1 := m[1] // 获取值
	fmt.Println(p1)

	delete(m, 2) // 删除元素
	elem, ok := m[2] // 判断是否存在
	if ok == true {
		fmt.Println(elem)
	} else {
		fmt.Println(ok)
	}
}
```



## 3.6 闭包

Go 函数可以是一个闭包。闭包是一个函数值，它引用了其函数体之外的变量。该函数可以访问并赋予其引用的变量的值，换句话说，该函数被这些变量“绑定”在一起。

例如，函数 `adder` 返回一个闭包。每个闭包都被绑定在其各自的 `sum` 变量上。



``` go
import "fmt"

func add() func(x int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	fun1, fun2 := add(), add()

	for i := 0; i < 5; i++ {
		fun1(i)
		fmt.Println(fun2(i))
	}
	fmt.Println("fun2...")
	for i := 0; i < 5; i++ {
		fmt.Println(fun1(i))
	}
	fmt.Println("fun1...")
}
/*
0
1
3
6
10
fun2...
10
11
13
16
20
fun1...

Process finished with exit code 0
*/
```



### 3.6.1 斐波那契数列

``` go
import "fmt"

func fib() func() int {
	a := 0
	b := 1
	return func() int {
		sum := a + b
		a = b
		b = sum
		return sum
	}
}

func main() {
	f1 := fib()
	for i := 1; i <= 10; i++ {
		fmt.Println(f1())
	}
}
```





# 四、方法和接口



## 4.1 方法

Go 没有类。不过可以为结构体类型定义方法。

方法就是一类带特殊的 **接收者** 参数的函数。

方法接收者在它自己的参数列表内，位于 `func` 关键字和方法名之间。

`dis`方法拥有一个名字为`p`，类型为`Point`的接收者

``` go
type Point struct {
	x, y int
}

func (p Point) dis() int {
	return p.x + p.y
}

func main() {
	p1 := Point{1, 1}
	println(p1.dis())
}
```



方法只是个带接收者参数的函数。

现在这个 `dis`的写法就是个正常的函数，功能并没有什么变化。

也可以为非结构体类型声明方法。

在此例中，我们看到了一个带 `Abs` 方法的数值类型 `MyFloat`。

你只能为在同一包内定义的类型的接收者声明方法，而不能为其它包内定义的类型（包括 `int` 之类的内建类型）的接收者声明方法。

> 就是接收者的类型定义和方法声明必须在同一包内；不能为内建类型声明方法。



``` go
type MyInt int

func (i MyInt) add(a MyInt) MyInt {
	return i + a
}

func main() {
	i := MyInt(1)
	j := MyInt(2)
	println(i.add(j))
}
```





## 4.2 指针接收者

可以为指针接收者声明方法。

这意味着对于某类型 `T`，接收者的类型可以用 `*T` 的文法。（此外，`T` 不能是像 `*int` 这样的指针。）

例如，这里为 `*Point`定义了 `modify` 方法。

指针接收者的方法可以修改接收者指向的值（就像 `modify`在这做的）。由于方法经常需要修改它的接收者，指针接收者比值接收者更常用。

若使用值接收者，那么 `modify` 方法会对原始 `Point` 值的副本进行操作。（对于函数的其它参数也是如此。）`modify` 方法必须用指针接受者来更改 `main` 函数中声明的 `Point` 的值。



``` go
type Point struct {
	x,y int
}

func (p *Point) modify() {
	p.x ++
	p.y ++
}

func main() {
	point := Point{1, 1}
	point.modify()
	println(point.x)
	println(point.y)
}

/*
2
2
*/
```

不使用指针接收者

``` go
type Point struct {
	x,y int
}

func (p Point) modify() {
	p.x ++
	p.y ++
}

func main() {
	point := Point{1, 1}
	point.modify()
	println(point.x)
	println(point.y)
}
/*
1
1
*/
```



## 4.3. 方法与指针重定向

比较以下方法和函数的区别

``` go
type Point struct {
	x,y int
}

func (p Point) modify() {
	p.x ++
	p.y ++
}

func modify2(p *Point) {
	p.x ++
	p.y ++
}
```

``` go
point := Point{1, 1}
p := &point
modify2(point) // Error
modify2(&point) //Correct
point.modify() // Correct
p.modify() // Correct
```

带指针参数的函数必须接受一个指针，而以指针为接收者的方法被调用时，接收者既能为值又能为指针：

对于语句`point.modify()`即便`point`是个值，并非指针，带指针接收者的方法也能被直接调用。为方便起见，Go 会将语句 `point.modify()` 解释为 `(&point).modify()`。



## 4.3 选择值或者指针作为接收者

使用指针接收者的原因有二：

首先，方法能够修改其接收者指向的值。

其次，这样可以避免在每次调用方法时复制该值。若值的类型为大型结构体时，这样做会更加高效。

通常来说，所有给定类型的方法都应该有值或指针接收者，但并不应该二者混用。



## 4.4 接口

**接口类型** 是由一组方法签名定义的集合。

接口类型的变量可以保存任何实现了这些方法的值。



``` go
import "fmt"

type MyInterface interface {
	fun1()
}

type MyStruct1 struct {
	x, y int
}

type MyStruct2 struct {
	x, y float32
}

func (i *MyStruct1) fun1()  {
	fmt.Println(i.x, " ", i.y)
}

func (i MyStruct2) fun1() {
	fmt.Println(i.x, " ", i.y)
}

func main() {
	var a MyInterface
	m1 := MyStruct1{1, 1} // *MyStruct1 实现了MyInterface的fun1()方法
	m2 := MyStruct2{1.1, 2.2} // MyStruct 实现了MyInterface的fun1()方法
	a = &m1;
	a.fun1()

	a = m2
	a.fun1()
}
```



### 4.4.1 接口与隐式实现

类型通过实现一个接口的所有方法来实现该接口。既然无需专门显式声明，也就没有“implements”关键字。

隐式接口从接口的实现中解耦了定义，这样接口的实现可以出现在任何包中，无需提前准备。

因此，也就无需在每一个实现上增加新的接口名称，这样同时也鼓励了明确的接口定义。



### 4.4.2 接口值

接口也是值。它们可以像其它值一样传递。

接口值可以用作函数的参数或返回值。

在内部，接口值可以看做包含值和具体类型的元组：

```go
(value, type)
```

接口值保存了一个具体底层类型的具体值。

接口值调用方法时会执行其底层类型的同名方法。



# 数据结构与算法

## 排序

### 快速排序

``` go
package main

import (
	"fmt"
)


var (
	n int
	nums []int
)


func main() {
	fmt.Scanf("%d", &n)
	nums = make([]int, n, n)

	for i := 0; i < n; i++ {
		fmt.Scanf("%d", &nums[i])
	}
	quick_sort(nums, 0, n - 1)
	for i := 0; i < n; i++ {
		fmt.Print(nums[i], " ")
	}
}

func quick_sort(a []int, l, r int) {
	if l >= r {
		return
	}
	i := l - 1
	j := r + 1
	x := a[(l + r) >> 1]
	for i < j {
		for {
			i++
			if a[i] >= x {
				break
			}
		}
		for {
			j--
			if a[j] <= x {
				break
			}
		}
		if i < j {
			a[i], a[j] = a[j], a[i]
		}
	}
	quick_sort(a, l, j)
	quick_sort(a, j + 1, r)
}
```

