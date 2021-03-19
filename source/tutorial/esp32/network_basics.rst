网络基础
==============

网络模块用于配置WiFi连接。有两处WiFi接口，一个用于 station （当ESP32连接到路由器时），一个用于热点（access point）（用于其他设备与ESP32连接）。
使用以下指令创建这些对象的实例::

    >>> import network
    >>> sta_if = network.WLAN(network.STA_IF)
    >>> ap_if = network.WLAN(network.AP_IF)

您可使用以下指令检查接口是否有效::

    >>> sta_if.active()
    False
    >>> ap_if.active()
    True

您可使用以下指令检查接口的网络设置::

    >>> ap_if.ifconfig()
    ('192.168.4.1', '255.255.255.0', '192.168.4.1', '8.8.8.8')

返回值为：IP地址、网络掩码、网关、DNS。

WiFi的配置
-------------------------

全新安装时，ESP32配置为热点模式，因此AP_IF接口有效，STA_IF接口无效。您可使用STA_IF接口将此模块配置为与您自己的网络连接。
Upon a fresh install the ESP32 is configured in access point mode, so the
AP_IF interface is active and the STA_IF interface is inactive.  You can
configure the module to connect to your own network using the STA_IF interface.

首先激活station接口::

    >>> sta_if.active(True)

然后连接到您的WiFi网络::

    >>> sta_if.connect('<your ESSID>', '<your password>')

使用以下指令检查连接是否建立::

    >>> sta_if.isconnected()

建立后，您可检查IP地址::

    >>> sta_if.ifconfig()
    ('192.168.0.2', '255.255.255.0', '192.168.0.1', '8.8.8.8')

若您不再需要热点接口，可禁用该接口::

    >>> ap_if.active(False)

下面的函数可以自动运行来自动连接到WiFi网络，放入boot.py可以自启动::
您可运行此函数，便可自动连接到您的WiFi网络::

    def do_connect():
        import network
        sta_if = network.WLAN(network.STA_IF)
        if not sta_if.isconnected():
            print('connecting to network...')
            sta_if.active(True)
            sta_if.connect('<essid>', '<password>')
            while not sta_if.isconnected():
                pass
        print('network config:', sta_if.ifconfig())

Sockets
-------

WiFi建立后，访问网络的方式即使用sockets。一个Socket代表网络设备上的端点，两个socket连接在一起时，即可进行通信。
网络协议建立在socket的上层，例如电子邮件（SMTP）、网络（HTTP）、远程登录协议、安全外壳协议等。
每个协议都被分配了一个特定端口，此端口即一个数字。给定一个IP地址和一个端口数字，您即可连接到一个远程设备并开始与之通信。
本教程的下一章节讨论如何使用socket来执行一些常见的网络任务
