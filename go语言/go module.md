
# Go module

通常从github中导入的Go项目依赖了很多包，通常需要使用GOPATH来管理，Go module的出现可以方便的管理这些包。

我们以导入开源项目istio为例，该项目为go语言开发

## 前置条件

已经安装了go 和 goland IDE。通常go安装后，会包含三个环境变量：

1. GOROOT
2. GOPATH
3. GOPROXY

## 下载依赖

进入istio项目的根目录，使用如下命令下载

```shell
$ go mod download -json
```

因无法访问很多国外的网站，因此可以使用代理，修改GOPROXY环境变量值即可：

```shell
$ export GOPROXY=goproxy.cn 
```

下载后的相关包存储的位置为`$GOPATH/go/pkg/mod`

## 导入项目

在goland中open project项目，项目打开后选择Goland -> Preferences -> Go -> Go Modules -> **勾选**Enable Go Modules integration。

配置后，External Libraries中会显示Go Modules信息。

