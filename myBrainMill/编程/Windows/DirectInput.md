[Introduction to DirectInput | Microsoft Learn](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/ee418273(v=vs.85))
DirectInput是一个API，可以输入joystick以及game controller，即使程序后台运行，也可以获取数据。
# 基本概念
DirectInput对象，根接口
Device，表示某个设备，如鼠标，键盘，joystick
DirectInputDevice对象，描述一个device
Device对象，描述DirectInputDevice对象上的一个按钮，摇杆等
# 流程
1. 创建根接口DirectInput对象，用根接口的函数来枚举设备，创建DirectInputDevice对象
2. 枚举设备，找到某个device的时候可以检测它的能力属性，为这个device生成一个id，用于创建DirectInputDevice对象。
3. 创建DirectInputDevice对象
4. 初始化device。设置cooperative等级，即device可不可以与其他程序共享。设置数据格式，例如数据缓冲区的大小和格式
5. acquire该device，即通知DirectInput本程序开始要从device中获取数据了
6. 获取数据，以一定时间间隔获取device的数据
7. 关闭，unacquire该device，release该device，release根接口DirectInput对象
# action mapping
可以简化用户设置。
为某个设备设置action mapping，让DirectInput来决定使用哪个按钮来生成想要的action。例如，可以创建一个叫做AXIS_BRAKE的action，让DirectInput来自行决定设备上的哪个按钮能产生这个action。然后每帧获取数据时，不是获取按钮状态，而是获取action的事件
# 一些解释
1. cooperative level包括程序是否前台后台运行时都能从device中获取数据，以及程序是否独占这个device。
2. 必须accquire某个device，然后才能从其中获取数据。想要更改该device的属性必须先release该device
3. 支持缓冲数据或实时数据。实时数据例如摇杆实时位置，缓冲数据多用于判断event，比如button是否长按。可以同时使用缓冲和实时。
4. 如何判断数据是否准备好了，可以使用polling机制或事件通知机制。
    事件机制是DirectInput把按钮信息以事件的形式通知一个waitForSingleObject的线程。 
    polling机制，有些设备不产生硬件中断，只有调用poll才会使得设备准备好数据。到底需不需要先poll再获取数据需要使用GetCapabilities查询，但是可以无脑调用poll再获取数据，多调用个poll不会有坏结果。
1. 缓冲数据会带有毫秒时间戳和递增的序号
2. joystick的axis数据要自行判断死区和饱和区。也可以使用CPOINT结构来指定axis的输出曲线
# 版本
 IDirectInputDevice8接口中需要指定基于DirectX的版本号，宏DIRECTINPUT_VERSION默认为0x0080，
# 查询GUID和连接状态
GUID可以用DIDEVICEINSTANCE::guidInstance获得
`LPDIRECTINPUT8::GetDeviceStatus`可以查询当前手柄连接状态,形参为GUID。
poll函数不能在循环中无脑调用，最好先调用`LPDIRECTINPUT8::GetDeviceStatus`，手柄当前正常连接，再调用`LPDIRECTINPUTDEVICE8::poll`和`LPDIRECTINPUTDEVICE8::GetDeviceState`获得手柄数据
# 配置摇杆
设置axis为abs
```
DIPROPDWORD dipdw;
dipdw.diph.dwSize = sizeof(DIPROPDWORD);
dipdw.diph.dwHeaderSize = sizeof(DIPROPHEADER);
dipdw.diph.dwObj = 0; // device property 
dipdw.diph.dwHow = DIPH_DEVICE;
hr = mJoystick[i]->GetProperty(DIPROP_AXISMODE, &dipdw.diph);
if (hr != DI_OK)
{
	spdlog::info("GetProperty failed joystick {}", i);
	return;
}
if (dipdw.dwData != DIPROPAXISMODE_ABS)
{
	DIPROPDWORD dipdw;
	dipdw.diph.dwSize = sizeof(DIPROPDWORD);
	dipdw.diph.dwHeaderSize = sizeof(DIPROPHEADER);
	dipdw.diph.dwObj = 0; // device property 
	dipdw.diph.dwHow = DIPH_DEVICE;
	dipdw.dwData = DIPROPAXISMODE_ABS;
	hr = mJoystick[i]->SetProperty(DIPROP_AXISMODE, &dipdw.diph);
	if (hr != DI_OK)
	{
		spdlog::info("SetProperty DIPROP_AXISMODE failed joystick {}", i);
		return;
	}
}
```
设置axis_X的deadzone
```
	int dz{ 200 };
	DIPROPDWORD dipdw;
	dipdw.diph.dwSize = sizeof(DIPROPDWORD);
	dipdw.diph.dwHeaderSize = sizeof(DIPROPHEADER);
	dipdw.diph.dwObj = DIJOFS_X;
	dipdw.diph.dwHow = DIPH_BYOFFSET;
	dipdw.dwData = dz;
	hr = mJoystick[i]->SetProperty(DIPROP_DEADZONE, &dipdw.diph);
	if (hr != DI_OK)
	{
		spdlog::info("SetProperty DIPROP_DEADZONE x failed joystick{}", i);
		return;
	}
```