# 版本
0.8.0
# 下载地址
https://github.com/jbeder/yaml-cpp/releases/tag/0.8.0
# 编译
cmake 3.26.5
win11
x64
取消"BUILD_GMOCK", "BUILD_MOCK", "BUILD_TESTING"
选中"YAML_BUILD_SHARED_LIBS", "YAML_MSVC_SHARED_RT"
# 使用
写入include路径，lib路径，PATH路径，添加lib文件
使用trycatch
如果需要像opencv一样在文件头部写入yaml的版本，可以手动在fstream中写入