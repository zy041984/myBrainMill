- osgQOpenGLWidget
   编译没通过，要把几个头文件加入moc的步骤

- Win32Window的程序的关闭流程

按下标题栏的关闭按钮，先生成WM_CLOSE消息，在基类``GraphicsWindow::_eventQueue``里添加一个closeWindow事件

然后在``osgViewer::Viewer``的eventTraversal里先获得窗口``osg::View::_camera``, 获得``osg::GraphicsContext``，再通过``graphicsWindowWin32``的``checkEvent``里调用``::PeekMessage``把这个消息添加到其消息队列，然后处理这个``osgGA::GUIEventAdapter::CLOSE_WINDOW``事件，进行清空资源等操作，析构GraphicsContext、camera、osgViewer::Viewer等

注意没有生成过WM_DESTROY和WM_QUIT消息，没有响应过``osgViewer::Viewer::eventTraversal``处理这个``osgGA::GUIEventAdapter::QUIT_APPLICATION``

无法判断按esc键的结束流程是否相同

- osgQOpenGLWidget的关闭流程
如果从外部调用``getOsgViewer()→getEventQueue()→quitApplication()``，确实会触发``osgViewer::Viewer::eventTraversal``处理这个``osgGA::GUIEventAdapter::QUIT_APPLICATION``，使得后续执行的``ViewerBase::frame``函数不执行任何内容，也就是无法处理消息循环和绘制。

如果从外部调用``getOsgViewer()→getEventQueue()→closeWindow()``，则没反应，不会触发``osgViewer::Viewer::eventTraversal``处理这个``osgGA::GUIEventAdapter::CLOSE_WINDOW``事件，不会调用``graphicsContext::close``

原因是无论是3.4.0版本的``osgQt::GraphicsWindowQt``还是3.6.5版本的osgQOpenGLWidget没有重载checkEvents，只能调用基类graphicsWindow的虚函数checkEvents，它只是检查消息队列是否为空。所以osgQOpenGLWidget分发事件时，不能把close事件分发到osg的消息队列，所以没办法处理这个closeWindow消息，只能通过析构时释放Viewer资源

一些事件只能通过Qt传给osg
graphicsContext只能从eventQueue里面takeEvents出resize、鼠标的push、drag、键盘keyUp，keyDown事件，
frame事件是从QOpenGLWidget::paintGL里面调用的，没加入事件队列
大体来说，没看到qt的gc是怎么传递给``osg::camera``的