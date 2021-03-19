:mod:`uctypes` --  以结构化方式访问二进制数据
========================================================

.. module:: uctypes
   :synopsis:  以结构化方式访问二进制数据

该模块为MicroPython实现“外部数据接口”。它背后的想法类似于CPython的 ``ctypes`` 模块，但实际的API是不同的，流线型和小尺寸优化。
该模块的基本思想是定义具有与C语言允许的大致相同功率的数据结构布局，然后使用熟悉的点语法访问它以引用子字段。

.. warning::

  ``uctypes`` 模块允许访问机器的任意内存地址（包括I / O和控制寄存器）。不小心使用它可能会导致崩溃，数据丢失，甚至硬件故障。

.. seealso::

    :mod:`ustruct` 模块: 用于访问二进制数据结构的标准Python方法（不能很好地扩展到大型和复杂的结构）。


用法示例::

    import uctypes

    # Example 1: Subset of ELF file header
    # https://wikipedia.org/wiki/Executable_and_Linkable_Format#File_header
    ELF_HEADER = {
        "EI_MAG": (0x0 | uctypes.ARRAY, 4 | uctypes.UINT8),
        "EI_DATA": 0x5 | uctypes.UINT8,
        "e_machine": 0x12 | uctypes.UINT16,
    }

    # "f" is an ELF file opened in binary mode
    buf = f.read(uctypes.sizeof(ELF_HEADER, uctypes.LITTLE_ENDIAN))
    header = uctypes.struct(uctypes.addressof(buf), ELF_HEADER, uctypes.LITTLE_ENDIAN)
    assert header.EI_MAG == b"\x7fELF"
    assert header.EI_DATA == 1, "Oops, wrong endianness. Could retry with uctypes.BIG_ENDIAN."
    print("machine:", hex(header.e_machine))


    # Example 2: In-memory data structure, with pointers
    COORD = {
        "x": 0 | uctypes.FLOAT32,
        "y": 4 | uctypes.FLOAT32,
    }

    STRUCT1 = {
        "data1": 0 | uctypes.UINT8,
        "data2": 4 | uctypes.UINT32,
        "ptr": (8 | uctypes.PTR, COORD),
    }

    # Suppose you have address of a structure of type STRUCT1 in "addr"
    # uctypes.NATIVE is optional (used by default)
    struct1 = uctypes.struct(addr, STRUCT1, uctypes.NATIVE)
    print("x:", struct1.ptr[0].x)


    # Example 3: Access to CPU registers. Subset of STM32F4xx WWDG block
    WWDG_LAYOUT = {
        "WWDG_CR": (0, {
            # BFUINT32 here means size of the WWDG_CR register
            "WDGA": 7 << uctypes.BF_POS | 1 << uctypes.BF_LEN | uctypes.BFUINT32,
            "T": 0 << uctypes.BF_POS | 7 << uctypes.BF_LEN | uctypes.BFUINT32,
        }),
        "WWDG_CFR": (4, {
            "EWI": 9 << uctypes.BF_POS | 1 << uctypes.BF_LEN | uctypes.BFUINT32,
            "WDGTB": 7 << uctypes.BF_POS | 2 << uctypes.BF_LEN | uctypes.BFUINT32,
            "W": 0 << uctypes.BF_POS | 7 << uctypes.BF_LEN | uctypes.BFUINT32,
        }),
    }

    WWDG = uctypes.struct(0x40002c00, WWDG_LAYOUT)

    WWDG.WWDG_CFR.WDGTB = 0b10
    WWDG.WWDG_CR.WDGA = 1
    print("Current counter:", WWDG.WWDG_CR.T)

定义结构布局
-------------------------

结构布局由“描述符”定义 - 一个Python字典，它将字段名称编码为键，以及将它们作为关联值访问它们所需的其他属性::

    {
        "field1": <properties>,
        "field2": <properties>,
        ...
    }

目前，``uctypes`` 需要明确规定每个字段的偏移量。从结构开始以字节为单位给出偏移量。

以下是各种字段类型的编码示例:

* 标量类型::

    "field_name": offset | uctypes.UINT32

  换句话说，该值是标量类型标识符，与结构起始处的字段偏移量（以字节为单位）进行或运算。

* 递归结构::

    "sub": (offset, {
        "b0": 0 | uctypes.UINT8,
        "b1": 1 | uctypes.UINT8,
    })

  即，值是2元组，其第一个元素是偏移量，第二个是结构描述符字典（注意：递归描述符中的偏移量与其定义的结构相关）。
  当然，递归结构不仅可以通过文字字典指定，还可以通过按名称引用结构描述符字典（前面定义）来指定。

* 原始类型的数组::

      "arr": (offset | uctypes.ARRAY, size | uctypes.UINT8),

  即，value是一个2元组，其第一个元素是ARRAY标志ORed与offset，第二个是标量元素类型ORed数组中的元素。

* 聚合类型的数组::

    "arr2": (offset | uctypes.ARRAY, size, {"b": 0 | uctypes.UINT8}),

  即，value是一个3元组，其第一个元素是ARRAY标志ORed与offset，第二个是数组中的元素数，第三个是元素类型的描述符。

* 指向原始类型的指针::

    "ptr": (offset | uctypes.PTR, uctypes.UINT8),

  即，value是一个2元组，其第一个元素是PTR标志，与偏移量进行OR运算，第二个元素是标量元素类型。

* 指向聚合类型的指针::

    "ptr2": (offset | uctypes.PTR, {"b": 0 | uctypes.UINT8}),

  ie值是一个2元组，其第一个元素是PTR标志ORed with offset，second是指向的类型的描述符。

* 位地址::

    "bitf0": offset | uctypes.BFUINT16 | lsbit << uctypes.BF_POS | bitsize << uctypes.BF_LEN,

ie value是一种包含给定位域的标量值（类型名称类似于标量类型，但带有前缀BF），ORed带有包含位域的标量值的偏移量，并进一步与位内的位值和位域内的位长度进行或运算。
标量值，分别通过BF_POS和BF_LEN位移位。位域位置从标量的最低有效位（具有0的位置）计数，并且是字段的最右位的数量（换句话说，它是标量需要向右移位的位数）提取位域）。

在上面的例子中，首先在偏移0处提取UINT16值（当访问硬件寄存器时，这个细节可能很重要，需要特定的访问大小和对齐），
然后是最右边的位是此UINT16的lsbit位的位域，以及length是bitsize bits，将被提取。
例如，如果lsbit为0且 bitsize为8，那么它将有效地访问UINT16的最低有效字节。

注意，位域操作独立于目标字节字节序，特别是上面的例子将在小端和大端结构中访问UINT16的最低有效字节。
但它取决于最低有效位被编号为0.某些目标可能在其原生ABI中使用不同的编号，但uctypes始终使用上述标准化编号。

模块内容
---------------

.. class:: struct(addr, descriptor, layout_type=NATIVE)

    基于内存中的结构地址，描述符（编码为字典）和布局类型（参见下文）来实例化“外部数据结构”对象。

.. data:: LITTLE_ENDIAN

    little-endian压缩结构的布局类型。（打包意味着每个字段占用描述符中定义的字节数，即对齐为1）。

.. data:: BIG_ENDIAN

    big-endian压缩结构的布局类型。

.. data:: NATIVE

    本机结构的布局类型 - 数据字节顺序和对齐符合运行MicroPython的系统的ABI。

.. function:: sizeof(struct, layout_type=NATIVE)

    以字节为单位返回数据结构的大小。的结构参数可以是一个类结构或特定实例化结构对象（或其聚集体字段）。

.. function:: addressof(obj)

    返回对象的地址。参数应该是字节，字节数组或其他支持缓冲区协议的对象（该缓冲区的地址实际上是返回的）。

.. function:: bytes_at(addr, size)

    以给定的地址和大小捕获内存作为bytes对象。由于bytes对象是不可变的，因此内存实际上是复制并复制到bytes对象中，因此如果内存内容稍后更改，则创建的对象将保留原始值。

.. function:: bytearray_at(addr, size)

    将给定地址和大小的内存捕获为bytearray对象。与上面的bytes_at（）函数不同，内存是通过引用捕获的，因此它也可以写入，并且您将在给定的内存地址访问当前值。

.. data:: UINT8
          INT8
          UINT16
          INT16
          UINT32
          INT32
          UINT64
          INT64

    结构描述符的整数类型。提供了8,16,32和64位类型的常量，包括有符号和无符号。

.. data:: FLOAT32
          FLOAT64

    结构描述符的浮点类型。

.. data:: VOID

    ``VOID`` 是一个别名 ``UINT8`` ，用于方便地定义C的void指针：。( ``uctypes.PTR`` , ``uctypes.VOID`` )

.. data:: PTR
          ARRAY

    输入指针和数组的常量。请注意，结构没有显式常量，它是隐式的：没有 ``PTR`` 或者 ``ARRAY`` 标志的聚合类型是结构。

结构描述符和实例化结构对象
---------------------------------------------------------

给定结构描述符字典及其布局类型，您可以使用 :class:`uctypes.struct()`  构造函数在给定的内存地址处实例化特定的结构实例。
内存地址通常来自以下来源:


* 访问裸机系统上的硬件寄存器时的预定义地址。在特定MCU / SoC的数据表中查找这些地址。
* 作为从调用某些FFI（外部函数接口）函数的返回值。

* 从 `uctypes.addressof()`,当您想要将参数传递给FFI函数时，或者，为了访问I / O的某些数据（例如，从文件或网络套接字读取的数据）。

结构对象
-----------------

结构对象允许使用标准点表示法访问各个字段：``my_struct.substruct1.field1`` 。
如果字段是标量类型，获取它将产生与字段中包含的值对应的原始值（Python整数或浮点数）。
标量字段也可以分配给。

如果字段是数组，则可以使用标准下标运算符访问其各个元素 ``[]`` - 包括读取和分配。

如果一个字段是一个指针，它可以使用 ``[0]`` 语法解除引用（对应于C  ``*`` 运算符，但也 ``[0]`` 适用于C）。还支持使用其他整数值（但是为0）订阅指针，其语义与C中相同。

总而言之，访问结构字段通常遵循C语法，除了指针取消引用，当您需要使用 ``[0]`` 运算符而不是 ``*`` 。

限制
-----------

1. 访问非标量字段会导致分配中间对象以表示它们。这意味着应特别注意布局在禁用内存分配时需要访问的结构（例如，来自中断）。建议如下:

  * 避免访问嵌套结构。例如，代替 ``mcu_registers.peripheral_a.register1`` 为每个外围设备定义单独的布局描述符，以便进行访问 ``peripheral_a.register1`` 。或者只缓存特定的外围设备: 如果寄存器由多个位域组成，则需要缓存对特定寄存器的引用: ``peripheral_a = mcu_registers.peripheral_areg_a = mcu_registers.peripheral_a.reg_a``

  * 避免使用其他非标量数据，例如数组。例如，而不是 peripheral_a.register[0]使用peripheral_a.register0。同样，另一种方法是缓存中间值，例如 ``register0 = peripheral_a.register[0]`` 

2. ``uctypes`` 模块支持的偏移范围有限。支持的确切范围被认为是实现细节，一般建议是将结构定义拆分为从几千字节到几十千字节的最大值。
在大多数情况下，无论如何这都是一种自然情况，例如，在一个结构中定义MCU的所有寄存器（扩展到32位地址空间）没有意义，而是通过外围模块定义外设模块。
在某些极端情况下，您可能需要人工分割几个部分的结构（例如，如果在中间访问具有多兆字节数组的本机数据结构，尽管这将是非常合成的情况）。