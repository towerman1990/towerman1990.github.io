---
title: Spring Boot 中使用 Thrift
date: 2022-04-16 11:38:55
tags:
 - Java
---

### <span id="thrift-brief">Thrift简介</span>

Thrift是一种接口描述语言和二进制通讯协议，它被用来定义和创建跨语言的服务。它被当作一个远程过程调用（RPC）框架来使用，是由`Facebook`为“大规模跨语言服务开发”而开发的。它通过一个代码生成引擎联合了一个软件栈，来创建不同程度的、无缝的跨平台高效服务，可以使用`C#`、`C++`（基于`POSIX`兼容系统）、`Cappuccino`、`Cocoa`、`Delphi`、`Erlang`、`Go`、`Haskell`、`Java`、`Node.js`、`OCaml`、`Perl`、`PHP`、`Python`、`Ruby`和`Smalltalk`。虽然它以前是由`Facebook`开发的，但它现在是`Apache`软件基金会的开源项目了。该实现被描述在2007年4月的一篇由`Facebook`发表的技术论文中，该论文现由`Apache`掌管。[<sup>[1]</sup>](#refer-anchor-1)

<!-- more -->

### <span id="steps">具体使用步骤</span>

#### <span id="thrift-install">1. 安装Thrift</span>

由于我用的是`Windows`系统，直接去[`Thrift`官网](https://thrift.apache.org)下载`EXE`文件就可以，其他系统的安装可以参见官网的安装说明。

#### <span id="define-interface-file">2. 定义接口文件</span>

我定义两个文件，一个是数据结构文件（`BookModel.thrift`），另一个是需要实现的方法接口文件（`BookService.thrift`）：

```thrift
namespace java cn.towerman1990.learn_thrift.model

enum Status {
    NO = 0;
    YES = 1;
}
struct BookModel {
    1: required i32 id;
    2: required string type;
    3: required string name;
    4: optional string description;
    5: required Status status;
}
```

```thrift
namespace java cn.towerman1990.learn_thrift.service

include "BookModel.thrift"

exception InvalidOperationException {
    1: i32 code,
    2: string description
}

service BookService {

    BookModel.BookModel getById(1: i32 id) throws (1: InvalidOperationException e),

    BookModel.BookModel create(1: BookModel.BookModel book) throws (1: InvalidOperationException e),

    BookModel.BookModel update(1: BookModel.BookModel book) throws (1: InvalidOperationException e),

    list<BookModel.BookModel> getList() throws (1: InvalidOperationException e),
}
```

#### <span id="generate-java-file">3. 生成 Java 文件</span>

使用下载好的`Thrift`程序，将定义的接口文件转换成`Java`文件：

```powershell
& "C:\tools\thrift\thrift-0.16.0.exe" -r --gen java BookService.thrift
```

在两个 `.thrift` 文件所在目录执行以上命令，并将可执行文件路径替换为实际安装路径。`-r` 会同时生成被引用文件的代码，默认输出到 `gen-java` 目录。这里的 `BookModel.BookModel` 中，前一个名称表示被包含的 IDL 文件，后一个名称表示该文件中定义的结构体。

#### <span id="new-project">4. 新建项目</span>

新建一个`Maven`项目，并在`pom.xml`文件中增加`Thrift`依赖。本文沿用 Java 17、Spring Boot 2.6.6 和 Thrift 0.16.0 的示例版本，Thrift 编译器与 Java 运行库使用相同版本：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.towerman1990.learn_thrift</groupId>
    <artifactId>learn-thrift</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <spring.version>2.6.6</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.thrift</groupId>
            <artifactId>libthrift</artifactId>
            <version>0.16.0</version>
        </dependency>
    </dependencies>

</project>
```

创建项目后，将 `gen-java` 下的 `cn` 目录复制到 `src/main/java` 下，保留生成的包目录结构。此时应能找到 `cn.towerman1990.learn_thrift.model.BookModel`、`Status` 和 `cn.towerman1990.learn_thrift.service.BookService` 等类型。

如果 Thrift 0.16.0 生成的代码引用了 `javax.annotation.Generated`，在 Java 17 下还需要补充 `javax.annotation:javax.annotation-api:1.3.2` 依赖，或生成代码时使用 `--gen java:generated_annotations=suppress` 关闭该注解的生成。

#### <span id="implement-interface">5. 实现定义的接口</span>

创建一个`BookService.Iface`的实现类：

```java
package cn.towerman1990.learn_thrift.impl;

import cn.towerman1990.learn_thrift.model.BookModel;
import cn.towerman1990.learn_thrift.model.Status;
import cn.towerman1990.learn_thrift.service.BookService;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

@Service
public class BookModelServiceImpl implements BookService.Iface {

    @Override
    public cn.towerman1990.learn_thrift.model.BookModel getById(int id) {
        // todo
        // find data from database, then transfer to BookModel;
        var bookModel = new BookModel();
        bookModel.setId(id);
        bookModel.setType("小说");
        bookModel.setName("《红楼梦》");
        bookModel.setDescription("《红楼梦》，中国古典长篇章回小说，是中国四大小说名著。");
        bookModel.setStatus(Status.YES);
        return bookModel;
    }

    @Override
    public cn.towerman1990.learn_thrift.model.BookModel create(cn.towerman1990.learn_thrift.model.BookModel bookModel) {
        return bookModel;
    }

    @Override
    public cn.towerman1990.learn_thrift.model.BookModel update(cn.towerman1990.learn_thrift.model.BookModel bookModel) {
        return bookModel;
    }

    @Override
    public List<cn.towerman1990.learn_thrift.model.BookModel> getList() {
        return new ArrayList<>();
    }
}
```

#### <span id="server-implement">6. 服务端实现</span>

创建一个名为`ThriftServerApplication`的类文件，代码如下：

```java
package cn.towerman1990.learn_thrift;

import cn.towerman1990.learn_thrift.impl.BookModelServiceImpl;
import cn.towerman1990.learn_thrift.service.BookService;
import org.apache.thrift.TProcessor;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.server.TServer;
import org.apache.thrift.server.TSimpleServer;
import org.apache.thrift.transport.TServerSocket;
import org.apache.thrift.transport.TTransportException;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ThriftServerApplication {
    public static void main(String[] args) {
        var context = SpringApplication.run(ThriftServerApplication.class, args);

        try {
            System.out.println("Server start...");
            BookService.Iface handler = context.getBean(BookModelServiceImpl.class);
            TProcessor tprocessor = new BookService.Processor<BookService.Iface>(handler);
            TServerSocket serverTransport = new TServerSocket(1314);
            TServer.Args tArgs = new TServer.Args(serverTransport);
            tArgs.processor(tprocessor);
            tArgs.protocolFactory(new TBinaryProtocol.Factory());
            TServer server = new TSimpleServer(tArgs);
            server.serve();
        } catch (TTransportException e) {
            e.printStackTrace();
        }
    }
}
```

#### <span id="client-implement">7. 客户端实现</span>

服务端通过 Spring 容器获取带有 `@Service` 注解的实现类，避免手动 `new` 导致该类中的依赖注入失效。这里的 `TSimpleServer` 用于演示基础调用，`serve()` 会阻塞当前线程；实际项目还需要根据并发需求选择服务模型，并管理启动、停止和资源释放。

创建一个名为`ThriftClientApplication`的类文件，代码如下：

```java
package cn.towerman1990.learn_thrift;

import cn.towerman1990.learn_thrift.model.BookModel;
import cn.towerman1990.learn_thrift.model.Status;
import cn.towerman1990.learn_thrift.service.BookService;
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TProtocol;
import org.apache.thrift.transport.TSocket;
import org.apache.thrift.transport.TTransport;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ThriftClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ThriftClientApplication.class, args);

        System.out.println("Client start...");
        try (TTransport transport = new TSocket("localhost", 1314, 30000)) {
            TProtocol protocol = new TBinaryProtocol(transport);
            BookService.Client client = new BookService.Client(protocol);
            transport.open();
            var result = client.getById(1);
            System.out.println(result);
        } catch (TException e) {
            e.printStackTrace();
        }
    }
}

```

#### <span id="test">8. 测试</span>

先启动`ThriftServerApplication`类，后启动`ThriftClientApplication`类，打印结果如下：

```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.6.6)

2022-04-16 12:53:52 [main] INFO  [org.springframework.boot.StartupInfoLogger        ] >>> Starting ThriftClientApplication using Java 17.0.2 on DESKTOP-T3TFEF9 with PID 9800 (D:\dev\java\spring-boot-learn\learn-thrift\target\classes started by chunyu in D:\dev\java\spring-boot-learn)
2022-04-16 12:53:52 [main] INFO  [org.springframework.boot.SpringApplication        ] >>> No active profile set, falling back to 1 default profile: "default"
2022-04-16 12:53:52 [main] INFO  [org.springframework.boot.StartupInfoLogger        ] >>> Started ThriftClientApplication in 0.638 seconds (JVM running for 0.878)
Client start...
BookModel(id:1, type:小说, name:《红楼梦》, description:《红楼梦》，中国古典长篇章回小说，是中国四大小说名著。, status:YES)

Process finished with exit code 0
```

注意：示例将客户端和服务端启动类放在同一个包中，二者都带有 `@SpringBootApplication`。正式拆分项目时，应将它们放在独立模块或互不扫描的包中；普通 Thrift 客户端本身也可以不启动 Spring 容器，直接创建连接并调用。

完整演示代码地址：[*仓库*](https://github.com/towerman1990/spring-boot-learn/tree/main/learn-thrift)。本文中的 IDL 包名和类型名已统一，使用仓库代码时也应核对其生成文件与 IDL 是否一致。


<div id="refer-anchor-1"></div>

- [1] [Thrift](https://zh.wikipedia.org/wiki/Thrift)
- [Apache Thrift IDL 文档](https://thrift.apache.org/docs/idl)
- [Apache Thrift Java 示例](https://thrift.apache.org/tutorial/java.html)
