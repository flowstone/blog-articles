---
title: "Java多线程-线程中止"
date: 2019-08-26 12:41:06
draft: false
categories: "Java"
---

## 不正确的线程中止-Stop

Stop:中止线程，并且清除监控器锁的信息，但是可能导致
线程安全问题，JDK不建议用。
Destroy: JDK未实现该方法。

``` java
/**
 * @author simon
 */
public class StopThread extends Thread {
    private int i = 0, j = 0;
    @Override
    public void run() {
        synchronized (this) {
            // 增加同步锁，确保线程安全
            ++i;
            try {
                // 休眠10秒,模拟耗时操作
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            ++j;
        }
    }
    /** * 打印i和j */
    public void print() {
        System.out.println("i=" + i + " j=" + j);
    }
}
```

``` java
/**
 * @author simon
 * 示例3 - 线程stop强制性中止，破坏线程安全的示例
 */
public class Demo {
    public static void main(String[] args) throws InterruptedException {
        StopThread thread = new StopThread();
        thread.start();
        // 休眠1秒，确保i变量自增成功
        Thread.sleep(1000);
        // 暂停线程
        thread.stop(); // 错误的终止
        //thread.interrupt(); // @正确终止
        while (thread.isAlive()) {
            // 确保线程已经终止
        } // 输出结果
        thread.print();
    }
}
```
理想状态：要么自增成功i=1, j=1，要么自增失败i=0, j=0
真正程序执行结果：i=1, j=0

![](https://ueyao.github.io/image-hosting/blog/2019/thread-stop-01.png)

没有保证同步代码块里面数据的一致性，破坏了线程安全
stop方法直接停止线程

## 正确的线程中止-interrupt

如果目标线程在调用Object class的wait()、wait(long)或wait(long, int)方法、join()、join(long, int)或sleep(long, int)方法时被阻塞，那么Interrupt会生效，该线程的中断状态将被清除，抛出InterruptedException异常。

如果目标线程是被I/O或者NIO中的Channel所阻塞，同样，I/O操作会被中断或者返回特殊异常值。达到终止线程的目的。


如果以上条件都不满足，则会设置此线程的中断状态。

对Demo中的示例，stop()改成interrupt()后，最终输出为"i=1 j=1"，数据一致。

![](https://ueyao.github.io/image-hosting/blog/2019/thread-stop-02.png)

## 正确的线程中止-标志位

```java
/** 通过状态位来判断 */
public class Demo4 extends Thread {
  public volatile static boolean flag = true;

  public static void main(String[] args) throws InterruptedException {
    new Thread(() -> {
      try {
        while (flag) { // 判断是否运行
          System.out.println("程序运行中");
          Thread.sleep(1000L);
        }
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }).start();
    // 3秒之后，将状态标志改为False，代表不继续运行
    Thread.sleep(3000L);
    flag = false;
    System.out.println("程序运行结束");
  }
}
```
在上方代码逻辑中，增加一个判断，用来控制线程执行的中止。

![](https://ueyao.github.io/image-hosting/blog/2019/thread-stop-03.png)