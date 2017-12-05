---
layout: post
title: Go 语言的安装和配置
category : [Go]
tagline: "Supporting tagline"
tags : [Go]
---
{% include JB/setup %}
# Go 语言的安装和配置
---


### 安装和配置
#### 下载安装包，由于官网需要访问不了，所以可以到 Go 语言中文网（https://studygolang.com/dl）下载安装包。

#### 将下载的源码包解压至 /usr/local目录。 
``` 
$ tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz
```
解压完成后，在 /usr/local 目录下生成 go 目录。 

#### 配置环境变量  

``` 
$ vim /etc/profile 
``` 
将 /usr/local/go/bin 目录加入至 PATH 环境变量 
```  
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin 
``` 
编译生效
```  
$ source /etc/profile 
``` 

#### 验证 Go 环境配置成功 
``` 
$ go version 
``` 
如果正确打印出 Go 语言版本号，则表示安装和配置成功：
``` 
go version go1.9.2 linux/amd64 
```  

<!--break-->

### 在 idea 中的使用 

#### 在 idea 中安装 “go” 插件。
![](/images/2017-12-02-go-plugin.png)  

#### 新建 Go 项目。
！[](/images/2017-12-02-new-go-project.png) 