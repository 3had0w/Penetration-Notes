#####打造不一样的Shfit映像劫持后门
<hr>
```
##实验环境：
目标机-Win 7
攻击机-Kali Linux
```
<hr>

##演示过程：

>最近研究留后门姿势发现一个比较有意思的跟大家分享一下，大佬勿喷！

######0x00 大家一定都知道映像劫持后门，修改注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`下`sethc.exe`，添加一个`Debugger`字符值（REG_SZ），并且赋值为`cmd.exe`的执行路径为`C:\windows\system32\cmd.exe`，如图：
![修改注册表](https://upload-images.jianshu.io/upload_images/6450757-2915d8e6045104c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######0x01 如上文所述，修改IFEO中的“debugger”键值，用来替换原有程序的执行。
`如：键入五下Shift执行sethc.exe程序时会执行cmd.exe程序`
![普通Shift后门](https://upload-images.jianshu.io/upload_images/6450757-858f8c4f87309c55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######0x02 与上文对比，不一样的Shift后门是怎样的呢？
`实现效果：键入五下Shift执行时，先执行sethc.exe程序，当sethc.exe程序静默退出时，执行cmd.exe程序`

######0x03 怀揣着0x02的目标我们开始复现，在网上收集资料时发现，注册表`Image File Execution Options`下有一个`GlobalFlag`项值，在MSDN的博客上，发现`GlobalFlag`由`gflags.exe`控制。
文章地址：https://blogs.msdn.microsoft.com/junfeng/2004/04/28/image-file-execution-options/
![博客截图.png](https://upload-images.jianshu.io/upload_images/6450757-569cd417961e9ba2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######0x04 下载`gflags.exe`开始研究，在`Silent Process Exit`这个选项卡中发现了挺有趣的东西。根据微软官方介绍，从Windows7开始，可以在`Silent Process Exit`选项卡中，可以启用和配置对进程静默退出的监视操作。在此选项卡中设定的配置都将保存在注册表中。

![填写配置](https://upload-images.jianshu.io/upload_images/6450757-add6f1cfcc112414.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


######0x05 填入如上配置后点击应用，开始测试。使用Process Explorer进行检测进程的变化发现键入五下Shift执行时，先执行sethc.exe程序，当sethc.exe程序静默退出时，执行cmd.exe程序。
![完成目标](https://upload-images.jianshu.io/upload_images/6450757-89edcd9c648c5169.gif?imageMogr2/auto-orient/strip)

######0x06 进一步研究，发现其实是工具帮我们添加并修改了IFEO目录下`sethc.exe`的`GlobalFlag值`，如图：
![工具自动添加值](https://upload-images.jianshu.io/upload_images/6450757-93cb45dccbffba15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######以及SilentProcessExit下ReportingMode和MonitorProcess两个项值，如图：
![工具自动添加值](https://upload-images.jianshu.io/upload_images/6450757-eda013735bed01d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######0x07 那么，接下来我们可以修改上文中`MonitorProcess`值来放我们的后门，例如Powershell反弹shell，配合五下Shift就可以神不知鬼不觉的进行反连。
`如：键入五下Shift后正常弹粘滞键，关闭之后执行我们的powershell代码`，如图：
![powershell+shift后门](https://upload-images.jianshu.io/upload_images/6450757-13c2bfc6f33e5cb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

######0x08 脑补一下连接成功的画面~实验结束，希望大佬勿喷。
![成功反弹shell](https://upload-images.jianshu.io/upload_images/6450757-f42a68e9c4bfa50e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考文章：
[https://blogs.msdn.microsoft.com/junfeng/2004/04/28/image-file-execution-options/](https://blogs.msdn.microsoft.com/junfeng/2004/04/28/image-file-execution-options/)
[https://blog.csdn.net/johnsonblog/article/details/8165861](https://blog.csdn.net/johnsonblog/article/details/8165861)
[http://download.microsoft.com/download/A/6/A/A6AC035D-DA3F-4F0C-ADA4-37C8E5D34E3D/setup/WinSDKDebuggingTools_amd64/dbg_amd64.msi](http://download.microsoft.com/download/A/6/A/A6AC035D-DA3F-4F0C-ADA4-37C8E5D34E3D/setup/WinSDKDebuggingTools_amd64/dbg_amd64.msi)


<hr>

