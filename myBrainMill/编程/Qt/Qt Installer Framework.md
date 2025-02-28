版本4.8.0
用来创建installer的，跨平台，开源和商业都可以用
installer有几个page，每个被安装的包可以包含脚本，可以离线安装可以在线安装
# 步骤
1. 创建一个文件夹，只有子文件夹config和packages
2. 在`config`文件夹里创建一个配置文件`config.xml`，脚本和图标文件也放在`config`文件夹里
        `<Name>ImagePlayer</Name>`字段就是应用程序的exe的名字，快捷方式要链接到这个名字
        `<ControlScript>installscript.js</>`字段就是脚本的名字，关闭了选择组件的那个页面
        `<Banner>small.png</Banner>`是顶部的图片
        `<InstallerWindowIcon>icon<`是一个图标
        `<InstallerApplicationIcon>icon2<`是一个图标
        `<WizardStyle>Mordern<`是对话框风格，好像这个modern要好一些
        `<WizardShowPageList>true<`指的是左侧有竖着的那个窗口列表
3. 在`config`文件夹里创建一个脚本文件`installscript.js`，内容就是关闭某个页面
4. 在`package\com.我的公司.我的程序\meta`文件夹里创建一个配置文件`package.xml`，脚本、ui、ts、qm也放到这个文件夹里
    `<Script>installscript.qs<`字段就是创建快捷方式的按钮的脚本
    `<UserInterface>readmecheckboxform.ui<`字段就是创建快捷方式的按钮的窗口
    `<Translation>zh_CN.qm<`字段就是创建快捷方式的按钮的窗口的翻译文件
5. 在`package\com.我的公司.我的程序\meta`文件夹里创建一个脚本文件`installscript.qs`，内容就是响应按钮创建快捷方式
6. 为ui文件创建翻译，打开qt的命令行，输入`lupdate "E:\ImagePlayer\packages\com.yonke.yonke MR system ImagePlayer\meta\desktopshortcutcheckboxform.ui" -ts "E:\ImagePlayer\packages\com.yonke.yonke MR system ImagePlayer\meta\zh_CN.ts"`
7. 把上述ts文件在qt翻译家里制成qm文件，放在同目录下
8. 把要安装的内容压缩成zip文件，复制到`package\com.我的公司.我的程序\data`，zip文件不要有子文件夹，
9. 使用`binarycreator`创建安装包，在cmd中切换到文件夹`E:\ImagePlayer\`输入
    `C:\Qt\QtIFW-4.8.0\bin\binarycreator.exe -c config\config.xml -p packages 我的安装程序.exe`
10. 这样就会生成安装程序，执行完成后会在桌面和开始菜单添加快捷方式。
11. 文件夹里的`maintenancetool.exe`是卸载工具，会自动删除安装目录和快捷方式
## 问题
1. 翻译文件不好用
2. 没能在开始菜单生成文件夹，config.xml里的设置好像没用
    `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\MR`
    `C:\Users\Admin\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\MR`
    两个文件夹里都没有在开始菜单中显示文件夹MR，只能把imagePlayer显示在开始菜单
    这个问题好像是只装一个程序是不能在开始菜单中显示文件夹的，只有装两个程序，才能在开始菜单中显示文件夹