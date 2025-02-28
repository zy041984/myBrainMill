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