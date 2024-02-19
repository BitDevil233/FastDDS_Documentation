# 1.从二进制文件安装Linux
本页提供了在Linux环境中通过二进制文件安装eProsima Fast DDS的说明。
- 安装
- 目录
- 运行应用程序
- 在CMake项目中包含Fast DDS
- 卸载

## 1.1 安装
eProsima Fast DDS for Linux的最新版本可在eProsisma网站的下载选项卡上找到。下载后，提取首选目录中的内容。然后，要在系统中安装eProsima Fast DDS及其所有依赖项，请使用管理权限执行install.sh脚本：
```shell
cd <extraction_directory>
sudo ./install.sh
```
**`注意`**

> 默认情况下，eProsima Fast DDS不编译测试模块。要激活测试模块，请参阅“从源代码安装Linux”页面。
> 要使用xml验证工具，请参阅“从源代码安装Linux”页面。

### 1.1.1内容
src文件夹包含以下包：
- **foonathan_memory_fildor**，一个STL兼容的C++内存分配器库。
- **fastcdr**，一个根据CDR标准进行数据序列化的C++库（第10.2.1.2节OMG CDR）。
- **fastrtps**，eProsima Fast DDS库的核心库。
- **fastddsgen**，一个Java应用程序，它使用IDL文件中定义的数据类型生成源代码。

如果这些组件中的任何一个是不需要的，可以简单地将其重命名或从src目录中删除。

### 1.1.2.运行应用程序
当使用eProsima Fast DDS运行应用程序实例时，它必须与安装软件包的库/usr/local/lib/链接。有两种可能性：

通过在用于运行eProsima Fast DDS实例的控制台中键入以下命令，在本地准备环境：
```shell
export LD_LIBRARY_PATH=/usr/local/lib/
```
通过执行以下操作将其永久添加到PATH中：
```shell
echo 'export LD_LIBRARY_PATH=/usr/local/lib/' >> ~/.bashrc
```

### 1.1.3.在CMake项目中包含 Fast DDS
安装程序部署CMake配置文件，通过find_package CMake API将Fast-DDS合并到任何CMake项目中。

通过设置CMake变量BUILD_SHARED_LIBS，可以在CMake生成器阶段选择所需的链接（动态或静态库）。如果变量丢失，则生成过程将默认为静态链接。

例如，为了构建动态链接到Fast DDS的示例，请执行以下操作：
```shell
$cmake -Bbuildexample -DBUILD_SHARED_LIBS=ON .
$cmake --build buildexample --target install
```

## 1.2.卸载
要卸载所有已安装的组件，请执行uninstall.sh脚本（使用管理权限）：
```shell
cd ＜extraction_directory＞
sudo ./uninstall.sh
```

`警告`

>如果任何其他组件已经以其他方式安装在系统中，它们也将被删除。为了避免这种情况，请在执行脚本之前对其进行编辑。

# 2. Windows平台下以二进制方式安装
TO DO

# 3. Linux平台下以源码方式安装
TO DO

# 4. Windows平台下以源码方式安装
TO DO

# 5. Mac OS 下以源码方式安装
TO DO

# 6. QNX 下以源码方式安装
TO DO

# 7. CMake选项
eProsima Fast DDS提供了许多CMake选项，用于更改Fast DDS的行为和配置。这些选项允许用户通过在CMake执行时将这些选项定义为ON/OFF来启用/禁用某些快速DDS设置。本节的结构如下：首先，描述了Fast DDS的一般配置的CMake选项；然后，提出了与第三方库相关的选择；最后，定义了构建FastDDS测试的可能选项。

## 7.1. 一般选项
下面显示了用于配置常规设置的Fast DDS CMake选项，以及它们的描述和对其他选项的依赖关系。









