温度和湿度
========================

DHT（数字湿度&温度）感应器为使用电容式湿度传感器和温度传感器来测量周围的空气的低成本数字式传感器。
其特点为处理模拟数字转换的芯片且提供单线接口。新的传感器额外提供了一个I2C接口。

DHT11（蓝）和DHT22（白）传感器提供相同的单线接口，但是DHT22需要一个特殊的对象，因为其计算更为复杂。
DHT22在湿度和温度读数上都有一个小数的分辨率。DHT11的温度和湿度都是整数。

自定义1-wire协议与Dallas 单线不同，此协议用于从感应器中获取测量结果。有效数据包括一个湿度值、一个温度值和一个校验和。

使用单线接口，应根据数据引脚构建对象::

    >>> import dht
    >>> import machine
    >>> d = dht.DHT11(machine.Pin(4))

    >>> import dht
    >>> import machine
    >>> d = dht.DHT22(machine.Pin(4))

使用以下指令测量和读取值::

    >>> d.measure()
    >>> d.temperature()
    >>> d.humidity()

``temperature()`` 返回的值单位为摄氏度，而自 ``humidity()`` 返回的值为相对湿度的百分比。

The DHT11只能以每秒1次的频率调用，DHT22为结果准确可达每秒2次。感应器准确度会随时间推移降低。每个感应器都支持不同的运算范围。详细信息请参考产品数据手册。 

在单线模式中，4个引脚中只有3个启用；在I2C模式下，4个引脚全部启用。旧版感应器可能仍有4个引脚，尽管旧版并不支持I2C。第3个引脚不连接。

引脚配置：

Sensor without I2C in 1-wire mode (eg. DHT11, DHT22, AM2301, AM2302):

    1=VDD, 2=Data, 3=NC, 4=GND

Sensor with I2C in 1-wire mode (eg. DHT12, AM2320, AM2321, AM2322):

    1=VDD, 2=Data, 3=GND, 4=GND

Sensor with I2C in I2C mode (eg. DHT12, AM2320, AM2321, AM2322):

    1=VDD, 2=SDA, 3=GND, 4=SCL

您应将上拉电阻用于数据、SDA和SCL引脚。

要在向后兼容的单线模式下运行新版的I2C感应器，您必须将引脚3和4与GND连接。这会禁用I2C接口。

DHT22 sensors are now sold under the name AM2302 and are otherwise identical.
