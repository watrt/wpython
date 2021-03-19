Network - TCP sockets
=====================

大多数互联网的基本组件即为TCP socket。这些sockets提供了相连的网络设备间的可靠字节流。本教程的此章节将演示在不同情况下如何使用TCP socket。

Star Wars Asciimation
---------------------

最简单的事情就是从网上下载数据。在这种情况下，我们将使用由blinkenlights.nl网站提供的Star Wars Asciimation设备。
其使用端口23上的远程登录协议来向任何与之相连的设备传输流数据。使用方法很简单，因为并不需要您进行鉴别（给定一个用户名或密码），您可直接开始下载数据。


首先，确保我们的socket模块可用::

    >>> import socket

然后获取服务端的IP地址::

    >>> addr_info = socket.getaddrinfo("towel.blinkenlights.nl", 23)

实际上，``getaddrinfo`` 函数会返回一个地址列表，且每个地址的信息都超出我们的需要。
我们只想获取首个有效地址和服务端的IP地址和端口。使用以下指令来完成这一步骤。

    >>> addr = addr_info[0][-1]

若您在提示框中输入 ``addr_info``和 ``addr`` ，您看到的就是其中包含的信息。

使用IP地址，我们可以制作一个socket并连接到服务端::

    >>> s = socket.socket()
    >>> s.connect(addr)

我们已建立连接，所以可下载和显示数据::

    >>> while True:
    ...     data = s.recv(500)
    ...     print(str(data, 'utf8'), end='')
    ...
 
当此循环执行时，应开始动了（使用ctrl-C可中断）。

若您想尝试一下，您也可以使用常规Python在个人电脑上运行这一代码。


HTTP GET 请求
----------------

下一个示例演示如何下载网页。HTTP端口使用端口80，下载前，您首先需要发送"GET"请求。在该请求中，您应指定检索的页面。

我们来定义一个可下载和打印URL的函数::

    def http_get(url):
        _, _, host, path = url.split('/', 3)
        addr = socket.getaddrinfo(host, 80)[0][-1]
        s = socket.socket()
        s.connect(addr)
        s.send(bytes('GET /%s HTTP/1.0\r\nHost: %s\r\n\r\n' % (path, host), 'utf8'))
        while True:
            data = s.recv(100)
            if data:
                print(str(data, 'utf8'), end='')
            else:
                break
        s.close()

确保您在运行此函数之前输入socket模块。然后您可尝试::

    >>> http_get('http://micropython.org/ks/test.html')

此指令应检索页面并将HTML打印到控制台。

简单的 HTTP 服务器
------------------

下列代码创建一个简易HTTP服务器，该服务器只提供包含所有GPIO引脚的状态的列表的页面：::

    import machine
    pins = [machine.Pin(i, machine.Pin.IN) for i in (0, 2, 4, 5, 12, 13, 14, 15)]

    html = """<!DOCTYPE html>
    <html>
        <head> <title>ESP32 Pins</title> </head>
        <body> <h1>ESP32 Pins</h1>
            <table border="1"> <tr><th>Pin</th><th>Value</th></tr> %s </table>
        </body>
    </html>
    """

    import socket
    addr = socket.getaddrinfo('0.0.0.0', 80)[0][-1]

    s = socket.socket()
    s.bind(addr)
    s.listen(1)

    print('listening on', addr)

    while True:
        cl, addr = s.accept()
        print('client connected from', addr)
        cl_file = cl.makefile('rwb', 0)
        while True:
            line = cl_file.readline()
            if not line or line == b'\r\n':
                break
        rows = ['<tr><td>%s</td><td>%d</td></tr>' % (str(p), p.value()) for p in pins]
        response = html % '\n'.join(rows)
        cl.send(response)
        cl.close()
