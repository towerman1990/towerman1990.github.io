---
title: Golang 中使用 Mockey 和 GoConvey 编写单元测试
date: 2024-05-12 10:30:28
tags:
 - Golang
 - 单元测试
---

在 Go 项目中，一个业务方法往往会调用数据库、缓存或者其他服务。如果每次测试都依赖真实环境，不但准备数据麻烦，也很难稳定地复现超时、查询失败等情况。

单元测试可以把这些依赖替换成可控制的实现，让我们专注验证业务逻辑。本文通过一个查询用户资料的例子，介绍如何使用字节跳动的 [Mockey](https://github.com/bytedance/mockey) 和 [GoConvey](https://github.com/smartystreets/goconvey) 编写单元测试。

两者的职责不同：**Mockey 负责打桩，GoConvey 负责组织测试场景和执行断言**。测试入口和执行命令仍然使用 Go 自带的 `testing` 和 `go test`。

<!-- more -->

### 单元测试应该测试什么

假设我们要测试一个根据用户 ID 获取昵称的方法，通常需要考虑以下情况：

| 场景 | 期望行为 |
| --- | --- |
| ID 小于或等于 0 | 返回参数错误，不查询数据 |
| 查询依赖返回错误 | 返回错误，并保留原始错误供调用方判断 |
| 查询结果为空 | 返回用户不存在 |
| 用户昵称为空或只有空白 | 返回默认昵称 |
| 正常查询 | 返回去除首尾空白后的昵称 |

测试不应该只验证“执行时没有报错”，还要验证返回值、错误类型以及必要的依赖调用行为。比如参数非法时，即使最终返回了参数错误，如果仍然访问数据库，也可能意味着实现存在问题。

对于纯计算逻辑，直接使用真实实现即可。对于数据库、RPC 等外部依赖，可以在边界上打桩。真正的数据库查询语句、网络协议和连接配置，则需要集成测试单独验证。

### 创建示例项目

在博客仓库之外创建一个独立的 Go 示例目录，然后执行：

```shell
mkdir go-unit-demo
cd go-unit-demo
go mod init example.com/go-unit-demo
go get github.com/bytedance/mockey@v1.4.8
go get github.com/smartystreets/goconvey@v1.8.1
```

本文完整示例已在 Windows/amd64、Go 1.27.1、Mockey 1.4.8 和 GoConvey 1.8.1 的环境中运行通过，5 个场景共 17 条断言。

上面的命令会解析依赖并记录到 `go.mod` 和 `go.sum`。团队项目应提交这两个文件，固定实际验证过的依赖版本，同时统一本地和 CI 的 Go 工具链版本。

项目结构如下：

```text
go-unit-demo/
├── go.mod
├── go.sum
└── user/
    ├── service.go
    └── service_test.go
```

按照 [Go testing 的约定](https://pkg.go.dev/testing)，测试文件以 `_test.go` 结尾，测试函数采用 `func TestXxx(t *testing.T)` 的形式。本文测试与被测代码都放在 `user` 包中，因此可以直接访问未导出的函数。

### 准备业务代码

创建 `user/service.go`：

```go
package user

import (
    "context"
    "errors"
    "fmt"
    "strings"
)

var (
    ErrInvalidID = errors.New("invalid user id")
    ErrUserNotFound = errors.New("user not found")
)

type User struct {
    ID int64
    Nickname string
}

// findUser 表示数据访问边界；真实项目中这里可以查询数据库或调用 RPC。
// 示例故意不提供真实实现，避免把外部环境引入单元测试。
func findUser(ctx context.Context, id int64) (*User, error) {
    return nil, errors.New("findUser: data source is not configured")
}

func GetDisplayName(ctx context.Context, id int64) (string, error) {
    if id <= 0 {
        return "", ErrInvalidID
    }

    u, err := findUser(ctx, id)
    if err != nil {
        return "", fmt.Errorf("find user %d: %w", id, err)
    }
    if u == nil {
        return "", ErrUserNotFound
    }

    name := strings.TrimSpace(u.Nickname)
    if name == "" {
        return "游客", nil
    }
    return name, nil
}
```

这里需要验证的是 `GetDisplayName` 的业务行为。因此只替换 `findUser`，保留参数判断、错误包装、空结果处理和字符串处理的真实逻辑。

如果直接把 `GetDisplayName` 打桩为固定返回值，再验证它返回了这个值，就没有测试到业务实现。

### 使用 PatchConvey 组织测试

创建 `user/service_test.go`：

```go
package user

import (
    "context"
    "errors"
    "testing"

    "github.com/bytedance/mockey"
    "github.com/smartystreets/goconvey/convey"
)

func TestGetDisplayName(t *testing.T) {
    mockey.PatchConvey("查询用户展示名称", t, func() {
        ctx := context.Background()

        mockey.PatchConvey("ID 非法时不查询数据", func() {
            query := mockey.Mock(findUser).
                Return((*User)(nil), nil).Build()

            for _, id := range []int64{0, -1} {
                name, err := GetDisplayName(ctx, id)
                convey.So(name, convey.ShouldBeEmpty)
                convey.So(errors.Is(err, ErrInvalidID), convey.ShouldBeTrue)
            }
            convey.So(query.Times(), convey.ShouldEqual, 0)
        })

        mockey.PatchConvey("查询失败时保留原始错误", func() {
            dbErr := errors.New("database unavailable")
            query := mockey.Mock(findUser).
                Return((*User)(nil), dbErr).Build()

            name, err := GetDisplayName(ctx, 1)

            convey.So(name, convey.ShouldBeEmpty)
            convey.So(errors.Is(err, dbErr), convey.ShouldBeTrue)
            convey.So(query.Times(), convey.ShouldEqual, 1)
        })

        mockey.PatchConvey("用户不存在", func() {
            mockey.Mock(findUser).Return((*User)(nil), nil).Build()

            name, err := GetDisplayName(ctx, 1)

            convey.So(name, convey.ShouldBeEmpty)
            convey.So(errors.Is(err, ErrUserNotFound), convey.ShouldBeTrue)
        })

        mockey.PatchConvey("空昵称使用默认名称", func() {
            mockey.Mock(findUser).
                Return(&User{ID: 1, Nickname: " \t "}, nil).Build()

            name, err := GetDisplayName(ctx, 1)

            convey.So(err, convey.ShouldBeNil)
            convey.So(name, convey.ShouldEqual, "游客")
        })

        mockey.PatchConvey("正常查询并传递正确的参数", func() {
            type requestKey struct{}
            requestCtx := context.WithValue(ctx, requestKey{}, "req-001")
            var gotCtx context.Context
            var gotID int64
            query := mockey.Mock(findUser).To(
                func(ctx context.Context, id int64) (*User, error) {
                    gotCtx, gotID = ctx, id
                    return &User{ID: id, Nickname: "  Andy  "}, nil
                },
            ).Build()

            name, err := GetDisplayName(requestCtx, 42)

            convey.So(err, convey.ShouldBeNil)
            convey.So(name, convey.ShouldEqual, "Andy")
            convey.So(gotID, convey.ShouldEqual, 42)
            convey.So(gotCtx == requestCtx, convey.ShouldBeTrue)
            convey.So(query.Times(), convey.ShouldEqual, 1)
        })
    })
}
```

最外层的 `PatchConvey` 接收 `t`，内层场景不需要重复传入。它连接 GoConvey 的场景组织能力，并管理当前作用域内的 Mockey 补丁；退出作用域时会自动释放补丁。相关规则可以参见 [Mockey 官方说明](https://github.com/bytedance/mockey#readme)。

每个分支都创建自己的桩，避免前一个场景的返回值影响下一个场景。GoConvey 的嵌套场景会沿不同执行路径重新运行外层准备逻辑，因此不要把它理解为“一次执行的普通嵌套代码块”，也不要在外层累计跨场景共享的计数。

上面的例子没有使用点导入，保留 `mockey.` 和 `convey.` 前缀，阅读时更容易区分各个 API 的来源。

### 理解打桩和断言

#### 固定返回值与自定义替身

`Mock(findUser).Return(...).Build()` 用于固定返回值，`Build()` 才会让补丁生效。返回值需要与目标函数的数量、顺序和类型匹配，示例使用 `(*User)(nil)` 明确表达空用户指针。

需要检查输入参数或者根据输入构造结果时，可以使用 `To` 提供一个替身函数。本文的替身保留了原函数的完整签名，并在调用结束后断言收集到的参数。[Mockey 的实现与 API](https://github.com/bytedance/mockey/blob/main/mock.go)还提供了调用计数等能力。

#### 方法打桩

如果数据访问逻辑是结构体的指针接收器方法，例如 `func (r *Repository) Find(...)`，通常使用方法表达式 `mockey.Mock((*Repository).Find)` 定位目标。使用 `To` 时，可以显式把 `*Repository` 接收器放在替身函数的第一个参数位置。

这种补丁会影响目标方法的其他实例调用，不能把它当成只替换某个对象的私有状态。为避免混淆，先确认调用发生在值接收器还是指针接收器方法上。

#### 常用断言

GoConvey 的断言采用 `So(实际值, 断言规则, 期望值)` 的形式，部分规则不需要期望值。常见规则可参见[官方断言文档](https://github.com/smartystreets/goconvey/wiki/Assertions)：

| 规则 | 适用场景 |
| --- | --- |
| `ShouldEqual` | 字符串、整数等值比较 |
| `ShouldResemble` | 结构体、切片、Map 等深度比较 |
| `ShouldBeNil` | 验证没有错误或结果为空 |
| `ShouldBeTrue` | 验证布尔条件 |
| `ShouldBeEmpty` | 验证字符串或容器为空 |

对于使用 `%w` 包装的错误，推荐先调用 `errors.Is`，再断言结果为真。仅比较错误字符串容易受到上下文文案变化的影响，也不能充分表达错误链的语义。

### 运行测试与查看覆盖率

Mockey 会修改运行时的函数指令。按照其[官方文档](https://github.com/bytedance/mockey/blob/main/README_cn.md)，运行测试时需要关闭编译优化和内联：

```shell
go test -gcflags="all=-N -l" -count=1 -v ./...
```

其中 `-N` 关闭编译优化，`-l` 关闭内联；`all=` 将参数应用到参与构建的所有包。`-count=1` 用于禁用测试结果缓存，调试打桩问题时可以确保测试重新执行。

只运行本文的测试入口：

```shell
go test -gcflags="all=-N -l" -count=1 -v -run '^TestGetDisplayName$' ./user
```

`-run` 在这里匹配的是 Go 的测试函数名，内层的场景描述不是 `t.Run` 创建的子测试名，不能直接照搬子测试路径进行筛选。

生成覆盖率报告：

```shell
go test -gcflags="all=-N -l" -count=1 -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
go tool cover -html=coverage.out -o coverage.html
```

覆盖率可以帮助发现没有执行到的代码，但执行过某一行不代表断言充分。本文虽然打桩了 `findUser`，却没有验证真实查询实现，因此不能用业务方法的覆盖率证明数据库访问正确。

### 在项目中使用时需要注意什么

#### 本地、IDE 和 CI 使用相同的测试参数

命令行通过而 IDE 运行失败时，首先检查 IDE 的 Go 测试运行配置是否也传入了 `-gcflags="all=-N -l"`。CI 也应该使用同样的参数，不要只在本地关闭优化。

Mockey 与 Go 工具链、操作系统和 CPU 架构有关。升级 Go 或依赖后，应在项目实际使用的组合上验证；不要把文档里的最低版本描述理解为对所有未来版本的兼容保证。

#### 补丁必须及时恢复

本文统一使用 `PatchConvey` 管理补丁。如果使用普通的 `testing` 测试，也可以在创建补丁之后立即注册清理：

```go
patch := mockey.Mock(findUser).Return((*User)(nil), nil).Build()
t.Cleanup(patch.UnPatch)
```

这段代码应放在普通测试函数内部。通常选择一种清理方式即可，不需要在同一段代码中同时混用自动清理、手动恢复和全局恢复。

#### 避免相互干扰的并发测试

对同一函数的补丁可能影响同一测试进程中的其他调用。本文用例不要调用 `t.Parallel()`，也不要在补丁生效期间启动会访问同一目标函数的后台任务。

`go test -parallel=1` 也不能替代合理的并发管理，它不会停止业务代码自己创建的 goroutine。如果被测逻辑会启动 goroutine，应在补丁释放前等待它们结束，并对测试替身中记录的共享状态做好同步。本文示例的参数记录仅适用于同步调用。

#### 不要对所有东西都打桩

稳定的纯函数通常不需要替换；业务核心逻辑应真实执行。对于新代码，可以通过小接口和构造函数注入依赖，再用简单的 fake 实现测试。对于已有代码中的包级函数、具体类型方法或较难重构的依赖，Mockey 能减少测试准备工作。

打桩测试验证的是“在这些依赖行为下，业务逻辑是否符合预期”。真实数据库、缓存和 RPC 的连通性与行为，仍应由对应的集成测试覆盖。

### 参考资料

- [Go testing 标准库文档](https://pkg.go.dev/testing)
- [Mockey 项目及中文说明](https://github.com/bytedance/mockey/blob/main/README_cn.md)
- [Mockey 函数打桩实现](https://github.com/bytedance/mockey/blob/main/mock.go)
- [GoConvey 项目](https://github.com/smartystreets/goconvey)
- [GoConvey 测试组织方式](https://github.com/smartystreets/goconvey/wiki/Composition)
- [GoConvey 执行顺序](https://github.com/smartystreets/goconvey/wiki/Execution-order)
- [GoConvey 断言规则](https://github.com/smartystreets/goconvey/wiki/Assertions)
