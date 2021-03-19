1-wire 设备
==========================

1-wire总线是仅将单一线用于通信的串行总线（地线和电源线除外）。
DS18B20温度传感器是一个广为人知的1-wire设备，此处我们会显示如何使用1-wire协议读取数据。

运行以下代码，您至少需有DS18S20或DS18B20温度传感器，且传感器的数据线与GPIO12相连接。
您必须给传感器供电，并在数据引脚与电源引脚间连接一个4.7k欧姆的电阻::

    import time
    import machine
    import onewire, ds18x20

    # the device is on GPIO12
    dat = machine.Pin(12)

    # create the onewire object
    ds = ds18x20.DS18X20(onewire.OneWire(dat))

    # scan for devices on the bus
    roms = ds.scan()
    print('found devices:', roms)

    # loop 10 times and print all temperatures
    for i in range(10):
        print('temperatures:', end=' ')
        ds.convert_temp()
        time.sleep_ms(750)
        for rom in roms:
            print(ds.read_temp(rom), end=' ')
        print()

注意：您必须执行convert_temp()函数以启动温度读数，并在读取数值前等待至少750ms。