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
    