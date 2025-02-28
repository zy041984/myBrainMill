[GitHub - vrpn/vrpn: Virtual Reality Peripheral Network - Official GitHub Repository](https://github.com/vrpn/vrpn)
# 简介
是一系列class，实现了一个网络透明的接口，这个接口可以在应用程序与设备程序之间传递信息。
提供了一个抽象层，所有同类的设备使用同样的封装，这些设备虽然不同，但对外接口看起来是一样。

支持optiTrack的Motive和vicon，把VRPPN server构建成vendor server

# 编译
- 使用CMake，没什么可说的
- 使用VS，打开vrpn_Configuration.h文件，看看有什么配置想修改的。然后用vrpn.sln编译即可。
- quat是四元数库
- core vrpn server library和core vrpn client library是服务器与客户端用到的库文件
- vrpn_server是一个示例的服务器，通过读配置文件来变成某类设备的服务器，向外发消息
- vrpn_print_device是一个示例的客户端，连接一个服务器，并打印接收到的消息
# 快速开始
server用来运行一个device，client用来接收信息。Remote的object就是client，device的object就是server
- 如何快速运行一个服务器端
    - 编辑配置文件vrpn.cfg，运行vrpn_server.exe
- 如何快速测试这个服务器端
    - 运行vrpn_print_devices，给出配置文件vrpn.cfg里的名字和机器，如Tracker0@localhost
- 如何自己写一个客户端
    - 初始化一个设备类的object，例如vrpn_Tracker_Remote,每个类型有自己的头文件
    - 为这个设备写一个handler，注册这个handler
    - 为每个object调用mainloop函数
    - 当按下一个按钮时,客户端会从设备收到这个消息，vrpn会调用handler
- 客户端的类名称为vrpn_设备_Remote，基类名称为vrpn_设备，使用名字和地址来实例化客户端对象
```
vrpn_Tracker_Remote myTracker("Tracker0@127.0.0.1")
```
- mainloop函数的内容，
    - 对于客户端，检查网络缓存，为每个信息调用callback
    - 对于服务器端，把外发的数据放入网络缓存
# 杂项
vrpn_Tracker_Remote发送的位置信息单位是米，
位置分量顺序是x=0 y=1 z=2，数据类型double
姿态信息是单位四元数XYZW，数据类型double
tracker使用了UDP协议，如果客户端没有频繁调用mainloop来及时地把新数据取走，网卡缓冲区就会装满。但不幸的是，UDP丢弃的是新数据，保留了缓冲区的旧数据。所以如果发现数据延迟太大，可以在一次应用程序主循环中，多次调用vrpn的mainLoop把这些旧数据都取走
# 设备类型
## tracker
检测个物体移动到了新位姿
## poser
一个运动平台需要达到的新位姿
## button
一个按钮，有按下和释放两种状态
momentary类型的按钮，按下的时候变成on，松开变成off
toogle类型的按钮，按一下变成on，再按一下变成off
## analog
一个joystick，可以发送多个通道的模拟数值
## analog_output
客户端可以命令服务器端的analog数值
## dial
一个摇杆，可以发送一个角度值
## ForceDevice
一个设备同时发出三种信息，tracker，button，force。所以同时建立三个object，即vrpn_Tracker_remote, vrpn_Button_Remote, vrpn_ForceDevice_Remote来同时接收三种信息。这个设备比较复杂
## Sound
需要vrpn_Sound_Client和vrpn_Sound_Server共同合作，可以设置声学材质，设置场地模型等。这个设备看起来也比较复杂。
## imager
没有过多介绍

# 使用vs编译器的注意事项
- 需要`define _WIN32`
- 需要`define _MCS_VER`
- snprintf在旧版本的vs里没有，请平替
- 注意网络字节顺序hton和ntoh
- 接收USB数据时也要注意字节顺序，`vrpn_unbuffer_from_little_endian`
- 因为vrpn库很小，所以推荐使用lib，不用dll
# troubleShoot
- client一定要频繁调用mainloop，从网卡缓冲区取走server发来的数据，否则会造成server对其他client失去响应
- 所以也不要把mainLoop跟其他循环放在一起，会造成mainLoop持续收到消息，持续在处理消息，无法返回
- 所有的VRPN的callback一定要有类型VRPN_CALLBACK，在windows上会被解释为__stdcall，
- 3883是server的默认端口，不要一台机器运行两个相同端口的server
- vrpn不是线程安全的，为一个object调用mainLoop时通常会导致另一个object也调用callback，所以要保护好资源
# 文本输出
有一个全局的`vrpn_System_TextPrinter`，vrpn程序里每个创建的object都会有这个全局的`vrpn_System_TextPrinter`，它会打印出全部错误和警告信息，可以设置打印信息的level，或者调用`set_ostream_to_use(NULL)`来停止打印消息。
# 写一个server
vrpn_Connection类负责server和client的通信
每种device也有基类，例如vrpn_Tracker.h，在server端和client可以各自继承vrpn_Tracker实现各自的功能
## genericServer通用server
在`server_src/vrpn.C`。从配置文件中读出要打开什么device。
配置文件的每一行列出了类型、tracker名字和机器名字、传感器数量、串口、波特率
## 自己的server
所有的object以`vrpn_`开头，客户端的名字加后缀`_Remote`
比如Vicon公司的tracker类在server端可以命名为`vrpn_Tracker_Vicon`，是否客户端命名就是`vrpn_Tracker_Vicon_Remote`吗
一个object基类应该能够处理交流时使用的信息，例如注册信息类型和发送者名称。应该能编码和解码通信数据。
一个vrpn的server会创建一个`vrpn_Connection`，把它置于监听状态，并创建几个server objects，每个object对应一个设备，每个object都连接到这个监听态的`vrpn_Connection`。然后server就在这几个object之间循环，为每个object调用mainLoop，最后为`vrpn_Connection`调用mainLoop
# 编写设备的driver
每种设备的基类要实现的基本功能，注册信息数据类型，注册发送者名字，编码解码数据。

为设备找到类似的driver，再修改的步骤
1. 找到相似的device，复制已有device的vrpn_Tracker_Vicon.h为vrpn_Tracker_myDevice.h，vrpn_Tracker_Vicon.C为vrpn_Tracker_myDevice.C。
2. 编辑这两个文件，把类重新命名为myDevice
3. 把这两个文件加入到vcxproj中
4. 修改这两个文件的实际内容，实现新设备的功能
5. 在vrpn_Generic_server_object类中增加一个函数，h和c文件都要修改。通常是setup_myDevice，然后增加一个#include "vrpn_Tracker_myDevice.h"
6. 编辑vrpn.cfg文件，增加一行描述。
7. 重新编译vrpn
为设备改写已有的类的步骤
为设备编写新的driver的步骤
1. 所有的object以`vrpn_`开头，客户端的名字加后缀`_Remote`。比如Vicon公司的tracker类在server端可以命名为`vrpn_Tracker_Vicon`，客户端命名就是`vrpn_Tracker_Vicon_Remote`。这server端的类要继承`vrpn_BaseClass`，client端的类继承server端的类。
2. driver需要功能大部分封装在vrpn_BaseClass类中，新的driver应该继承vrpn_BaseClass，并实现自己的编码解码，重写基类的纯虚函数。
3. 从多个类组合的device，必须所有的子类都使用相同的server名字，这个子类才只有一份成员函数和成员变量。例如Phantom设备，继承了Tracker，Button和ForceDevice。客户都打开三个Remote设备才能通信，但是三个Remote具有相同的server名字。
4. 函数`register_autodeleted_handler`用来注册handler，使得设备可以收到信息，并保证该object被析构时handler也被删除。
5. 函数`send_text_message`使得object可以传送文本信息
6. server用来运行一个device，client用来接收信息。Remote的object就是client，device的object就是server。在各自类的mainLoop中，client调用client_mainLoop，server调用server_mainLoop
7. 子类的构造函数应该调用`vrpn_BaseClass::init`，并且需要传入object的名字，以及vrpn_Connection的指针。client通常根据名字需要新建一个connection，server通常需要根据连接请求打开一个connection。构造函数还会初始化d_sender_id和d_text_message_id。init函数会调用纯虚函数register_types，使得子类可以注册消息类型。
# 使用vrpn_Connection
vrpn库的核心是vrpn_Connection类。使用网络来分发消息，并使用callback进行回调。用户的代码通常不会接触到这个类。server和client都会创建各自的vrpn_Connection对象，并在两个connection对象之间建立网络连接。
## 建立连接
server使用监听，client使用connect。
server使用`vrpn_create_server_connection()`来在指定的UDP端口上建立监听，可以监听多个客户端同时发起的连接。如果同多个客户都建立了连接，则之后的循环中server会向客户端群发设备的数据。
client使用`vrpn_get_connection_by_name`来打开指定名字的设备
## 初始化连接
客户端首先打开一个TCP来监听，然后通过UDP向指定端口发送数据包，告诉客户端的tcp端口和地址。server端收到UDP包之后，解析出TCP端口和地址，并发起tcp的connect。客户端的tcp要accept来建立tcp连接。
然后双方在tcp连接中互相发送vrpn的版本号，如果版本号不符合，则断开tcp连接。
如果版本号符合，则双方各自再新建一个UDP端口，使用UDP再建立一个连接。然后双方告诉对方自己的信息类型。
server端可以注册一个handler来响应vrpn_got_connection，每当有client建立连接后，server端的vrpn_Connection对象向server发送这个消息，server端可以在handler中建立正确的响应。
## 关闭连接
每次server端运行mainloop的一次循环时，会调用tcp的select。无论是select、read、write，哪个发生异常，都会使server端断开连接，并再次转入监听状态。
server端可以注册一个handler来响应vrpn_dropped_connection，每当有client断开连接后，server端的vrpn_Connection对象向server发送这个消息，server端可以在handler中建立正确的响应。
## 注册
所有的vrpn信息都有属性，时间，发送者，信息类型。
发送者可以是设备名称，如button0
信息类型表示信息的种类，例如可以是button toggle或button release。
发送者和信息类型本质上都是字符串，但是要映射为一个token，这个映射过程是通过在connection中register来完成的。`long vrpn_Connection::register_sender(char *name)`
每个vrpn_Connection对象都会维护注册到自己的一个sender容器，
类似地，信息类型也可以通过调用`long vrpn_Connection::register_message_type(char *name)`在connection中注册。
## 发送消息
注册发送者和消息类型之后，就可以发送消息了。打包数据，默认8字节对齐。使用vrpn_buffer()和vrpn_unbuffer()来确保字节顺序。
根据标志位vrpn_CONNECTION_RELIABLE来设定使用tcp还是udp来传送消息。
调用mainloop的时候，UDP和TCP的消息都会被发送出去。
## 注册一个handler
server可以调用`register_handler`来注册一个handler，处理感兴趣的sender和类型的消息。
handler的类型声明为`typedef	int (*vrpn_MESSAGEHANDLER)(void *userdata, vrpn_HANDLERPARAM p)`
第一个参数是数据指针，可以把unpack之后的数据写入，或者是处理unpack数据需要的其他数据；第二个参数是一个结构体，描述消息的属性
## forward
vrpn_ConnectionForwarder和vrpn_StreamForwarder可以把消息从一个connection转发到另一个connection。