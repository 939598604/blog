---
title: javaEE教程
date: 2019-08-09 20:32:59
password: 123
abstract: 欢迎来到test, 请输入密码.
message: 欢迎来到test, 请输入密码.
toc: false
mathjax: false
tags:  [汇总] 
category: 汇总
---



## 多线程

### Synchronized

#### Synchronized使用方式有几种

Synchronized可以使用在三个地方：

1. 非静态方法
2. 静态方法
3. 代码块

首先我们先看一个普通的多线程方法

```
public class Main6  {

    public void method1() {
        System.out.println("method1 start");
        try {
            System.out.println("method1 exec");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("method1 completion");
    }

    public void method2() {
        System.out.println("method2 start");
        try {
            System.out.println("method2 exec");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("method2 completion");
    }

    public static void main(String[] args) {
        Main6 m1 = new Main6();
        Main6 m2 = new Main6();

        new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method1();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method2();
            }
        }).start();
    }
}
```

在这个方法执行的时候，method1()和method2()并行运行，但由于method2()的运行时间短，因此总是会先运行结束。 

1.非静态方法 

```
public class Main6  {

    public synchronized void method1() {
        System.out.println("method1 start");
        try {
            System.out.println("method1 exec");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("method1 completion");
    }

    public synchronized void method2() {
        System.out.println("method2 start");
        try {
            System.out.println("method2 exec");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("method2 completion");
    }

    public static void main(String[] args) {
        Main6 m1 = new Main6();
        Main6 m2 = new Main6();

        new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method1();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method2();
            }
        }).start();
    }
}
```

在非静态方法上添加synchronized关键字，当我们使用同一个对象分别在不同线程中调用该方法时候，会出现如下顺序的结果：

method1 start
method1 exec
method1 completion
method2 start
method2 exec
method2 completion

这是由于synchronized会获取当前调用对象的锁，当线程1在执行m1.method1();时候获取到了m1对象的锁，线程2企图执行m1.method2()方法时发现m1的锁已经被其他线程获取，因此会进入阻塞状态，直到线程1对method1()方法执行完。

注意，由于调用线程1的start()方法并不是一定会立即开始执行线程1，它先会与main线程争夺cpu等资源，因此有可能method2先执行，method1进入阻塞，直到method2执行完成后才会开始执行。

同时由于synchronized使用在非静态方法上获取的是当前对象的锁，因此假如线程1和2分别使用的是两个不同对象的mthod1方法和method2方法，则不会发生阻塞。

2.静态方法

```
public class Main6  {

    public static synchronized void method1() {
        System.out.println("method1 start");
        try {
            System.out.println("method1 exec");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("method1 completion");
    }

    public static synchronized void method2() {
        System.out.println("method2 start");
        try {
            System.out.println("method2 exec");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("method2 completion");
    }

    public static void main(String[] args) {
        Main6 m1 = new Main6();
        Main6 m2 = new Main6();

        new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method1();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                m2.method2();
            }
        }).start();
    }
}
```

静态方法与非静态方法的不同处在于方法是属于类，而不是属于当前对象。

这样即使我们调用的是不同对象的method1方法和method2方法，但是它们获取的锁是类对象的锁(Class对象的锁)，因此，不论使用的是哪个对象的method1和method2，均会发生线程阻塞的情况。只有当一个线程中的静态synchronized执行完成后才会执行另一个线程中的静态synchronized方法。

3.代码块

```
public class Main6  {

    public void method1() {
        System.out.println("method1 start");
        synchronized(this) {
            try {
                System.out.println("method1 exec");
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("method1 completion");
    }

    public void method2() {
        System.out.println("method2 start");
        synchronized (this) {
            try {
                System.out.println("method2 exec");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("method2 completion");
    }

    public static void main(String[] args) {
        Main6 m1 = new Main6();new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method1();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                m1.method2();
            }
        }).start();
    }
}
```

输出的结果可能如下：

method1 start
method1 exec
method2 start
method1 completion
method2 exec
method2 completion

当synchronized被使用在代码块上时，它所获取的锁是括号中接收的对象的锁(如样例代码中的this指代的是当前调用它的对象)，这样，假设method1方法正在执行，并且进入代码块休眠3秒，这时method2获取cpu，开始执行第一行代码打印"method2 start"，然后遇见代码块，尝试获取this对象的锁，却发现已经被method1获取(sleet不释放锁)，因此method2进入阻塞状态，直到method1执行完毕后释放锁，method2获取锁开始执行代码块。

#### synchronized的案例分析

**（1）标准访问，先打印短信还是邮件**

```
public class Phone {
    public synchronized void sendSMS(){
            System.out.println("----sendSMS");
    }
    public synchronized void sendEmail(){
        System.out.println("----sendEmail");
    }
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{phone.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone.sendEmail();},"BB").start();
    }
}
```

----sendSMS
----sendEmail

由于方法上的锁是this，同样的phone对象，A先进入方法，获取到锁，B只能等待

**（2）短信方法内停4秒，先打印短信还是邮件**

```
public class Phone {
    public synchronized void sendSMS(){
        try {
            Thread.sleep(4000);
            System.out.println("----sendSMS");
        }catch (Exception e){}
    }
    public synchronized void sendEmail(){
        System.out.println("----sendEmail");
    }
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{phone.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone.sendEmail();},"BB").start();
    }
}
```

----sendSMS
----sendEmail

由于方法上的锁是this，同样的phone对象，A先进入方法，获取到锁，且sleep方法是不释放锁的，B只能等待A先打印sendEmail

**（3）新增的hello方法，先打印短信还是hello**

```
public class Phone {
    public synchronized void sendSMS(){
        try {
            Thread.sleep(4000);
            System.out.println("----sendSMS");
        }catch (Exception e){ }
    }
    public void getHello(){
        System.out.println("----getHello");
    }

    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{phone.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone.getHello();},"BB").start();
    }
}
```

----getHello

----sendSMS

没有和锁有关系

**（4）现有2部手机，先打印短信还是邮件**

```
public class Phone {
    public synchronized void sendSMS(){
        try {
            Thread.sleep(4000);
            System.out.println("----sendSMS");
        }catch (Exception e){ }
    }
    public synchronized void sendEmail(){
        System.out.println("----sendEmail");
    }
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{phone1.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone2.sendEmail();},"BB").start();
    }
}
```

----sendEmail
----sendSMS

2部手机，由于方法上的锁是this，不同的phone对象，各自维护，sendSMS睡眠了4秒，先打印sendEmail

**（4）两个静态同步方法，1部手机，先打印短信还是邮件**

```
public class Phone {
    public static synchronized void sendSMS(){
        try {
            Thread.sleep(4000);
            System.out.println("----sendSMS");
        }catch (Exception e){ }
    }
    public static synchronized void sendEmail(){
        System.out.println("----sendEmail");
    }
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        new Thread(()->{phone1.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone1.sendEmail();},"BB").start();
    }
}
```

----sendSMS

----sendEmail

静态方法的锁是类的class字节码，只有一份，所以A先进入方法，获取到锁，B只能等待A

**（4）两个静态同步方法，2部手机，先打印短信还是邮件**

```
public class Phone {
    public static synchronized void sendSMS(){
        try {
            Thread.sleep(4000);
            System.out.println("----sendSMS");
        }catch (Exception e){ }
    }
    public static synchronized void sendEmail(){
        System.out.println("----sendEmail");
    }
    public void getHello(){
        System.out.println("----getHello");
    }
    public static void main(String[] args) throws InterruptedException {
        Phone phone1 = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{phone1.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone2.sendEmail();},"BB").start();
    }
}
```

----sendSMS

----sendEmail

静态方法的锁是类的class字节码，虽然是不同的对象调用，但是字节码还是只有一份，所以A先进入方法，获取到锁，B只能等待A

**（5）一个静态方法，一个普通方法，先打印短信还是邮件**

```
public class Phone {
    public static synchronized void sendSMS(){
        try {
            Thread.sleep(4000);
            System.out.println("----sendSMS");
        }catch (Exception e){ }
    }
    public synchronized void sendEmail(){
        System.out.println("----sendEmail");
    }
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{phone.sendSMS();},"AA").start();
        Thread.sleep(1000); //让A先启动
        new Thread(()->{phone.sendEmail();},"BB").start();
    }
}
```







#### [synchronized(this) 与synchronized(class) 之间的区别](https://www.cnblogs.com/huansky/p/8869888.html)

###  ReentrantLock





####  ReentrantReadWriteLock读写锁获取数据

1个线程写数据，100个线程读取数据

```
public class ReentrantReadWriteTest {
    ReentrantReadWriteLock lock=new ReentrantReadWriteLock();
    private Object obj;
    public void write(Object obj){
        lock.writeLock().lock();
        this.obj=obj;
        System.out.println(Thread.currentThread().getName()+"set--"+this.obj);
        lock.writeLock().unlock();
    }
    public void get(){
        lock.readLock().lock();
        System.out.println(Thread.currentThread().getName()+"--get---"+this.obj);
        lock.readLock().unlock();
    }
    public static void main(String[] args){
        ReentrantReadWriteTest rw = new ReentrantReadWriteTest();
        new Thread(()->{rw.write("测试");},"write").start();
        for (int i = 0; i < 100; i++) {
            new Thread(()->{rw.get();},"read--"+i).start();
        }
    }
}
```



###  CountDownLatch

**CoundDownLatch**这个类能够**使一个线程等待其他线程完成各自的工作后再执行**。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。 

编写20个同学离开教室之后，班长才锁门

```
public class CountDownLatchTest {
    public static void main(String[] args) {
        CountDownLatch cd = new CountDownLatch(20);
        for (int i = 0; i < 20; i++) {
            new Thread(()->{
                try {
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName()+"离开教室");
                    cd.countDown();
                } catch (InterruptedException e) {}
            },"线程"+i).start();
        }
        new Thread(()->{
            try {
                cd.await();
            } catch (InterruptedException e) {}
            System.out.println("班长锁门");
        },"线程").start();
    }
}
```

###   CyclicBarrier

**CyclicBarrier**和**CountDownLatch**是非常类似的，**CyclicBarrier**核心的概念是在于设置一个等待线程的数量边界，到达了此边界之后进行执行。

**CyclicBarrier**类是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点（Common Barrier Point）。

**CyclicBarrier**类是一种同步机制，它能够对处理一些算法的线程实现同。换句话讲，它就是一个所有线程必须等待的一个栅栏，直到所有线程都到达这里，然后所有线程才可以继续做其他事情。







###   多线程代码编写

####  （一）依次打印数字和字母

 * 12A34B56C78D910E1112F1314G1516H1718I1920J2122K2324L2526M2728N2930O3132P3334Q3536R3738S3940T4142U4344V4546W4748X4950Y5152Z

   **（1）使用等待唤醒机制**

```
package com.gzstrong.t;
public class NumChar {
    private int n=1;
    private int e=65;
    private boolean flag=true;
    public synchronized  void getNum(){
        for (int i=n;i<53;i=i+2) {
            try {
                while (!flag){
                    this.wait();
                }
                System.out.print(i+""+(i+1));
                flag=false;
            } catch (InterruptedException e1) {
                e1.printStackTrace();
            }
            this.notify();
        }
    }

    public synchronized  void getChar() {
        for (int i=e;i<91;i++) {
            try {
                while (flag){
                    this.wait();
                }
                System.out.print((char)i);
                flag=true;
            } catch (InterruptedException e1) {
                e1.printStackTrace();
            }
            this.notify();
        }
    }

    public static void main(String[] args) {
        NumChar numChar = new NumChar();
        new Thread(()->{
            numChar.getNum();
        }).start();
        new Thread(()->{
            numChar.getChar();
        }).start();
    }
}
```

**（2）使用ReentrantLock方式**

```
package com.gzstrong.t;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class NumCharLock {
    private int n=1;
    private int e=65;
    private boolean flag=true;
    Lock lock=new ReentrantLock();
    Condition condition = lock.newCondition();
    public void getNum(){
        lock.lock();
        for (int i=n;i<53;i=i+2) {
            while (!flag){
                try {
                    condition.await();
                } catch (InterruptedException e1) {
                    e1.printStackTrace();
                }
            }
            System.out.print(i+""+(i+1));
            flag=false;
            condition.signal();
        }
        lock.unlock();
    }

    public  void getChar() {
        lock.lock();
        for (int i=e;i<91;i++) {
            while (flag){
                try {
                    condition.await();
                } catch (InterruptedException e1) {
                    e1.printStackTrace();
                }
            }
            System.out.print((char)i);
            flag=true;
            condition.signal();
        }
        lock.unlock();
    }

    public static void main(String[] args) {
        NumCharLock numAA = new NumCharLock();
        new Thread(()->{
            numAA.getNum();
        }).start();
        new Thread(()->{
            numAA.getChar();
        }).start();
    }
}
```



##  Spring Cloud Alibaba学习

###  一.Nacos

#### 1.配置中心

springboot 启动报错

this is very likely to create a memory leak.



### 二.Apollo

####  (一)Apollo分布式配置中心部署

**1.下载源码**

https://github.com/ctripcorp/apollo

比较重要的几个项目：

- apollo-configservice：提供配置获取接口，提供配置更新推送接口，接口服务对象为Apollo客户端
- apollo-adminservice：提供配置管理接口，提供配置修改、发布等接口，接口服务对象为Portal，以及Eureka

- apollo-portal：提供Web界面供用户管理配置

- apollo-client：Apollo提供的客户端程序，为应用提供配置获取、实时更新等功能

**2.配置中心执行流程**

- 用户在Portal操作配置发布
- Portal调用Admin Service的接口操作发布
- Admin Service发布配置后，发送ReleaseMessage给各个Config Service
- Config Service收到ReleaseMessage后，通知对应的客户端

**3.数据库初始化**
下面的sql为大写格式，注意数据库的大小写敏感设置

- ApolloPortalDB：执行apollo-master\scripts\db\migration\portaldb\V1.0.0__initialization.sql
- ApolloConfigDB：执行apollo-master\scripts\db\migration\configdb\V1.0.0__initialization.sql

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904170710.png)

**4.调整项目配置**



### 三.Sentinel

#### (一)Spring Cloud Alibaba整合Sentinel流控

##### 1.部署Sentinel Dashboard

- 下载地址：[sentinel-dashboard-1.6.0.jar](https://github.com/alibaba/Sentinel/releases/download/1.6.0/sentinel-dashboard-1.6.0.jar)
- 其他版本：[Sentinel/releases](https://github.com/alibaba/Sentinel/releases)

```shell
java -jar sentinel-dashboard-1.6.0.jar  
```

要自定义端口号等内容的话，可以通过在启动命令中增加参数来调整，比如：`-Dserver.port=8888`

默认情况下，sentinel-dashboard以8080端口启动，所以可以通过访问：`localhost:8080`来验证是否已经启动成功 

> **注意**：只有1.6.0及以上版本，才有这个简单的登录页面。默认用户名和密码都是`sentinel`。对于用户登录的相关配置可以在启动命令中增加下面的参数来进行配置：

- `-Dsentinel.dashboard.auth.username=sentinel`: 用于指定控制台的登录用户名为 sentinel；
- `-Dsentinel.dashboard.auth.password=123456`: 用于指定控制台的登录密码为 123456；如果省略这两个参数，默认用户和密码均为 sentinel
- `-Dserver.servlet.session.timeout=7200`: 用于指定 Spring Boot 服务端 session 的过期时间，如 7200 表示 7200 秒；60m 表示 60 分钟，默认为 30 分钟； 

#####  2.整合Sentinel

（1）在gzstrong-system应用的`build.gradle`中引入Spring Cloud Alibaba的Sentinel模块： 

```java
      compile("org.springframework.cloud:spring-cloud-starter-alibaba-sentinel:0.2.2.RELEASE")
```

（2）在Spring Cloud应用中通过`spring.cloud.sentinel.transport.dashboard`参数配置sentinel dashboard的访问地址，比如： 

```java
spring.cloud.sentinel.transport.dashboard=localhost:8080
```

（3）创建应用主类，并提供一个rest接口，比如： 

```java
package com.gzstrong;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableDiscoveryClient
@EnableFeignClients
public class SystemApp {
    public static void main(String[] args) {
        SpringApplication.run(SystemApp.class, args);
    }

    @Slf4j
    @RestController
    static class TestController {

        @SentinelResource(value="hello-anon",blockHandler="handleException",blockHandlerClass=ExceptionUtil.class)
        @GetMapping("/hello-anon")
        public String hello() {
            return "didispace.com";
        }

    }

    @Bean
    @ConditionalOnMissingBean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }

}
class ExceptionUtil {
    public static String handleException(BlockException ex) {
        return "扛不住了啊....";
    }
}
```

启动应用，然后通过postman或者curl访问几下`localhost:9097/hello-anon`接口 

##  Git使用

###  1.初始化仓库

**查看状态**

git status  查看工作区代码相对于暂存区的差别

###  2.移除版本控制

git rm -r --cached ".gradle/"

###  3.添加忽略文件规则

touch .gitignore

忽略文件的规则

```
#               表示此为注释,将被Git忽略
*.a             表示忽略所有 .a 结尾的文件
!lib.a          表示但lib.a除外
/TODO           表示仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/          表示忽略 build/目录下的所有文件，过滤整个build文件夹；
doc/*.txt       表示会忽略doc/notes.txt但不包括 doc/server/arch.txt
 
bin/:           表示忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件
/bin:           表示忽略根目录下的bin文件
/*.c:           表示忽略cat.c，不忽略 build/cat.c
debug/*.obj:    表示忽略debug/io.obj，不忽略 debug/common/io.obj和tools/debug/io.obj
**/foo:         表示忽略/foo,a/foo,a/b/foo等
a/**/b:         表示忽略a/b, a/x/b,a/x/y/b等
!/bin/run.sh    表示不忽略bin目录下的run.sh文件
*.log:          表示忽略所有 .log 文件
config.php:     表示忽略当前路径的 config.php 文件
 
/mtk/           表示过滤整个文件夹
*.zip           表示过滤所有.zip文件
/mtk/do.c       表示过滤某个具体文件
```

###  4.添加到暂存区

git add .   不加参数默认为将修改操作的文件和未跟踪新添加的文件添加到git系统的暂存区，注意不包括删除

###  5.提交到本地的版本库

git commit -m ‘message’  -m 参数表示注释

###  6.将本地版本库的分支推送到远程服务器



###  7.更新代码到本地仓库

git pull 

###  8.分支

git branch  查看分支

git chechout aaa 切换分支aaa   

git branck aaa 创建aaa分支

## JMeter并发

###  1 添加线程组

右键点击“测试计划” -> “添加” -> “Threads(Users)” -> “线程组” 

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904170759.png)

这里可以配置线程组名称，线程数，准备时长（Ramp-Up Period(in seconds)）循环次数，调度器等参数：

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904170826.png)

线程组参数详解： 

1. 线程数：设置多少个线程数  
2. Ramp-Up Period(in seconds)准备时长：线程数 需要多长时间全部启动。如果线程数为10，准备时长为2，那么需要2秒钟启动10个线程，也就是每秒钟启动5个线程。  一般为好理解设为1
3. 循环次数：每个线程发送请求的次数。如果线程数为10，循环次数为100，那么每个线程发送100次请求。总请求数为10*100=1000 。如果勾选了“永远”，那么所有线程会一直发送请求，一到选择停止运行脚本。  
4. Delay Thread creation until needed：直到需要时延迟线程的创建。  
5. 调度器：设置线程组启动的开始时间和结束时间(配置调度器时，需要勾选循环次数为永远)  持续时间（秒）：测试持续时间，会覆盖结束时间  启动延迟（秒）：测试延迟启动时间，会覆盖启动时间  启动时间：测试启动时间，启动延迟会覆盖它。当启动时间已过，手动只需测试时当前时间也会覆盖它。  结束时间：测试结束时间，持续时间会覆盖它。因为接口调试需要，我们暂时均使用默认设置，待后面真正执行性能测试时再回来配置

###  2 添加HTTP请求

右键点击“线程组” -> “添加” -> “Sampler” -> “HTTP请求” 

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904170855.png)

对于我们的接口<http://www.baidu.com/s?ie=utf-8&wd=jmeter>性能测试，可以参考下图填写：

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904170912.png)

 Http请求主要参数详解：

Web服务器  协议：向目标服务器发送HTTP请求协议，可以是HTTP或HTTPS，默认为HTTP  

服务器名称或IP ：HTTP请求发送的目标服务器名称或IP  

端口号：目标服务器的端口号，默认值为80  2.Http请求  方法：发送HTTP请求的方法，可用方法包括GET、POST、HEAD、PUT、OPTIONS、TRACE、DELETE等。  路径：目标URL路径（URL中去掉服务器地址、端口及参数后剩余部分）  Content encoding ：编码方式，默认为ISO-8859-1编码，这里配置为utf-8同请求一起发送参数  在请求中发送的URL参数，用户可以将URL中所有参数设置在本表中，表中每行为一个参数（对应URL中的 name=value），注意参数传入中文时需要勾选“编码”