[ZeroMQ](https://zeromq.org/)
[GitHub - zeromq/libzmq: ZeroMQ core engine in C++, implements ZMTP/3.1](https://github.com/zeromq/libzmq)
[bit.ly](http://bit.ly/ZeroMQ-OReilly)
https://www.doc88.com/p-3187187515829.html
3.2
[Preface | ØMQ - The Guide (zeromq.org)](https://zguide.zeromq.org/docs/preface/)
zeroMQ的socket只代表一个客户端，不代表传统中与另一个节点的connection。传统中一个thread代表已经与另一个节点建立了connection。
zeroMQ的客户代码中只有socket，connection已经被隐藏。使用zeroMQ时无需关心connection。

[45/ZWS | ZeroMQ RFC](https://rfc.zeromq.org/spec/45/)
启用ZMQ_BUILD_DRAFT_API宏，可以使用函数zmq_connect_peer，zmq_join，zmq_leave

## chapter1
zeroMQ只传送内容，不关心内容是什么，所以如果传输了字符串，要特别注意传输的字符串有没有后缀0，长度包不包括后缀0
退出时一定要注意，如果有消息正在发送，如果有socket没有close，则调用zmq_ctx_term时会阻塞
如果涉及到多线程，不要在多个线程中共享socket。关闭socket时候，先设置一个较低的LINGER值(1秒)，再close这个socket。
关闭context的时候，socket被阻塞的操作会返回错误，捕获这个错误，设置一个较低的LINGER，再close这个socket。不要为一个context多次调用term
## chapter2
socket的生存有四个部分
- 创建和析构，zmq_socket和zmq_close
- 配置，zmq_setsockopt
- 把socket连接到网络，zmq_bind和zmq_connect
- 收发信息，zmq_msg_send和zmq_msg_recv
socket是个void指针，消息是个结构体。socket这个指针由库来管理，消息这个结构体由用户负责内存管理。
但是要记住库是异步的，这点影响很大。
1.服务器用zmq_bind，地址endpoint已知，客户端用zmq_connect，客户端地址任意。
zmq的connection与其他库不一样，
- 可以有ipc，tcp，pgm等协议
- 可以有多个传入传出的connection
- 没有accept方法，服务器自动接纳客户端
- zmq会自动处理网络掉线又重连的
假设开启服务器之前，就打开了客户端，普通的机制会出错，但是zmq没问题。只要客户端调用了connect，就会有连接，等到服务器bind，zmq就会在两个端点之间发送信息。
zmq的socket就像udp，带有消息，不像tcp是面对流的。
zmq的socket在后台线程处理io，调用zmq_send发送消息的时候实际是把消息压入待发送内部容器，不会阻塞；调用zmq_recv接受消息的时候实际是把消息存入已接受内部容器。
zmq的TCP和IPC不依赖于服务器和客户端已经连接起来，就可以直接发送消息。
zmq的线程间通信，inter-thread-transport是一个连接的信号传递
zmq不是中性carrier，但是自适应各种协议。你可以用zmq写http
### IO线程
使用zmq_ctx_set的时候可以设置线程数量，一个线程一个context，一个context一个socket，一个socket可以处理好多个connection
### 消息模式
zmq可以把大量数据快速高效地发到节点，节点可以是线程，进程，其他endpoint。你无需关系底层通信协议是tcp还是udp还是ipc。它会自动处理节点上线下线，自动把消息放入容器，自动限制容器大小，而且无锁。这一切都依赖于**模式**。
模式有以下这些，request-reply，pub-sub，pipeline，exclusive pair
以下模式就是connect-bind对
pub-sub，req-rep，req-router，dealer-rep，dealer-router，dealer-dealer，router-router，push-pull，pair-pair。
还有些更高级的模式reliable-req-rep，即Majordomo
### 处理消息
用zmq_msg_init来初始化消息
用zmq_msg_data来读取消息内容
用zmq_msg_get来获取消息属性
用zmq_msg_copy来操作消息
其实消息就是一块内存，你可以protoBUf，json等序列化。
用zmq_msg_init来创造一个空消息，传给zmq_msg_recv来接收消息
用zmq_msg_init_size来从已有大小初始化一个消息，memcpy来填充数据，再用zmq_msg_send发送消息，这个消息然后会被zmq清除。
用zmq_msg_close释放消息
如果想把一个消息多次发送，用zmq_msg_init初始化第二个消息，用zmq_msg_copy来创造第一个消息的引用，再第二次发送。
可以把一个大消息分块发送，每块是一个frame，frame有一个bit表示是单独帧还是分块帧，frame的结构体就是zmq_msg_t，low-level的api可以用这些结构体。
分块的消息要么全部发送，要么就不发送。而且由于异步发送，必须确保发送的时候这些分块都在内存中有效。
可以发送一个零长度的消息，就像信号
发完信息不用zmq_msg_close，接收消息再处理完成后，调用zmq_msg_close释放消息。
这些API是为了优化而设计的，不是为了好用而设计的，所有用之前确认读过man
### 多个socket
如果同时从多个endpoint接收消息，该怎么设计
最简单的办法是用fan-in模式一对多，pull-push模式下就不能这样用了
用zmq_poll形成reactor模式，
### 分块消息
可以用多个frame组成一个大块消息，每个frame是一个zmq_msg结构，把这些结构放在一个容器里，依次发送出去。zmq_msg_send的最后参数是ZMQ_SNDMORE，表示这个消息是分块。但是由于异步的关系，最后一块被send后，才会真正地发送
接收的时候，如果调用了zmq_poll，当接收了第一个分块的时候，其实所有的分块都到了。无论是否调用了查询。
### 代理和中间件
代理，中间层，中间件，中间人，转发器，队列都是一类概念，在中间处理两边的东西。
### 动态发现
XPUB和XSUB扮演中间代理的角色，所有的PUB接到XPUB节点，所有的SUB接到XSUB节点，而XPUB和XSUB在处理消息传递。新加入的节点只要连接到代理节点即可，无需知道有没有对等的SUB或PUB通信端
# 抓包发现
TCP双方握手之后，客户端向服务器发送一个消息的flag是PSH，ACK，服务器会回复一个ACK表示收到了这个消息。服务器向客户端发送消息同理。
# zmq的doc4.3.5
## zmq_errno
为调用线程返回错误变量errno的值，
## zmq_strerror(int)
为调用线程返回错误变量errno对应的文本`const char*`
## zmq_socket
在形参的context中创建一个socket，返回其不透明的handle。创建之后是unbind状态。要么直接connect一个endpoint，要么bind到一个endpoint进行监听。
与传统的TCP或UDP的socket不同，这是一个异步信息队列的socket。
连接方式可以一对多，多对一，一对一，即connect或bind的时候可以输入多个endpoint
有些socket是线程安全的，
有些非线程安全
关于type
client-server模式，一个server可以连接多个client。client先建立连接，然后双方可以相互通信。server不能主动先向client通信。
radio-dish模式，一个publisher可以向多个subscriber发送消息。使用了group，多个dish可以加入一个group。草案阶段。
publish-subscrib模式，一个publisher可以向多个subscriber发送消息。subsciber可以选择订阅某些消息。
pipeline模式，数据依次从流水线的一个阶段发到下一个阶段，每个阶段可以有若干个节点，这些节点共同收到该阶段的数据。
peer-to-peer模式，一个peer可以connect到别的peer，也可以监听别的peer的connect。草案阶段。
request-reply模式，客户端发起request，然后服务器reply。一个客户端可以连接到多个服务器。通信顺序只能是一问一答，如果客户端多问，后面的问可能被阻塞。
## zmq_bind
形参socket使用本地的endpoint来接收外来的连接，本地的endpoint可以使用tcp/ipc/inproc/udp/vmci/pgm/epgm连接类型，有些类型的地址可以使用通配符。调用bind之后，socket进入了mute状态，建立连接后进入ready状态。在mute状态下，要么阻塞，要么抛弃消息。
成功返回0
## zmq_recv
在形参socket上接收消息，存入形参char*，
形参0阻塞式的，形参ZMQ_DONTWAIT表示非阻塞式的，如果非阻塞且没有消息，设置错误EAGAIN
返回值表示接收到的字节数，返回-1表示错误，

## zmq_close
关闭形参socket，一个socket只能调用一次。
返回值0表示成功

## zmq_ctx_term
结束一个zmq_context，成功则返回0
具体步骤是
1.除了zmq-close，其余所有阻塞的socket操作会立刻返回ETERM，在socket上后续的任何操作会返回ETERM
2.阻塞，直到本zm_context所有的socket都调用了zmq_close
socket上的消息已经被发送或超时

## zmq_ctx_shutdown
关闭一个zmq_context，成功则返回0
具体步骤是
除了zmq-close，其余所有阻塞的socket操作会立刻返回ETERM，在socket上后续的任何操作会返回ETERM
建议客户调用zmq_ctx_term，而zmq_ctx_shutdown掉不掉用都行
## zmq_msg_init
初始化一个新的空消息结构体，在zmq_msg_recv之前调用
不要直接访问zmq_msg_t的成员，用函数来访问
zmq_msg_init和zmq_msg_init_data和zmq_msg_init_size和zmq_msg_init_buffer是互斥的
成功返回0
## zmq_msg_init_size
按照形参大小初始化一个新的消息结构体，不会重置内存，应用程序自行选择形参zmq_msg_t在堆还是栈上
## zmq_msg_init_buffer
按照形参大小初始化一个新的消息结构体，并把内容复制进去，应用程序自行选择形参zmq_msg_t在堆还是栈上
## zmq_msg_init_data
把形参zmq_msg_t进行初始化，使得它的大小为形参size_t，数据为`void*`，不会发生内容复制，zmq库来接手控制`void*`这块内存。必须提供一个线程安全的内存析构函数，当zmq库不需要这块内存时，能够释放掉它。
## zmq_msg_set
设置message的属性，例如Socket-Type，Routing-Id，PEER_ADDRESS，应用程序也可以自定义一些属性。属性是null结尾的utf8字符
## zmq_msg_close
通知context形参zmq_msg_t关联的资源已经不需要了。什么时候资源被释放用户不会知道。
调用zmq_msg_send之后无需调用zmq_msg_close
## zmq_msg_copy
把一个zmq_msg_t的内容浅复制到另一个zmq_msg_t，dest的内存必须提前被初始化，dest之前的内容会被释放。
如果需要深复制，请使用zmq_msg_init_buffer
## zmq_msg_move
把一个zmq_msg_t的内容移动到另一个zmq_msg_t，dest的内存必须提前被初始化，dest之前的内容会被释放。src会变成空
## zmq_msg_send
把形参zmq_msg_t送到形参socket的发送缓冲队列
如果指定了标志位ZMQ_DONTWAIT，则表明如果socket是dealer的push端，且正在阻塞，则zmq_msg_send应该被执行为非阻塞模式，如果消息不能被放入发送缓冲队列，则返回错误EAGAIN
如果指定了标志位ZMQ_SNDMORE，则表明是分块message的一块
函数成功返回后，形参zmq_msg_t被清空，不能再被使用，如果想把一个消息发给多个socket，必须先调用zmq_msg_copy来复制多份消息。
函数返回失败后，形参zmq_msg_t未改变，该消息必须被后续的zmq_msg_send使用，或者调用zmq_msg_close来析构，否则就会产生内存泄漏
成功的返回值是消息的大小，失败的返回值是-1
## zmq_msg_recv
从形参socket接收一个消息，保存在形参zmq_msg_t，如果形参之前有内容，会被正确析构，默认是阻塞。
如果设置标志位ZMQ_DONTWAIT，表示如果没有消息就返回，并且设置错误值EAGAIN。
成功的返回值是消息的大小，失败的返回值是-1
## zmq_send
不用把内容先封装到zmq_msg_t，而是直接用形参指针和形参大小来指定要发送的数据，把数据压入形参socket的待发送缓冲队列。
## zmq_recv
从形参socket接收一个消息，保存在形参指针和形参大小指定的内存，如果内存不够，会丢弃未保存的数据，默认是阻塞。
如果设置标志位ZMQ_DONTWAIT，表示如果没有消息就返回，并且设置错误值EAGAIN。
成功的返回值是消息的大小，失败的返回值是-1
## zmq_poll 已废弃
提供了一个在多个socket间使用多路IO复用的机制。(内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程。)
形参zmq_pollitem_t是一个容器，每个元素是一个结构体，保存了socket，订阅的事件句柄events，实际接收的事件句柄revents，文件描述符fd
zmq_poll轮询容器，每个元素的socket和events有效，如果接收的事件符合哪个元素的订阅，就把该元素的revents置为有效。
如果接收的事件不符合任意元素的订阅，则等到超时时间，如果时间参数为0，则zmq_poll函数立刻返回，如果时间参数为-1，则zmq_poll函数阻塞
事件是一些位的OR组合，例如ZMQ_POLLIN，ZMQ_POLLOUT，ZMQ_POLLERR，ZMQ_POLLPRI

每个元素只能由调用zmq_poll的线程使用
如果一个socket在多个元素中，这些元素都在不同线程，则这个socket必须保证线程安全。
## zmq_poller_new,zmq_poller_destroy...
使用`zmq_poller_*`系列函数来实现多路IO复用
## zmq_ctx_new
生成一个线程安全的context，返回其不透明的handle。这个函数代替了废弃的zmq_init
# 学习过程
## 安装wireshark
安装了4.4.3，为了能从127.0.0.1抓包，还需要安装tap-windows和Npcap，这是附在wireshark安装包里的