# 简介
为了解决3d内容制作出来如何快速地通过网络发布到客户端并渲染出来
因此，要包含几何数据，场景数据，纹理数据，还要便于渲染引擎使用
gltf不仅是一种新的格式，而是一种transmission format
场景结构用json来描述，几何数据以渲染引擎便于使用的格式存储，无需提前解压缩。
所以好多原始数据格式可以被转换为gltf，好多场景工具可以生成出gltf格式的场景，好多引擎也支持直接渲染gltf
[glTF Project Explorer (khronos.org)](https://github.khronos.org/glTF-Project-Explorer/)可以用来转换到gltf格式，
# 版本
2.0

# 一些细节
应该用UTF8-NoBOM来书写json文件
整数值可以写为整数，.0，或者科学计数法。浮点数最好使用Grisu2这种序列反序列方法，不会被圆整影响。

每个json文件必须有一个`asset`字段，指明符合的gltf标准版本号，

二进制文件必须是LE

# 坐标系和单位
同GL坐标系，单位米和弧度
# 基本结构

scene graph场景树 ^sceneGraph

node子节点 ^node

mesh即几何数据，包括顶点数组，索引数组，法线数组等 ^mesh

material描述了[[#^mesh|几何数据]]的材质 ^material

所有这些节点用json字段来描述，json可以用数组来表示多个并列的对象。每个json字段可以表示链接到其他json字段。
[[#^sceneGraph|场景树]]由[[#^node|子节点]]构成，例如一个[[#^sceneGraph|场景树]]里有多个[[#^node|子节点]]。某个[[#^node|子节点]]可以通过键值对和索引来指定使用了哪个[[#^mesh|mesh]]，同理某个[[#^mesh|mesh]]可以通过键值对和索引来指定使用了哪个[[#^material|材质]]，[[#^material|材质]]可以指定使用的纹理，纹理可以指定使用的sampler和image，
[[#^node|子节点]]可以有位姿属性，可以指定其他[[#^node|子节点]]形成父子关系，但是必须形成disjoint strict trees不相交严格树，即每个节点只能有0或1个parent，不能形成圈。也可以指定[[#^mesh|mesh]]或camera，表明本节点会形成几何体或本节点与相机有关系。
表示旋转的单位四元数顺序是XYZW，表明的是在本节点坐标系的旋转
位姿的变换顺序为TRS，也可以写为`4*4`矩阵，顺序是00 10 20 30 01 11 21 31……，最后4个数是位移


详细的各种节点可以看参考文档


几何数据和纹理图形等二进制数据多保存在另外的文件中。
