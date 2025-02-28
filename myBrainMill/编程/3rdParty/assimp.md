# 下载
https://github.com/assimp/assimp
# 编译
使用cmake
选择使用自己的zlibstaticd.lib和zlibstatic.lib
顺序
UpdateAssimpLibsDebugSymbolsAndDlls
assimp
uninstall
unit
INSTALL

项目选项C++中添加`/source-charset:utf-8 /execution-charset:utf-8` 
# 使用
OpenGL加载无贴图有材质的模型
https://github.com/assimp/assimp/blob/master/samples/SimpleOpenGL/Sample_SimpleOpenGL.c

发现还是使用OpenGL1.x版本的函数生成list