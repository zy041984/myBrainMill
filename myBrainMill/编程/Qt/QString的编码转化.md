# Windows和Qt版本 
Qt5.12.12，windows11
# 程序定义
一个程序的字符集定义为Unicode
c++的Preprocessor没有unicode相关宏
# 结果
QString内部每个元素是QChar即wchar_t
std::wstring内部每个元素是wchar_t

//假设name的内容是"U3d测试"
//则name的内存如下，QString的每个元素是QChar即wchar_t
//0x0055 U
//0x0033 3
//0x0064 d
//0x00e6 0x00b5 0x008b 测的UTF8的双字节序列
//0x00e8 0x00af 0x0095 试的UTF8的双字节序列
//name.toLatin1()返回一个QByteArray，每个元素是QChar即wchar_t,依次保存上述双字节
//QByteArray.data()返回一个char*，每个元素保存了上述双字节的低字节，即形成了一个UTF8的字节序列
//QString::fromUtf8(),输入一个char的UTF8字节序列，构造一个QString，内部使用UTF16编码