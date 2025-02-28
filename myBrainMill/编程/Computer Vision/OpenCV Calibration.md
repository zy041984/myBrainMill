4.8.0
[OpenCV: Camera Calibration and 3D Reconstruction](file:///D:/download/opencv/doc-4.8.0/d9/d0c/group__calib3d.html)
对于针孔相机模型，distortion是常数，可以被标定即矫正。
OpenCV考虑的distortion分为径向radial和切向tangential两种
- 径向distortion产生鱼眼镜头或长炮镜头效果
    方程为
    $$x_{distorted}=x \left( 1+k_1r^2+k_2r^4+k_3r^6 \right) \\ y_{distorted}=y \left( 1+k_1r^2+k_2r^4+k_3r^6 \right) $$
- 切向distortion的原因是透镜与成像平面不平行
    方程为
    $$
x_{distorted}=x+2p_1xy+p_2\left( r^2+2x^2 \right) \\  y_{distorted}=y+2p_2xy+p_1\left( r^2+2y^2 \right)
$$
使用$1*5$矩阵来描述$distortion cofficients=\left( k_1,k_2,p_1,p_2,k_3\right)$

对于相机$3*3$矩阵，未知的参数就是焦距$f_x,f_y$和中心偏移$c_x,c_y$
确定以上两个矩阵的过程就是标定。

OpenCV使用的标定物有三种，棋盘格，对称或非对称的圆型，ChArUco的板
最好拍10张不同姿态的照片
## 寻找内角点
棋盘格寻找的是内角点，ChArUco板也寻找内角点，但是corner用ArUco marker来标志。圆型标定物就是找圆形本身。
ArUco返回可见的pattern和可见corner的id和坐标
`cv::cornerSubPix`可以对粗略计算的角点位置进一步优化
每张照片寻找到的点保存在`std::vector<cv::Point2f> pointbuf`
所有照片找到的点保存在`std::vector<std::vector<cv::Point2f>> imagePoints`
## 标定计算
标定的函数是`cv::calibrateCameraRO`，参数如下
- 物点三维坐标`objectPoints`，z坐标为0，所有内角点的坐标用calcBoardCornerPositions计算。
    如果有左上内角点到右上内角点精确的测量结果单位毫米或像素，可以写在`grid_width`里面
- 更新的物点坐标，标定计算过后，内角点的三维坐标写在`newObjPoints`里面
- 物点二维坐标`imagePoints`，每张照片中，每个内角点的二维坐标
- `iFixedPoint`如果使用新的标定算法，需要传递将要被修正的点的id，即`boardSize.width-1`。如果使用旧的标定算法，此参数为-1即可。
- 输出参数就是相机的标定矩阵，distort矩阵，拍摄每个相片时相机的旋转矩阵和平移矩阵，重新计算的内角点世界坐标`newObjPoints`
## 计算重投影误差
已知相机的标定矩阵，distort矩阵，拍摄每个相片时相机的旋转矩阵和平移矩阵，就可以把三维世界的内角点重新投影到二维相片，使用`cv::projectPoints`计算重投影误差
## 保存标定结果
保存chessboard的内角点数量到`board_width`和`board_height`
保存相片的分辨率到`image_width`和`image_height`
保存chessboard每个方格的尺寸到`square_size`
保存相机传感器每个像素的宽高比到`aspectRatio`
保存计算的flag到`flags`
保存相机的$3*3$矩阵到`camera_matrix`
保存distortion的$5*1$矩阵到`distortion_coefficients`
保存所有图片的重投影误差之平均值到`avg_reprojection_error`
保存每张图片的重投影误差到`per_view_reprojection_errors`
保存拍摄每张图片时的外参即旋转平移到`extrinsic_parameters`，每个图片一行，三个旋转，三个平移，三个旋转是rodrigues即轴角，由相机系到标定板系的变换，先T再R
保存重新计算的角点三维坐标到`grid_points`，坐标系原点在左上角点
保存每张图片的角点二维坐标到`image_points`



如果计算出来的cx和cy差很多，或者fx和fy差很多，可能是使用了非方形的chessboard，且方向反了
RO版本的是带有release object的，如果标的物尺寸不准确，会提高标定精度。但是每幅图必须全部看见内角点，标定物需为刚体。参数ifixedPoint表示可以提供右上内角点准确世界坐标

calibrationMatrixValues()可以从相机内参矩阵计算fov，focalLength，主点坐标等参数

cv::findChessboardCornersSB

相机的坐标系是x向右y向下z从光心指向世界
照片的像素坐标系是左上为0，水平向右为x，垂直向下为y
角点三维坐标，左上角点为$(0,0,0)$，右上角点为$(w,0,0)$，z轴垂直标定板指向标定板背面。