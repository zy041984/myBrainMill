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

# 代码
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