使用CMake的命令行进行configure和generate
-S指定源文件夹 -B指定输出的solution位置 -G指定生成器 -A指定架构
```
cmake -S path/to/glfw -B path/to/build -G "Visual Studio 17 2022" -A x64
```
`--build <dir> [<options>] `为编译
`--install <dir> [<options>]`为安装，可以指定绝对路径`--install-prefix <dirAbs>`
`-D`指定一些选项的属性


`-D CMAKE_DEBUG_POSTFIX=d`可以让生成的debug模式的名字加入后缀d
`-D GLFW_BUILD_TESTS=FALSE`可以为glfw在cmake-gui中显示的某些选项赋值
例如glfw，生成Debug和Release版本要分别调用一次cmake，安装也要为每个configuration分别调用cmake
```
.\cmake -S "D:\download\glfw-3.3.8" -B "D:\download\glfw-3.3.8-build" -G "Visual Studio 17 2022" -A x64 -D BUILD_SHARED_LIBS=TRUE -D CMAKE_DEBUG_POSTFIX=d -D GLFW_BUILD_TESTS=FALSE -D GLFW_BUILD_EXAMPLES=FALSE --install-prefix "C:\Program Files\glfw"
```
```
.\cmake --build "D:\download\glfw-3.3.8-build" --config Debug
.\cmake --build "D:\download\glfw-3.3.8-build" --config Release
```
```
.\cmake --install "D:\download\glfw-3.3.8-build" --config Debug
.\cmake --install "D:\download\glfw-3.3.8-build" --config Release
```
例如zlib，生成Debug和Release版本要分别调用一次cmake，安装也要为每个configuration分别调用cmake
```
.\cmake -S "D:\download\zlib-1.3" -B "D:\download\zlib-1.3-build" -G "Visual Studio 17 2022" -A x64 -D BUILD_SHARED_LIBS=TRUE -D CMAKE_DEBUG_POSTFIX=d -D GLFW_BUILD_TESTS=FALSE -D GLFW_BUILD_EXAMPLES=FALSE --install-prefix "C:\Program Files\zlib"
```
```
.\cmake --build "D:\download\zlib-1.3-build" --config Debug
.\cmake --build "D:\download\zlib-1.3-build" --config Release
```
```
.\cmake --install "D:\download\zlib-1.3-build" --config Debug
.\cmake --install "D:\download\zlib-1.3-build" --config Release
```
## 一个库依赖另一个库的情况
例如libpng，依赖于zlib，
- 需要-D来指定include和debug和release的绝对路径
```
.\cmake -S "D:/download/libpng-1.6.40" -B "D:/download/libpng-1.6.40-build" -G "Visual Studio 17 2022" -A x64 -D PNG_DEBUG=TRUE -D CMAKE_DEBUG_POSTFIX=d -D PNG_DEBUG_POSTFIX=d -D PNG_STATIC=FALSE -D PNG_TESTS=FALSE -D ZLIB_INCLUDE_DIR="D:\download\zlib-1.3-VS2022-x64\zlib\include" -D ZLIB_LIBRARY_DEBUG="D:\download\zlib-1.3-VS2022-x64\zlib\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE="D:\download\zlib-1.3-VS2022-x64\zlib\lib\zlib.lib" --install-prefix "C:\Program Files\libpng"
```

```
.\cmake --build "D:/download/libpng-1.6.40-build" --config Debug
.\cmake --build "D:/download/libpng-1.6.40-build" --config Release
```

```
.\cmake --install "D:/download/libpng-1.6.40-build" --config Debug
.\cmake --install "D:/download/libpng-1.6.40-build" --config Release
```

```
D:\cmake-3.26.5-windows-x86_64\bin\cmake.exe -S "./libpng-1.6.40" -B "./libpng-1.6.40-build" -G "Visual Studio 17 2022" -A x64 -D PNG_DEBUG=TRUE -D CMAKE_DEBUG_POSTFIX=d -D PNG_DEBUG_POSTFIX=d -D PNG_STATIC=FALSE -D PNG_TESTS=FALSE -D ZLIB_INCLUDE_DIR=".\zlib-1.3-VS2022-x64\zlib\include" -D ZLIB_LIBRARY_DEBUG=".\zlib-1.3-VS2022-x64\zlib\lib\zlibd.lib" -D ZLIB_LIBRARY_RELEASE=".\zlib-1.3-VS2022-x64\zlib\lib\zlib.lib" --install-prefix "C:\Program Files\libpng"
```

```
.\cmake --build "./libpng-1.6.40-build" --config Debug
.\cmake --build "./libpng-1.6.40-build" --config Release
```

```
.\cmake --install "./libpng-1.6.40-build" --prefix ".\libpng" --config Debug
.\cmake --install "./libpng-1.6.40-build" --prefix ".\libpng" --config Release
```

- 或者指定依赖库的安装路径，自动调用cmakeList.txt的find_package函数去找头文件和库。
`share\cmake-3.26\Modules\FindZLIB.cmake`里面说可以定义`ZLIB_ROOT`，在这个路径下面寻找，但是执行时发现需要调用`cmake_policy(set CMP0074 NEW)`，或者在cmake命令中添加`-D CMAKE_POLICY_DEFAULT_CMP0074=NEW`才能使得`ZLIB_ROOT`生效

```
.\cmake -S "D:/download/libpng-1.6.40" -B "D:/download/libpng-1.6.40-build" -G "Visual Studio 17 2022" -A x64 -D CMAKE_POLICY_DEFAULT_CMP0074=NEW -D ZLIB_ROOT="D:\download\zlib-1.3-VS2022-x64\zlib" -D PNG_DEBUG=TRUE -D CMAKE_DEBUG_POSTFIX=d -D PNG_DEBUG_POSTFIX=d -D PNG_STATIC=FALSE -D PNG_TESTS=FALSE --install-prefix "C:\Program Files\libpng-1.6.40"
```

```
D:\cmake-3.26.5-windows-x86_64\bin\cmake.exe -S "./libpng-1.6.40" -B "./libpng-1.6.40-build" -G "Visual Studio 17 2022" -A x64 -D PNG_DEBUG=TRUE -D CMAKE_POLICY_DEFAULT_CMP0074=NEW -D ZLIB_ROOT=".\zlib-1.3-VS2022-x64\zlib" -D CMAKE_DEBUG_POSTFIX=d -D PNG_DEBUG_POSTFIX=d -D PNG_STATIC=FALSE -D PNG_TESTS=FALSE
```

```
D:\cmake-3.26.5-windows-x86_64\bin\cmake --build "./libpng-1.6.40-build" --config Debug
D:\cmake-3.26.5-windows-x86_64\bin\cmake --build "./libpng-1.6.40-build" --config Release
```
安装路径还是不要加版本号
```
D:\cmake-3.26.5-windows-x86_64\bin\cmake --install "./libpng-1.6.40-build" --prefix ".\libpng" --config Debug
D:\cmake-3.26.5-windows-x86_64\bin\cmake --install "./libpng-1.6.40-build" --prefix ".\libpng" --config Release
```
