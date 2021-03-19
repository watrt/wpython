.. _intro:

开始在ESP32上使用MicroPython
===============================================

使用MicroPython可以有效利用您的ESP32板，反之亦然，ESP32芯片是使用MicroPython的绝佳平台。
本教程将引导您创建MicroPython、获取指示符、使用WebREPL、连接到网络并使用因特网通信、使用硬件外设并控制一些外设。

我们开始吧！


必要条件
------------

首先，您需要一块带有ESP32芯片的电路板。MicroPython软件支持ESP32芯片本身，所以任何板子都可运行。
板子的主要特征是其FlashROM空间的大小、GPIO引脚与外界连接的方式以及其是否包括一个内置的USB串口转换器以便在您的电脑上使用UART。

FlashROM空间的最低要求为1Mbyte，大多数ESP32模块都为4MByte。

教程中将使用芯片名称命名引脚名称（例如GPIO0），如此一来，在您的特定的板上查找对应的引脚就很轻松了。

板的供电方式
------------------

若您的板上有USB连接器，则此板很可能是通过USB与电脑连接来供电。否则您就需要直接供电。了解关于板的更多信息，请查阅相应的参考资料。

获取固件
--------------------

首先您需要下载最新的MicroPython固件二进制文件以载入您的ESP32设备。您可从MicroPython 下载页面 <http://micropython.org/download#esp32>`_下载。


下载固件
----------------------

您下载固件（编译代码）后，就需要将其下载到您的ESP32设备中。
主要步骤有二：首先您需要使您的设备处于引导载入模式，然后您需要通过固件来复制。具体步骤取决于特定的板，详细信息请您查阅相关文件。

若您的板有USB连接器、一个USB串口芯片，且DTR和RTS引脚以某种特殊方式连接，则配置固件极为简单，其所有步骤都可自动完成。
满足上述条件的板包括Adafruit Feather HUZZAH、SingTown ESP32 board、sparkfun esp32 things boards。

为获得最佳效果，建议您在安装新版MicroPython固件前先将设备的FlashROM完全清除。

目前我们仅支持esptool.py通过固件进行复制。您可在以下网址获得这一工具：`<https://github.com/espressif/esptool/>`__, 或使用pip安装：

    ``pip install esptool``

其他烧写程序也可运行，所以请尽情尝试或查阅您的板的相关文档以获得使用建议。

esptool.py可以使用此指令，清除FlashROM。

    ``esptool.py --chip esp32 --port /dev/tty.SLAB_USBtoUART erase_flash``

然后使用以下指令烧写新的固件：

    ``esptool.py --chip esp32 --port /dev/tty.SLAB_USBtoUART write_flash -z 0x1000 firmware.bin``

你也可以用下面的命令设置 忽略MTDI引脚(GPIO12)在启动时对VDD_SDIO的影响。

    ``espefuse.py --port /dev/tty.SLAB_USBtoUART set_flash_voltage 3.3V``

见 Strapping Pin: :ref:`pin` 更多关于引脚的信息

您可能需要将"端口"设置更改为与您的个人电脑相关的内容。若您在烧写时出现错误，则可能需要降低波特率（例如降低至115200）。
固件的文件名也应与您的文件相匹配。


若上述指令运行无误，那么就已成功烧写在您的板上了！

串口交互式命令行
-------------

在设备上安装固件后，您就可以通过与USB串口转换器连接的UART0（GPIO1=TX, GPIO3=RX）访问REPL（Python 交互式命令行），
这取决于您的板子。波特率为115200。本教程将在下一章节对提示进行详细介绍。

WiFi
----

本教程将在后续章节详细介绍WiFi配置问题。


安装时的故障
-------------------------------------

注意 ：需要重写

If you experience problems during flashing or with running firmware immediately
after it, here are troubleshooting recommendations:

* Be aware of and try to exclude hardware problems. There are 2 common problems:
  bad power source quality and worn-out/defective FlashROM. Speaking of power
  source, not just raw amperage is important, but also low ripple and noise/EMI
  in general. If you experience issues with self-made or wall-wart style power
  supply, try USB power from a computer. Unearthed power supplies are also known
  to cause problems as they source of increased EMI (electromagnetic interference)
  - at the very least, and may lead to electrical devices breakdown. So, you are
  advised to avoid using unearthed power connections when working with ESP32
  and other boards. In regard to FlashROM hardware problems, there are independent
  (not related to MicroPython in any way) reports
  `(e.g.) <http://internetofhomethings.com/homethings/?p=538>`_
  that on some ESP32 modules, FlashROM can be programmed as little as 20 times
  before programming errors occur. This is *much* less than 100,000 programming
  cycles cited for FlashROM chips of a type used with ESP32 by reputable
  vendors, which points to either production rejects, or second-hand worn-out
  flash chips to be used on some (apparently cheap) modules/boards. You may want
  to use your best judgement about source, price, documentation, warranty,
  post-sales support for the modules/boards you purchase.

* The flashing instructions above use flashing speed of 460800 baud, which is
  good compromise between speed and stability. However, depending on your
  module/board, USB-UART convertor, cables, host OS, etc., the above baud
  rate may be too high and lead to errors. Try a more common 115200 baud
  rate instead in such cases.

* If lower baud rate didn't help, you may want to try older version of
  esptool.py, which had a different programming algorithm::

    pip install esptool==1.0.1

  This version doesn't support ``--flash_size=detect`` option, so you will
  need to specify FlashROM size explicitly (in megabits). It also requires
  Python 2.7, so you may need to use ``pip2`` instead of ``pip`` in the
  command above.

* The ``--flash_size`` option in the commands above is mandatory. Omitting
  it will lead to a corrupted firmware.

* To catch incorrect flash content (e.g. from a defective sector on a chip),
  add ``--verify`` switch to the commands above.

* Additionally, you can check the firmware integrity from a MicroPython REPL
  prompt (assuming you were able to flash it and ``--verify`` option doesn't
  report errors)::

    import esp
    esp.check_fw()

  If the last output value is True, the firmware is OK. Otherwise, it's
  corrupted and need to be reflashed correctly.

* If you experience any issues with another flashing application (not
  esptool.py), try esptool.py, it is a generally accepted flashing
  application in the ESP32 community.

* If you still experience problems with even flashing the firmware, please
  refer to esptool.py project page, https://github.com/espressif/esptool
  for additional documentation and bug tracker where you can report problems.

* If you are able to flash firmware, but ``--verify`` option or
  ``esp.check_fw()`` return errors even after multiple retries, you
  may have a defective FlashROM chip, as explained above.
