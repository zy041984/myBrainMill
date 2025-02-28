game controller指的是类似于XBOX的手柄gamepads，有两个模拟轴，一个dpad，右侧四个按钮，每侧shoulder两个按钮。SDL2知道这个手柄各个轴，按钮的属性。

joystick指的是各种模拟手柄，方向盘，等等。DirectInput或系统的API根本不知道各个轴的范围，信息，属性等等，只能让用户自己设置。

所以如果知道输入是gamepads，就用gamecontroller这个类就可以
如果知道输入是joystick，就用通用joystick API，用户来配置每个轴的属性，默认位置，死区，最大值最小值。