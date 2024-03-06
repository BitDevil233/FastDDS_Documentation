# 3.DDS层
eProsima Fast DDS公开了两个不同的 API 来与不同级别的通信服务进行交互。主要的API是数据分发服务（DDS）数据中心发布订阅（DCPS）平台独立模型（PIM）API，简称DDS DCPS PIM ，由数据分发服务（DDS）1.4版规范定义，Fast DDS遵守该规定。本节主要讲解**该API在Fast DDS下的主要特点和使用方式**，并对其分为五个模块进行深入讲解：
- 核心(Core)：它定义了由其他模块重定义的抽象类和接口。它还提供服务质量 (QoS) 定义，以及对基于通知的中间件交互方式的支持。

- 域(Domain)：它包含DomainParticipant充当服务入口点的类，以及许多类的工厂。DomainParticipant还充当组成服务的其他对象的容器。

- 发布器(Publisher)：描述发布端使用的类，包括Publisher和DataWriter类，以及PublisherListener和DataWriterListener接口。

- 订阅器(Subscriber)：描述订阅端使用的类，包括Subscriber和DataReader类，以及SubscriberListener和DataReaderListener接口。

- 主题：描述了用于定义通信主题和数据类型的类，包括Topic和TopicDescription类，以及TypeSupport和TopicListener接口。

- 3.1. 核
  - 3.1.1. 实体
  - 3.1.2. 策略
  - 3.1.3. 状态
  - 3.1.4. 条件和等待时间
- 3.2. 域
    - 3.2.1. 域参与者
    - 3.2.2. 域参与者监听器
    - 3.2.3. 域参与者工厂
    - 3.2.4. 创建域参与者
    - 3.2.5. 分区
- 3.3. 发布器
    - 3.3.1. 发布器
    - 3.3.2. 发布器监听器
    - 3.3.3. 创建发布器
    - 3.3.4. 数据写入器
    - 3.3.5. 数据写入监听器
    - 3.3.6. 创建数据写入器
    - 3.3.7. 发布数据
- 3.4. 订阅器
    - 3.4.1. 订阅器
    - 3.4.2. 订阅器监听器
    - 3.4.3. 创建订阅器
    - 3.4.4. 数据读取器
    - 3.4.5. 数据读取器监听器
    - 3.4.6. 创建数据读取器
    - 3.4.7. 样本信息
    - 3.4.8. 访问接收到的数据
- 3.5. 话题
    - 3.5.1. 话题、键和实例
    - 3.5.2. 话题描述
    - 3.5.3. 话题
    - 3.5.4. 内容过滤主题
    - 3.5.5. 话题监听器
    - 3.5.6. 数据类型的定义
    - 3.5.7. 创建话题
    - 3.5.8. 过滤某个话题的数据
    - 3.5.9. 默认的类似 SQL 的过滤器
    - 3.5.10. 使用自定义过滤器
    - 3.5.11. 过滤应用在哪里：写入方与读取方
    - 3.5.12. 用于数据类型源代码生成的Fast DDS-Gen

## 3.1 核心
该模块定义了其他模块将使用的基础设施类和类型。它包含实体类、QoS 策略和状态的定义。

- 实体：实体是一个 DDS 通信对象，具有状态并且可以配置策略。

- 策略：控制实体行为的每个配置对象。

- 状态：与实体关联的每个对象，其值表示该实体的通信状态。

### 3.1.1.实体
`Entity`是所有 DDS 实体的抽象基类，表示支持 QoS 策略、侦听器和状态的对象。
#### 3.1.1.1.实体类型
- 域参与者：该实体是服务的入口点，充当发布器、订阅器和话题的工厂。有关更多详细信息，请参阅DomainParticipant。

- 发布器：它充当可以创建任意数量的 DataWriter 的工厂。有关详细信息，请参阅发布器。

- 订阅器：它充当可以创建任意数量的 DataReader 的工厂。有关详细信息，请参阅订阅者。

- 主题：该实体介于发布实体和订阅实体之间，充当通道。请参阅主题了解更多详细信息。

- 数据写入器(DataWriter)：是负责数据分发的对象。有关详细信息，请参阅数据写入器。

- 数据读取器(DataReader)：是用于访问接收到的数据的对象。有关详细信息，请参阅DataReader 。

下图显示了所有DDS实体之间的层次结构：

![](https://fast-dds.docs.eprosima.com/en/latest/_images/entity_diagram.svg)

#### 3.1.1.2. 共同实体特征
所有实体类型都共享一些实体概念所共有的特征。那些是：

##### 3.1.1.2.1. 实体标识符
每个实体都由唯一的 ID 来标识，该 ID 在 DDS 实体及其对应的 RTPS 实体（如果存在）之间共享。该 ID 存储在 Entity 基类上声明的   `Instance Handle`对象上，可以使用 getter 函数访问该对象get_instance_handle()。

##### 3.1.1.2.1.Qos策略

每个实体的行为可以通过一组配置策略进行配置。对于**每种实体类型，都有一个对应的服务质量（Quality of Service，QoS）类**，用于分组影响该实体类型的所有策略。用户可以创建这些QoS类的实例，根据自己的需求修改其中的策略，并使用它们来配置实体，可以在创建实体时或稍后使用每个实体公开的set_qos()函数（DomainParticipant::set_qos()、Publisher::set_qos()、Subscriber::set_qos()、Topic::set_qos()、DataWriter::set_qos()、DataReader::set_qos()）进行配置。有关可用策略及其描述，请参阅策略（Policy）部分。有关QoS类及其包含的策略的详细信息，请参阅每种实体类型的文档。

##### 3.1.1.2.3.监听器

监听器是一个具有函数的对象，实体将根据事件调用这些函数。因此，监听器充当一个异步通知系统，允许实体通知应用程序有关实体状态变化的情况。

所有实体类型都定义了一个`抽象`监听器接口，该接口包含实体将触发的回调函数，以便将状态变化通知应用程序。用户可以`继承`这些接口并实现所需的回调函数来实现自己的监听器。然后，他们可以将这些监听器与每个实体链接在一起，可以在创建实体时或稍后使用每个实体公开的set_listener()函数（DomainParticipant::set_listener()、Publisher::set_listener()、Subscriber::set_listener()、Topic::set_listener()、DataWriter::set_listener()、DataReader::set_listener()）进行链接。每种实体类型的监听器接口和其回调函数在每种实体类型的文档中有详细解释。当事件发生时，具有非空监听器并且在其StatusMask中启用了相应回调的最低级别实体处理该事件。更高级别的监听器从低级别的监听器继承，如下图所示：

![监听器继承图](https://fast-dds.docs.eprosima.com/en/latest/_images/listeners_inheritance_diagram.svg)


> **笔记：**
> 
> 在on_data_available()之前，on_data_on_readers()回调会拦截消息。这意味着如果启用了DomainParticipantListener，用户应该考虑到默认情况下监听器使用StatusMask::all()。由于保持了回调实体层次结构，因此在这种情况下会调用on_data_on_readers()。如果应用程序想要使用on_data_available()，则应禁用相应的StatusMask位。。

> **重要**
>
>在创建实体时使用StatusMask::none() 只会禁用DDS标准回调,包括：
> - on_sample_rejected()
> - on_liveliness_changed()
> - on_requested_deadline_missed()
> - on_requested_incompatible_qos()
> - on_data_available()
> - on_subscription_matched()
> - on_sample_lost()
> - on_offered_incompatible_qos()
> - on_offered_deadline_missed()
> - on_liveliness_lost()
> - on_publication_matched()
> - on_inconsistent_topic()
> - on_data_on_readers()

任何特定于Fast DDS 的回调都始终是启用的。

> - on_participant_discovery()
> - onParticipantAuthentication()
> - on_subscriber_discovery()
> - on_publisher_discovery()
> - on_type_discovery()
> - on_type_dependencies_reply()
> - on_type_information_received()
> - on_unacknowledged_sample_removed()

>**警告**
>
>Only one thread is created to listen for every listener implemented, so it is encouraged to keep listener functions simple, leaving the process of such information to the proper class.
>【不知道怎么翻译这句话，因此保留原文】


>**警告**
>
>不要在侦听器成员函数范围内创建或删除任何实体，因为这可能会导致未定义的行为。建议使用 Listener 类作为信息通道，并使用上层 Entity 类来封装此类行为。

##### 3.1.1.2.5.状态

每个实体都与一组状态对象相关联，这些状态对象的值标识了该实体的通信状态。这些状态值的更改会触发监听器回调以异步的方式通知应用程序。有关所有状态对象的列表及其内容的描述，请参阅状态。您还可以在其中找到哪种状态适用于哪种实体类型。

##### 3.1.1.2.5.状态条件
每个实体都拥有一个 StatusCondition，每当其启用状态发生变化时都会收到通知。StatusCondition 提供实体和等待集之间的链接。有关详细信息，请参阅条件和等待集部分。

##### 3.1.1.2.6.启用实体
所有实体都可以创建为启用或不启用。默认情况下，工厂被配置为创建`启用的实体`，但可以使用启用的工厂上的 EntityFactoryQosPolicy 进行更改。禁用工厂会创建禁用实体，无论其 QoS 如何。禁用实体的操作仅限于以下操作：

- 设置/获取实体 QoS 策略。

- 设置/获取实体监听器。

- 创建/删除子实体。

- 获取实体的状态，即使它们不会改变。

- 查找操作。

在此状态下调用的任何其他函数都将返回`NOT_ENABLED`。

### 3.1.2.策略
服务质量 (QoS) 用于指定服务的行为，允许用户定义每个实体的行为方式。为了增加系统的灵活性，QoS被分解为多个可以独立配置的QoS策略。然而，在某些情况下，多项政策可能会发生冲突。这些冲突通过QoS 设置器函数返回的ReturnCodes通知给用户。

每个 Qos 策略都有一个在`QosPolicyId_t`枚举器中定义的唯一ID。此ID在某些状态实例中用于标识状态所引用的特定 Qos 策略。

有些 QoS 策略是不可变的，这意味着只能在实体创建时或调用启用操作之前指定。

每个 DDS 实体都有一组特定的 QoS 策略，这些策略可以是标准 QoS 策略、XTypes 扩展和 eProsima 扩展的组合。

#### 3.1.2.1.标准Qos策略
本节介绍每项 DDS 标准 QoS 策略：
- 截止日期Qos策略
- 目的地顺序Qos策略
- 持久性Qos策略
- 持久性服务Qos策略
- EntityFactoryQos策略
- 组数据Qos策略
- 历史Qos策略
- 延迟预算Qos策略
- 寿命Qos策略
- 活力Qos策略
- 所有权Qos策略
- 所有权实力Qos政策
- 分区QoS策略
- 呈现Qos策略
- Reader数据生命周期Qos策略
- 可靠性Qos策略
- 资源限制Qos策略
- 基于时间的过滤Qos策略
- 主题数据Qos策略
- 传输优先Qos策略
- 用户数据Qos策略
- 写入器数据生命周期Qos策略

##### 3.1.2.1.1.截止日期(Deadline)Qos策略
当新样本的频率低于特定阈值时，此 QoS 策略会发出警报。它对于需要定期更新数据的情况非常有用（请参阅 参考资料DeadlineQosPolicy）。

在发布方面，截止日期定义了应用程序预计提供新样本的最长期限。在订阅方，它定义了应接收新样本的最长期限。

对于带有密钥的主题，此 QoS 按密钥应用。假设必须定期发布某些车辆的位置。在这种情况下，可以将车辆的ID设置为数据类型的关键字，并将最终期限QoS设置为期望的发布周期。

QoS 策略数据成员列表：
|数据成员名称|类型|默认值|
|---|---|---|
|`period`|`Duration_t`|`c_TimeInfinite`|

>**注意**
>
>该Qos策略涉及Topic、DataReader、DataWriter实体
>它可以在启动的实体上更改

>**警告**
>
>为了使 DataWriter 和 DataReader 匹配，它们必须遵循兼容性规则。有关更多详细信息，请参阅兼容性规则。

**兼容性规则**
为了保持 DataReaders 和 DataWriter 中 DeadlineQosPolicy 之间的兼容性，提供的截止时间（在 DataWriter 上配置）必须小于或等于请求的截止时间（在 DataReader 上配置），否则，实体将被视为不兼容。

DeadlineQosPolicy 必须与TimeBasedFilterQosPolicy设置一致，这意味着截止时间必须高于或等于最小间隔。

**例子**

*C++*
```cpp
DeadlineQosPolicy deadline;
//The DeadlineQosPolicy is default constructed with an infinite period.
//Change the period to 1 second
deadline.period.seconds = 1;
deadline.period.nanosec = 0;
```

##### 3.1.2.1.2.目的地顺序(DestinationOrder)Qos策略
> **警告**
>
> 此 QoS 策略将在未来版本中实施。
> 
[待补充]

##### 3.1.2.1.3.持久性(Durability)Qos策略
即使网络上没有DataReader ，DataWriter也可以在整个主题中发送消息。此外，在DataWriter写入某些数据后加入主题的 DataReader 可能有兴趣访问该信息(请参阅参考资料DurabilityQosPolicy)。

DurabilityQoSPolicy 定义了系统对于 DataReader 加入之前主题上存在的样本的行为方式。系统的行为取决于DurabilityQosPolicyKind的值。

QoS 策略数据成员列表：

|数据成员名称|类型|默认值|
|---|---|---|
|kind|DurabilityQosPolicyKind|VOLATILE_DURABILITY_QOS for DataReaders TRANSIENT_LOCAL_DURABILITY_QOS for DataWriters|



