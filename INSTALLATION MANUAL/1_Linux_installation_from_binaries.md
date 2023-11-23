# 1.从二进制文件安装Linux
本页提供了在Linux环境中通过二进制文件安装eProsima Fast DDS的说明。
- 安装
- 目录
- 运行应用程序
- 在CMake项目中包含Fast DDS
- 卸载

## 1.1 安装
eProsima Fast DDS for Linux的最新版本可在eProsisma网站的下载选项卡上找到。下载后，提取首选目录中的内容。然后，要在系统中安装eProsima Fast DDS及其所有依赖项，请使用管理权限执行install.sh脚本：
```
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



