[Windows 文件恢复 - Microsoft 支持](https://support.microsoft.com/zh-cn/windows/windows-%E6%96%87%E4%BB%B6%E6%81%A2%E5%A4%8D-61f5b28a-f5b8-3cc2-0f8e-a63cb4e1d4c4)

# 下载
微软应用商店
# 使用
## 常规模式
将 Documents 文件夹从 C： 驱动器恢复到 E： 驱动器上的恢复文件夹。 不要忘记文件夹末尾的反斜杠 (\) 。
`Winfr C: E: /regular /n \Users\<username>\Documents\`
## 广泛模式
将 jpeg 和 png 照片从“图片”文件夹恢复到 E： 驱动器上的恢复文件夹。
`Winfr C: E: /extensive /n \Users\<username>\Pictures\*.JPEG /n\Users\<username>\Pictures\*.PNG`
## 使用哪种模式
使用下表可帮助你确定要使用的模式。 如果不确定，请从常规模式开始。

| 文件系统        | 情况 下     | 建议模式    |
| ----------- | -------- | ------- |
| NTFS        | 最近删除     | Regular |
| NTFS        | 前一段时间已删除 | 广泛      |
| NTFS        | 格式化磁盘后   | 广泛      |
| NTFS        | 损坏的磁盘    | 广泛      |
| FAT 和 exFAT | 任何       | 广泛      |
## 常规语法
下表总结了每个高级开关的用途。

|**参数/开关**|**说明**|**支持的模式 ()**|
|---|---|---|
|Source-drive:|指定文件丢失的存储设备。 必须与目标驱动器不同。|全部|
|Destination-drive:|指定要放置恢复文件的存储设备和文件夹。 必须与源驱动器不同。|全部|
|/regular|常规模式，未损坏的 NTFS 驱动器的标准恢复选项|Regular|
|/extensive|广泛模式，适用于所有文件系统的彻底恢复选项|广泛|
|/n<filter>|使用文件名、文件路径、文件类型或通配符扫描特定文件。 例如：<br><br>- File name: /n myfile.docx<br>    <br>- File path: /n /users/<username>/Documents/<br>    <br>- Wildcard: /n myfile.*<br>    <br>- /n *.docx<br>    <br>- /n *<string>*|全部|
|/?|一般用户的语法和开关摘要。|全部|
|/!|高级用户的语法和开关摘要。|全部|

## 高级语法
下表总结了每个高级开关的用途。

|**开关**|**说明**|**支持的模式**|
|---|---|---|
|/ntfs|NTFS 模式，使用主文件表实现正常 NTFS 驱动器的快速恢复选项|NTFS|
|/segment|段模式，使用文件记录段的 NTFS 驱动器的恢复选项|用户群|
|/signature|使用文件标头的所有文件系统类型的签名模式和恢复选项|签名|
|/y:<type(s)>|恢复特定扩展组，逗号分隔|签名|
|/#|签名模式扩展组和支持的文件类型。|签名|
|/p:<folder>|将恢复操作的日志文件保存在与恢复驱动器上的默认位置不同的位置， (例如， D:\logfile) 。|全部|
|/a|重写用户提示，这在脚本文件中很有用。|全部|
|/u|例如，从回收站中恢复未删除的文件。|NTFS  <br>段|
|/k|恢复系统文件。|NTFS  <br>段|
|/o:<a\|n\|b>|指定在选择是否覆盖文件时，是始终 () ，从不 (n) ，还是始终保留两者 (b) 。 默认操作是提示覆盖。|NTFS  <br>段|
|/g|恢复没有主数据流的文件。|NTFS  <br>段|
|/e|为了保持结果易于管理并专注于用户文件，默认情况下会筛选某些文件类型，但此开关会删除该筛选器。 有关这些文件类型的完整列表，请参阅此表后面的信息。|NTFS  <br>段|
|/e:<extension>|指定筛选的文件类型。 有关这些文件类型的完整列表，请参阅此表后面的信息。|NTFS  <br>段|
|/s:<sectors>|指定源设备上的扇区数。 若要查找扇区信息，请使用 [fsutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil)。|段  <br>签名|
|/b:<bytes>|指定源设备上 (分配单元) 群集大小。|段  <br>签名|