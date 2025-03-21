# 下载windows binary地址
https://sourceforge.net/projects/geoserver/files/GeoServer/2.26.2
# 环境
2.26.2+win11+edge浏览器
# 依赖
## 不可行的jre8
jre8不可以
根据网页所说
[Installing and using Oracle Java on Windows](https://www.java.com/zh-cn/download/help/java_windows.html)
[What is the offline method for downloading and installing Java for a Windows computer?](https://www.java.com/en/download/help/windows_offline_download.html)
先卸载旧的java，java7update80
再安装新的java，jre-8u441-windows-x64.exe
## 可以的javaDK17
[Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/?er=221886#java17-windows)
安装之后设置环境变量JAVA_HOME,在path里面添加`%JAVA_HOME%\bin`
# 安装
installation type选择了manually
在环境变量中，把`EOSERVER_DATA_DIR`改为了`E:\data\GeoServer`
# 运行
打开目录`C:\Program Files\GeoServer\bin`,运行`startup.bat`，不要关闭命令行窗口。
在浏览器访问`http://localhost:8080/geoserver`,即可访问管理界面，默认用户名为admin和aadd
# overview
OGC的WFS和WCS的参考实现就是由geoserver做出来的
# getting started
## 使用网络管理界面
网址为`http://localhost:8080/geoserver`，可以使用自己的网址和端口
### layer preview
点击open layer可以自动新打开一个浏览器页面预览使用的图层
## 发布一个Image
先准备数据源
1. 假设用yws.tiff,复制到geoserver的数据目录，`E:\data\GeoServer\data\newYWS`
2. 创建workspace，workspace就是把相似的layer放在一起，workspace里还可以有store。在数据->工作空间中点击添加新的工作空间。指定name和命名空间URI。name不能超过十个字符，不能有空格。命名空间URI像个网址，但不一定是可以访问的网址。
3. 创建store，store就是仓库，geoserver到这里找实际的数据。在数据->存储仓库中点击添加新的store。选择类型，例子中用的是既有栅格也有矢量的worldImage。这里yws选择geotiff只有栅格。
    在数据源窗口输入“数据源名称”，这个就是geoserver在硬盘上创建的文件夹，所以不要有空格之类很难处理的字符。在连接参数的URL中，选择yws.tiff的文件位置。点击保存
4. 创建layer，图层就是
    在YWS右侧的操作列中点击发布。
    在编辑图层界面中输入命名，标题，坐标系。其中标题很重要是对外可见的。
    坐标系选项的Force declared意思是强制使用geoserver内部的WGS84而不用yws.tiff定义的prj文件，yws.tiff也没有提供prj文件。
    包围盒 native bounding box可以选择从数据中计算，然后经纬边框可以选择从nativebounds计算
    点击应用，再回到页面顶端，进入发布页面
    在WMS设置部分的默认样式选择raster
    点击保存
5. 预览，点击数据->图层预览，在常用格式列点击OpenLayers，弹出新页面显示图层预览
# geowebcache
geowebCache是一个瓦片服务器，已经被集成在geoserver中，它在mapserver和client之间充当代理，可以提供缓存服务。
## Tile缓存的切片图层
这里是可观测到的图层列表，
每个项目的操作列中，seed就是制作瓦片
# 翻译
[Translating GeoServer — GeoServer 2.27.x Developer Manual](https://docs.geoserver.org/latest/en/developer/translation.html)
# 译文
[GeoServer @ OSGeo Weblate](https://weblate.osgeo.org/projects/geoserver/#components)