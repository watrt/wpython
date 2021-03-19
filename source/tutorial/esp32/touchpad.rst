触摸传感器
============================

触摸传感器系统建立在一个基底上，此基底在保护平面下带有电极和相关连接。当用户触碰表面时，即触发电容变化，并产生一个表示触摸是否有效的二进制信号

A touch-sensor system is built on a substrate which carries electrodes and relevant connections under a
protective flat surface. When a user touches the surface, the capacitance variation is triggered
and a binary signal is generated to indicate whether the touch is valid.


ESP32可提供多达10个触摸GPIO。
这10个触摸GPIO为 0, 2, 4, 12, 13, 14, 15, 27, 32, 33

    >>> import machine, Pin
    >>> tc = machine.TouchPad(Pin(35))


当引脚为悬空时:

    >>> tc.read()
    1129

当用户触碰表面时:

    >>> tc.read()
    89

todo:

TouchPad.config()
