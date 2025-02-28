# 坐标系
- 世界坐标系 左手 x向屏幕里y右z上 
- 眼坐标系 左手 x向右y向上z向屏幕里
- clip坐标系 左手 clip空间的n映射为0，f映射为1或者反过来n映射为1，f映射为0
# 透视投影矩阵
有两种处理方法，znormal和zreverse，见源文件PerspectiveMatrix.h
- zNormal指的是变换之后n映射到0，f映射到1
    $$zNormal\_fNotINF=\begin{bmatrix} \frac{1}{\tan(FovH/2)} & 0 & 0 & 0 \\ 0 & \frac{1}{\tan(FovV/2)} & 0 & 0 \\ 0 & 0 & \frac{f}{f-n} & \frac{fn}{n-f} \\ 0 & 0 & 1 & 0 \end{bmatrix}$$
    如果f为无穷远，则上矩阵可以进一步简化为
    $$zNormal\_fINF=\begin{bmatrix} \frac{1}{\tan(FovH/2)} & 0 & 0 & 0 \\ 0 & \frac{1}{\tan(FovV/2)} & 0 & 0 \\ 0 & 0 & 1 & -n \\ 0 & 0 & 1 & 0 \end{bmatrix}$$
- zReverse指的是变换之后n映射到1，f映射到0
$$zReverse\_fNotINF=\begin{bmatrix} \frac{1}{\tan(FovH/2)} & 0 & 0 & 0 \\ 0 & \frac{1}{\tan(FovV/2)} & 0 & 0 \\ 0 & 0 & \frac{n}{n-f} & \frac{fn}{f-n} \\ 0 & 0 & 1 & 0 \end{bmatrix}$$
    如果f为无穷远，则上矩阵可以进一步简化为
    $$zReverse\_fINF=\begin{bmatrix} \frac{1}{\tan(FovH/2)} & 0 & 0 & 0 \\ 0 & \frac{1}{\tan(FovV/2)} & 0 & 0 \\ 0 & 0 & 0 & n \\ 0 & 0 & 1 & 0 \end{bmatrix}$$
# 正交投影矩阵
有两种处理方法，znormal和zreverse，见源文件OrthoMatrix.h
- zNormal指的是变换之后n映射到0，f映射到1
    $$zNormal\_fNotINF=\begin{bmatrix} \frac{2}{H} & 0 & 0 & 0 \\ 0 & \frac{2}{V} & 0 & 0 \\ 0 & 0 & \frac{1}{f-n} & \frac{n}{n-f} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
   
- zReverse指的是变换之后n映射到1，f映射到0
$$zReverse\_fNotINF=\begin{bmatrix} \frac{2}{H} & 0 & 0 & 0 \\ 0 & \frac{2}{V} & 0 & 0 \\ 0 & 0 & \frac{1}{n-f} & \frac{f}{f-n} \\ 0 & 0 & 0 & 1 \end{bmatrix}$$
