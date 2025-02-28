四种图像类
[[QImage]]，适用于IO，直接读写像素和操作
[[QPixmap]]，适用于屏幕显示
QBitmap，继承于QPixmap，深度为1的一个特例
QPicture，用于记录和回放[[QPainter]]的命令

各种绘制命令都可以记录，使用`QPainter::begin`和`QPainter::end`来包围绘制命令