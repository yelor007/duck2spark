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

#. 设置好 Arduino IDE 后，加载 "DigisparkKeyboard " 的例子，并将 Sketch 源码替换成保存在 ``sketch.ino`` 中的源码。

要获得帮助，请执行 ``duck2spark.py -h``。

开始使用 DuckyScript
--------------------------------

Here's an introduction_ to DuckyScript

.. _introduction: http://usbrubberducky.com/?duckyscript#!duckyscript.md

使用 DuckEncoder 与 duck2spark 的附加提示
--------------------------------------------------------------------

* DuckEncoder 在编码 "GUI "或 "WINDOWS " 键时有一个问题，没有附加键。Windows 上常见的情况是 "GUI r " 等键的组合，但仅使用 "GUI " 会产生不正确的字符 ``e```` 输出。这个问题已经解决了`这里<https://github.com/hak5darren/USB-Rubber-Ducky/issues/51>`_https://github.com/hak5darren/USB-Rubber-Ducky/issues/51。由于希望会有一个补丁 duck2spark 不处理这个问题。事实上，在一个已经编码的脚本中无法区分 "GUI "键和 "e "键。Encoder.java 的补丁版本可以在这里找到 <https://github.com/mame82/USB-Rubber-Ducky/tree/GUI-Key-fix/Encoder/src>`https://github.com/mame82/USB-Rubber-Ducky/tree/GUI-Key-fix/Encoder/src。

* 在 DuckyScript 中使用长的延迟会导致大的有效载荷，因为超过 250 毫秒的延迟会被分割成多个延迟，每个延迟最多只有 250 毫秒。每个延迟都会在最终的有效载荷中消耗 2 个字节。由于 digispark 的内存有限，建议使用 ``duck2spark的`` 延迟选项。Duck2spark 依赖于 DigiKeyboard.delay()，在内存消耗方面更加友好。

* 在 DuckyScript 中使用 "PREPEAT <N>" 指令，会导致重复前一条指令的整个按键序列，从而在最终的有效载荷中消耗<N>倍的内存。同样，由于 Digispark 的内存不足，建议尽可能使用 ``duck2spark 的`` 循环选项。使用 "repeat 500 " 打印出一个 10 个字符的字符串 500 次，会产生一个 10000 字节的有效载荷，这对 Digispark 来说太大了。用一个10个字符的 DuckyScript 编码一个 DuckyScript 只消耗 20 个字节，可以与 ``duck2spark.py -l 500`` 结合起来，实现 500 次重复，而不需要进一步消耗内存。
