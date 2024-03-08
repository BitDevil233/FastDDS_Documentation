# 入门
本章定义了DDS（Data Distribution Service）和RTPS（The Real-time Publish-Subscribe Protocol）。同时提供了一个编写简单的Fast DDS (formerly Fast RTPS) 应用程序的手把手教程。
## 1.1什么是DDS？
数据分发服务（DDS）是一种以数据为中心的通信协议，用于分布式软件应用的通信。它描述了通信应用程序的编程接口和允许数据提供者和数据使用者之间通信的通信语义。  
DDS是一个以数据为中心的发布订阅模型，在它的实现中定义了三个关键的应用程序实体：发布实体。发布实体定义了信息生产对象和这些对象的属性。订阅实体。订阅实体定义了信息消费对象和这些对象的属性。配置实体。配置实体定义了作为主题传输的信息类型，并使用其服务质量策略（QoS）属性来创建发布者和订阅者，确保上述实体的功能正常。  
DDS使用QoS来定义DDS实体的行为特征。QoS由单个QoS策略（继承自QoSPolicy的派生类）组成。这些都在QoS策略中进行了说明。
### 1.1.1数据中心发布订阅（DCPS）概念模型
在DCPS模型中，为开发通信应用程序系统定义了四个基本的要素。
- Publisher（发布器）。发布器是DCPS的实体，负责数据写入器（DataWriter）的创建和配置。数据写入器是用于消息实际发布的实体。每个数据写入器都有一个指定的主题（Topic）  
Sub- scriber（订阅器）。订阅器是DCPS的实体，负责接受其在订阅主题下发布的数据。订阅器服务于一个或多个数据读取器（DataReader）对象，这些对象负责将新数据传送给应用程序。  
- Topic（主题）。它是绑定发布和订阅的实体，在DDS域中是唯一的。通过主题的描述，保证了发布和订阅数据类型的一致性。  
- Domain（域）。域用于链接所有的发布器和订阅器。这些发布订阅器属于一个或者多个应用程序，它们在不同的主题下进行数据交换。这些在域中的单个应用程序称为域参与者（DomainParticipant）。DDS域由域ID（Domain ID）标识，域参与者定义域ID来指定它所属的DDS域。如果两个域参与者的域ID不同，那么它们不知道对方在网络中的存在。因此，可以创建多个通信通道。这适用于多个DDS应用程序的场景，应用程序各自的域参与者可以相互通信，但是这些DDS应用程序之间不能相互干涉。域参与者是其它DCPS实体的容器，是发布器、订阅器和主题的工厂，并在域中提供管理服务。有关更多详细信息，请参阅域。

这些元素如下图所示。  
![DDS域中的DCPS模型实体](https://fast-dds.docs.eprosima.com/en/latest/_images/dds_domain.svg)
## 1.2. 什么是RTPS

实时发布订阅（Real-Time Publish Subscribe, RTPS）协议是为支持DDS应用而开发的一种在UDP/IP等有效传输协议上的发布订阅通信中间件。此外，FsatDDS提供了对TCP和共享内存传输的支持。

实时发布订阅旨在支持单播和多播通信。  

RTPS的顶层承袭自DDS，它也有一个RTPS域，定义了一个单独的通信平面。几个RTPS域可以独立共存。RTPS域可以有任意数量的域参与者，因而具备收发数据的能力。为此，RTPS参与者使用它们的Endpoint（端点）：
- RTPSWriter：可以发送数据的端点。
- RTPSReader：可以接收数据的端点。
RTPSParticipant可以有任意数量的写入端点和读取端点。
  
通信围绕Topic，Topic定义和标记了正在交换的数据。Topic不属于特定的Participant。Participant通过RTPSWriter对在Topic下发布的数据进行更改，并通过RTPSReader接收与其订阅的Topic关联的数据。通信单元称为Change，它表示写入Topic下的数据的更新。RTPSReaders/RTPSWriters在其历史记录上注册这些通信单元（change），它是一种作为一种缓存的数据结构，为最新更改服务的。
在eProsima FastDDS的默认配置中，当您通过RTPSWriter端点发布更改时，背后会执行这些步骤：
1. 通信单元被添加到RTPSWriter的历史纪录缓存中。
2. RTPSWriter将通信单元发送到它所知道的任何RTPSReaders。
3. RTPSReader接受数据后，使用新的通信单元更改其历史缓存。  

FastDDS支持许多配置，允许您更改RTPSWriters/RTPSReaders的行为。对RTPS实体默认配置进行修改意味着RTPSWriters和RTPSReaders之间的数据交流进行了变化。此外，通过选择QoS，您可以用多种方式来更改这些历史缓存的管理方式，但是通信循化是保持不变的。


## 1.3 写一个简单的C++发布器和订阅器应用程序
本章详细介绍了如何使用C++API创建一个简单的FastDDS应用程序。还可以使用eProsiam Fast-Gen工具自动生成一个与本节中实现的示例相似的例子。在构建发布/订阅应用程序中解释了这种附加方法。


### 1.3.1 背景
DDS是实现DCPS模型的以数据为中心的通信中间件。该模型基于发布器（用于数据生成）和订阅器（用于数据消费）的开发。这些实体是通过Topic进行通信，Topic是绑定两个DDS实体的元素。发布器在Topic下生成信息，订阅器在同一Topic来接收信息。

### 1.3.2 先决条件
首先，您需要按照安装手册中概述的步骤安装eProsima FastDDS及其所有依赖项。您还需要完成安装手册概述的步骤来安装eProsima FastDDS-Gen工具。此外，本教程中提供的所有命令都是针对Linux环境的。

### 1.3.3 创建应用程序工作区
应用程序工作区在项目结束时将具有以下结构。文件 build/DDSHelloWorldPublisher和 build/DDSHelloWorldSubscriber分别是发布器应用程序和订阅器应用程序。
```shell
.
└── workspace_DDSHelloWorld
    ├── build
    │   ├── CMakeCache.txt
    │   ├── CMakeFiles
    │   ├── cmake_install.cmake
    │   ├── DDSHelloWorldPublisher
    │   ├── DDSHelloWorldSubscriber
    │   └── Makefile
    ├── CMakeLists.txt
    └── src
        ├── HelloWorld.cxx
        ├── HelloWorld.h
        ├── HelloWorld.idl
        ├── HelloWorldCdrAux.hpp
        ├── HelloWorldCdrAux.ipp
        ├── HelloWorldPublisher.cpp
        ├── HelloWorldPubSubTypes.cxx
        ├── HelloWorldPubSubTypes.h
        └── HelloWorldSubscriber.cpp
```

让我们先创建目录树。
```shell
    mkdir workspace_DDSHelloWorld && cd workspace_DDSHelloWorld
    mkdir src 
```

### 1.3.4 导入连接库及其依赖项
DDS应用程序需要Fast DDS和Fast CDR库。根据安装程序的不同，这些库可用于我们DDS应用程序的过程将略有不同。

#### 1.3.4.1 从二进制文件安装以及手动安装
如果我们已经通过二进制文件安装或者手动安装，那么这些库已经可以从工作区访问了。在Linux系统上，头文件可以分别在Fast DDS和Fast CRD的目录/user/include/fastrtps和/user/include/fastcdr中找到

#### 1.3.4.2 安装Colcon工具
从Colcon安装中导入库的方式有几种。如果这些库只需要当前可用，请运行以下命令：
```shell
    source <path/to/Fast-DDS/workspace>/install/setup.bash
```
通过将FastDDS安装目录添加到Shell配置文件中的环境变量，可以从任何会话中访问它们。

```shell
    echo 'source <path/to/Fast-DDS/workspace>/install/setup.bash' >> ~/.bashrc
```
这将在每个用户登陆后设置环境。

### 1.3.5 配置CMake项目
我们将使用CMake工具来管理项目的构建。使用首选的文本编辑器，创建一个名为CMakelists的新文件，复制并粘贴以下代码片段。将此文件保存在工作区的根目录中，如果遵循了这些步骤，那么它应该是workspace_DDSHelloWorld。
```cmake
    cmake_minimum_required(VERSION 3.20)

    project(DDSHelloWorld)
    #find requirements
    if(NOT fastcdr_FOUND)
        find_package(fastcdr 2 REQUIRED)
    endif()

    if(NOT fastrtps_FOUND)
        find_package(fastrtps 2.12 REQUIRED)
    endif()

    #set C++ 11
    include(CheckCXXCompilerFlag)
    if(CMAKE_COMPILER_IS_GUNCXX OR CMAKE_COMPILER_IS_CLANG OR
            CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        check_cxx_compiler_flag(-std=c++11 SUPPORTS_CXX11)
        if(SUPPORTS_CXX11)
            add_compile_options(-std=c++11)
        else()
            message(FATAL_ERROR "Compiler does't support C++11")
        endif()
    endif()
    message(STATUS "Configuring HelloWorld publisher/subscriber example...")
    file(GLOB DDS_HELLOWORLD_SOURCES_CXX "src/*.cxx")
```
在每个部分中，我们将完成这个文件，以包括特定生成的文件。

### 1.3.6 生成Topic数据类型
eProsima Fast DDS-Gen是一个Java应用程序，它使用接口描述语言（IDL）文件中定义的数据类型生成源代码。这个应用程序可以做两件事情：
1.为自定义的Topic生成C++数据类型定义
2.生成一个使用Topic数据的函数示例。
本教程将实现前者。对于这个项目，我们将使用Fast DDS-Gen应用程序来定义发布器和订阅器接收消息的数据类型。
在工作区目录中，执行以下命令：
```shell
    cd src && touch HelloWorld.idl
```
这将在src目录中创建HelloWorld.idl文件。在文本编辑器中打开该文件，并复制粘贴以下代码片段：
```cpp
    struct HelloWorld
    {
        unsigned long index;
        string message;
    };
```
通过这样做，我们定义了HelloWorld数据类型，它有两个元素：uint32_t类型的index和std::string类型的message。剩下的工作就是生成在C++11中实现这种数据类型的源代码。为此，从src目录运行以下命令。
```shell
    <path/to/Fast DDS-Gen>/scripts/fastddsgen HelloWorld.idl
```
这将生成以下文件：
- HelloWorld.cxx:HelloWorld 类型定义
- Helloorld.h：HelloWorld.cxx的头文件
- HelloWorldPubSubTypes.cxx:接口，用于支持HelloWorld类型
- HelloWorldPubSubTypes.h：HelloWorldPubSubTypes.cxx的头文件
- HelloWorldCdrAux.ipp:HelloWorld类型的序列化和反序列化代码
- HelloWorldCdrAux.hpp:HelloWorldCdrAux.ipp的头文件

#### 1.3.6.1 CmakeLists.txt
在前面创建的CmakeLists.txt文件的末尾包含以下代码片段，这将包含我们刚刚创建的文件。
```cmake
    target_link_libraries(DDSHelloWorldPublisher fastrtps fastcdr)
```

### 1.3.7 编写FastDDS发布器
在工作区的src目录中，运行以下命令下载HelloWorldPublisher.cpp文件。

```shell
wget -0 HelloWorldPublisher.cpp \ https://raw.githubusercontent.com/eProsima/Fast-RTPS-docs/master/code/Examples/C++/DDSHelloWorldPublisher.cpp
```
这是发布器应用程序的C++源码。它将以HelloWorldTopic为topic发送10份出版物。

--长代码

#### 1.3.7.1 查看和分析代码的过程
在文件的开头，我们有一个Doxygen样式的注释块，其中@file字段告诉我们文件名。
```cpp
    /**
     * @file HelloWorldPublisher.cpp
     *
     */
```
下面是C++头的内容。第一个包含HelloWorldPubSubTypes.h文件，这个文件包含我们在上一节中定义的数据类型的序列化和反序列化函数。
下面一个代码块包括允许使用FastDDS API的C++头文件。
- DomainParticipantFactory:允许创建和销毁domainparticipant对象
- DomainParticipant:充当所有其它实体对象
- DomainParticipantFactory:允许创建和销毁domainparticipant对象
- DomainParticipant:充当所有其它实体对象
- TypeSupport:为Participant提供序列化、反序列化和获取特定数据类型的键的函数。
- Publisher：它是负责创建datawriter的对象。
- DataWriter：允许应用程序设置要在给定Topic下发布的数据的值。
- DataWriterListener：允许重新定义DataWriterListener的函数。
```cpp
    #include <chrono>
    #include <thread>

    #include <fastdds/dds/domain/DomainParticipant.hpp>
    #include <fastdds/dds/domain/DomainParticipantFactory.hpp>
    #include <fastdds/dds/publisher/DataWriter.hpp>
    #include <fastdds/dds/publisher/DataWriterListener.hpp>
    #include <fastdds/dds/publisher/Publisher.hpp>
    #include <fastdds/dds/topic/TypeSupport.hpp>
```
接下来，定义包含我们将在应用程序中使用的eProsima Fast DDS类和函数的名称空间。
```cpp
    using namespace eprosima::fastdds::dds;
```
下一行创建实现发布者的HelloWorldPublisher类。
```cpp
    class HelloWorldPublisher
```
继续讨论类的私有数据成员，hello_数据成员被定义为HelloWorld类的对象，该类定义了我们用IDL文件创建的数据类型。接下来，定义与参与者、发布者、主题、DataWriter和数据类型相对应的私有数据成员。TypeSupport类的type_对象是将用于在DomainParticipant中注册主题数据类型的对象。
```cpp
private:

    HelloWorld hello_;

    DomainParticipant* participant_;

    Publisher* publisher_;

    Topic* topic_;

    DataWriter* writer_;

    TypeSupport type_;
```
然后，通过继承DataWriterListener类来定义PubListener类。该类重载默认的DataWriter侦听器回调函数，允许在发生事件时执行例程。重载的的回调函数on_publication_matched()允许在检测到新DataReader在侦听（DataWriter发布的Topic）时定义一系列操作。info.current_count_change()用于检测与DataWriter匹配的DataReader的这些更改。这是MatchedStatus结构体中的一个成员，它允许跟踪状态的变化。最后，类的listener_对象被定义为PubListener的一个实例。
```cpp
class PubListener : public DataWriterListener
{
public:

    PubListener()
        : matched_(0)
    {
    }

    ~PubListener() override
    {
    }

    void on_publication_matched(
            DataWriter*,
            const PublicationMatchedStatus& info) override
    {
        if (info.current_count_change == 1)
        {
            matched_ = info.total_count;
            std::cout << "Publisher matched." << std::endl;
        }
        else if (info.current_count_change == -1)
        {
            matched_ = info.total_count;
            std::cout << "Publisher unmatched." << std::endl;
        }
        else
        {
            std::cout << info.current_count_change
                    << " is not a valid value for PublicationMatchedStatus current count change." << std::endl;
        }
    }

    std::atomic_int matched_;

} listener_;
```
HelloWorldPublisher类的公共构造函数和析构函数定义如下。构造函数将类的私有数据成员初始化为nullptr，但TypeSupport对象例外，它被初始化为HelloWorldPubSubType类的实例。类析构函数删除这些成员变量，从而清理系统内存。
```cpp
    HelloWorldPublisher()
        : participant_(nullptr)
        , publisher_(nullptr)
        , topic_(nullptr)
        , writer_(nullptr)
        , type_(new HelloWorldPubSubType())
    {
    }

    virtual ~HelloWorldPublisher()
    {
        if (writer_ != nullptr)
        {
            publisher_->delete_datawriter(writer_);
        }
        if (publisher_ != nullptr)
        {
            participant_->delete_publisher(publisher_);
        }
        if (topic_ != nullptr)
        {
            participant_->delete_topic(topic_);
        }
        DomainParticipantFactory::get_instance()->delete_participant(participant_);
    }
```
继续看HelloWorldPublisher类的公共成员函数，下一段代码定义公共发布者的初始化成员函数。这个函数执行几个动作:
1. 初始化HelloWorld类型hello_结构成员的内容。 
2. 通过DomainParticipant的QoS为Participant分配一个名称。
3. 使用DomainParticipantFactory创建Participant。
4. 注册IDL中定义的数据类型。 
5. 为发布创建topic。
6. 创建publisher。
7. 使用先前创建的listener创建DataWriter。
可以看到，除了participant的名字之外，所有实体的QoS配置都是默认配置(PARTICIPANT_QOS_DEFAULT, PUBLISHER_QOS_DEFAULT, TOPIC_QOS_DEFAULT, DATAWRITER_QOS_DEFAULT)。每个DDS实体的QoS的缺省值可以在DDS标准中查看。
```cpp
    //!Initialize the publisher
bool init()
{
    hello_.index(0);
    hello_.message("HelloWorld");

    DomainParticipantQos participantQos;
    participantQos.name("Participant_publisher");
    participant_ = DomainParticipantFactory::get_instance()->create_participant(0, participantQos);

    if (participant_ == nullptr)
    {
        return false;
    }

    // Register the Type
    type_.register_type(participant_);

    // Create the publications Topic
    topic_ = participant_->create_topic("HelloWorldTopic", "HelloWorld", TOPIC_QOS_DEFAULT);

    if (topic_ == nullptr)
    {
        return false;
    }

    // Create the Publisher
    publisher_ = participant_->create_publisher(PUBLISHER_QOS_DEFAULT, nullptr);

    if (publisher_ == nullptr)
    {
        return false;
    }

    // Create the DataWriter
    writer_ = publisher_->create_datawriter(topic_, DATAWRITER_QOS_DEFAULT, &listener_);

    if (writer_ == nullptr)
    {
        return false;
    }
    return true;
}
```
为了发布，实现了公共成员函数publish()。在DataWriter的侦听器（listener）回调中(声明DataWriter已经与侦听发布主题的DataReader匹配)，数据成员matched_被更新。它包含发现的datareader的数量。因此，当发现第一个DataReader时，应用程序开始发布。这只是通过DataWriter对象写入更改。
```cpp
    //!Send a publication
bool publish()
{
    if (listener_.matched_ > 0)
    {
        hello_.index(hello_.index() + 1);
        writer_->write(&hello_);
        return true;
    }
    return false;
}
```
公有的run函数执行发布给定次数的操作，在发布之间等待1秒。
```cpp
    //!Run the Publisher
void run(
        uint32_t samples)
{
    uint32_t samples_sent = 0;
    while (samples_sent < samples)
    {
        if (publish())
        {
            samples_sent++;
            std::cout << "Message: " << hello_.message() << " with index: " << hello_.index()
                        << " SENT" << std::endl;
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
}
```
最后，初始化HelloWorldPublisher并在main函数中运行。
```cpp
    int main(
        int argc,
        char** argv)
{
    std::cout << "Starting publisher." << std::endl;
    uint32_t samples = 10;

    HelloWorldPublisher* mypub = new HelloWorldPublisher();
    if(mypub->init())
    {
        mypub->run(samples);
    }

    delete mypub;
    return 0;
}
```
#### 1.3.7.2. CMakeLists.txt
在前面创建的CMakeList.txt文件的末尾包含以下代码片段。这将添加构建可执行文件所需的所有源文件，并将可执行文件和库链接在一起。
```cmake
    add_executable(DDSHelloWorldPublisher src/HelloWorldPublisher.cpp ${DDS_HELLOWORLD_SOURCES_CXX})
    target_link_libraries(DDSHelloWorldPublisher fastrtps fastcdr)
```
此时，项目已准备好构建、编译和运行发布者应用程序。在工作区的构建目录中，运行以下命令。
```shell
    cmake ..
    cmake --build .
    ./DDSHelloWorldPublisher
```
1.3.8. 编写Fast DDS subscriber
从工作区的src目录中，执行以下命令下载HelloWorldSubscriber.cpp文件。
```shell
wget -O HelloWorldSubscriber.cpp \
    https://raw.githubusercontent.com/eProsima/Fast-RTPS-docs/master/code/Examples/C++/DDSHelloWorld/src/HelloWorldSubscriber.cpp
```
这是订阅器应用程序的c++源代码。应用程序运行订阅器，直到在主题HelloWorldTopic下收到10个示例。此时，订阅器停止。
```cpp
    //长代码
```
#### 1.3.8.1. 代码检查
由于Publisher和subscriber应用程序的源代码在很大程度上是相同的，因此本文将重点讨论它们之间的主要区别，省略已经解释过的代码部分。按照与Publisher解释中相同的结构，第一步是包含c++头文件。在这些文件中，包含publisher类的文件由subscriber类替换，而包含datawriter类的文件由datareader类替换。
- subscriber：它是负责创建和配置datareader的对象。
- DataReader：它是负责实际接收数据的对象。它在应用程序中注册topic(TopicDescription)，该topic标识要读取的数据并访问订阅者接收到的数据。 
- DataReaderListener：这是分配给data reader的listener。
- DataReaderQoS：结构体，该结构体定义了DataReader的QoS。
- SampleInfo：“读取”或“采集”的是每个样本附带的信息。
```cpp
    #include <fastdds/dds/domain/DomainParticipant.hpp>
    #include <fastdds/dds/domain/DomainParticipantFactory.hpp>
    #include <fastdds/dds/subscriber/DataReader.hpp>
    #include <fastdds/dds/subscriber/DataReaderListener.hpp>
    #include <fastdds/dds/subscriber/qos/DataReaderQos.hpp>
    #include <fastdds/dds/subscriber/SampleInfo.hpp>
    #include <fastdds/dds/subscriber/Subscriber.hpp>
    #include <fastdds/dds/topic/TypeSupport.hpp>
```
下一行定义了实现subscriber的HelloWorldSubscriber类。
```cpp
    class HelloWorldSubscriber
```
从类的私有成员变量开始，值得一提的是data reader的listener实现。类的私有成员变量将是participant、subscriber、topic、data reader和data type。与data writer的情况一样，listener实现在事件发生时要执行的回调函数。SubListener的第一个被覆写的回调函数是on_subscription_matched()，它类似于DataWriter的on_publication_matched()回调函数。
```cpp
    void on_subscription_matched(
        DataReader*,
        const SubscriptionMatchedStatus& info) override
{
    if (info.current_count_change == 1)
    {
        std::cout << "Subscriber matched." << std::endl;
    }
    else if (info.current_count_change == -1)
    {
        std::cout << "Subscriber unmatched." << std::endl;
    }
    else
    {
        std::cout << info.current_count_change
                << " is not a valid value for SubscriptionMatchedStatus current count change" << std::endl;
    }
}
```
第二个被重写的回调函数是on_data_available()。在这种情况下，data reader可以访问的下一个接收样本被获取，并被处理以显示样本内容。在这里定义了SampleInfo类的对象，它确定是否已经读取或获取了样本。每次读取一个样本，接收到的样本计数器就会增加。
```cpp
    void on_data_available(
        DataReader* reader) override
{
    SampleInfo info;
    if (reader->take_next_sample(&hello_, &info) == ReturnCode_t::RETCODE_OK)
    {
        if (info.valid_data)
        {
            samples_++;
            std::cout << "Message: " << hello_.message() << " with index: " << hello_.index()
                        << " RECEIVED." << std::endl;
        }
    }
}
```
类的公有构造函数和析构函数定义如下。
```cpp
    HelloWorldSubscriber()
    : participant_(nullptr)
    , subscriber_(nullptr)
    , topic_(nullptr)
    , reader_(nullptr)
    , type_(new HelloWorldPubSubType())
{
}

virtual ~HelloWorldSubscriber()
{
    if (reader_ != nullptr)
    {
        subscriber_->delete_datareader(reader_);
    }
    if (topic_ != nullptr)
    {
        participant_->delete_topic(topic_);
    }
    if (subscriber_ != nullptr)
    {
        participant_->delete_subscriber(subscriber_);
    }
    DomainParticipantFactory::get_instance()->delete_participant(participant_);
}
```
接下来是subscriber初始化公有成员函数。这与为HelloWorldPublisher定义的初始化公有成员函数相同。除participant’s name外，所有实体的QoS配置均为默认QoS (PARTICIPANT_QOS_DEFAULT、SUBSCRIBER_QOS_DEFAULT、TOPIC_QOS_DEFAULT、DATAREADER_QOS_DEFAULT)。每个DDS实体的QoS的缺省值可以在DDS标准中查看。
```cpp
    //!Initialize the subscriber
bool init()
{
    DomainParticipantQos participantQos;
    participantQos.name("Participant_subscriber");
    participant_ = DomainParticipantFactory::get_instance()->create_participant(0, participantQos);

    if (participant_ == nullptr)
    {
        return false;
    }

    // Register the Type
    type_.register_type(participant_);

    // Create the subscriptions Topic
    topic_ = participant_->create_topic("HelloWorldTopic", "HelloWorld", TOPIC_QOS_DEFAULT);

    if (topic_ == nullptr)
    {
        return false;
    }

    // Create the Subscriber
    subscriber_ = participant_->create_subscriber(SUBSCRIBER_QOS_DEFAULT, nullptr);

    if (subscriber_ == nullptr)
    {
        return false;
    }

    // Create the DataReader
    reader_ = subscriber_->create_datareader(topic_, DATAREADER_QOS_DEFAULT, &listener_);

    if (reader_ == nullptr)
    {
        return false;
    }

    return true;
}
```
公有成员函数run()确保subscriber一直运行，直到接收到所有sample。该成员函数实现subscriber的活动等待，并使用100ms的睡眠间隔来缓解CPU的压力。
```cpp
    //!Run the Subscriber
void run(
    uint32_t samples)
{
    while(listener_.samples_ < samples)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}
```
最后，实现subscriber的participant被初始化并在main中运行。
```cpp
    int main(
        int argc,
        char** argv)
{
    std::cout << "Starting subscriber." << std::endl;
    uint32_t samples = 10;

    HelloWorldSubscriber* mysub = new HelloWorldSubscriber();
    if(mysub->init())
    {
        mysub->run(samples);
    }

    delete mysub;
    return 0;
}
```
#### 1.3.8.2. CMakeLists.txt
在前面创建的CMakeList.txt文件的末尾包含以下代码片段。这将添加构建可执行文件所需的所有源文件，并将可执行文件和库链接在一起。
```cmake
    add_executable(DDSHelloWorldSubscriber src/HelloWorldSubscriber.cpp ${DDS_HELLOWORLD_SOURCES_CXX})
    target_link_libraries(DDSHelloWorldSubscriber fastrtps fastcdr)
```
此时，项目已准备好构建、编译和运行Subscriber应用程序。在工作区的构建目录中，运行以下命令。
```shell
    cmake ..
    cmake --build .
    ./DDSHelloWorldSubscriber
```
### 1.3.9. 综上
最后，在构建目录中，从两个终端运行publisher和subscriber应用程序。
```shell
    ./DDSHelloWorldPublisher
    ./DDSHelloWorldSubscriber
```
### 1.3.10. 总结
在本教程中，您构建了一个publisher和一个subscriber DDS应用程序。您还学习了如何构建用于源代码编译的CMake文件，以及如何在项目中包含和使用Fast DDS和Fast CDR库。
### 1.3.11. 下一个步骤
在eProsima Fast DDS Github存储库中，您将找到更复杂的示例，这些示例为大量用例和场景实现了DDS通信。你可以在这里找到它们。











        








    