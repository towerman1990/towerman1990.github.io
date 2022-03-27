---
title: Golang 中的稀疏矩阵
date: 2020-09-19 10:14:05
tags:
 - Golang
---

### <span id="what-is-sparse-matrix">什么是稀疏矩阵</span>

**稀疏矩阵**（英语：sparse matrix），在数值分析中，是其元素大部分为零的矩阵。反之，如果大部分元素都非零，则这个矩阵是稠密的。在科学与工程领域中求解线性模型时经常出现大型的稀疏矩阵。

在使用计算机存储和操作稀疏矩阵时，经常需要修改标准算法以利用矩阵的稀疏结构。由于其自身的稀疏特性，通过压缩可以大大节省稀疏矩阵的内存代价。更为重要的是，由于过大的尺寸，标准的算法经常无法操作这些稀疏矩阵[<sup>[1]</sup>](#refer-anchor-1)。

### <span id="sparse-matrix-in-golang">Golang中的稀疏矩阵</span>

在 Golang 中我们可以使用一个二维数组模拟一个稀疏矩阵：

```golang
    var chessMap [8][8]int
    chessMap[1][1] = 1
    chessMap[2][2] = 2
    chessMap[3][3] = 3
```

对其打印效果如下：

```bash
0    0    0    0    0    0    0    0
0    1    0    0    0    0    0    0
0    0    2    0    0    0    0    0
0    0    0    3    0    0    0    0
0    0    0    0    0    0    0    0
0    0    0    0    0    0    0    0
0    0    0    0    0    0    0    0
0    0    0    0    0    0    0    0
```

<!-- more -->

将二维数组中的数据转换为一个稀疏数组：

```golang
    var sparseArr []Node
    node := Node{
        row: 8,
        col: 8,
        val: 0,
    }
    sparseArr = append(sparseArr, node)
    for i, v := range chessMap {
        for j, v2 := range v {
            if v2 != 0 {
                node := Node{
                    row: i,
                    col: j,
                    val: v2,
                }
                sparseArr = append(sparseArr, node)
            }
        }
    }
```

将稀疏数组存储到一个名为“chess_data.txt”文件中：

```golang
    file, err := os.OpenFile("chess_data.txt", os.O_CREATE|os.O_WRONLY, 0666)
    if err != nil {
        fmt.Println("open file failed, err:", err)
        return
    }

    for _, v := range sparseArr {
        file.WriteString(fmt.Sprintf("%d %d %d\n", v.col, v.row, v.val))

    }
    file.Close()
```

重新打开数据存储文件，并将数据恢复至一个稀疏数组中：

```golang
    file, err = os.OpenFile("chess_data.txt", os.O_CREATE|os.O_RDONLY, 0666)
    defer file.Close()
    var sparseArr2 []Node
    reader := bufio.NewReader(file)
    for {
        line, err := reader.ReadString('\n')
        if err == io.EOF {
            if len(line) != 0 {
                fmt.Println(line)
            }
            fmt.Println("read file complete.")
            break
        }
        if err != nil {
            fmt.Println("read file failed. err: ", err)
            return
        }
        lineSlice := strings.Split(strings.Replace(line, "\n", "", -1), " ")
        node := Node{}
        for i, num := range lineSlice {
            n, err := strconv.ParseInt(num, 10, 32)
            if err != nil {
                fmt.Println(err)
            }
            if i == 0 {
                node.col = int(n)
            } else if i == 1 {
                node.row = int(n)
            } else {
                node.val = int(n)
            }
        }
        sparseArr2 = append(sparseArr2, node)
    }

```

将稀疏数组中的数据转成原始数组

```golang
    var chessMap2 [8][8]int
    for i, v := range sparseArr2 {
        if i == 0 {
            continue
        }
        chessMap2[v.col][v.row] = v.val
    }
```

打印恢复后的数据：

```bash
read file complete.
0    0    0    0    0    0    0    0
0    1    0    0    0    0    0    0
0    0    2    0    0    0    0    0
0    0    0    3    0    0    0    0
0    0    0    0    0    0    0    0
0    0    0    0    0    0    0    0
0    0    0    0    0    0    0    0
0    0    0    0    0    0    0    0
```

### <span id="complete-code-and-description">完整代码及说明：</span>

```golang
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
    "strconv"
    "strings"
)

// Node 代表稀疏数组中的一条数据
type Node struct {
    row int // 横坐标
    col int // 纵坐标
    val int // 值
}

func main() {
    var chessMap [8][8]int
    chessMap[1][1] = 1
    chessMap[2][2] = 2
    chessMap[3][3] = 3

    for _, v := range chessMap {
        for _, v2 := range v {
            fmt.Printf("%d\t", v2)
        }
        fmt.Println()
    }

    // 将原始数组转成稀疏数组
    var sparseArr []Node
    node := Node{
        row: 8,
        col: 8,
        val: 0,
    }
    sparseArr = append(sparseArr, node)
    for i, v := range chessMap {
        for j, v2 := range v {
            if v2 != 0 {
                node := Node{
                    row: i,
                    col: j,
                    val: v2,
                }
                sparseArr = append(sparseArr, node)
            }
        }
    }

    // 创建一个可写入文件
    file, err := os.OpenFile("chess_data.txt", os.O_CREATE|os.O_WRONLY, 0666)
    if err != nil {
        fmt.Println("open file failed, err:", err)
        return
    }

    for _, v := range sparseArr {
        // 写入数据
        file.WriteString(fmt.Sprintf("%d %d %d\n", v.col, v.row, v.val))

    }
    file.Close()

    // 打开存储稀疏数组数据的文件
    file, err = os.OpenFile("chess_data.txt", os.O_CREATE|os.O_RDONLY, 0666)
    defer file.Close()
    var sparseArr2 []Node
    reader := bufio.NewReader(file)
    for {
        line, err := reader.ReadString('\n')
        if err == io.EOF {
            if len(line) != 0 {
                fmt.Println(line)
            }
            fmt.Println("read file complete.")
            break
        }
        if err != nil {
            fmt.Println("read file failed. err: ", err)
            return
        }
        // 处理每行数据
        lineSlice := strings.Split(strings.Replace(line, "\n", "", -1), " ")
        node := Node{}
        for i, num := range lineSlice {
            n, err := strconv.ParseInt(num, 10, 32)
            if err != nil {
                fmt.Println(err)
            }
            // 将数据恢复到稀疏数组的节点中
            if i == 0 {
                node.col = int(n)
            } else if i == 1 {
                node.row = int(n)
            } else {
                node.val = int(n)
            }
        }
        sparseArr2 = append(sparseArr2, node)
    }

    // 将稀疏数组转成原始数组
    var chessMap2 [8][8]int
    for i, v := range sparseArr2 {
        // 第一条数据为矩阵规格 需要丢弃
        if i == 0 {
            continue
        }
        chessMap2[v.col][v.row] = v.val
    }

    for _, v := range chessMap2 {
        for _, v2 := range v {
            fmt.Printf("%d\t", v2)
        }
        fmt.Println()
    }
}
```

<div id="refer-anchor-1"></div>

- [1] [wiki-稀疏矩阵](https://zh.wikipedia.org/wiki/稀疏矩阵)