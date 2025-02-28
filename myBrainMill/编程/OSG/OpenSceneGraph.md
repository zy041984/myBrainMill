#单例 #FrameStamp #Timer #Camera #osgView #osgViewerView #osgViewerScene
# 单例
有些类由单例进行实例化
在Object里有一个宏
```
#define OSG_INIT_SINGLETON_PROXY(ProxyName, Func) static struct ProxyName{ ProxyName() { Func; } } s_##ProxyName;
```
即定义一个`static`的结构体，该结构体初始化的时候调用`Func`这个函数，`Func`这个函数声明并初始化一个内部的`static`变量。在库中任意位置多次调用`Func`函数时，均会获得这个内部`static`变量，这就实现了全局单例

如`osgViewer::Scene`类实例化了一个`OSG_INIT_SINGLETON_PROXY`宏，
```
OSG_INIT_SINGLETON_PROXY(SceneSingletonProxy, getSceneSingleton())
```
函数`getSceneSingleton()`声明并初始化了一个`static`内部变量，并返回该变量。
`static SceneSingleton s_sceneSingleton`，而结构体`SceneSingleton`内部维护了一个`osgViewer::Scene`类的容器，使得全局只有这一个容器保存全部`Scene`对象
发生要访问`osgViewer::Scene`对象的情况时，调用`osgViewer::Scene::getScene()`，内部访问`getSceneSingleton`，从而访问这个保存了`osgViewer::Scene`容器的全局结构体`SceneSingleton`

像这样的全局单例还有`DisplaySettings`、`GraphicsContext::WindowingSystemInterfaces`等
`osg::Timer`用传统的instance()函数建立内部临时static变量方式也实现了单例


## 一些内部的关键类
osg::FrameStamp封装了帧序号和时间count。
osg::Timer用QueryPerfomanceCount来获得时间count。

osg::Camera是类似OpenGL观察的相机，有记录Projection和View的Matrix，有Viewport。
内部还关联了osg::GraphicsContext和osg::View、osg::GraphicsOperation。
GraphicsContext与窗口系统相关。
osg::View关联了灯光和时间_frameStamp。
osg::GraphicsOperation关联了cull和draw等显卡相关的操作。
Camera与GraphicsContext像一个网，Camera知道自己关联的GraphicsContext，同时GraphicsContext知道自己关联了哪些Camera。
Camera与osg::View也像一个网，Camera知道自己关联的osg::View

osg::View管理相机，有一个主相机负责用户交互但不绘制，其余相机作为Slave进行绘制。并管理灯光和时间_frameStamp。

osgViewer::View继承于osg::View，负责用户交互的EventQueue和EventHandler，负责关联场景osgViewer::Scene，负责记录每帧的开始时间`Timer_t _startTick`，负责控制相机运动的CameraManipulator

osgViewer::ViewerBase提供了各种函数的接口，记录了线程模式，刷新模式，是否退出flag，是否第一帧flag

`osgViewer::Scene`使用成员变量`_sceneData`记录了`osg::Node`场景节点

同一个osg::View里的slave相机好像统一接收主camera的UI交互，因此应该是osgViewer::View的cameraManipulator可以同时改变这些slave的视角

如何管理多个osgViewer::Viewer，使用osgViewer::CompositeViews，使用容器型成员变量管理多个osgViewer::View，这样应该能实现每个Viewer有各自的事件队列和manipulator


## 操作系统的窗口系统
### Windows
