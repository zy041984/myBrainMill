# PE文件格式(用于代码逆向分析)
**PE文件是指32位可执行文件，也称PE32；64位的可执行文件称为PE+或PE32+，是PE文件的一种扩展形式。EXE、DLL、SYS、COM都是PE文件，**
# 基本结构
| -头- | -头的部分- | 子模块 | 名称 | 字节 |
| ---- | ---- | ---- | ---- | ---- |
| 头 | DOS头 | MZ头 | IMAGE_DOS_HEADER | 64字节 |
| - | - | DOS块 | MS-DOS stub Program | 不定 |
| - | NT头 | Signatrue | IMAGE_NT_HEADERS | 4个字节，开始位置在MZ头中指定 |
|  |  | IMAGE_FILE_HEADER |  | 20字节 |
|  |  | IMAGE_OPTIONAL_HEADER |  | 字节数量在IMAGE_FILE_HEADER中指定 |
| - | Section头 | - |  | 每个section有40个字节，section的数量在IMAGE_FILE_HEADER中指定 |
| 数据 | - | - |  |  |

[【逆向】【PE入门】使用PEView分析PE文件-CSDN博客](https://blog.csdn.net/qq_43633973/article/details/102378477)
[PE文件结构及其加载机制 - bokernb - 博客园 (cnblogs.com)](https://www.cnblogs.com/bokernb/p/6116512.html)
