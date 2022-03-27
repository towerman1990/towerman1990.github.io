---
title: Golang 中defer修改返回值问题
date: 2020-10-18 20:56:14
tags:
 - Golang
---
### <span id="what-is-defer">什么是defer</span>

在 Golang 中 `defer` 是一个关键字，它用来实现 `延迟调用函数`：当 `defer` 语句被执行时，跟在 `defer` 后面的函数会被延迟执行。直到包含该 `defer` 语句的函数执行完毕时，`defer` 后的函数才会被执行，不论包含defer语句的函数是通过 `return` 正常结束，还是由于 `panic` 导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反[<sup>[1]</sup>](#refer-anchor-1)。

既然 `defer` 之后的函数是在“主函数”（这个主函数是先对于 `defer` 定义的延迟调用函数而言的）执行完毕后执行，那就可能出现这样一种情况：“主函数”在执行完成之后生成了返回值，延迟调用函数继续执行，假如此时延迟调用函数修改了返回值，会发生什么？

<!-- more -->

### <span id="what-is-queue">Deferred函数执行顺序</span>

在解决这个问题之前，我们先要知道Deferred函数的执行顺序：

```golang
package main

import "fmt"

func main() {
    example()
    fmt.Println("main function executed.")
}

func example() {
    defer func() {
        fmt.Println("first")
    }()

    defer func() {
        fmt.Println("second")
    }()

    defer func() {
        fmt.Println("third")
    }()
    fmt.Println("example function executed.")
}
```

执行结果：

```bash
example function executed.
third
second
first
main function executed.
```

可以看到，如果有多个延迟函数，它们会被存储在一个 `栈` 中，因此，最后被 `defer` 修饰的函数会在函数体返回之后先执行。

### <span id="what-is-queue">延迟调用函数修改返回值</span>

延迟调用函数修改返回值会有这么几种情况：

1. 返回值列表声明了返回值名称和类型

```golang
package main

import "fmt"

func main() {
    fmt.Println(example())
}

func example() (i int) {
    defer func() {
        i++
    }()
    defer func() {
        i++
    }()
    return i
}
```

执行结果：

```bash
2
```

可以看到，延迟调用函数成功的两次修改返回变量 `i`。

2. 返回值列表只定义了类型

```golang
package main

import "fmt"

func main() {
    fmt.Println(example())
}

func example() int {
    i := 0
    defer func() {
        i++
    }()
    defer func() {
        i++
    }()
    return i
}
```

执行结果：

```bash
0
```

我们修改了返回变量`i`的值，但函数的返回值并未被修改。

从上面的案例可以看到，只有当返回值名称被声明了，延迟调用函数才可以成功的修改返回值。但假如返回的是一个指针呢？

```golang
package main

import "fmt"

func main() {
    fmt.Println(example())
    fmt.Println(*example2())
}

func example() int {
    i := 0
    defer func(i *int) {
        *i++
    }(&i)
    defer func(i *int) {
        *i++
    }(&i)
    return i
}

func example2() *int {
    i := 0
    defer func() {
        i++
    }()
    defer func() {
        i++
    }()
    return &i
}
```

执行结果：

```bash
0
2
```

最后的返回结果似乎和我们想象的不一样。事实上，`example2()` 函数的 `返回值` 并未被修改，因为它的返回值本身就是一个指针，只是指针指向的变量值变了，但指针没有变化。

我个人觉得对于 `defer` 一种比较好的理解方式，就是假设 Golang 的函数需有一个“真正”的返回值变量。如果在函数的返回值列表中做了声明，那么这个“真正”的返回值变量就等同于我们定义的返回值变量，一切对于返回值的修改都是“有效”的；如果没有声明，那么 Golang 会自动创建一个“匿名”的返回值变量，在“主函数”执行完成后，将返回值赋值给“匿名”的返回值变量，此时再对返回值变量修改就是无效的了，因为已经不会再次赋值。

<div id="refer-anchor-1"></div>

- [1] [5.8. Deferred函数](http://books.studygolang.com/gopl-zh/ch5/ch5-08.html)