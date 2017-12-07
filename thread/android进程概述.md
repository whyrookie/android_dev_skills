## 读书笔记(1)-android进程概述
每次介绍android都会遇见的一张图:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/android-stack_new.png)

从上至下:

System apps:

应用层，会使用到下一层的内容，即Core Java和Java Api Framework


Java Api framework:

控制Window系统，UI,Resources等，定义并且管理Android各个组件的生命周期以及组件之间的通讯。包含了HandlerThread，AsyncTask，IntentService，AsyncQueryHandler，Loaders 等一系列异步类，用来简化和管理Android线程。

Native Libraries:

c/c++库，包含了 graphics, media, database, fonts, OpenGL, 等，通常app不会直接直接与这些库交互，Java Api framework已经对其进行了封装.

Runtime:
Dalvik 或者 ART(Api19引入),沙盒机制，每个app运行在单独的Runtime，拥有独占的虚拟机.

Linux kernel:
Linux内核管理sound, network, camera等驱动，使上层代码可以控制硬件，同时管理Linux 进程，每个进程控制一个Runtime。内核会分配给每个进程以及进程中包含的线程的运行时间。


Core Librarie:
核心java库，基于jdk1.5,供Applications和Application framework使用,是Apache Harmony(已退役)实现的一个子集，并不完全兼容Java SE和Java ME.
它提供了基本的Java线程机制:包含了 java.lang.Thread 类和java.util.concurrent 包.

Linux prosess

Linux 属于多用户系统，每个用户分配一个Id（数字）,系统根据这个Id追踪用户，每个用户拥有获取自身私有资源的权限，除了root(linux中的超级用户)，其他用户无法获取该权限。在Android中，每个app包拥有一个唯一的Id,对应了Linux中唯一的用户Id。Android 实际上是在Linux proecess中添加了一个Runtime，例如虚拟机(Davik/Art)，具体关系:

![](https://github.com/whyrookie/android_dev_skills/blob/master/images/android_process.jpg)

默认情况下，Application 和process是一一对应的的关系，特殊情况下，一个Application可以运行在多个prosess上，多个Application也可以运行在一个process上。

Application 的生命周期:

应用的生命周期被封装在了process中，Java 中用Application这个类作为代表，Application对象在Runtime调用它的onCreate()方法时开始初始化，在Runtime调用它的onTerminate()方法时停止
,Linux 内核有可能在Runtime调用Application的onTerminate()之前被杀死,Application会也立即被销毁。Application在整个app的生命周期中是第一个被创建，最后一个被销毁的。

Application的启动顺序:
任何组件都可以作为Application启动的入口，一旦第一个组件被触发创建，Linux 进程就启动了，Application也随之初始化，简化的顺序:

1.启动 Linux process
2.创建 Runtime
3.创建 Application 对象
4.创建 Application的入口组件

1.2两个步骤是长时间的操作，所以如果每次都要执行这两步的话，app的启动时间肯定会很长，体验太差，因此，Android 为了节约每次app的创建时间，在系统启动的时候就创建了一个特殊的进程--Zygote(受精卵).Zygote拥有一些通用的预加载库，新的app创建的时候，只要forkZygote进程，而不需要重新加载那些预加载库，因为可以共享这些核心库，节省了大量的启动时间和内存资源。

Application停止:

当系统内存资源紧缺的时候，系统会选择停止某些app。因为用户可能在任何时候启动app,所以为了快速启动app，即使app中的所以组件都被销毁了，app也没有完全被系统杀死，除非系统缺少资源，才会考虑杀死某个app，但如何在多个app中选择需要被杀死的app?Android引入了一个进程等级机制，等级高低取决于app组件的显示和运行状态。级别高到低:

Foreground:app有一个可见的组件，或者一个serviece被远程的一个可见的Activity绑定，或者BroadcastReceiver正在运行。

Visible:app可见，但有部分模糊

Service:Service运行在后台，并且没有被可见的组件绑定

Background:不可见的Activity，大部分app都处于这种状态

Empty：空进程，没有任何存活的组件，当系统回收资源时，最先被回收。

线程的引入:
Android设备都是多处理器的,可以同时运行多个任务。如果一个app没有并发运行就只能利用一个cpu,效率太低。其中的一个解决方法就是把任务放在多个进程中执行，但是多个进程必然占用大量的资源，并且进程间的通讯速度太慢，也没有高效的异步运行机制，所以使用多线程是个很好的解决方法。因为Android只允许在UI线程更新ui，而Android的绘制是要求是16ms/帧,这样人眼才不会感觉到卡顿，所以长时间的任务不能放在UI线程阻塞UI的渲染，而应该放在后台线程去执行,这就需要用到多线程。长时间的任务通常包括以下几个:

网络请求

读/写文件

创建，删除，更新数据库中的数据

读/写 SharedPreferences

图像处理

文本解析


Android提供了很多类帮助我们简化多线程的管理，并提供了Handler机制让非UI线程与UI线程交互。

注:内容来自《Effecient Android Threading》
