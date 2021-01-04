## 一、关于go modules
### 1.1 go modules 是go1.11 新加的特性
```
$ go version
go version go1.15.6 darwin/amd64
```

### 1.2关于modules 官方定义
模块是相关Go包的集合。modules是源代码交换和版本控制的单元。 go命令直接支持使用modules，包括记录和解析对其他模块的依赖性。modules替换旧的基于GOPATH的方法来指定在给定构建中使用哪些源文件。

### 1.3 使用modules 的配置
##### 配置GO111MODULE
- GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。

- GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH/src目录下查找。 （pkg 包都存放在 $GOPATH/pkg 下）

- GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。（pkg 包都存放在 $GOPATH/pkg 下）

### 1.4 本人配置
export GO111MODULE=auto

## 二、go mod 的一些命令
| 命令 | 说明 |
| --- | --- |
| download | download modules to local cache(下载依赖包 重要) |
| edit | edit go.mod from tools or scripts（编辑go.mod |
| graph | print module requirement graph (打印模块依赖图) |
| init	 | initialize new module in current directory（在当前目录初始化mod 重要） |
| tidy | add missing and remove unused modules(拉取缺少的模块，移除不用的模块 重要) |
| vendor | make vendored copy of dependencies(将依赖复制到vendor下) |
| verify | verify dependencies have expected content (验证依赖是否正确） |
| why | explain why packages or modules are needed(解释为什么需要依赖) |

## 三、如何使用 go mod
### 3.1 简单使用
```
mkdir hello
cd hello 
go mod init hello 
# 此时会出现一个hello下会出现一个 go.mod 目录
# 需要下载 所有第三方包时 go mod download
# 下载第三包可以直接使用 go get need_pkg
# 下载好的依赖 和 版本 会加入到 go.mod 里面,
# 下载好的第三包 会放在到$GOPATH/pkg/mod 中
# 没有设置GOPATH的话 下载好的第三方包会放在~/go/pkg/mod
# 如果你想放在当前目前可以执行如下命令

go mod tidy
# 检测依赖的包，下载使用到的包，剔除未使用的包

# 如果你希望将第三方依赖包放至在本项目下，可以使用该命令，此时会将第三方依赖下载至vendor 目录中
go mod vendor    
```

### 3.2 关于依赖升级
```
删除 go.mod中需要升级的第三方依赖
然后执行 go mod tidy ,会自动下载最新的。

关于依赖的版本也是可以指定的
在go.mod 中使用指定版本的话，可以自行设置。
```

### 3.3 关于依赖打包
```
go build  -ldflags="-s -w" -o app ./main.go
# -ldflags="-s -w" 压缩程序
```

### 3.4 github 拉取 他人含有go.mod的项目时,下载所有第三方包
```
go mod tidy
```

### 3.5 关于如何使用自定义包
```
hello
    |--conf
        |-conf.go
    |-main.go
    |-go.mod
如何导入conf 包呢?
先查看go.mod 中的module 后的定义的module_name
在导入时  直接使用module_name/conf   即可
```

## 四、goland的配置
goland 升级到最新的,旧的goland 版本时不支持go mod,
在preferences -> go -> Go Modules(vgo) 
给Enable Go Modules (vgo) Integration 打勾勾就行