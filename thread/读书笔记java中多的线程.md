在Android经常会用到多线程，虽然多线程提高了性能，但也带来了一些复杂性:

1.需要使用Java中处理并发编程模型

2.需要确保在多线程环境中的数据一致性(同步)

3.需要设置任务的执行策略

线程基本概念:
其实软件运行的本质是指示硬件去做一些操作(包括展示一张图片，存储数据等)，这些指令有代码实现，由CPU按顺序执行，线程其实就是这些指令的高级定义。从应用的角度来看，一个线程是沿着Java的代码路径顺序执行的，在一个线程上按顺序执行的代码路径被称为一个任务，一个线程可以顺序执行一个或者多个任务。

线程的执行:

在Android中线程由java.lang.Thread来代表，当Thread调用start时，开始运行任务，当任务被执行完毕并且没有更多任务时，thread停止。thread的存活时间取决于任务的长短。Thread支持实现了java.lang.Runnable 接口的任务，最简单的例子:
```java
  private class MyTask implements Runnable {
    public void run() {
      int i = 0;//变量保存在线程本地堆栈上
    }
  }

  Thread myThread = new Thread(new MyTask());
  myThread.start();

```

从操作系统层面看，一个线程拥有一个指令指针和一个栈指针，指令指针每次都会指向下一条要执行的指令，而栈指针指向一块线程私有的内存区域(不能被其他线程读取)，用来存储线程本地数据。Cpu每次只是执行一条指令，但是系统通常有很多进程或者线程需要同时执行，比如adnroid同时运行多个app.如果按顺序执行每一个任务，那排到最后的任务也太惨了，用户需要等待很久。为了让用户觉得应用是同时在运行的，cpu就得在多个线程中分享运行时间(一个cpu实质上每次还是运行一个线程，但由于线程切换的时间很短，所以用户感觉不出来)。这就需要有个调度策略决定哪个线程要马上被运行以及运行多长时间,调度策略可以有很多种，但通常使用线程优先级来调度。高优先级线程会优先于低优先级线程被调用,并被赋予更多的运行时间.在java中线程的优先级从1(最低优先级)到10(最高优先级),如果不设置的话，默认为5:
  myThread.setPriority(8)
但是仅仅根据优先级来调度的话，那么低优先级的线程可能无法获得足够的运行时间(饥饿线程)，因此，调度策略还会考虑每个线程的处理时间，以此来更改运行的线程，线程的更改就是所谓的上下文切换，上下文切换会记录当前线程暂停运行时的数据和状态，以便下一次切换回来时恢复原先的状态。两个同时运行的线程  运行在一个处理器中，被分割成执行间隔的例子，如下所示:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/%E7%BA%BF%E7%A8%8B%E4%B8%8A%E4%B8%8B%E6%96%87%E5%88%87%E6%8D%A2%20(1).png)
```java
Thread T1 = new Thread(new MyTask());
    T1.start();

Thread T2 = new Thread(new MyTask());
    T2.start();
```
每个调度的点都消耗一些时间，用于cpu计算线程切换，在图中这个时间表示为C这个时间段

多线程应用:
对于多线程来说，因为应用代码可以分割成多个操作步骤，所以看起来就像是并行运行一样。如果执行线程的数量超过处理器的数量，那其实还不算真正的并行，只是通过上下文切换实现线程的分割运行，实际上每条指令还是顺序运行的。多线程虽然提高了运行效率，也带来了一些代价,包括增加了复杂度、增加了内存占用、运行顺序的不确定性，这些都需要应用程序去管理。

资源占用:
线程在处理器和内存方面会带来开销。每个线程都会分配一个私有内存区域，主要用来存储该线程的本地变量和方法运行时的参数。只要线程是存活的，它就会占据一定的系统资源，即使它此时处于空闲状态或者阻塞状态。而处理器占用是指，在每次的上文切换中，处理器需要计算、存储和恢复线程状态，越多的上下文切换，对性能的影响也就越大。

增加复杂度:
对于单线程来说，由于代码的执行是有序的，所以我们分析代码行为时是很容易的。但是一旦到多线程，因为线程的执行顺序和执行时间是不确定的，我们在处理的时候是很容易出错的，并且一旦出错，调试起来也很麻烦。

数据的不同步:
多线程执行顺序的不确定，导致对数据访问顺序的不确定性，如果一个变量被2个以上的线程共享，每个线程都可以改变它的值，那么这个最终的值是不好把握的。举个例子:线程t1和t2都能够修改变量sharedResource，访问的顺序是不确定的，它可能先被加或先被减。

```java
public class RaceCondition {
    int sharedResource = 0;
    public void startTwoThreads() {       
       Thread t1 = new Thread(new Runnable() {         
            @Override            
            public void run() {               
               sharedResource++;            
             }       
            });       
             t1.start();

              Thread t2 = new Thread(new Runnable() {           
                 @Override           
                  public void run() {              
                      sharedResource--;           
                     }        
                   });        
                   t2.start();   
                  }
    }

```

sharedResource暴露了一个竞争条件，它的结果会随着线程执行顺序的不同而不同，我们没法保证t1和t2哪个会先改变sharedResource的值。在这个例子中，更为细致的顺序是二进制指令的顺序，修改一块内存区域的值时包括读、修改、写入三个操作，而这三个操作都不是原子操作。线程的上下文切换可能在这三个指令之间发生，这样sharedResource的最终结果就取决于两个线程的6个指令操作的顺序，结果可能为0,-1或者1.第一个结果发生在第一个线程在第二个线程读取sharedResource之前写入。后两个结果发生在最先读取的都是初始化值0，最后的写入操作决定了最后的结果。因为有些数据的读写不应该被中断不然可能会出现上述情况，所以对于这样的一些数据应该在代码中提供原子区域代码块(原子操作，不被中断)，如果一个线程运行原子区域中的代码，其他想要访问相同代码块的线程将会被阻塞，直到没有线程在运行这个代码块。因此，Java中的原子区域是互斥的，因为它只允许访问一个线程，原子区域的创建有很多方法，最常见的就是使用关键字synchronized：
```java
  synchronized (this) {   
     sharedResource++;
    }
```
线程安全:

让多个线程共享相同的对象是一个很快捷的线程交互方式，但也引发了上文提到的线程安全的问题。如果一个被多个线程访问的对象，每次被线程访问时都是一个正确的状态，那就是线程安全的，这可以通过同步来实现，同步可以保证在一块代码块内当前只能有一个线程执行该代码块，这样的代码块称为临界区，并且它只能是原子操作。在java中同步是通过锁机制来实现的，判断临界区代码是否已经有线程在执行了，如果已经有线程在执行，其他想要执行该代码块的线程将会被阻塞。Android中线程锁机制包括:
1.对象锁:synchronized 关键字
2.显式锁定:java.util.concurrent.locks.ReentrantLock和java.util.concurrent.locks.ReentrantReadWriteLock

对象锁和java监听器:

synchronized关键字在每个Java对象中隐含可用的对象锁上运行,对象锁是互斥，所以保证了当前只有一个线程在运行关键代码块，对象锁类似于一个监听器，java监听器可以有三种状态建模:

挂起:
  线程在等待监听器被另一个线程释放
运行:
  唯一的一个线程持有该监听器，并且正在运行临界区
等待:
  线程在运行完临界区的全部代码之前自愿放弃对监听器的持有，让其他线程运行临界区，自身等待被系统唤起从而再次拥有该监听器

线程在这三种状态中转换的效果图:

！[](https://github.com/whyrookie/android_dev_skills/blob/master/images/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.png)

当线程运行被对象锁保护的代码块时，根据监听器的不同状态，线程会处于不同的过渡状态:

1.进入监听器:一个线程想要访问被对象锁保护的代码块，它已经进入了监视器中，但如果已经有线程拥有对象锁，那这个线程会被挂起，等待
2.请求锁:如果当前没有线程拥有这个监视器，那么一个被阻塞的线程会去请求获取锁，并且运行代码块，但如果有多个线程同时被阻塞，系统会通过调度策略来判断哪个线程该获得这个监视器
3.释放锁并且等待:线程通过调用 Object.wait() 将自己挂起，通常是因为在运行时有些条件尚未满足(比如io读取未结束)
4.在被唤起之后请求锁:等待的线程通常在其他线程调用 Object.notify() 或者 Object.notifyAll()后被唤起，并通过系统调度，可以再次获得监视器。
5.释放锁并且离开监视器:当代码块执行完毕后，线程离开，释放锁和监视器，以便其他线程可以获取。
下面的代码分别表示以上吴5种状态在代码中的位置.

```java
synchronized (this) { // (1)
     // Execute code (2)   
      wait(); // (3)   
       // Execute code (4) }
       // (5)

```

对象锁的结果不同级别:

1方法级别:
```java`
synchronized void changeState() {   
  sharedResource++;
}
```

2.代码块级别:
```java
void changeState() {   
   synchronized(this) {        
     sharedResource++;    
  }
 }
```

3.使用其他对象的对象锁的代码块级别:
```java

private final Object mLock = new Object();
void changeState() {    
  synchronized(mLock) {            
  sharedResource++;    
  }
}
```

4.对封闭类实例的内部锁进行操作的方法级别:
```java
synchronized static void changeState() {    
  staticSharedResource++;
}
```

5.在封闭类实例的对象锁上运行的块级别:
```java
static void changeState() {    
  synchronized(this.getClass()) {        
    staticSharedResource++;    
  }
}
```

代码块级别和方法级别代码中的this是同一个对象，但是使用代码块级别你可以更加精确地控制临界区，只关心你实际想要保护的状态，我们应该尽可能地缩小原子操作的范围，过大的原子操作范围会降低应用的性能。

我们还可以在一个类中使用其他对象的对象锁，一个应用应该尽可能地使用一个锁保护他的每一个状态，因此，如果一个类中有多个独立的状态，最好需要多个锁来提高性能.

使用显式的锁机制:
如果需要更加高级的锁机制，可以使用ReentrantLock和 ReentrantReadWriteLock替代synchronized，例子:
```java
int sharedResource;
private ReentrantLock mLock = new ReentrantLock();
public void changeState() {    
  mLock.lock();    
  try {       
     sharedResource++;   
    } finally {        
      mLock.unlock();   
     }
    }
```

synchronized关键字和ReentrantLock具有相同的语义：如果另一个线程已经进入该区域，这方式都会阻塞所有尝试执行临界区的线程,这是一种防御性的策略，它们假设所有的并发访问都是有问题的，但是多线运行多线程同时读取一个共享变量是没有害处的。因此，synchronized和ReentrantLock可能过度保护了.ReentrantReadWriteLock 运行多线程并发读取，但是不允许边读边写以及同时写入:
```java
int sharedResource; private ReentrantReadWriteLock mLock = new ReentrantReadWriteLock();
public void changeState() {    
  mLock.writeLock().lock();    
  try {                
    sharedResource++;
  } finally {                
    mLock.writeLock().unlock();       
   }
 }

 public int readState() {        
   mLock.readLock().lock();        
   try {               
      return sharedResource;        
    }        
    finally {               
       mLock.readLock().unlock();       
      }
    }


ReentrantReadWriteLock 相对比较复杂，在判断是否线程是该执行还是该阻塞上会比synchronized和ReentrantLock花更多的时间，因此在使用上需要有个取舍。通常较好的策略是当多线程有很多读取操作并且很少的写入操作时，选择ReentrantReadWriteLock会比较好。

```
