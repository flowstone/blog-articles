---
title: "Java多线程-线程状态"
date: 2019-08-25 12:40:14
draft: false
categories: "Java"
---

# 线程状态

6个状态定义：java.lang.Thread.State

1. New: 尚未启动的线程的线程状态。
2. Runnable: 可运行线程的线程状态，等待CPU调度。
3. Blocked: 线程阻塞等待监视器锁定的线程状态。处于synchronized同步代码块或方法中被阻塞。
4. Waiting: 等待线程的线程状态。下列不带超时的方式：Object.wait、Thread.join、LockSupport.park
5. Timed Waiting: 具有指定等待时间的等待线程的线程状态。下列超时的方式：Thread.sleep、Object.wait、Thread.join、LockSupport.parkNanos、LockSupport.parkUntil

![线程状态](https://ueyao.github.io/image-hosting/blog/2019/8/thread-state-01.png)

## 常见线程状态切换
### 新建->运行->终止
``` java
Thread thread1 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println("thread1当前状态：" + Thread.currentThread().getState().toString());
				System.out.println("thread1 执行了");
			}
		});
System.out.println("没调用start方法，thread1当前状态：" + thread1.getState().toString());
thread1.start();
Thread.sleep(2000L); // 等待thread1执行结束，再看状态
System.out.println("等待两秒，再看thread1当前状态：" + thread1.getState().toString());
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/thread-state-02.png)

### 新建->运行->等待->运行->终止
``` java
Thread thread2 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {// 将线程2移动到等待状态，1500后自动唤醒
					Thread.sleep(1500);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("thread2当前状态：" + Thread.currentThread().getState().toString());
				System.out.println("thread2 执行了");
			}
		});
System.out.println("没调用start方法，thread2当前状态：" + thread2.getState().toString());
thread2.start();
System.out.println("调用start方法，thread2当前状态：" + thread2.getState().toString());
Thread.sleep(200L); // 等待200毫秒，再看状态
System.out.println("等待200毫秒，再看thread2当前状态：" + thread2.getState().toString());
Thread.sleep(3000L); // 再等待3秒，让thread2执行完毕，再看状态
System.out.println("等待3秒，再看thread2当前状态：" + thread2.getState().toString());
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/thread-state-03.png)


### 新建->运行->阻塞->运行->终止
``` java
Thread thread3 = new Thread(new Runnable() {
			@Override
			public void run() {
				synchronized (Demo2.class) {
					System.out.println("thread3当前状态：" + Thread.currentThread().getState().toString());
					System.out.println("thread3 执行了");
				}
			}
		});
synchronized (Demo2.class) {
    System.out.println("没调用start方法，thread3当前状态：" + thread3.getState().toString());
    thread3.start();
    System.out.println("调用start方法，thread3当前状态：" + thread3.getState().toString());
    Thread.sleep(200L); // 等待200毫秒，再看状态
    System.out.println("等待200毫秒，再看thread3当前状态：" + thread3.getState().toString());
}
Thread.sleep(3000L); // 再等待3秒，让thread3执行完毕，再看状态
System.out.println("等待3秒，让thread3抢到锁，再看thread3当前状态：" + thread3.getState().toString());
```

![](https://ueyao.github.io/image-hosting/blog/2019/8/thread-state-04.png)