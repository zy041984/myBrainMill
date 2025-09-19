# 下载
https://github.com/gabime/spdlog/releases
# 1.12.0
## 编译选项
- cmake3.26.5
- vs2022 x64 vc143 WinSDK10.0.26100
- 选中FETCHCONTENT_FULLY_DISCONNECTED避免从网上下载源代码
- 选中 BUILD_SHARED,
- 这几项WCHAR_FILENAMES,WCHAR_SUPPORT,BUILD_TEST均未选中，example才编译通过
- USE_STD_FORMAT选不选中，都可以编译通过
- 顺序 Zero_check spdlog example all_build install package
- 看cmakelists.txt，好像**选中了USE_STD_FORMAT就要支持cxx20**
- 如果没选中USE_STD_FORMAT,没选中SPDLOG_FMT_EXTERNAL，没选中 SPDLOG_FMT_EXTERNAL_HO，include就要多几个bundled的fmt头文件
## 使用
见项目TestWinThings\testSpdLog
未知如果log文件名有UTF-8字符，能不能支持，后面测试，需要修改选项重新编译
## 代码
```
//1编译的库定义了SPDLOG_USE_STD_FORMAT，本项目也要定义SPDLOG_USE_STD_FORMAT，且使用c++20
//定义了SPDLOG_USE_STD_FORMAT必须用c++20
#include "spdlog///spdlog.h"
#include "spdlog/sinks/basic_file_sink.h"
#include "spdlog/sinks/stdout_color_sinks.h"
//spdlog
//输出到console的spdlog
auto console_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
if (level.compare(QString("debug"), Qt::CaseInsensitive) == 0)
	console_sink->set_level(spdlog::level::debug);
else if (level.compare(QString("trace"), Qt::CaseInsensitive) == 0)
	console_sink->set_level(spdlog::level::trace);
else
	console_sink->set_level(spdlog::level::info);
console_sink->set_pattern("[multi_sink_example] [%^%l%$] %v");
//输出到文件的//spdlog
//获得系统时间
std::chrono::time_point now{ std::chrono::system_clock::now() };//需要c++17
std::time_t tt = std::chrono::system_clock::to_time_t(now);
tm tmGMT;
gmtime_s(&tmGMT, &tt); //GMT (UTC)
tm tmLocal;
localtime_s(&tmLocal, &tt);//Locale time-zone, usually UTC by default.
char mbstr[100];
std::strftime(mbstr, sizeof(mbstr), "%F-%H_%M_%S", &tmLocal);//系统时间转格式文本
//获得日志文件的相对路径名
std::string fileName = appPath + std::string("\\logs\\") + std::string(mbstr) + std::string(".txt");
auto file_sink = std::make_shared<spdlog::sinks::basic_file_sink_mt>(fileName.c_str(), true);
if (level.compare(QString("debug"), Qt::CaseInsensitive) == 0)
	file_sink->set_level(spdlog::level::debug);
else if (level.compare(QString("trace"), Qt::CaseInsensitive) == 0)
	file_sink->set_level(spdlog::level::trace);
else
	file_sink->set_level(spdlog::level::info);
bool useConsoleAndFile = (toConsole != 0);
if (useConsoleAndFile)
{
	spdlog::logger logger("multi_sink", { console_sink, file_sink });
spdlog::set_default_logger(std::make_shared<spdlog::logger>(logger));
}
else
{
	//spdlog::logger logger("console_sink", console_sink);
	spdlog::logger logger("file_sink", file_sink);
spdlog::set_default_logger(std::make_shared<spdlog::logger>(logger));
}
if (level.compare(QString("debug"), Qt::CaseInsensitive) == 0)
	spdlog::set_level(spdlog::level::debug);
else if (level.compare(QString("trace"), Qt::CaseInsensitive) == 0)
	spdlog::set_level(spdlog::level::trace);
else
	spdlog::set_level(spdlog::level::info);
spdlog::set_pattern("[%H:%M:%S %z] [%^%L%$] [thread %t] %v");
////spdlog::flush_every(std::chrono::seconds(3));
spdlog::info("Welcome to //spdlog version {}.{}.{}  !", SPDLOG_VER_MAJOR, SPDLOG_VER_MINOR, SPDLOG_VER_PATCH);
// Release all spdlog resources, and drop all loggers in the registry.
// This is optional (only mandatory if using windows + async log).
//spdlog::shutdown();
```
## 20250811更新
更新了windowsAPI，需要重新编译这个库，趁机测试一下能不能输出中文日志，测试结果成功。
### 编译
选中WCHAR_FILENAMES,WCHAR_SUPPORT,USE_STD_FORMAT,BUILD_SHARED
vs的character set选中Unicode
### 测试工程
- 编译选项在Preprocess Definition中加入`SPDLOG_WCHAR_TO_UTF8_SUPPORT;SPDLOG_WCHAR_FILENAMES;SPDLOG_USE_STD_FORMAT;_DEBUG;_CONSOLE`
- 加入`/source-charset:utf-8 /execution-charset:utf-8`同时cpp文件保存选项为UTF8NoBOM，即源文件编码形式为UTF8，exe的执行字符集设置为UTF8
- vs的character set选中Unicode
### 测试代码加入
- 控制台输出中文字符
`SetConsoleOutputCP(CP_UTF8);`
- 日志文件名为中文
- spdlog也可以输出中文
```
std::wstring tmpWStr(L"好");
spdlog::info(L"{}", tmpWStr.c_str());
spdlog::info(L"中文");
```
### 结果
日志文件名支持中文，文件内容有中文字符，同时控制台也能输出中文。
# 版本1.15.1
## 编译
### cmake选项
取消cpack
选中BUILD_SHARED,INSTALL,MSVC_UTF8,USE_STD_FMT,WCHAR_CONSOLE,WCHAR_FILENAMES,WCHAR_SUPPORT,FWRITE_UNLOCKED
### 用msvc2022 x64
编译spdlog成功，生成了dll和lib
example如果要成功，需要把某些字符串前面加L变成wchar，运行看起来成功了
install成功
## 使用
同上面例子
加入定义SPDLOG_USE_STD_FORMAT，选择c++20