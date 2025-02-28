#duration #time_point #system_clock #steady_clock
几个概念

[[duration]]如字面意思，指的是一段时间，如一秒，一小时。第一个参数是多少个tick可以是浮点型，第二个参数是两个tick之间的时间间隔以秒记
`std::chrono::duration<double, std::ratio<1, 30>> fr(2.5)`表示该duration两个tick之间的间隔是三十分之一秒，tick类型是double，对象fr有2.5个tick，

[[time_point]]指的是时间长河中的一点，第一个参数是用的什么clock标准，比如系统时钟或者单调时钟，第二个参数是当前点距离起点的[[duration]]

clock的类型
[[steady_clock]]指的是单调递增的，这个clock的时间点只能增加不能减少，而且tick之间的时间间隔是恒定的。这个clock与墙上钟表可以无关，可以用来测量interval，例如自从上次重启的时间间隔。常用的静态成员函数就是now
[[system_clock]]指得是墙上时钟，系统时间。有可能不是递增的。结果可以被转化为c风格的时间。大多数系统时间的起点是未指定的，大多数系统使用UTC的1970年1月1日。

典型应用
1.使用两个time_point对象的now函数记录两个时间点a和b，两个时间点的差b-a就是duration，然后`std::chrono::duration_cast<std::chrono::microseconds>(b-a).count()`就可以知道按照某种tick间隔(比如微秒)而得到的计数(两个时间点的微秒数)
2.把时间转化为字符串
```
#include <chrono>
#include <ctime>
#include <iomanip>

#define WIN32_LEAN_AND_MEAN //从 Windows 头文件中排除Cryptography, DDE, RPC, Shell, and Windows Sockets
#define NOCOMM
#include <windows.h>

std::chrono::time_point now{ std::chrono::system_clock::now() };//需要c++17
std::time_t tt = std::chrono::system_clock::to_time_t(now);
tm tmGMT;
errno_t err = gmtime_s(&tmGMT, &tt);
tm tmLocal;
errno_t err2 = localtime_s(&tmLocal, &tt);//Locale time-zone, usually UTC by default.
char mbstr[100];
std::strftime(mbstr, sizeof(mbstr), "%F-%H_%M_%S", &tmLocal);//系统时间转格式文本
```