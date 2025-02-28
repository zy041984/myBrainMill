# 下载
https://github.com/gabime/spdlog/releases
# 编译
- cmake3.26.5
- 选中FETCHCONTENT_FULLY_DISCONNECTED避免从网上下载源代码
- 选中 BUILD_SHARED,
- 这几项WCHAR_FILENAMES,WCHAR_SUPPORT,BUILD_TEST均未选中，example才编译通过
- USE_STD_FORMAT选不选中，都可以编译通过
- 顺序 Zero_check spdlog example all_build install package
- 看cmakelists.txt，好像**选中了USE_STD_FORMAT就要支持cxx20**
- 如果没选中USE_STD_FORMAT,没选中SPDLOG_FMT_EXTERNAL，没选中 SPDLOG_FMT_EXTERNAL_HO，include就要多几个bundled的fmt头文件
# 使用
见项目TestWinThings\testSpdLog

未知如果log文件名有UTF-8字符，能不能支持