# Web Map Tile Service Implementation Standard
## introduction
WMS侧重点是服务器按照客户端的需求渲染并提供图像，这样服务器压力很大
WTMS解决策略是预先把图像切片，客户端只能根据可视范围要求服务器当前有的图片，自己回去渲染合成
WTMS靠getCapabilityes来告诉客户我有什么，这个内容是标准化的，所有的服务器客户端都可以按照统一标准提高互操作性。
WMTS（Web Map Tile Service，Web 地图瓦片服务）标准的两个阶段。第一阶段是抽象规范，描述了服务器提供的资源和客户端请求的语义，包括服务元数据文档、瓦片图像或表示以及可选的特征信息文档。第二阶段定义了客户端和服务器之间的具体的交换机制。机制适用于不同的架构风格，包括基于过程的架构风格和基于资源的架构风格。基于过程的架构风格使用多种不同的消息编码，如键值对（KVP）、XML 消息或嵌入在 SOAP 信封中的 XML 消息。基于资源的架构风格则允许客户端通过简单的 URL 端点请求服务元数据、瓦片和特征信息资源。
基于资源的架构风格便于部署，适用多种规模，架设WMTS时非常易用，只要提前准备好切片，无需考虑实现实时渲染。尤其是很多ISP不允许使用CGI，ASP。
## 4.8 procedure oriented architectural style
这种设计与平台无关，专注在抽象层面定义操作，参数，结果。例如派生出使用KVP或SOAP通信。
## 4.9 resource oriented architectural style
这种设计与平台无关，专注于在抽象层面定义资源，资源的展现形式，交互操作。
## 4.11 tile
一块矩形图形，表示地理信息。
## 4.12 tile matrix
很多tile按行列组成矩阵，显示更大的区域
## 4.13 tile matrix set
瓦片矩阵的集合，每个瓦片矩阵可以在各自的LOD上显示
## 6 WMTS overview
把图片分为tile，把图片发给客户，以快速响应客户的显示需求。
服务器可以响应两种需求，基于resource oriented architectural style的客户端发来的资源需求，基于procedural oriented architectural style的客户端发来的操作需求。
例如为基于procedural oriented architectural style的客户端提供service metada，tile，featureInfo
好像所有的OGC web services都会提供getCapabalities接口，WTMS继承于OGC web services，并独有getTile和getFeatureInfo
WMTS一次只能提供一个layer的一个tile，不能像WMS把多个layer的tile合成后提供给客户
### 6.1 matrix set
在一个tile map layer，地理空间的表现形式被限制为一系列参数。每个matrix set包含一个或多个tile matrix，这些定义了crs可用的tile
每个tile matrix包含了如下参数
1. 瓦片的比例，以比例分母的形式，相对于标准化渲染像素尺寸的比例。一个标准化渲染像素尺寸为`0.28mm*0.28mm`。好像说的是比例尺
2. 每个瓦片的宽和高，单位像素
3. 左上角点的tile matrix的包围盒的坐标。左上角点是最大的y，最小的x
4. 每个tile matrix的宽和高，以tile为单位，例如tile的数量
### 6.2 well-known scale set
WMTS服务器只能以有限的坐标系和比例来提供数据，因为数据是预先提供的，有些客户端不能自行坐标转换或缩放。如果客户端先接了一个服务器，又接了一个服务器，这些服务器必须按照预定的一些规则提供数据，比如相同的坐标系，一系列预定的比例。
## 7 WMTS实现模型
描述了客户端到底能从服务器获得什么。
### 7.1 service元数据
即在procedure oriented architectural style时，客户端向服务器发送getCapabilities时，服务器的回复。
### 7.2 tile
## 8 使用HTTP的KVP编码
客户端可以把要查询的信息写成KVP，用Get或Post进行打包，通过HTTP发送到服务器。