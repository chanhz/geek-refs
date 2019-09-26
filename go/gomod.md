# Go Module

## Overview
```bash
$ go mod
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

        go mod <command> [arguments]

The commands are:

        download    download modules to local cache
        edit        edit go.mod from tools or scripts
        graph       print module requirement graph
        init        initialize new module in current directory
        tidy        add missing and remove unused modules
        vendor      make vendored copy of dependencies
        verify      verify dependencies have expected content
        why         explain why packages or modules are needed

Use "go help mod <command>" for more information about a command.
```


## 坑
go mod 命令在 $GOPATH 里默认是执行不了的，因为 GO111MODULE 的默认值是 auto。默认在 $GOPATH 里是不会执行， 如果一定要强制执行，就设置环境变量为 on。
go mod init 在没有接module名字的时候是执行不了的，会报错 go: cannot determine module path for source directory。可以这样执行：

```bash
go mod init module-name
```

## 解析
启用 go mod 管理包后，go mod 下载的包将缓存在 $GOPATH/pkg 目录下。
相应的，Goland 会去索引这个目录下的包。


## Goland 设置
如果希望 Goland 只索引当前目录下的包，可以将 GOPATH 设置成当前目录的上上一层，前提是你的项目在 `src` 目录下。
这样做的好处是：
1. 加快 Goland 检索速度；
2. Goland 中包引入能够立即体现引用关系，将源码放到 Jenkins/VM 上编译时一般不会产生包依赖缺失的问题。

## 命令行

### bash

```bash
# Enable the go modules feature
export GO111MODULE=on
# Set the GOPROXY environment variable
export GOPROXY=https://goproxy.io
```

### Powershell
```powershell
# Enable the go modules feature
$env:GO111MODULE="on"
# Set the GOPROXY environment variable
$env:GOPROXY="https://goproxy.io"
```

### Go version >= 1.13
```bash
go env -w GOPROXY=https://goproxy.io,direct
# Set environment variable allow bypassing the proxy for selected modules
go env -w GOPRIVATE=*.corp.example.com
```