[XInput 游戏控制器 API - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/_xinput/)
# 版本
XInput 1.4是唯一可以在 C++/DirectX Windows 应用商店应用中使用的版本
# 功能
XInput 使应用程序能够从 XUSB 控制器接收输入。 API 通过 DirectX SDK 提供，驱动程序通过 Windows 更新提供。
# 与DirectInput比较
与使用 [DirectInput](https://learn.microsoft.com/zh-cn/previous-versions/windows/desktop/ee416842(v=vs.85)) 相比，使用 XInput 有几个优点：

- 与 [DirectInput](https://learn.microsoft.com/zh-cn/previous-versions/windows/desktop/ee416842(v=vs.85)) 相比，XInput 更易于使用且需要更少的设置
- Xbox 和 Windows 编程都将使用相同的核心 API 集，这使得编程能够更轻松地跨平台转换
- 将有大量控制器需要安装
- 仅当使用 XInput API 时，XInput 设备才具有振动功能
- 使用也DirectInput的EnumDevice可以查询到支持XInput的控制器，但是实测无法获得这个控制器的数据。因此如果这个控制器支持XInput，最好用XInput来获得数据。微软提供了一个函数来判断当前控制器是否支持XInput，https://learn.microsoft.com/zh-cn/windows/win32/xinput/xinput-and-directinput
# 心得
- 看头文件当前版本是1.4的
- 最多支持4个手柄
- 全部配对蓝牙以后，关闭蓝牙再按照不同顺序开启蓝牙，不知道手柄的顺序会不会变，不会
- 使用非常简单，轮询调用函数XInputGetState即可，输入的第一个参数就是手柄index，如果返回ERROR_DEVICE_NOT_CONNECTED，就说明手柄未连接。如果返回ERROR_SUCCESS，就可以查询每个按钮的状态
- 出于性能原因，不要对每个帧的未连接手柄调用XInputGetState。改为每隔几秒钟检查一次新控制器。
- XInputGetState的dwPacketNumber是一个数字。如果这个数字有了变化，表示手柄状态有变化，按下按钮或者释放按钮都能引起状态变化。最好不要用这种机制，突然释放摇杆时，位置不准确
手柄的新设计
配置文件里写好