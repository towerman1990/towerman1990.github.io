---
title: Golang 中的链表
date: 2021-05-23 17:47:42
tags:
 - Golang
---

### <span id="what-is-queue">什么是链表</span>

在计算机科学中，**链表**（Linked list）是一种常见的基础数据结构，是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的指针(Pointer)[<sup>[1]</sup>](#refer-anchor-1)。

### <span id="single-queue-in-golang">Golang 实现单向链表</span>

<!-- more -->

实现代码如下：

```golang
package main

import "fmt"

const COUNT = 5

type LinkNode struct {
	No   int
	next *LinkNode
}

// 依序向链表中插入结点
func InsertNode(head *LinkNode, insertNode *LinkNode) {
	for {
		if head.next == nil {
			head.next = insertNode
			fmt.Printf("Insert node %d in tail.\n", head.next.No)
			return
		}
		// 不想顺序插入可以去掉这条判断
		if head.next.No > insertNode.No {
			insertNode.next = head.next
			head.next = insertNode
			fmt.Printf("Insert node %d in middle.\n", head.next.No)
			return
		}
		// Golang中没有引用传递 head只是实参拷贝的值 所以可以用它做查询数据时的“指针”
		head = head.next
	}
}

// 删除链表中指定结点
func DeleteNode(head *LinkNode, no int) {
	if no <= 0 {
		fmt.Printf("Invalid node %d.\n", no)
		return
	}
	for {
		if head.next == nil {
			fmt.Printf("Not found node %d.\n", no)
			return
		}
		if head.next.No == no {
			head.next = head.next.next
			fmt.Printf("Delete node %d.\n", no)
			return
		}
		head = head.next
	}
}

// 输出链表所有结点
func ListNode(head *LinkNode) {
	for {
		fmt.Printf("List node number: %v\n", head.No)
		if head.next == nil {
			fmt.Println("List is empty.")
			return
		}
		head = head.next
	}
}

func main() {
	linkNodeMap := make(map[int]*LinkNode, COUNT)
	for i := 1; i <= COUNT; i++ {
		linkNodeMap[i] = &LinkNode{No: i}
	}
	headNode := &LinkNode{}
	// 利用map的遍历顺序是随机的特性 进行数据的随机插入
	for _, node := range linkNodeMap {
		InsertNode(headNode, node)
	}
	ListNode(headNode)
	DeleteNode(headNode, 1)
	DeleteNode(headNode, 3)
	DeleteNode(headNode, 6)
	ListNode(headNode)
}
```

程序执行结果：
```bash
Insert node 1 in tail.
Insert node 2 in tail.
Insert node 3 in tail.
Insert node 4 in tail.
Insert node 5 in tail.
List node number: 0
List node number: 1
List node number: 2
List node number: 3
List node number: 4
List node number: 5
List is empty.
Delete node 1.
Delete node 3.
Not found node 6.
List node number: 0
List node number: 2
List node number: 4
List node number: 5
List is empty.
```

### <span id="single-queue-in-golang">Golang 实现双向链表</span>

实现代码如下：

```golang
package main

import "fmt"

const COUNT = 5

type LinkNode struct {
	No       int
	previous *LinkNode
	next     *LinkNode
}

// 依序向链表中插入结点
func InsertNode(head *LinkNode, insertNode *LinkNode) {
	for {
		if head.next == nil {
			head.next = insertNode
			insertNode.previous = head
			fmt.Printf("Insert node %d in tail.\n", head.next.No)
			return
		}
		// 不想顺序插入可以去掉这条判断
		if head.next.No > insertNode.No {
			insertNode.next = head.next
			insertNode.previous = head
			head.next.previous = insertNode
			head.next = insertNode
			fmt.Printf("Insert node %d in middle.\n", head.next.No)
			return
		}
		// Golang中没有引用传递 head只是实参拷贝的值 所以可以用它做查询数据时的“指针”
		head = head.next
	}
}

// 删除链表中指定结点
func DeleteNode(head *LinkNode, no int) {
	if no <= 0 {
		fmt.Printf("Invalid node %d.\n", no)
		return
	}
	for {
		if head.next == nil {
			fmt.Printf("Not found node %d.\n", no)
			return
		}
		if head.next.No == no {
			head.next = head.next.next
			head.next.previous = head
			fmt.Printf("Delete node %d.\n", no)
			return
		}
		head = head.next
	}
}

// 输出链表所有结点
func ListNode(head *LinkNode) {
	for {
		fmt.Printf("List node number: %v\n", head.No)
		if head.next == nil {
			fmt.Println("List is empty.")
			return
		}
		head = head.next
	}
}

func main() {
	linkNodeMap := make(map[int]*LinkNode, COUNT)
	for i := 1; i <= COUNT; i++ {
		linkNodeMap[i] = &LinkNode{No: i}
	}

	headNode := &LinkNode{}
	// 利用map的遍历顺序是随机的特性 进行数据的随机插入
	for _, node := range linkNodeMap {
		InsertNode(headNode, node)
	}
	ListNode(headNode)
	DeleteNode(headNode, 1)
	DeleteNode(headNode, 3)
	DeleteNode(headNode, 6)
	ListNode(headNode)
}

```

程序执行结果：
```bash
Insert node 4 in tail.
Insert node 5 in tail.
Insert node 1 in middle.
Insert node 2 in middle.
Insert node 3 in middle.
List node number: 0
List node number: 1
List node number: 2
List node number: 3
List node number: 4
List node number: 5
List is empty.
Delete node 1.
Delete node 3.
Not found node 6.
List node number: 0
List node number: 2
List node number: 4
List node number: 5
List is empty.
```

### <span id="single-queue-in-golang">Golang 实现单向循环链表</span>

实现代码如下：

```golang
package main

import "fmt"

const COUNT = 10

type LinkNode struct {
	No   int
	next *LinkNode
}

// 依序向链表中插入结点
func InsertNode(head *LinkNode, insertNode *LinkNode) {
	mark := head
	for {
		// 第一个结点
		if head.next == nil {
			insertNode.next = head
			head.next = insertNode
			fmt.Printf("Insert first node %d.\n", head.next.No)
			return
		}
		// 不想顺序插入可以去掉这条判断
		if head.next.No > insertNode.No {
			insertNode.next = head.next
			head.next = insertNode
			fmt.Printf("Insert node %d in middle.\n", head.next.No)
			return
		}
		// 插入尾部
		if head.next == mark {
			insertNode.next = head.next
			head.next = insertNode
			fmt.Printf("Insert node %d in tail.\n", head.next.No)
			return
		}
		// Golang中没有引用传递 head只是实参拷贝的值 所以可以用它做查询数据时的“指针”
		head = head.next
	}
}

// 删除链表中指定结点
func DeleteNode(head *LinkNode, no int) *LinkNode {
	if no <= 0 {
		fmt.Printf("Invalid node %d.\n", no)
		return head
	}
	mark := head
	for {
		if head.next == mark {
			// 如果删除头结点 需要将新的头结点返回
			if head.next.No == no {
				head.next = head.next.next
				fmt.Printf("Delete node %d.\n", no)
				return head.next
			}
			fmt.Printf("Not found node %d.\n", no)
			return head.next
		}
		if head.next.No == no {
			head.next = head.next.next
			fmt.Printf("Delete node %d.\n", no)
			return mark
		}
		head = head.next
	}
}

// 输出链表所有结点
func ListNode(head *LinkNode) {
	mark := head
	for {
		fmt.Printf("List node number: %v\n", head.No)
		if head.next == mark {
			fmt.Println("List is empty.")
			return
		}
		head = head.next
	}
}

func main() {
	linkNodeMap := make(map[int]*LinkNode, COUNT)
	for i := 1; i <= COUNT; i++ {
		linkNodeMap[i] = &LinkNode{No: i}
	}
	headNode := &LinkNode{}
	// 利用map的遍历顺序是随机的特性 进行数据的随机插入
	for _, node := range linkNodeMap {
		InsertNode(headNode, node)
	}
	ListNode(headNode)
	headNode = DeleteNode(headNode, 0)
	headNode = DeleteNode(headNode, 3)
	headNode = DeleteNode(headNode, 6)
	headNode = DeleteNode(headNode, 11)
	ListNode(headNode)
}
```

程序执行结果：
```bash
Insert first node 1.
Insert node 2 in tail.
Insert node 3 in tail.
Insert node 4 in tail.
Insert node 5 in tail.
List node number: 0
List node number: 1
List node number: 2
List node number: 3
List node number: 4
List node number: 5
List is empty.
Invalid node 0.
Delete node 3.
Not found node 6.
Not found node 11.
List node number: 0
List node number: 1
List node number: 2
List node number: 4
List node number: 5
List is empty.
```

### <span id="single-queue-in-golang">使用链表解决约瑟夫问题</span>

实现代码如下：

```golang
package main

import (
	"fmt"
)

type LinkNode struct {
	No   int
	next *LinkNode
}

// 创建包含指定数量结点链表
func CreateList(count int) *LinkNode {
	head := &LinkNode{No: 1}
	if count <= 1 {
		head.next = head
		return head
	}
	list := head
	for i := 2; i <= count; i++ {
		list.next = &LinkNode{No: i}
		list = list.next
	}
	list.next = head
	return head
}

// 输出链表所有结点
func ListNode(head *LinkNode) {
	mark := head
	for {
		fmt.Printf("List node number: %v\n", head.No)
		if head.next == mark {
			fmt.Println("List is empty.")
			return
		}
		head = head.next
	}
}

// 循环删除指定报数结点
func DeleteNode(head *LinkNode, no int) {
	tail := head
	for {
		if head.next == tail {
			tail = head
			head = tail.next
			break
		}
		head = head.next
	}
	start := 1
	for {
		if start == no {
			if head.next == head {
				fmt.Printf("Latest node %d.\n", head.No)
				return
			}
			start = 1
			fmt.Printf("Delete node %d.\n", head.No)
			head = head.next
			tail.next = head
			continue
		}
		start++
		head = head.next
		tail = tail.next
	}
}

func main() {
	list := CreateList(10)
	ListNode(list)
	DeleteNode(list, 3)
}
```

程序执行结果：
```bash
List node number: 1
List node number: 2
List node number: 3
List node number: 4
List node number: 5
List node number: 6
List node number: 7
List node number: 8
List node number: 9
List node number: 10
List is empty.
Delete node 3.
Delete node 6.
Delete node 9.
Delete node 2.
Delete node 7.
Delete node 1.
Delete node 8.
Delete node 5.
Delete node 10.
Latest node 4.
```

<div id="refer-anchor-1"></div>

- [1] [wiki-队列](https://zh.wikipedia.org/wiki/链表)