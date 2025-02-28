# 大概三种办法
- 当前环境Locale是GB2312时，直接定义``std::wstring text(L"中文")``，这时观察内存使用了UTF16BE，``osgText::Text::setText``可以接收形参为wchar_t*，
然后``osgText::Text::setFont("simfang.ttf")``使用中文字体即可。
实现了中英文数字混合在一个wstring

- 当前环境Locale是GB2312时，直接定义``std::wstring text(L"中文")``，这时观察内存使用了UTF16BE。调用``WideCharToMultiByte(CP_ACP``, 把UTF16编码的字符串text转换为了GB2312的``std::string nameStr``，然后``osgText::Text::setText(nameStr.c_str()， osgText::String::ENCODING_CURRENT_CODE_PAGE)``
然后``osgText::Text::setFont("simfang.ttf")``使用中文字体即可。
实现了中英文数字混合在一个wstring

- 当前环境locale是GB2312时，直接定义``std::wstring text(L"中文")``，这时观察内存使用了UTF16。调用``WideCharToMultiByte(CP_UTF8)``，把UTF16编码的字符串text转换为了UTF8的``std::string nameStr``，然后``osgText::Text::setText(nameStr.c_str()，
osgText::String::ENCODING_UTF8)``
然后``osgText::Text::setFont("simfang.ttf")``使用中文字体即可。
实现了中英文数字混合在一个wstring

查看osg的代码，发现使用``ENCODING_CURRENT_CODE_PAGE``，其实就是用osg内部的函数把宽字符串转换为UTF8的字符串，再把UTF8的字符串转化为UTF16BE，最后保存在``osgText::TextBase``里面的是UTF32BE。

查看osg的代码，发现使用``ENCODING_UTF8``，其实就是把UTF8的字符串转化为UTF16BE，最后保存在``osgText::TextBase``里面的是UTF32BE。

# 总结

- ``osgText::Text``的setText可以接收三种编码，
UTF16BE即wchar_t，
UTF8即char，
CURRENT_CODE_PAGE即char

- QString可以产生三种编码，
fromWCharArray和toStdWString产生UTF16BE即wchar_t，
fromWCharArray和toStdString产生UTF8即char，
fromWCharArray和toLocal8Bit产生UTF8即char

- winAPI可以产生三种编码，
L宏可以产生UTF16BE即wchar_t， 
WideCharToMultiByte(CP_UTF8可以产生UTF8即char，
WideCharToMultiByte(CP_ACP可以产生CURRENT_CODE_PAGE即char

- 如果有把握字符串都是UTF8的char，那就最方便。
- 如果有把握让L宏产生的就是UTF16的wchar_t，可以使用qt或WideCharToMultiByte(CP_UTF8来把UTF16转成UTF8的char

- 注意osg一定要使用中文字体

- 在c++项目中最简单的办法
    - 在编译选项添加`/source-charset:utf-8 /execution-charset:utf-8 `
    - characterSet用`Use Unicode Character Set`或`Use Multi-Byte Character Set`好像都行
    - 代码写```
```
auto strUTF = u8"中文hello";
osgText::Text::setText(strUTF, osgText::String::ENCODING_UTF8)
```

[字符串和字符文本 (C++) | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/cpp/string-and-character-literals-cpp?view=msvc-140)