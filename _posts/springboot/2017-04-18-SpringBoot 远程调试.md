

#### 启动远程服务器 jar 包

```
[root@centos6 agent2]# nohup java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n -Djava.library.path=runtime/ -jar nxms-collector.agent-1.0-SNAPSHOT.jar &
```

#### 查看

```
[root@centos6 agent2]# tail -f nohup.out 
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.1.RELEASE)

过滤器初始化
Listening for transport dt_socket at address: 8000

```

#### 启动远程调试

1. Edit Configurations  —> Add New Configuration —> Remote
2. Host 填写的是远程服务器的 IP 地址，远程调试端口8000（可更改）。
3. 在源码上打断点。
4. debug模式启动项目。