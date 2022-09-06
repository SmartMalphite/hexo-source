---
title: Go的字符串:编码与操作
date: 2022-07-24 21:00:49
categories: 
    - Go学习笔记
tags: 
    - 字符串
    - 编解码
---

## Go语言中字符串的编码:
Go 语言在看待 Go 字符串组成这个问题上，有两种视角。

一种是**字节视角**，也就是和所有其它支持字符串的主流语言一样，Go 语言中的字符串值也是一个可空的字节序列，字节序列中的字节个数称为该字符串的长度。一个个的字节只是孤立数据，不表意

如果要表意，我们就需要从字符串的另外一个视角来看，也就是字符串是由一个可空的字符序列构成(即**字符视角**)。

```go
var s = "中国人"

// 字节视角
fmt.Printf("the length of s = %d\n", len(s)) // 9
for i := 0; i < len(s); i++ {
  fmt.Printf("0x%x ", s[i]) // 0xe4 0xb8 0xad 0xe5 0x9b 0xbd 0xe4 0xba 0xba
}


// 字符视角
fmt.Println("the character count in s is", utf8.RuneCountInString(s)) // 3
for _, c := range s {
  fmt.Printf("0x%x ", c) // 0x4e2d 0x56fd 0x4eba
}
```

### Unicode
Go 采用的是 Unicode 字符集，每个字符都是一个 Unicode 字符，上述例子中字符视角下输出的 `0x4e2d`、`0x56fd` 和 `0x4eba` 就是 Unicode 字符的表示。以 `0x4e2d` 为例，它是汉字“中”在 Unicode 字符集表中的码点（Code Point）。Unicode字符集为绝大部分语言的字符提供了统一的编码集。

Unicode 字符集中的每个字符，都被分配了统一且唯一的字符编号。所谓 Unicode 码点，就是指将 Unicode 字符集中的所有字符“排成一队”，字符在这个“队伍”中的位次，就是它在 Unicode 字符集中的码点。也就说，一个码点唯一对应一个字符。

### UTF-8
UTF-8 编码解决的是 Unicode 码点值在计算机中如何存储的问题。那么为什么不直接用unicode码点存储呢? 主要有如下几个问题:
* 由于Unicode采用的是4个字节的固定长度编码，与ASCII无法兼容。
* 浪费存储空间

UTF-8方案使用了变长设计，对Unicode的码点进行编码。其长度从1到4不等，并且前128个与ASCII兼容(即内存中如`A` `B` `C` 之类的字符时， ASCII字符编码可以被当做UTF-8编码直接使用)
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220724230323.png)

### Go语言如何表示一个码点
Go 使用 rune 这个类型来表示一个 Unicode 码点。rune 本质上是 int32 类型的别名类型，它与 int32 类型是完全等价的

由于一个 Unicode 码点唯一对应一个 Unicode 字符。所以我们可以说，一个 rune 实例就是一个 Unicode 字符，一个 Go 字符串也可以被视为 rune 实例的集合。我们可以通过字符字面值来初始化一个 rune 变量。

那么现在我们就使用 Go 在标准库中提供的 UTF-8 包，对 Unicode 字符（rune）进行编解码试试看：
```go

// rune -> []byte                                                                            
func encodeRune() {                                                                          
    var r rune = 0x4E2D                                                                      
    fmt.Printf("the unicode charactor is %c\n", r) // 中                                     
    buf := make([]byte, 3)                                                                   
    _ = utf8.EncodeRune(buf, r) // 对rune进行utf-8编码                                                           
    fmt.Printf("utf-8 representation is 0x%X\n", buf) // 0xE4B8AD                            
}                                                                                            
                                                                                             
// []byte -> rune                                                                            
func decodeRune() {                                                                          
    var buf = []byte{0xE4, 0xB8, 0xAD}                                                       
    r, _ := utf8.DecodeRune(buf) // 对buf进行utf-8解码
    fmt.Printf("the unicode charactor after decoding [0xE4, 0xB8, 0xAD] is %s\n", string(r)) // 中
}
```

### Go字符串类型的底层实现
```go

// $GOROOT/src/reflect/value.go

// StringHeader是一个string的运行时表示
type StringHeader struct {
    Data uintptr
    Len  int
}
```
从源码中和下面的图中可以看出string类型并不是单纯的"字符的数组"，而是有一个指向数据数据的指针和长度字段组成的结构体。
这也就很好的解释了string类型获取长度的时间复杂度是常数的原因。

![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220724230920.png)

## 常用字符串操作

### 下标操作
在字符串的实现中，真正存储数据的是底层的数组。字符串的下标操作本质上等价于底层数组的下标操作

```go
var s = "中国人"
fmt.Printf("0x%x\n", s[0]) // 0xe4：字符“中” utf-8编码的第一个字节
```
我们可以看到，通过下标操作，我们获取的是**字符串中特定下标上的字节，而不是字符。**

### 迭代操作
Go 有两种迭代形式：常规 for 迭代与 for range 迭代。需要注意的是，通过这两种形式的迭代对字符串进行操作得到的结果是不同的。
通过常规 for 迭代对字符串进行的操作是一种字节视角的迭代，而通过 for range 迭代，我们每轮迭代得到的是字符串中 Unicode 字符的码点值，以及该字符在字符串中的偏移值。

```go
var s = "中国人"

// for 迭代
for i := 0; i < len(s); i++ {
  fmt.Printf("index: %d, value: 0x%x\n", i, s[i])
}

// for range迭代
for i, v := range s {
    fmt.Printf("index: %d, value: 0x%x\n", i, v)
}
```
运行一下这段代码，我们得到:

```go
// for 迭代
index: 0, value: 0xe4
index: 1, value: 0xb8
index: 2, value: 0xad
index: 3, value: 0xe5
index: 4, value: 0x9b
index: 5, value: 0xbd
index: 6, value: 0xe4
index: 7, value: 0xba
index: 8, value: 0xba

// for range迭代
index: 0, value: 0x4e2d
index: 3, value: 0x56fd
index: 6, value: 0x4eba
```

### 字符串连接
Go 原生支持通过 +/+= 操作符进行字符串连接

```go
s := "Rob Pike, "
s = s + "Robert Griesemer, "
s += " Ken Thompson"

fmt.Println(s) // Rob Pike, Robert Griesemer, Ken Thompson
```
虽然通过 +/+= 进行字符串连接的开发体验是最好的, 但性能略差，Go 还提供了 strings.Builder、strings.Join、fmt.Sprintf 等函数来进行字符串连接操作以更好的解决性能问题

### 字符串比较
Go 字符串类型支持各种比较关系操作符，包括 = =、!= 、>=、<=、> 和 <。在字符串的比较上，Go 采用字典序的比较策略，分别从每个字符串的起始处，开始逐个字节地对两个字符串类型变量进行比较。

```go
func main() {
        // ==
        s1 := "世界和平"
        s2 := "世界" + "和平"
        fmt.Println(s1 == s2) // true

        // !=
        s1 = "Go"
        s2 = "C"
        fmt.Println(s1 != s2) // true

        // < and <=
        s1 = "12345"
        s2 = "23456"
        fmt.Println(s1 < s2)  // true
        fmt.Println(s1 <= s2) // true

        // > and >=
        s1 = "12345"
        s2 = "123"
        fmt.Println(s1 > s2)  // true
        fmt.Println(s1 >= s2) // true
}
```

### 字符串转换
Go 支持字符串与字节切片、字符串与 rune 切片的双向转换，并且这种转换无需调用任何函数，只需使用显式类型转换即可

```go
var s string = "中国人"
                      
// string -> []rune
rs := []rune(s) 
fmt.Printf("%x\n", rs) // [4e2d 56fd 4eba]
                
// string -> []byte
bs := []byte(s) 
fmt.Printf("%x\n", bs) // e4b8ade59bbde4baba
                
// []rune -> string
s1 := string(rs)
fmt.Println(s1) // 中国人
                
// []byte -> string
s2 := string(bs)
fmt.Println(s2) // 中国人
```

## 扩展内容
### UTF-8采用不定长的设计，如何辨别几个字节为一个字符?

| unicode 符号范围 | utf-8 编码方式 |
| ----------- | ----------- |
| 00000000 ~ 0000007F | 0xxxxxxx | 
| 00000080 ~ 000007FF | 110xxxxx 10xxxxxx | 
| 00000800 ~ 0000FFFF | 1110xxxx 10xxxxxx 10xxxxxx | 
| 00010000 ~ 0010FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx | 

总结下来，针对UTF8，编码规则其实只有两条：
1. 单字节规则： 对于 单字节 的符号，字节的第一位（最高位）设为 0，后面 7 位为这个符号的 unicode 码。
2. n字节规则： 对于 n 字节的符号（n>1），第一个字节的前 n 位都设为 1，第 n+1 位设为 0，后面字节的前两位一律设为 10。剩下的没有提及的二进制位，全部为这个符号的 unicode 码。