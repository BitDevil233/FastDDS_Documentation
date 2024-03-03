# 2.库概述
Fast DDS(以前成为Fast RTPS)是DDS规范的搞笑、高性能实现，DDS规范是一种用于分布式应用软件的以数据为中心的通信中间件(DCPS)。本节回顾Fast DDS的架构、操作和主要特性。
## 2.1 架构
Fast DDS的架构如下图所示，可以看到以下不同环境的层模型。
- 应用层。利用 Fast DDS API 在分布式系统中实现通信的用户应用程序。

- Fast DDS层。DDS通信中间件的稳健实现。它允许部署一个或多个DDS 域，其中同一域内的 DomainParticipants 通过在域主题下发布/订阅来交换消息。

- RTPS 层。实施实时发布-订阅 (RTPS) 协议，以实现与 DDS 应用程序的互操作性。该层充当传输层的抽象层。

- 传输层。快速 DDS 可用于各种传输协议，例如不可靠传输协议 (UDP)、可靠传输协议 (TCP) 或共享内存传输协议 (SHM)。

![架构图](https://fast-dds.docs.eprosima.com/en/latest/_images/library_overview.svg "FastDDS层模型架构")

### 2.1.1 DDS层
Fast DDS 的 DDS 层定义了通信的几个关键元素。用户将在其应用程序中创建这些元素，从而整合DDS应用程序元素并创建以数据为中心的通信系统。 Fast DDS遵循DDS规范，将通信中涉及的这些元素定义为`实体`。(可以这样理解)DDS实体是支持服务质量配置 (QoS)并实现侦听器的任何对象。
- 服务质量(Qos)。定义每个实体的行为的机制。
- 监听器(Liscener)。在应用程序执行过程中通知实体可能出现的事件的机制。

下面列出了DDS实体及其描述和功能。有关每个实体、其QoS及其侦听器的更详细说明，请参阅DDS层部分。

- 域(Domain)：一个正整数，用于标识DDS域。每个域参与者将被分配一个DDS域，因此相同域中的域参与者可以进行通信，并且可以隔离不同DDS域之间的通信。应用程序开发人员在创建域参与者时必须提供这个数值。
- 域参与者(DomainParticipant)：域参与者是包含其他DDS实体(例如发布者、订阅者、主题和多主题)的对象,它允许创建其包含的实体(如发布器、订阅器等)以及配置实体行为。
- 发布器：发布器使用DataWriter发布主题(Topic)下的数据，DataWriter将数据写入传输(流)中。发布器是创建和配置其包含的DataWriter实体的实体，并且可能包含其中一个或多个。
- 数据写入器。它是负责发布消息的实体：用户在创建该实体时必须提供一个主题，该主题将作为发布数据的主题。发布是通过将数据对象写入 `(数据写入器历史记录)DataWriterHistory`中的变化来完成的。
- 数据写入器历史记录：这是数据对象更改的列表。当DataWriter继续在特定主题下发布数据时，它实际上会在此数据中创建更改。正是这种变化被记录在历史中。然后，这些更改将发送到订阅该特定主题的DataReader。
- 订阅器(Subscriber)：订阅器使用`数据读取器(DataReader)`订阅主题(Topic)，DataReader从传输中读取数据。订阅是创建和配置其包含的DataReader实体的实体，并且可能包含一个或多个DataReader实体。
- 数据读取器(DataReader)。它是订阅主​​题以接收发布器(发布的数据)的实体。用户在创建该实体时必须提供订阅主题(Topic)。DataReader接收其 HistoryDataReader中发生更改的消息。
- 数据读取器历史记录(DataReaderHistory)。它包含了DataReader由于订阅某个主题而收到的数据对象的更改。
- 话题(Topic)。将发布者的DataWriter与订阅者的DataReader绑定的实体。

### 2.1.2.RTPS层
如上所述，Fast DDS 中的 RTPS 协议允许从传输层抽象 DDS 应用实体。根据上图，RTPS 层有四个主要实体。
- RTPSDomain。它是DDS域对RTPS协议的扩展。
- RTPSParticipant。包含其他 RTPS 实体的实体。它允许配置和创建它包含的实体。
- RTPSWriter。消息的来源。它读取 DataWriterHistory 中写入的更改，并将它们传输到之前匹配的所有 RTPSReader。
- RTPSReader。消息的接收实体。它将 RTPSWriter 报告的更改写入 DataReaderHistory中。

### 2.1.3.传输层
Fast DDS 支持通过各种传输协议实施应用程序。它们是 UDPv4、UDPv6、TCPv4、TCPv6 和共享内存传输 (SHM)。默认情况下，DomainParticipant 实现 UDPv4 和 SHM 传输协议。所有支持的传输协议的配置在`传输层`部分中有详细介绍。

## 2.2.编程和执行模型
Fast DDS 是并发的且基于事件的。以下内容解释了控制 Fast DDS 操作的`多线程模型`以及可能发生的事件。
### 2.2.1.并发和多线程