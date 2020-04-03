======================================
 duck2spark by MaMe82 (Marcus Mengs)
======================================

该项目提供了一个python脚本，能够将 DuckEncoder 生成的有效载荷转换为 Arduino Sketch 源的 DigiSpark。
这个脚本解决了两个问题：

* 现有的解决方案和教程在 DigiSpark 上模拟 RuberDucky 的解决方案和教程都是由于对非美国语言的键盘布局支持不足。这个问题可以通过 "外包 " 给 DuckEncoder_ 来解决，它支持多种键盘布局。
* 由于 DigiKeyboard.print() 和 DigiKeyboard.println() 的解决方案受限于 DigiSparks 的 RAM 限制（小于512字节），因此使用DigiKeyboard.print() 和 DigiKeyboard.println() 的解决方案会受到字符串大小的限制。这可以通过将有效载荷存储在 FLASH 内存中来解决。

附加功能
-------------------

* 支持 DuckyScript 的 "DELAY "和 "REPEAT "命令。
* 初始延迟选项，用于处理由于目标上的驱动程序初始化时间不足而导致的缺失按键。
* 可选择重复执行有效载荷（计数循环、单次运行、无休止运行）。
* 当有效载荷执行完成时，可选择闪烁状态指示灯（默认为亮起，无尽循环除外）。

.. _DuckEncoder: https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Encoder/encoder.jar

项目文件
=============

* duck2spark.py - 主脚本
* README.rst - 这个文件
* example.sh - 通过运行 Duck2spark.py 后运行 DuckEncoder 建立一个有效载荷的脚本示例（编码器.jar必须存在）。
* example.ducky - RubberDucky 脚本，包含 example.sh 所使用的测试用例。

配置要求
============

* `Arduino IDE`_编译并将生成的 Sketch 上传到 DigiSpark
* 必须配置 Arduino IDE 来对 DigiSpark 进行编程，请遵循本指南_。
* 一个、两个或多个DigiSparks;-)
* DuckEncoder_从 DuckyScript 生成一个原始的有效载荷，如果你想远离Java，请使用 "我的 DuckEncoder 的 python 端口<https://github.com/mame82/duckencoder.py>`_。
* Python 2的安装

.. _Arduino IDE: https://www.arduino.cc/en/main/software
.. _guide: https://digistump.com/wiki/digispark/tutorials/connecting
.. _DuckEncoder: https://github.com/hak5darren/USB-Rubber-Ducky/blob/master/Encoder/encoder.jar


使用方法
=====

#. 生成一个 DuckyScript ``test.duck``你想用它作为输出：:

	echo "STRING Hello World" > test.duck

#. 用 DuckEncoder 编译脚本，用你的键盘布局（例子中的de）或使用 `my python port <https://github.com/mame82/duckencoder.py>`_::

	java -jar encoder.jar -i test.duck -o raw.bin -l de

#. 使用 duck2spark.py 将其转换为 Arduino Sketch（选项为单次运行，2秒启动延迟）：:
	
	duck2spark.py -i raw.bin -l 1 -f 2000 -o sketch.ino

#. 设置好 Arduino IDE 后，加载 "DigisparkKeyboard " 的例子，并将 Sketch 源码替换成保存在 `sketch.ino`` 中的源码。

要获得帮助，请执行 ``duck2spark.py -h````。

开始使用 DuckyScript
--------------------------------

Here's an introduction_ to DuckyScript

.. _introduction: http://usbrubberducky.com/?duckyscript#!duckyscript.md

Additional Hints on using DuckEncoder in conjunction with duck2spark
--------------------------------------------------------------------

* DuckEncoder has an issue encoding "GUI" or "WINDOWS" key without an additional key. The common scenario on Windows is a key combination like "GUI r", but using "GUI" alone would produce the incorrect character ``e`` as output. The issue is adressed `here <https://github.com/hak5darren/USB-Rubber-Ducky/issues/51>`_. As there hopefully will be a patch duck2spark doesn't handle this issue. In fact it isn't possible to distinguish between "GUI" key and "e" key in an already encoded script. A patched version of Encoder.java could be found `here <https://github.com/mame82/USB-Rubber-Ducky/tree/GUI-Key-fix/Encoder/src>`_.

* Using long delays in a DuckyScript results in big payloads, as delays longer than 250 milliseconds are split up into multiple delays, with a maximum of 250 milliseconds each. Each of these delays consumes 2 bytes in the final payload. As the memory of digispark is far more limited, it is suggested to use ``duck2spark's`` delay options instead. Duck2spark relies on DigiKeyboard.delay() and is more friendly in terms of memory consumption.

* Using the "PREPEAT <N>" instruction in DuckyScript results in repeating the whole key sequence of the former command and thus consumes <N> times as much memory in the final payload. Again, as Digispark is short on memory, it is suggested to use ``duck2spark's`` loop option whenever possible. Printing out a 10 character string 500 times by using "REPEAT 500" results in a payload 10000 bytes in size, which is to large for Digispark. Encoding a DuckyScript with a single 10 character string consumes only 20 bytes and could be combined with ``duck2spark.py -l 500`` to achieve a 500 times repetition without further memory consumption.
