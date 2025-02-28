# UE4中使用GIS
## 1 GIS插件
总体来说可以使用很多插件，来自超图、arc、Map3d
## 1.1 超图
超图有单独的插件，从UE4商城中搜索Supermap即可安装到引擎
新建项目，项目启动后，可在**编辑>插件>已安装**中找到Supermap插件，但**插件默认是没有启动的**，我们**在启动复选框已启用上勾选，点击立即重启**；稍等片刻，重启以后关闭插件窗口 ；插件第一次启动需要在视图选项中勾选**显示引擎内容和显示插件内容；** 然后在内容选项卡下就可以找到Supermap的相关内容了，双击这个文件夹，我们打开本次教程相关的内容；Map文件夹下，超图的范例自带了几个在线服务：
**1 BIM模型**
**2 地形影像数据**
**3 3D Max精模**
**4 倾斜摄影模型**
**5 点云数据**
**6 全球地形影像数据**
**7 城市白膜**

超图ue4 https://blog.csdn.net/qq_45590504/article/details/125820989

SuperMap为UE4和U3D都提供了插件可在这里下载 http://support.supermap.com.cn/product/iObjects.aspx

### 使用方法

主要通过SuperMap Desktop生成缓存，本地直接加载对应的文件。在安装包里有操作的详细说明。
### 常用的功能

1、SuperMap Georeference 组件，可以将 UE 对象移动到指定经纬度、高度的位置。

SuperMap Georeference 组件中经纬度写的是double类型，所以直接在UE4蓝图中没法使用，将其中的Longitude、Latitude、Height改成float类型。

2、飞行定位功能。

3、地理坐标到笛卡尔坐标，迪卡儿坐标求地理坐标。

4、图层材质设置。

5、获取鼠标当前点坐标。
其实这原理很简单，就是已经知道屏幕位置，怎么求这个屏幕位置对应的世界位置和方向。可惜目前SuperMap不支持UE4射线检测，不能直接射线检测在图层上的位置。不然也不会采用这样的方式获取坐标点。

优点：SuperMap Scene SDK 功能多多，更加符合国人操作。SuperMap Desktop缓存特别方便。同时支持球面与平面，以及本地与在线服务。最后结尾我会说一下这三个插件怎么本地开服务。

缺点：自己看源码，教程太少，bug太多。

## 1.2 利用blender插件和shp数据
[UE4智慧城市可视化实例教程-UE4蓝图和材质基础入门-新手向保姆级教学_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV19T4y1P7RS)

## 1.3 Cesium for Unreal

[UE4 SuperMap Scene SDK、ArcGIS Maps SDK、Cesium for Unreal 基本使用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/532672126)

Cesium应该说是更新比其它两款积极。[Github](https://link.zhihu.com/?target=https%3A//github.com/CesiumGS/cesium-unreal)上可以下最新版[https://github.com/CesiumGS/cesium-unreal/](https://link.zhihu.com/?target=https%3A//github.com/CesiumGS/cesium-unreal/)

官网学习链接 [Cesium for Unreal](https://link.zhihu.com/?target=https%3A//cesium.com/learn/unreal/)

cesium 目前来说，教程比较多，用的人也是最多的，常用的功能已经满足使用，cesium坐标系转换都写好了直接暴露给蓝图，不需要你从c++找。

Cesium for Unreal 官网上说目前不支持 KML、SHP、GeoJSON 和 CZML 等矢量数据格式。但是cesiumlab3支持shp面模型通过通用模型切片处理和shp点生成实例模型处理。

在处理shp线的时候可以生成面然后生成体。以这样的方式加载shp文件同时还可以显示shp属性字段。

### 启http服务的小工具

在上面的视频教程里提供了启http服务的小工具。地址 [https://www.npmjs.com/package/serve](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/serve)

不需要cesiumlab服务。下图可快速开启http服务。

## 1.4 ArcGIS Maps SDK

### 下载地址[arcgis maps sdk for game engines](https://link.zhihu.com/?target=https%3A//www.cityengine.cn/product/view480.html)

### 使用教程

官网链接[ArcGIS Maps SDK for Unreal Engine](https://link.zhihu.com/?target=https%3A//developers.arcgis.com/unreal-engine/)

个人感觉ArcGIS是cesium 增强版，多了空间和数据分析，以及图层透明度设置。ArcGIS同样支持在线服务与本地加载，不过ArcMap生成的本地TPK 文件速度比不上超图和cesium。