### 安装

```bash
ARCH=$(dpkg --print-architecture)
GO_VERSION="go1.23.2"
wget https://go.dev/dl/${GO_VERSION}.linux-$ARCH.tar.gz &&
sudo rm -rf /usr/local/go &&
sudo tar -C /usr/local -xzf ${GO_VERSION}.linux-$ARCH.tar.gz &&
export PATH=$PATH:/usr/local/go/bin &&
echo 'export PATH=$PATH:/usr/local/go/bin' >> .bashrc && 
rm ${GO_VERSION}.linux-$ARCH.tar.gz

# 配置 Go 模块的代理
go env -w GOPROXY=https://goproxy.io,direct
```

### 基础语法

#### 变量声明

##### 数据类型

-   字符串：string，只能用一对双引号（""）或反引号（``）括起来定义，不能用单引号（''）定义
-   布尔：bool，只有 true 和 false，默认为 false
-   整型：(u)int(8, 16, 32, 64)，int和uint具体长度取决于CPU位数
-   浮点型：float32, float64

##### 常量声明

单个常量声明

-   const 变量名称 数据类型 = 变量值 （如果不赋值，采用默认值）
-   const 变量名称 = 变量值 （根据变量值，自行判断数据类型）

多个常量声明

-   const 变量名称，变量名称 ...，数据类型 = 变量值，变量值 ...
-   const 变量名称，变量名称 ... = 变量值，变量值 ...

```go
package main

import "fmt"

func main() {
	const name string = "Tom"
	fmt.Println(name)

	const age = 30
	fmt.Println(age)

	const name1, name2 string = "Tom", "Jay"
	fmt.Println(name1, name2)

	const name3, age1 = "Tom", 30
	fmt.Println(name3, age1)
}
```

##### 变量声明

单个变量声明

-   var 变量名称 数据类型 = 变量值 （如果不赋值，采用默认值）
-   var变量名称 = 变量值 （根据变量值，自行判断数据类型）
-   变量名称 := 变量值 （省略了 var 和数据类型，变量名称一定要是未声明过的）

多个变量声明

-   var 变量名称，变量名称 ...，数据类型 = 变量值，变量值 ...
-   var变量名称，变量名称 ... = 变量值，变量值 ...
-   变量名称,变量名称 ... := 变量值,变量值 ...

```go
package main

import "fmt"

func main() {
	var age1 uint8 = 31
	var age2 = 32
	age3 := 33
	fmt.Println(age1, age2, age3)

	var age4, age5, age6 int32 = 34, 35, 36
	fmt.Println(age4, age5, age6)

	var name1, age7 = "Tom", 37
	fmt.Println(name1, age7)

	name2, is_boy, height := "Jay", false, 178.88
	fmt.Println(name2, is_boy, height)
}
```

#### 输出方法

-   **fmt.Print**：输出到控制台（仅只是输出）
-   **fmt.Println**：输出到控制台并换行
-   **fmt.Printf**：仅输出格式化的字符串和字符串变量（整型和整型变量不可以）
-   **fmt.Sprintf**：格式化并返回一个字符串，不输出。

```go
package main

import "fmt"

func main() {
	fmt.Print("不换行")
	fmt.Println("------")
	fmt.Println("换行")
	fmt.Printf("name = %s, age = %d\n", "Tom", 30)
	fmt.Printf("name = %s, age = %d, height = %v\n", "Tom", 30, fmt.Sprintf("%.2f", 180.567))
}
```

#### 数组

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成，一旦声明了，数组的长度就固定了，不能动态变化

`len()` 和 `cap()` 返回结果始终一样

##### 声明数组

```go
package main

import "fmt"

func main() {
	// 一维数组
	var arr1 [5]int32
	fmt.Println(arr1)

	var arr2 = [5]int32{1, 2, 3, 4, 5}
	fmt.Println(arr2)

	arr3 := [5]int32{1, 2, 3, 4, 5}
	fmt.Println(arr3)

	arr4 := [...]int32{1, 2, 3, 4, 5, 6}
	fmt.Println(arr4)

    // 将数组第 0 位赋值为 3，第 1 位为 5，第 4 位为 6
	arr5 := [5]int32{0: 3, 1: 5, 4: 6} // [3 5 0 0 6]
	fmt.Println(arr5)

	// 二维数组
	var arr6 = [3][5]int32{{1, 2, 3, 4, 5}, {6, 7, 8, 9, 0}, {2, 3, 4, 5, 6}}
	fmt.Println(arr6)

	arr7 := [3][5]int32{{1, 2, 3, 4, 5}, {6, 7, 8, 9, 0}, {2, 3, 4, 5, 6}}
	fmt.Println(arr7)

	arr8 := [...][5]int32{{1, 2, 3, 4, 5}, {6, 7, 8, 9, 0}, {0: 3, 1: 5, 4: 6}}
	fmt.Println(arr8)
}
```

##### 注意事项

1.   数组不可动态变化问题，一旦声明了，其长度就是固定的

```go
package main

import "fmt"

func main() {
	var arr = [5]int{1, 2, 3, 4, 5}
	arr[5] = 6 // invalid argument: index 5 out of bounds [0:5]
	fmt.Println(arr)
}
```

2.   数组是值类型问题，在函数中传递的时候是传递的值，如果传递数组很大，这对内存是很大开销

```go
package main

import "fmt"

func main() {
	var arr = [5]int32{1, 2, 3, 4, 5}
	modifyArr(arr)
	fmt.Println(arr) // [1 2 3 4 5]
}

func modifyArr(a [5]int32) {
	a[1] = 20
}
```

```go
package main

import "fmt"

func main() {
	var arr = [5]int32{1, 2, 3, 4, 5}
	modifyArr(&arr)
	fmt.Println(arr) // [1 20 3 4 5]
}

func modifyArr(a *[5]int32) {
	a[1] = 20
}
```

3.   数组赋值问题，同样类型的数组（长度一样且每个元素类型也一样）才可以相互赋值，反之不可以

```go
package main

import "fmt"

func main() {
	var arr = [5]int32{1, 2, 3, 4, 5}
	var arr1 [5]int32 = arr
	var arr2 [6]int32 = arr // cannot use arr (variable of type [5]int32) as [6]int32 value in variable declaration
}
```

#### 切片-Slice

切片是一种动态数组，比数组操作灵活，长度不是固定的，可以进行追加和删除

`len()` 和 `cap()` 返回结果可相同和不同

```go
package main

import "fmt"

func main() {
	var sli1 []int32 // nil 切片
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli1), cap(sli1), sli1)

	var sli2 = []int32{} // nil 切片
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli2), cap(sli2), sli2)

	var sli3 = []int32{1, 2, 3, 4}
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli3), cap(sli3), sli3)

	sli4 := []int32{1, 2, 3, 4, 5}
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli4), cap(sli4), sli4)

	var sli5 = make([]int32, 5, 8)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli5), cap(sli5), sli5)

	sli6 := make([]int32, 4, 9)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli6), cap(sli6), sli6)
}

/* output
	len = 0 cap = 0 slice = []
	len = 0 cap = 0 slice = []
	len = 4 cap = 4 slice = [1 2 3 4]
	len = 5 cap = 5 slice = [1 2 3 4 5]
	len = 5 cap = 8 slice = [0 0 0 0 0]
	len = 4 cap = 9 slice = [0 0 0 0]
*/
```

截取切片

```go
package main

import "fmt"

func main() {
	sli := []int32{1, 2, 3, 4, 5, 6}
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli), cap(sli), sli)

	fmt.Println("sli[1] = ", sli[1])
	fmt.Println("sli[:] = ", sli[:])
	fmt.Println("sli[1:] = ", sli[1:])
	fmt.Println("sli[:4] = ", sli[:4])

	fmt.Println("sli[0:3] = ", sli[0:3])
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli[0:3]), cap(sli[0:3]), sli[0:3])

	fmt.Println("sli[0:3:4] = ", sli[0:3:4])
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli[0:3:4]), cap(sli[0:3:4]), sli[0:3:4])
}

/* output
	len = 6 cap = 6 slice = [1 2 3 4 5 6]
	sli[1] =  2
	sli[:] =  [1 2 3 4 5 6]
	sli[1:] =  [2 3 4 5 6]
	sli[:4] =  [1 2 3 4]
	sli[0:3] =  [1 2 3]
	len = 3 cap = 6 slice = [1 2 3]
	sli[0:3:4] =  [1 2 3]
	len = 3 cap = 4 slice = [1 2 3]
*/
```

追加切片

```go
package main

import "fmt"

func main() {
	sli := []int32{1, 2, 3}
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli), cap(sli), sli)

    // append 时，容量不够需要扩容时，cap 会翻倍
	sli = append(sli, 4, 5)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli), cap(sli), sli)

	sli = append(sli, 7, 8)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli), cap(sli), sli)

	sli = append(sli, 9)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli), cap(sli), sli)
}

/* output
	len = 3 cap = 3 slice = [1 2 3]
	len = 5 cap = 6 slice = [1 2 3 4 5]
	len = 7 cap = 12 slice = [1 2 3 4 5 7 8]
	len = 8 cap = 12 slice = [1 2 3 4 5 7 8 9]
*/
```

删除切片

```go
package main

import "fmt"

func main() {
	sli := []int32{1, 2, 3, 4, 5, 6, 7, 8}
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli), cap(sli), sli)

	// 删除尾部 2 个元素
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli[:len(sli)-2]), cap(sli[:len(sli)-2]), sli[:len(sli)-2])

	// 删除首部 2 个元素
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli[2:]), cap(sli[2:]), sli[2:])

	// 删除中间 2 个元素
	sli1 := append(sli[:3], sli[5:]...)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli1), cap(sli1), sli1)
	

}

/* output
	len = 8 cap = 8 slice = [1 2 3 4 5 6 7 8]
	len = 6 cap = 8 slice = [1 2 3 4 5 6]
	len = 6 cap = 6 slice = [3 4 5 6 7 8]
	len = 6 cap = 8 slice = [1 2 3 6 7 8]
*/
```

```go
// append 会在第一个参数的上面，进行新的值的添加，即会修改第一个参数的值
package main

import "fmt"

func main() {
	sli := []int32{1, 2, 3, 4, 5, 6, 7, 8}
	fmt.Printf("slice = %v\n", append(sli[:2], sli[5:]...))

	fmt.Printf("slice = %v\n", sli)

	sli1 := []int32{1, 2, 3, 4, 5, 6, 7, 8}
	sli2 := append(sli1[:2], 9)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli1), cap(sli1), sli1)
	fmt.Printf("len = %d cap = %d slice = %v\n", len(sli2), cap(sli2), sli2)
}

/* output
	slice = [1 2 6 7 8]
	slice = [1 2 6 7 8 6 7 8]
	len = 8 cap = 8 slice = [1 2 9 4 5 6 7 8]
	len = 3 cap = 8 slice = [1 2 9]
*/
```

#### 结构体

结构体是将零个或多个任意类型的变量，组合在一起的聚合数据类型，也可以看做是数据的集合

##### 声明结构体

```go
package main

import "fmt"

type Person struct {
	name string
	age  int32
}

func main() {
	var p1 Person
	p1.name = "Tom"
	p1.age = 30
	fmt.Println("p1 = ", p1)

	var p2 = Person{name: "Jay", age: 22}
	fmt.Println("p2 = ", p2)

	p3 := Person{name: "Amy", age: 23}
	fmt.Println("p3 = ", p3)

	// 匿名结构体
	p4 := struct {
		a string
		b int32
	}{a: "a", b: 2}
	fmt.Println("p4 = ", p4)
}
```

##### 序列化和反序列化

```go
package main

import "fmt"
import "encoding/json"

// Go语言通过首字母的大小写来控制访问权限
// 无论是方法，变量，常量或是自定义的变量类型，如果首字母大写，则可以被外部包访问，反之则不可以
// 而结构体中的字段名，如果首字母小写的话，则该字段无法被外部包访问和解析。
type Result struct {
	// JSON 标签用于指定在编码和解码 JSON 时使用的字段名称。
	// 当将 Result 结构体编码为 JSON 时，Code 字段会变成 "code"，Message 字段会变成 "msg"
	Code    int    `json:"code"`
	Message string `json:"msg"`
}

func main() {
	var res = Result{Code: 200, Message: "success"}

	// 序列化
	jsons, errs := json.Marshal(res)
	if errs != nil {
		fmt.Println("json marshal error:", errs)
	}
	fmt.Println("json data:", string(jsons))

	// 反序列化
	var res2 Result
	errs = json.Unmarshal(jsons, &res2)
	if errs != nil {
		fmt.Println("json unmarshal error:", errs)
	}
	fmt.Println("res2:", res2)
}

/* output
	json data: {"code":200,"msg":"success"}
	res2: {200 success}
*/
```

#### map 集合

Map 集合是无序的 key-value 数据结构

Map 集合中的 key / value 可以是任意类型，但所有的 key 必须属于同一数据类型，所有的 value 必须属于同一数据类型，key 和 value 的数据类型可以不相同

##### 声明 map

```go
package main

import "fmt"

func main() {
	var p1 map[int]string = make(map[int]string)
	p1[1] = "Tom"
	fmt.Println("p1:", p1)

	var p2 map[int]string = map[int]string{}
	p2[2] = "Tom"
	fmt.Println("p2:", p2)

	p3 := make(map[int]string)
	p3[3] = "Tom"
	fmt.Println("p3:", p3)

	p4 := map[int]string{}
	p4[4] = "Tom"
	fmt.Println("p4:", p4)

	p5 := map[int]string{
		5: "Tom",
	}
	fmt.Println("p5:", p5)
}
```

##### 序列化和反序列化

```go
package main

import "fmt"
import "encoding/json"

func main() {
    // 创建一个空的 map，键为字符串，值为空接口（可以存储任何类型）
	res := make(map[string]interface{})
	res["code"] = 200
	res["msg"] = "success"
	res["data"] = map[string]interface{}{
		"username" : "Tom",
		"age":"30",
		"hobby":[]string{"读书", "爬山"},
	}
	fmt.Println("map data:",res)

	// 序列化
	jsons, _ := json.Marshal(res)
	fmt.Println("json data:",string(jsons))

	// 反序列化
	res2 := make(map[string]interface{})
	json.Unmarshal(jsons, &res2)
	fmt.Println("map data:", res2)
}
```

##### 编辑和删除

```go
package main

import "fmt"

func main() {
	person := map[int]string{
		1: "Tom",
		2: "Jay",
		3: "John",
	}
	fmt.Println(person)

	delete(person, 2)
	fmt.Println(person)

	person[2] = "Jack"
	person[3] = "amy"
	fmt.Println(person)
}
```

#### 循环

##### 循环array

```go
package main

import "fmt"

func main() {
	person := [3]string{"Tom", "Jay", "Amy"}

	for k, v := range person {
		fmt.Printf("person[%d]: %s\n", k, v)
	}
	fmt.Println("")

	for i := range person {
		fmt.Printf("person[%d]: %s\n", i, person[i])
	}
	fmt.Println("")

	for i := 0; i < len(person); i++ {
		fmt.Printf("person[%d]: %s\n", i, person[i])
	}
	fmt.Println("")

	for _, name := range person {
		fmt.Printf("name: %s\n", name)
	}
}
```

##### 循环slice

```go
package main

import "fmt"

func main() {
	person := []string{"Tom", "Jay", "Amy"}

	for k, v := range person {
		fmt.Printf("person[%d]: %s\n", k, v)
	}
	fmt.Println("")

	for i := range person {
		fmt.Printf("person[%d]: %s\n", i, person[i])
	}
	fmt.Println("")

	for i := 0; i < len(person); i++ {
		fmt.Printf("person[%d]: %s\n", i, person[i])
	}
	fmt.Println("")

	for _, name := range person {
		fmt.Printf("name: %s\n", name)
	}
}
```

##### 循环map

```go
package main

import "fmt"

func main() {
	person := map[int]string{
		1: "Tom",
		2: "Amy",
		3: "Jay",
	}

	for k, v := range person {
		fmt.Printf("person[%d]: %s\n", k, v)
	}
	fmt.Println("")

	for i := range person {
		fmt.Printf("person[%d]: %s\n", i, person[i])
	}
	fmt.Println("")

	for _, name := range person {
		fmt.Printf("name: %s\n", name)
	}
}
```

##### break

跳出当前循环，可⽤于 for、switch、select

```go
package main

import "fmt"

func main() {
	for i := 0; i < 10; i++ {
		if i == 6 {
			break
		}
		fmt.Println(i)
	}
}
```

##### continue

跳过本次循环，只能用于 for

```go
package main

import "fmt"

func main() {

	for i := 0; i < 10; i++ {
		if i%2 == 0 {
			continue
		}
		fmt.Println(i)
	}
}
```

##### goto

改变函数内代码执行顺序，不能跨函数使用

```go
package main

import "fmt"

func main() {

	for i := 0; i < 10; i++ {
		if i == 6 {
			goto END
		}
		fmt.Println(i)
	}
	fmt.Println("test")
END:
	fmt.Println("end")
}
```

##### switch

默认每个 case 带有 break，case 中可以有多个选项，fallthrough 不跳出，并执行下一个 case

```go
package main

import "fmt"

func main() {

	i := 2
	switch i {
		case 1:
			fmt.Println("1")
		case 2:
			fmt.Println("2")
			fallthrough
		case 3:
			fmt.Println("3")
			fallthrough
		default:
			fmt.Println("xxx")
	}
}
```

#### 函数

Go 语言默认是值传递

```go
func functionName(parameter1 type1, parameter2 type2) returnType {
    // 函数体
    // ...
    return value // 返回值
}

// 无参数无返回值
func sayHello() {
	fmt.Println("hello")
}

// 带参数
func add(a int, b int) int {
	return a + b
}

// 带多个返回值
func divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, fmt.Errorf("division by zero")
	}
	return a / b, nil
}

// 命名返回值
func swap(x, y int) (a, b int) {
	a = y
	b = x
	return // 直接返回命名返回值
}

// 匿名函数
func() {
    fmt.Println("hello")
}()

// 带参数和返回值
res := func(a, b int) int {
    return a + b
}(3, 5)
fmt.Println(res)

// 匿名函数赋值给变量
add := func(a, b int) int {
    return a + b
}

fmt.Println(add(3, 4))

// 创建闭包，即可以访问其外部作用域中的变量的匿名函数
counter := 0
increment := func() int {
    counter++
    return counter
}
fmt.Println(increment())
fmt.Println(increment())
```

#### go 关键字

在 go 关键字后面加一个函数，就可以创建一个线程，函数可以为已经写好的函数，也可以是匿名函数

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("main start")

	go func() {
		fmt.Println("goroutine")
	}()

	time.Sleep(1 * time.Second)
	fmt.Println("main end")
}
```

#### chan通道

用于不同 goroutine 之间进行通信的机制。它可以用来安全地传递数据，使得并发编程更容易管理

##### 声明 chan

不带缓冲的通道，进和出都会阻塞

带缓冲的通道，进一次长度 +1，出一次长度 -1，如果长度等于缓冲长度时，再进就会阻塞

```go
// 声明不带缓冲的通道
ch1 := make(chan string)

// 声明带10个缓冲的通道
ch2 := make(chan string, 10)

// 声明只读通道
ch3 := make(<-chan string)

// 声明只写通道
ch4 := make(chan<- string)

// 将 value 发送到通道 ch
ch <- value

// 从通道 ch 接收数据并赋值给 value
value := <-ch

// 关闭通道
// 只读的 chan 不能 close
// close 以后还可以读取数据
close(ch)
```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		ch <- "Hello from goroutine"
	}()

	msg := <-ch
	fmt.Println(msg)
}
```

#### defer 函数

用于延迟函数的执行，直到包含它的函数返回时

`defer` 语句通常用于清理资源，例如关闭文件或解锁互斥锁

```go
package main

import "fmt"

func main() {
	defer fmt.Println("This is deferred")
	fmt.Println("printed first")
}

/*
	printed first
	This is deferred
*/
```

如果有多个 `defer` 语句，它们会按照后进先出（LIFO）的顺序执行

```go
package main

import "fmt"

func main() {
	defer fmt.Println("First deferred.")
	defer fmt.Println("Second deferred.")
	fmt.Println("Main function.")
}

/*
	Main function.
	Second deferred.
	First deferred.
*/
```

`defer` 可以捕获函数调用时的参数值，而不是在执行时的值

```go
package main

import "fmt"

func main() {
	for i := 0; i < 3; i++ {
		defer fmt.Println(i) // 捕获的是当前 i 的值
	}
}

/*
	2
	1
	0
*/
```

#### 解析 json 数据

```go
package main

import (
	"encoding/json"
	"fmt"
)

type MobileInfo struct {
	Resultcode string `json:"resultcode"`
	Reason     string `json:"reason"`
	Result     struct {
		Province string `json:"province"`
		City     string `json:"city"`
		Areacode string `json:"areacode"`
		Zip      string `json:"zip"`
		Company  string `json:"company"`
		Card     string `json:"card"`
	} `json:"result"`
}

func main() {
	jsonStr := `
		{
			"resultcode": "200",
			"reason": "Return Successd!",
			"result": {
				"province": "浙江",
				"city": "杭州",
				"areacode": "0571",
				"zip": "310000",
				"company": "中国移动",
				"card": ""
			}
		}
	`

	var mobile MobileInfo
	err := json.Unmarshal([]byte(jsonStr), &mobile)
	if err != nil {
		fmt.Println(err.Error())
	}
	fmt.Println(mobile.Resultcode)
	fmt.Println(mobile.Reason)
	fmt.Println(mobile.Result.City)
}
```

json 格式的数据类型不确定

或者 json 格式的数据 result 中参数不固定

使用开源库 https://github.com/mitchellh/mapstructure

```go
// 数据类型不确定
package main

import (
	"encoding/json"
	"fmt"
	"github.com/mitchellh/mapstructure"
)

type MobileInfo struct {
	Resultcode string `json:"resultcode"`
}

func main() {
	jsonStr := `
		{
			"resultcode": 200
		}
	`

	var result map[string]interface{}
	err := json.Unmarshal([]byte(jsonStr), &result)
	if err != nil {
		fmt.Println(err.Error())
	}

	var mobile MobileInfo
	err = mapstructure.WeakDecode(result, &mobile)
	if err != nil {
		fmt.Println(err.Error())
	}

	fmt.Println(mobile.Resultcode)
}
```

```go
package main

import (
	"fmt"
	"github.com/mitchellh/mapstructure"
)

type Family struct {
	LastName string
}
type Location struct {
	City string
}
type Person struct {
	// 用于控制在将 map（通常是 JSON、YAML 等格式的反序列化）转换为 Person 结构体时的行为
	// squash 意味着 Family 中的字段会被“压缩”到 Person 结构体的顶层
	// 意味着可以直接访问 LastName字段，而不需要通过 Family.LastName 来访问
	Family    `mapstructure:",squash"`
	Location  `mapstructure:",squash"`
	FirstName string
}

func main() {
	input := map[string]interface{}{
		"FirstName": "Mitchell",
		"LastName":  "Hashimoto",
		"City":      "San Francisco",
	}

	var result Person
	err := mapstructure.Decode(input, &result)
	if err != nil {
		panic(err)
	}

	fmt.Println(result.FirstName)
	fmt.Println(result.LastName)
	fmt.Println(result.City)
}
```

#### 分文件编写

```go
// study.go
package demo

import "github.com/pkg/errors"

// 断言 study 类型实现了 Study 接口。
// 这个声明没有实际的代码逻辑，主要用于编译时检查。
// 如果 study 没有实现 Study 接口，编译时会报错
var _ Study = (*study)(nil)

// 定义一个接口 Study，声明了四个方法
// 这些方法的参数是字符串类型 msg，返回值也是字符串
// 这个接口规定了所有符合该接口的类型必须实现这些方法
type Study interface {
	Listen(msg string) string
	Speak(msg string) string
	Read(msg string) string
	Write(msg string) string
}

// 定义一个结构体 study，它将实现 Study 接口
type study struct {
	Name string
}

// 实现 Study 接口的 Listen 方法
func (s *study) Listen(msg string) string {
	return s.Name + " 听 " + msg
}

// 实现 Study 接口的 Speak 方法
func (s *study) Speak(msg string) string {
	return s.Name + " 说 " + msg
}

// 实现 Study 接口的 Read 方法
func (s *study) Read(msg string) string {
	return s.Name + " 读 " + msg
}

// 实现 Study 接口的 Write 方法
func (s *study) Write(msg string) string {
	return s.Name + " 写 " + msg
}

// New 函数用于创建一个新的 Study 类型实例
func New(name string) (Study, error) {
	if name == "" {
		return nil, errors.New("name required")
	}

	// 返回一个新的 study 实例，Name 字段设置为传入的 name
	return &study{
		Name: name,
	}, nil
}
```

```go
// main.go
package main

import (
	"fmt"
	"go-hello/demo"
)

func main() {
	name := "Tom"
	s, err := demo.New(name)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(s.Listen("english"))
	fmt.Println(s.Speak("english"))
	fmt.Println(s.Read("english"))
	fmt.Println(s.Write("english"))
}
```

```go
// go.mod
module go-hello

go 1.23.2

require github.com/pkg/errors v0.9.1
```

```bash
doraemon@doraemon-virtual-machine:~/Code/go$ tree
.
├── demo
│   └── study.go
├── go.mod
├── go.sum
└── main.go
```

#### 不定参数

```go
package main

import (
	"fmt"
)

func Add(a int, args ...int) (result int) {
	result += a
	for _, arg := range args {
		result += arg
	}
	return
}

func main() {
	fmt.Println(Add(1, 2, 3, 4))
}
```

#### 时间格式化

```go
package main

import (
	"fmt"
	"time"
)



func main() {
	RFC3339Str := "2020-11-08T08:18:46+08:00"
	ts, _ := time.Parse(time.RFC3339, RFC3339Str)

	// Go 语言格式化时间模板不是常见的 Y-m-d H:i:s，而是 2006-01-02 15:04:05
	ats := ts.Format("2006-01-02 15:04:05")

	fmt.Println(ats)
}
```

