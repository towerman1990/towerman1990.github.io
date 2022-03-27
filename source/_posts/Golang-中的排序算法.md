---
title: Golang 中的排序算法
date: 2021-06-19 15:10:20
tags:
---

### <span id="what-is-sorting-algorithm">什么是排序</span>

在计算机科学与数学中，一个**排序算法**（英语：Sorting algorithm）是一种能将一串资料依照特定排序方式进行排列的一种算法[<sup>[1]</sup>](#refer-anchor-1)。本文中将使用Golang实现几种常见的排序算法。

### <span id="bubble_sort">Golang 实现冒泡排序</span>

**冒泡排序**（英语：Bubble Sort）又称为泡式排序，是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端[<sup>[2]</sup>](#refer-anchor-2)。

<!-- more -->

实现代码如下：

```golang
package main

import "fmt"

// 冒泡排序
func BubbleSort(arr []int) {
	// 最后一次排序只剩一个元素，故只需len(arr)-1个元素进行比较排序。
	for i := 0; i < len(arr)-1; i++ {
		// 最后一次比较第j+1个元素为最后一个元素，故只需len(arr)-i-1次比较。
		for j := 0; j < len(arr)-i-1; j++ {
			// 第j个元素与第j+1个元素的值作比较，若大于则交换位置。
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
			}
		}
	}
}

func main() {
	// 定义一个无序数组
	arr := [...]int{10, -2, -9, 0, -1, -7, 3, 8, -4, 5}
	// 打印原数组
	fmt.Println("原数组：", arr)
	// 进行排序
	BubbleSort(arr[:])
	// 打印排序后的数组
	fmt.Println("排序后：", arr)
}
```

程序执行结果：
```bash
原数组： [10 -2 -9 0 -1 -7 3 8 -4 5]
排序后： [-9 -7 -4 -2 -1 0 3 5 8 10]
```

### <span id="selection_sort">Golang 实现选择排序</span>

**选择排序**（Selection Sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。[<sup>[3]</sup>](#refer-anchor-3)。

实现代码如下：

```golang
package main

import "fmt"

// 选择排序
func SelectionSort(arr []int) {
	// 最后一次查找只剩一个元素，故只需len(arr)-1个元素进行比较排序。
	for i := 0; i < len(arr)-1; i++ {
		// 假设第i个元素为最小值
		min := i
		// 查找剩余元素
		for j := i + 1; j < len(arr); j++ {
			if arr[j] < arr[min] {
				min = j
			}
		}
		// 对新找到的“最小元素”排序。
		arr[i], arr[min] = arr[min], arr[i]
	}
}

func main() {
	// 定义一个无序数组
	arr := [...]int{10, -2, -9, 0, -1, -7, 3, 8, -4, 5}
	// 打印原数组
	fmt.Println("原数组：", arr)
	// 进行排序
	SelectionSort(arr[:])
	// 打印排序后的数组
	fmt.Println("排序后：", arr)
}
```

程序执行结果：
```bash
原数组： [10 -2 -9 0 -1 -7 3 8 -4 5]
排序后： [-9 -7 -4 -2 -1 0 3 5 8 10]
```


### <span id="insertion_sort">Golang 实现插入排序</span>

**插入排序**（英语：Insertion Sort）是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通常采用in-place排序（即只需用到 ***O*** **(1)** 的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间[<sup>[4]</sup>](#refer-anchor-4)。

实现代码如下：

```golang
package main

import "fmt"

// 插入排序
func InsertionSort(arr []int) {
	// 第一个元素默认已排序，故只需len(arr)-1次排序。
	for i := 0; i < len(arr)-1; i++ {
		j := i + 1
		// 只要插入元素比排序元素小，就不断向前“插入”，直到找到比自己小的元素或到达数组头部。
		for j > 0 && arr[j] < arr[j-1] {
			// 通过元素交换的形式让元素向前“插入”
			arr[j-1], arr[j] = arr[j], arr[j-1]
			j--
		}
	}
}

func main() {
	// 定义一个无序数组
	arr := [...]int{10, -2, -9, 0, -1, -7, 3, 8, -4, 5}
	// 打印排原数组
	fmt.Println("原数组：", arr)
	// 进行排序
	InsertionSort(arr[:])
	// 打印排序后的数组
	fmt.Println("排序后：", arr)
}
```

程序执行结果：
```bash
原数组： [10 -2 -9 0 -1 -7 3 8 -4 5]
排序后： [-9 -7 -4 -2 -1 0 3 5 8 10]
```

### <span id="insertion_sort">Golang 实现快速排序</span>

**快速排序**（英语：Quicksort），又称**分区交换排序**（partition-exchange sort），简称**快排**，一种排序算法，最早由东尼·霍尔提出。[<sup>[5]</sup>](#refer-anchor-5)。

快速排序使用**分治法**（Divide and conquer）策略来把一个**序列**（list）分为较小和较大的2个子序列，然后递归地排序两个子序列。

步骤为：

- 挑选基准值：从数列中挑出一个元素，称为“基准”（pivot），
- 分割：重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（与基准值相等的数可以到任何一边）。在这个分割结束之后，对基准值的排序就已经完成，
- 递归排序子序列：递归地将小于基准值元素的子序列和大于基准值元素的子序列排序。

递归到最底部的判断条件是数列的大小是零或一，此时该数列显然已经有序。

实现代码如下：

```golang
package main

import "fmt"

func QuickSort(arr []int, left int, right int) {
	if left >= right {
		return
	}
	// 找到基准元素的位置，然后做左右划分
	pivot := Partition(arr, left, right)
	QuickSort(arr, left, pivot-1)
	QuickSort(arr, pivot+1, right)
}

func Partition(arr []int, left int, right int) int {
	pivot := left
	for left < right {
		// 找到大于等于基准元素的元素
		for left < right && arr[left] < arr[pivot] {
			left++
		}
		// 找到小于基准元素的元素
		for left < right && arr[right] >= arr[pivot] {
			right--
		}
		// 交换元素
		if left < right {
			arr[left], arr[right] = arr[right], arr[left]
		}
	}
	// 将基准元素放入基准位置
	arr[left], arr[pivot] = arr[pivot], arr[left]
	return left
}

func main() {
	// 定义一个无序数组
	arr := [...]int{10, -2, -9, 0, -1, -7, 3, 8, -4, 5}
	// 打印排原数组
	fmt.Println("原数组：", arr)
	// 进行排序
	QuickSort(arr[:], 0, len(arr)-1)
	// 打印排序后的数组
	fmt.Println("排序后：", arr)
}
```

程序执行结果：
```bash
原数组： [10 -2 -9 0 -1 -7 3 8 -4 5]
排序后： [-9 -7 -4 -2 -1 0 3 5 8 10]
```

也可以使用一个限定内的随机元素作为基准元素：

```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func QuickSort(arr []int, left int, right int) {
	if left >= right {
		return
	}
	// 找到基准元素的位置，同时做左右划分
	pivot := Partition(arr, left, right)
	QuickSort(arr, left, pivot-1)
	QuickSort(arr, pivot+1, right)
}

func Partition(arr []int, left int, right int) int {
	rand.Seed(time.Now().UnixNano())
	// 生成随机的基准元素
	pivot := rand.Intn(right-left) + left
	for left < right {
		// 找到大于等于基准元素的元素
		for left < right && arr[left] < arr[pivot] {
			left++
		}
		// 找到小于基准元素的元素
		for left < right && arr[right] >= arr[pivot] {
			right--
		}
		// 交换元素
		if left < right {
			// 如果交换元素为基准元素，需要同时移动基准元素的位置
			if left == pivot {
				pivot = right
			} else if right == pivot {
				pivot = left
			}
			arr[left], arr[right] = arr[right], arr[left]
		}
	}
	arr[left], arr[pivot] = arr[pivot], arr[left]
	return left
}

func main() {
	// 定义一个无序数组
	arr := [...]int{10, -2, -9, 0, -1, -7, 3, 8, -4, -12, 5, 22, -4, -2, 10}
	// 打印排原数组
	fmt.Println("原数组：", arr)
	// 进行排序
	QuickSort(arr[:], 0, len(arr)-1)
	// 打印排序后的数组
	fmt.Println("排序后：", arr)
}
```

程序执行结果：
```bash
原数组： [10 -2 -9 0 -1 -7 3 8 -4 -12 5 22 -4 -2 10]
排序后： [-12 -9 -7 -4 -4 -2 -2 -1 0 3 5 8 10 10 22]
```

<div id="refer-anchor-1"></div>

- [1] [wiki-排序算法](https://zh.wikipedia.org/wiki/排序算法)

<div id="refer-anchor-2"></div>

- [2] [wiki-冒泡排序](https://zh.wikipedia.org/wiki/冒泡排序)

<div id="refer-anchor-3"></div>

- [3] [wiki-选择排序](https://zh.wikipedia.org/wiki/选择排序)

<div id="refer-anchor-4"></div>

- [4] [wiki-插入排序](https://zh.wikipedia.org/wiki/插入排序)

<div id="refer-anchor-5"></div>

- [5] [wiki-插入排序](https://zh.wikipedia.org/wiki/快速排序)