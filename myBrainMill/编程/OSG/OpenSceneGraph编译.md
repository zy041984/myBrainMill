#qt #GL2 #GL3 #版本340
20241026试通的osg340和osgEarth270
 原则
 根据osgEarth2.7版本的时间2015.7，选择了时间上比较匹配的osg版本3.4.0和第三方库的版本，osg和osgEarth使用vs2022进行编译，操作系统是win10，windowsAPI的版本是10.0.22621。
 有些第三方库使用vs2015进行了编译，操作系统是win10，windowsAPI的版本是10.0.22621。
 protobuf暂时使用的新版，没使用2015左右的版本
 编译过程中重要的问题
 1. osg不能使用UTF8FileName，osgEarth不支持。如果osg非要用UTF8FileName，就要改osgEarth的源码。
 2. osg可以使用GL2或GL3。如果使用GL3，要参照osg3.6.5改GLDefines.h部分源代码，以解决GL3.h中没有定义GL_VERTEX_PROGRAM_TWO_SIDE的问题
 3. Qt暂时使用了5.12.12，osgEarth运行之后退出时有bug，而且最好使用osg单线程
 4. cmake暂时使用了3.6.3，但是估计使用高版本的应该也没问题
 5. 单独测试osgGL2和GL3，读png/gif/tiff/rgb/jpeg纹理没问题了，显示中英文文字没问题。GL3显示文字的时候，按照samples，使用额外的fs。
 6. osgEarthQt要解决moc文件没有include源类的头文件问题，生成sln之后需要在`\src\osgEarthQt\moc_AnnotationDialogs.cpp_parameters`中加入一行 `-f绝对路径/AnnotationDialogs`
 7. osgEarthSymbology使用geos_c_d.lib死活link不过，最后发现链接geosd.lib才成功
 8. 现在都可以使用cmake命令行或nmake命令行来生成了，将来是不是可以形成一个自动化的脚本
 9. osgEarth的sample在虚拟机不能运行，好像是openGL版本认得不对
 10. osgViewer不能读出图片文件，还没有调试原因
 11. 生成的sln和vcxproj工程如果拿到别的电脑编译，需要把源码，工程，依赖放到编译设置的文件夹位置，还要把cmake放在编译时设置的位置。能不能让cmakeList建立工程的时候使用相对文件路径呢
# 第三方库
## 1.zlib
### 地址
http://www.zlib.net/
### 版本
1.2.8 2013.4.28
### 使用cmake-GUI
无特殊选项
### vs编译顺序
1 zero_check
2 zib 生成了dll和lib
3 zlibstatic
4 example
5 minigzip
6 install
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\zlib-1.2.8" -B "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\zlib" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" --config Release
```
测试
```
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" -C debug
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\zlib-1.2.8-vs2015" -C release
```
### 被其他库调用时
其他库使用动态库zlib.dll和zlib.lib
## 2.png
### 地址
http://www.libpng.org/pub/png/libpng.html
### 版本
1.6.18 2015.7.23
### 依赖
zlib
### 使用cmake-GUI
无特殊选项，需要设置zlib的目录，可以设置PNG_STATIC为OFF
### vs编译顺序
png16 生成了libpng16.lib和dll
pngstest
pngtest
pngvalid
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\libpng-1.6.18" -B "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\libpng" -D PNG_STATIC=FALSE -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib\include" -D ZLIB_LIBRARY_DEBUG="C:\installFiles\SrcToBld\zlib\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" --config Release
```
### 测试
```
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" -C debug
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\libpng-1.6.18-vs2015" -C release
```
### 被其他库调用时
其他库使用动态库libpng16.dll和libpng16.lib
## 3.freetype
### 地址
https://freetype.org/
### 版本
2.5.5  2014.12.30
### cmake最后生成了一个static的lib
### vs编译顺序
freetype
install
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\freetype-2.5.5" -B "C:\installFiles\SrcToBld\freetype-2.5.5-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\freetype" -D CMAKE_DEBUG_POSTFIX=d --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\freetype-2.5.5-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\freetype-2.5.5-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\freetype-2.5.5-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\freetype-2.5.5-vs2015" --config Release
```
### 被其他库调用时
其他库使用静态库freetype.lib
## 4.libjpeg-turbo
### 地址
可以搜索到两个相关的库，一个是旧的libjpeg，一个是号称性能更好的libjpeg-turbo，这里使用libjpeg-turbo。
https://www.libjpeg-turbo.org/ 号称性能比libjpeg好 https://sourceforge.net/projects/libjpeg-turbo/files/1.4.2/ 2015
旧的libjpeg库在http://www.ijg.org/ 文件jpegsr9b.zip是在http://www.ijg.org/files/，日期是01 17 2016
### 版本
1.4.2
### cmake
Enable中取消ENABLE_STATIC,取消WITH_SIMD，增加一个CMAKE_DEBUG_POSTFIX
### vs编译顺序
jpeg 生成了jpeg62.dll和lib
turbojpeg 生成了turbojpeg.dll和lib，好像别人不好用
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2" -B "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\libjpeg-turbo64" -D CMAKE_DEBUG_POSTFIX=d -D ENABLE_STATIC=OFF -D WITH_SIMD=OFF --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" --config Release
```
测试
```
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" -C debug
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\libjpeg-turbo-1.4.2-vs2015" -C release
```
### 被其他库调用时
其他库使用动态库jpeg62d.dll和jpegd.lib
## 5.libtiff
### 地址
http://www.libtiff.org/
### 版本
4.0.5
### 使用cmakeGUI
选择zLIB，jpeg
要手动把TiffTestCommon.cmake复制一份，改名为TiffTest.cmake，还要把zlib和jpeg的dll复制过来，test才能成功
### vs编译顺序
tiff  生成tiffd.lib和tiffd.dll
tiffxx
tiff_port
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\tiff-4.0.5" -B "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\tiff" -D CMAKE_DEBUG_POSTFIX=d -D JPEG_INCLUDE_DIR="C:\installFiles\SrcToBld/libjpeg-turbo64/include" -D JPEG_LIBRARY="C:\installFiles\SrcToBld/libjpeg-turbo64/lib/jpeg.lib" -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib\include" -D ZLIB_LIBRARY="C:\installFiles\SrcToBld\zlib\lib\zlib.lib"  --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" --config Release
```
测试
```
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" -C debug
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\tiff-4.0.5-vs2015" -C release
```
### 被其他库调用时
其他库使用动态库tiff.dll
## 6.gif
### 地址
https://sourceforge.net/projects/giflib/files/
### 版本
5.1.1 20150106
### vs编译
自己建立一个static lib工程，添加makefile中指定的十个文件(
```
SOURCES= dgif_lib.c egif_lib.c gifalloc.c gif_err.c gif_font.c gif_hash.c openbsd-reallocarray.c HEADERS= gif_hash.h  gif_lib.h  gif_lib_private.h
```
)，取消头文件unistd.h，取消warning4996。编译即可。
### 坑
据说除了内存泄漏，最大的问题c99是bool变量占4个byte，c++的bool1个byte
### 被其他库调用时
其他库使用静态库libgif.lib
## 7.curl
### 地址
https://curl.se/download/
### 版本
7.43.0  20150617
### cmake
使用了自行编译的zlib
### vs编译顺序
curltool
curlu
libcurl_object
libcurl_shared   生成libcurl-d_imp.lib和libcurl-d.dll
curl
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\curl-7.43.0" -B "C:\installFiles\SrcToBld\curl-7.43.0-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\curl" -D CMAKE_DEBUG_POSTFIX=d -D BUILD_CURL_TESTS=OFF -D ENABLE_MANUAL=OFF -D ENABLE_UNIX_SOCKETS=OFF -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib\include" -D ZLIB_LIBRARY="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\curl-7.43.0-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\curl-7.43.0-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\curl-7.43.0-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\curl-7.43.0-vs2015" --config Release
```
### 被其他库调用时
其他库使用动态库libcurl.dll和libcurl_imp.lib
## 8. tcl
### 地址
https://www.tcl.tk/software/tcltk/download.html
### 版本
8.6.13
### 编译
进入win文件夹
`nmake /f makefile.vc release --enable-64bit INSTALLDIR="C:\installFiles\tcl"`
`nmake /f makefile.vc release --enable-64bit install INSTALLDIR="C:\installFiles\tcl"`
生成了tclsh86t.exe要新建一个副本为tclsh.exe.
生成了tcl86t.lib要新建一个副本为tcl86.lib
下面好像不需要
`nmake /f makefile.vc INSTALLDIR="C:\installFiles\tcl" TCLDIR="C:\installFiles\tcl8.6.13\win"`
`nmake /f makefile.vc install INSTALLDIR="C:\installFiles\tcl" TCLDIR="C:\installFiles\tcl8.6.13\win"`
生成了wish86t.exe要放在Tcl安装文件夹
## 9. sqlite3
https://www.sqlite.org/download.html
`nmake /f Makefile.msc`
生成了dll和lib和exe
安装还要手动
## 10.proj
### 地址
https://proj.org/en/9.3/install.html
### 版本
4.9.2 2015-09-03
### cmakeGUI
无依赖和其他设置
### vs编译顺序
proj 生成lib
cct
cs2cs
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\proj-4.9.2" -B "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\PROJ" -D CMAKE_DEBUG_POSTFIX=d --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" --config Release
```
测试
```
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" -C debug
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\proj-4.9.2-vs2015" -C release
```
### 被其他库调用时
其他库使用静态库proj_d.lib
## 11.geos
### 地址
https://libgeos.org/
### 版本
3.5.0  2015-8-16
3.12.0版本不兼容osgEarth2.7.0
### 3.12.0版本cmake
需要手动注释cmakelist.txt的189行
`#set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo" "ASAN")`
### 3.5.0用vs2022或vs2015需要对下面代码修改
1.geos_ts_c.cpp,注释了103行`//#include "../geos_svn_revision.h"`
修改了行3499`//sprintf(version, "%s r%d", GEOS_CAPI_VERSION, GEOS_SVN_REVISION);`
因为GEOS_SVN_REVISION这个整数需要在cmake时候通过svn得到
2.BufferOp.cpp，修改了行90，91，92，把std::max改为max
### 3.5.0vs编译顺序
geos
geos_static
geos_c
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\geos-3.5.0" -B "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\GEOS" -D CMAKE_DEBUG_POSTFIX=d -D GEOS_ENABLE_TESTS=OFF -D BUILD_TESTING=OFF --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" --config Release
```
测试
```
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" -C debug
ctest.exe  --force-new-ctest-process --test-dir  "C:\installFiles\SrcToBld\geos-3.5.0-vs2015" -C release
```
### 被其他库调用时
优先使用c语言版本的动态库geos_c.dll和geos_c.lib
## 12. libgeotif
### 地址
https://github.com/OSGeo/libgeotiff
### 版本
1.4.1 2015
### 编译
依赖libtif和proj
修改proj的debug库引用，修改几个exe项目debug配置的输出，修改bin\cmake_install.cmake的debug配置的exe文件名
config
```
cmake -S "C:\installFiles\SrcToBld\libgeotiff-1.4.1\libgeotiff" -B "C:\installFiles\SrcToBld\libgeotiff-1.4.1-vs2015" -G "Visual Studio 14 2015" -A x64 --install-prefix "C:\installFiles\SrcToBld\libgeotiff" -D CMAKE_DEBUG_POSTFIX=d -D GEOTIFF_CSV_DATA_DIR="C:/installFiles/SrcToBld/libgeotiff-1.4.1/libgeotiffGUI/csv" -D PROJ4_INCLUDE_DIR="C:/installFiles/SrcToBld/PROJ/local/include" -D PROJ4_LIBRARY="C:/installFiles/SrcToBld/PROJ/local/lib/proj_4_9.lib" -D TIFF_INCLUDE_DIR="C:/installFiles/SrcToBld/tiff/include" -D TIFF_LIBRARY_DEBUG="C:/installFiles/SrcToBld/tiff/lib/tiffd.lib" -D TIFF_LIBRARY_RELEASE="C:/installFiles/SrcToBld/tiff/lib/tiff.lib" -D WITH_PROJ4=ON -D WITH_TIFF=ON -D WITH_TOWGS84=ON -D WITH_UTILITIES=ON -D WITH_ZLIB=OFF -D WITH_JPEG=OFF --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\libgeotiff-1.4.1-vs2015" --config Debug
cmake --build "C:\installFiles\SrcToBld\libgeotiff-1.4.1-vs2015" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\libgeotiff-1.4.1-vs2015" --config Debug
cmake --install "C:\installFiles\SrcToBld\libgeotiff-1.4.1-vs2015" --config Release
```
## 13.gdal
### 地址
https://gdal.org/en/latest/download_past.html#download-past
https://download.osgeo.org/gdal/1.11.2/
### 版本
1.11.2 2015.2
无法用cmake进行编译，要使用nmake
未使用tif和geotif
1.把winsdk10的rc.exe所在目录加入环境变量path
2.修改nmake.opt，添加geos，proj，curl，png
取消了ODBC
3.然后debug版本
`nmake -f makefile.vc MSVC_VER=1900 DEBUG=1 WIN64=yes`
`nmake -f makefile.vc MSVC_VER=1900 DEBUG=1 WIN64=yes install`
4.换一个文件夹release版本
`nmake -f makefile.vc MSVC_VER=1900 WIN64=yes`
`nmake -f makefile.vc MSVC_VER=1900 WIN64=yes install`
### 被其他库调用时
使用动态库gdal.dll和gdal.lib
## 14.abseil-cpp
### 地址
### 版本
20240116.2
### cmake
选中ABSL_PROPAGATE_CXX_STD=ON，取消test
### vs编译顺序
太多了
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\abseil-cpp-20240116.2" -B "C:\installFiles\SrcToBld\abseil-cpp-20240116.2" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\Program Files\absl" -D CMAKE_DEBUG_POSTFIX=d -D BUILD_TESTING=OFF -D ABSL_PROPAGATE_CXX_STD=ON --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\abseil-cpp-20240116.2" --config Debug
cmake --build "C:\installFiles\SrcToBld\abseil-cpp-20240116.2" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\abseil-cpp-20240116.2" --config Debug
cmake --install "C:\installFiles\SrcToBld\abseil-cpp-20240116.2" --config Release
```
### 被protobuf库调用时
静态库
## 15.protobuf
### 地址
### 版本
28.2
### cmake
取消test，abseil选择package，然后输入dir。选择zlib，取消protobuf_MSVC_STATIC_RUNTIME
### vs编译顺序
utf8
protobuf
protoc
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\protobuf-28.2" -B "C:\installFiles\SrcToBld\protobuf-28.2" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\Program Files\protobuf" -D CMAKE_DEBUG_POSTFIX=d -D protobuf_ABSL_PROVIDER=package -D absl_DIR="C:/installFiles/SrcToBld/absl-x64-vc143/lib/cmake/absl" -D protobuf_BUILD_TESTS=OFF -D protobuf_BUILD_LIBUPB=OFF  -D protobuf_BUILD_LIBPROTOC=ON -D protobuf_MSVC_STATIC_RUNTIME=OFF -D protobuf_WITH_ZLIB=ON -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib-1.3-x64-vc143\include" -D ZLIB_LIBRARY_DEBUG="C:\installFiles\SrcToBld\zlib-1.3-x64-vc143\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE="C:\installFiles\SrcToBld\zlib-1.3-x64-vc143\lib\zlib.lib" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\protobuf-28.2" --config Debug
cmake --build "C:\installFiles\SrcToBld\protobuf-28.2" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\protobuf-28.2" --config Debug
cmake --install "C:\installFiles\SrcToBld\protobuf-28.2" --config Release
```
### 被其他库调用时
使用静态库libprotobufd.lib
# 16.OpenSceneGraph
## 使用GL2-Qt-noUTF8的版本
3.4.0
### 使用cmake图形界面
GL2，QT5.12.12，vs2022版本
使用了上面生成的第三方库zlib，tiff，png，jpeg，gif，gdal，freetype，curl
在vs中检查几个生成的项目，
gdal和ogr和gif的debug修改了lib为debug名字
暂时不选择OSG_USE_UTF8_FILENAME=TRUE，osgEarth2.7不支持
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0" -B "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL2" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\installFiles\SrcToBld\OpenSceneGraph" -D USE_3RDPARTY_BIN=FALSE -D CURL_IS_STATIC=FALSE -D OSG_USE_UTF8_FILENAME=OFF -D CURL_INCLUDE_DIR="C:/installFiles/SrcToBld/curl/include" -D CURL_LIBRARY_DEBUG="C:/installFiles/SrcToBld/curl/lib/libcurld_imp.lib" -D CURL_LIBRARY_RELEASE="C:/installFiles/SrcToBld/curl/lib/libcurl_imp.lib" -D FREETYPE_INCLUDE_DIR_freetype2="C:\installFiles\SrcToBld\freetype\include\freetype2" -D FREETYPE_INCLUDE_DIR_ft2build="C:\installFiles\SrcToBld\freetype\include\freetype2" -D FREETYPE_LIBRARY="C:/installFiles/SrcToBld/freetype/lib/freetype.lib" -D FREETYPE_LIBRARY_DEBUG="C:/installFiles/SrcToBld/freetype/lib/freetyped.lib" -D FREETYPE_LIBRARY_RELEASE="C:/installFiles/SrcToBld/freetype/lib/freetype.lib" -D GDAL_INCLUDE_DIR="C:/installFiles/SrcToBld/gdal/include" -D GDAL_LIBRARY="C:/installFiles/SrcToBld/gdal/lib/gdal_id.lib" -D GIFLIB_INCLUDE_DIR="C:/installFiles/SrcToBld/libgif/include" -D GIFLIB_LIBRARY="C:/installFiles/SrcToBld/libgif/lib/giflib.lib" -D JPEG_INCLUDE_DIR="C:\installFiles\SrcToBld\libjpeg-turbo64/include" -D JPEG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/libjpeg-turbo64/lib/jpegd.lib" -D JPEG_LIBRARY_RELEASE="C:/installFiles/SrcToBld/libjpeg-turbo64/lib/jpeg.lib" -D PNG_PNG_INCLUDE_DIR="C:/installFiles/SrcToBld/libpng/include" -D PNG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/libpng/lib/libpng16d.lib" -D PNG_LIBRARY_RELEASE="C:/installFiles/SrcToBld/libpng/lib/libpng16.lib" -D TIFF_INCLUDE_DIR="C:/installFiles/SrcToBld/tiff/include" -D TIFF_LIBRARY_DEBUG="C:/installFiles/SrcToBld/tiff/lib/tiffd.lib" -D TIFF_LIBRARY_RELEASE="C:/installFiles/SrcToBld/tiff/lib/tiff.lib" -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib\include" -D ZLIB_LIBRARY_DEBUG="C:\installFiles\SrcToBld\zlib\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" -D ZLIB_LIBRARY="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" -D QT_INCLUDE_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\include" -D QT_MOC_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\moc.exe" -D QT_QMAKE_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\qmake.exe" -D QT_UIC_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\uic.exe" -D QT_QTMAIN_LIBRARY="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\qtmain.lib" -D Qt5Widgets_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\cmake\Qt5Widgets" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL2" --config Debug
cmake --build "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL2" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL2" --config Debug
cmake --install "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL2" --config Release
```
## 使用GL3-Qt-noUTF8的版本
使用了上面生成的第三方库zlib，tiff，png，jpeg，gif，gdal，freetype，curl
暂时不选择OSG_USE_UTF8_FILENAME=TRUE，osgEarth2.7不支持
在vs中检查几个生成的项目，
freetype的debug和release里没有加png的lib，
curl，jpeg，png，tiff的release的lib名字写错了
gdal和gif的debug修改了lib为debug名字
GL_VERTEX_PROGRAM_TWO_SIDE这个常量在gl3.h和glcorearb.h里已经没有了，被移到了glext.h中，但glcorearb.h头文件中说包含了glcorearb.h就不要再包含gl.h,也不要再包含glext.h。gl3.h也说不要再包含glext.h。
怎么办
参考osg3.6.5的GLDefines的代码吧，手动修改GLDefines.h
不知道怎么在cmake命令行里增加`/I "C:\installFiles\SrcToBld"`所以把`GL/gl3.h`放在了源代码的include文件夹
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0" -B "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL3" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\installFiles\SrcToBld\OpenSceneGraph" -D USE_3RDPARTY_BIN=OFF -D CURL_IS_STATIC=OFF -D OSG_USE_UTF8_FILENAME=OFF -D OSG_GL3_AVAILABLE=ON -D OSG_GL1_AVAILABLE=OFF -D OSG_GL2_AVAILABLE=OFF -D OSG_GLES1_AVAILABLE=OFF -D OSG_GLES2_AVAILABLE=OFF -D OSG_GL_DISPLAYLISTS_AVAILABLE=OFF -D OSG_GL_FIXED_FUNCTION_AVAILABLE=OFF -D OSG_GL_MATRICES_AVAILABLE=OFF -D OSG_GL_VERTEX_ARRAY_FUNCS_AVAILABLE=OFF -D OSG_GL_VERTEX_FUNCS_AVAILABLE=OFF -D OSG_USE_QT=ON -D OPENGL_PROFILE=GL3 -D CURL_INCLUDE_DIR="C:/installFiles/SrcToBld/curl/include" -D CURL_LIBRARY_DEBUG="C:/installFiles/SrcToBld/curl/lib/libcurld_imp.lib" -D CURL_LIBRARY_RELEASE="C:/installFiles/SrcToBld/curl/lib/libcurl_imp.lib" -D FREETYPE_INCLUDE_DIR_freetype2="C:\installFiles\SrcToBld\freetype\include\freetype2" -D FREETYPE_INCLUDE_DIR_ft2build="C:\installFiles\SrcToBld\freetype\include\freetype2" -D FREETYPE_LIBRARY="C:/installFiles/SrcToBld/freetype/lib/freetype.lib" -D FREETYPE_LIBRARY_DEBUG="C:/installFiles/SrcToBld/freetype/lib/freetyped.lib" -D FREETYPE_LIBRARY_RELEASE="C:/installFiles/SrcToBld/freetype/lib/freetype.lib" -D GDAL_INCLUDE_DIR="C:/installFiles/SrcToBld/gdal/include" -D GDAL_LIBRARY="C:/installFiles/SrcToBld/gdal/lib/gdal_id.lib" -D GIFLIB_INCLUDE_DIR="C:/installFiles/SrcToBld/libgif/include" -D GIFLIB_LIBRARY="C:/installFiles/SrcToBld/libgif/lib/giflib.lib" -D JPEG_INCLUDE_DIR="C:\installFiles\SrcToBld\libjpeg-turbo64/include" -D JPEG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/libjpeg-turbo64/lib/jpegd.lib" -D JPEG_LIBRARY_RELEASE="C:/installFiles/SrcToBld/libjpeg-turbo64/lib/jpeg.lib" -D PNG_PNG_INCLUDE_DIR="C:/installFiles/SrcToBld/libpng/include" -D PNG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/libpng/lib/libpng16d.lib" -D PNG_LIBRARY_RELEASE="C:/installFiles/SrcToBld/libpng/lib/libpng16.lib" -D TIFF_INCLUDE_DIR="C:/installFiles/SrcToBld/tiff/include" -D TIFF_LIBRARY_DEBUG="C:/installFiles/SrcToBld/tiff/lib/tiffd.lib" -D TIFF_LIBRARY_RELEASE="C:/installFiles/SrcToBld/tiff/lib/tiff.lib" -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib\include" -D ZLIB_LIBRARY_DEBUG="C:\installFiles\SrcToBld\zlib\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" -D ZLIB_LIBRARY="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" -D QT_INCLUDE_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\include" -D QT_MOC_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\moc.exe" -D QT_QMAKE_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\qmake.exe" -D QT_UIC_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\uic.exe" -D QT_QTMAIN_LIBRARY="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\qtmain.lib" -D Qt5Widgets_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\cmake\Qt5Widgets" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL3" --config Debug
cmake --build "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL3" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL3" --config Debug
cmake --install "C:\installFiles\SrcToBld\OpenSceneGraph-3.4.0-GL3" --config Release
```
# 17.osgEarth
## 地址
## 版本
2.7.0   2015.07
### cmake
curl geos gdal qt protobuf sqlite3 zlib
1 在vs中检查几个生成的项目，
release版本关于zlib的库用错了，在vsCode中搜索替换
把
```
optimized.lib;C:\installFiles\SrcToBld\zlib\lib\zlib.lib;debug.lib;C:\installFiles\SrcToBld\zlib\lib\zlibd.lib
```
替换为
```
C:\installFiles\SrcToBld\zlib\lib\zlib.lib
```
2 增加了gdal的debug
用vscode打开文件夹，
把
```
libcurld_imp.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
libcurld_imp.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
把
```
geos_cd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
geos_cd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
把
```
OpenThreadsd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
OpenThreadsd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
把
```
osgEarthSymbologyd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
osgEarthSymbologyd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
把
```
osgGAd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
osgGAd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
把
```
osgEarthUtild.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
osgEarthUtild.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
把
```
osgEarthd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_i.lib
```
替换为
```
osgEarthd.lib;C:\installFiles\SrcToBld\gdal\lib\gdal_id.lib
```
3 osgEarthSymbology链接时报错GEOS，后来把这个项目用的GEOS库从c改为c++，居然能link了
4 osgEarthQT编译不过，`moc_(%Filename).cpp`中没有包含头文件，所以需要在`\src\osgEarthQt\moc_AnnotationDialogs.cpp_parameters`中加入一行 `-f绝对路径/AnnotationDialogs`
查看`src\osgEarthQt\CMakeLists.txt`里面可以看到哪些文件需要moc
uic好像没这个问题
Tool osgearth_package_qt项目也是同样的问题
注意如果改了编译路径或源代码路径，这些cpp_parameters文件中的路径也要改
5 好像要把zlib.dll拷贝到osgVersion.exe的路径中
### vs编译顺序
太多了
### 使用cmake命令行
config
```
cmake -S "C:\installFiles\SrcToBld\osgearth-2.7" -B "C:\installFiles\SrcToBld\osgearth-2.7-vs2022" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\installFiles\SrcToBld\OSGEARTH" -D CMAKE_DEBUG_POSTFIX=d -D CPACK_BINARY_NSIS=OFF -D CPACK_SOURCE_7Z=OFF -D CPACK_SOURCE_ZIP=OFF -D OSGEARTH_USE_QT=ON -D THIRD_PARTY_DIR="C:\installFiles\SrcToBld\3rdParty" -D CURL_INCLUDE_DIR="C:/installFiles/SrcToBld/curl/include" -D CURL_IS_STATIC=OFF -D CURL_LIBRARY="C:/installFiles/SrcToBld/curl/lib/libcurl_imp.lib" -D CURL_LIBRARY_DEBUG="C:/installFiles/SrcToBld/curl/lib/libcurld_imp.lib" -D GDAL_INCLUDE_DIR="C:/installFiles/SrcToBld/gdal/include" -D GDAL_LIBRARY="C:/installFiles/SrcToBld/gdal/lib/gdal_i.lib" -D GEOS_INCLUDE_DIR="C:/installFiles/SrcToBld/GEOS/include" -D GEOS_LIBRARY="C:/installFiles/SrcToBld/GEOS/lib/geos_c.lib" -D GEOS_LIBRARY_DEBUG="C:/installFiles/SrcToBld/GEOS/lib/geos_cd.lib" -D OPENSCENEGRAPH_MAJOR_VERSION=3 -D OPENSCENEGRAPH_MINOR_VERSION=4 -D OPENSCENEGRAPH_PATCH_VERSION=0 -D OPENSCENEGRAPH_SOVERSION=130 -D OPENTHREADS_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/OpenThreads.lib" -D OPENTHREADS_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/OpenThreadsd.lib" -D OSGDB_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgDB.lib" -D OSGDB_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgDBd.lib" -D OSGFX_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgFX.lib" -D OSGFX_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgFXd.lib" -D OSGGA_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgGA.lib" -D OSGGA_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgGAd.lib" -D OSGMANIPULATOR_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgManipulator.lib" -D OSGMANIPULATOR_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgManipulatord.lib" -D OSGPARTICLE_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgParticle.lib" -D OSGPARTICLE_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgParticled.lib" -D OSGQT_LIBRARY="C:\installFiles\SrcToBld\OpenSceneGraph\lib\osgQt.lib" -D OSGQT_LIBRARY_DEBUG="C:\installFiles\SrcToBld\OpenSceneGraph\lib\osgQtd.lib" -D OSGSHADOW_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgShadow.lib" -D OSGSHADOW_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgShadowd.lib" -D OSGSIM_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgSim.lib" -D OSGSIM_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgSimd.lib" -D OSGTERRAIN_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTerrain.lib" -D OSGTERRAIN_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTerraind.lib" -D OSGTEXT_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgText.lib" -D OSGTEXT_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTextd.lib" -D OSGUTIL_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgUtil.lib" -D OSGUTIL_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgUtild.lib" -D OSGVIEWER_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgViewer.lib" -D OSGVIEWER_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgViewerd.lib" -D OSGWIDGET_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgWidget.lib" -D OSGWIDGET_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgWidgetd.lib" -D OSG_DIR="C:/installFiles/SrcToBld/OpenSceneGraph" -D OSG_GEN_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSG_INCLUDE_DIR="C:\installFiles\SrcToBld\OpenSceneGraph\include" -D OSG_LIBRARY="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osg.lib" -D OSG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgd.lib" -D OSG_PLUGINS="osgPlugins-3.4.0" -D OSG_VERSION_EXE="C:/installFiles/SrcToBld/OpenSceneGraph/bin/osgversion.exe" -D Protobuf_INCLUDE_DIR="C:\installFiles\SrcToBld\protobuf\include" -D Protobuf_LIBRARY_DEBUG="C:\installFiles\SrcToBld\protobuf\lib\libprotobufd.lib" -D Protobuf_LIBRARY_RELEASE="C:\installFiles\SrcToBld\protobuf\lib\libprotobuf.lib" -D Protobuf_LITE_LIBRARY_DEBUG="C:\installFiles\SrcToBld\protobuf\lib\libprotobuf-lited.lib" -D Protobuf_LITE_LIBRARY_RELEASE="C:\installFiles\SrcToBld\protobuf\lib\libprotobuf-lite.lib" -D Protobuf_PROTOC_EXECUTABLE="C:\installFiles\SrcToBld\protobuf\bin\protoc.exe" -D Protobuf_PROTOC_LIBRARY_DEBUG="C:\installFiles\SrcToBld\protobuf\lib\libprotocd.lib" -D Protobuf_PROTOC_LIBRARY_RELEASE="C:\installFiles\SrcToBld\protobuf\lib\libprotoc.lib" -D Protobuf_SRC_ROOT_FOLDER="C:\installFiles\SrcToBld\protobuf-28.2\src\google\protobuf" -D SQLITE3_INCLUDE_DIR="C:\installFiles\SrcToBld\sqlite3\include" -D SQLITE3_LIBRARY="C:\installFiles\SrcToBld\sqlite3\lib/sqlite3.lib" -D ZLIB_INCLUDE_DIR="C:\installFiles\SrcToBld\zlib\include" -D ZLIB_LIBRARY_DEBUG="C:\installFiles\SrcToBld\zlib\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE="C:\installFiles\SrcToBld\zlib\lib\zlib.lib" -D QT_QMAKE_EXECUTABLE="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\bin\qmake.exe" -D Qt5Core_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\cmake\Qt5Core" -D Qt5Gui_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\cmake\Qt5Gui" -D Qt5OpenGL_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\cmake\Qt5OpenGL" -D Qt5Widgets_DIR="C:\Qt\Qt5.12.12\5.12.12\msvc2017_64\lib\cmake\Qt5Widgets" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\osgearth" --config Debug
cmake --build "C:\installFiles\SrcToBld\osgearth" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\osgearth" --config Debug
cmake --install "C:\installFiles\SrcToBld\osgearth" --config Release
```
# 18.virtualPlanetBuilder
## 地址
https://github.com/openscenegraph/VirtualPlanetBuilder
## 版本
0.9.13   2015.07
## cmake3.26.5GUI
gdal
##使用cmake命令行
修改release配置下lib的optimize和debug
修改debug配置下使用的gdal库
config
```
cmake -S "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-0.9.13" -B "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-0.9.13-vs2022" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\installFiles\SrcToBld\VirtualPlanetBuilder" -D CMAKE_DEBUG_POSTFIX=d -D DYNAMIC_VIRTUALPLANETBUILDER=on -D OSG_USE_AGGRESSIVE_WARNINGS=on -D 3rdPartyRoot="C:\installFiles\SrcToBld\3rdParty\bin" -D GDAL_INCLUDE_DIR="C:/installFiles/SrcToBld/gdal/include" -D GDAL_LIBRARY="C:/installFiles/SrcToBld/gdal/lib/gdal_i.lib" -D OPENTHREADS_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OPENTHREADS_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/OpenThreads.lib" -D OPENTHREADS_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/OpenThreadsd.lib" -D OSG_DIR="C:/installFiles/SrcToBld/OpenSceneGraph" -D OSG_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSG_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osg.lib" -D OSG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgd.lib" -D OSGDB_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGDB_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgDB.lib" -D OSGDB_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgDBd.lib" -D OSGFX_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGFX_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgFX.lib" -D OSGFX_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgFXd.lib" -D OSGGA_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGGA_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgGA.lib" -D OSGGA_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgGAd.lib" -D OSGSIM_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGSIM_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgSim.lib" -D OSGSIM_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgSimd.lib" -D OSGTERRAIN_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGTERRAIN_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTerrain.lib" -D OSGTERRAIN_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTerraind.lib" -D OSGTEXT_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGTEXT_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgText.lib" -D OSGTEXT_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTextd.lib" -D OSGUTIL_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGUTIL_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgUtil.lib" -D OSGUTIL_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgUtild.lib" -D OSGVIEWER_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGVIEWER_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgViewer.lib" -D OSGVIEWER_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgViewerd.lib" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-0.9.13-vs2022" --config Debug
cmake --build "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-0.9.13-vs2022" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-0.9.13-vs2022" --config Debug
cmake --install "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-0.9.13-vs2022" --config Release
```
如果是1.00-rc1版本
## 使用cmake命令行
config有错误，很奇怪，文件与0.9.8版本没差什么
config
```
cmake -S "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-1.0.0-rc1" -B "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-1.0.0-rc1-vs2022" -G "Visual Studio 17 2022" -A x64 --install-prefix "C:\installFiles\SrcToBld\VirtualPlanetBuilder" -D CMAKE_DEBUG_POSTFIX=d -D DYNAMIC_VIRTUALPLANETBUILDER=on -D OSG_USE_AGGRESSIVE_WARNINGS=on -D 3rdPartyRoot="C:\installFiles\SrcToBld\3rdParty\bin" -D GDAL_INCLUDE_DIR="C:/installFiles/SrcToBld/gdal/include" -D GDAL_LIBRARY="C:/installFiles/SrcToBld/gdal/lib/gdal_i.lib" -D OPENTHREADS_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OPENTHREADS_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/OpenThreads.lib" -D OPENTHREADS_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/OpenThreadsd.lib" -D OSG_DIR="C:/installFiles/SrcToBld/OpenSceneGraph" -D OSG_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSG_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osg.lib" -D OSG_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgd.lib" -D OSGDB_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGDB_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgDB.lib" -D OSGDB_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgDBd.lib" -D OSGFX_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGFX_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgFX.lib" -D OSGFX_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgFXd.lib" -D OSGGA_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGGA_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgGA.lib" -D OSGGA_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgGAd.lib" -D OSGSIM_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGSIM_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgSim.lib" -D OSGSIM_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgSimd.lib" -D OSGTERRAIN_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGTERRAIN_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTerrain.lib" -D OSGTERRAIN_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTerraind.lib" -D OSGTEXT_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGTEXT_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgText.lib" -D OSGTEXT_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgTextd.lib" -D OSGUTIL_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGUTIL_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgUtil.lib" -D OSGUTIL_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgUtild.lib" -D OSGVIEWER_INCLUDE_DIR="C:/installFiles/SrcToBld/OpenSceneGraph/include" -D OSGVIEWER_LIBRARY_RELEASE="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgViewer.lib" -D OSGVIEWER_LIBRARY_DEBUG="C:/installFiles/SrcToBld/OpenSceneGraph/lib/osgViewerd.lib" --fresh
```
build
```
cmake --build "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-1.0.0-rc1-vs2022" --config Debug
cmake --build "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-1.0.0-rc1-vs2022" --config Release
```
install
```
cmake --install "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-1.0.0-rc1-vs2022" --config Debug
cmake --install "C:\installFiles\SrcToBld\VirtualPlanetBuilder-VirtualPlanetBuilder-1.0.0-rc1-vs2022" --config Release
```