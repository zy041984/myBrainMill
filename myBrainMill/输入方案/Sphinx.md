# 安装
[Installing Sphinx — Sphinx documentation (sphinx-doc.org)](https://www.sphinx-doc.org/en/master/usage/installation.html#installation-of-the-latest-development-release)
在powershell里面输入`pip install -U sphinx`
# 使用
包含`conf.py`的文件夹为源文件夹
在index.rst中添加目录的文件

```
usage/installation
usage/quickstart
```

然后新建文件夹usage，新建文件installation.rst和quickstart.rst，编辑两个文件的内容

在powershell中运行如下命令
`sphinx-build -M html "E:\work\testRestructedText\source" "E:\work\testRestructedText\build"`
即可生成html的帮助文档
中间有一些报错，

# vscode
vscode安装了reStructeredText的扩展，
python安装了linter的扩展`pip install doc8`
vscode安装了esbonio扩展来实现预览rst文档