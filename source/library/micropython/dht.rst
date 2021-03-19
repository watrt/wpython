.. _dht:
:mod:`dht` --- dht模块
=========================================

dht模块中提供了dht系列温湿度传感器读取相关的函数。


类 DHT22
---------

构建对象
~~~~~~~
.. class:: DHT22(pin)

创建一个与引脚pin相连的DHT22传感器对象。

  - ``pin``:引脚

  .. admonition:: 支持引脚
      :class: attention

      * ESP32:GPIO0/2/4/5/16/17/18/19/21/22/23/25/26/27
      * 掌控板: P0/1/8/9/13/14/15/16

示例::

  from machine import Pin
  import dht

  d = dht.DHT22(Pin(Pin.P0))

方法
~~~~~~~

.. method:: DHT22.humidity()

读取并返回传感器的湿度值。 

示例::

  d.measure()
  print(d.humidity())

.. method:: DHT22.temperature()

读取并返回传感器的温度值。  

示例::

  d.measure()
  print(d.temperature())





类 DHT11
---------

与DHT22()函数类似，不再赘述。
