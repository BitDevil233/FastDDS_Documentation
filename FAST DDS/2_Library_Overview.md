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
Fast DDS实现了一个并发多线程系统。每个DomainParticipant生成一组线程来处理后台任务，比如日志记录、消息接收和异步通信。这不应影响您使用库的方式，即Fast DDS API是线程安全的，因此您可以放心地从不同线程调用同一个DomainParticipant上的任何方法。然而，当外部函数访问被库内部运行的线程修改的资源时，必须考虑这种多线程实现。其中一个例子是实体监听器回调中修改的资源。
Fast DDS 生成的完整`线程集`如下所示。仅当使用适当的传输时才会创建与传输相关的线程（标记为 UDP、TCP 和 SHM 类型）。
|Name | Type | Cardinality | OS thread name | Description|
|---|---|---|---|---|
***`此表格待更新`***

其中一些线程仅在满足某些条件时才会产生：

- 仅当使用数据共享时才会创建数据共享侦听器线程。

- 仅当 DomainParticipant 配置为 `Discovery Server SERVER`时，才会创建 Discovery Server 事件线程。

- TCP 保活线程要求保活周期配置为大于零的值。

- 安全日志记录和共享内存数据包日志记录线程都需要启用某些配置选项。

- 仅当使用FASTDDS_ENVIRONMENT_FILE时才会生成文件监视线程。

关于传输线程，Fast DDS 默认使用`UDP`和`共享内存`传输。可以配置端口配置以满足部署的特定需求，但默认配置是始终使用元流量端口和单播用户流量端口。这适用于UDP和共享内存，因为TCP不支持多播。更多信息可以在默认监听定位器页面找到。

Fast DDS提供了通过 ThreadSettings 配置其创建的线程的某些属性的可能性。

### 2.2.2事件驱动架构
有一个`时间-事件`系统，使Fast DDS能够响应某些条件，并安排定期操作。其中很少对用户可见，因为大多数与 DDS 和 RTPS 元数据相关。然而，用户可以通过从该类继承来在其应用程序中定义周期性时间事件`TimedEvent`。
## 2.3. 功能
Fast DDS具有一些附加功能，用户可以在其应用程序中实现和配置这些功能。这些概述如下。

### 2.3.1. 发现协议
发现协议定义了在给定主题下发布的 DataWriter 和订阅同一主题的 DataReader 进行匹配的机制，以便它们可以开始共享数据。这适用于沟通过程中的任何时刻。Fast DDS提供以下发现机制：

- 简单的发现。这是默认的发现机制，在RTPS 标准中定义，并提供与其他 DDS 实现的兼容性。这里，DomainParticipants 在早期阶段被单独发现，以便随后匹配它们实现的 DataWriter 和 DataReader。

- 发现服务器。这种发现机制使用集中式发现架构，其中服务器充当元流量发现的中心。

- 静态发现。这实现了 DomainParticipant 之间的发现，但如果远程 DomainParticipant 事先知道这些实体，则可以跳过每个 DomainParticipant (DataReader/DataWriter) 中包含的实体的发现。

- 手动发现。该机制仅与RTPS层兼容。它允许用户使用其选择的任何外部元信息通道手动匹配和取消匹配 RTPSParticipants、RTPSWriters 和 RTPSReaders。

Fast DDS中实现的所有发现协议的详细解释和配置可以在Discovery部分看到。

### 2.3.2. 安全
Fast DDS可配置为通过在三个级别实现可插拔安全性来提供安全通信：

远程域参与者的身份验证。DDS :Auth:PKI-DH插件使用受信任的证书颁发机构 (CA) 和 ECDSA 数字签名算法提供身份验证来执行相互身份验证。它还使用椭圆曲线 Diffie-Hellman (ECDH) 密钥协商协议建立共享密钥。

实体的访问控制。DDS :Access:Permissions插件在 DDS 域和主题级别提供对 DomainParticipants 的访问控制。

数据加密。DDS :Crypto:AES-GCM-GMAC插件使用伽罗瓦计数器模式 (AES-GCM) 中的高级加密标准 (AES) 提供经过身份验证的加密。

有关Fast DDS中安全配置的更多信息，请参阅安全部分。

### 2.3.3. 记录
Fast DDS提供了可扩展的日志记录系统。Log类是日志系统的入口点。它公开了三个宏定义以方便使用：`EPROSIMA_LOG_INFO`、`EPROSIMA_LOG_WARNING`和`EPROSIMA_LOG_ERROR`。此外，除了已有的类别（`INFO_MSG`、`WARN_MSG`、`ERROR_MSG`）之外，它还允许定义新类别。它使用正则表达式提供按类别过滤，以及对日志系统的详细程度的控制。可能的日志系统配置的详细信息可以在日志部分找到。

### 2.3.4. XML 配置文件配置
Fast DDS提供了使用 XML 配置文件来更改其默认设置的可能性。因此，可以修改DDS实体的行为，而无需用户实现任何程序源代码或重新构建现有应用程序。

用户拥有每个 API 功能的 XML 标签。因此，可以通过 标签构建和配置 DomainParticipant 配置文件，或者分别使用和标签<participant>构建和配置 DataWriter 和 DataReader 配置文件。<data_writer><data_reader>

为了更好地了解如何编写和使用这些 XML 配置文件配置文件，您可以继续阅读XML 配置文件部分。

### 2.3.5。环境变量
环境变量是通过操作系统功能在程序范围之外定义的变量。Fast DDS依赖于环境变量，因此用户可以轻松自定义 DDS 应用程序的默认设置。请参阅环境变量部分，了解影响Fast DDS的环境变量的完整列表和说明。