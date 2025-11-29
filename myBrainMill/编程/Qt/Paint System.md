Qt的paint system使得在屏幕和打印设备上绘制时可以使用同一套API，基于三个类`QPainter`，`QPaintDevice`，`QPaintEngine`。
[[QPainter]]用来执行绘制操作
[[QPaintDevice]]是一个二维空间，可以被绘制
QPaintEngine为[[QPainter]]提供了绘制的接口，内部使用。
这样所有的绘制可以按照相同的流水线来进行，将来为新特性提供支持时可以很方便地实现。
# 绘制的类
图元 QPoint QLine QVector QIcon QPainterPath QPolygon QRect QRegion
图像 QBitmap QImage QPixmap
支持类 QFont QGradient QColor QMargin QSize QTransform QRgba64 QSVGGenerator QPen QBrush
# Paint的Device和Backend
## device
- Widget，被绘制在屏幕上的GUI
- Image，硬件独立的图像
- Pixmap，离屏的图像，为在屏幕上显示进行了优化
- QOpenGLPaintDevice，专门为QPainter使用OpenGL进行绘制而提供的
- Picture，为了记录QPainter的绘制动作
## backend
如果需要实现新的显示端，自行继承[[QPaintDevice]]并实现虚函数QPaintDevice::paintEngine即可

# 绘制和填充
基本图元使用[[QPainter]]直接绘制，
路径使用QPainterPath，
需要填充的路径使用QPainterPathStroke
QPen指定线型和宽度，及末端样式，连接处的样式
QBrush指定填充的图案Qt::BrushStyle,甚至可以指定纹理图案以及渐变QGradient，颜色QColor
QFont指定绘制文本时的字体
抗锯齿使用`QPainter::RenderHint`指定
# 坐标系统
二维平面，原点在左上，x向右，y向下，单位是屏幕的像素或打印机的点(一英寸的1/72)。一般来说打印机的物理分辨率要高一些，每英寸600个点。屏幕的物理分辨率一般是每英寸72到100个点
绘制图元时，会指定图元的位置，尺寸，这是基于逻辑坐标的，会按照零宽度的线条绘制基本图形。但是实际上会设置大于0的绘制宽度，因此哪些像素将被填充，这是按抗锯齿和绘制宽度来共同决定的。
假设给定坐标绘制一个点，点的尺寸为1像素，则该坐标的右侧和下侧那个方格会被填充。如果设置了抗锯齿，不同尺寸的宽度会导致不同的像素格被填充。
[[QPainter]]的逻辑坐标到[[QPaintDevice]]的物理坐标的转化由[[QPainter]]的矩阵，viewport和window来完成。
逻辑坐标和物理坐标初始时重合，原点都在左上。可以使用矩阵对逻辑坐标系随意变换，然后绘制，并指定逻辑坐标系的一个矩形区域即window被显示。这个window区域被显示在[[QPaintDevice]]用物理坐标描述的对应区域或指定区域即viewport。
[[QPainter]]绘制时使用的是逻辑坐标，这个坐标然后会被转化到[[QPaintDevice]]的物理坐标。使用到[[QPainter]]的worldTransform(),viewport(),window()三个函数。
貌似[[QPainter]]绘制时使用逻辑坐标，可以把逻辑坐标按照矩阵进行一定转换，二维平面的哪些部分被显示是由setWindow来设置，这个区域随后再被映射到[[QPaintDevice]]的一个矩形物理区域，这个矩形物理区域由setViewport来设置。
# 读写图像文件
- 使用[[QImage]]或[[QPixmap]]的构造函数。
- 如果想要更多控制，可以使用QImageWriter和QImageReader。
- 如果想读入动画，可以使用QMovie