某日灵光乍现，想到如果在VRCHAT会不会有一种方法可以直接控制现实中的单片机并且直接看到反馈，于是这个项目它出现了！

首先我思考了具体的实现方案，因为之前在网上看到了Vowgan的教程 https://youtu.be/O3VeBzV9HgI?si=9Ud0GD97BvrAFTBp 让我知道了vrchat中通过设定可以实现某些物品的网络同步，
所以思考到了一个方案，玩家A通过按下vrchat的按钮在某个不为人知的地方生成一个物品，然后因为玩家B的世界也会接收到这个同步事件，所以可以通过VRchat中的摄像机捕捉到这个切换的物品，然后再通过屏幕扫描程序来识别不同物品向单片机发送对应指令，

于是最后的方案变成这样：UNITY端：玩家A按下按钮生成一个二维码,这个生成的二维码在玩家B的世界中出现,Python端：屏幕捕捉程序捕捉到二维码识别二维码中的指令并通过串口发送到单片机，Arduino端：接收到串口指令并实现相应动作，比如开关某LED，让LED变为呼吸状态或者随音乐变化亮度，最后通过手机ADB调试软件让多个手机的摄像头画面在电脑显示器中同步显示，然后利用OBS推流到VRchat的直播播放器完成多机位直播按下按钮后的灯光变化

最后就到了编程的环节啦，另外感谢jaynam对于Python程序的共同构建https://github.com/JaynamPan
1，Python程序用于捕捉识别二维码，此工具包实现了框选屏幕中间的一个区域进行识别，运行时会弹出一个窗口可以可视化正在捕捉的区域以方便快速和二维码对其，识别到二维码之后解析二维码里蕴含的文字指令，并把文字指令发送到单片机
【框选区域大小可以自己调整，设计为只框选为中间部分区域是为了提升响应速度降低延迟】
2，Arduino程序用于将接收到的串口指令转化为相应动作，目前在程序已有的动作有：

"SET ALL DUTY CYCLES TO 15"：将所有LED的占空比设置为15。

"SET ALL DUTY CYCLES TO 35"：将所有LED的占空比设置为35。

"SET ALL DUTY CYCLES TO 100"：将所有LED的占空比设置为100。

"TURN_OFF_LIGHT"：关闭所有LED灯。

"BLINK_LIGHT"：启用呼吸灯效果。

"TURN_OFF_BREATHING_EFFECT"：关闭呼吸灯效果，并恢复之前的亮度值。

"TURN_OFF_RED_LED"：关闭红色LED。

"SET RED DUTY CYCLE TO 15"：将红色LED的占空比设置为15。

"SET RED DUTY CYCLE TO 35"：将红色LED的占空比设置为35。

"SET RED DUTY CYCLE TO 100"：将红色LED的占空比设置为100。

"TURN_OFF_GREEN_LED"：关闭绿色LED。

"SET GREEN DUTY CYCLE TO 15"：将绿色LED的占空比设置为15。

"SET GREEN DUTY CYCLE TO 35"：将绿色LED的占空比设置为35。

"SET GREEN DUTY CYCLE TO 100"：将绿色LED的占空比设置为100。

"TURN_OFF_BLUE_LED"：关闭蓝色LED。

"SET BLUE DUTY CYCLE TO 15"：将蓝色LED的占空比设置为15。

"SET BLUE DUTY CYCLE TO 35"：将蓝色LED的占空比设置为35。

"SET BLUE DUTY CYCLE TO 100"：将蓝色LED的占空比设置为100。

"ENABLE_SOUND_CONTROL"：启用声音控制亮度功能。

"DISABLE_SOUND_CONTROL"：禁用声音控制亮度功能，并恢复之前的亮度值。

"SPRAY FOR 5 SECONDS"：启动喷雾功能，如果喷雾未被锁定，则执行喷雾操作15秒，并锁定60秒。

3.Unity程序与模型：首先我通过RHINO建模了以下按钮![image](https://github.com/8I-X-I8/VRCHAT-connect-realworld/blob/main/Unity_Button.jpg)

渐变色开关：LED整体亮度控制

三个圆盘开关：单独调整各自颜色三档亮度：对应三个最大的圆盘，每个圆盘最中间是15占空比再往外就是35，100

X标志按钮：单独调整各自颜色开关开关：每一个颜色LED开关对应各自颜色的X标志按钮

红色OFF标志开关：LED总体关闭

蓝色波浪标志开关：总体LED呼吸效果

绿色频谱标志开关：总体LED随音乐变化亮度

最中间半透明的圆球：烟雾机开关

建模完按钮来到Usharp脚本部分，以下是三个脚本各自的作用,整个程序的运行逻辑就是按下对应按钮，触发对应二维码从不可视的位置移动到可视的位置，之后按下其他按钮之前的二维码会退回不可视区域从而只显示最后按下按钮的对应二维码

ButtonScript
用来在本地触发二维码位置切换

QRCodeManager
管理并同步所有二维码，通过识别新按钮按下从而更新所有二维码状态，以让旧的二维码返回不可视位置

QRCodeScript
在检测到按钮切换指令时让对应二维码变更位置到可视区域，并触发VRCgameobject网络同步事件




