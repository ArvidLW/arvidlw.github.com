# Windows命令

***

## 打开开机启动文件夹

win+r，打开运行窗口，输入shell:starup，回车！

.

## Intellij打开失败（cannot lock system folders!）

**修复网络应用错误**

cmd运行：netsh winsock reset

　　netsh 是从windows xp sp2开始提供的功能强大的网络配置命令行工具。 是一个能够通过命令行操作几乎所有网络相关设置的接口，比如设置IP，DNS，网卡，无线网络等。

　　winsock是Windows网络编程接口，winsock工作在应用层，它提供与底层传输协议无关的高层数据传输编程接口。reset是对Winsock的重置操作。当执行完winsock的命令重启计算机后，需要重新配置IP等网络配置信息。

netsh winsock reset 是把它恢复到默认状态。作用是重置 Winsock 目录。如果一台机器上的Winsock协议配置有问题的话将会导致网络连接等问题，可通过使用netsh winsock reset命令来重置Winsock目录尝试恢复。这个命令可以重新初始化网络环境，以解决由于软件冲突、病毒原因造成的参数错误问题。例如电脑突然出现成功连接网络但不能上网、网络受限，打开网页出现显示问题，无法自动获取IP地址或指定IP地址仍无数据包传输，使用蓝牙适配器传输文件时提示“使用了与请求的协议不兼容的地址”等等网络协议应用相关故障。

