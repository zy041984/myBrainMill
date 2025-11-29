# 安装git客户端
## 地址 https://git-scm.com/download/win
  https://github.com/git-for-windows/git/releases/tag/v2.42.0.windows.2
## 设置git的用户名和密码
  注意git的用户名与github的用户名不一样，git用户名只是用来区分本机谁来提交的代码。
### 打开git bash
### 设置git用户名
```git config --global user.name "myGitName"```
### 确认git用户名 
```git config --global user.name```
为什么git操作github时候总是要密码，这是因为可能在clone URL时使用了HTTPS，据说使用HTTPS remote URL比用SSH有优点。便于初始设置，可以顺利通过防火墙和代理。
为了避免每次输入密码，可以使用Git Credential Manager这样的帮助程序，GCM可以安全地保存身份，安全地通过HTTPS连接GitHub。
GCM自动进行2factor authentication。自动存储你使用的用户名。
# 设置gitHub电子邮件
当你使用自己的用户提交代码时，github使用一个commit电子邮件来标识你的身份。为了不让别人知道你的真实电子邮件地址，可以使用一个隐蔽形式的电子邮件地址。在github账户的emails设置页面选择Keep my email addresses private，然后gitHub会自动为你生成这个电子邮件地址。把这个隐蔽电子邮件地址设置到git即可。
## 通过git端设置隐蔽形式的电子邮件
```git config --global user.email "myMail"```
# 在github网页端创建一个仓库
还不知道怎么选择license为CC-BY-SA，保留署名，允许修改，允许商用，保留同协议
# 通过HTTPS连接你的仓库
从git图形界面clone了一个到本地
# 安装Obsidian客户端
# 建立本地Obsidian的仓库
# 输入各种内容