#QWindow #QWidget #QDialog #QMainWindow #5.12.12
# QWindow和QWidget、QDialog、QMainWindow的区别
[[QWiget]]继承于QObject和QPaintDevice，是界面以及界面控件的基类，
- 没有嵌在父widget的widget称为window，是顶层的，通常情况下window有标题栏title bar和边框frame。
- 嵌在父widget的widget称为子widget，它的几何通常受父widget的限制。
- 每个QWidget负责自己的绘制。

[[QDialog]]继承于QWidget，集成了确定，取消按钮及其信号和槽。具有模态和非模态。
[[QMainWindow]]继承于QWidget，集成了菜单栏，状态栏，工具栏，dock区域，中间有一个centralWidet

[[QWindow]]继承于QSurface和QObject，封装了操作系统中的一个窗口。猜测应该自行实现了与QWidget类似的功能.
程序通常喜欢用QWidget或QQuickView来实现UI，QWindow似乎是为了便于使用OpenGL或QPainter来绘制的，

#Example
# GraphicsView的一些例子
- anchorlayout显示一个GraphicsScene可以有一个背景颜色``setBackGroundBrush(Qt::darkGreen)``，上面一个对话框
- collidingmice显示一个GraphicsView可以设置一个背景图片
``setBackGroundBrush(QPixmap("*.jpg"))
- dragdroprobot显示一个QGraphicsObject里面可以有painter，进行自定义绘制，绘制成item，添加到scene
- elasticnodes显示一个QGraphicsView可以重载``drawBackGround``虚函数，用painter进行drawText，drawRect
- embeddeddialogs显示一个QGraphicsScene里面可以加入多个QGraphicsProxyWidget，这些窗口可以定义位置，可以互相遮挡，这些proxy窗口可以再自行定义每个里面是什么对话框之类的