## Go 环境变量
- GOROOT : Go 的安装路径
- GOPATH : 作为编译后二进制的存放目的地和 import 包时的搜素路径
- GOBIN : 代表 Go 编译生成的程序的安装目录，比如通过 go install 命令，会把生成的 Go 程序安装到 GOBIN 目录下，以供你在终端使用。

#### mac下配置go环境变量, ~/.bash_profile
```
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/Users/xinput/go
export GOBIN=$GOPATH/bin
```

## ch01/main.go
``` 
package main                       // 1
import "fmt"                       // 2
func main() {                      // 3
    fmt.Println("Hello, 世界")      // 4
}                                  // 5
```

这五行代码就构成了一个完整的 Go 程序。

```
$ go run ch01/main.go
Hello, 世界
```

## 程序结构分析
要让一个 Go 语言程序成功运行起来，只需要 package main 和 main 函数这两个核心部分。 

- package main 代表的是一个可运行的应用程序。
- main 函数则是这个应用程序的主入口。

- 第一行的 package main。代表当前的 ch01/main.go 文件属于哪个包，其中 package 是 Go 语言声明包的关键字，main 是要声明的包名。在 Go 语言中 main 包是一个特殊的包，代表你的 Go 语言项目是一个可运行的应用程序，而不是一个被其他项目引用的库。

- 第二行的 import "fmt" 是导入一个 fmt 包，其中 import 是 Go 语言的关键字，表示导入包的意思，这里我导入的是 fmt 包，导入的目的是要使用它

- 第三行的 func main() 是定义了一个函数，其中 func 是 Go 语言的关键字，表示要定义一个函数或者方法的意思，main 是函数名，() 空括号表示这个 main 函数不接受任何参数。在 Go 语言中 main 函数是一个特殊的函数，它代表整个程序的入口，也就是程序在运行的时候，会先调用 main 函数，然后通过 main 函数再调用其他函数，达到实现项目业务需求的目的。

- 第四行的 fmt.Println("Hello, 世界") 是通过 fmt 包的 Println 函数打印“Hello, 世界”这段文本。其中 fmt 是刚刚导入的包，要想使用一个包，必须先导入。Println 函数是属于包 fmt 的函数，这里我需要它打印输出一段文本，也就是“Hello, 世界”。

- 第五行的大括号 } 表示 main 函数体的结束。

## 常用的几个go命令
- go run main.go  运行 main 中的go程序

- go build main.go  当前目录生成 main 可执行文件

- go install main.go 把它安装到 $GOBIN 目录或者任意位置，你就可以在任意

## if 条件语句
```
func main() {
    i:=10
    if i >10 {
        fmt.Println("i>10")
    } else {
        fmt.Println("i<=10")
    }
}
```

关于 if 条件语句的使用有一些规则：

- if 后面的条件表达式不需要使用 ()，这和有些编程语言不一样，也更体现 Go 语言的简洁；

- 每个条件分支（if 或者 else）中的大括号是必须的，哪怕大括号里只有一行代码（如示例）；

- if 紧跟的大括号 { 不能独占一行，else 前的大括号 } 也不能独占一行，否则会编译不通过；

- 在 if……else 条件语句中还可以增加多个 else if，增加更多的条件分支。

## switch 选择语句
```
switch i:=6;{
case i>10:
    fmt.Println("i>10")
case i>5 && i<=10:
    fmt.Println("5<i<=10")
default:
    fmt.Println("i<=5")
}
```

## 数组
```
array:=[5]string{"a","b","c","d","e"}
```
- 类型定义为 [5]string
- 大括号中的元素用于初始化数组
- 注意：[5]string 和 [4]string 不是同一种类型，也就是说长度也是数组类型的一部分

## 切片
### 基于数组生成切片
```
array:=[5]string{"a","b","c","d","e"}
slice:=array[2:5]
fmt.Println(slice)
```
- 注意：这里是包含索引 2，但是不包含索引 5 的元素，即在 : 右边的数字不会被包含
- array[:4] 等价于 array[0:4]
- array[1:] 等价于 array[1:len(array)]
- array[:] 等价于 array[0:len(array)]

### 切片声明
```
// 声明一个元素类型为string，长度为4的切片
slice1 := make([]string,4)
fmt.Println(slice1)

// 声明一个元素类型为string，长度为4，容量为8的切片.
slice2 := make([]string,4,8)
fmt.Println(slice2)
fmt.Println(len(slice2))
```

- 切片的容量不能比切片的长度小
- 在创建新切片的时候，最好要让新切片的长度和容量一样，这样在追加操作的时候就会生成新的底层数组，从而和原有数组分离，就不会因为共用底层数组导致修改内容的时候影响多个切片。

## Map
在 Go 语言中，map 是一个无序的 K-V 键值对集合，结构为 map[K]V。其中 K 对应 Key，V 对应 Value。map 中所有的 Key 必须具有相同的类型，Value 也同样，但 Key 和 Value 的类型可以不同。此外，Key 的类型必须支持 == 比较运算符，这样才可以判断它是否存在，并保证 Key 的唯一。

### Map 声明初始化
```
nameAgeMap := make(map[string]int)
nameAgeMap := map[string]int{"xinput":17}
nameAgeMap := map[string]int{}
```

- 方式1、内置的 make 函数
- 方式2、通过字面量的方式
- 方式3、创建 map 的同时添加键值对，如果不想添加键值对，使用空大括号 {} 即可，要注意的是，大括号一定不能省略。

### Map 获取和删除
map 的操作和切片、数组差不多，都是通过 [] 操作符，只不过数组切片的 [] 中是索引，而 map 的 [] 中是 Key。

Go 语言的 map 可以获取不存在的 K-V 键值对，如果 Key 不存在，返回的 Value 是该类型的零值，比如 int 的零值就是 0。所以很多时候，我们需要先判断 map 中的 Key 是否存在

map 的 [] 操作符可以返回两个值：

- 第一个值是对应的 Value；
- 第二个值标记该 Key 是否存在，如果存在，它的值为 true。

### Map 的遍历
```
// 遍历key
for key := range nameAgeMap {
	fmt.Println(key, " = ", nameAgeMap[key])
}
fmt.Println("n")
// 遍历value
for _, value := range nameAgeMap {
	fmt.Println( value)
}

// 遍历 key,value
for key, value := range nameAgeMap {
	fmt.Println(key, " = ", value)
}
```

## String 和 []byte
```
s:="Hello飞雪无情"
bs:=[]byte(s)
fmt.Println(bs)
fmt.Println(s[0],s[1],s[15])
```
- 因为字符串是字节序列，每一个索引对应的是一个字节，而在 UTF8 编码下，一个汉字对应三个字节，所以字符串 s 的长度其实是 17。

- 如果你想把一个汉字当成一个长度计算，可以使用 utf8.RuneCountInString 函数。 fmt.Println(utf8.RuneCountInString(s))

- 使用 for range 对字符串进行循环时，也恰好是按照 unicode 字符进行循环的，所以对于字符串 s 来说，循环了 9 次

```
for i,r:=range s{
	fmt.Println(i,r)
}
```

## 函数
### 函数构成
- 1、任何一个函数的定义，都有一个 func 关键字，用于声明一个函数，就像使用 var 关键字声明一个变量一样；
- 2、然后紧跟的 main 是函数的名字，命名符合 Go 语言的规范即可，比如不能以数字开头；
- 3、main 函数名字后面的一对括号 () 是不能省略的，括号里可以定义函数使用的参数，这里的 main 函数没有参数，所以是空括号 () ；
- 4、括号 () 后还可以有函数的返回值，因为 main 函数没有返回值，所以这里没有定义
- 5、最后就是大括号 {} 函数体了，你可以在函数体里书写代码，写该函数自己的业务逻辑。

### 函数声明
```
func funcName(params) result {
    body
}
```
- 1、关键字 func；
- 2、函数名字 funcName；
- 3、函数的参数 params，用来定义形参的变量名和类型，可以有一个参数，也可以有多个，也可以没有；
- 4、result 是返回的函数值，用于定义返回值的类型，如果没有返回值，省略即可，也可以有多个返回值；
- 5、body 就是函数体，可以在这里写函数的代码逻辑。

### 多值返回
```
func sum1(a,b int) (int,error)  {
	if(a<0 || b<0){
		return 0, errors.New("a或者b不能是负数")
	}
	return a+b, nil
}
```

- 如果函数有多个返回值，返回值部分的类型定义需要使用小括号括起来，也就是 (int,error)
- 这代表函数 sum 有两个返回值，第一个是 int 类型，第二个是 error 类型，我们在函数体中使用 return 返回结果的时候，也要符合这个类型顺序。
- 在函数体中，可以使用 return 返回多个值，返回的多个值通过逗号分隔即可，返回多个值的类型顺序要和函数声明的返回类型顺序一致

### 命名返回参数
不止函数的参数可以有变量名称，函数的返回值也可以，也就是说你可以为每个返回值都起一个名字，这个名字可以像参数一样在函数体内使用。

```
func sum2(a,b int) (sum int,err error)  {
	if(a<0 || b<0){
		return 0, errors.New("a或者b不能是负数")
	}

	sum = a+b
	err = nil
	return
}
```

### 可变参数
```
func sum3(params ...int) int  {
	sum := 0
	for _, param := range params {
		sum += param
	}
	return sum
}
```
- 定义可变参数，只要在参数类型前加三个点 … 
- 可变参数的类型其实就是切片

### 包级函数
如果不同包的函数要被调用，那么函数的作用域必须是公有的，也就是函数名称的首字母要大写，比如 Println

- 1、函数名称首字母小写代表私有函数，只有在同一个包中才可以被调用；
- 2、函数名称首字母大写代表公有函数，不同的包也可以调用；
- 3、任何一个函数都会从属于一个包
- 提示：Go 语言没有用 public、private 这样的修饰符来修饰函数是公有还是私有，而是通过函数名称的大小写来代表，这样省略了烦琐的修饰符，更简洁

### 匿名函数
```
// 匿名函数和闭包
sum2 := func(a, b int) int{
	return a + b
}
fmt.Println(sum2(1,2))
```

### 闭包
有了匿名函数，就可以在函数中再定义函数（函数嵌套），定义的这个匿名函数，也可以称为内部函数。更重要的是，在函数内定义的内部函数，可以使用外部函数的变量等，这种方式也称为闭包。

```
func main()  {
	cl:=colsure()
	fmt.Println(cl())
	fmt.Println(cl())
	fmt.Println(cl())
}

// 闭包
func colsure() func() int {
	i := 0
	return func() int {
		i++
		return i
	}
}
```

输出的结果

```
1
2
3
```
- 这都得益于匿名函数闭包的能力，让我们自定义的 colsure 函数，可以返回一个匿名函数，并且持有外部函数 colsure 的变量 i。因而在 main 函数中，每调用一次 cl()，i 的值就会加 1。


## 方法
### 不同于函数的方法
在 Go 语言中，方法和函数是两个概念，但又非常相似，不同点在于方法必须要有一个接收者，这个接收者是一个类型，这样方法就和这个类型绑定在一起，称为这个类型的方法。

```
func main()  {
    age:=Age(25)
    age.String()
}

func (age Age) String()  {
	fmt.Println("the age is ", age)
}
```

示例中方法 String() 就是类型 Age 的方法，类型 Age 是方法 String() 的接收者。

和函数不同，定义方法时会在关键字 func 和方法名 String 之间加一个接收者 (age Age) ，接收者使用小括号包围。

接收者的定义和普通变量、函数参数等一样，前面是变量名，后面是接收者类型。

现在方法 String() 就和类型 Age 绑定在一起了，String() 是类型 Age 的方法。

定义了接收者的方法后，就可以通过点操作符调用方法

## 结构体
```
type structName struct{
    fieldName typeName
    ....
    ....
}
```

## 接口
```
type Stringer interface {
    String() string
}

func (p person) String() string  {
	return fmt.Sprintf("the name is %s,age is %d",p.name,p.age)
}
```

- 当值类型作为接收者时，person 类型和*person类型都实现了该接口。
- 当指针类型作为接收者时，只有*person类型实现了该接口。

## 工厂函数
工厂函数一般用于创建自定义的结构体，便于使用者调用，我们还是以 person 类型为例，用如下代码进行定义：

```
func newPerson(name string) *person {
    return &person{name: name}
}
```












