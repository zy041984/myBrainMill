# 2.7版本文档
## 使用earth文件
earth文件本质就是xml，描述了场景的属性，主要包括以下几类
    投影的种类，地心或平面
    影像，高程，矢量，模型的数据源
    缓存位置
同一类数据源，排列顺序很重要，最好按照从低清晰度到高清晰度排列。数据源使用插件机制来进行读写。每种数据源下面有更详细的选项。
为了提高性能，影像高程等数据源解析后会提前制成tile，制作步骤包括下载，重投影，裁减，模糊，合并等，并把处理结果保存在缓存文件夹中。缓存的种类可以是`filesystem`或`leveldb`。
缓存内的文件可以指定生命周期，过期后再重新生成。
每个earth文件可以为Map指定一种缓存策略，每个layer还可以定义自己的缓存策略，来覆盖map的缓存策略。通过环境变量指定的缓存策略再高级一些。最高级的是为每种driver设置自己的缓存策略。
## feature和symbology
feature是矢量数据，可以用矢量形式或光栅化形式来渲染。矢量形式就是ModelLayer，光栅化形式就是ImageLayer
agglite可以把矢量进行光栅化
```
<model name="my layer" driver="agglite">
    <features name="states" driver="ogr">
        <url>states.shp</url>
    </features>
```
feature_geom则会按照矢量形式显示数据
```
<model name="my layer" driver="feature_geom">
    <features name="states" driver="ogr">
        <url>states.shp</url>
    </features>
    <styles>
        <style type="text/css">
            states {
                stroke:       #ffff00;
                stroke-width: 2.0;
            }
        </style>
    </styles>
</model>
```
这段代码也表示出，要显示一个矢量数据，必须用feature来描述数据源，用style来描述绘制形式，这个stylesheet也就是symbology。和web界常用的css类似，内容和形式实现了分离。
形式比如说，clamp表示是否要把矢量图形贴地;extrude表示矢量图形是否要拉伸像一堵墙，extrude-height表示墙的高度，extrude-flatten表示拉伸后要不要平齐;fill表示图形的填充颜色;text表示需要显示的文本;substitution表示是否要把几何图形用外部model来代替;rendering可以更详细地指定光照，混合，深度测试。
每个featurelayer都需要一个stylesheet，一个stylesheet里面可以有多个style，
### 地形跟随的几种方法
map-clamping，即从高程文件中采样得出高度信息，再应用到矢量的顶点上。渲染质量很高，但速度慢。依赖底层高程文件的解析率。采样方法可以选择逐点采样或中心采样。
```
altitude-clamping:   terrain;
altitude-technique:  map;    
altitude-resolution: 0.005;  
```
draping平铺方法，就像把地毯盖在不平的地面上。也就是把矢量图形直接覆盖在地形上，具体实现是把矢量图形绘制到一个纹理，再把这个纹理覆盖在地形的图像纹理上。无需担心解析精度，如果有透明度时可能会不正常。多边形的表现比线的表现要好。边缘会有jagging
```
altitude-clamping:   terrain;
altitude-technique:  drape;  
```
gpu-clamping，即在vs中使用深度场采样法把每个顶点定位到地形上，再在fs中使用depth-offset算法进行改善zfighting。线的处理很好。边缘没有jagging
```
altitude-clamping:   terrain;
altitude-technique:  gpu;  
```
### 大矢量数据集
使用分片和分页技术来解决大量的矢量数据
feature display layout选项可以开启分页和分瓦块
```
<layout>
       <tile_size>250000</tile_size>
       <level name="data" max_range="100000"/>
</layout>
```
layout表示开启分页，在后台进行矢量数据的加载和编译tile_size指每个瓦块的尺寸，单位米。
level表示开启瓦块和lod，`max_range`指的是在这个视点距离时使用这个level,可以有一个或多个level
一般level是事先指定好的，而tile_size则需要根据几何图形的疏密手动调整，没有经验公式来计算。原则就是让openGL能实现按patch把同种类的图元发送到显卡。
## shader
使用模块化方法把一个GLSL的shader分为几部分，即shader composition
shader composition的作用
    自动提供main函数，自己只需要写子函数，并告知何时调用子函数
    
## 优化
如果想要显示地理数据，首先使用gdal驱动来加载，如果太慢，可以按以下方式优化
1 本地数据
1.1 重投影
如果源数据未指定坐标系，osgEarth会在加载的时候重投影数据。可以使用gdal，global mapper或arcgis来重投影数据。
`gdalwarp -t_srs epsg:4326 my_utm_image.tif my_gd_image.tif`
1.2 建立瓦片tile
geotiff通常使用扫描线数据结构，但是osgEarth使用瓦片tile形式加载数据。可以使用gdal来转换
`gdal_translate -of GTiff -co "TILED=YES" myfile.tif myfile_tiled.tif`
1.3 建立overview
可以建立金字塔式结构，进一步提升表现
`gdaladdo -r average myimage.tif 2 4 8 16`
2 网络数据
2.1 建立瓦片集合
把数据结构建立为quad-tree的离散瓦片，这样osgEarth能快速加载。事实上，读取一个geotiff时，osgEarth要在内部建立一个tiles集合。所以，事先建立好tile集合比较好。
把数据打包为`osgearth_package`格式，
把earth文件里的内容打成一个`osgearth_package`包
`osgearth_package file.earth --tms --out out_folder`
这个命令会把文件里的内容生成一个tms仓库。

# FAQ
## 1.怎么知道下载的geotiff有没有坐标系，是扫描线格式还是瓦片格式
回答1.用osgEarth的gdalDriver试一试，没有坐标系的tif不能在球几何体上显示。blueMarble系列的Explorer Base Map、bathymetry有坐标系，old blue marble2没有坐标系，june系列没有tiff文件，更没有坐标系
2.用global mapper打开tif文件，有坐标系的可以直接显示图片，没有坐标系的会弹出对话框要求设置坐标系。高度数据的单位也可以自行设置

# 快速开始
1.无earth文件简单测试读取geotiff文件，
使用osgEarth提供的world.tif文件，执行以下代码
```
#include <string>
#include <iostream>
#include <osg/Node>
#include <osgViewer/Viewer>
#include <osgEarth/Map>
#include <osgEarth/MapNode>
#include <osgEarthDrivers/gdal/GDALOptions>
int main()
{
	osg::ref_ptr<osgEarth::Map> map = new osgEarth::Map();
	osg::ref_ptr<osgEarth::MapNode> mapNode = new osgEarth::MapNode(map);
	osgEarth::Drivers::GDALOptions gdal;
	gdal.url() = "C:\\installFiles\\SrcToBld\\OSGEARTH\\data\\world.tif";
	osg::ref_ptr<osgEarth::ImageLayer> layer = new osgEarth::ImageLayer("BlueMarble", gdal);
	map->addImageLayer(layer);
	osgViewer::Viewer viewer;
	viewer.setSceneData(mapNode);
	//osg::ref_ptr< osgEarth::Util::EarthManipulator> mainManipulator = new osgEarth::Util::EarthManipulator;
	//viewer.setCameraManipulator(mainManipulator);
	viewer.setUpViewInWindow(100, 100, 800, 600);
	return viewer.run();
	std::cout << "Hello World!\n";
}
```
生成了一个球形几何体，表面覆盖了纹理图片，能够使用osg的manipulator。

下载了srtm30plus_stripped.tif，运行同样的代码，也是生成了一个球形几何体，表面覆盖了纹理图片，能够使用osg的manipulator。

2.使用.earth文件简单测试读取geotiff文件，
这是一个有高度数据的图片，如何生成3d地形呢，使用elevation Layer即可
要在earth文件中加入vertical_scale，同时程序里也要处理vertical_scale
代码
```
#include <iostream>
#include <osg/Node>
#include <osgViewer/Viewer>
#include <osgEarth/Config>
#include <osgEarth/Map>
#include <osgEarth/MapNode>
#include <osgEarthDrivers/gdal/GDALOptions>
#include <osgEarthUtil/EarthManipulator>
#include <osgEarthUtil/VerticalScale>
int main()
{
	std::cout << "Hello World!\n";
	std::string earthFile = "E:\\work\\osgPractice\\02-testEarthFile\\simple-elevationLayer.earth";
	osg::Node* node = osgDB::readNodeFile(earthFile);
	osg::ref_ptr<osgEarth::MapNode> mapNode;
	if(node!=nullptr)
		mapNode = osgEarth::MapNode::get(node);
	if (mapNode.valid())
	{
		const osgEarth::Config& externals = mapNode->externalConfig();
		const osgEarth::Config& vertScaleConf = externals.child("vertical_scale");
		// Install vertical scaler
		if (!vertScaleConf.empty())
		{
			mapNode->getTerrainEngine()->addEffect(new osgEarth::Util::VerticalScale(vertScaleConf));
		}
	}
	osgViewer::Viewer viewer;
	if (node)
	{
		viewer.setSceneData(node);
		viewer.getCamera()->setSmallFeatureCullingPixelSize(-1.0f);
		return viewer.run();
	}
}
```
earth文件
```
<map name="MyMap" type = "geocentric" version="2">
    <image name="bluemarble" driver="gdal">
            <url>E:\data\blueMarble\Explorer Base Map\eo_base_2020_clean_geo.tif</url>
    </image>
    <elevation name="dem" driver="gdal">
        <url>E:\data\dem\30\srtm30plus_stripped.tif</url>
    </elevation>
    <external>
	    <vertical_scale scale="30.0"/>
	</external>
</map>
```
image这个layer就是栅格图像，elevation这个layer就是高程数据，external是一些外部设置
为了明显地看到地形起伏，最好使用高度缩放倍数，代码和earth文件都要处理。全球的数据10倍效果都不好，30倍效果才凑合使用

# 用gdal进行文件转化
接下来就是优化，tif数据格式为扫描线，用gdal转成tile形式
`.\gdal_translate.exe -of GTiff -co "TILED=YES" "E:\data\dem\30\srtm30plus_stripped.tif" "E:\data\dem\30\srtm30plus_stripped_tiled.tif"`
`.\gdal_translate.exe -of GTiff -co "TILED=YES" "E:\data\blueMarble\Explorer Base Map\eo_base_2020_clean_geo.tif" "E:\data\blueMarble\Explorer Base Map\eo_base_2020_clean_geo_tiled.tif"`
把无坐标信息的png图片转为有坐标信息的tif
`set GDAL_DATA=C:\installFiles\SrcToBld\3rdParty\data`
`cd C:\installFiles\SrcToBld\3rdParty\bin`
`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 -180 90 -gcp 21600 21600 -90 0 -gcp 21600 0 -90 90 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.A1.png" "E:\data\blueMarble\JuneRecify\A1_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 -180 0 -gcp 21600 21600 -90 -90 -gcp 21600 0 -90 0 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.A2.png" "E:\data\blueMarble\JuneRecify\A2_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 -90 90 -gcp 21600 21600 0 0 -gcp 21600 0 0 90 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.B1.png" "E:\data\blueMarble\JuneRecify\B1_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 -90 0 -gcp 21600 21600 0 -90 -gcp 21600 0 0 0 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.B2.png" "E:\data\blueMarble\JuneRecify\B2_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 0 0 -gcp 21600 21600 90 -90 -gcp 21600 0 90 0 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.C2.png" "E:\data\blueMarble\JuneRecify\C2_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 0 90 -gcp 21600 21600 90 0 -gcp 21600 0 90 90 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.C1.png" "E:\data\blueMarble\JuneRecify\C1_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 90 0 -gcp 21600 21600 180 -90 -gcp 21600 0 180 0 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.D2.png" "E:\data\blueMarble\JuneRecify\D2_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 90 90 -gcp 21600 21600 180 0 -gcp 21600 0 180 90 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x21600.D1.png" "E:\data\blueMarble\JuneRecify\D1_tiled_LZW.tif"`

`.\gdal_translate.exe -of GTiff -a_srs "EPSG:4326" -co "PROFILE=GeoTIFF" -co "TILED=YES" -co "COMPRESS=LZW" -gcp 0 0 -180 90 -gcp 21600 10800 180 -90 -gcp 21600 0 180 90 "E:\data\blueMarble\June\world.topo.bathy.200406.3x21600x10800.png" "E:\data\blueMarble\JuneRecify\All_tiled_LZW.tif"`

其中 gcp表示控制点，即png图片的角点对应的经纬度
-co "PROFILE=GeoTIFF"表示输出文件的tag只有GeoTIFF相关的tag
-co "COMPRESS=LZW"表示使用LZW压缩
-a_srs EPSG:4326表示设置坐标系，EPSG:4326即WGS84

观察发现，转化之后的tif就可用GIS打开了。
但是用windows自带的照片查看器打开685Mb的eo_base_2020_clean_geo_tiled.tif非常快，打开685Mb未经LZW压缩的All_tiled.tif就慢一些，打开200Mb经LZW压缩的All_tiled_LZW.tif基本上就打不开
用gdalinfo.exe对比两个tif文件，eo_base_2020_clean_geo_tiled.tif多了几个属性
```
Origin = (-180.000000000000000,90.000000000000000)
Pixel Size = (0.016666666666667,-0.016666666666667)
TIFFTAG_RESOLUTIONUNIT=2 (pixels/inch)
TIFFTAG_SOFTWARE=Adobe Photoshop 21.1 (Macintosh)
TIFFTAG_XRESOLUTION=72
TIFFTAG_YRESOLUTION=72
```
不知道是不是这个原因


earth文件变为
```
<map name="MyMap" type = "geocentric" version="2">
    <image name="bluemarble" driver="gdal">
            <url>E:\data\blueMarble\Explorer Base Map\eo_base_2020_clean_geo_tiled.tif</url>
    </image>
    <elevation name="dem" driver="gdal">
        <url>E:\data\dem\30\srtm30plus_stripped_tiled.tif</url>
    </elevation>
    <external>
	    <vertical_scale scale="30.0"/>
	</external>
</map>
```
感觉效果一般吧，好像没提升多少表现
# 地心地固坐标系ECEF与osg场景坐标系
ECEF坐标系也叫地心地固直角坐标系，一个点的坐标为xyz
原点：地球的质心，
x轴：原点延伸通过本初子午线（0度经度）和赤道（0维度）的交点。
z轴：原点延伸通过的北极，也就是理想地球旋转轴。

WGS84坐标系的轴向与ECEF的轴向相同，但是WGS84坐标系给出的坐标为\[北纬,东经,海拔\]

把osg::EllipsoidModel的format设置为WGS84时
WGS84坐标对应osg场景坐标系xyz
(0,0,0)对应场景坐标系的(r,0,0)
(0,90,0)对应场景坐标系的(0,r,0)
(0,180,0)对应场景坐标系的(-r,0,0)

就是说地球的北为osg的z轴，0度经线位于osg的+x轴。
osg场景坐标系与ECEF坐标系各轴方向相同

地球上每个点都有个局部坐标系，局部x轴为该点的东方向，局部y轴为该点的北方向，局部z轴为该点的天方向
`osg::EllipsoidModel::computeCoordinateFrame`就是计算osg场景坐标系到每个经纬度点的局部坐标系的变换矩阵，再转置。
`osg::EllipsoidModel::computeLocalToWorldTransformFromLatLongHeight`计算的也是这个坐标系。

# osg::Matrixd::makeLookAt
使用osg场景坐标系的eye，center和up
构造了GLCamera到osg场景坐标系的变换矩阵
数学上的$M\mathop{{}}\nolimits_{{GLCamera}}^{{OSG}}$
# osg::Camera::getViewMatrix
返回的数学矩阵是GLCamera到osg场景坐标系的变换矩阵，即数学上的$M\mathop{{}}\nolimits_{{GLCamera}}^{{OSG}}$，osg存储为转置
# osg::Camera::getProjectionMatrix
返回的数学矩阵是GLNDC到GLCamera坐标系的变换矩阵，即数学上的$M\mathop{{}}\nolimits_{{GLNDC}}^{{GLCamera}}$，osg存储为转置
# osg::Viewport::computeWindowMatrix
返回的数学矩阵是窗口到GLNDC坐标系的变换矩阵，即数学上的$M\mathop{{}}\nolimits_{{window}}^{{GLNDC}}$，osg存储为转置
OSG视口左下角为原点
# osgGA::OrbitManipulator::getMatrix
返回的数学矩阵是osg场景坐标系到GLCamera的变换矩阵，即数学上的$M\mathop{{}}\nolimits_{{OSG}}^{{GLCamera}}$，osg存储为转置
# 设置场景不要自动计算近远裁剪面
```
struct myCPM :public osg::CullSettings::ClampProjectionMatrixCallback
{
	virtual bool clampProjectionMatrixImplementation(osg::Matrixf& projection, double& znear, double& zfar) const { return true; }
	virtual bool clampProjectionMatrixImplementation(osg::Matrixd& projection, double& znear, double& zfar)const { return true; }
};
viewer->getCamera()->setClampProjectionMatrixCallback(new myCPM);
```
# osgEarth::EarthManipulator的矩阵变换过程：
1. 平移到_center
2. 旋转_centerRotation,应该是把OSG场景转到本地ENU
3. 旋转_rotation,应该是把本地ENU坐标系转到GLCam，使得纬度与viewpoint要求的pitch相同
4. 平移`(_viewOffset.x,_viewOffset.y,_distance)`
# osgEarth::EarthManipulator
`_homeViewpoint`的pitch好像是GL坐标系下的角度，不确定
`_srs->isGeographic()`返回true
`_center`是焦点在OSG场景坐标系的位置
`_centerMap`是焦点在地球坐标系的位置LLH
`_centerRotation`是OSG场景系到_center的ENU的变换四元数
`_rotation`是本地ENU到GLCamera的旋转矩阵，应该是反映了setViewport的形参中head和pitch，还没搞懂head和pitch是怎么算出来的，只是觉得不准确，最好使用_rotation，而不要单独设置head或pitch，head就是绕本地ENU的up轴旋转，正head值是绕右手系正向旋转，pitch就是绕本地ENU的e轴旋转，但是好像是按左手系正向

`_centerLocalToWorld`是场景系到_center的ENU的变换矩阵
`setByMatrix`能在OnKeyHandle中改变视点，顺序是先平移到_center即LLH指定的osg世界点，再旋转_centerRotation，即经纬度以及绕x转90，再旋转_rotation,即heading和pitch，再平移xy，再沿z轴平移_dist。
`setByMatrix`用的是是场景坐标系到GLCameraosg的变换矩阵
即数学上的$M\mathop{{}}\nolimits_{{OSG}}^{{GLCamera}}$

可以通过getViewpoint知道`_center`，再根据computeCenterRotation的方法反算出`_centerRotation`，通过getRotation获得`_rotation`
`computeCenterRotation`好像专门计算`_centerRotation`
`getRotation`返回`_rotation`，只与当前viewpoint的head和pitch有关
`getViewpoint`返回的是`_center`的LLH坐标，`range`表示视点到焦点的距离`_distance`，

`setViewpoint`能在OnKeyHandle中改变焦点，而且无论鼠标怎么左键旋转右键缩放，getViewpoint返回的head和pitch与最开始的形参一致。例如果形参最初设置的head为0，pitch为-90度，则getViewPoint返回的head为0，pitch为-90。
如果setViewpoint的形参设置了head和pitch为其他的数值，则视点姿态被改变，但是无论鼠标怎么左键旋转右键缩放，这两个角度不变。
但是鼠标中键拖动，是会改变getViewpoint的head和pitch的
这是因为鼠标左键改变的是经纬度，缩放改变的是_dist，所以不影响焦点，焦点指的是地球表面上被看的那一点

左键是pan，中键是rotate，右键是zoom
中键rotate修改了_rotation
左键pan修改了_center和_centerRotation
右键zoom主要修改了`_distance`，`_center`和`_centerRotation`稍有改变

# 相机矩阵
主camera加slave与Manipulator的结论

主`camera::getViewMatrix`返回的矩阵数学上是GLCamera到osg场景坐标系的变换矩阵$M\mathop{{}}\nolimits_{{GLCamera}}^{{OSG}}$

`EarthManipulator::getMatrix`或`OrbitManipulator::getMatrix`返回的矩阵数学上是osg场景坐标系到GLCamera的变换矩阵$M\mathop{{}}\nolimits_{{OSG}}^{{GLCamera}}$

代码上`主camera::getViewMatrix*Slave::_viewOffset`等于`Slave::_camera::getViewMatrix`
数学上`Slave::_viewOffset*主camera::getViewMatrix=Slave::_camera::getViewMatrix`
即数学上`Slave::_viewOffset`的意义是$M\mathop{{}}\nolimits_{{slave相机GL}}^{{主相机GL}}$

代码上`主camera::getProjectionMatrix*Slave::_projectionOffset`等于`Slave::_camera::getProjectionMatrix`
数学上`Slave::_projectionOffset*主camera::getProjectionMatrix=Slave::_camera::getProjectionMatrix`
即数学上`Slave::_projectionOffset`的意义是$projM\mathop{{}}\nolimits_{{slave相机}}*projM\mathop{{}}\nolimits_{{主相机}}^{{-1}}$

# osgEarth::GeoPoint
绝对高度指的是平均海平面MSL为基准的高度

# osgEarth::Viewpoing
range指的是eye与center的距离，会被赋给EarthManipulator的_distance