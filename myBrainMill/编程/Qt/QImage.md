四种图像类
QImage，适用于IO，直接读写像素和操作
[[QPixmap]]，适用于屏幕显示
QBitmap，继承于QPixmap，深度为1的一个特例
[[QPicture]]，用于记录和回放QPainter的命令


QImage继承于[[QPaintDevice]]，是一种独立于硬件的图形表述方式，允许直接读写像素数据，可以作为[[QPaintDevice]]。
可以用`uchar*`和尺寸来构造，可以转换格式，可以抠色形成mask，可以scaled，可以直接使用[[QPainter]]来在图像上绘制
32位格式的直接存储ARGB信息。
像素数据的起点貌似在左上角，每个像素的数据是一个整数
可以提供一个矩阵来对图像进行变换