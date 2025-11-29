# 如何在vs code中调试c++
https://code.visualstudio.com/docs/languages/cpp
https://code.visualstudio.com/docs/cpp/config-msvc#_prerequisites
## 安装
- 安装vs code
- 安装vs c++ extension
- 安装vs studio，确认安装了c++ workload
- 验证具有ms的c++编译器
在developer command prompt for vs studio中输入cl，可以输出版本信息。注意这里可能要使用x64的版本
## 从x64 developer commander prompt中启动vs code
```
mkdir projects
cd projects
mkdir helloworld
cd helloworld
code .
```
切换到盘符，cd到某个目录，运行上述命令，就可以在developer commander prompt中打开vscode。
这样就在vscode中打开了工作区，过后创建了三个文件
- task.json用于build instruction如MDd，include，lib，define，x64
- launch.json用于debugger settings，如path，argu，
- c_cpp_properties.json用于编译器路径和define和include和windowsSDK和intellisense设置
## 调试
- 添加代码文件
- 点击play按钮
- 在弹出的窗口中选择cl.exe
这样就会编译代码，生成exe和pdb和tasks.json。以后再次编译时，使用tasks.json中的设置。
可以添加断点，调试运行
可以创建一个launch.json文件，保存调试的配置。例如argu，path
- 还可以在命令面板ctrl+shift+p输入命令`C/C++: Edit Configurations (UI)`，即生成c_cpp_properties.json文件，并在弹出的UI中编辑c_cpp_properties.json文件。例如增加include，define，设定winSDK，设定c++版本。这个配置文件可以复制到其它的项目中去
- 另外，如果SSH远程调试，就没办法在developer command prompt中启动vscode。可以在tasks.json中增加一个路径，指定VsDevCmd.bat
# 使用CMake构建工程

## 安装
- 可以使用vs自带的cmake，也可以使用自己下载的。但要保证cmake的exe路径在PATH中，或者在配置`cmake.cmakePath`里面，`cmake.cmakePath`配置的值写为`C:\cmake-3.30.8-windows-x86_64\bin\cmake.exe`
- 在vs code中安装CMake Tools扩展
## 配置项目
两种方法，使用CMake preset或CMake kits/Variants
下面按照推荐使用CMake preset，把配置写在一个json文件里，这个文件甚至可以分享给其它编译环境
- 在developer command prompt中打开vscode，导航到cmakeList.txt所在文件夹
- 在`ctrl+shift+p`打开命令面板，输入`CMake:Quick Start`，选择编译器amd64，选择  `add a new preset`，接下来选择`create from a compiler`，接下来的编译器选择`x64`，起个名字为`x64`，自动创建`CMakePresets.json`
- 在`ctrl+shift+p`打开命令面板，输入`CMake:Configure`，成功后输入`CMake:Build`
## 什么是CMake的Configure
它是一个过程，检测必须的第三方库，检测一些编译设置，然后产生`cmake cache`，后续build产生最终的工程文件
`CMake Cache`是kv对，除非删除或重新初始化，即使多次运行config，这些kv对存储的信息也不会变化。
所以可以把`-flag`和`#include`文件写在这里，这些数值很难在运行`config`期间获得
可以把固定不变的信息写在这里，例如第三方库的位置
可以把一些编译配置写在这里，用于控制最后的工程文件，例如是否`BUILD_TESTING`
一般都是用`-D`来定义Config中的参数
最终根据本机所安装的工具链生成工程文件，如本机安装了ms，则最终生成sln和project
CMake Tools插件根据文件里写的配置使用cmake-file-api来驱动cmake干活
可以使用命令`CMake:Delete Cache and Reconfigure`来删除旧的文件并重新计算新的,
可以使用命令`CMake:Configure with Debugger`来调试config过程，这可以看到cmakeList.txt被配置的过程
## 两种配置文件
`CMakePresets.json`和`CMakeUserPreset.json`
`CMakePresets.json`用来存储工程相关的builds，`CMakeUserPreset.json`用来存储开发者本地的builds
CMake tools集成了`CMakePreset.json`，并且支持版本2，从命令行调用`CMake`需要在3.20以上的版本
把`cmake.useCMakePresets`设置为`auto`，则当`cmakeList.txt`文件夹下如果有`CMakePresets.json`文件，则启用了
## build
build的结果写入文件夹`cmake.buildDirectory`，默认的目标是`ALL_BUILD`，可以点击铅笔按钮来选择要执行的目标，这些目标有些是来自`CMakeList.txt`的，有些是来自于文件`CMakePresets.json`的，
也可以把多个build写成一个task，存储在`tasks.json`文件中，一次执行多个build
CMake Tools把`--build`的参数传递给cmake
build环境是从调用vs code的进程继承来的，也来自cmake的环境
LAUNCH就是一个sln中，点击debug按钮时，开始执行的那个project
DEBUG栏中可以选择调试的目标，可以创建一个文件`launch.json`来指定调试的命令行参数和目录
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(msvc) Launch",
            "type": "cppvsdbg",
            "request": "launch",
            // Resolved by CMake Tools:
            "program": "${command:cmake.launchTargetPath}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [
                {
                    // add the directory where our target was built to the PATHs
                    // it gets resolved by CMake Tools:
                    "name": "PATH",
                    "value": "${env:PATH}:${command:cmake.getLaunchTargetDirectory}"
                },
                {
                    "name": "OTHER_VALUE",
                    "value": "Something something"
                }
            ],
            "externalConsole": true
        }
    ]
}
```