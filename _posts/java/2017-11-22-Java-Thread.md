---
layout: post
title: Java Thread
category : [Java]
tagline: "Supporting tagline"
tags : [Java, Thread]
---
{% include JB/setup %}
# Java Thread
--- 

> [Future模式](https://www.cnblogs.com/uptownBoy/articles/1772483.html)
> [Java 中的 Runnable、Callable、Future、FutureTask 的区别与示例](http://blog.csdn.net/bboyfeiyu/article/details/24851847)

<!--break-->


```
package commands;

import org.crsh.cli.Command;
import org.crsh.cli.Option;
import org.crsh.cli.Usage;
import org.crsh.command.BaseCommand;
import org.springframework.util.StringUtils;

import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.concurrent.*;

public class Agent extends BaseCommand {

    @Usage("exec unix/linux/win command")
    @Command
    public String exec(@Usage("command") @Option(names = {"c", "command"}) String command, @Usage("format") @Option(names = {"f", "format"}) String format,
                       @Usage("exec grep command ex:-g win/linux/unix") @Option(names = {"g", "grep"}) String grep) {
        if (command != null) {
            if (StringUtils.isEmpty(format)) {
                format = "UTF-8";
            }

            Process p;
            ExecutorService executorService;
            try {
                if (!StringUtils.isEmpty(grep)) {
                    String[] commands = new String[3];
                    commands[2] = command;
                    if (grep.equals("win")) {
                        commands[0] = "cmd.exe";
                        commands[1] = "/c";
                    } else {
                        commands[0] = "/bin/sh";
                        commands[1] = "-c";
                    }
                    p = Runtime.getRuntime().exec(commands);
                } else {
                    p = Runtime.getRuntime().exec(command);
                }
                executorService = Executors.newSingleThreadExecutor();
                StreamCallback streamCallback = new StreamCallback(p.getInputStream(), format);
                FutureTask<String> feature = new FutureTask(streamCallback);
                executorService.submit(feature);
                return feature.get();
            } catch (Exception e) {
                return e.getMessage();
            }
        }
        return null;
    }

    class StreamRunnable implements Runnable {
        BufferedReader bReader = null;
        String type = null;
        String line = "";


        StreamRunnable(InputStream is, String _type) {
            try {
                bReader = new BufferedReader(new InputStreamReader(new BufferedInputStream(is), _type));
                type = _type;
            } catch (Exception ignored) {
            }
        }

        public void run() {
            try {
                String temp;
                while ((temp = bReader.readLine()) != null) {
                    if (temp != null) {
                        line += temp + "\r\n";
                    }
                }
                bReader.close();
            } catch (Exception ignored) {
            }
        }
    }

    class StreamCallback implements Callable<String> {

        BufferedReader bReader = null;
        String type = null;
        String line = "";

        StreamCallback(InputStream is, String _type) {
            try {
                bReader = new BufferedReader(new InputStreamReader(new BufferedInputStream(is), _type));
                type = _type;
            } catch (Exception ignored) {
            }
        }

        @Override
        public String call() throws Exception {
            try {
                String temp;
                while ((temp = bReader.readLine()) != null) {
                    if (temp != null) {
                        line += temp + "\r\n";
                    }
                }
                return line;
            } catch (Exception ignored) {
                return "";
            } finally {
                bReader.close();
            }
        }
    }
}
```


FAQ：

什么是线程同步?

当使用多个线程来访问同一个数据时，非常容易出现线程安全问题 (比如多个线程都在操作同一数据导致数据不一致), 所以我们用同步机制来解决这些问题。

实现同步机制有两个方法：

1.同步代码块：

synchronized(同一个数据){} 同一个数据：就是 N 条线程同时访问一个数据。

2.同步方法：

public synchronized 数据返回类型 方法名 (){}

就是使用 synchronized 来修饰某个方法，则该方法称为同步方法。对于同步方法而言，无需显示指定同步监视器，同步方法的同步监视器是 this 也就是该对象的本身（这里指的对象本身有点含糊，其实就是调用该同步方法的对象）通过使用同步方法，可非常方便的将某类变成线程安全的类，具有如下特征：

1，该类的对象可以被多个线程安全的访问。

2，每个线程调用该对象的任意方法之后，都将得到正确的结果。

3，每个线程调用该对象的任意方法之后，该对象状态依然保持合理状态。

注：synchronized 关键字可以修饰方法，也可以修饰代码块，但不能修饰构造器，属性等。

实现同步机制注意以下几点： 安全性高，性能低，在多线程用。性能高，安全性低，在单线程用。

1，不要对线程安全类的所有方法都进行同步，只对那些会改变共享资源方法的进行同步。

2，如果可变类有两种运行环境，当线程环境和多线程环境则应该为该可变类提供两种版本：线程安全版本和线程不安全版本 (没有同步方法和同步块)。在单线程中环境中，使用线程不安全版本以保证性能，在多线程中使用线程安全版本.

线程通讯：

为什么要使用线程通讯?

当使用 synchronized 来修饰某个共享资源时 (分同步代码块和同步方法两种情况）, 当某个线程获得共享资源的锁后就可以执行相应的代码段，直到该线程运行完该代码段后才释放对该 共享资源的锁，让其他线程有机会执行对该共享资源的修改。当某个线程占有某个共享资源的锁时，如果另外一个线程也想获得这把锁运行就需要使用 wait() 和 notify()/notifyAll() 方法来进行线程通讯了。

Java.lang.object 里的三个方法 wait() notify() notifyAll()

wait 方法导致当前线程等待，直到其他线程调用同步监视器的 notify 方法或 notifyAll 方法来唤醒该线程。

wait(mills) 方法

都是等待指定时间后自动苏醒，调用 wait 方法的当前线程会释放该同步监视器的锁定，可以不用 notify 或 notifyAll 方法把它唤醒。

notify()

唤醒在同步监视器上等待的单个线程，如果所有线程都在同步监视器上等待，则会选择唤醒其中一个线程，选择是任意性的，只有当前线程放弃对该同步监视器的锁定后，也就是使用 wait 方法后，才可以执行被唤醒的线程。

notifyAll() 方法

唤醒在同步监视器上等待的所有的线程。只用当前线程放弃对该同步监视器的锁定后，才可以执行被唤醒的线程。
