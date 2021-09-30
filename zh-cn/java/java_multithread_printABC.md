---
title: 'Java多线程 - 交替打印字符"A","B","C"'
created: '2021-09-29T07:30:23.659Z'
modified: '2021-09-30T06:02:22.517Z'
---

# Java多线程 - 交替打印字符"A","B","C"

有A，B，C三个线程，A线程输出"A"，B线程输出"B"，C线程输出"C",要求，同时启动三个线程，按顺序输出"ABC"，循环10次。

这类题主要考察线程协作相关知识点，这里给出我的思考过程。

## 思路一：volatile

三个线程无限循环，直到volatile修饰的变量值与当前线程要打印的值相匹配，则打印对应的字符，修改volatile要打印的下一个字符，同时退出循环；如果不匹配，线程短暂休眠后重新参与比较，以此类推。这里有一个取巧点，将线程名与要打印的字符故意设置为一样，减少维护一层映射关系。伪代码如下所示:
```java

// 保存字符的交替打印顺序
Map<String,String> destMap = new HashMap<>();
destMap.put("A","B");
destMap.put("B","C");
destMap.put("C","A");

// 待打印的字符
volatile String currentLetter = "A";

class PrintThread implements Runnable{
  @Override
  public void run(){
      //题目要求打印10次
      for(int i=0;i<10;i++){
          while(true){
            String threadName = Thread.currentThread().getName();
            //线程名与待打印的字符匹配则直接打印
            if(threadName.equals(currentLetter)){
                System.out.println(String.format("thread%s : 打印 %s",threadName,currentLetter));
                currentLetter = destMap.get(currentLetter);
                // 本次打印完直接退出while循环
                break;
            }else{
                //与待打印字符未匹配的线程临时休眠
                TimeUnit.MILLISECONDS.sleep(10);
            }
          }
      }
  } 
}

// new 3个线程并启动
new Thread(new PrintThread(),"A").start();
new Thread(new PrintThread(),"B").start();
new Thread(new PrintThread(),"C").start();

```
利用volatile可见性的特征，从实现上来看相对简单，因为没有锁开销和线程之间的协作，但会让CPU空转。

## 思路二：synchronized

为了避免CPU空转，我们利用锁来实现线程间的协作。

构造一个公共锁对象，来实现三个线程之间的互斥，在锁对象中创建一个成员变量用于保存待打印的字符。

与volatile实现类似，三个线程先抢锁，先持有锁对象的线程可以判断当前线程名与打印字符是否一致，如果一致，直接打印，修改下一个要打印的字符，**唤醒任意一个等待中的线程**，同时退出本次循环；如果不一致，当前线程等待被唤醒。

```java
// 保存字符的交替打印顺序
Map<String,String> destMap = new HashMap<>();
destMap.put("A","B");
destMap.put("B","C");
destMap.put("C","A");

/**
* 线程间共享的锁对象
*/
class LockObject{
    //当前待打印的字符
    private String currentLetter;

    public LockObject(String currentLetter){
        this.currentLetter = currentLetter;
    }

    public void setCurrentLetter(String currentLetter){
        this.currentLetter = currentLetter;
    }

    public String getCurrentLetter(){
        return this.currentLetter;
    }
}

class PrintThread implements Runnable{

    private LockObject lockObject;

    public class PrintThread(LockObject lockObject){
        this.lockObject = lockObject;
    }

    public LockObject getLockObject(){
        return this.lockObject;
    }

    @Override
    public void run(){
        //打印10次
        for(int i =0;i<10;i++){
            while(true){
                //尝试获取锁
                synchronized(this.lockObject){
                    //取到锁
                    String threadName = Thread.currentThread().getName();
                    //持有锁的线程与待打印字符匹配
                    if(this.lockObject.getCurrentLetter().equals(threadName)){
                        System.out.println(String.format("thread%s : 打印 %s",threadName,this.lockObject.getCurrentLetter()));
                        this.lockObject.setCurrentLetter(destMap.get(this.lockObject.getCurrentLetter()));
                        //打印完后唤醒其他线程
                        this.lockObject.notify();
                        //跳出本次循环
                        break;  
                    }else{
                        //持有锁的线程与待打印字符不匹配，当前线程进入阻塞
                        this.lockObject.wait();
                    }
                }
            }
        }
    }
}

//客户端
LockObject  lockObject = new LockObject("A");
new Thread(new PrintThread(lockObject),"A");
new Thread(new PrintThread(lockObject),"B");
new Thread(new PrintThread(lockObject),"C");
```
上面这个实现乍看起来能够满足要求，但实际上运行起来三个线程有很大的概率会挂住，谁也没法继续执行下去。我们简单分析一下：
1. A、B、C三个线程同时启动，假设现在线程A取到锁，线程B、C进入lockObject的等待队列
2. 线程A开始进入临界区代码并执行，先取线程名为"A"，锁对象中待打印的字符也是"A"，于是开始打印，修改锁对象中待打印的字符为"B"，唤醒等待队列中的线程并退出当前循环
3. 这时队列中只有B、C两个线程，假设现在唤醒的是线程C，C执行的时候发现待打印字符与线程名不匹配，于是线程C释放锁并再次进入等待队列；同时，线程A准备进入第二次打印，但需要先获取锁，于是线程A也要加入等待队列中
4. 这时A、B、C三个线程都进入了等待队列，并且不会再有机会被唤醒

以上是其中一种执行情况，发生这种现象的根本原因是 *this.lockObject.notify()* 在唤醒线程的时候有很强的随机性，我们没有办法控制它能唤醒哪个线程，如果线程A执行完，刚好唤醒的是线程B，线程B执行完，又刚好唤醒线程C，线程C执行完，又刚好唤醒线程A，依次类推，才能满足我们的要求。那有没有什么简单的办法能避免三个线程被挂住呢？我们知道三个线程被挂住主要是因为调用了对象的wait方法，所以我们从这里找突破口：

```java
//使用带超时时间的wait方法
this.lockObject.wait(10);
```
这时我们再来分析下，还是上面的执行流程：
5. 第4步三个线程都进入等待队列，但由于设置了超时时间，假设现在线程B持有的锁超时
6. 线程B拿到了锁，发现待打印字符和线程名匹配，于是打印字符，修改锁对象中的待打印字符为"C"，唤醒等待队列中的线程并退出当前循环
7. 在后面的过程中，我们会发现，只有线程C被唤醒，程序才会正常往下执行，无论是线程A还是线程B被唤醒，都会进入等待直到超时，以此类推

通过上面的分析，我们知道虽然线程被唤醒有很大的随机性，但只有我们指定的线程被唤醒程序才会正常执行，基本能满足题目要求。只是这样产生了多次无效的超时、再唤醒。那有没有什么办法能精准的实现线程A执行完，唤醒线程B，线程B执行完，唤醒线程C，线程C执行完，唤醒线程A，这应该是题目本来的意思。我们来看思路三。

## 思路三：ReentrantLock

为了能准确控制被唤醒的线程，我们引入ReentrantLock中的Condition。

创建三个条件变量conditionA、conditionB和conditionC，分别对应线程A、线程B和线程C，初始状态下，设置待打印的字符是"A"，三个线程抢锁，如果是线程B或线程C抢到锁，由于当前待打印的字符不是B，也不是C，所以conditionB或conditionC都会进入await状态，如果是线程A抢到锁，当前待打印字符与线程A匹配，则可以打印字符，然后修改待打印字符为B，接着就调用conditionB.signal()，实现精准唤醒线程B，最后退出本次循环，进行下一轮字符打印，依次类推。

我们用下面的伪代码来表示：

```java
//锁声明
ReentrantLock lock = new ReentrantLock();
//条件对象声明
Condition conditionA = lock.newCondition();
Condition conditionB = lock.newCondition();
Condition conditionC = lock.newCondition();

//条件对象等待集合
Map<String,Condition> awaitMap = new HashMap<>();
awaitMap.put("A",conditionA);
awaitMap.put("B",conditionB);
awaitMap.put("C",conditionC);

//条件对象唤醒集合
Map<String,Condition> signalMap = new HashMap<>();
signalMap.put("A",conditionB);
signalMap.put("B",conditionC);
signalMap.put("C",conditionA);

//待打印字符
String currentLetter = "A";

Map<String,String> destMap = new HashMap<>();
destMap.put("A","B");
destMap.put("B","C");
destMap.put("C","A");

class PrintThread implements Runnable{
    @Override
    public void run(){
        for(int i=0;i<10;i++){
            while(true){
              //加锁
              lock.lock();
              try{
                  String threadName = Thread.currentThread().getName();
                  //待打印字符匹配上
                  if(currentLetter.equals(threadName)){
                      //打印字符
                      System.out.println(String.format("thread%s : 打印 %s",threadName,currentLetter));
                      //设置下一个待打印字符
                      currentLetter = destMap.get(currentLetter);
                      //唤醒下一个待打印的线程
                      signalMap.get(threadName).signal();
                      //退出本次循环
                      break;
                  }else{
                      //未匹配则当前线程条件变量阻塞
                      awaitMap.get(threadName).await();
                  }
              }catch(InterruptedException e){
                  e.printStackTrace();
              }catch(Exception e1){
                  e1.printStackTrace();
              }finally{
                  //释放锁
                  lock.unlock();
              }
            }
        }
    }
}

//客户端启动线程
new Thread(new PrintThread(),"A").start();
new Thread(new PrintThread(),"B").start();
new Thread(new PrintThread(),"C").start();
```

以上三种实现思路，从执行时间来看，思路一由于CPU空转耗费时间，思路二因为抢锁的随机性也会额外消耗一部分时间，只有思路三能够精准依次唤醒对应的线程，实现由规律的字符打印。下面给出了三种思路的可执行源码，欢迎共同交流分析。

- 思路一
```java
package sample;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
/**
 * @author grapeqin
 */
public class PrintCharacterWithVolatileTest {

    public static final Integer BOUND = 10;

    final static Map<String, String> destMap = new HashMap<>();

    static volatile String currentCharacter = "A";

    static CountDownLatch countDownLatch = new CountDownLatch(3);

    static {
        destMap.put("A", "B");
        destMap.put("B", "C");
        destMap.put("C", "A");
    }


    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        new Thread(new PrintThread(), "A").start();
        new Thread(new PrintThread(), "B").start();
        new Thread(new PrintThread(), "C").start();
        countDownLatch.await();
        System.out.printf("类实现:%s 花费时间:%d ms",PrintCharacterWithVolatileTest.class.getName(),(System.currentTimeMillis()-start));
    }

    static class PrintThread implements Runnable {
        @Override
        public void run() {
            for (int i = 0; i < BOUND; i++) {
                while (true) {
                    String threadName = Thread.currentThread().getName();
                    //当前待打印字符与线程名匹配则打印同时切换
                    if (currentCharacter.equals(threadName)) {
                        System.out.println(String.format("thread%s : 打印 %s", threadName, currentCharacter));
                        currentCharacter = destMap.get(currentCharacter);
                        //打印完即跳出本次循环
                        break;
                    } else {
                        try {
                            TimeUnit.MICROSECONDS.sleep(1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
            countDownLatch.countDown();
        }
    }
}

```

- 思路二

```java
package sample;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CountDownLatch;

/**
 * @author grapeqin
 */
public class PrintCharacterWithSynchronizedTest {

    public static final Integer BOUND = 10;

    final static Map<String, String> destMap = new HashMap<>();

    static CountDownLatch countDownLatch = new CountDownLatch(3);

    static {
        destMap.put("A", "B");
        destMap.put("B", "C");
        destMap.put("C", "A");
    }


    public static void main(String[] args) throws InterruptedException {
        LockObject lockObject = new LockObject("A");
        long start = System.currentTimeMillis();
        new Thread(new PrintThread(lockObject), "A").start();
        new Thread(new PrintThread(lockObject), "B").start();
        new Thread(new PrintThread(lockObject), "C").start();
        countDownLatch.await();
        System.out.printf("类实现:%s 花费时间:%d ms", PrintCharacterWithSynchronizedTest.class.getName(),(System.currentTimeMillis()-start));
    }

    /**
     * 对象锁
     */
    static class LockObject {

        private String currentLetter;

        public LockObject(String currentLetter) {
            this.currentLetter = currentLetter;
        }

        public String getCurrentLetter() {
            return currentLetter;
        }

        public void setCurrentLetter(String currentLetter) {
            this.currentLetter = currentLetter;
        }
    }

    /**
     * 打印线程实现
     */
    static class PrintThread implements Runnable {

        private LockObject lockObject;

        public PrintThread(LockObject lockObject) {
            this.lockObject = lockObject;
        }

        @Override
        public void run() {
            for (int i = 0; i < BOUND; i++) {
                while (true) {
                    synchronized (this.lockObject) {
                        String threadName = Thread.currentThread().getName();
                        if (this.lockObject.getCurrentLetter().equals(threadName)) {
                            System.out.println(String.format("线程%s : 打印 %s", threadName, this.lockObject.currentLetter));
                            this.lockObject.setCurrentLetter(destMap.get(this.lockObject.getCurrentLetter()));
                            this.lockObject.notify();
                            //本次打印完成就退出
                            break;
                        } else {
                            try {
                                //这里一定要使用带超时时间的wait
                                this.lockObject.wait(10);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }
            countDownLatch.countDown();
        }
    }
}
```

- 思路三

```java
package sample;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author grapeqin
 */
public class PrintCharacterWithLockConditionTest {

    public static final Integer BOUND = 10;

    final static ReentrantLock LOCK = new ReentrantLock();
    final static Condition CONDITIONA = LOCK.newCondition();
    final static Condition CONDITIONB = LOCK.newCondition();
    final static Condition CONDITIONC = LOCK.newCondition();

    //待阻塞的线程集合
    final static Map<String, Condition> awaitMap = new HashMap<>();

    //待唤醒的线程集合
    final static Map<String, Condition> signalMap = new HashMap<>();

    final static Map<String, String> destMap = new HashMap<>();

    static String currentCharacter = "A";

    static CountDownLatch countDownLatch = new CountDownLatch(3);

    static {
        awaitMap.put("A", CONDITIONA);
        awaitMap.put("B", CONDITIONB);
        awaitMap.put("C", CONDITIONC);

        signalMap.put("A", CONDITIONB);
        signalMap.put("B", CONDITIONC);
        signalMap.put("C", CONDITIONA);

        destMap.put("A", "B");
        destMap.put("B", "C");
        destMap.put("C", "A");
    }


    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        new Thread(new PrintThread(), "A").start();
        new Thread(new PrintThread(), "B").start();
        new Thread(new PrintThread(), "C").start();
        countDownLatch.await();
        System.out.printf("类实现:%s 花费时间:%d ms",PrintCharacterWithLockConditionTest.class.getName(),(System.currentTimeMillis()-start));
    }

    static class PrintThread implements Runnable {
        @Override
        public void run() {
            for (int i = 0; i < BOUND; i++) {
                while (true) {
                    LOCK.lock();
                    try {
                        String threadName = Thread.currentThread().getName();
                        if (currentCharacter.equals(threadName)) {
                            System.out.println(String.format("thread%s : 打印 %s", threadName, threadName));
                            currentCharacter = destMap.get(threadName);
                            signalMap.get(threadName).signal();
                            //退出循环
                            break;
                        } else {
                            awaitMap.get(threadName).await();
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        LOCK.unlock();
                    }
                }
            }
            countDownLatch.countDown();
        }
    }
}
```
