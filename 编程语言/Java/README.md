# JAVA

## 概述

**Java**(英式发音[ˈʤɑːvə],美式发音[ˈʤɑvə])是一种广泛使用的计算机[编程语言](https://zh.wikipedia.org/wiki/程式設計語言)，拥有[跨平台](https://zh.wikipedia.org/wiki/跨平台)、[面向对象](https://zh.wikipedia.org/wiki/物件導向)、[泛型编程](https://zh.wikipedia.org/wiki/泛型程式設計)的特性，广泛应用于企业级Web应用开发和移动应用开发。

任职于[Sun微系统](https://zh.wikipedia.org/wiki/昇陽電腦)的[李翰庭](https://zh.wikipedia.org/w/index.php?title=李翰庭&action=edit&redlink=1)等人于1990年代初开发Java语言的雏形，最初被命名为Oak，目标设置在[家用电器](https://zh.wikipedia.org/wiki/家用电器)等小型系统的[编程语言](https://zh.wikipedia.org/wiki/程式語言)，应用在[电视机](https://zh.wikipedia.org/wiki/电视机)、[电话](https://zh.wikipedia.org/wiki/电话)、[闹钟](https://zh.wikipedia.org/wiki/闹钟)、[烤面包机](https://zh.wikipedia.org/wiki/烤面包机)等家用电器的控制和通信。由于这些[智能化](https://zh.wikipedia.org/w/index.php?title=智能化&action=edit&redlink=1)家电的市场需求没有预期的高，[太阳计算机系统](https://zh.wikipedia.org/wiki/太阳计算机系统)（[Sun公司](https://zh.wikipedia.org/wiki/Sun公司)）放弃了该项计划。随着1990年代[互联网](https://zh.wikipedia.org/wiki/網際網路)的发展，[Sun公司](https://zh.wikipedia.org/wiki/Sun公司)看见Oak在[互联网](https://zh.wikipedia.org/wiki/網際網路)上应用的前景，于是改造了Oak，于1995年5月以Java的名称正式发布。Java伴随着互联网的迅猛发展而发展，逐渐成为重要的网络编程语言。

Java编程语言的风格十分接近[C++](https://zh.wikipedia.org/wiki/C%2B%2B)语言。继承了[C++](https://zh.wikipedia.org/wiki/C%2B%2B)语言面向对象技术的核心，舍弃了容易引起错误的[指针](https://zh.wikipedia.org/wiki/指针_(信息学))，以[引用](https://zh.wikipedia.org/wiki/參照)取代；移除了C++中的[运算符重载](https://zh.wikipedia.org/wiki/运算符重载)和[多重继承](https://zh.wikipedia.org/wiki/继承_(计算机科学))特性，用[接口](https://zh.wikipedia.org/wiki/接口_(Java))取代；增加[垃圾回收器](https://zh.wikipedia.org/wiki/垃圾回收_(計算機科學))功能。在Java SE 1.5版本中引入了[泛型编程](https://zh.wikipedia.org/wiki/泛型)、[类型安全](https://zh.wikipedia.org/wiki/类型安全)的枚举、不定长参数和自动装/拆箱特性。Sun微系统对Java语言的解释是：“Java编程语言是个简单、面向对象、分布式、解释性、健壮、安全、与系统无关、可移植、高性能、多线程和动态的语言”。

Java不同于一般的[编译语言](https://zh.wikipedia.org/wiki/編譯語言)或[解释型语言](https://zh.wikipedia.org/wiki/直譯語言)。它首先将源代码编译成[字节码](https://zh.wikipedia.org/wiki/字节码)，再依赖各种不同平台上的虚拟机来解释执行字节码，从而具有“[一次编写，到处运行](https://zh.wikipedia.org/wiki/一次编写，到处运行)”的跨平台特性。在早期JVM中，这在一定程度上降低了Java程序的运行效率。但在J2SE1.4.2发布后，Java的执行速度有了大幅提升。

与传统类型不同，Sun公司在推出Java时就将其作为开放的技术。全球的Java开发公司被要求所设计的Java软件必须相互兼容。“Java语言靠群体的力量而非公司的力量”是Sun公司的口号之一，并获得了广大软件开发商的认同。这与[微软](https://zh.wikipedia.org/wiki/微软)公司所倡导的注重精英和封闭式的模式完全不同，此外，[微软公司](https://zh.wikipedia.org/wiki/微软公司)后来推出了与之竞争的[.NET平台](https://zh.wikipedia.org/wiki/.NET_Framework)以及模仿Java的[C#](https://zh.wikipedia.org/wiki/C＃)语言。后来Sun公司被[甲骨文公司](https://zh.wikipedia.org/wiki/甲骨文公司)并购，Java也随之成为甲骨文公司的产品。

现时，移动[操作系统](https://zh.wikipedia.org/wiki/作業系統)[Android](https://zh.wikipedia.org/wiki/Android)大部分的代码采用Java[编程语言](https://zh.wikipedia.org/wiki/程式設計語言)编程。

## 用途

- 桌面GUI应用程序：

Java通过抽象窗口工具包（AWT），Swing和JavaFX等多种方式提供GUI开发。虽然AWT包含许多预先构建的组件，如菜单，按钮，列表以及众多第三方组件，但Swing（一个GUI小部件工具包）还提供某些高级组件，如树，表格，滚动窗格，选项卡式面板和列表。JavaFX是一组图形和媒体包，提供了Swing互操作性，3D图形功能和自包含的部署模型，可以快速编写Java小应用程序和应用程序的脚本。[[21\]](https://zh.wikipedia.org/wiki/Java#cite_note-25)

- 移动应用程序：

Java Platform，Micro Edition（Java ME或J2ME）是一个跨平台框架，用于构建可在所有Java支持的设备（包括功能手机和智能手机）上运行的应用程序。此外，最受欢迎的移动操作系统之一的Android应用程序通常使用Android软件开发工具包（SDK）或其他环境在Java中编写脚本。

- 嵌入式系统：

从微型芯片到专用计算机的嵌入式系统是执行专门任务的大型机电系统的组件。诸如SIM卡，蓝光光盘播放器，公用事业仪表和电视机等多种设备都使用嵌入式Java技术。据甲骨文公司称，100％的蓝光光盘播放器和1.25亿台电视设备都采用Java技术。

- Web应用程序：

Java通过Servlets，Struts或JSP提供对Web应用程序的支持。编程语言提供的简单编程和更高的安全性使得大量政府应用程序可用于基于Java的健康，社会安全，教育和保险。Java也可以使用Broadleaf等开源电子商务平台开发电子商务Web应用程序。

- 分布式系统：

Java更多时候用于构建大型分布式应用, 基于java的分布式生态非常丰富, 各种成熟的基础组件帮助java开发者迅速搭建起分布式系统. 比较著名的分布式框架有: spring cloud, dubbo, zookeeper

- Web服务器和应用程序服务器：

今天的Java生态系统包含多个Java Web服务器和应用程序服务器。虽然Apache Tomcat，Simple，Jo !, Rimfaxe Web服务器（RWS）和Project Jigsaw占据了Web服务器空间，但WebLogic，WebSphere和Jboss EAP在商业应用服务器领域占据重要地位[[22\]](https://zh.wikipedia.org/wiki/Java#cite_note-26)。

- 企业应用程序：

Java企业版（Java EE）是一种流行的平台，为脚本和运行企业软件（包括网络应用程序和Web服务）提供API和运行时环境。甲骨文宣称Java在97％的企业计算机上运行。Java中更高的性能保证和更快的计算能力导致像Murex这样的高频交易系统被编入脚本中。它也是各种银行应用程序的中枢，它们将Java从前端用户端运行到后端服务器端。

- 科学应用：

Java是许多软件开发人员用于编写涉及科学计算和数学运算的应用程序的选择。这些程序通常被认为是快速和安全的，具有更高的便携性和低维护性。像MATLAB这样的应用程序使用Java来交互用户界面和作为核心系统的一部分。