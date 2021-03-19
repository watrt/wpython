控制 NeoPixels
=====================

NeoPixels，又名WS2812 LEDs，是串行连接的、单独可寻址的全色LED，且可将其红、绿、蓝组件设置在0至255之间。
这些LED需要精准计时来实现控制，于是NeoPixels模块应运而生。

按照下列步骤创建一个NeoPixel对象::

    >>> import machine, neopixel
    >>> np = neopixel.NeoPixel(machine.Pin(4), 8)

这在具有GPIO4上配置了8像素的 NeoPixel 灯带。您可调整"4"（引脚编号）和"8"（像素数）来配合您的设置。

使用以下指令设置像素的颜色::

    >>> np[0] = (255, 0, 0) # set to red, full brightness 设置为红色，全亮
    >>> np[1] = (0, 128, 0) # set to green, half brightness 设置为绿色，半亮
    >>> np[2] = (0, 0, 64)  # set to blue, quarter brightness 设置为蓝色，1/4亮度

对于颜色超过3种的LED而言，例如RGBW或RGBY，NeoPixel类传递一个``bpp`` 参数。
设置RGBW Pixel的NeoPixel 对象，请遵循以下步骤::

    >>> import machine, neopixel
    >>> np = neopixel.NeoPixel(machine.Pin(4), 8, bpp=4)


在4-bpp模式下，请记得使用4-元组来设置颜色，而非3-元组。例如，使用以下指令来设置前三个像素::

    >>> np[0] = (255, 0, 0, 128) # Orange in an RGBY Setup
    >>> np[1] = (0, 255, 0, 128) # Yellow-green in an RGBY Setup
    >>> np[2] = (0, 0, 255, 128) # Green-blue in an RGBY Setup

然后使用 ``write()`` 方法将颜色输出到LED::

    >>> np.write()

下面的demo函数在LED上进行了展示::

    import time

    def demo(np):
        n = np.n

        # cycle
        for i in range(4 * n):
            for j in range(n):
                np[j] = (0, 0, 0)
            np[i % n] = (255, 255, 255)
            np.write()
            time.sleep_ms(25)

        # bounce
        for i in range(4 * n):
            for j in range(n):
                np[j] = (0, 0, 128)
            if (i // n) % 2 == 0:
                np[i % n] = (0, 0, 0)
            else:
                np[n - 1 - (i % n)] = (0, 0, 0)
            np.write()
            time.sleep_ms(60)

        # fade in/out
        for i in range(0, 4 * 256, 8):
            for j in range(n):
                if (i // 256) % 2 == 0:
                    val = i & 0xff
                else:
                    val = 255 - (i & 0xff)
                np[j] = (val, 0, 0)
            np.write()

        # clear
        for i in range(n):
            np[i] = (0, 0, 0)
        np.write()

使用以下指令执行::

    >>> demo(np)
