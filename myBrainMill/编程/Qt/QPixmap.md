四种图像类
[[QImage]]，适用于IO，直接读写像素和操作
QPixmap，适用于屏幕显示
QBitmap，继承于QPixmap，深度为1bit的一个特例
[[QPicture]]，用于记录和回放QPainter的命令


QPixmap继承于[[QPaintDevice]]，是一种离屏的图形表述方式，可以使用QLabel或QAbstractButton来显示在屏幕上，[[QPainter]]可以直接在QPixmap上绘制。
不能直接访问像素。
可以与[[QImage]]之间互相转换
可以对图像使用矩阵进行变换