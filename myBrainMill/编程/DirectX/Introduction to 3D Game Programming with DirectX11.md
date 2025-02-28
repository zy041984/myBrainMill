书摘
# 1
## 1.6 XNA
D3DX11现在的数学库为XNA Math库，在Win平台上使用了SSE2(Streaming SIMD Extension 2)指令集。可以用一个SIMD指令一次运算128bit的四个float运算。
# 2
## 2.1 矩阵定义
数学上m行n列的数字，本书使用的矩阵元素为$M_{ij}$，$i$代表行，$j$代表列
## 2.8 XNA矩阵
点$p_r$用$(1,4)$矩阵表示，变换矩阵$m_r$为$(4,4)$，变换后的点为$p_r*m_r$。
后面用vs的union
```
union{
    float m[4][4];
    float m[16];
    struct
    {
	    float _00, _01, _02, _03;
	    float _10, _11, _12, _13;
	    float _20, _21, _22, _23;
	    float _30, _31, _32, _33;
    };
    vector row[4];
}
```
union中前三个部分元素的对应顺序为```
```
[0]=[0][0],[1]=[0][1],[2]=[0][2],[3]=[0][3],[4]=[1][0],[5]=[1][1],[6]=[1][2]
```
第四部分表示行向量，
# 3 变换
点变换为行向量乘矩阵，即点$p$用$(1,4)$矩阵表示，变换矩阵$m$为$(4,4)$，变换后的点为$p*m$。假设绕z轴旋转某角度，无论坐标系是左手右手，变换矩阵是相同的。
**但是点写成行向量还是列向量会影响这个矩阵**，
# 4 Direct3D初始化
### 4.1.1 Direct3D概述
Direct3D是一个底层图形API。一个显卡必须支持Direct3D11的全部功能集合，以及一些额外功能。
# 5 流水线
流水线指的是把照相机能看到的东西计算成2D图像。
## 5.4 流水线
Input Assembler Stage(IA)
Vertex Shader Stage(VS)
Hull Shader Stage(HS)
Tessellator Stage
Domain Shader Stage(DS)
Geometry Shader Stage(GS)
Stream Output Sgate(SO)
Rasterizer Stage(RS)
Pixel Shader Stage(PS)
Output Merger Stage(OM)
### 5.6.2 view空间
照相机的坐标系为，坐标系位于视点，x轴向右，y轴向上，z轴指向屏幕内侧形成左手坐标系。使用的矩阵为view Matrix
假设世界坐标系到相机坐标系的转换为$R_rT_r$，则viewMatrix指的是$(R_rT_r)^{-1}$，下角标$_r$表示点为行向量。
这个viewMatrix的结果与使用点为列向量的矩阵$(T_cR_c)^{-1}$互为转置。下角标$_c$表示点为列向量。
## 5.6.3 投影和齐次裁剪空间
使用的眼坐标系为左手坐标系
裁剪空间为左手坐标系
NDC为左手坐标系，z的范围为$[0,1]$
其余与OpenGL的推导过程应该一样
"眼坐标\*矩阵第四列"的结果希望保留眼坐标系的z坐标，所以矩阵第四列应该为$0,0,1,0$，
眼坐标\*proj矩阵的结果为clip系坐标，
接下来进行NDC计算从clip坐标1\*4得到NDC的坐标1\*3，z的范围为$[0,1]$，近裁剪面映射到0，远裁剪面映射到1

# 16 点选
## 16.1 screen to projection window变换
把NDC转换为屏幕坐标的矩阵也是4\*4的
backBuffer的左上为坐标0，0
NDC空间的点的坐标为