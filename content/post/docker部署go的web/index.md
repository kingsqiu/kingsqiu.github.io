---
author: 
title: 使用Docker部署Go的web应用
description: 
date: 2020-01-03
slug: docker-goweb
image: 
categories:
    - 工具汇总
tags: 
    - docker
    - go
---

## Docker的必要性

**适用场景**：在本地开发应用程序后，有很多的依赖环境或包，甚至对依赖的具体版本都有严格的要求。当开发过程完成后，若希望将应用程序部署到web服务器。这个时候必须确保所有依赖项都安装正确并且版本也完全相同，否则应用程序可能会崩溃并无法运行。如果想在另一个web服务器上也部署该应用程序，那么必须从头开始重复这个过程。这种场景就是Docker发挥作用的地方。

对于运行应用程序的主机，不管是笔记本电脑还是web服务器，唯一需要做的就是运行一个docker容器平台。从以后，就不需要担心使用的是MacOS，Ubuntu，Arch还是其他。只需定义一次应用，即可随时随地运行。

## Docker部署示例

### 创建Docker镜像

> 镜像（image）包含运行应用程序所需的所有东西——代码或二进制文件、运行时、依赖项以及所需的任何其他文件系统对象。

或者简单地说，镜像（image）是定义应用程序及其运行所需的一切。



### 编写Dockerfile

要创建Docker镜像（image）必须在配置文件中指定步骤。这个文件默认为`Dockerfile`。（虽然这个文件名可以随意命名它，但最好还是使用默认的`Dockerfile`。）

具体内容如下：

```dockerfile
FROM golang:alpine

# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 移动到工作目录：/build
WORKDIR /build

# 将代码复制到容器中
COPY . .

# 将我们的代码编译成二进制可执行文件app
RUN go build -o app .

# 移动到用于存放生成的二进制文件的 /dist 目录
WORKDIR /dist

# 将二进制文件从 /build 目录复制到这里
RUN cp /build/app .

# 声明服务端口
EXPOSE 8888

# 启动容器时运行的命令
CMD ["/dist/app"]
```

#### Dockerfile解析

官方文档：https://hub.docker.com/_/golang

**From：**这里使用基础镜像`golang:alpine`。是一个我们能够访问的存储在Docker仓库的基础镜像。这个镜像运行的是alpine Linux发行版，该发行版的大小很小并且内置了Go。

**Env：**用来设置编译阶段需要用的环境变量。

 **EXPORT，CMD：**最后，声明服务端口，因为应用程序监听的是这个端口并通过这个端口对外提供服务。并且我们还定义了在我们运行镜像的时候默认执行的命令`CMD ["/dist/app"]`。

### 构建镜像

示例代码：

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", hello)
	server := &http.Server{
		Addr: ":8888",
	}
  fmt.Println("server startup...")
	if err := server.ListenAndServe(); err != nil {
		fmt.Printf("server startup failed, err:%v\n", err)
	}
}

func hello(w http.ResponseWriter, _ *http.Request) {
	w.Write([]byte("hello world"))
}
```

在项目目录下，执行下面的命令创建镜像，并指定镜像名称为`goweb_app`：

```bash
docker build . -t goweb_app
```

等待构建过程结束，输出如下提示：

```bash
...
Successfully built 90d9283286b7
Successfully tagged goweb_app:latest
```

现在我们已经准备好了镜像，但是目前它什么也没做。我们接下来要做的是运行我们的镜像，以便它能够处理我们的请求。运行中的镜像称为容器。

执行下面的命令来运行镜像：

```bash
docker run -p 8888:8888 goweb_app
```

标志位`-p`用来定义端口绑定。由于容器中的应用程序在端口8888上运行，我们将其绑定到主机端口也是8888。如果要绑定到另一个端口，则可以使用`-p $HOST_PORT:8888`。例如`-p 5000:8888`。

现在就可以测试下我们的web程序是否工作正常，打开浏览器输入`http://127.0.0.1:8888`就能看到我们事先定义的响应内容如下：

```bash
hello world
```

## 分阶段构建示例

Go程序编译之后会得到一个可执行的二进制文件，其实在最终的镜像中是不需要go编译器的，也就是说我们只需要一个运行最终二进制文件的容器即可。

Docker的最佳实践之一是通过仅保留二进制文件来减小镜像大小，为此这里使用一种称为多阶段构建的技术，这意味着我们将通过多个步骤构建镜像。

```dockerfile
FROM golang:alpine AS builder

# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 移动到工作目录：/build
WORKDIR /build

# 将代码复制到容器中
COPY . .

# 将我们的代码编译成二进制可执行文件 app
RUN go build -o app .

###################
# 接下来创建一个小镜像
###################
FROM scratch

# 从builder镜像中把/dist/app 拷贝到当前目录
COPY --from=builder /build/app /

# 需要运行的命令
ENTRYPOINT ["/app"]
```

使用这种技术，剥离了使用`golang:alpine`作为编译镜像来编译得到二进制可执行文件的过程，并基于`scratch`生成一个简单的、非常小的新镜像。将二进制文件从命名为`builder`的第一个镜像中复制到新创建的`scratch`镜像中。

有关scratch镜像的更多信息，查看https://hub.docker.com/_/scratch