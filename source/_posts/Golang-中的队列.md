---
title: Golang 中的队列
date: 2020-09-24 22:27:08
tags:
 - Golang
---

### <span id="what-is-queue">什么是队列</span>

**队列**，又称为伫列（queue），计算机科学中的一种抽象资料型别，是先进先出（FIFO, First-In-First-Out）的线性表。在具体应用中通常用链表或者数组来实现。队列只允许在后端（称为rear）进行插入操作，在前端（称为front）进行删除操作。

队列的操作方式和堆栈类似，唯一的区别在于队列只允许新数据在后端进行添加[<sup>[1]</sup>](#refer-anchor-1)。

### <span id="single-queue-in-golang">Golang 实现单链队列（数组）</span>

在 Golang 中使用数组实现一个单链队列，除了需要创建一个数组存储数据外，还要定义最大值（maxSize）、头指针（front）和尾指针（rear），其中头指针和尾指针的初始值为-1。

当向队列中插入数据时，头指针不变，尾指针+1，最后在数组的尾指针位置赋值。

当从队列中取出数据时，尾指针不变，头指针+1，最后在数组的头指针位置取值。

<!-- more -->

实现代码如下：

```golang
package main

import (
    "errors"
    "fmt"
)

// 队列结构
type Queue struct {
    front   int
    rear    int
    maxSize int
    data    [5]int
}

func main() {
    var queue = &Queue{
        front:   -1,
        rear:    -1,
        maxSize: 5,
    }

    var selectOption int
    var inputValue int
exit:
    for {
        fmt.Print("请选择一种操作：\n")
        fmt.Print("1. 将数据加入队列。\n")
        fmt.Print("2. 从队列中取出数据。\n")
        fmt.Print("3. 显示队列中所有数据。\n")
        fmt.Print("4. 退出。\n")
        fmt.Scanln(&selectOption)
        switch selectOption {
        case 1:
            fmt.Print("输入数据：\n")
            fmt.Scanln(&inputValue)
            err := queue.EnQueue(inputValue)
            if err != nil {
                fmt.Println(err.Error())
            } else {
                fmt.Println(("加入数据成功！\n"))
            }
        case 2:
            val, err := queue.DeQueue()
            if err != nil {
                fmt.Println(err.Error())
            } else {
                fmt.Printf("取出队列值为：%d\n", val)
            }
        case 3:
            queue.LtQueue()
        case 4:
            break exit
        }
    }
    fmt.Println("退出成功！")
}

// 将数据加入队列
func (this *Queue) EnQueue(val int) error {
    if this.rear == this.maxSize-1 {
        return errors.New("队列已满.")
    }
    this.rear++
    this.data[this.rear] = val
    return nil
}

// 从队列中取出数据
func (this *Queue) DeQueue() (int, error) {
    if this.front == this.rear {
        return -1, errors.New("队列为空.")
    }
    this.front++
    return this.data[this.front], nil
}

// 显示队列中的所有数据
func (this *Queue) LtQueue() {
    if this.front == this.rear {
        fmt.Println("队列为空.")
    }

    for i := this.front + 1; i <= this.rear; i++ {
        fmt.Printf("Queue %d = %d\n", i, this.data[i])
    }
}
```

### <span id="loop-queue-in-golang">Golang 实现循环队列（数组）</span>

在 Golang 中使用数组实现一个循环队列，需要的队列结构同单链队列一样，不过头指针和尾指针的初始值为0。

> 一个定义了最大元素数**n**的队列，其存储的实际最大元素数量为**n-1**。一开始我以为是我实现的方式有问题，后来经过各种方案的尝试后，发现这种结构实现循环队列就是这样。

当向队列中插入数据时，头指针不变，在数组的尾指针位置赋值，最后尾指针 +1 并取余。

当从队列中取出数据时，尾指针不变，在数组的头指针位置取值，最后头指针 +1 并取余。

实现代码如下：

```golang
package main

import (
    "errors"
    "fmt"
)

// 队列结构
type Queue struct {
    front   int
    rear    int
    maxSize int
    data    [5]int
}

func main() {
    queue := &Queue{
        front:   0,
        rear:    0,
        maxSize: 5,
    }
    var selectOption int
    var inputValue int

exit:
    for {
        fmt.Print("请选择一种操作：\n")
        fmt.Print("1. 将数据加入队列。\n")
        fmt.Print("2. 从队列中取出数据。\n")
        fmt.Print("3. 显示队列中所有数据。\n")
        fmt.Print("4. 退出。\n")
        fmt.Scanln(&selectOption)
        switch selectOption {
        case 1:
            fmt.Print("输入数据：\n")
            fmt.Scanln(&inputValue)
            err := queue.EnQueue(inputValue)
            if err != nil {
                fmt.Println(err.Error())
            } else {
                fmt.Println(("加入数据成功！"))
            }
        case 2:
            val, err := queue.DeQueue()
            if err != nil {
                fmt.Println(err.Error())
            } else {
                fmt.Printf("取出队列值为：%d\n", val)
            }
        case 3:
            err := queue.LtQueue()
            if err != nil {
                fmt.Println(err.Error())
            }
        case 4:
            break exit
        }
    }
    fmt.Println("退出成功！")
}

// 将数据加入队列
func (this *Queue) EnQueue(val int) error {
    if (this.rear+1)%this.maxSize == this.front {
        return errors.New("队列已满")
    }
    this.data[this.rear] = val
    this.rear = (this.rear + 1) % this.maxSize
    return nil
}

// 从队列中取出数据
func (this *Queue) DeQueue() (val int, error) {
    if this.front == this.rear {
        return -1, errors.New("队列为空")
    }
    val = this.data[this.front]
    this.front = (this.front + 1) % this.maxSize
    return
}

// 显示队列中的所有数据
func (this *Queue) LtQueue() error {
    if this.front == this.rear {
        return errors.New("队列为空")
    }
    for i := 0; i < this.Length(); i++ {
        fmt.Printf("Queue [%d] = %d\n", i, this.data[(this.front+i)%this.maxSize])
    }
    return nil
}

// 队列长度
func (this *Queue) Length() int {
    return (this.maxSize - this.front + this.rear) % this.maxSize
}
```
<div id="refer-anchor-1"></div>

- [1] [wiki-队列](https://zh.wikipedia.org/wiki/队列)