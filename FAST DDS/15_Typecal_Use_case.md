## 15.8 Fast DDS over TCP
### 15.8.1 使用多播发现的TCP/SHM通信
以下片段显示了如何配置Fast DDS DomainParticipants以通过UDP多播运行`PDP` `发现阶段`，并通过TCP传输协议传输应用程序数据。使用这种方法，管理大样本的应用程序可以通过TCP或SHM传输数据，同时具有自动发现的灵活性。
```
在FATDDS中，"发现阶段"指的是一种机制，用于在网络中发现新加入的数据发布者和订阅者。当一个新的数据发布者或订阅者加入到网络中时，它会发送一个特定的多播消息，该消息包含有关其自身及其所需数据的信息。其他节点可以接收并处理这些消息，从而了解新节点的存在和其需求。

通过"发现阶段"，FATDDS可以自动地建立数据发布者和订阅者之间的连接，从而实现数据的可靠传输。同时，这一机制还可以处理网络中节点的故障和断开连接等问题，保证数据的稳定传输和可靠性。
```
```
在FATDDS中，PDP是指"Participant Discovery Protocol"，即"参与者发现协议"。它是FATDDS中的一部分，用于在网络中发现新加入的数据发布者和订阅者，并为它们建立连接。PDP使用UDP多播消息进行通信，可以快速地实现节点的发现和连接建立。

当一个新的数据发布者或订阅者加入到网络中时，它会发送一个特定的PDP多播消息，该消息包含有关其自身及其所需数据的信息。其他节点可以接收并处理这些消息，从而了解新节点的存在和其需求。同时，这些节点也会向新节点发送PDP消息，告知它们自己的存在和需求，从而建立连接。

通过PDP协议，FATDDS可以自动地建立数据发布者和订阅者之间的连接，从而实现数据的可靠传输。同时，PDP协议还可以处理网络中节点的故障和断开连接等问题，保证数据的稳定传输和可靠性。
```

```cpp
eprosima::fastdds::dds::DomainParticipantQos pqos = PARTICIPANT_QOS_DEFAULT;

/* Transports configuration */
// UDPv4 transport for PDP over multicast and SHM / TCPv4 transport for EDP and application data
pqos.setup_transports(eprosima::fastdds::rtps::BuiltinTransports::LARGE_DATA);

/* Create participant as usual */
eprosima::fastdds::dds::DomainParticipant* participant =
        eprosima::fastdds::dds::DomainParticipantFactory::get_instance()->create_participant(0, pqos);

```
`注意`
> 内置传输的`LARGE_DATA`配置也将沿着UDP和TCP传输创建SHM传输。只要有可能，就会使用共享内存。如果在SHM可行的情况下需要TCP通信，则需要手动配置

```cpp
eprosima::fastdds::dds::DomainParticipantQos pqos = PARTICIPANT_QOS_DEFAULT;

/* Transports configuration */
// UDPv4 transport for PDP over multicast
auto pdp_transport = std::make_shared<eprosima::fastdds::rtps::UDPv4TransportDescriptor>();
pqos.transport().user_transports.push_back(pdp_transport);

// TCPv4 transport for EDP and application data (The listening port must to be unique for
// each participant in the same host)
constexpr uint16_t tcp_listening_port = 0;
auto data_transport = std::make_shared<eprosima::fastdds::rtps::TCPv4TransportDescriptor>();
data_transport->add_listener_port(tcp_listening_port);
pqos.transport().user_transports.push_back(data_transport);

pqos.transport().use_builtin_transports = false;

/* Locators */
// Define locator for PDP over multicast
eprosima::fastrtps::rtps::Locator_t pdp_locator;
pdp_locator.kind = LOCATOR_KIND_UDPv4;
eprosima::fastrtps::rtps::IPLocator::setIPv4(pdp_locator, "239.255.0.1");
pqos.wire_protocol().builtin.metatrafficMulticastLocatorList.push_back(pdp_locator);

// Define locator for EDP and user data
eprosima::fastrtps::rtps::Locator_t tcp_locator;
tcp_locator.kind = LOCATOR_KIND_TCPv4;
eprosima::fastrtps::rtps::IPLocator::setIPv4(tcp_locator, "0.0.0.0");
eprosima::fastrtps::rtps::IPLocator::setPhysicalPort(tcp_locator, tcp_listening_port);
eprosima::fastrtps::rtps::IPLocator::setLogicalPort(tcp_locator, tcp_listening_port);
pqos.wire_protocol().builtin.metatrafficUnicastLocatorList.push_back(tcp_locator);
pqos.wire_protocol().default_unicast_locator_list.push_back(tcp_locator);

/* Create participant as usual */
eprosima::fastdds::dds::DomainParticipant* participant =
        eprosima::fastdds::dds::DomainParticipantFactory::get_instance()->create_participant(0, pqos);
```
