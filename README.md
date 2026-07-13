
# hoverboard-firmware-hack-FOC - 经典四档位卡丁车长春笨鸟升级版（按键换档）
这是基于著名的 EmanuelFeru 平衡车主板 FOC larsmm 优化版bbcar的基础上进一步修改优化而来的经典四档位卡丁车版本，是bbcar固件 的改进版本。

![bobbycar pic](http://xbbcar.com/zb_users/upload/2025/10/202510231761181911998954.jpg)

### 功能特点
- 通过方向盘上的两个电位器控制：1. 前进，2. 刹车/倒车/涡轮增压模式
- 4种驾驶模式，分别对应不同的速度、加速度和功能
- 支持加速、刹车坡度控制，涡轮增压坡度控制，( 可在 config.h 中禁用)
- 其他功能与原版固件完全一致： https://github.com/EFeru/hoverboard-firmware-hack-FOC
- 倒车安全提示蜂鸣器 :( 可在 config.h 中禁用)
### 安全功能
- 电位器超出范围检测：可控刹车并断电（发出快速"滴-滴-滴"的提示音）
- 低电压蜂鸣提示并断电（请正确校准电池并使用质量可靠的保护板！）
- 电机故障检测
- 查看蜂鸣代码含义： 诊断说明
### 硬件适配
- 本固件仅兼容 搭载STM32F103RCT6和GD32F103RCT6芯片的单主板，GD32e103r8tb芯片可参考xbbcar相关项目。
- 电位器接线原理图： https://larsm.org/wp-content/uploads/2019/08/connecting-potis-v2.png) ([full guide](https://larsm.org/allrad-e-bobby-car/
- 使用一个自复位按键巡航控制档位，或者两个按键，一个增加档位一个减小档位。

### 换挡功能按键
- 主板最右侧的绿黑线（button2）每按一下增加一个档位，到4档后回到1档。
- 主板最右侧的懒黑线（button1）每按一下增加一个档位，到4档后回到1档。

### 换挡按键跳档情况
- 一些主板有信号干扰情况（如果pwm模式下失控的原理一样），按键也有可能存在不稳定情况，并不是所有主板都会跳档。
- 如果出现跳档情况，可尝试这两种处理办法。1、尝试尽量缩短换档按键的线路看是否改善；2、缩短线路无效可在按键两根线之间（信号线和对地）加104电容或100K电阻，优先推荐加电容方式。

### 驾驶模式
开机时按住一个或多个电位器可激活对应模式（数据基于满电12S锂电池）：

- 模式1 - 儿童模式：左电位器按住开机，最高速度 ~3 km/h，倒车速度极慢，无涡轮增压
- 模式2 - 合规模式：不按任何电位器开机，最高速度 ~<6 km/h，倒车速度慢，无涡轮增压
- 模式3 - 娱乐模式：右电位器按住开机，最高速度 ~12 km/h，无涡轮增压
- 模式4 - 动力模式：两个电位器同时按住开机，最高速度 ~22 km/h，开启涡轮增压后可达 ~39 km/h
开机后会播放欢迎旋律，然后用1-4次快速蜂鸣提示当前选择的模式，默认模式为模式2。

### 涡轮增压功能
仅在模式4下可用弱磁提速。仅在速度达到最高速度的80%以上才能激活，保持前进电位器完全压下，然后将倒车电位器完全按下即可激活涡轮增压，额外获得40%动力。请注意：此速度非常危险，请务必小心操作！

### 功率与电池
满油门+涡轮增压在干燥沙地峰值电流约为50A，对应12S锂电池约2300W功率，正常道路行驶平均电流会低很多。更大功率实际意义不大，因为轮子已经无法把更多动力传递到地面。主板散热没有问题，在主板过热前轮子就会先过热。10Ah 12S电池在模式4不开启涡轮增压的情况下续航约20km。请务必使用质量可靠的BMS！

### 编译与烧录
新主板芯片默认开启写保护，必须 先解锁 。

- 安装 Visual Studio Code
- 在VSCode扩展商店安装 PlatformIO IDE
- 点击 文件 --> 打开文件夹：选择 hoverboard-firmware-hack-FOC-xbbcar
- 将st-link v2调试器连接到主板（不要连接3.3V） 参考图
- 给主板连接电池或电源
- 按住电源按钮，点击"Upload"按钮，等待上传完成后松开电源按钮即可
### 校准ADC（电位器，霍尔踏板）
如果ADC未校准，开发板会发出快速的"滴-滴-滴"提示音，开机后立即断电

- 断开串口和st-link适配器
- 确保两个主板一起校准！
- 开机后按住按钮不放直到听到蜂鸣（大约10秒），再过1秒会听到另一个更低的蜂鸣
- 然后你有20秒时间，把两个电位器都从最小位置移动到最大位置，最后停在中间位置（这种情况相当于最小位置）。等待20秒结束，或者短按一次电源按钮完成校准。
### ADC（电位器）精细调整
- 电位器范围的起点和终点通常会存在死区
- 编辑 config.h 的 ### DEFAULT SETTINGS ### 部分：
- 减小 ADC_MARGIN 可以缩小死区，死区能调多小取决于你的电位器质量。如果设置太小，车辆可能会在没按电位器时缓慢自行启动，或者无法达到最高速度。我推荐使用霍尔电位器代替机械电位器。
- 每次修改 ADC_MARGIN 后都需要重新上传固件并重新校准电位器。
### 电池电压校准
- 编辑 config.h 的 ### BATTERY ### 部分：
- 将usb串口转换器（GND、TX、RX， 千万不要接3.3V！ ）连接到右侧主板的排针。 参考图
- 点击"串口监控"按钮，以115200波特率查看主板的调试输出。
- 给主板接入已知电压的电源，用万用表确认电压准确。如果实际电压是45.00V，就把 BAT_CALIB_REAL_VOLTAGE 设置为4500。如果主板立即断电，要么是ADC没有校准（先校准ADC），要么是欠压保护触发了（请使用更高电压，最高支持51V）
- 将串口输出的 BatADC 值填写到 config.h 的 BAT_CALIB_ADC 中
- 确保 BAT_CELLS 设置和你实际的电池节数一致
- 重新上传固件即可
### 随时切换FOC控制模式
- 按住电源按钮直到听到蜂鸣，松开后立即再次按住至少2秒，听到"滴滴"声表示已切换到FOC模式。要切回正弦模式，关机重启即可。弱磁提速在FOC模式下效果不佳，速度会明显更慢。
### 故障排查
- 有时候会出现一个主板开机一个主板关机的罕见情况，最简单的方法是：把一个电位器按到一半，然后按电源按钮，尝试开机的主板会检测到非法状态然后关机。
- 本固件禁用了通过特殊方式按开机按钮触发的电流和最大速度校准功能。
- 开机后立即断电且没有额外蜂鸣码：这种情况是电位器在有效范围内，但不在检测驾驶模式的有效位置上：最大和最小位置是有效的，中间是无效位置。使用串口调试可以获得更多信息。
### 待办事项
- 目前固件运行在正弦模式，不是FOC模式。这是因为旧版本中弱磁功能在FOC下无法工作，未来需要重新测试。
### 更多信息
- https://github.com/EFeru/hoverboard-firmware-hack-FOC
- https://larsm.org/allrad-e-bobby-car/
- https://figch.de/index.php?nav=bobbycar
- https://github.com/larsmm/hoverboard-firmware-hack-bbcar
- https://xbbcar.com/


## hoverboard-firmware-hack-FOC - bobby car edition

This is a 4-wheel-drive-bobbycar-optimized version of the famous [hoverboard mainboard FOC firmware by EmanuelFeru](https://github.com/EFeru/hoverboard-firmware-hack-FOC). This an improved version of my [old bbcar firmware](https://github.com/larsmm/hoverboard-firmware-hack-bbcar).

![bobbycar pic](https://raw.githubusercontent.com/larsmm/hoverboard-firmware-hack-bbcar/master/pic1.jpg)

### Features
* Controlled by 2 potis on the steering wheel: 1. forward, 2. break or backward or turbo mode
* 4 driving modes with different speeds, accelerations and features
* Acceleration and breaking ramps, turbo ramp
* Everything else is identical to: https://github.com/EFeru/hoverboard-firmware-hack-FOC
* Backwards beep for safety :D (can be disabled in config.h)

### Safety features
* Poti-out-of-range detection: controlled breaking and poweroff (fast di-de-di-de-di-de-di-de sound)
* Low battery beep and poweroff (calibrate battery and use a good BMS!)
* Motor fault detection
* See [beep codes](https://github.com/EFeru/hoverboard-firmware-hack-FOC/wiki/Diagnostics)

### Hardware
* This firmware is compatible with [Single Mainboards with STM32F103RCT6 and GD32F103RCT6 chips](https://github.com/EFeru/hoverboard-firmware-hack-FOC/wiki/Firmware-Compatibility) only.
* Schematic for [connecting potis](https://larsm.org/wp-content/uploads/2019/08/connecting-potis-v2.png) ([full guide](https://larsm.org/allrad-e-bobby-car/))
* Use 1 power button and connect it to both boards

### Driving modes
You can activate them by holding one or more of the potis while poweron. (km/h @ full 12s battery):
* Mode 1 - Child: left poti, max speed ~3 km/h, very slow backwards, no turbo
* Mode 2 - STVO: no poti, max speed ~<6 km/h (verify it), slow backwards, no turbo
* Mode 3 - Fun: right poti, max speed ~12 km/h, no turbo
* Mode 4 - Power: both potis, max speed ~22 km/h, with turbo ~39 km/h

After poweron it beeps the welcome melody, then the mode in 1 to 4 fast beeps. Default is mode 2.

### Turbo
Field weakening is availible in mode 4 only. It can be activated above 80% of top speed only. Keep forward poti fully pressed. To activate turbo, fully press the backwards poti to get additional 40% more power. :) Please be very careful, this speed is dangarous!

### Power and battery
Peak power on full throttle+turbo on dry sand is around 50A = ~2300W at 12 lithium battery cells. The average current on normal road is much less. More power does not make much sense. The wheels are not able to get the power onto the ground. Cooling of the board is no problem, the wheels will get too hot before. You will get a range of ~20km out of a 10Ah 12s battery on mode 4 without turbo. Use a good BMS!!!

### Build and flash
On new boards the chip is write-protected and must be [unlocked first](https://github.com/EFeru/hoverboard-firmware-hack-FOC/wiki/How-to-Unlock-MCU-flash).
* Install Visual Studio Code
* Install PlatformIO IDE within VSCode extension system
* File --> open folder: hoverboard-firmware-hack-FOC-bbcar
* Connect st-link v2 adapter to the board (do not connect 3.3V) [pic](https://github.com/EFeru/hoverboard-firmware-hack-FOC#hardware)
* Connect battery or power supply to the board
* Press and hold poweron button, press "Upload" button, wait for upload to finish, release poweron button

### Calibrate ADCs (potis)
If the ADCs are not calibrated, the board will make a fast di-de-di-de-di-de-di-de sound and will poweroff emiedetly after poweron
* Disconnect serial and st-link adapters
* Make sure you calibrate both boards together!
* Poweron and hold the button until a beep sound (after ~10s). After 1 sec you will hear another lower beep.
* Now you have 20s to move both potis to min and max positions. Release in the mid position (which equals min position in this case). Wait for the 20s to end or short press poweron button.

### Fine tune ADCs (potis)
* You have a dead zone at the start and end of the poti range.
* Edit config.h ### DEFAULT SETTINGS ### section:
* Make ADC_MARGIN smaller to make the dead zone smaller. It depends on the quality of your potis how small you can make it. If it is too small, the car sometimes starts driving slowly without pressing the potis or you will not reach max speed. I recommand to use hall-potis instead of mechanical ones.
* Re-upload and recalibrate potis after every ADC_MARGIN-change.

### Calibrate battery voltage
* Edit config.h ### BATTERY ### section:
* Connect usb-serial converter (GND, TX, RX, NOT 3.3V!) to the right sideboard cable. [pic](https://github.com/EFeru/hoverboard-firmware-hack-FOC#hardware)
* Press "Serial Monitor" button to get debug feedback from the board at 115200.
* Connect a known voltage to the board, verify with multimeter. For 45.00V, write 4500 to BAT_CALIB_REAL_VOLTAGE. If board powers off emiedetly, either the ADCs are not calibrated (calibrate them first) or undervoltage protection jumps in (use higher voltage up to 51V)
* Write BatADC from serial output to BAT_CALIB_ADC in config.h
* Make sure BAT_CELLS matches your cell count
* Re-upload

### Switch to FOC_CTRL on the go
* press the power button until beep. release the button and immediately press again for >= 2 sek. beep-beep indicates the mode switch to FOC. To switch back to SIN, poweroff and poweron again. Field weakening is not working very well in FOC mode. It is significantly slower.

### Trouble shooting
* Sometimes 1 board is on and the other off. This is a rare case. The easiest way to poweroff both boards is to press a poti half way and press the power button. The board which is trying to poweron will detect this as an invalid state and poweroff.
* The current and max speed calibration function initiated by pressing poweron button in a special way is disabled in thes firmware.
* After poweron the board immediately powers off without an additional beep code: This happens if potis are in valid range but not in a valid position for detecting driving mode: max and min positions are valid, in between is invald. Use serial debug to get more info.

### ToDo
* At the moment it runs on sinus mode, not foc. This is because in older versions field weakening was not working on foc. Have to try again.

### FAQ
* Why SIN instead of FOC? At the time when I created the code, FOC was not working properly, so I used SIN. ~2 years later I tried FOC again. It was working but driving behaviour was worse.
* Can I use sideboards, other inputs like nunchuck, serial, my own mixing, ...? No, it is not possible. I heavily modified the structure of the code. I even skipped the orgiginal mixing and made my own one.

### More info
* https://github.com/EFeru/hoverboard-firmware-hack-FOC
* https://larsm.org/allrad-e-bobby-car/
* https://figch.de/index.php?nav=bobbycar
