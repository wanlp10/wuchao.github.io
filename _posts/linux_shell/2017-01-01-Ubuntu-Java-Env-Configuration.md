---
layout: post
title: Ubuntu 系统的开发环境安装和配置
category : [Linux]
tagline: "Supporting tagline"
tags : [Ubuntu]
---
{% include JB/setup %}
# Ubuntu 系统开发环境的安装和配置
---

> 安装时语言选择中文，不然不带中文输入法，且以后安装中文输入法比较麻烦。


<!--break-->

## 安装 Ubuntu 16.04 后要做的事 
> https://blog.csdn.net/whatlookingfor/article/details/52382538


## unetbootin 的安装

> [ubuntu下制作u盘启动盘](http://blog.csdn.net/l0605020112/article/details/20048899)
>
> [Ubuntu下格式化U盘的方法(基于格式化命令)](http://mtoou.info/ubuntu-geshihua-upan/)

### 安装u盘制作工具 unetbootin
```
$ sudo apt-get install unetbootin
```


### 格式化u盘
对于要格式化的分区必须要先用 `umount` 命令卸载掉才能格式化：
```  
$ sudo fdisk -l #查看U盘盘符，假设为/dev/sdb
$ sudo umount /dev/sdb #先卸载u盘
```

格式化为 FAT 分区（对于u盘我们一般格式化为 FAT 格式或者 FAT32 格式，不过在 linux 下这些会都显示为 FAT 格式。）：
```
$ sudo mkfs.vfat -F 32 /dev/sdb  #格式化为fat32格式
```

格式化为 NTFS 分区，先要安装 ntfsprogs：
```
$ sudo apt-get install ntfsprogs
$ sudo mkfs.ntfs /dev/sdb
```

格式化为 ext4/3/2：
```  
$ sudo mkfs.ext4 /dev/sda1 # 格式化为ext4分区
$ sudo mkfs.ext3 /dev/sda1 # 格式化为ext3分区
$ sudo mkfs.ext2 /dev/sda1 #格式化为ext2分区
```


## Vim 的安装

> Ubuntu 无法安装 vim 软件包:  http://blog.csdn.net/haida_liudan/article/details/8768215

```
($ sudo apt-get remove vim-common)
$ sudo apt-get install vim
```
如果出现下面的错误：
```
google-chrome-stable : 依赖: libappindicator1 但是它将不会被安装
```
解决办法：
```
sudo apt-get -f install libappindicator1 libindicator7
```


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


## JDK 的安装  
> [http://www.linuxidc.com/Linux/2016-11/136958.htm](http://www.linuxidc.com/Linux/2016-11/136958.htm)
>    
> [http://blog.csdn.net/gobitan/article/details/24322561](http://blog.csdn.net/gobitan/article/details/24322561)   
>
> [http://www.wikihow.com/Install-Oracle-Java-JDK-on-Ubuntu-Linux](http://www.wikihow.com/Install-Oracle-Java-JDK-on-Ubuntu-Linux)   

### 1 JRE vs OpenJDK vs [Oracle](http://www.linuxidc.com/topicnews.aspx?tid=12) JDK

#### 1.1 JRE 和 JDK

- `JRE(Java Runtime Environment)` 它是你运行一个基于 Java 语言应用程序的所正常需要的环境。如果你不是一个程序员的话，这些足够你的需要。
- `JDK` 代表 Java 开发工具包，如果你想做一些有关 Java 的开发 (阅读程序), 这正是你所需要的。

#### 1.2 Open JDK 和 Oracle JDK

- `OpenJDK` 是 Java 开发工具包的开源实现
- `Oracle JDK` 是 Java 开发工具包的官方 Oracle 版本

尽管 OpenJDK 已经足够满足大多数的案例，但是许多程序比如 Android Studio 建议使用 Oracle JDK， 以避免 UI / 性问题。

### 2 检查 Java 是否已经安装在 Ubuntu 上

打开终端，使用下面的命令：

```
$ java -version
```

如果你看到像下面的输出，这就意味着你并没有安装过 Java：

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

看了各种类型 Java 的不同之后, 让我们看如何安装他们。

在 Ubuntu 和 Linux Mint 上安装 JRE 

#### 3.1 安装 OpenJDK

在 Ubuntu 和 Linux Mint 上安装 OpenJDK 

在终端，使用下面的命令安装 OpenJDK Java 开发工具包：

```
$ sudo apt-get install default-jre
$ sudo apt-get install default-jdk
```

特殊地, 如果你想要安装 Java7 或者 Java6 等等，你可以使用 `openjdk-7-jdk/openjdk-6jdk`, 但是记住在此之前安装 `openjdk-7-jre/openjdk-6-jre`。

#### 3.2 安装 Oracle JDK

在 Ubuntu 和 Linux Mint 上安装 Oracle JDK。 

##### 3.2.1 使用源安装（推荐方式）

使用下面的命令安装，只需一些时间，它就会下载许多的文件，所及你要确保你的网络环境良好：

```
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
$ sudo apt-get install oracle-java8-set-default
```

如果你想安装 Java7(i.e Java 1.7) , 在上面的命令中用 java7 代替 java8。

##### 3.2.2 通过 bin 包安装

此外可以用 Linux 上通用的 bin 包安装，下载官方 bin 包，终端下面安装解压，然后修改环境变量指向那个 jdk 便可。

按照需要选择不同的版本，下载 bin 包。

```
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```

将下载下来的 `jdk-8u101-linux-x64.tar.gz` 文件解压。

使用如下命令解压：

```
$ sudo tar zxvf ./jdk-8u144-linux-x64.tar.gz
```

#### 3.3 配置环境变量
编辑 `/etc/profile` 文件：
```
JAVA_HOME=/usr/local/java/jdk1.8.0_144
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME PATH
```
如果是使用源方法安装的, 则默认的安装路径是在 `/usr/lib/jvm/java-8-oracle` 中, 则配置对应的 JAVA_HOME 即可。（可以不配置）
```
JAVA_HOME=/usr/lib/jvm/java-8-oracle
```

#### 3.4 验证

通过以上步骤，JDK 已安装完成，输入以下命令验证。

```
$ java -version
```
正确打印：
```
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

如果打印：
```
openjdk version "1.8.0_131"
OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-2ubuntu1.16.04.3-b11)
OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```
则配置如下：

```
# 配置 ubuntu 的 JDK 和 JRE 的位置
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.8.0_131/bin/java" 1
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.8.0_131/bin/javac" 1
$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/jdk1.8.0_131/bin/javaws" 1


# 配置 Oracle 为系统默认 JDK/JRE
$ sudo update-alternatives --set java /usr/local/java/jdk1.8.0_131/bin/java
$ sudo update-alternatives --set javac /usr/local/java/jdk1.8.0_131/bin/javac
$ sudo update-alternatives --set javaws /usr/local/java/jdk1.8.0_131/bin/javaws
```   


## Git 的安装

### 安装   

> http://blog.csdn.net/yhl_leo/article/details/50760140

```
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt-get update
$ sudo apt-get install git  
$ git --version
```
### git lfs 的安装 
``` 
$ curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash 
$ sudo apt-get install git-lfs 
# 切换到项目路径
$ git lfs install 
$ git lfs track /cache/深入深入分布式缓存\ 从原理到实践.pdf 
$ git add .gitattributes
$ git add cache/深入分布式缓存\ 从原理到实践.pdf 
$ git commit -m "Add a large file"
$ git push origin master 
```
> [Git LFS的使用](https://www.jianshu.com/p/493b81544f80)  
> [https://git-lfs.github.com/](https://git-lfs.github.com/)



## Gradle 的安装   

### 安装   

> [install gradle](https://gradle.org/install)  

#### 方法1 package manager 安装  

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

#### 方法2 下载 Gradle distribution 包安装

http://blog.csdn.net/stwstw0123/article/details/47809189  

> Ubuntu 官方源的 Gradle 太陈旧了。。。陈旧到不支持 [android](http://lib.csdn.net/base/android) Studio 的 `jcenter`方法，如果强行编译，会出现如下错误：
>
> Could not find method jcenter() for arguments [] on repository [Container](http://lib.csdn.net/base/docker).
>
> 所以，起码到现在 (2015-08-20), 不要用 `$sudo apt-get install gradle` 来安装 *gradle*，如果安装了，要用 `$ sudo apt-get remove gradle` 卸载掉。

##### 下载 Gradle Distribution

```
distribution 的地址： https://services.gradle.org/distributions/
```

##### 安装

```
$ sudo unzip gradle-3.5-bin.zip -d /usr/local/gradle/
```

##### 配置

```
$ sudo vim /etc/profile
```

文件末尾添加：

```
export GRADLE_HOME=/usr/local/gradle/gradle-3.5
export PATH=$GRADLE_HOME/bin:$PATH
```

载执行：
```
$ source /etc/profile
```

##### 验证
```
$ gradle -v
```

#### 方法3 Upgrade with the Gradle Wrapper

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


## Mysql 的安装

### 安装   

安装 mysql-server：
Update: Before installing mysql, make sure that no other mysql packages are present:
```
$ dpkg -l | grep mysql - returns list of mysql packages.
```

Use apt-get purge <package name> to purge them.

```
$ sudo apt-get install mysql-server (命令执行完成后会要求输入数据库密码)
$ # sudo apt-get install mysql-client (上面的命令默认安装 server-client)
$ sudo apt-get install libmysqlclient-dev
```
> 替换了淘宝源后，mysql安装不了，把备份的初始源追加到淘宝源后面。
>  
>  ```
>  $ cd /etc/apt  
>  $ cat source.list.bak >> source.list  
>  ```

安装指定版本:
```
# 依赖
$ sudo apt-get install mysql-client-core-5.6 mysql-client-5.6
$ apt-get install mysql-server-5.6
```
通过下载压缩包安装指定版本:   

```
# 下载
$ wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-server_5.6.34-1ubuntu14.04_amd64.deb-bundle.tar

# 解压
$ tar -xvf mysql-server_5.6.34-1ubuntu14.04_amd64.deb-bundle.tar

# 安装
> https://www.cnblogs.com/oldfish/p/5039772.html
```  
通过下载 deb 安装包安装:
```` 

````


验证安装：
```
$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.20-0ubuntu0.16.04.1 (Ubuntu)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  
```

新建数据库：

```
CREATE DATABASE database_name DEFAULT CHARACTER SET utf8;  
```

### 卸载
> http://www.linuxidc.com/Linux/2013-04/82934.htm

```
# 删除 mysql
# $ sudo apt-get autoremove --purge mysql-server-*
# $ sudo apt-get remove mysql-common
$ sudo apt-get autoremove --purge mysql-*

# 清理残留数据
$ dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
``` 

### Mysql 忘记密码
在终端输入下面的命令进入 mysqld.cnf 
``` 
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf 

```  
   
在 [mysqld] 下方的 skip-external-locking 下面添加一行：
``` 
skip-grant-tables 
``` 
   
重启 mysql 服务
``` 
$ sudo service mysql restart
``` 
  
在终端输入下面的命令进入 mysql 
``` 
$ mysql
```
  
输入 use mysql 切换至 mysql 数据库
``` 
mysql> use mysql
```
  
将 root 用户的密码修改为 "root":
``` 
mysql> update mysql.user set authentication_string=password('root') where user='root' and host ='localhost';
```
  
修改字段 plugin 
``` 
mysql> update user set plugin="mysql_native_password";
```
> plugin 字段标识可以用于验证用户身份的插件，如果字段为空，服务器使用内建授权验证机制验证用户身份。
  
  
刷新 mysql 的系统权限相关表 
``` 
mysql> flush privileges;
``` 
> mysql 新设置用户或更改密码后需用 flush privileges 刷新 mysql 的系统权限相关表，否则会出现拒绝访问，还有一种方法，就是重新启动 mysql 服务器，来使新设置生效。
    
退出
``` 
mysql> quit;
```

把 `/etc/mysql/mysql.conf.d/mysqld.cnf` 文件中添加的一行注释掉

> 参考: [Ubuntu 16.04下忘记MySQL密码解决方法](https://blog.csdn.net/hq815601489/article/details/78608644)


## DBeaver 的安装
> Free universal database manager and SQL client.  

下载链接 : [Linux Debian package 64 bit (installer)](https://dbeaver.jkiss.org/download/)   
  
然后使用 `dpkg` 命令安装: 
``` 
sudo dpkg -i dbeaver-ce_4.3.4_amd64.deb  
```


## Redis 的安装
> [linux安装redis](https://www.cnblogs.com/oskyhg/p/7293905.html)

``` 
$ wget http://download.redis.io/releases/redis-3.0.1.tar.gz 
$ tar xzf redis-3.0.1.tar.gz
$ cd redis-3.0.1
$ make 
$ ./src/redis-server
   
# 直接make 编译
make
# 可使用root用户执行`make install`，将可执行文件拷贝到/usr/local/bin目录下。这样就可以直接敲名字运行程序了。
make install
```



## 开启 SSH 服务
SSH 分客户端 openssh-client 和 openssh-server
如果你只是想登陆别的机器的 SSH 只需要安装 openssh-client（ubuntu 有默认安装），如果没有则： 
``` 
sudo apt-get install openssh-client
``` 
如果要使本机开放 SSH 服务就需要安装 openssh-server：
``` 
sudo apt-get install openssh-server
``` 
然后确认 sshserver 是否启动了：
``` 
ps -e | grep ssh
```
如果看到 sshd 那说明 ssh-server 已经启动了。
如果没有则可以这样启动：
``` 
sudo /etc/init.d/ssh start 
```
ssh-server 配置文件位于 `/etc/ssh/sshd_config`，在这里可以定义 SSH 的服务端口，默认端口是 22，你可以自己定义成其他端口号，如 222。
然后重启 SSH 服务：
``` 
sudo /etc/init.d/ssh stop  
sudo /etc/init.d/ssh start
```
然后使用以下方式登陆 SSH：
``` 
ssh tuns@192.168.xx.xx    # tuns 为 192.168.xx.xx 机器上的用户名，需要输入密码。
```
断开连接：
``` 
exit
```


## curl 的安装
```
$ sudo add-apt-repository ppa:costamagnagianfranco/ettercap-stable-backports  
$ sudo apt-get update  
$ sudo apt-get install curl  
```


## Node.JS 的安装

### 安装
#### 方法1 包方式安装(推荐方式)
```
$ sudo apt-get install -y python-software-properties software-properties-common
$ sudo add-apt-repository ppa:chris-lea/node.js
$ sudo apt-get update
$ sudo apt-get install nodejs  
$ nodejs -v

$ sudo apt-get install nodejs-legacy
$ node -v

# 安装 npm （nodejs默认npm，但是有时安装后没有安装npm，可以试着先升级看看是否安装了npm）
$ sudo apt-get install npm
$ npm -v
# 更新 npm 
$ sudo npm i -g npm  
```

#### 方法2 源码方式安装
> http://blog.csdn.net/awj3584/article/details/18401539  

```
# 下载源码包
wget https://nodejs.org/dist/node-latest.tar.gz

# 首先确保系统安装来 python,gcc,g++, 如果没有则安装：
$ sudo apt-get install python
$ sudo apt-get install build-essential
$ sudo apt-get install gcc
$ sudo apt-get install g++

# 解压
$ tar -xzf node-latest.tar.gz
$ cd node-latest.tar.gz

# 编译并安装
$ ./configure
$ make
$ make install

# 测试安装成功
$ node -v  
```

### 卸载
```
$ sudo apt-get autoremove nodejs-legacy
$ sudo apt-get autoremove nodejs
$ sudo rm -fr /usr/local/bin/node (which node 命令查看路经)
此时输入 node -v,没有打印版本号.
```

### 更新
```
$ sudo chmod -R 777 /usr/local
$ sudo npm install -g n
$ n stable
```
如果此时报：
```
cp: 无法获取 "/usr/local/n/versions/node/0.10.40/bin" 的文件状态 (stat): 没有那个文件或目录
cp: 无法获取 "/usr/local/n/versions/node/0.10.40/lib" 的文件状态 (stat): 没有那个文件或目录
cp: 无法获取 "/usr/local/n/versions/node/0.10.40/share" 的文件状态 (stat): 没有那个文件或目录
```
删除 `/usr/local/n/versions/node/` 目录，重试
```
$ sudo rm -fr /usr/local/n/versions/node/
```
出现下面的内容，说明更新成功：
```

     install : node-v8.4.0
       mkdir : /usr/local/n/versions/node/8.4.0
       fetch : https://nodejs.org/dist/v8.4.0/node-v8.4.0-linux-x64.tar.gz
######################################################################## 100.0%
   installed : v8.4.0

```
此时输入 `node -v`,打印：
```
v8.4.0
```

### 使用淘宝源
```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```


## Yarn 的安装
```
$ sudo npm install -g yarn
$ yarn -v
```


## Bower 的安装
```
# npm 方式安装
$ sudo npm install -g bower
$ bower -v

# yarn 方式安装
$
$ bower -v  
```


## Gulp 的安装
```
$ npm install --global gulp-cli
```



## Lantern 的安装
下载安装包（https://github.com/getlantern/lantern），Deepin 操作系统也可以下载 Ubuntu 的安装包。

安装：
```
$ sudo dpkg -i lantern-installer-64-bit.deb  
```
Deepin 安装好之后启动不起来，解决办法：
```
$ sudo apt-get install libappindicator3-dev -y  
```


## Shadowsocks 的安装

> [Ubuntu下ss的安装与使用](https://www.cnblogs.com/Dumblidor/p/5450248.html)  

```  
$ sudo apt-get install python-pip
$ sudo pip install shadowsocks
$ sslocal -s 1.1.1.1 -p 8388 -k "your passwd" -b 127.0.0.1 -l 1080
```
-s 后面跟你的服务器 ip ， -p 后面跟你远程端口号（默认 8388），-k 后面跟你的密码（写在双引号之间），其他的用默认选项就好。


## 基于 L2TP 的 VPN  

> [在Ubuntu使用基于L2TP的VPN设置](http://blog.csdn.net/xiaoreqing/article/details/50968467) 



## Google Chrome 的安装
> [Ubuntu14.04下安装google chrome浏览器](http://blog.csdn.net/xuwenneng/article/details/52316743)
```
$ wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
dpkg -i google-chrome-stable_current_amd64.deb
```
安装时若提示：
```
dpkg: 依赖关系问题使得 google-chrome-stable 的配置工作不能继续：
 google-chrome-stable 依赖于 libappindicator1；然而：
  未安装软件包 libappindicator1。
```
解决办法：
```
$ sudo apt-get -f install libappindicator1 libindicator7
``` 

#### 登陆谷歌账号
从谷歌浏览器打开“拓展程序”页面，将[谷歌访问助手-2.3.0-Crx4Chrome.com.crx](/attachments/谷歌访问助手-2.3.0-Crx4Chrome.com.crx)拖到页面上，然后就可以使用谷歌了。 


## 搜狗输入法 for Linux 的安装
> [Ubuntu 16.04 LTS安装sogou输入法详解](http://blog.csdn.net/qq_21792169/article/details/53152700)
>  
> [Ubuntu 16.04下安装sogou 拼音输入法的错误问题](http://blog.csdn.net/blueheart20/article/details/51901867)

安装依赖包：
```
sudo apt install libopencc1 fcitx-libs fcitx-libs-qt fonts-droid-fallback libqtwebkit4 libqt4-opengl
```

安装操作系统选择中文时默认安装了 fcitx，如果没有，则手动安装：
```
$ sudo add-apt-repository ppa:fcitx-team/nightly
$ sudo apt-get update
$ sudo apt-get -f install fcitx
$ fcitx -v
```

到 [搜狗输入法官网](https://pinyin.sogou.com/linux/) 下载 Linux 版搜狗输入法，并安装：
```
$ sudo dpkg -i sogoupinyin_2.2.0.0102_amd64.deb  
```
重启电脑：
```
sudo reboot 
``` 
搜狗输入法只能输入英文不能输入中文：    
打开 Home 目录窗口，按 `Ctrl + H` 快捷键查看当前目录的隐藏文件和文件夹。找到 `.config` 文件夹，删除其中的 `SogouPY`，`SogouPY.users` 和 `sogou-qimpanel` 这三个文件夹，重启电脑。 


## IntelliJ IDEA 的安装
下载安装包

解压

运行启动脚本（要先安装 jdk）:
```
$ ./bin/idea.sh
```


## Eclipse 的安装
进入下载页面 [https://www.1eclipse.org/downloads/packages/](https://www.eclipse.org/downloads/packages/)
选择合适的系统版本点击下载.


## Atom 的安装
```
$ sudo add-apt-repository ppa:webupd8team/atom
$ sudo apt-get update  
$ sudo apt-get install atom
```


## WPS 
``` 
wget http://kdl.cc.ksosoft.com/wps-community/download/a21/wps-office_10.1.0.5672~a21_amd64.deb
sudo dpkg -i wps-office_10.1.0.5672_a21_amd64.deb 
```
如果出现下面的错误:
``` 
dpkg: 依赖关系问题使得 wps-office 的配置工作不能继续：
 wps-office 依赖于 libpng12-0；然而：
  未安装软件包 libpng12-0。 
``` 
> [Linux for Ubuntu 解决WPS安装缺少libpng12-0的问题](https://blog.csdn.net/tydyz/article/details/74991048)

``` 
sudo dpkg -i libpng12-0_1.2.54-1ubuntu1_amd64.deb
``` 
启动 WPS 提示 No necessary symbol fonts : 
> [WPS for Linux（ubuntu）字体配置(字体缺失解决办法)](https://www.cnblogs.com/liangml/p/5969404.html) 



## Ubuntu 系统升级 
> [将 Ubuntu 16.04 LTS 升级到 Ubuntu 18.04 LTS](https://jingyan.baidu.com/article/2f9b480de8b64941cb6cc2a1.html) 

``` 
# 建议升级之前更新/升级所有已安装的软件包:
sudo apt update 
sudo apt dist-upgrade 
sudo apt autoremove 
  
# 安装 Ubuntu update manager: 
sudo apt-get install update-manager-core
   
# 打开 update-manager 配置文件并确保提示行设置为 lts
# sudo vim /etc/update-manager/release-upgrades 
  
# 执行升级命令:
sudo do-release-upgrade -d
  
# 升级系统后重启电脑
``` 

> [最新的ubuntu 18.04触控板右键失灵的解决方法](https://blog.csdn.net/qq_36317016/article/details/80143557)
``` 
gsettings set org.gnome.desktop.peripherals.touchpad click-method areas 
```


## 卸载软件和缓存 



## TLP 的安装 
TLP 是一款优化电源设置的软件，可以延长电池寿命。
``` 
sudo add-apt-repository -y ppa:linrunner/tlp 
sudo apt-get update 
sudo apt-get install -y tlp tlp-rdw tp-smapi-dkms acpi-call-tools 
sudo tlp start
``` 