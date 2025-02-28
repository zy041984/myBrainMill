# 网上的一些旧说法
查询是否安装了
Microsoft Visual C++ 2022 X64 Debug Runtime - 14.36.32532网上说要检查注册表键值
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{44B8E53D-68C7-4FCD-A0D7-753CA2C2EF94}`
但是这个键值随版本变化，不好确定当前电脑装了哪个版本

# 微软官方对2015-2022的说明
在微软官方找到如下说明
[重新分发 Visual C++ 文件 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/windows/redistributing-visual-cpp-files?view=msvc-170&source=recommendations)
现在2015-2022是同一个安装包
最新版本是 `14.40.33810.0`，这个版本经常更新，不区分语言，只区分x86和x64
安装程序会自动检测当前计算机上是否有更新版本，如果已有版本比较低，就可以更新。
当前安装的版本号存储在 **`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\VisualStudio\14.0\VC\Runtimes\{x86|x64|arm64}`** 项中。
版本号存储在 `REG_SZ` 字符串值 **`Version`** 中，也存储在 **`Major`**、**`Minor`**、**`Bld`**、**`Rbld`** 和 `REG_DWORD` 值的集中。 为了避免在安装时出错，如果当前安装的版本较新，必须跳过可再发行程序包的安装。
# 做了实验
自己实验从vs2015-2019更新到2015-2022，过程中会删除注册表中一些旧的项，添加新的注册表项，以及改写旧的注册表值。
# 安装了vc redistritubable 2015-2019版本14.29.30153
在注册表中搜索`14.29.30153`，关键的项如下
```
HKEY_CLASSES_ROOT\Installer\Products\4F221FE4AD7FB5F47A1AF37EEA4358D0
HKEY_CLASSES_ROOT\Installer\Products\E69030F0F18F0D84EA0EF95831DC88F3
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Installer\Products\4F221FE4AD7FB5F47A1AF37EEA4358D0
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Installer\Products\E69030F0F18F0D84EA0EF95831DC88F3
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeAdditional
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeMinimum
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\14.0\VC\Runtimes\X64
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\4F221FE4AD7FB5F47A1AF37EEA4358D0
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\E69030F0F18F0D84EA0EF95831DC88F3
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{0F03096E-F81F-48D0-AEE0-9F8513CD883F}
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{4EF122F4-F7DA-4F5B-A7A1-3FE7AE34850D}
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeAdditional
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeMinimum
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\VisualStudio\14.0\VC\Runtimes\X64
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\{9057ceb3-ab14-4d3a-aa99-38d2d660e604}
```

已经可以跑PT程序了

# 卸载vc redistritubable 2015-2019版本14.29.30153
卸载之后PT程序就不能跑了，提示找不到MSVCP140.dll,VCRUNTIME140_1.dll,VCRUNTIME140.dll
# 卸载vc redistritubable 2015-2019版本14.29.30153
卸载之后的注册表
`HKEY_CLASSES_ROOT\Installer\Products\`下面已经没有`4F221FE4AD7FB5F47A1AF37EEA4358D0`和`E69030F0F18F0D84EA0EF95831DC88F3`
`HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Installer\Products\`下面已经没有`4F221FE4AD7FB5F47A1AF37EEA4358D0`和`E69030F0F18F0D84EA0EF95831DC88F3`
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft`下面已经没有`DevDiv`和`VisualStudio`
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\`下面已经没有`4F221FE4AD7FB5F47A1AF37EEA4358D0`和`E69030F0F18F0D84EA0EF95831DC88F3`
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`下面已经没有`{0F03096E-F81F-48D0-AEE0-9F8513CD883F}`和`{4EF122F4-F7DA-4F5B-A7A1-3FE7AE34850D}`
`HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\DevDiv`下面已经没有`VC`
`HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft`下面已经没有`VisualStudio`
`HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\`下面已经没有`{9057ceb3-ab14-4d3a-aa99-38d2d660e604}`

# 安装vc redistritubable 2015-2022版本14.40.33810  
EULAID Cpp_2015-2022_CHS.2052
PT程序可以运行了
注册表搜索14.40.33810，多了以下项
```
HKEY_CLASSES_ROOT\Installer\Products\A4BB3B8BD01A15F4197B6AF4AF3CE17A
HKEY_CLASSES_ROOT\Installer\Products\F84DEC95EFBEC084A883CF70C9B2CEF0
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Installer\Products\A4BB3B8BD01A15F4197B6AF4AF3CE17A
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Installer\Products\F84DEC95EFBEC084A883CF70C9B2CEF0
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeAdditional
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeMinimum
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\14.0\VC\Runtimes\X64
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\A4BB3B8BD01A15F4197B6AF4AF3CE17A
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\F84DEC95EFBEC084A883CF70C9B2CEF0
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{59CED48F-EBFE-480C-8A38-FC079C2BEC0F}
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\{B8B3BB4A-A10D-4F51-91B7-A64FFAC31EA7}
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeAdditional
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\DevDiv\VC\Servicing\14.0\RuntimeMinimum
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\VisualStudio\14.0\VC\Runtimes\X64
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\{5af95fd8-a22e-458f-acee-c61bd787178e}
```

# 不卸载2015-2019，直接安装2015-2022
在注册表中搜索14.29.30153已经搜不到了
`HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\VisualStudio\14.0\VC\Runtimes\X64`的`Major`为14，`Minor`从29变为40，`Bld`从30153变为33810，`installed`为1

