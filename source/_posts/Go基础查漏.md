---
title: Go基础查漏
tags: 学习记录
categories: 
- 学习记录
- Go
cover: https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241448220.png
date: 2022-03-24 19:28:08
updated: 2022-03-24 19:28:44
keywords: Go
---
# 闭包的理解

1. 变量是存在闭包当中，下一次调用的时候，还是会用这个变量的值
2. 当函数`func2()`有变量值的时候,函数的调用时候的变量值是存在于调用的函数栈当中，也就是val值存在`printFunc2()`函数栈中，所以`defer`函数对于val是有影响的。
3. 如果函数没有变量值，val仅限于`func3()`中。

![image-20220324103527599](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241035753.png)

- 1，2
- 101
- 100 

# Goroutine

下面的代码会出现重复打印多次相同的元素。

出现的原因就是在goroutine栈中使用变量的时候，gorutine栈中的变量还没有刷新

```go
func main() {
	data := make(map[int]int, 10)
	for i := 1; i <= 10; i++ {
		data[i] = i
	}
	for key, value := range data {
		go func() {
			fmt.Println("k->", key, "v->", value)
		}()
	}
	time.Sleep(time.Second * 5)
}
```

**解决办法**，传入值，强制刷新变量值。

```go
func main() {
	data := make(map[int]int, 10)
	for i := 1; i <= 10; i++ {
		data[i] = i
	}
	for key, value := range data {
		go func(key,value int) {
			fmt.Println("k->", key, "v->", value)
		}(key,value)
	}
	time.Sleep(time.Second * 5)
}
```

## Goroutine栈

- go-v1(最小栈空间4kB)
- go-v1.2(最小栈空间8kB)
- go-v1.3(分段栈替换为连续栈)
- go-v1.4(最小栈空间2kB)

### 分段栈的Hot split问题：

栈空间的地址是不连续的，它们之间采用指针的方式把他们串联起来。

在之前扩缩容算法：栈空间满了立即触发扩容，不满立即触发缩容。当我们调用一个函数的时候，相应的需要一个栈的扩容，调用完这个函数之后，就会对这个栈进行缩容，如果用for循环调用一个函数，就会频繁的扩缩容，这个过程会产生一个巨大的额外开销，对性能影响比较大

在1.3以前栈有一个扩容的现象的时候，内存地址是不会变化的；1.3及以后的连续栈内存地址会发生变化，主要是复制到一个新分配的栈空间中去，

# defer

defer原理就是一个栈，同时出现两个defer会先运行后出现的defer

```go
func main(){
    m := 10
    defer fmt.Printf("first defer %d\n",m)
    m = 100
    defer fmt.Printf("second defer %d\n",m)
}
```

输出结果自然先second后first

结合闭包：

```go
func main() {
	m := 10
	defer fmt.Printf("first defer %d\n", m)
	m = 100
	defer func() {
		fmt.Printf("second defer %d\n", m)
	}()
	m *= 10
	defer fmt.Printf("third defer %d\n", m)

	funcVal := func1()
	funcVal()
}
func func1() func() {
	fmt.Println("before return")
	return func() {

		defer fmt.Println("in the return")
	}
}
```

![image-20220324154332583](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241543618.png)

```go
func main() {
	defer fmt.Printf("main %d\n", func2())
}
func func2() int {
	sumA := 100
	sumB := 100
	sum := sumA + sumB
	defer func() {
		fmt.Printf("func2 first %d\n", sum)
	}()
	defer fmt.Printf("func2 second %d\n", sum)
	return sum * 10
}

```

sum*10只能作用与main函数中。所以结果变化为

![image-20220324154718510](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241547538.png)

# slice扩容

少于1024双倍扩容，大于1024是1.25倍扩容

```go
func main() {
	var arr1 []int64
	for i := 0; i < 512; i++ {
		arr1 = append(arr1, int64(i))
	}
	fmt.Printf("len=%d,cap=%d\n", len(arr1), cap(arr1))
}
```

![image-20220324155439702](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241554730.png)

```go
func main() {
	var arr1 []int64
	for i := 0; i < 513; i++ {
		arr1 = append(arr1, int64(i))
	}
	fmt.Printf("len=%d,cap=%d\n", len(arr1), cap(arr1))
}

```

![image-20220324155638086](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241556113.png)

```go
func main() {
	var arr1 []int64
	for i := 0; i < 1025; i++ {
		arr1 = append(arr1, int64(i))
	}
	fmt.Printf("len=%d,cap=%d\n", len(arr1), cap(arr1))
}

```

![image-20220324155741278](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241557304.png)

## 批量append和单个append

**查表（sizeclasses.go）：比这个大最接近的数**

```go
// 1024 * 1.25 = 1280
// 一个一个append
// int32 1024 * 1.24* 4 = 5120 查表 5376/4 = 1344
// int64 1024 * 1.24* 8 = 10240 查表 10240/8 = 1280
func main() {
	var arr1 []int32
	for i := 0; i < 1025; i++ {
		arr1 = append(arr1, int32(i))
	}
	fmt.Printf("len=%d,cap=%d\n", len(arr1), cap(arr1))
}
```

![image-20220324160444311](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241604341.png)

```go
// 批量append
// 1025*8 = 8200 查表 9472/8=1184
func main() {
	var arr1 []int64
	var arr2 []int64
	for i := 0; i < 1025; i++ {
		arr1 = append(arr1, int64(i))
	}
	fmt.Printf("len=%d,cap=%d\n", len(arr1), cap(arr1))
	arr2 = append(arr2, arr1...)
	fmt.Printf("len=%d,cap=%d\n", len(arr2), cap(arr2))
}
```

![image-20220324160803975](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241608003.png)

# nil是否一定相等？

非空接口iface，空接口eface。它们除了data都还有一个变量，_type表示data的数据类型

![image-20220324163652777](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241636812.png)

```go
func main() {
	var x *int = nil
	var y interface{} = x
	fmt.Println(x == y)
	fmt.Println(x == nil)
	fmt.Println(y == nil)// 此时的_type不为nil，_type为*int类型

	var z interface{} = nil
	fmt.Println(z == nil)
	fmt.Println(y == z)
}
```

![image-20220324164058113](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241640145.png)

# 并发

Mutex，互斥锁、不可重入锁；在同一个goroutine下，重复上锁的话，就会发生panic

defer是在return之后运行

下面代码会panic

```go
var mu sync.Mutex
var chain string

func main(){
	chain = "main"
    A()
    fmt.Println(chain)
}

func A(){
    mu.Lock()
    defer mu.Unlock()
    char = char + "--> A"
    B()
}

func B(){
    char = char + "--> B"
    C()
}
func C(){
    mu.Lock()
    defer mu.Unlock()
    char = char + "--> C"
}
```

正确应该为：

```go
var mu sysnc.Mutex
func main(){
    mu.Lock()
    go func(){
        mu.Lock()
        fmt.Println("I'm OK")
        defer mu.Unlock()
    }()
    mu.Unlock()
    time.Sleep(time.second)
    return
}
```

# 如何调用panic

![image-20220324191512200](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203241915248.png)

```go
func proc() {
	panic("ok")
}

func main() {
	go func() {
		t := time.NewTicker(time.Second)
		for {
			select {
			case <-t.C:
				go func() {
					defer func() {
						if err := recover(); err != nil {
							fmt.Println(err)
						}
					}()
					fmt.Println(time.Now())
					proc()
				}()
			}
		}
	}()
	select {}
}
```

