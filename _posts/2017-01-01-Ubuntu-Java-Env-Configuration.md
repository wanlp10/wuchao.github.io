---
layout: post
title: Ubuntu 系统的开发环境安装和配置
category : [Ubuntu, Java]
tagline: "Supporting tagline"
tags : [Ubuntu, Java]
---
{% include JB/setup %}
# Ubuntu 系统的开发环境安装和配置
---

## Vim 的安装 

> Ubuntu 无法安装 vim 软件包:  http://blog.csdn.net/haida_liudan/article/details/8768215

``` 
($ sudo apt-get remove vim-common)
$ sudo apt-get install vim
``` 

<!--break-->

## 解压工具 unar
可以解决 ubuntu 下解压文件后文件名中文乱码的问题
``` 
$ sudo apt-get install unar   
``` 
使用: 
``` 
unar xxx.jar  
```

### 阿里源   

> http://blog.csdn.net/u011148119/article/details/50338355 

``` 
$ cd /etc/apt
$ sudo cp /etc/apt/source.list /etc/apt/source.list.bak 
$ sudo vim /etc/apt/source.list
```

删除原来的列表内容，添加如下内容： 

``` 
deb http://mirrors.aliyun.com/ubuntu/ vivid main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ vivid-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ vivid-backports main restricted universe multiverse 
```

更新源 

``` 
$ sudo apt-get update 
$ sudo apt-get upgrade 
```



## Idea 的安装 




## JDK 的安装  

> http://www.linuxidc.com/Linux/2016-11/136958.htm 

### 1 JRE vs OpenJDK vs [Oracle](http://www.linuxidc.com/topicnews.aspx?tid=12) JDK 

#### 1.1 JRE 和 JDK

- `JRE(Java Runtime Environment)`它是你运行一个基于 Java 语言应用程序的所正常需要的环境。如果你不是一个程序员的话，这些足够你的需要.
- `JDK`代表`Java`开发工具包，如果你想做一些有关`Java`的开发 (阅读程序), 这正是你所需要的.

#### 1.2 Open JDK 和 Oracle JDK

- `OpenJDK`是`Java`开发工具包的开源实现
- `Oracle JDK`是`Java`开发工具包的官方`Oracle`版本

尽管`OpenJDK`已经足够满足大多数的案例，但是许多程序比如 `Android Studio`建议使用`Oracle JDK`, 以避免 UI / 性问题.

### 2 检查 Java 是否已经安装在 Ubuntu 上 

打开终端，使用下面的命令：

```
java -version
```

如果你看到像下面的输出，这就意味着你并没有安装过 Java:

```
The program ‘java’ can be found in the following packages:
*default-jre
* gcj-4.6-jre-headless
* openjdk-6-jre-headless
* gcj-4.5-jre-headless
* openjdk-7-jre-headless
Try: sudo apt-get install
```

### 3 在 Ubuntu 和 Linux Mint 上安装 Java

看了各种类型`Java`的不同之后, 让我们看如何安装他们.

在`Ubuntu`和`Linux Mint`上安装`JRE`

#### 3.1 安装 jre

打开终端，使用下面的命令安装 JRE :

```
sudo apt-get install default-jre
```

#### 3.2 安装 OpenJDK

在`Ubuntu`和`Linux Mint`上安装`OpenJDK`

在终端，使用下面的命令安装 OpenJDK Java 开发工具包：

```
sudo apt-get install default-jdk
```

特殊地, 如果你想要安装`Java 8`, `Java 7`或者`Java 6`等等，你可以使用`openjdk-7-jdk/openjdk-6jdk`, 但是记住在此之前安装`openjdk-7-jre/openjdk-6-jre`

#### 3.3 安装 Oracle JDK

在`Ubuntu`和`Linux Mint`上安装`Oracle JDK`

##### 3.3.1 使用源安装

使用下面的命令安装，只需一些时间，它就会下载许多的文件，所及你要确保你的网络环境良好：

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default
```

如果你想安装`Java 8(i.e Java 1.8)`, 在上面的命令中用`java7`代替`java8`.

##### 3.3.2 通过 bin 包安装

此外可以用 Linux 上通用的 bin 包安装，下载官方 bin 包，终端下面安装解压，然后修改环境变量指向那个 jdk 便可

按照需要选择不同的版本, 下载 bin 包

```
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```

将下载下来的`jdk-8u101-linux-x64.tar.gz`文件解压。

使用如下命令解压：

```
sudo tar zxvf ./jdk-8u101-linux-x64.tar.gz
```

为了方便管理, 可将解压后的文件移至另一个文件夹, 笔者将文件移至了`/usr/java/jdk1.8.0_101`目录下。 

##### 3.3.3 设置环境变量 

编辑. bashrc 文件

```
JAVA_HOME=/usr/java/jdk1.8.0_101
JRE_HOME=$JAVA_HOME/jre
JAVA_BIN=$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

如果是使用源方法安装的, 则默认的安装路径是在`/usr/lib/jvm/java-8-oracle`中, 则配置对应的 JAVA_HOME 即可

```
JAVA_HOME=/usr/lib/jvm/java-8-oracle
```

为了让更改立即生效，请在终端执行如下命令：

```
source ~/.bashrc
```

#### 3.4 验证

通过以上步骤，JDK 已安装完成，输入以下命令验证。

``` 
java -version
```



## Gradle 的安装

> [install gradle](https://gradle.org/install)  

### 1 package manager 安装  

[SDKMAN!](http://sdkman.io/) is a tool for managing parallel versions of multiple Software Development Kits on most Unix-based systems.

```
$ sdk install gradle 4.0.1

```

[Homebrew](http://brew.sh/) is “the missing package manager for macOS”.

```
$ brew update && brew install gradle

```

[Scoop](http://scoop.sh/) is a command-line installer for Windows inspired by Homebrew.

```
$ scoop install gradle 
```

### 2 下载 Gradle distribution 包安装

http://blog.csdn.net/stwstw0123/article/details/47809189  

> Ubuntu 官方源的 Gradle 太陈旧了。。。陈旧到不支持 [android](http://lib.csdn.net/base/android) Studio 的 `jcenter`方法，如果强行编译，会出现如下错误：
>
> Could not find method jcenter() for arguments [] on repository [Container](http://lib.csdn.net/base/docker).
>
> 所以，起码到现在 (2015-08-20), 不要用`$sudo apt-get install gradle`来安装 *gradle*，如果安装了，要用 `$ sudo apt-get remove gradle`卸载掉。 

#### 下载 Gradle distribution 

``` 
distribution 的地址： https://services.gradle.org/distributions/
```

#### 安装

``` 
$ sudo unzip gradle-2.6-all.zip -d /opt/gradle/ 
```

#### 配置 

``` 
$ sudo vim /etc/profile
```

文件末尾添加：

```
export GRADLE_HOME=/opt/gradle/gradle-2.6
export PATH=$GRADLE_HOME/bin:$PATH 
```

#### 重启

重启机器，然后就可以运行 `gradle -v`

```
$ sudo reboot
$ gradle -v 
```

### 3 Upgrade with the Gradle Wrapper 

If your existing Gradle-based build uses the [Gradle Wrapper](https://docs.gradle.org/4.0.1/userguide/gradle_wrapper.html), you can easily upgrade by running the `wrapper` task, specifying the desired Gradle version:

```
$ ./gradlew wrapper --gradle-version=4.0.1 --distribution-type=bin

```

Note that it is not necessary for Gradle to be installed to use the Gradle wrapper. The next invocation of `gradlew` or `gradlew.bat` will download and cache the specified version of Gradle.

```
$ ./gradlew tasks
Downloading https://services.gradle.org/distributions/gradle-4.0.1-bin.zip
... 
```



## Git 的安装 (apt 方式)

> http://blog.csdn.net/yhl_leo/article/details/50760140 

``` 
$ sudo add-apt-repository ppa:git-core/ppa 
$ sudo apt-get update
$ sudo apt-get install git  
$ git --version 
```



## Mysql 的安装 

替换了淘宝源后，mysql安装不了，把备份的初始源追加到淘宝源后面。

``` 
$ cd /etc/apt  
$ cat source.list.bak >> source.list  
```

安装 mysql-server： 

``` 
$ sudo apt-get install mysql-server 
```

设置账号密码，创建数据库：

``` 
CREATE DATABASE `test2` DEFAULT CHARACTER SET utf8;  
```





