---
title: go defer return 顺序
date: 2020-06-02 14:43:01
tags: go
categories: go
---

今天遇到了一道 defer 的题目。题目如下

```go
package main

import "fmt"

func test() (i int) {
    defer func() {
        i++
    }()
    return 1
}

func test2() int {
    i := 1
    defer func() {
        i++
    }()
    return i
}

func main() {
    fmt.Println(test(), test2())
}
```

打印结果为：

2 1

原因：

1. 多个defer的执行顺序为“后进先出”。

2. 所有函数在执行RET返回指令之前，都会先检查是否存在defer语句，若存在则先逆序调用defer语句进行收尾工作再退出返回。

3. 匿名返回值是在return执行时被声明，有名返回值则是在函数声明的同时被声明，因此在defer语句中只能访问有名返回值，而不能直接访问匿名返回值。

4. return其实应该包含前后两个步骤：第一步是给返回值赋值（若为有名返回值则直接赋值，若为匿名返回值则先声明再赋值）；第二步是调用RET返回指令并传入返回值，而RET则会检查defer是否存在，若存在就先逆序插播defer语句，最后RET携带返回值退出函数。

参考链接：<https://my.oschina.net/henrylee2cn/blog/505535>
