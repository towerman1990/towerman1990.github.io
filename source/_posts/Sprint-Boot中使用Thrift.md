---
title: Sprint Boot中使用Thrift
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
namespace java cn.towerman1990.server.model

enum Status {
    No = 0;
    YES = 1;
}
struct Book {
    1: required i32 id;
    2: required string type;
    3: required string name;
    4: optional string description;
    5: required Status status;
}
```

```thrift
namespace java cn.towerman1990.server.service

include "BookModel.thrift"

exception InvalidOperationException {
    1: i32 code,
    2: string description
}

service Book {

    BookModel.Book getById(1: i32 id) throws (1: InvalidOperationException e),

    BookModel.Book create(1:BookModel.Book book) throws (1:InvalidOperationException e),

    BookModel.Book update(1:BookModel.Book book) throws (1:InvalidOperationException e),

    list <BookModel.Book> getList() throws (1:InvalidOperationException e),
}
```

#### <span id="generate-java-file">3. 生成java文件</span>

使用下载好的`Thrift`程序，将定义的接口文件转换成`Java`文件：

```bash
$thrift-path/thrift-0.16.0.exe -r --gen java BookService.thrift
```

#### <span id="new-project">4. 新建项目</span>

新建一个`Maven`项目，并在`pom.xml`文件中增加`Thrift`依赖：

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

创建项目后需将之前生成的java文件放入到项目的对应路径下。

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
        SpringApplication.run(ThriftServerApplication.class, args);

        try {
            System.out.println("Server start...");
            TProcessor tprocessor = new BookService.Processor<BookService.Iface>(new BookModelServiceImpl());
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

完整演示代码地址：[*仓库*](https://github.com/towerman1990/spring-boot-learn/tree/main/learn-thrift)


<div id="refer-anchor-1"></div>

- [1] [Thrift](https://zh.wikipedia.org/wiki/Thrift)