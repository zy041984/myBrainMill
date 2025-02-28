# 编译
1. 下载msys2
    https://www.msys2.org/
    安装在系统文件夹，

2. 打开x64 Native Tools Command Prompt for VS2022，运行`msys2_shell.cmd -ucrt64 -use-full-path -where D:\download\build\FFmpeg-n6.1`，会弹出bash窗口
3. 在bash窗口中运行pacman -S nasm和pacman -S yasm，安装了yasm.exe
    把yasm.exe的路径添加到PATH
    在bash窗口中运行pacman -S base-devel，安装一些编译工具
4. 在bash窗口中改变路径到ffmpeg的安装目录，运行`./configure --enable-shared --disable-manpages --arch=x86 --toolchain=msvc`
    如果需要debug版本
    `./configure --enable-shared --disable-manpages --arch=x86 --toolchain=msvc --build-suffix=d --disable-optimizations --disable-doc --enable-debug`

    ```
    ./configure --disable-static --enable-shared --disable-manpages --disable-podpages --disable-txtpages --disable-doc --arch=x86 --toolchain=msvc --build-suffix=d --disable-optimizations --enable-debug
```

5. 运行config的输出是little-endlian win32thread
6. 运行`make`
    好像少了postproc.lib
8. 运行`make install`
    安装到了msys2的安装文件夹的`usr\local\bin`和`usr\local\include`和`usr\local\share`


[leandromoreira/ffmpeg-libav-tutorial: FFmpeg libav tutorial - learn how media works from basic to transmuxing, transcoding and more. Translations: 🇺🇸 🇨🇳 🇰🇷 🇪🇸 🇻🇳 🇧🇷 (github.com)](https://github.com/leandromoreira/ffmpeg-libav-tutorial)

[[总结]FFMPEG视音频编解码零基础学习方法_零基础ffmpeg 雷霄骅-CSDN博客](https://blog.csdn.net/leixiaohua1020/article/details/15811977)

[window10_ffmpeg-msys2-msvc编译_msys2编译ffmpeg_Loken2020的博客-CSDN博客](https://blog.csdn.net/u012117034/article/details/123131135)

[linux 下 编译 x264 遇到的 No working C compiler found 错误-CSDN博客](https://blog.csdn.net/yaotianhao1005/article/details/102486422)

# 使用
## AVCodecContext
编码和解码的过程，与下面四个函数有关
```
avcodec_send_packet()
avcodec_receive_frame()
avcodec_send_frame()
avcodec_receive_packet()
```
编码的过程。
- 初始化并打开AVCodecContext
- `avcodec_send_frame()`把源数据`AVFrame`提供给编码器，
- 在一个循环中调用`avcodec_receive_packet()`获得编码的结果，结果在形参`AVPackets`中，如果给出错误码EAGAIN，表示所有的输入数据都编码了，应该再次输入新数据了
    - 在编码的开始阶段，可能输入几个源数据，才能获得一个编码后的压缩数据
    - 在最后一帧，需要特殊处理。
        - `avcodec_send_frame()`给出空数据
        - 调用`avcodec_receive_packet()`直到给出错误码AVERROR_EOF
        - `avcodec_flush_buffers()`重置AVCodecContext
## AVFormatContext
功能有三个，
- 解流demux，把一个媒体文件分成不同的stream。
- 混流mux，把多个数据写在一个指定格式的容器里
- IO功能，支持多种协议(网络，文件)来读写数据
结构上包括
- 输入格式输出格式
- array容器包装的AVStream，每个流描述了媒体文件的一种数据
- IO context，用来帮助输出
mux例子
- 把编码的`AVPackets`数据写入指定容器格式的文件或其他输出字节流。先调用`avformat_write_header()`写入头，然后调用`av_write_frame()`写入帧数据，最后调用`av_write_trailer()`来结束文件。
- 先调用`avformat_alloc_context()`来创建muxing的context，然后填充一些属性，如oformat。
    - 如果format不是AVFMT_NOFILE，就用`avio_open2()`返回值填充pb
    - 如果format不是AVFMT_NOSTREAMS，就用`avformat_new_stream()`创建一个stream，并填充属性AVStream.codecpar和AVCodecParameters.codec_type，AVCodecParameters.code_id和其他属性。填充AVStream.time_base
- 调用`avformat_write_header()`
- 重复调用`av_write_frame()`把数据送入mux
- 结束时调用`av_write_trailer()`和`avformat_free_context`
## AVFrame
表示原始数据或解码后的音频或视频数据
## AVPacket
表示编码后的数据，音频可以包含多个压缩的frame，视频一般只包含一个压缩的frame
## AVStream
没有特殊解释
## AVIOContext
字节流IO的上下文



# nginx
1. 自行下载编译

下载1.24.0版本的nginx，在http://hg.nginx.org/nginx/rev/05cf7574d94b
下载1.2.2版本的模块rtmp，在https://github.com/arut/nginx-rtmp-module/releases/tag/v1.2.2
可能还要下载msys，pcre，zlib，openssl
两个说明文档
参照这个编译带rtmp模块的nginx https://www.jianshu.com/p/cc008d24ad82
http://nginx.org/en/docs/howto_build_on_win32.html

2. 直接下载带rtmp模块的nginx
在这个地址下载http://nginx-win.ecsds.eu/download/，下载文件nginx 1.7.11.3 Gryphon.zip 
- 解压到目录
- 修改`nginx.conf`
    添加以下内容
    ```
    rtmp {
          server {
                listen 19235;
                application mystream {
                        live on;
                        record off;
                }
        }
    }
    ```
- 运行nginx.exe即可，打开浏览器，输入地址`localhost:80`可以访问说明nginx运行成功


# 命令行实现摄像头推拉流

 `ffplay -f dshow -i video="HIK 1080P Camera"`测试本地相机是否正常
 ```
 ffmpeg -f dshow -i video="HIK 1080P Camera" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://127.0.0.1:19235/mystream/123
```

123为密码，rtmp://127.0.0.1:19235为地址，mystream为nginx配置节点
命令行拉流
```
ffplay rtmp://127.0.0.1:19235/mystream/123
```

命令行进行本地媒体文件的推流 只有-f flv才能被ffplay播放，采用-vcodec h264会清晰一些，不采用-vcodec h264则很模糊
```
ffmpeg -i "D:\\you-get-0.4.1432\\11.mp4" -vcodec h264 -vframes 500 -an -sn -f flv rtmp://127.0.0.1:19235/mystream/123
```
-vcodec h264 与 -f h264 能读取文件并播放，但拉流端不显示
-vcodec h264 与 -f flv 能读取文件并播放，拉流端正常
-f flv 能读取文件并播放，拉流端正常但很不清晰
-vcodec mpeg4 与 -f flv 提示flv不支持mpeg4
-vcodec ljpeg 与 -f flv 提示ljpeg不正确参数
-vcodec jpeg2000 与 -f flv 提示flv不支持jpeg2000
-vcodec h264 与 -f mpeg 能读取文件并播放，但拉流端不显示


```
ffmpeg -i "E:\work\OpenGLTest\simpleGLFW\x64\Debug\\F0-200-YUVA420P.jpeg2000" -vcodec jpeg2000 -vframes 500 -an -sn -f flv rtmp://127.0.0.1:19235/mystream/123
```

-vcodec jpeg2000 与 -f flv 提示flv不支持jpeg2000
-vcodec jpeg2000 与 -f mpeg 提示错误-10054，提交packet给muxer错误10054 buffer underflow

```
ffmpeg -i "E:\work\OpenGLTest\simpleGLFW\x64\Debug\\F0-200-BGRA.ljpeg" -vcodec ljpeg -vframes 500 -an -sn -f mpeg rtmp://127.0.0.1:19235/mystream/123
```
-vcodec ljpeg 与 -f mpeg 提示错误-10054，提交packet给muxer错误10054 buffer underflow