模拟数字转换
============================


ESP32有ADC，可用于读取模拟电压并将之转换为数值。
您可使用以下指令创建这样的ADC引脚对象

    >>> import machine, Pin
    >>> adc = machine.ADC(Pin(35))

然后使用以下方法读取值::

    >>> adc.read()
    58

由``read()`` 函数返回的值介于0（0.0伏特）至4095（1.0伏特）之间。请注意：此输入仅能承受1.0伏的电压，因此您必须使用分压通路来测量大于1.0伏的电压

ADC 在引脚 32~39 上可用。

GPIO37-38 一般连接到一个电容，用于 ADC_PRE_AMP。
