## 死锁编码及定位分析

### 是什么

   死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力干涉那他们都将无法推进下去，如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。

   ```mermaid
   graph TD
    threadA(线程A)
    threadB(线程B)
    lockA((锁A))
    lockB((锁B))
    threadA--持有-->lockA
    threadB--试图获取-->lockA
    threadB--持有-->lockB
    threadA--试图获取-->lockB
   ```

### 产生死锁的主要原因

   - 系统资源不足
   - 进程运行推进的顺序不合适
   - 资源分配不当

### 死锁示例

   ```java
   package com.jian8.juc.thread;
   
   import java.util.concurrent.TimeUnit;
   
   /**
    * 死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力干涉那他们都将无法推进下去，
    */
   public class DeadLockDemo {
       public static void main(String[] args) {
           String lockA = "lockA";
           String lockB = "lockB";
           new Thread(new HoldThread(lockA,lockB),"Thread-AAA").start();
           new Thread(new HoldThread(lockB,lockA),"Thread-BBB").start();
       }
   }
   
   class HoldThread implements Runnable {
   
       private String lockA;
       private String lockB;
   
       public HoldThread(String lockA, String lockB) {
           this.lockA = lockA;
           this.lockB = lockB;
       }
   
       @Override
       public void run() {
           synchronized (lockA) {
               System.out.println(Thread.currentThread().getName() + "\t自己持有：" + lockA + "\t尝试获得：" + lockB);
               try {
                   TimeUnit.SECONDS.sleep(2);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               synchronized (lockB) {
                   System.out.println(Thread.currentThread().getName() + "\t自己持有：" + lockB + "\t尝试获得：" + lockA);
               }
           }
       }
   }
   
   ```

### 解决

   1. 使用`jps -l`定位进程号
   2. `jstack 进程号`找到死锁查看

#### 如何避免死锁

1. 避免多次加锁,尽量避免同一个线程对多个lock进行锁定,例如:主线程对a,b两个对象加锁,副线程同样对a,b两个线程加锁,就容易产生死锁
2. 尽量保持相同的加锁顺序,如果多个线程需要对多个 Lock 进行锁定，则应该保证它们以相同的顺序请求加锁。比如上面的死锁程序，主线程先对 A 对象的 Lock 加锁，再对 B 对象的 Lock 加锁；而副线程则先对 B 对象的 Lock 加锁，再对 A 对象的 Lock 加锁。这种加锁顺序很容易形成嵌套锁定，进而导致死锁。如果让主线程、副线程按照相同的顺序加锁，就可以避免这个问题。
3. 使用定时锁。程序在调用 acquire() 方法加锁时可指定 timeout 参数，该参数指定超过 timeout 秒后会自动释放对 Lock 的锁定，这样就可以解开死锁了。
4. 死锁检测。死锁检测是一种依靠算法机制来实现的死锁预防机制，它主要是针对那些不可能实现按序加锁，也不能使用定时锁的场景的。

