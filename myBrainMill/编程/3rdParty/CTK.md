编译
https://github.com/commontk/CTK
下载了2023.7.13的源码，
把某一行修改为如下
`ctk_enable_option(PluginFramework "Enable Plugin Framework" ON CTK_LIB_PluginFramework)`
拖入cmake编译
CTM的group里面只选中了5个
CTK_BUILD_SHARED_LIBS
CTK_ENABLE_PluginFramework
CTK_LIB_CORE
CTK_LIB_PluginFramework
CTK_SuperBuild
然后qt选择5，不要qttest和test
这样就生成了一个sln，只有ctk这个重要项目。它会生成CTKCore和CTKPluginFramework两个库
第二种编译
下载了2023.7.13的源码，
把某一行修改为如下
`ctk_enable_option(PluginFramework "Enable Plugin Framework" ON CTK_LIB_PluginFramework)`
拖入cmake编译
CTM的group里面只选中了11个
CTK_BUILD_SHARED_LIBS
CTK_ENABLE_PluginFramework
CTK_LIB_Core
CTK_LIB_Widgets
CTK_PLUGIN_org.commontk.configadmin
CTK_PLUGIN_org.commontk.eventadmin
CTK_PLUGIN_org.commontk.log
CTK_PLUGIN_org.commontk.metatype
CTK_PLUGIN_org.commontk.plugingenerator.core
CTK_PLUGIN_org.commontk.plugingenerator.ui
CTK_SuperBuild
然后qt选择5，一定不要qttest和test，上面某个与qtest和test会让编译时下载ctk_Data
这样就生成了一个sln，只有ctk这个重要项目。它会生成CTKCore,CTKPluginFramework,CTKWidgets.dll,liborg_commontk_configadmin.dll,liborg_commontk_eventadmin.dll,liborg_commontk_log.dll,liborg_commontk_metatype.dll,liborg_commontk_plugingenerator_core.dll,liborg_commontk_plugingenerator_ui.dll

[octo/CTK-examples (gitee.com)](https://gitee.com/iotmedical/CTK-examples#https://gitee.com/link?target=http%3A%2F%2Fgitbook.cn%2Fgitchat%2Fcolumn%2F5ad02029f8164454a34a089b)

[CTK框架(一): CTK Plugin Framework简介-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/140423993)
插件层就是符合接口的dll
服务层实现插件间通信
生命周期层实现插件的安装启动停止卸载

插件由Activator启动，Activator可以获得框架的context
插件就是dll，包含资源文件和元数据。元数据位于manifest.mt

服务层，让其他插件发现本插件上线了。本插件告诉其他插件我上线了。

生命周期层
	ctkPlugActivator定义插件的启动和停止，每个插件必须实现这个接口
	ctkPlugContext定义插件与框架的交互，
	ctkPlug表示框架中已经安装的插件

ctk基于qt plug system和qt service framework，增加了元数据，
[CTK框架(二): 接口、插件和服务_ctk插件-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/140451712)

接口interface，一般是纯虚类，声明了服务的功能
插件plugin，继承于接口，具体实现了接口
服务service，根据接口实例化的对象

接口interface的`Q_DECLARE_INTERFACE`定义的字符串就是调用`ctkPluginContext::registerService`时的形参`clazz`，ctkPlugContext内部就是用这个clazz来区分接口类的



调用图
初始化
ctkPluginFramework::init
	ctkPlugins::load()从目录中加载dll，填充容器
安装插件
	installPlugin返回ctkPlugin类对象某个插件
		ctkPluginFramework::plugins按照dll名字找到插件
启动插件
ctkPlugin::start
	ctkPluginPrivate::finalizeActivation
		ctkPluginPrivate::start0
			加载dll，判断dll里面有函数qt_plugin_instance？
			pluginActivator->start
				自定义的PluginActivator->start
					new自定义的插件类
						ctkPluginContext::registerService
其实
`ctkPluginFrameworkContext::services`内部有一个容器`QHash<QString, QList<ctkServiceRegistration> > classServices`，调用`ctkPluginContext::registerService`时，就是把接口interface类的`Q_DECLARE_INTERFACE`定义的字符串AAA制成一个元素，添加到容器中

如果插件中有一个派生的ServiceTracker，则调用图
pluginActivator->start
	new派生ServiceTracker类，追踪的插件的字符串为AAA
	ctkPluginContext::getServiceReferences AAA
		从容器classServices中找到AAA对应的元素
	派生ServiceTracker类open
		派生ServiceTracker类的addingService，
			基类ctkServiceTracker的addingService
				最终返回context中对应字符串AAA的插件对象
	然后再new自定义的插件类
	registerService(自定义插件类的实例)



到这里，例子01-HelloWorldPlug的dll和02-EmbedCTK都能运行了，注意MF里的Plugin-Name随便取，不一定是dll名字。qrc里的prefix指的是`MANIFEST.MF文件的路径+/META-INF`。

一个接口可以由多个插件实现，
一个插件里面可以有多个接口，即一个Impl类里面有多个Service的接口函数
一个插件可以注册多个服务，即一个Activator类里面要调用registerService把Impl类的每个接口都要注册
一个dll对应一个Minifest文件

https://blog.csdn.net/haokan123456789/article/details/142025345
获得上下文context的两种办法
一
使用ctkPluginFrameworkFactory获得ctkPluginFramework
ctkPluginFramework调用init和start后
使用ctkPluginFramework的getPluginContext
二
调用ctkPluginFrameworkLauncher的start
调用ctkPluginFrameworkLauncher的getPluginContext

https://blog.csdn.net/haokan123456789/article/details/142030276
插件的编写
dll工程和exe工程
dll工程包括接口，Impl类，Activator类，minifest.文件
exe工程使用dll，获得context后查询其中的service，获得ctkServiceReference之后调用该服务的接口函数

[CTK框架(五):插件的配置文件MANIFEST.MF_ckt 添加 manifest.mf-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/142031137)
minifest.mf文件的内容，constants可以在ctkPluginConstants.h中找到定义

[CTK框架(六):服务工厂_ctk服务工厂-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/142067919)
在ctkPluginActivator的start时registerService。
使用ServiceFactory，在需要(是不是别人getService)的时候再实例化impl的对象，有点像懒汉模式，起到延迟初始化作用，
创建一个ctkServiceFactory，

[CTK框架(七):事件监听_ctk csdn-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/142069585)
监听框架的结束时间
监听插件的安装，启动，结束，卸载
监听服务的注册，修改，注销
使用ctkPluginContext类的函数connectPluginListener，connectFrameworkListener，connectServiceListener在自定义的槽函数中响应事件
使用ctkPluginContext创建一个监听者来实现监听

https://blog.csdn.net/haokan123456789/article/details/142110625
ServiceTracker可以在FrameWork中提供追踪和管理插件提供的服务，一个插件registerService后，其他插件可以通过serviceTracker的`getService`来发现使用这个service。

[CTK框架(九):插件间通信_ctk通信框架-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/142127553)
通过eventadmin来进行通信，这是发布订阅的方式，
过程
接收者想要某信息
发送者发送信息
接收者接收了信息并响应

event由两部分属性组成
- 主题topic，定义了事件大类别，用斜杠写成，具有层级
- property，描述了事件的数据

发送者可以是个类
类同步发信息的过程就是`ctkSnippetReportManager.h`的函数`reportGenerated`，
1. 获得`getServiceReference<ctkEventAdmin>`
2. 获得`ctkEventAdmin* eA =getService<ctkEventAdmin>`
3. 把属性封装在结构`ctkDictionary props`中`props["title"] = "aa";`
4. 构造一个ctkEvent, `ctkEvent myE("com/e/rtr/ED", props)`
5. 发送消息，`eA->sendEvent(myE)`
文档中说还可以异步发消息`postEvent`，而且即使同步发消息，事件的处理也是在另一个线程中进行的，所以发消息的线程最好不要使用锁。因此最好使用异步消息。


接收者可以是一个派生了ctkEventHandler接口的类，该类只需实现消息回调函数`void handleEvent(const ctkEvent& inEvent)`
接收者首先找机会声明自己订阅了什么topic的消息
1. 先封装一个具有topic的属性对象
```
ctkDictionary props;
props[ctkEventConstants::EVENT_TOPIC] = "com/e/rtr/ED";
```
2. 调用`ctkPluginContext->registerService<ctkEventHandler>(接收者, props)`来声明接收者需要某个topic的消息，
3. 然后接收者要重写回调函数`handleEvent`，在这个函数中解析消息，消息就是形参
然后接收者等着frameworkContext传递消息即可

发消息好像不一定需要是个独立的接口类，可以在某个自定义的接口类中定义一个发消息函数即可

例子
`CTK-2023.07.13\Libs\PluginFramework\Documentation\Snippets\EventAdmin-Intro`
第二种方法是使用信号槽，文档说性能不如eventAdmin
发送者的工作流程
1. 要有个类，声明一个signal，注册到eventAdmin，
    即`eA->publishSignal(this, SIGNAL(我的信号函数(ctkDictionary)),"com/e/rtr/ED", Qt::DirectConnection);`
    具体在`ctkSnippetReportManager.h`的函数`ReportManager`
2. 封装数据，发送这个信号
    即emit 我的信号函数(ctkDictionary属性)
    具体在`ctkSnippetReportManager.h`的函数`reportGenerated`

接收者的工作流程
1. 写一个派生自QObject的类，定义一个槽函数`void handleEvent(const ctkEvent& event)`，在这个槽函数中解析数据即可
2. 在eventAdmin中调用subscribeSlot订阅说需要这个topic的消息



[CTK框架(十):PluginAdmin插件_ctk插件卸载-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/142186420)
没找到pluginAdmin这个类
[CTK框架(十一):使用的常见问题-CSDN博客](https://blog.csdn.net/haokan123456789/article/details/142704519)