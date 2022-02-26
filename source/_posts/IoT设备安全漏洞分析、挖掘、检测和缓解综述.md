---
title: >-
  【翻译】A Survey of Security Vulnerability Analysis,Discovery, Detection, and
  Mitigation on IoT Devices
tags:
  - 论文翻译
abbrlink: 62497
date: '2021-06-17 18:55'
---
在前面：
完整阅读完的第一篇论文。这篇出自2019年MDPI的Future Internet，虽然最近两年出了更新的成果和趋势，但也本文也没有太过时，算是讲全了IoT研究的发展和角度。
翻译和自己阅读还是挺不一样的，发现了自己语文水平蒟蒻，连啥时候用逗号都把握不住... DeepL 和 Google Translate 还是各有千秋。

> 文章由本人首发于安全客：[IoT设备安全漏洞分析、挖掘、检测和缓解综述 - 安全客，安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/244711)

<!--more-->

# IoT设备安全漏洞分析、挖掘、检测和缓解综述

来源：http://netsec.ccert.edu.cn/chs/publications/futureinternet20-iot

<!-- vscode-markdown-toc -->

* 1. [摘要](#)
* 2. [介绍](#-1)
* 3. [背景](#-1)

  * 3.1.[IoT架构](#IoT)
  * 3.2.[设备构成](#-1)
  * 3.3.[攻击面](#-1)
    * 3.3.1.[硬件层](#-1)
    * 3.3.2.[软件层](#-1)
    * 3.3.3.[协议接口层](#-1)
* 4. [漏洞分析、挖掘、检测和缓解](#-1)

  * 4.1.[漏洞分析的基础框架研究](#-1)
  * 4.2.[漏洞挖掘技术研究](#-1)
    * 4.2.1.[动态分析方法](#-1)
    * 4.2.2.[静态分析方法](#-1)
  * 4.3.[漏洞检测研究](#-1)
    * 4.3.1.[网络扫描](#-1)
  * 4.4.[相似性检测](#-1)
    * 4.4.1.[源代码相似性检测](#-1)
    * 4.4.2.[二进制代码相似性检测](#-1)
  * 4.5.[漏洞缓解研究](#-1)
    * 4.5.1.[自动化生成补丁](#-1)
    * 4.5.2.[访问控制](#-1)
* 5. [讨论](#-1)

  * 5.1.[评估](#-1)
  * 5.2.[难点](#-1)
    * 5.2.1.[（1）设备的复杂性和异构性（Complexity and Heterogeneity of Device）](#1ComplexityandHeterogeneityofDevice)
    * 5.2.2.[（2）设备资源的限制（Limitations of device resources）](#2Limitationsofdeviceresources)
    * 5.2.3.[（3）闭源措施（Closed-Source Measures）](#3Closed-SourceMeasures)
  * 5.3.[机会](#-1)
    * 5.3.1.[（1）AI技术的应用](#1AI)
    * 5.3.2.[（2）对第三方和开源代码的依赖](#2)
    * 5.3.3.[（3）外围设备系统的开发](#3)
* 6. [研究方向](#-1)
* 7. [结论](#-1)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->

<!-- /vscode-markdown-toc -->

<!--more-->

## 1. `<a name=''></a>`摘要

随着IoT产业的兴盛，多种多样的IoT设备开始迅速发展，智能家居、智能穿戴、智能制造、智能汽车等一系列和生活相关的领域得到大量使用。随之而来的是层出不穷的IoT设备上的漏洞。安全问题的增加会对用户的隐私和财产带来严重威胁。本文首先介绍了研究背景，包括IoT架构、设备构成和攻击面。我们回顾了有关IoT设备漏洞挖掘、检测、缓解技术最前沿的研究，随后通过评估指出了现在面临的困难和机会，最后预测和讨论了IoT设备上漏洞分析技术的研究方向。

## 2. `<a name='-1'></a>`介绍

IoT正在成为最广泛和实用的在线平台。它将大量的传感器和控制器联网，帮助人们实现和万物之间的无缝通信。IoT正在成为互联网未来的关键，特别是近几年，随着IoT产业的兴盛，多种多样的IoT设备开始迅速发展，全球活跃的IoT设备已经达到70亿[^1]，它们在智能家居、智能穿戴、智能制造、智能汽车等一系列和生活相关的领域广泛使用，我们认为这会极大提升我们的生活质量。同时，IoT设备的安全问题经常发生，也难以解决。惠普的报告表明70%的IoT设备包含安全漏洞，平均每台设备有25个漏洞[^2]。攻击者利用这些漏洞控制设备，进行一系列非法的活动。最著名的例子是在2016年Mirai病毒控制了成百上千台IoT设备，用这些设备建造了一个僵尸网络发动TB级的dos攻击，攻击目标包括DNS服务提供商Dyn，这次攻击造成了严重的后果，包括导致美国部分的网络瘫痪[^3]。总的来说，随着IoT设备的广泛使用、安全漏洞的增加会对用户的隐私和安全，甚至人类的生活和财产带来严重威胁。面对频繁的攻击，IoT安全研究变得越来越流行。在美国Auto-ID于1999年第一次提出“物联网”的概念[^4]后，安全研究者们投身于IoT行业，研究安全架构和通信的标准[^5,^6]。随后产生了许多关于IoT安全的问题[^7-^12]。Zhang等人[^13]、Mahmoud等人[^14]指出了面临的问题和研究方向，于是，研究者开始在IoT安全领域使用传统安全研究的方法[^15]。随着AI的发展，将机器学习和深度学习应用于IoT安全上的概念开始产生[^16]。Alrawi等人[^17]，系统的总结了智能家居中设备、手机应用、云端和通信的IoT攻击点。Xie等人[^18]总结及了检测IoT漏洞的技术。最近，Zheng等人[^19]发表了IoT漏洞挖掘的技术概论。在上述的两篇论文中，漏洞挖掘和漏洞检测的界限较为模糊。在本文中，漏洞挖掘的技术即挖掘未知的漏洞，漏洞检测指针对已知漏洞的检测。通过以上调查，我们发现现有的研究关注于IoT安全问题，缺少分析技术，其次，漏洞分析技术的重点在漏洞挖掘和检测，缺少漏洞缓解技术，综上，现有的IoT安全的技术总结还不够全面。为了解决前面提到的问题，我们希望对以下个方面做出一些贡献：

- 首先，我们把重点从IoT架构转移到IoT设备；其次，精简IoT设备安全技术的分类；另外我们总结了现有的研究，包括漏洞分析的基础框架、挖掘未知的漏洞、检测已知的漏洞、漏洞缓解
- 我们评估了现有的关于IoT设备漏洞分析的研究。除此之外，我们深入分析了阻碍安全研究技术发展的原因，指出了面临的困难和机会
- 我们回顾了技术发展的背景，并为相关研究者提出了未来研究的方向

这篇论文的结构如下：第二节描述了IoT安全的背景，介绍了IoT设备的架构、设备构成和攻击面；第三节回顾了已有的IoT设备安全研究，包括漏洞分析、挖掘、检测和缓解；第四节基于对漏洞分析技术的评估，总结了目前遇到的问题和机会；第五节提出了未来研究的热点方向；第六节对论文进行了总结

## 3. `<a name='-1'></a>`背景

### 3.1. `<a name='IoT'></a>`IoT架构

随着网络的快速发展，越来越多的家用和工业设备开始联网，给我们带来了多元化的生活。物联网架构主要有两个发展方向：消费者层面和产业层面
在消费者层面，如果我们将它们按照应用场景划分，有几类设备类型例如工业制造、智能家居、智能医疗和智能汽车，其中智能家居的发展相对成熟。互联网巨头——三星、谷歌、苹果和小米占据大部分市场份额，同时，它们也发布了SmartThings[^20]、Google Weave[^21]、Apple HomeKit[^22]、HomeAssistant[^23] 和 XiaoMi IoT[^24]等物联网平台。通过调查这些平台，我们发现绝大多数IoT遵循“设备<->云端<->用户”这一架构（如图1）。
![20210613122500](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210613122500.png)
智能设备大部分都放在家中，它们和云服务器进行通信，然后直接或间接通过WiFi[^25]、ZigBee[^26]、蓝牙[^27]或其他协议接入网络。它们上传传感器收集的数据，接收发送给执行单元的控制命令。IoT架构不仅依赖供应商的云，同时也依赖第三方的云，它们相互支持，并为各种功能提供多样化的服务。用户可以通过手机或者电脑连接云查看状态并下载数据。对于一些简单场景例如可穿戴设备，“设备<->用户”架构更加实用。

对于产业层面，IoT架构延续了IT的做法，通过服务器集中管理用户和设备的交互（如图2），区别是设备首先通过OT和PLC进行通信。因此产业中的设备等同于PLC和“传感器+执行器”。安全研究的重点在于PLC。“设备<->用户”架构也存在于产业物联网：管理员用配置软件控制设备。虽然面向用户的工业终端例如智能仪表也尝试了云的模式，但基于安全考虑没有得到广泛使用
![20210617205408](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210617205408.png)

### 3.2. `<a name='-1'></a>`设备构成

不论是汽车制造中大而复杂的机器，亦或穿戴设备里小巧智能的手环，它们都包含了相对固定的零件例如芯片、闪存、固件等。它们主要包含硬件和软件部分

1. 硬件

- 逻辑芯片：对于复杂的设备来说，它们需要多个逻辑芯片或cpu来运行内置的操作系统；对于简单的嵌入式设备或许只需要一个微处理器来运行程序
- 内存：为系统和程序运行提供空间，大小从KB到GB不等
- 闪存：储存IoT设备固件。部分设备的bootloader也存放在闪存
- 网络模块。IoT设备和传统嵌入式设备的区别就是前者连接了网络。他们通常采用无线技术连接到互联网，如AP
- 串行调试接口：IoT设备需要与外部进行通信，以便调试。串行调试接口可以让开发人员发送和接收命令。最常见的接口是通用异步接收器/发送器UART

2. 软件

- Bootloader：在IoT设备系统启动前，它初始化了硬件设备，将固件加载到引导设备。它使系统的软件和硬件环境达到合适的状态
- 固件：固件包括了操作系统、文件系统和一系列服务程序。IoT设备上的安全研究通常从固件分析开始

### 3.3. `<a name='-1'></a>`攻击面

针对IoT设备的攻击面不仅包括传统软件安全领域，因为它们的特殊结构和功能，也包括了新的攻击领域。根据IoT架构和设备构成，攻击面可以分为三个层面（图3）

- 图3![20210613163604](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210613163604.png)

#### 3.3.1. `<a name='-1'></a>`硬件层

硬件层面的攻击不同于传统的安全领域，它主要包括三个角度：不安全的调试接口、未保护的闪存芯片、硬件敏感信息的泄露

1. 不安全的调试接口当IoT设备被制造的时候，调试接口比如UART会被留在电路板上以便维修。如果它缺少身份验证或者仅有弱身份验证，攻击者就可以通过接口来获得高权限，对固件进行修改或者替换。调试接口在IoT安全检查中排在第一位
2. 未保护的闪存芯片因为闪存通常用来存储固件，因此也成为了关注的重点。如果芯片没有读写保护，安全研究者就可以通过读取固件来分析或者修改固件，来绕过接口的身份验证
3. 硬件敏感信息的泄露
   硬件电路的密封性并不好，诸如声音和电量消耗的硬件信息泄露可以造成侧信道攻击[^28-^31]，攻击者可以由此获得重要的信息，比如密钥。

#### 3.3.2. `<a name='-1'></a>`软件层

软件层面的攻击对应着设备构成中bootloader和固件的软件部分，它主要包括以下五个方面：不安全的bootloader、不安全的操作系统、固件敏感信息泄露、不安全的应用服务、不正确的配置策略

1. 不安全的bootloader因为bootloader是一段在设备运行后加载的代码，因此是一个容易被忽略的攻击点。它的功能是初始化并加载固件，因此当问题出现时它的危险程度很高。例如checkm8这个Boot ROM 漏洞被称为是iphone、ipad、apple TV和 apple watch上的史诗级漏洞
2. 不安全的操作系统由于研发周期短和轻量化的需求，IoT设备的操作系统内核是定制的，版本也不经常更新，这导致了大量的缓冲区溢出问题，如提权等。除此之外，设备使用了各种各样的传感器和通信模块，包括内核中大量的驱动。例如，Marvell WiFi芯片驱动找到了多个漏洞，包括 CVE-2019-14901, CVE-2019-14897 和CVE-2019-14896，它们导致了内核中基于栈或堆的缓冲区溢出。这也是攻击面中重要的一部分
3. 固件敏感信息泄露IoT设备的本地存储通常使用轻量化的存储方案，开发者通常忽略了它的安全性，并使用了明文或只是进行了简单的加密，这很容易导致敏感数据的泄露
4. 不安全的应用服务应用服务开发缺少安全标准。为了加快产品的开发，通常直接编译、使用了简单、不安全的应用代码，因此引入未知的漏洞。IoT安全研究者们已经发现了大量开发时产生的应用漏洞，包括出于未知原因留下的后门
5. 不正确的配置策略
   为了方便管理IoT设备，ssh、telnet等服务是默认的开启，这样会造成配置问题。默认配置下的弱验证策略使攻击者容易获得设备的权限。例如，Telestar Digital GmbH 的物联网收音机可通过未经验证的telnet服务器被远程攻击者劫持利用[^34]，这些漏洞已经被CVE-2019-13473[^35]和CVE-2019-13474[^36]收录。

#### 3.3.3. `<a name='-1'></a>`协议接口层

协议接口层的攻击包括了通信和API，它涉及到用户侧直接控制和由云端间接控制的设备，以及以上两种通信过程中的信息保护问题，并不涉及到协议的安全。例如，IoT通信协议的滥用和AR-Ddos[^37]攻击正是通过IoT通信协议CoAP[^38]、SSDP[^39]和SNMP执行的，它的目标不是IoT设备，但是它也是IoT安全一个重要的研究方向。协议接口层的攻击主要包括一下三个角度：不安全的远程管理接口、数据传输过程中的信息泄露、弱身份验证

1. 不安全的远程管理接口为了方便管理，IoT设备使用http服务之类的远程管理方式，这带来了诸多漏洞，例如sql注入、XSS和远程执行漏洞等
2. 数据传输过程中的信息泄露IoT通信协议使用了弱加密算法或者根本不进行加密，导致敏感信息泄露。例如论文*Passwords in the Air*[^40]中提到的，当IoT设备接入网络时，WiFi密码以明文传输
3. 弱身份验证
   由于安全需要，管理IoT设备需要身份验证绑定，于是产生了一个新的攻击面。攻击者可以绕过身份验证，重复绑定然后获得用户的信息，论文*Phantom Device Attack*[^41]在这个攻击面上找到了四种攻击方法

## 4. `<a name='-1'></a>`漏洞分析、挖掘、检测和缓解

现阶段，对IoT安全没有精确的分类，此外，安全研究的核心在于漏洞，因此我们把重点放在设备的漏洞。在研究周期中，研究分为三个阶段：挖掘、检测和缓解。因为IoT安全的特殊性导致不可能有用于分析的标准接口，因此，针对物联网的基础分析框架的研究内容也很有价值。为了审视现有的物联网安全技术，我们从以下四个角度进行总结：（1）漏洞分析的基础框架的研究，使用了固件模拟来帮助分析IoT安全问题；（2）漏洞挖掘技术研究，主要针对挖掘IoT设备中的未知漏洞的手段；（3）漏洞检测，研究基于现有漏洞的特征来检测已知漏洞；（4）漏洞缓解技术的研究，研究了自动修复漏洞或者加入访问控制来限制恶意行为。另外，这节总结的IoT漏洞分析技术需要一系列前提条件，例如固件提取[^42]，因此我们标注了技术要求，但并不总结它们。

### 4.1. `<a name='-1'></a>`漏洞分析的基础框架研究

为了解决人们对物联网安全的担忧，即使没有源代码或者硬件资料，对固件二进制文件的准确分析也极为重要[^43]。然而，由于缺少专门的基础框架，IoT安全领域的漏洞分析收到了阻碍。例如，动态分析依赖于在可控的环境（通常是设备化的模拟器[^43]）中执行程序，因此基础框架主要提供了通过半仿真和全仿真的功能。它可以进行复杂的动态分析以支持IoT安全研究。**技术要求：**能够获得IoT设备固件对于缺少专门用于分析固件，特别是用于动态分析的工具，Avatar[^43]提出了一个结合了在模拟器上的模拟执行模式和在真实设备上的实际执行模式的框架来分析固件。当固件在模拟模式下运行时，当发生I/O时，Avatar将操作转发到设备，设备执行操作后将结果传给模拟器，以便模拟器继续运行。它有效地应对了特定外围设备缺少源码和文档的问题。随后，Prospect[^44]和Surrogate[^45]也提出了类似的动态分析框架。四年后，Avatar开发团队开发出了Avatar2[^46]，允许安全研究者在不同动态分析框架、调试器、模拟器和实际设备之间交互操作，除此之外，作者还展示了如何使用Avatar2来记录设备的执行流。Chen等人[^47]提出了用于linux设备的Firmadyne，首先使用软件进行系统仿真，然后采用扫描和探测等动态分析方法来挖掘漏洞。以上框架的模拟功能都基于QEMU[^48]。对于不易模拟的传感器操作，半模拟框架[^43-^46]在执行表1的固件指令时，通过软件代理的方式引导对物理硬件的I/O操作

- **表1**是漏洞分析基础框架的总结。
  Semi-simulation指的是框架需要真实的设备来进行I/O访问![20210613214401](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210613214401.png)

### 4.2. `<a name='-1'></a>`漏洞挖掘技术研究

随着IoT设备漏洞的增加和攻击趋势的上升，安全研究者在设备的漏洞挖掘上花费越来越多时间。这一届表述了漏洞挖掘的技术，包括动态和静态分析。通过学习传统的软件安全分析技术，我们发现动态分析主要包括fuzz[^49]和污点检测[^50]，静态分析主要包括符号执行[^51]、污点分析、数据流分析[^50]。

#### 4.2.1. `<a name='-1'></a>`动态分析方法

动态分析方法需要仿真固件工具来进行动态调试，或者在物理设备上进行片上调试，来获得反馈信息。主要通过fuzz来得到漏洞的触发点
**技术要求：**在IoT设备上进行动态调试的能力
在卡片安全研究领域，Alimi等人[^52]使用了一个通用的算法生成测试样本，对手机卡和银行卡进行fuzz。对于现在一些内置web服务器的智能卡片，Kamel等人[^53]发现了一些基于HTTP协议生成方式的bug，并据此对web服务器进行了fuzz。在汽车安全领域，Koscher[^54]和Lee[^55]通过修改发送给CAN的数据包来改变汽车的状态[^56]，来对汽车的智能系统进行fuzz。
因为IoT固件提取的困难，IoTFuzzer[^57]通过从用户侧捕获崩溃信息来避免了这个问题。首先，它在手机应用中的交互协议代码里插桩，然后修改从桩获取的数据，最后根据心跳包（Heartbeat packet）和响应来判断fuzz的效率。因为设备很难直接调试，研究者开始结合仿真技术来发现漏洞。Costin等人[^58]实现了应用动态固件分析技术来进行嵌入式固件镜像中web接口的漏洞自动化挖掘框架。最近，Srivastava等人的目标不再仅限于web接口，他们展示了FirmFuzz[^59]，一个独立于设备的、针对Linux固件镜像的自动化仿真和动态分析框架。Zheng等人[^60]提出了Firm-AFL，它是第一个针对IoT固件的高吞吐量灰盒fuzzer，他们扩展了AFL[^61]，把它变成了现在IoT领域最流行的fuzzer。对于fuzz的研究，Muench等人分析了传统的IoT设备异常状态检测方法的通用性，然后实现了一个基于Avatar[^43]和[^63]的系统，另外，他们比较了黑盒fuzzer在不同配置下的吞吐量，包括本机执行（直接向硬件输入）、部分仿真（仅将硬件请求重定向到硬件）、全仿真[^60]，这是一项对漏洞分析技术性能的评估。上述提到的动态分析技术发现的漏洞类型详见表2，主要是内存问题例如缓冲区溢出和空指针解引用，由于对web接口进行了研究，也有一些web服务器漏洞例如XSS、SQL注入。

#### 4.2.2. `<a name='-1'></a>`静态分析方法

静态分析不需要运行固件就可以挖掘漏洞。寻找bug的过程可以理解程序代码，因此它具有可扩展性。**技术要求：**能够获得IoT设备的固件静态分析步骤如下：（1）提取固件，（2）逆向固件中的程序，（3）通过人工审计找到安全问题。在学术界，研究人员主要探索通过自动化静态分析方法来找到漏洞。在表2，我们总结了静态分析的研究目标、细分的技术和发现的漏洞种类。Costin等人[^64]首先自动化分析了大量嵌入式设备的固件。他们自动解压运行了固件，然后使用模糊哈希来匹配固件中的弱密钥。FIE[^65]基于KLEE[^66]构建了嵌入式设备的符号执行引擎。它制定了内存规范、中断规范、芯片规范，以发现固件中违反自定义安全规范的问题。Firmalice[^67]也是一个基于符号执行的框架，它通过后门的输入确定性来找到身份验证绕过漏洞。SainT[^68]和DTaint[^69]提出了静态污点分析方法，分别在设备软件或二进制代码上挖掘漏洞。

- **表2**：
  BO=缓冲区溢出漏洞；NPD=空指针解引用；CL=命令注入；CSRF=跨站请求伪造![20210614224801](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210614224801.png)

### 4.3. `<a name='-1'></a>`漏洞检测研究

前面提到的动态和静态分析技术也可以应用于检测已知的漏洞。在大规模的检测场景，动态分析依赖于使用架构专用的工具来执行程序，静态分析检测已知漏洞的方式和发现0day漏洞的方式相同，但提高了性能和耗时。目前，研究者主要使用以下两种方法：网络扫描和代码相似性检测。

#### 4.3.1. `<a name='-1'></a>`网络扫描

网络扫描通过向在线的IoT设备服务发送带有payload的探测包来检测已知漏洞。网络扫描在安全领域更为通用，随着IoT安全的发展，产生了网络扫描物联网设备的议题。
**技术要求：**了解漏洞的有关信息，如POC
Cui等人[^70]扫描了网络上存在的嵌入式设备，发现了一系列设备含有弱密码和其他漏洞。在2013年后，搜索引擎例如Shodan[^71]和Censys[^72]以及Zoomeye[^73]产生了，它们能够识别和检测弱密码、后门、和已知的漏洞。然而，仅靠外部扫描只能发现一小部分漏洞，而且，未授权扫描联网设备也存在道德问题。因此，网络扫描漏洞常常在内网和实验室中进行。网络扫描的优点在于从服务层进行检测并不需要考虑设备的结构，并且它高效快速，适合大规模测试。现有的商用漏洞检测系统大多基于这种方法。

### 4.4. `<a name='-1'></a>`相似性检测

由于IoT设备中存在大量没有修复过的已知漏洞，安全研究员们提出软件代码相似性检测方法来检测已知漏洞。在现阶段，相似性检测的研究主要针对传统软件安全领域，然后通过跨架构逐步支持物联网设备，而不存在专门研究IoT固件相似性检测的论文。如图4，相似性检测的基本思路是从代码中提取原有特征，如字符串、指令序列、基本块、语法树和函数调用图等，接着，通过算法测量特征的相似度，最终确定相应的代码片段中是否存在漏洞。
![图4.相似性检测结构](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210615165124.png)

**技术要求：**获得IoT设备固件

#### 4.4.1. `<a name='-1'></a>`源代码相似性检测

对于在源代码上检测已知漏洞， CP-Miner[^74]提出了基于token的方法，使了用词法分析器来产生token序列，然后搜索重复的token序列来衡量相似性。ReDeBug[^75]提出了一个可扩展的方法，它结合代码补丁来确定修复前的漏洞代码的特征，并且可以识别未打补丁的代码。然而，以上基于源代码的方法并没有应用在IoT上，在绝大多数情况下，安全研究员们无法获得固件的源代码。

#### 4.4.2. `<a name='-1'></a>`二进制代码相似性检测

对于在二进制代码上检测已知漏洞，研究者主要面临的问题是，不同的编译器代码生成算法、编译优化选项以及不同指令集导致难以检测相似性。N-Grams[^67]和N-Perms[^77]是早期的漏洞搜索方法[^78]。Karim等人使用内存中的二进制片段或代码来匹配算法，由于没有对代码语义进行了解，这种方法难以应对不同编译级别带来的指令重排序问题。为了提升匹配的准确性，Tracelet-based[^80]提出将代码重构为可执行序列，然后使用求解器来处理程序约束和数据约束，因此它解决了指令重排序的问题。此外，TEDEM[^81]采用符号简化二进制程序，并通过树编辑距离作为基本块来判断代码相似性，它甚至可以找到不同操作系统的漏洞。
由于一些相同的语义特征，导致难以表示两个二进制程序基本块特征相似性。研究者开始考虑使用CFG[^82]来描述程序的行为，因此可以通过图来进行相似性比较。BinDiff[^83]和Binslayer[^84]能够通过检测CFG相似度，来检查两个二进制程序的相似性，不过它们并不是专门为漏洞检测设计的。通过比较两个完全不同的二进制文件的CFG，还是难以发现跨平台的漏洞片段。Egele等人[^85]提出了Blanket Execution 并指出，基于静态分析的二进制语义相似性研究容易受编译链和编译优化级别的影响。因此他们建议提取程序动态运行时的特征，来应对造成CFG改变的影响。BinHunt[^86]和iBinHunt[^87]使用了符号执行和理论证明，来检查基本块之间的语义等价性，并找出哪些语义有所不同。
然而不同IoT设备的固件差异很大，包括多种架构如MIPS、ARM、PPC、x86等，它们的操作码、寄存器名称和内存寻址方式都有差别，因此，以上提到的方法难以应用于大规模的跨架构代码漏洞检测。直到最近两三年，研究人员开始研究二进制代码基础上的跨架构代码相似性检测[^88-^90]。Multi-MH[^88]是第一个基于二进制代码的跨架构代码相似性检测方法。首先，将二进制代码转换为中间代码，然后使用特定的输入来测试程序，并根据I/O的行为来捕获基本块的语义，最后根据捕获的CFG来检测漏洞。然而它在处理大量函数时的性能开销过大。DiscovRE[^89]通过图匹配算法检查一组函数对的CFG是否相似，并通过预筛选来加快CFG匹配过程。然而它的预筛选过程并不可靠且会漏报过多漏洞。BinGo[^90]通过引入选择性内联相关库函数以及用户定义的用于跨平台代码搜索的函数来捕捉完整功能语义。然而它并不是特别为IoT设备设计的。Genius[^91]使用机器学习的传统方式，从CFG学习高层特征表征。另外，它将图嵌入[^92]编码为一个高位数字特征向量，然后使用图匹配算法测量目标函数和一组二进制函数的相似性，这可以有效的提升性能和可扩展性。Xu等人[^93]首先提出了基于深度学习的跨架构二进制代码相似性检测方法，使用了神经网络模型的图嵌入技术。在跨版本代码相似性检测，αDiff[^94]迈出了重要的一步。它基于DNN模型，提取了三个语义特征，包括函数、功能间和模块间的特征，来进行检测。Gao等人提出了VulSeeker[^95]和VulSeeker-Pro[^96]。这些漏洞搜索方法都通过与深度学习结合，来提高检测的准确性，后面提到的两种方法被证实比目前其他的（例如Gemini[^93]）方法准确度都高。

### 4.5. `<a name='-1'></a>`漏洞缓解研究

在漏洞挖掘和检测的基础上，漏洞缓解措施也是行业关注的一个研究问题。根据公开的文献研究，主要研究热点是自动化生成补丁和访问控制。前者旨在修复漏洞，后者研究如何限制恶意行为。

#### 4.5.1. `<a name='-1'></a>`自动化生成补丁

这节所说的自动化生成补丁技术并不专指IoT领域，而是一个传统安全领域的扩展。漏洞修复通常由开发团队在源代码上完成。在获得外部漏洞报告后，他们通过漏洞触发条件和分析漏洞机制来评估漏洞。自动生成补丁可以自动修复软件错误，而不需要开发者人工判断、理解、修正。[^97]
**技术要求：**获得和升级IoT设备固件
软件工程领域的研究者们提出，通过学习正确的c语言[^97,^98]、Java[^99]和其它源代码级别中的正确代码可以自动生成补丁，这一想法取得了初步可行成果。另一种想法是改变程序的形式而不改变它的功能。GenProg[^100]使用了遗传编程的扩展形式来演化程序变体，该变体保留了所需功能但不易受到给定缺陷的影响，然而由于突变操作的随机性，它会生成无意义的补丁。因此Kim等人[^101]提出了基于模式的自动化程序修复（PAR）来解决上述的问题。在安卓平台上，Zhang等人[^102]提出了AdaptKpatch，一个自适应内核修补程序框架和LuaKpatch，它将一个类型安全的动态语言引擎插入内核来执行补丁。这两个方案解决了安卓平台的补丁链过长、碎片化和平台生态布局不匹配、细分修复不及时的问题。然而它们没有考虑解决在跨CPU架构自动进行热修复的问题，他们依旧需要基于知识和经验进行手工编写。DARPA的CGC[^103]引领了在二进制代码级别上的自动化防御方法，然而主要采用的依旧是通用的防御方法，例如二进制代码加固、边界检查和指针修复[^106]

#### 4.5.2. `<a name='-1'></a>`访问控制

访问控制方法是通过管理IoT设备的用户侧或平台许可，来阻止或结束攻击者的恶意行为。
**技术要求：**用户侧或云端的可扩展性
Fernandes等人[^107]首先深入研究了IoT平台的安全例如SmartThings，他们发现，由于功能粗粒度，大量的应用获得了多余的权限，
智能家居中的许多设备获得了过多的权限，模糊的权限管理，导致了许多针对IoT设备的攻击以及隐私泄露。密歇根大学的研究者们想出了一系列办法来解决这些问题。2016年，他们提出了Flowfence[^108]，一个基于数据流来保护隐私泄露的系统，它将程序分为两部分：（1）一系列负责操作沙箱中敏感数据的隔离的模块。（2）不操作敏感数据，但通过污点跟踪不透明的句柄将隔离的模块链接在一起来协调执行的代码，其中涉及到的数据的数据只能在沙箱内被解引用。于是在2017年，他们又提出了基于上下分信息的ContextIoT[^109]，它可以帮助用户提升访问控制的有效性，通过识别敏感操作上下文标识和保证运行时的上下文完整性，来抵御攻击者进行危险操作。2018年，一个针对智能家居的基于风险控制的模型Tyche[^110]被提出，它建立了访问控制列表（ACCLs），在源代码层面解决访问权限过多的问题。Smartauth[^111]和FACT[^112]同样基于ACCLs，然而他们采取了不同的方法来建立ACCLs。Smartauth通过NLP识别的文件和APP源代码建立ACCLs，FACT在设备开发的阶段就建立了ACCLs。

<!--↑-->

## 5. `<a name='-1'></a>`讨论

在前面几节，我们深入探索了目前关于IoT漏洞分析技术的研究。在这一节，我们首先评估了漏洞分析技术，其次，通过评估指出现阶段研究的难点，最后提出了应对这些难点的技术机会。

### 5.1. `<a name='-1'></a>`评估

在表3，我们从5个角度进行评估，包括攻击面、技术要求、支持的架构、支持的操作系统和是否与AI结合。在漏洞分析过程中，研究者需要仿真、调试接口、网络流量等方面的技术支持。例如，IoTFuzzer[^57]使用了把目标转移到APP的外围系统分析方法。这种方法优点是能更好的避免架构的复杂性，	缺点是粗粒度的崩溃信息阻碍了对漏洞的进一步分析。从这些角度的的评估能够更容易的对技术难题和未来的发展趋势进行分析。

![20210616193125](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210616193125.png)

![20210616193206](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210616193206.png)

### 5.2. `<a name='-1'></a>`难点

前面的评估反映出当今IoT设备上的漏洞分析存在的难题。如表4中展示出的，不同技术领域对研究的影响不同，至于IoT设备漏洞分析需要的技术，现存的难题如下：

- **表4：**这张表总结了难点和机会的影响范围，影响的范围包括四种：IoT分析的基础框架（T1）、漏洞挖掘技术（T2）、漏洞检测技术（T3）、漏洞缓解技术（T4）。√=难点或机会和这个领域的技术有关![20210616202548](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210616202548.png)

#### 5.2.1. `<a name='1ComplexityandHeterogeneityofDevice'></a>`（1）设备的复杂性和异构性

这个问题一直都是IoT设备漏洞分析技术的最大难点，IoT设备的差异性比PC和手机都大。它使用了多种CPU架构如ARM、MIPS、x86，以及不同的操作系统如Linux、Windows和安卓，它通常使用定制化的固件和内存使用，这使得难以直接在IoT领域应用行业上的自动化检测、挖掘漏洞技术。IoT设备的复杂性使得静态和动态调试更加难以进行。我们发现现阶段主要选择路由器等基于arm的Linux设备来作为研究对象。相似性检测方面的研究扩展到了跨架构的场景[^88-^91,^93-^96]。其他研究并没有遇到这个问题。

#### 5.2.2. `<a name='2Limitationsofdeviceresources'></a>`（2）设备资源的限制

由于产品轻量化的需求，IoT设备大多运行在精简的操作系统，或者甚至只在微控制器上运行一个程序。上述原因造成了设备资源有限的特点。对于IoT设备安全测试来说，通过向目标部署相关分析模块，来实现对运行程序外围的监控分析并不容易。安全研究者并不能使用传统的安全分析手段和工具，他们需要重构分析平台。另外，由于设备硬件的计算能力有限，动态分析的性能下降了。最近几年，研究者开发了一套仿真系统，在基础架构[^43-^47]和漏洞挖掘[^56,^60,^64]领域应对了这个问题。然而这个问题还没有完全解决，且会是一个长期的难题。

#### 5.2.3. `<a name='3Closed-SourceMeasures'></a>`（3）闭源措施

对于常规的软件，我们可以在源代码或二进制程序层面挖掘或检测漏洞，对于IoT设备制造商，由于他们的闭源措施，这些方法并不能使用。代码审计，例如[^3.3.2]节的"源代码层面的相似性分析"不再适用于IoT漏洞分析。他们甚至对固件进行加密、加强对串行调试接口的身份验证，并且认为这样会更加安全。例如Dlink-882(867,878)、360 clear robots最新的固件都被加密了。因此，基于源代码、固件和调试接口的漏洞分析变得越来越困难。通过之前的评估，我们发现近两年的漏洞挖掘和检测技术已经绕开了调试接口[^59,^60]和固件，然而又出现了信息不完整等问题。

### 5.3. `<a name='-1'></a>`机会

IoT的特点不仅给漏洞分析带来了难题，同时也带来了新的机会。

#### 5.3.1. `<a name='1AI'></a>`（1）AI技术的应用

在最近几年，AI和IoT两种新技术已经出现了结合，促进社会进入了AIoT的时代。AI技术的发展也给IoT安全带来了新的措施和解决方案。如今，出现了运用AI进行访问控制[^111]和相似性检测[^89-^91,^93-^96]的研究，随着IoT和AI的发展，新的漏洞挖掘、检测和缓解技术不可避免的出现了，当AI应用在IoT设备上时，这同时也是AI对抗攻击和防御的新机会。例如，攻击者污染智能音箱的数据集，诱导它对某些问题回复一些负面信息（辱骂性词语），而安全研究者们通过改进AI算法来避免这类问题。

#### 5.3.2. `<a name='2'></a>`（2）对第三方和开源代码的依赖

IoT固件开发依赖了大量第三方和开源代码，制造商通常只把新功能、高性能和低功耗作为产品的主要目标，同时尽可能缩短开发周期以提升市场竞争力。因此，他们采用了敏捷开发。许多IoT设备制造商直接重复使用开源代、参考公共代码实现、交叉编译PC平台的代码、依赖第三方库。Cui等人[^113]发现80.4%的打印机固件在发布时包含大量已知漏洞，许多最新的固件升级包仍然含有第三方库漏洞，有些漏洞在8年前就已经披露。尽管这件事情暴露了大量安全问题，但仍然带来了独特的漏洞挖掘技术，可以通过不同层次信息的相似性来挖掘同源漏洞。相似性检测也将推动IoT领域的应用程序。

#### 5.3.3. `<a name='3'></a>`（3）外围设备系统的开发

IoT设备的交互性越来越强，它逐渐提升促进IoT外围设备的发展。IoT设备通常使用终端（PC和手机）、云端和其他系统进行交互。这不仅增加了新的攻击面，同时有助于外围设备分析技术的发展，以解决固件提取集和分析的困难。例如，现有的IoTFuzzer[^57]和访问控制框架[^108-^112]都具备对于外围系统的自动分析和保护技术。

## 6. `<a name='-1'></a>`研究方向

在前面几节中，我们介绍了面临的难题和机会，我们发现IoT漏洞挖掘、检测和环节技术延续了传统安全研究的轨道，但同时也有其不同的研究方向。

- **基于AI的漏洞挖掘和分析技术**不论是在功能还是安全上，IoT和AI技术都在迅速结合。现有的AI技术成功运用在了漏洞检测上。随着研究的继续，AI会扩展到其他的漏洞分析技术上。例如，GANs[^114]已经应用在了IoT系统异常行为检测上[^115]。在未来，GANs或许可以应用在IoT漏洞挖掘领域，因为它可以学习不同的攻击场景，来生成类似0day攻击的样本，并为算法提供一系列现有攻击外的样本。
- **大规模漏洞检测技术**在4.2节中提到，IoT设备的复杂性和异构性妨碍了大规模、自动化的漏洞分析技术研究。然而，这个技术需求在IoT安全产业已经迫在眉睫。安全研究者需要一个跨平台方案来克服这个问题，这也是一个长期的研究方向。
- **自动化漏洞利用**
  为了利用IoT设备中的漏洞并保护设备免受入侵，我们需要自动化生成poc，因为它能帮助更好理解漏洞的危害和成因。随着IoT领域的发展，自动化攻击和防御也将成为热点。
- **外围设备的漏洞分析**
  通过之前对现有难题的分析，我们发现难以通过静态和动态分析直接分析设备。IoT设备的交互性变得越来越强，不仅与外围系统结合的漏洞会越来越多，对外围系统的分析方法的研究也会增加。
- **在二进制代码层面自动生成多平台补丁**
  由于一些IoT厂商将代码闭以及和不注重安全性，设备固件没有办法及时打上补丁。为此，我们需要跨平台二进制代码漏洞自动化修复方案。自动化生成二进制代码层的补丁需要完全理解Bug的成因和消除方法。如果我们完全依赖该领域专家的知识，将会出现数以千计的安全漏洞模板，因此难以达成可扩展且可行的解决方案。同时，操作系统和硬件架构的多样性也带来了技术上的难题。解决自动生成多平台的二进制代码补丁这一难题，将是全安全领域一个长期的目标。

## 7. `<a name='-1'></a>`结论

随着IoT的迅速发展，确保用户的安全和隐私保护带来了显著的影响和挑战。虽然关于IoT设备安全的研究数量逐渐上升，但在信息安全的领域仍然处于起始阶段。因此，需要一个对于现有研究的综合摘要来指引IoT安全的发展。这篇论文分析了消费者层和产业层的IoT设备架构和攻击面，展示了当前研究的背景。我们首先从四个方面完善了分类：分析工具、漏洞挖掘、漏洞检测和漏洞缓解。基于这四个方面，我们回顾了漏洞分析的技术。另外，我们总结了目标、特点和研究方向。随后，我们评估了漏洞分析的技术，发现现有研究面临的难点，包括设备的复杂性和异构性、设备资源的限制、长期闭源的措施。困难同样伴随着机会。AI技术和外围设备分析将会广泛应用在IoT安全领域。在未来，将会有越来越多的技术和新领域结合，来实现大规模、跨架构的自动化漏洞分析。

## 8. 参考

[^1]: Lueth, K.L. State of the IoT 2018: Number of IoT Devices Now at 7B—Market Accelerating. Available online:https://IoT-analytics.com/state-of-the-IoT-update-q1-q2-2018-number-of-IoT-devices-now-7b/ (accessed on 6 December 2019).
    
[^2]: Rawlinson, K. Internet of Things Research Study. Available online: https://www8.hp.com/us/en/hp￾news/press-release.html?id=1744676 (accessed on 6 December 2019).
    
[^3]: Wikipedia. Mirai(malware). Available online: https://en.wikipedia.org/wiki/Mirai_(malware) (accessed on 6 December 2019).
    
[^4]: Trevor, H. Internet of Things (IoT) History. Available online: https://www.postscapes.com/IoT-history/(accessed on 6 December 2019).Future Internet 2019, 12, 27 18 of 23
    
[^5]: Gan, G.; Lu, Z.; Jiang, J. Internet of things security analysis. In Proceedings of the International Conference on Internet Technology and Applications, Wuhan, China, 16–18 August 2011.
    
[^6]: Suo, H.; Wan, J.; Zou, C.; Liu, J. Security in the internet of things: A review. In Proceeding of the International Conference on Computer Science and Electronics Engineering, Hangzhou, China, 23–25 March 2012.
    
[^7]: Zhao, K., Ge; L. A survey on the internet of things security. In Proceedings of the 2013 Ninth International Conference on Computational Intelligence and Security, Leshan, China, 14–15 December 2013.
    
[^8]: Pescatore, J.; Shpantzer, G. Securing the Internet of Things Survey; SANS Institute: Bethesda, MD, USA, 2014;pp. 1–22.
    
[^9]: Balte, A.; Kashid, A.; Patil, B. Security issues in Internet of things (IoT): A survey. Int. J. Adv. Res. Comput.Sci. Softw. Eng. 2018, 5, 450–455.
    
[^10]: Ngu, A.H.; Gutierrez, M.; Metsis, V.; Nepal, S.; Sheng, Q.Z. IoT middleware: A survey on issues and enabling technologies. IEEE Int. Things J. 2016, 4, 1–20. [CrossRef]
    
[^11]: Yang, Y.; Wu, L.; Yin, G.; Li, L.; Zhao, H. A survey on security and privacy issues in Internet-of-Things. IEEE Int. Things J. 2017, 4, 1250–1258. [CrossRef]
    
[^12]: Alaba, F.A.; Othman, M.; Hashem, I.A.T.; Alotaibi, F. Internet of Things security: A survey. J. Net. Comput.Appl. 2017, 88, 10–28. [CrossRef]
    
[^13]: Zhang, Z.K.; Cho, M.C.Y.; Wang, C.W.; Hsu, C.W.; Chen, C.K.; Shieh, S. IoT security: ongoing challenges and research opportunities. In Proceedings of the 7th IEEE International Conference on Service-Oriented Computing and Applications, Matsue, Japan, 17–19 November 2014.
    
[^14]: Mahmoud, R.; Yousuf, T.; Aloul, F.; Zualkernan, I. Internet of things (IoT) security: Current status, challenges and prospective measures. In Proceedings of the 10th International Conference for Internet Technology and Secured Transactions (ICITST), London, UK, 14–16 December 2015.
    
[^15]: Fernandes, E.; Rahmati, A.; Eykholt, K.; Prakash, A. Internet of things security research: A rehash of old ideas or new intellectual challenges. IEEE Secur. Priv. 2017, 15, 79–84. [CrossRef]
    
[^16]: Al-Garadi, M.A.; Mohamed, A.; Al-Ali, A.; Du, X.; Guizani, M. A survey of machine and deep learning methods for internet of things (IoT) security. arXiv 2018, arXiv:1807.11023. Available online: https://arxiv.org/abs/1807.11023 (accessed on 6 December 2019).
    
[^17]: Alrawi, O.; Lever, C.; Antonakakis, M.; Monrose, F. Sok: Security evaluation of home-based IoT deployments. In Proceedings of the IEEE Symposium on Security and Privacy (SP), San Francisco, CA, USA, 19–23 May 2019.
    
[^18]: Xie, W.; Jiang, Y.; Tang, Y.; Ding, N.; Gao, Y. Vulnerability detection in IoT firmware: A survey. In Proceedings of the IEEE 23rd International Conference on Parallel and Distributed Systems (ICPADS), Shenzhen, China, 15–17 December 2017.
    
[^19]: Zheng, Y.; Wen, H.; Cheng, K.; Song, Z.W.; Zhu, H.S.; Sun, L.M. A Survey of IoT Device Vulnerability Mining Techniques. J. Cyber Secur. 2019, 4, 61–75. [CrossRef]
    
[^20]: Samsung. Samsung SmartThings. Available online: https://www.smartthings.com/ (accessed on 6 December 2019).
    
[^21]: Google. Google Weave Project. Available online: https://developers.google.com/weave/ (accessed on 6 December 2019).
    
[^22]: Apple Inc. Apple HomeKit. Available online: http://www.apple.com/ios/home/ (accessed on 6 December 2019).
    
[^23]: Home, A. Home Assistant. Available online: https://www.home-assistant.io (accessed on 6 December 2019).
    
[^24]: Mi Inc. IoT Developer Platform. Available online: https://IoT.mi.com/ (accessed on 6 December 2019).
    
[^25]: WiFi, A. WiFi. Available online: https://www.wi-fi.org/ (accessed on 6 December 2019).
    
[^26]: Zigbee, A. Zigbee. Available online: https://zigbee.org/ (accessed on 6 December 2019).
    
[^27]: Bluetooth Technology Website. Available online: https://www.bluetooth.com/ (accessed on 6 December 2019).
    
[^28]: Liu, X.; Zhou, Z.; Diao, W.; Li, Z.; hang, K. When good becomes evil: Keystroke inference with smartwatch. In Proceedings of the 22nd ACM SIGSAC Conference on Computer and Communications Security, Denver, CO, USA, 12–16 October 2015.
    
[^29]: Das, A.; Borisov, N.; Caesar M. Do you hear what i hear?: Fingerprinting smart devices through embedded acoustic components. In Proceedings of the ACM SIGSAC Conference on Computer and Communications Security, Scottsdale, AZ, USA, 3–7 November 2014. Future Internet 2019, 12, 27 19 of 23
    
[^30]: Vasyltsov, I.; Lee, S. Entropy extraction from bio-signals in healthcare IoT. In Proceedings of the 1st ACM Workshop on IoT Privacy, Trust, and Security, Singapore, 14 April 2015.
    
[^31]: McCann, D.; Eder, K.; Oswald, E. Characterising and comparing the energy consumption of side channel attack countermeasures and lightweight cryptography on embedded device. In Proceedings of the International Workshop on Secure Internet of Things (SIoT), Vienna, Austria, 21–25 September 2015.
    
[^32]: Stokes, P., SentinelOne. Checkm8: 5 Things You Should Know about the New Ios Boot Rom Exploit. Available online: https://www.sentinelone.com/blog/checkm8-5-things-you-should-know-new-ios-boot￾rom-exploit/ (accessed on 6 December 2019).
    
[^33]: MITRE Corp. Marvell WiFi. Available online: https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=+Marvell+WiFi (accessed on 6 December 2019).
    
[^34]: Paganini, P. Million of Telestar Digital GmbH IoT Radio Devices Can Be Remotely Hacked. Available online:https://securityaffairs.co/wordpress/91069/hacking/telestar-IoT-radio-devices-hack.html (accessed on 6 December 2019).
    
[^35]: MITRE Corp. CVE-2019-13473. Available online: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-13473 (accessed on 6 December 2019).
    
[^36]: MITRE Corp. CVE-2019-13474. Available online: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-13474 (accessed on 6 December 2019).
    
[^37]: Costa Gondim, J.J.; de Oliveira Albuquerque, R.; Clayton Alves Nascimento, A.; García Villalba, L.J.;Kim, T.H. A methodological approach for assessing amplified reflection distributed denial of service on the internet of things. Sensors 2016, 16, 1855. [CrossRef] [PubMed]
    
[^38]: Wikipedia. Constrained Application Protocol. Available online: https://en.wikipedia.org/wiki/ Constrained_Application_Protocol (accessed on 6 December 2019).
    
[^39]: UPnP Corp. UPnP Device Architecture 1.0. Available online: http://www.upnp.org/specs/arch/UPnP￾arch-DeviceArchitecture-v1.0-20080424.pdf (accessed on 6 December 2019).
    
[^40]: Li, C.; Cai, Q.; Li, J.; Liu, H.; Zhang, Y.; Gu, D.; Yu, Y. Passwords in the Air: Harvesting Wi-Fi Credentials from SmartCfg Provisioning. In Proceedings of the 11th ACM Conference on Security & Privacy in Wireless and Mobile Networks, Stockholm, Sweden, 18–20 June 2018.
    
[^41]: Zhou, W.; Jia, Y.; Yao, Y.; Zhu, L.; Guan, L.; Mao, Y.; Zhang, Y. Phantom Device Attack: Uncovering the Security Implications of the Interactions among Devices, IoT Cloud, and Mobile Apps. arXiv 2018, arXiv:1811.03241.
    
[^42]: Vasile, S.; Oswald, D.; Chothia, T. Breaking All the Things—A Systematic Survey of Firmware Extraction Techniques for IoT Devices. In Proceedings of the International Conference on Smart Card Research and Advanced Applications, Montpellier, France, 12–14 November 2018.
    
[^43]: Zaddach, J.; Bruno, L.; Francillon, A.; Balzarotti, D. AVATAR: A Framework to Support Dynamic Security Analysis of Embedded Systems’ Firmwares. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 23–26 February 2014.
    
[^44]: Kammerstetter, M.; Platzer, C.; Kastner, W. Prospect: peripheral proxying supported embedded code testing.In Proceedings of the 9th ACM Symposium on Information, Computer and Communications Security, Kyoto, Japan, 3–6 June 2014.
    
[^45]: Koscher, K.; Kohno, T.; Molnar, D. SURROGATES: Enabling Near-Real-Time Dynamic Analyses of Embedded Systems. In Proceedings of the 9th USENIX Workshop on Offensive Technologies (WOOT 15), Washington, DC, USA, 10–11 August 2015.
    
[^46]: Muench, M.; Nisi, D.; Francillon, A.; Balzarotti, D. Avatar 2: A Multi-target Orchestration Platform. In Proceedings of the Workshop on Binary Analysis Research (colocated with NDSS Symposium), San Diego, CA, USA, 18 February 2018.
    
[^47]: Chen, D.D.; Woo, M.; Brumley, D.; Egele, M. Towards Automated Dynamic Analysis for Linux-based Embedded Firmware. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 21–24 February 2016.
    
[^48]: Bellard, F. QEMU, a fast and portable dynamic translator. In Proceedings of the USENIX Annual Technical Conference, Anaheim, CA, USA, 10–15 April 2005.
    
[^49]: Wikipedia. Fuzzing. Available online: https://en.wikipedia.org/wiki/Fuzzing (accessed on 6 December 2019). Future Internet 2019, 12, 27 20 of 23
    
[^50]: Wikipedia. Taint Checking. Available online: https://en.wikipedia.org/wiki/Taint_checking (accessed on 6 December 2019).
    
[^51]: King, J.C. Symbolic execution and program testing. Commun. ACM 1976, 19; 385–394. [CrossRef]
    
[^52]: Alimi, V.; Vernois, S.; Rosenberger, C. Analysis of embedded applications by evolutionary fuzzing.In Proceedings the 2014 International Conference on High Performance Computing & Simulation (HPCS), Bologna, Italy, 21–25 July 2014.
    
[^53]: Kamel, N.; Lanet, J.L. Analysis of HTTP protocol implementation in smart card embedded web server. Int. J. Inf. Netw. Security (IJINS) 2013, 2, 417. [CrossRef]
    
[^54]: Koscher, K.; Czeskis, A.; Roesner, F.; Patel, S.; Kohno, T.; Checkoway, S.; McCoy, D.; Kantor, B.; Anderson, D;Shacham, H.; et al. Experimental security analysis of a modern automobile. In Proceedings of the IEEE Symposium on Security and Privacy (SP), Berkeley, CA, USA, 16–19 May 2010.
    
[^55]: Lee, H.; Choi, K.; Chung, K.; Kim, J.; Yim, K. Fuzzing can packets into automobiles. In Proceedings of the 29th International Conference on Advanced Information Networking and Applications, Gwangiu, Korea,24–27 March 2015.
    
[^56]: Wikipedia. CAN bus. Available online: https://en.wikipedia.org/wiki/CAN_bus (accessed on 6 December 2019).
    
[^57]: Chen, J.; Diao, W.; Zhao, Q.; Zuo, C.; Lin, Z.; Wang, X.; Lau, W.C.; Sun, M.; Yang, R.; Zhang, K. IoTfuzzer:Discovering Memory Corruptions in IoT through App-Based Fuzzing. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 18–21 February 2018.
    
[^58]: Costin, A.; Zarras, A.; Francillon, A. Automated dynamic firmware analysis at scale: A case studyon embedded web interfaces. In Proceedings of the 11th ACM on Asia Conference on Computer and Communications Security, Xi’an, China, 30 May–3 June 2016.
    
[^59]: Srivastava, P.; Peng, H.; Li, J.; Okhravi, H.; Shrobe, H.; Payer, M. FirmFuzz: Automated IoT Firmware Introspection and Analysis. In Proceedings of the 2nd International ACM Workshop on Security and Privacy for the Internet-of-Things, London, UK, 15 November 2019.
    
[^60]: Zheng, Y.; Davanian, A.; Yin, H.; Song, C.; Zhu, H.; Sun, L. FIRM-AFL: high-throughput greybox fuzzing of IoT firmware via augmented process emulation. In Proceedings of the 28th USENIX Security Symposium (USENIX Security 19), Santa Clara, CA, USA, 14–16 August 2019.
    
[^61]: Zalewski, M. American Fuzzy Lop. Available online: http://lcamtuf.coredump.cx/afl (accessed on 6 December 2019).
    
[^62]: Muench, M.; Stijohann, J.; Kargl, F.; Francillon, A.; Balzarotti, D. What You Corrupt Is Not What You Crash: Challenges in Fuzzing Embedded Devices. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 18–21 February 2018.
    
[^63]: Dolan-Gavitt, B.; Hodosh, J.; Hulin, P.; Leek, T.; Whelan, R. Repeatable reverse engineering with PANDA.In Proceedings of the 5th Program Protection and Reverse Engineering Workshop, Los Angeles, CA, USA,15 December 2015.
    
[^64]: Costin, A.; Zaddach, J.; Francillon, A.; Balzarotti, D. A large-scale analysis of the security of embedded firmwares. In Proceedings of the 23rd USENIX Security Symposium (USENIX Security 14), San Diego, CA, USA, 20–22 August 2014.
    
[^65]: Davidson, D.; Moench, B.; Ristenpart, T.; Jha, S. FIE on Firmware: Finding Vulnerabilities in Embedded Systems Using Symbolic Execution. In Proceedings of the 22nd USENIX Security Symposium (USENIX Security 13), Washington, DC, USA, 14–16 August 2013.
    
[^66]: Celik, Z.B.; Babun, L.; Sikder, A.K.; Aksu, H.; Tan, G.; McDaniel, P.; Uluagac, A.S. KLEE: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs. In Proceedings of the 8th USENIX Symposium on Operating Systems Design and Implementation(OSDI 2008), San Diego, CA, USA, 8–10 December 2008.
    
[^67]: Shoshitaishvili, Y.; Wang, R.; Hauser, C.; Kruegel, C.; Vigna, G. Firmalice-Automatic Detection of Authentication Bypass Vulnerabilities in Binary Firmware. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 8–11 February 2015.
    
[^68]: Celik, Z.B.; Babun, L.; Sikder, A.K.; Aksu, H.; Tan, G.; McDaniel, P.; Uluagac, A.S. Sensitive information tracking in commodity IoT. In Proceedings of the 27th USENIX Security Symposium (USENIX Security 18), Baltimore, MD, USA, 15–17 August 2018. Future Internet 2019, 12, 27 21 of 23
    
[^69]: Cheng, K.; Li, Q.; Wang, L.; Chen, Q.; Zheng, Y.; Sun, L.; Liang, Z. DTaint: detecting the taint-style vulnerability in embedded device firmware. In Proceedings of the 48th Annual IEEE/IFIP International Conference on Dependable Systems and Networks (DSN), Luxembourg, 25–28 June 2018.
    
[^70]: Cui, A.; Stolfo, S.J. A quantitative analysis of the insecurity of embedded network devices: results of a wide-area scan. In Proceedings of the 26th Annual Computer Security Applications Conference, Austin, TX, USA, 6–10 December 2010.
    
[^71]: Al-Alami, H;, Ali, H.; Hussein, A.B. Vulnerability scanning of IoT devices in Jordan using Shodan. In Proceedings of the 2nd International Conference on the Applications of Information Technology in Developing Renewable Energy Processes & Systems (IT-DREPS), Amman, Jordan, 6–7 December 2017.
    
[^72]: Durumeric, Z.; Adrian, D.; Mirian, A.; Bailey, M.; Halderman, J.A. A search engine backed by Internet-wide scanning. In Proceedings of the 22nd ACM SIGSAC Conference on Computer and Communications Security, Denver, CO, USA, 12–16 October 2015.
    
[^73]: Knownsec, Inc. Zoomeye. Available online: https://www.zoomeye.org/ (accessed on 6 December 2019).
    
[^74]: Li, Z.; Lu, S.; Myagmar, S.; Zhou, Y. CP-Miner: A Tool for Finding Copy-paste and Related Bugs in Operating System Code. In Proceedings of the 6th Symposium on Operating System Design and Implementation (OSDI 2004), San Francisco, CA, USA, 6–8 December 2004.
    
[^75]: Jang, J.; Agrawal, A.; Brumley, D. ReDeBug: finding unpatched code clones in entire os distributions. In Proceedings of the IEEE Symposium on Security and Privacy (SP), San Francisco, CA, USA, 20–23 May 2012.
    
[^76]: Wikipedia. N-gram. Available online: https://en.wikipedia.org/wiki/N-gram (accessed on 6December 2019).
    
[^77]: Myles, G.; Christian, C. K-gram based software birthmarks. In Proceedings of the 2005 ACM Symposium on Applied Computing, Santa Fe, NM, USA, 13–17 March 2005.
    
[^78]: Khoo, W.M.; Mycroft, A.; Anderson R. Rendezvous: A search engine for binary code. In Proceedings of the 10th Working Conference on Mining Software Repositories, San Francisco, CA, USA, 18–19 May 2013.
    
[^79]: Karim, M.E.; Walenstein, A.; Lakhotia, A.; Parida, L. Malware phylogeny generation using permutations of code. J. Comput. Virol. 2005, 1, 13–23. [CrossRef]
    
[^80]: David, Y.; Yahav, E. Tracelet-based code search in executables. Acm Sigplan Notices 2014, 49, 349–360.[CrossRef]
    
[^81]: Pewny, J.; Schuster, F.; Bernhard, L.; Holz, T.; Rossow, C. Leveraging semantic signatures for bug search in binary programs. In Proceedings of the 30th Annual Computer Security Applications Conference, New Orleans, LA, USA, 8–12 December 2014.
    
[^82]: Allen, F.E. Control flow analysis. ACM Sigplan Notices 1970, 55, 7. [CrossRef]
    
[^83]: Dullien, T.; Rolles, R. Graph-based comparison of executable objects. In Proceedings of the SSTIC’05, Rennes, France, 1–3 July 2005.
    
[^84]: Bourquin, M.; King, A.; Robbins, E. Binslayer: accurate comparison of binary executables. In Proceedings of the 2nd ACM SIGPLAN Program Protection and Reverse Engineering Workshop, Rome, Italy, 26 January 2013.
    
[^85]: Egele, M.; Woo, M.; Chapman, P.; Brumley, D. Blanket execution: Dynamic similarity testing for program binaries and components. In Proceedings of the 23rd USENIX Security Symposium (USENIX Security 14), San Diego, CA, USA, 20–22 August 2014.
    
[^86]: Gao, D.; Reiter, M.K.;Song, D. Binhunt: Automatically finding semantic differences in binary programs. In Proceedings of the International Conference on Information and Communications Security, Birmingham, UK, 20–22 October 2008.
    
[^87]: Ming, J.; Pan, M.; Gao, D. iBinHunt: Binary hunting with inter-procedural control flow. In Proceedings of the International Conference on Information Security and Cryptology, Seoul, Korea, 28–30 November 2012.
    
[^88]: Pewny, J.; Garmany, B.; Gawlik, R.; Rossow, C.; Holz, T. Cross-architecture bug search in binary executables. In Proceedings of the IEEE Symposium on Security and Privacy (SP), San Jose, CA, USA, 17–21 May 2015.
    
[^89]: Eschweiler, S.; Yakdan, K.; Gerhards-Padilla, E. discovRE: Efficient Cross-Architecture Identification of Bugs in Binary Code. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 21–24 February 2016.Future Internet 2019, 12, 27 22 of 23
    
[^90]: Chandramohan, M.; Xue, Y.; Xu, Z.; Liu, Y.; Cho, C.Y.; Tan, H.B.K. Bingo: Cross-architecture cross-os binary search. In Proceedings of the 24th ACM SIGSOFT International Symposium on Foundations of Software Engineering, Seattle, WA, USA, 13–18 November 2016.
    
[^91]: Feng, Q.; Zhou, R.; Xu, C.; Cheng, Y.; Testa, B.; Yin, H. Scalable graph-based bug search for firmware images. In Proceedings of the ACM SIGSAC Conference on Computer and Communications Security, Vienna, Austria, 24–28 October 2016.
    
[^92]: Yan, S.; Xu, D.; Zhang, B.; Zhang, H.J.; Yang, Q.; Lin, S. Graph embedding and extensions: A general framework for dimensionality reduction. IEEE Transact. Pattern Anal. Mach. Intell. 2007, 29, 40–51. [CrossRef]
    
[^93]: Xu, X.; Liu, C.; Feng, Q.; Yin, H.; Song, L.; Song, D. Neural network-based graph embedding for cross-platform binary code similarity detection. In Proceedings of the ACM SIGSAC Conference on Computer and Communications Security, Dallas, TX, USA, 30 October–3 November 2017.
    
[^94]: Liu, B.; Huo, W.; Zhang, C.; Li, W.; Li, F.; Piao, A.; Zou, W. αDiff: cross-version binary code similarity detection with DNN. In Proceedings of the 33rd ACM/IEEE International Conference on Automated Software Engineering, Montpellier, France, 3–7 September 2018.
    
[^95]: Gao, J.; Yang, X.; Fu, Y.; Jiang, Y.; Sun, J. Vulseeker: a semantic learning based vulnerability seeker for cross-platform binary. In Proceedings of the 33rd ACM/IEEE International Conference on Automated Software Engineering, Montpellier, France, 3–7 September 2018.
    
[^96]: Gao, J.; Yang, X.; Fu, Y.; Jiang, Y.; Shi, H.; Sun, J. Vulseeker-pro: enhanced semantic learning based binary vulnerability seeker with emulation. In Proceedings of the 26th ACM Joint Meeting on European Software Engineering Conference and Symposium on the Foundations of Software Engineering, Tallinn, Estonia, 26–30 August 2019.
    
[^97]: Long, F.; Rinard, M. Prophet: Automatic Patch Generation via Learning from Successful Patches. https://core.ac.uk/download/pdf/78062945.pdf (accessed on 6 December 2019).
    
[^98]: Long, F.; Rinard, M. Automatic patch generation by learning correct code. In Proceedings of the 43rd Annual ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages, St. Petersburg, FL, USA, 20–22 January 2016.
    
[^99]: Long, F.; Amidon, P.; Rinard, M. Automatic inference of code transforms for patch generation. In Proceedings of the 11th Joint Meeting on Foundations of Software Engineering, Paderborn, Germany, 4–8 September 2017.
    
[^100]: Le Goues, C.; Nguyen, T.; Forrest, S.; Weimer, W. Genprog: A generic method for automatic software repair. IEEE Trans. Soft. Eng. 2011, 38, 54–72. [CrossRef]
    
[^101]: Kim, D.; Nam, J.; Song, J.; Kim, S. Automatic patch generation learned from human-written patches. In Proceedings of the International Conference on Software Engineering, San Francisco, CA, USA, 18–26 May 2013.
    
[^102]: Zhang, Y.; Chen, Y.; Bao, C.; Xia, L.; Zhen, L.; Lu, Y.; Wei, T. Adaptive kernel live patching: An open collaborative effort to ameliorate android n-day root exploits. In Proceedings of Black Hat USA, Las Vegas, NA, USA, 30 July–4 August 2016.
    
[^103]: DARPA. Cyber Grand Challenge. Available online: https://www.darpa.mil/program/cyber-grand￾challenge (accessed on 6 December 2019).
    
[^104]: Shoshitaishvili, Y.; Bianchi, A.; Borgolte, K.; Cama, A.; Corbetta, J.; Disperati, F.; Dutcher, A.; Grosen, J.;Grosen, P.;Machiry, A.; etc. Mechanical phish: Resilient autonomous hacking. IEEE Secur. Priv. 2018, 16, 12–22. [CrossRef]
    
[^105]: Shoshitaishvili, Y.; Weissbacher, M.; Dresel, L.; Salls, C.; Wang, R.; Kruegel, C.; Vigna, G. Rise of the hacrs: Augmenting autonomous cyber reasoning systems with human assistance. In Proceedings of the ACM SIGSAC Conference on Computer and Communications Security, Dallas, TX, USA, 30 October–3 November 2017.
    
[^106]: Nguyen-Tuong, A.; Melski, D.; Davidson, J.W.; Co, M.; Hawkins, W.; Hiser, J.D.;Morris, D.; Nguyen, D.; Rizzi, E. Xandra: An Autonomous Cyber Battle System for the Cyber Grand Challenge. IEEE Secur. Priv.2018, 16, 42–51. [CrossRef]
    
[^107]: Fernandes, E.; Jung, J.; Prakash, A. Security analysis of emerging smart home applications. In Proceedings of the IEEE Symposium on Security and Privacy (SP), San Jose, CA, USA, 22–26 May 2016.
    
[^108]: Fernandes, E.; Paupore, J.; Rahmati, A.; Simionato, D.; Conti, M.; Prakash, A. Flowfence: Practical data protection for emerging IoT application frameworks. In Proceedings of the 25th USENIX Security Symposium (USENIX Security 16), Austin, TX, USA, 10–12 August 2016. Future Internet 2019, 12, 27 23 of 23
    
[^109]: Jia, Y.J.; Chen, Q.A.; Wang, S.; Rahmati, A.; Fernandes, E.; Mao, Z.M.; Prakash, A. ContexloT: Towards Providing Contextual Integrity to Appified IoT Platforms. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 26 February–1 March 2017.
    
[^110]: Rahmati, A.; Fernandes, E.; Eykholt, K.; Prakash, A. Tyche: A risk-based permission model for smart homes. In Proceedings of the IEEE Cybersecurity Development (SecDev), Cambridge, MA, USA, 30 September–2 October 2018.
    
[^111]: Tian, Y.; Zhang, N.; Lin, Y.H.; Wang, X.; Ur, B.; Guo, X.; Tague, P. Smartauth: User-centered authorization for the internet of things. In proceedings of the 26th USENIX Security Symposium (USENIX Security 17), Vancouver, BC, Canada, 16–18 August 2017.
    
[^112]: Lee, S.; Choi, J.; Kim, J.; Cho, B.; Lee, S.; Kim, H.; Kim, J. FACT: Functionality-centric access control system for IoT programming frameworks. In Proceedings of the 22nd ACM on Symposium on Access Control Models and Technologies, Indianapolis, IN, USA, 21–23 June 2017.
    
[^113]: Cui, A.; Costello, M.; Stolfo, S. When firmware modifications attack: A case study of embedded exploitation. In Proceedings of the Network and Distributed System Security (NDSS) Symposium, San Diego, CA, USA, 24–27 February 2013.
    
[^114]: Goodfellow, I.; Pouget-Abadie, J.; Mirza, M.; Xu, B.; Warde-Farley, D.; Ozair, S.; Courville, A.; Bengio, Y. Generative adversarial nets. In Proceedings of the Advances in Neural Information Processing Systems 27 (NIPS 2014), Montreal, QC, Canada, 8–13 December 2014.
    
[^115]: Hiromoto, R.E.; Haney, M.; Vakanski, A. A secure architecture for IoT with supply chain risk management. In Proceedings of the 9th IEEE International Conference on Intelligent Data Acquisition and Advanced Computing Systems: Technology and Applications (IDAACS), Bucharest, Romania, 21–23 September 2017.
