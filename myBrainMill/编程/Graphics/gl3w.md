https://github.com/skaslev/gl3w/

使用Python version 2.7 or newer运行gl3w_gen.py，这个脚本自动从OpenGL的官网https://www.khronos.org/registry/OpenGL/api/GL下载适用的头文件和c文件。

打开IDLE，打开gl32_gen.py，按F5运行，显示下载了glcorearb.h和khrplatform.h，生成了gl3w.h和gl3w.cpp


然后在自己的项目中include这些头文件和c文件，glfw说要先包括gl3w.h，再包含glfw.h

在一台pc使用的gl3w.h和gl3w.cpp不能简单地复制到另一台pc，需要在新pc上重新下载头文件