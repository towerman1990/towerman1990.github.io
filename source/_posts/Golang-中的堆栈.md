---
title: Golang 中的堆栈
date: 2021-07-31 14:50:02
tags:
---

### <span id="what-is-queue">什么是堆栈</span>

**堆栈**（英语：stack）又称为**栈**或**堆叠**，是计算机科学中的一种抽象资料类型，只允许在有序的线性资料集合的一端（称为堆栈顶端，英语：top）进行加入数据（英语：push）和移除数据（英语：pop）的运算。因而按照后进先出（LIFO, Last In First Out）的原理运作[<sup>[1]</sup>](#refer-anchor-1)。

### <span id="operate">操作</span>

堆栈使用两种基本操作：推入（压栈，push）和弹出（弹栈，pop）：

- 推入：将资料放入堆栈顶端，堆栈顶端移到新放入的资料。
- 弹出：将堆栈顶端资料移除，堆栈顶端移到移除后的下一笔资料。

<!-- more -->

### <span id="character">特点</span>

堆栈的基本特点：

1. 先入后出，后入先出。
2. 除头尾节点之外，每个元素有一个前驱，一个后继。

### <span id="single-queue-in-golang">Golang 实现堆栈</span>

在 Golang 中使用数组实现一个堆栈，除了需要创建一个数组存储数据外，还要定义最大值（Max）、栈指针（Top），栈指针的初始值为`-1`。

实现代码如下：

```golang
package main

import (
	"errors"
	"fmt"
)

type Stack struct {
	Max int
	Top int
	arr [5]int
}

func (s *Stack) Push(val int) (err error) {
	if s.Top == s.Max-1 {
		fmt.Println("stack full")
		return errors.New("stack full")
	}
	s.Top++
	s.arr[s.Top] = val
	return
}
func (s *Stack) Pop() (val int, err error) {
	if s.Top == -1 {
		return 0, errors.New("stack empty")
	}
	val = s.arr[s.Top]
	s.Top--
	return val, nil
}

func (s *Stack) List() {
	if s.Top == -1 {
		fmt.Println("stack empty")
		return
	}
	for i := s.Top; i >= 0; i-- {
		fmt.Println("val =", s.arr[i])
	}
}

func main() {
	stack := &Stack{
		Max: 5,
		Top: -1,
	}
	for i := 0; i < 6; i++ {
		fmt.Println("push val:", i)
		stack.Push(i)
	}
	stack.List()
	for i := 0; i < 6; i++ {
		res, err := stack.Pop()
		if err != nil {
			fmt.Println(err.Error())
			continue
		}
		fmt.Println("pop val =", res)
	}
}

```

<div id="refer-anchor-1"></div>

- [1] [wiki-堆栈](https://zh.wikipedia.org/wiki/堆栈)