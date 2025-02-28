# 下载
https://github.com/microsoft/vcpkg/releases
# 安装
运行下列脚本，需要能连接到github
`.\vcpkg\bootstrap-vcpkg.bat -disableMetrics`
# 使用
先把源代码下载到download文件夹里面
然后再执行如下
`vcpkg install [packages to install]:x64-windows` 
## 使用vs
`vcpkg integrate install`
然后可以在IDE中创建或打开一个project。所有安装的库应该可以自动被intelliSense探测，并使用。
## 使用CMake
如果在IDE外面在CMake辅助下使用vcpkg，可以使用工具链文件
```
cmake -B [build directory] -S . -DCMAKE_TOOLCHAIN_FILE=[path to  vcpkg]/scripts/buildsystems/vcpkg.cmake
```
然后就可以使用cmake进行build
```
cmake --build [build directory]
```

# overview
## 2 设置项目
### 2.1 设置当前shell对话的临时环境变量
```
set VCPKG_ROOT="D:\vcpkg-2023.10.19"
set PATH=%VCPKG_ROOT%;%PATH%
```
### 2.2 创建项目文件夹
`mkdir helloworld && cd helloworld`
### 2.3 添加项目需要的文件
使用命令`vcpkg new --application`创建一个属性文件`vcpkg.json`
使用命令`vcpkg add port zlib`为此项目添加依赖zlib
文件内容类似下面
### 2.4 编辑Cmake需要的cmakeLists.txt
为了让cmake能识别到vcpkg提供的库，需要为cmakeLists.txt提供vcpkg.cmake工具链文件。新建一个CMakePresets.json文件，写明cmake执行的preset，文件夹，指定vcpkg.cmake文件位置。
### 2.5 使用cmake进行编译构建

# 一些概念
## 与编译构建系统的集成
### 与CMake集成
通过提供一个工具链文件来实现与find_package的无缝集成。在调用`CMAKE_TOOLCHAIN_FILE`或编写`CMakePresets.json`文件时，指定`<vcpkg root>/scripts/buildsystems/vcpkg.cmake`这个工具链文件。
之后find_package、find_library、find_path都可以从vcpkg的安装目录来找到需要的依赖库。
vcpkg也可以把manifest文件里声明的依赖库自动安装
### 手动集成
- 在经典模式下，第三方依赖库位于`<vcpkg root>/installed`。
- 在manifest模式下，第三方依赖库位于`<vcpkg.json directory>/vcpkg_installed`
头文件位于include
release的lib位于lib
release的dll位于bin
release的pc位于lib/pkgconfig或share/pkgconfig
debug的lib位于debug/lib
debug的dll位于debug/dll
debug的pc位于/debug/lib/pkgconfig或debug/share/pkgconfig
工具位于tools
例如经典模式下zlib位于`<vcpkg root>/installed/x64-windows/`文件夹，**有个疑问是否所有的库混在一起**

## port
- port是产生一系列文件的带版本的菜单。执行一个port会产生新的头文件或exe，从而影响安装图。
      port之间可以互相依赖，指定某些feature的时候可能造成新的依赖关系。
      port包括包的名字，版本，支持的feature，依赖，编译和安装本包的一些命令流程。
- port包含一个portfile.cmake的脚本文件，指定了怎么下载源代码，编译，安装
- vcpkg.json文件描述了port里面包含的包的元数据，包的名字，版本，依赖，平台，feature。
- 有时候port会有patch文件，包含了对源代码的补丁。
- 分类
    - 标准port，为了从源文件编译构建包
    - 元数据port，为了在多个包之间形成安装图
    - 脚本port，有可能只是一些脚本
## consuming package的模式
- 经典模式
- manifest模式
推荐manifest模式
经典模式下，每个vcpkg的instance维护一个集中的安装树。
manifest模式下，使用一个名为vcpkg.json的文件来描述项目或包的元数据。
包里的manifest文件
所有vcpkg的ports必须包含一个vcpkg.json来描述即将安装的包。这个文件用来记录包的依赖树信息，github地址，解决版本冲突。
项目里的manifest文件
项目里面要使用manifest文件来声明依赖及其版本
可以使用一个vcpkg-configuration.json文件来增加更多包

https://www.cnblogs.com/vcpkg/p/15019867.html