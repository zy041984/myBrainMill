asio的网址是https://think-async.com/Asio/index.html
- 有独立的版本，只有头文件
- 有集成在boost中的版本
用两种都可以，两种可以在一个项目中共存

现在使用的时独立版本1.24.0
源代码 https://github.com/chriskohlhoff/asio/tree/asio-1-24-0
文档 https://think-async.com/Asio/asio-1.24.0/doc/

ASIO是集成了异步操作的网络和线程库，跨平台的

使用了基于proactor模式的异步

# 简要描述
所有程序必须初始化一个`asio::io_context`，其余的asio对象，如果提供了IO功能，必须要借助构造函数，把`asio::io_context`传递进来。
必须先有一个需要异步等待的工作，然后调用`io_context::run()`，asio会进入流程。
哪个线程调用了`io_context::run()`，asio保证该线程会接收到completion handler，反应了异步调用的结果。

asio会调用一个异步等待的操作，进行异步操作。当一个异步操作完成后，会把结果送到completion handler。如果再没有异步的操作，`io_context::run()`就会返回。

如果只有一个线程调用了`io_context::run()`，那么所有的completion handler都会并发运行的。
如果线程池中的多个线程调用了`io_context::run()`，则当completion handlers访问某些共享资源时，就需要有一种同步方法，比如strand_。它是一个执行者适配器，保证从本对象分发的handler会依次执行，不会并发运行，也就是保护了共享资源不会被多个handler同时访问。

## 同步的TCP的流程
### client
用`ip::tcp::resolver`来把hostname和servicename转变为一系列endpoints，
定义一个`tcp::socket`，并调用`asio::connect`进行连接。以上endpoints可能含有v4和v6，asio自动去寻找有效的连接
接下来就可以调用`tcp::socket::read_some`从连接获取传输数据。当连接断掉时，会返回错误`asio::error::eof`
### server
用`ip::tcp::acceptor`同步调用`accept()`来监听握手的连接，为每一个握手建立一个`tcp::socket`，并调用`ip::tcp::acceptor::accept`来接收连接，这就完成了与client的连接。
接下来就可以调用`asio::write`向连接中写入数据
## 异步的TCP的流程
### server
新建`tcp::acceptor`调用`async_accpet`来监听连接，然后调用`io_context::run()`进行异步等待，并指定`handle_accept()`作为completion handler来处理客户端的握手请求。
在`handle_accept()`中完成与client的连接，有一个接入的连接，就新建一个connection实例。并调用`async_write()`异步写入数据，然后指定completion handler，让线程等待本次异步写结束。
## 同步UDP的流程
### client
用`ip::udp::resolver`来把hostname和servicename转变为一系列endpoints，并指定v4
定义一个`ip::udp::socket`，并调用`socket::open`进行连接。
接下来就可以调用`upp::socket::send_to`向连接发送数据。用`ip::udp::socket::receive_from()`接收数据
### server
创建一个`ip::udp::socket`来准备接收某端口的连接，调用`ip::udp::socket::receive_from()`接收客户端的传输数据，调用`upp::socket::send_to`向客户端发送数据。
## 异步UDP的流程
### server
直接调用异步的接收`ip::udp::socket::async_receive_from()`，开始异步监听一个连接请求。当接收到传入的数据后，调用指定的completion handler，处理接收到的数据，并再次开启一个异步听或发送的调用。

## chat程序
### server
初始化后调用`start_accept()`，进行`async_accept()`，该函数返回后，意味着有一个新聊天者发来了握手信息，于是执行`handle_accept()`，即调用`session::start()`，把新聊天者加入聊天室，把聊天室的已有信息发给新聊天者。让聊天者异步async_read接收聊天室的信息。然后server再次调用start_accept准备监听下一个新聊天者的握手。

## 自己写程序的心得
1. 未接收到一个handler时不要再进行下一次异步调用
2. socket的初始化顺序是open，bind，
3. socket的析构顺序是cancle，等待io_context_.stop()，close
4. wireshark抓包，以及网络调试助手两个工具。