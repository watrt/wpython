wPython帮助文档
----------------

.. toctree::
   :maxdepth: 2
   :caption: 快速入门
   :hidden:

   tutorial/index.rst

.. toctree::
   :maxdepth: 2
   :caption: MicroPython类库
   :hidden:

   library/pythonStd/index.rst
   library/micropython/index.rst
   library/wPython/index.rst


=========================================   ======================================================
 :ref:`Python标准库<pythonStd>`               兼容CPython,内含Python内建函数、常用module
 :ref:`MicroPython类库<microPythonModu>`      MicroPython的ESP32硬件控制层的模块
 :ref:`wPython类库<wpythonModu>`		      wPython的ESP32硬件控制层的模块
=========================================   ======================================================


您可以通过 ``help()`` 发现可用的内置库，在REPL中输入以下内容来导入::

    >>> help('modules')


除了本文档中描述的内置库之外，在 :term:`micropython-lib`  中还可以找到来自Python标准库的更多模块以及对它的进一步微Python扩展。


.. toctree::
   :caption: MicroPython 语法
   :maxdepth: 1

   reference/index.rst

有关MicroPython特定语言功能的语言参考信息


索引
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
