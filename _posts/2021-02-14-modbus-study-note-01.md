---
layout: post
title: Modbus Protocol学习笔记 
tags: [Modbus, RS232/485]
excerpt_separator: <!--more-->
---

Modbus Protocol学习过程小结

<!--more-->

Modbus协议是应用于电子控制器上的一种通讯语言，通过此协议，控制器之间经过网络和其他设备之间可以进行通讯。Modbus协议具有标准，开放，可以支持多种电器接口，数据帧格式简单紧凑，数据传输量大，实时性好等特点，在工业控制行业得到了广泛应用。Modbus RTU和Modbus ACSCII主要用于串行通讯领域，Modbus TCP主要用于以太网通讯。

### 专业术语
**校验码**：校验码是由前面的数据通过某种算法得出的，用以检验该组数据的正确性。代码作为数据在向计算机或其它设备进行输入时，容易产生输入错误，为了减少这种输入错误，编码专家发明了各种校验检错方法，并依据这些方法设置了校验码。




Reference: [工业控制系统安全之——Modbus学习笔记](https://www.freebuf.com/articles/ics-articles/148637.html)

