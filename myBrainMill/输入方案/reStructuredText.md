spec
https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html

编辑工具
vsCode

发布工具
sphinx
https://www.sphinx-doc.org/zh-cn/master/usage/index.html

python下安装`pip install -U sphinx`

然后在希望工作的目录内，运行`sphinx-quickstart`，选择语言zh_CN，
这样将建立source和build两个目录，并在source下有一个index.rst文件
编辑index.rst，用文件名来插入顶级目录
然后在每个目录下新建那个文件名，并开始编辑即可。每个文件至少要有title，然后可以设置subtitle，section。可以交叉引用
然后运行`make html`就会在build文件夹下生成一个html文件夹，查看index.html即可

还没太看懂怎么从代码生成文档，估计是代码的注释要写的很好

[Sphinx + Read the Docs 从懵逼到入门 - 文章详情 (itpub.net)](https://z.itpub.net/article/detail/A330D3FEC5B63BEB1005AD0967DAA6D3)
[Sphinx和rst在科研笔记和学术博客中的高效用法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/143141024)