#  The wire protocol（wire 传输协议）

本文为 [wiki](https://en.wikipedia.org/wiki/Wire_protocol) 上关于 **wire protocol** 的翻译，不保证准确性。



## 简介

在计算机网络中，**wire protocol** 指的是一种端到端（点到点）的数据获取方式，是多个应用<sup>[1]</sup>之间进行交互的必需品。它通常指的是一种高于物理层的协议。相较于传输层的各种传输协议 （TCP、UDP），**wire protocol** 术语习惯于被用来描述信息位于应用层上的一种通用表现形式，是一种应用层上的通用协议而非各类应用程序的通用型对象描述语意<sup>[2]</sup>。就像是 XML<sup>[3]</sup> 与 XSD<sup>[4]</sup> 之间的关系，XML 对传输的数据进行描述，而 XSD 对 XML 本身的文档结构的元素及语法进行描述。

**wire protocol** 要么是一种文本格式协议要么是一种二进制格式协议。虽然决定采用那种格式，这是一种非常重要的架构设计，但是这并不是 wire protocol 与程序API之间的区别。

在电子信息技术领域中<sup>[5]</sup>，一种 wire protocol 描述其讲数据从一个地方传输到其他地方的机制。



## 功能性 [Functionality]

**wire protocol** 提供应用与应用之间在互联网中进行交互的手段，通常指的是各种分布式对象协议<sup>[6]</sup>，其可能是一种使用专门设计用来协同工作的应用程序。顾名思义，这些分布式对象协议运行在互联网上一个或多个计算机的不同进程之间。



## 分类 [Types]

**wire protocol** 提供了一种互联网上运行于不同操作系统之间的程序进行交流的手段，就算是不同的平台或开发语言。


以下是一些 wire prootocol :

- [IIOP](https://en.wikipedia.org/wiki/IIOP) for [CORBA](https://en.wikipedia.org/wiki/CORBA)
- [RTPS](https://en.wikipedia.org/wiki/Real-Time_Publish-Subscribe_(RTPS)_Protocol) for [DDS](https://en.wikipedia.org/wiki/Data_Distribution_Service)
- Java Debug Wire Protocol ([JDWP](https://en.wikipedia.org/wiki/JDWP)) for Java debugging
- [JRMP](https://en.wikipedia.org/w/index.php?title=Java_Remote_Method_Protocol&action=edit&redlink=1) for [RMI](https://en.wikipedia.org/wiki/Java_remote_method_invocation)
- [SOAP](https://en.wikipedia.org/wiki/SOAP) for [Web services](https://en.wikipedia.org/wiki/Web_services)
- [AMQP](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) for [message-oriented middleware](https://en.wikipedia.org/wiki/Message-oriented_middleware)




## 注释引用

1. 应用：即 application。
2. 通用型对象描述语意：common object semantic
3. XML：[Extensible Markup Language](https://en.wikipedia.org/wiki/XML)
4. XSD：[XML Schema Definition](https://en.wikipedia.org/wiki/XML_Schema_(W3C))
5. 电子信息技术：[Electronics](https://en.wikipedia.org/wiki/Electronics)
6. 分布式对象协议：distributed object protocols
