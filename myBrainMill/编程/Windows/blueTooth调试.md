# 抓包
## 工具
[安装蓝牙测试平台包 - BTP - Windows drivers | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/bluetooth/testing-btp-setup-package)在这里下载了windows的蓝牙调试工具

还需要安装wireShark
## 抓包过程
- 控制台管理员模式运行`btvs.exe -Mode Wireshark`
- 自动弹出wireshark抓包，按需停止。
## 数据包说明
### Bluetooth HCI H4
指蓝牙主机控制器接口HCI通过UART传输层H4协议实现的数据通信格式
### Bluetooth HCI ACL Packet
蓝牙主机控制器接口HCI中用于传输异步无连接数据Asynchronous Connectionless Data的核心数据包类型
Handle:连接句柄,区分不同链路
Length:数据部分的长度
Data:实际传输的L2CAP数据，或应用层数据
### Bluetooth L2CAP Protocol
蓝牙协议栈中用于逻辑链路控制与适配的核心协议层
CID:标识逻辑通道,长度2Byte,如0x0004为ATT协议通道
Length:SDU长度,长度2Byte,经典蓝牙支持64KB,BLE基础模式无分段
SDU:实际传输的上层协议数据,长度可变,如GATT特征值、RFCOMM串口数据
### Bluetooth Attribute Protocol
蓝牙低功耗BLE协议栈中用于属性访问的核心协议层
把设备数据封装为属性,每个属性由handle,UUID,属性值组成
Opcode:操作类型,长度1Byte,如0x01表示Read request,0x02表示write request,0x1b表示handle value notification
handle:属性句柄,长度2Byte
value:属性值
# 分析工具
[BLEDebug.ZIP - 南京沁恒微电子股份有限公司](https://www.wch.cn/downloads/BLEDebug_ZIP.html)
可以查看连接设备的广播信息，设备特征调试等