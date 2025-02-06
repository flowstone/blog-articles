---
title: "Java多线程-线程通信"
date: 2019-09-03 12:42:43
draft: false
categories: "Java"
---

## 通信的方式
要想实现多个线程之间的协同，如：线程执行先后顺序、获取某个线程执行的结果等等。涉及到线程之间的相互通信，分为下面四类：
* 文件共享
* 网络共享
* 共享变量
* JDK提供的线程协调API
    * suspend/resume、wait/notify、park/unpark

### 文件共享
![](https://ueyao.github.io/image-hosting/blog/2019/9/thread-communication-01.png)

``` java
public class MainTest {

  public static void main(String[] args) {
    // 线程1 - 写入数据
    new Thread(() -> {
      try {
        while (true) {
          Files.write(Paths.get("test.log"),
          content = "当前时间" + String.valueOf(System.currentTimeMillis()));
          Thread.sleep(1000L);
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
    }).start();

    // 线程2 - 读取数据
    new Thread(() -> {
      try {
        while (true) {
          Thread.sleep(1000L);
          byte[] allBytes = Files.readAllBytes(Paths.get("test.log"));
          System.out.println(new String(allBytes));
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
    }).start();
  }
}
```

### 变量共享
![](https://ueyao.github.io/image-hosting/blog/2019/9/thread-communication-02.png)

``` java
public class MainTest {
  // 共享变量
  public static String content = "空";

  public static void main(String[] args) {
    // 线程1 - 写入数据
    new Thread(() -> {
      try {
        while (true) {
          content = "当前时间" + String.valueOf(System.currentTimeMillis());
          Thread.sleep(1000L);
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
    }).start();

    // 线程2 - 读取数据
    new Thread(() -> {
      try {
        while (true) {
          Thread.sleep(1000L);
          System.out.println(content);
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
    }).start();
  }
}
```
### 网络共享

### 线程协作－JDK API

JDK中对于需要多线程协作完成某一任务的场景，提供了对应API支持。

多线程协作的典型场景是：生产者－消费者模型。(线程阻塞、线程唤醒)

示例：线程１去买包子，没有包子，则不再执行。线程２生产出包子，通知线程－１继续执行。

![](https://ueyao.github.io/image-hosting/blog/2019/9/thread-communication-03.png)

#### API-被弃用的suspend和resume
作用：调用suspend挂起目标线程，通过resume可以恢复线程执行。
``` java
/** 包子店 */
public static Object baozidian = null;

/** 正常的suspend/resume */
public void suspendResumeTest() throws Exception {
    // 启动线程
    Thread consumerThread = new Thread(() -> {
        if (baozidian == null) { // 如果没包子，则进入等待
            System.out.println("1、进入等待");
            Thread.currentThread().suspend();
        }
        System.out.println("2、买到包子，回家");
    });
    consumerThread.start();
    // 3秒之后，生产一个包子
    Thread.sleep(3000L);
    baozidian = new Object();
    consumerThread.resume();
    System.out.println("3、通知消费者");
}
```
被弃用的主要原因是，容易写出不死锁的代码。所以用wait/notify和park/unpark机制对它进行替代

#### suspend和resume死锁示例
1、同步代码中使用

``` java
	/** 死锁的suspend/resume。 suspend并不会像wait一样释放锁，故此容易写出死锁代码 */
	public void suspendResumeDeadLockTest() throws Exception {
		// 启动线程
		Thread consumerThread = new Thread(() -> {
			if (baozidian == null) { // 如果没包子，则进入等待
				System.out.println("1、进入等待");
				// 当前线程拿到锁，然后挂起
				synchronized (this) {
					Thread.currentThread().suspend();
				}
			}
			System.out.println("2、买到包子，回家");
		});
		consumerThread.start();
		// 3秒之后，生产一个包子
		Thread.sleep(3000L);
		baozidian = new Object();
		// 争取到锁以后，再恢复consumerThread
		synchronized (this) {
			consumerThread.resume();
		}
		System.out.println("3、通知消费者");
	}
```
2、suspend比resume后执行
``` java
/** 导致程序永久挂起的suspend/resume */
	public void suspendResumeDeadLockTest2() throws Exception {
		// 启动线程
		Thread consumerThread = new Thread(() -> {
			if (baozidian == null) {
				System.out.println("1、没包子，进入等待");
				try { // 为这个线程加上一点延时
					Thread.sleep(5000L);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				// 这里的挂起执行在resume后面
				Thread.currentThread().suspend();
			}
			System.out.println("2、买到包子，回家");
		});
		consumerThread.start();
		// 3秒之后，生产一个包子
		Thread.sleep(3000L);
		baozidian = new Object();
		consumerThread.resume();
		System.out.println("3、通知消费者");
		consumerThread.join();
	}
```
#### ｗait/notify机制

这些方法只能由同一对象锁的持有者线程调用，也就是写在同步块里面，否则会抛出IllegalMonitorStateException异常。
wait方法导致当前线程等待，加入该对象的等待集合中，并且放弃当前持有的对象锁。
notify/notifyAll方法唤醒一个或所有正在等待这个对象锁的线程。
注意：虽然会wait自动解锁，但是对顺序有要求，如果在notify被调用之后，才开始wait方法的调用，线程会永远处于WAITING状态。
#### wait/notify代码示例
``` java
/** 正常的wait/notify */
	public void waitNotifyTest() throws Exception {
		// 启动线程
		new Thread(() -> {
			synchronized (this) {
				while (baozidian == null) { // 如果没包子，则进入等待
					try {
						System.out.println("1、进入等待");
						this.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			System.out.println("2、买到包子，回家");
		}).start();
		// 3秒之后，生产一个包子
		Thread.sleep(3000L);
		baozidian = new Object();
		synchronized (this) {
			this.notifyAll();
			System.out.println("3、通知消费者");
		}
	}
```

造成死锁的示例
``` java
/** 会导致程序永久等待的wait/notify */
	public void waitNotifyDeadLockTest() throws Exception {
		// 启动线程
		new Thread(() -> {
			if (baozidian == null) { // 如果没包子，则进入等待
				try {
					Thread.sleep(5000L);
				} catch (InterruptedException e1) {
					e1.printStackTrace();
				}
				synchronized (this) {
					try {
						System.out.println("1、进入等待");
						this.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}
			System.out.println("2、买到包子，回家");
		}).start();
		// 3秒之后，生产一个包子
		Thread.sleep(3000L);
		baozidian = new Object();
		synchronized (this) {
			this.notifyAll();
			System.out.println("3、通知消费者");
		}
	}
```

#### park/unpark机制
线程调用park则等待“许可”，unpark方法为指定线程提供“许可(permit)”
不要求park和unpark方法的调用顺序。
多次调用unpark之后，再调用park，线程会直接运行。
但不会叠加，也就是说，连续多次调用park方法，第一次会拿到"许可"直接运行,后续调用会进入等待。

``` java
/** 正常的park/unpark */
	public void parkUnparkTest() throws Exception {
		// 启动线程
		Thread consumerThread = new Thread(() -> {
			while (baozidian == null) { // 如果没包子，则进入等待
				System.out.println("1、进入等待");
				LockSupport.park();
			}
			System.out.println("2、买到包子，回家");
		});
		consumerThread.start();
		// 3秒之后，生产一个包子
		Thread.sleep(3000L);
		baozidian = new Object();
		LockSupport.unpark(consumerThread);
		System.out.println("3、通知消费者");
	}
```

造成死锁的示例
``` java
/** 死锁的park/unpark */
	public void parkUnparkDeadLockTest() throws Exception {
		// 启动线程
		Thread consumerThread = new Thread(() -> {
			if (baozidian == null) { // 如果没包子，则进入等待
				System.out.println("1、进入等待");
				// 当前线程拿到锁，然后挂起
				synchronized (this) {
					LockSupport.park();
				}
			}
			System.out.println("2、买到包子，回家");
		});
		consumerThread.start();
		// 3秒之后，生产一个包子
		Thread.sleep(3000L);
		baozidian = new Object();
		// 争取到锁以后，再恢复consumerThread
		synchronized (this) {
			LockSupport.unpark(consumerThread);
		}
		System.out.println("3、通知消费者");
	}
```
#### 伪唤醒
**警告!之前代码中用if语句来判断,是否进入等待状态,是错误的!**
官方建议**应该循环中检查等待条件**，原因是处于等待状态的线程可能会收到**错误警报和伪唤醒**，如果不在循环中检查等待条件，程序就会在没有满足结束条件的情况下退出。

伪唤醒是指线程并非因为notify、notifyall、unpark等api调用而唤醒，是更底层原因导致的。


