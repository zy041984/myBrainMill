# 下载
https://sourceforge.net/projects/log4cpp/files/log4cpp-1.1.x%20%28new%29/log4cpp-1.1/
# 编译
- 选择多线程是使用pthreads还是omnithreads？
- 用winSDK10.0.22621.0，工具集v143
- 字符集用了Unicode Character Set的时候编译不通过，用MB时候可以编译
有些项目通过，有些不通过
- CMake版本3.8不识别vs2022，不能编译
编译失败