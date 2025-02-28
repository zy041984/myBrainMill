# 2 Desktop App的UI
## 2.2 国际化
### 2.2.6 Unicode和字符集
新开发的程序要使用Unicode，CEF为UTF16。旧的程序为codepage机制的MBCS，更准确地说是DBCS。
把MBCS和DBCS字符串互相转化，使用MultiByteToWideChar
## 2.3 user interaction
### 2.3.12 legacy user interaction features
#### 2.3.12.2 键盘和鼠标输入
https://learn.microsoft.com/zh-CN/windows/win32/inputdev/user-input
讲了按下键盘如何发出消息，发出什么消息。
虚拟键值和扫描码的关系。
键盘布局
窗口如何获得键盘焦点
如何模拟键盘输入，发出一个假消息
# 12 图像和游戏

# 15 系统服务
## 15.7 DLL
进程之间共享dll的代码，但是有各自的dll的数据。进程加载dll后把它映射到自己的虚拟地址空间。
操作系统根据加载dll的进程数目为每个dll维护一个计数。运行的时候，dll函数运行在调用它的线程里。
所以dll从进程的虚拟地址空间中获得内存，使用线程的stack。该进程中的handle资源可以在各线程以及dll之间共享使用。
有两种方法调用DLL的函数
1. 加载时动态链接
2. 运行时动态链接
加载时需要事先把程序模块与dll先链接起来。而运行时则是使用函数`LoadLibrary`在运行时加载dll，再调用`GetProcAddress`来定位dll中的函数

## 15.9 IPC
FileMapping
把文件的内容当成进程地址空间里的一块内存。进程可以通过指针操作对文件内容进行读写。当多个进程访问同一个文件映射时，每个进程有独自的指针来访问文件映射。
当创建文件映射对象时，指定的是系统交换文件，可以创建一种独特的文件映射，叫做进程间命名共享内存。
## 15.11 内存管理
#### 15.11.1.6 file mapping
把硬盘上的文件的一部分读取到进程虚拟空间中，这就是file mapping对象，它在内存中。接下来可以为file mapping对象的一部分创建一个view，可以把view理解为像fstream的指针，指向文件的某个位置，进程可以用这个view来对file mapping对象进行随机读写或顺序读写。
当系统把file mapping对象所在的page进行swap out时，对file mapping对象的修改可以被写入硬盘的对应文件。

## 15.17 并发
#event #timer #APC
#### APC
##### 啥是APC
一个APC是在某个线程中被异步执行的函数。当一个APC被加入线程的APC队列时，系统触发一个软中断。下次该线程进入时间片时，就会执行APC。该APC不是在调用的时候被执行的，而是下次该线程进入时间片时才会执行APC，所以叫做异步。
##### APC分几类
系统产生的APC称为内核模式的APC
用户产生的APC称为用户模式的APC
##### APC由哪个线程执行
调用了SetWaitableTimer的线程才能执行APC。
一个线程必须处于可唤醒状态才能执行APC，
一个线程调用了SleepEx，SignalObjectAndWait，WaitForSingleObjectEx，等才能进入可唤醒状态。
也就是说要先SetWaitableTimer，然后SleepEx或者Wait，才能等着执行APC。
##### 怎样把APC加入一个线程的APC队列
每个线程都具有APC队列。调用SetWaitableTimer时可以定义APC参数，当Timer到期或到周期而变为signaled时，会把一个APC加入线程APC队列

##### 有个小提醒
当一个线程调用SetWaitableTimer时，该线程最好不要wait该timer。因为该线程会因为timer变为signaled而从wait中返回继续执行。而不是因为APC加入APC队列而从wait中返回。这样线程就离开了可唤醒状态，可能没机会执行APC。
##### event
###### 创建
CreateEventW
###### 属性 
- 是否手动重置
true表示手动重置，需要调用ResetEvent来使得event变为nonsignaled
false表示自动重置，当一个等待的线程被release后，该event自动变为nonsignaled
- 初始状态
true表示初始状态为signaled
false表示初始状态为nonsignaled
###### 作用
当event变为signaled时候，另一个线程的wait系列函数就会返回。使得这个等待中的线程变为released，可以继续向下执行
##### Timer
###### 属性
- 是否手动重置
true为手动重置
false为synchronization timer，当一个等待的线程被release后即从wait返回后，该event自动变为nonsignaled
- 周期 可以周期性地被reactive，手动重置和synchronization timer均可以为周期性
###### 作用
timer创建后为inacitive，调用SetWaitableTimer使其变为active。
当duetime到期时，timer变为inactive，异步调用被加入set该timer的线程，timer变为signaled，如果入set该timer的线程处于可以被唤醒的状态，就进行异步调用completion routine。后续timer根据手动或者wait来变成nonsignaled
注意线程池里的线程最好使用CreateThreadpoolTimer，不使用本类createWaitableTimer。

手动重置的timer
- 当duetime到期时，timer变为inactive，异步调用被加入set该timer的线程，timer变为signaled，并一直处于signaled，这种情况可以调用SetWaitableTimer来重置timer。
- 推测：setWaitableTimer只能把inactive的timer置为active，并让timer变为unsignaled。不能把active的timer从signaled变为unsignaled
周期手动重置的timer
- 当duetime到期时，timer变为inactive，异步调用被加入set该timer的线程，timer变为signaled。然后立刻被reactive还是过了period才被reactive，总之经过reactive后马上unsignaled，然后经过period再次变为signaled。
synchronization timer
- 处于signaled时，可以让一个等待该timer的线程从wait中完成，即返回

##### setWaitableTimer
参数
- lpDueTime 开始时间，负数表示相对时间，单位为100纳秒，表示某段时间之后状态变为signaled
- lPeriod 周期，单位毫秒miliseconds。0表示该对象只signaled一次，大于0表示该对象周期性地变为activate。
- pfn 表示completion routine。自定义的回调函数。当对象变为signaled时由本线程执行，本线程必须处于可唤醒状态。
- lpArg 表示回调函数的参数指针
- fResume，true表示对象signaled之后把系统恢复到省电模式。

##### WaitForSingleObject
- 作用 阻塞调用线程，等对象变为signaled或超时才返回，使得调用线程变为released状态可以继续执行。
- 返回值，指示了使得函数返回的事件，可以是signaled，可以是超时，可以是错误，可以是放弃了
##### WaitForSingleObjectEx
- 作用 阻塞调用线程，等对象变为signaled或超时或者一个APC被加入线程的APC队列，才返回，使得调用线程变为released状态可以继续执行。
- 返回值，指示了使得函数返回的事件，可以是signaled，可以是超时，可以是错误，可以是放弃了，可以是一个APC或IO completion routine被加入线程。
区别在于不仅可以等待signaled或超时，还可以等待APC队列的变化。当调用waitEx的时候APC队列里有对象，那么无须等待直接执行APC队列的对象。
## 15.18 Windows系统信息
### 15.18.1 Handle和Object
https://learn.microsoft.com/en-us/windows/win32/sysinfo/handles-and-objects
#handle #object #kernel-object 
object是一个数据结构，表示了一个系统资源，如一个文件，线程，图像。程序不能直接读取object的内部数据及系统资源。程序只能获得一个object的handle，handle也就是指针，用来访问系统资源。
#### 为什么使用Handle来访问资源
- 可以提高安全性，每个进程可以分配权限控制
- 便于操作系统升级，只要保证接口一致性，既可以升级到高版本操作系统
#### Object的管理和接口函数
object在系统的存储结构包括一个header和一个属性。且所有object的存储结构相同，可以用一个管理器来维护。
header包含了名称，权限描述等数据，名称可以让其它进程找到这个object。权限描述可以指定进程对object的访问权限。
接口函数分为创建object，创建handle，关闭handle，销毁object等。
一个进程结束时，系统自动关闭handle，并删除本进程创建的object
一个线程结束时，系统不关闭handle，不删除本线程创建的object。除非是windows，hook，DDE，window position型的object
object的所有handle都关闭后才会从内存中销毁
#### Object的分类
用户Object，进行窗口管理，如cursor，icon，menu，window，hook，window position，
GDI Object，进行图形绘制，如brush，bitmap，dc，font，pen，palette，region
内核Object，进行内存管理，进程管理，IPC。如event，file，filemapping，heap，mutex，pipe，semaphore，timer，mutex，process，thread，socket
#### Handle的限制
- 用户Object只能有一个handle，不能被子进程继承或其他进程复制。但是用户Object可以被所有进程访问，只要某进程对该object有访问权限
- GDI型Object只能有一个handle，不能被子进程继承或其他进程复制。创建该object的进程才可以使用该object
- 内核Object可以创建多个handle，可以被子进程继承或其他进程复制，只要某进程对该object有访问权限。多个handle的意义在于每个handle可以有不同的访问权限。
- 内核file object稍有不同，一个硬盘或内存中的file可以有多个file object，每个file object有一个handle，每个进程可以对一个file有多个这样的file object即handle。