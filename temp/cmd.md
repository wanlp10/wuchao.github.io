刷新 DNS 解析缓存：ipconfig /flushdns

查看端口：netstat -aon|findstr 8080
 TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       6160（PID）

查看端口被占用的应用：tasklist | findstr 6160
java.exe                      6160 Console                    1    760,612 K

关闭占用端口的应用：taskkill /f /t /im java.exe


查看端口被占用的进程 netstat -aon|findstr "8080"（打印结果最后一列是进程号）
```
TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       4980

```
kill 占用端口的进程号 taskkill /pid xxxx -t -f
(https://www.cnblogs.com/xwer/p/7780571.html)


修改cmd窗口字符编码为UTF-8，命令行中执行：chcp 65001
切换回中文：chcp 936
这两条命令只在当前窗口生效，重启后恢复之前的编码。

切换cmd窗口字符编码有风险，例如切换过以后中文显示乱码，并且不能永久切换回原来模式，只能每次chcp 936。
