---
layout:     post
title:     树莓派安装CUPS实现普通打印机通过iphone打印
subtitle:   
date:       2021-16-19
author:     laosuan
header-img: 
catalog: true
tags:
    - 



---



用树莓派和 CUPS 打印服务器将你的打印机变成网络打印机。

基本设置

设置树莓派是非常简单的事。我下载了 Raspbian[2] 镜像，并将它写入到我的 microSD 卡中。然后，使用它来引导一个连接了 HDMI 显示器、 USB 键盘和 USB 鼠标的树莓派。之后，我们开始对它进行设置！

这个树莓派系统自动引导到一个图形桌面，然后我做了一些基本设置：设置键盘语言、连接无线网络、设置普通用户帐户（pi）的密码、设置管理员用户（root）的密码。

我并不打算将树莓派运行在桌面环境下。我一般是通过我的普通的 [Linux](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.dataguru.cn%2Farticle-4696-1.html%3Funion_site%3Dinnerlink) 计算机远程来使用它。因此，我使用树莓派的图形化管理工具，去设置将树莓派引导到控制台模式，但不以 pi 用户自动登入。

重新启动树莓派之后，我需要做一些其它的系统方面的小调整，以便于我在家用网络中使用树莓派做为 “服务器”。我设置它的 DHCP 客户端为使用静态 IP 地址；默认情况下，DHCP 客户端可能任选一个可用的网络地址，这样我会不知道应该用哪个地址连接到树莓派。我的家用网络使用一个私有的 A 类地址，因此，我的路由器的 IP 地址是 10.0.0.1，并且我的全部可用地 IP 地址是 10.0.0.x。在我的案例中，低位的 IP 地址是安全的，因此，我通过在 /etc/dhcpcd.conf 中添加如下的行，设置它的无线网络使用 10.0.0.11 这个静态地址。

interface wlan0

static ip_address=10.0.0.11/24

static routers=10.0.0.1

static domain_name_servers=8.8.8.8 8.8.4.4

在我再次重启之前，我需要去确认安全 shell 守护程序（SSHD）已经正常运行（你可以在 “偏好” 中设置哪些服务在引导时启动它）。这样我就可以使用 SSH 从普通的 Linux 系统上基于网络连接到树莓派中。

打印设置

现在，我的树莓派已经连到网络上了，我通过 SSH 从我的 Linux 电脑上远程连接它，接着做剩余的设置。在继续设置之前，确保你的打印机已经连接到树莓派上。

设置打印机很容易。现代的打印服务器被称为 CUPS，意即“通用 Unix 打印系统”。任何的 Unix 系统都可以通过 CUPS 打印服务器来打印。为了在树莓派上设置 CUPS 打印服务器。你需要通过几个命令去安装 CUPS 软件，并使用新的配置来重启打印服务器，这样就可以允许其它系统来打印了。

$ sudo apt-get install cups

$ sudo cupsctl --remote-any

$ sudo /etc/init.d/cups restart

在 CUPS 中设置打印机也是非常简单的，你可以通过一个 Web 界面来完成。CUPS 监听端口是 631，因此你用常用的浏览器来访问这个地址：

https://10.0.0.11:631/

你的 Web 浏览器可能会弹出警告，因为它不认可这个 Web 浏览器的 https 证书；选择 “接受它”，然后以管理员用户登入系统，你将看到如下的标准的 CUPS 面板：





![img](https:////upload-images.jianshu.io/upload_images/16247009-3ab8184c735d8144.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)



这时候，导航到管理标签，选择 “Add Printer”。





![img](https:////upload-images.jianshu.io/upload_images/16247009-bba54b497304a54c.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)



如果打印机已经通过 USB 连接，你只需要简单地选择这个打印机和型号。不要忘记去勾选共享这个打印机的选择框，因为其它人也要使用它。现在，你的打印机已经在 CUPS 中设置好了。





![img](https:////upload-images.jianshu.io/upload_images/16247009-9f5a22753aa049e1.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)



客户端设置

从 Linux 中设置一台网络打印机非常简单。我的桌面环境是 GNOME，你可以从 GNOME 的“设置”应用程序中添加网络打印机。只需要导航到“设备和打印机”，然后解锁这个面板。点击 “添加” 按钮去添加打印机。

在我的系统中，GNOME 的“设置”应用程序会自动发现网络打印机并添加它。如果你的系统不是这样，你需要通过树莓派的 IP 地址，手动去添加打印机。





![img](https:////upload-images.jianshu.io/upload_images/16247009-d7b2a2b97c916f59.png?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)



设置到此为止！我们现在已经可以通过家中的无线网络来使用这台打印机了。我不再需要物理连接到这台打印机了，家里的任何人都可以使用它了！





教程2：

**准备工作：**

1.连接网络的树莓派

2.usb接口的打印机

步骤如下：

1.安装CUPS软件,”CUPS“是linux下可以用通用打印系统（百度百科[http://baike.baidu.com/view/887944.htm](https://links.jianshu.com/go?to=http%3A%2F%2Fbaike.baidu.com%2Fview%2F887944.htm)）

sudo apt-get install cups

**2.允许pi用户配置CUPS**

sudo usermod -a -G lpadmin pi

**3.备份替换CUPS配置：**

关闭服务

sudo service cups stop

**备份配置文件**

sudo mv /etc/cups/cupsd.conf /etc/cups/cupsd.conf.bak

**替换配置文件（root用户去掉“sudo”）**

sudo cd /etc/cups/ && sudo wget http://jxeeno.tk/local–files/blog:raspberry-pi:print-server/cupsd.conf

**重启服务**

sudo service cups start

**4.PC端用IE浏览器打开网站“https://树莓派ip:631/ ”**

5.点击“Administrator”界面添加对应的打印机，登录帐号和密码是树莓派的pi用户的密码

**在local printers中找到usb连接的打印机，打印机名“scx3405”**



![img](https:////upload-images.jianshu.io/upload_images/16247009-144399728a4cd94d.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/832/format/webp)







![img](https:////upload-images.jianshu.io/upload_images/16247009-52e453c7962e8e3b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/753/format/webp)







![img](https:////upload-images.jianshu.io/upload_images/16247009-7e88e0bfda507811.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/864/format/webp)









![img](https:////upload-images.jianshu.io/upload_images/16247009-91c860fcbdc8ede6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/886/format/webp)









![img](https:////upload-images.jianshu.io/upload_images/16247009-7f8b8d29c0c37add.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/949/format/webp)





**到此树莓派的打印机设置就完成了！**

**6.网络打印机的地址“http://树莓派ip:631/printers/brother”，PC端按照此地址添加网络打印机和驱动就行了**





![img](https:////upload-images.jianshu.io/upload_images/16247009-98891a130a2739d6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/562/format/webp)





![img](https:////upload-images.jianshu.io/upload_images/16247009-df63105773a6c508.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/626/format/webp)





![img](https:////upload-images.jianshu.io/upload_images/16247009-523dc4dbb934a1bd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/599/format/webp)





![img](https:////upload-images.jianshu.io/upload_images/16247009-78b40de4a2baa894.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/610/format/webp)





![img](https:////upload-images.jianshu.io/upload_images/16247009-b3214dbe0d8f8f88.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/857/format/webp)



教程3：HP1007

虽然很多文章提到了raspberry树莓派如何安装cups实现共享打印机服务，但是我自己试下来发现HP P1007总是无法使用，折腾了很久，终于找到了方法，记录一下。

默认raspberry树莓派已经做好基本设置，IP，SSH之类已经OK。

首先执行更新，我之前就是没执行更新，导致后续操作错误，下载安装时会提示404 找不到文件

sudo apt-get update

更新一番之后安装最新的HPLIP，按照国外的说法，安装好之后应该能解决大部分HP打印机的使用问题。

sudo apt-get install hplip

安装hplip的时候应该已经同步安装好Cups了，如果没有，可以执行下列命令继续安装。

sudo apt-get install cups

安装完hplip之后，配置用户，把pi用加到lpadmin，如果是别的用户，记得更换用户名

sudo usermod -a -G lpadmin pi

下面替换CUPS的配置文件，首先停止服务

sudo service cups stop

备份原有文件

sudo mv /etc/cups/cupsd.conf /etc/cups/cupsd.conf.bak

//2015.9.21 update 貌似配置文件无法下载了

//从服务器上下载新的配置文件

//cd /etc/cups/

//sudo wget http://jxeeno.tk/local–files/blog:raspberry-pi:print-server/cupsd.conf

更改配置文件部分参数如下：

\# Only listen for connections from the local machine.

\#Listen localhost:631



\#CHANGED TO LISTEN TO LOCAL LAN

Port631



\# Restrict access to the server…

Orderallow,deny

Allow@Local



\# Restrict access to the admin pages…

Orderallow,deny

Allow@Local



\# Restrict access to configuration files…

AuthTypeDefault

Requireuser@SYSTEM

Orderallow,deny

Allow@Local



保存，退出cupsd.conf。

完成，现在可以启动服务了

sudo service cups start

下一步设置打印机，首先将打印机连接到树莓派上，然后在浏览器中输入 https://你的树莓派ip:631 进入配置界面

打开的是一个网站，在上面菜单栏中找到Administrator，会提示你用户名和密码，输入树莓派的用户名和密码即可

登录后，在Printers栏目中点击Add Printer

显示的Local Printers会有你连接上的那台打印机，应该是带有USBxxx之类的，选中它点击continue

在这个界面Name是打印机的名字，以后用来在URL中确定打印机，所以短一些比较好

勾选Sharing，别的不用变，点Continue

下一个界面选择驱动程序，这边的时候记得选择recommended的那个驱动，如果自己有ppd，可以上传打印驱动程序

最后就设置完成了，复制最后一个界面的URL，也就是类似于http://你的树莓派ip:631/printers/打印机名 就可以在其他电脑上添加共享打印机了

如果，如果到这里你添加了树莓派共享的打印机，但是还是没有用的话，请往下看：

按照[http://foo2xqx.rkkda.com/](https://links.jianshu.com/go?to=http%3A%2F%2Ffoo2xqx.rkkda.com%2F)的意思，是不建议使用系统自带的[foo2zjs driver](https://links.jianshu.com/go?to=http%3A%2F%2Ffoo2zjs.rkkda.com%2F)或者[foo2xqx driver](https://links.jianshu.com/go?to=http%3A%2F%2Ffoo2xqx.rkkda.com%2F)的。需要自己根据自己的打印机编译对应的驱动。我的打印机是P1007，应该选择是的[foo2xqx driver](https://links.jianshu.com/go?to=http%3A%2F%2Ffoo2xqx.rkkda.com%2F)。接下来就是自己编译了，网站上也有教程。给人家打个广告吧。

**foo2xqx:**  **a linux printer driver for XQX stream protocol**

e.g. HP LaserJet P1005, P1006, P1007, P1008, P1505, P1505n, P2014, P2014n, M1005 MFP, M1120 MFP

首先下载foo2xqx

wget[http://foo2zjs.rkkda.com/foo2zjs.tar.gz](https://links.jianshu.com/go?to=http%3A%2F%2Ffoo2zjs.rkkda.com%2Ffoo2zjs.tar.gz)

解压缩

$ tar zxf foo2zjs.tar.gz

$ cd foo2zjs

编译和安装

Compile:

​    $ make

Get extra files from the web, such as .ICM profiles for color correction,

and firmware.  Select the model number for your printer:

​    $ ./getweb P1005     # Get HP LaserJet P1005 firmware file

​    $ ./getweb P1006     # Get HP LaserJet P1006 firmware file

​    $ ./getweb P1007     # Get HP LaserJet P1007 firmware file

​    $ ./getweb P1008     # Get HP LaserJet P1008 firmware file

​    $ ./getweb P1505     # Get HP LaserJet P1505 firmware file

Install driver, foomatic XML files, and extra files:

​    $ su			OR	$ sudo make install

​    \# make install

(Optional) Configure hotplug (USB; HP LJ P1005/P1006/P1007/P1008/P1505):

​    \# make install-hotplug      OR      $ sudo make install-hotplug

(Optional) If you use CUPS, restart the spooler:

​    \# make cups			OR	$ sudo make cups

这个时候重新在浏览器中输入 https://你的树莓派ip:631 进入配置界面，删掉原来的打印机，重新配置下打印机，应该就可以使用了。





转载:

https://www.jianshu.com/p/14a063c98831