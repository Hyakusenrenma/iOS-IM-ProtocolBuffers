# iOS-IM-ProtocolBuffers
iOS 集成Protocol Buffers并基于GCDAsyncSocket即时通讯。

-------
因为项目中需要自己搭建一套IM而不使环信等第三方服务，和后台伙伴讨论最后决定使用Google的[ProtocolBuffers](https://github.com/protocolbuffers/protobuf)序列化聊天内容并基于GCDAsyncSocket即时通讯。
本文主要两块内容，**配置Protocol Buffers**和**使用GCDAsyncSocket实现即时通讯**
# 配置ProtocolBuffers
### 啥是ProtocolBuffers
Protobuf是一个开源项目，Google开发的，官方给出的定义和描述：
>protocol buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

>Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。

>你可以定义数据的结构，然后使用特殊生成的源代码轻松的在各种数据流中使用各种语言进行编写和读取结构数据。你甚至可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。

简单的来说，Protobuf是结构数据序列化方法，干的事其实和我们平时用的Json差不多。
Protobuf基于自己的数据格式规范，可以转换成不同平台的数据结构类型。总结官方描述就是以下三个特点：
* 与语言平台无关，支持多种语言（具体支持哪些[这里](https://github.com/protocolbuffers/protobuf)都有），本文Objective-C。
* 灵活高效，开销小很多，这也是这次为啥选用这个的主要原因
* 兼容性好，后期加东西不破坏原有结构

### 配置步骤
1. 下载protobuf
2. 生成.proto文件->转换成Objective C文件
3. 集成到项目中

#### 下载
在这里[ProtocolBuffers](https://github.com/protocolbuffers/protobuf)
#### 生成.proto文件->转换成Objective C文件
##### 生成.proto
用文本编辑器新建一个文件比如叫 person.proto
格式选择制作纯文本。
按照[官方标准语法](https://developers.google.com/protocol-buffers/)编写通用代码。

```
syntax = "proto2";

package protocol;

option java_package = "com.crossoverjie.cim.common.protocol";
option java_outer_classname = "CIMRequestProto";

message Person {
  required int64 requestId = 2;
  required string name = 1;
  required int32 age = 3;

}
```
##### 转换成Objective C文件

首先环境安装
需要HomeBrew，如果没有先安装这个。

安装protoc 编译工具。那么需要进行一下几个步骤
使用终端 cd到下载下来的protobuf-master目录
依次执行

`./autogen.sh`

`./configure`

`make`

`make install`

如果编译顺利的话，就可以使用protoc命令将.proto文件转换为不同语言的代码。

然后在.proto所在目录执行

`protoc --proto_path=... --objc_out=... XXX.proto`

其中proto_path是我们创建的proto文件所在目录，objc_out为Objective-C文件输出路径,XXX.proto是我们创建的proto文件，可以一次转换多个proto文件，加在XXX.proto后面即可。

举例：我们在src目录下新建两个文件夹 OCC 和protocols文件夹，gen为输出目录，protocols用于存放proto文件，将创建的Person.proto放在protocols文件夹下，执行命令

`protoc --proto_path=protocols --objc_out=OCC protocols/Person.proto`
然后在gen文件夹下就会生成Person.pbobjc.h和Person.pbobjc.m文件。

如果一切顺利的话，现在应该已经生成了这两个文件
![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image2.png)

#### 集成到项目中
1. 在自己项目工程中按照自己的习惯创建个文件
我命名为Lib->protocolbuf
![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image8.png)
将下载的protobuf-master中objectivec里面的几十个文件复制到此文件夹![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image3.png)
2. Xcode打开项目，左下角+，引用protocolBuffer，注意里面有三个，我们选择ProtocolBuffers_iOS.xcodeproj![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image7.png)
3. 选中工程: targets —>  Build Phases  —>  Link Binary With Libraries， 引用源码静态库文件：libProtocolBuffers.a![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image6.png)
4. 在工程设置搜索静态库的头文件（pb文件在protocolbuf文件夹里面）:

　　 targets —>  Build Setting  —>  Search Paths  —> Header Search Paths, 写入：$(PROJECT)/protocolbuf 
5.把转换好的person也拖入项目。
#### 测试一下
看代码
![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image9.png)
结果
![](https://raw.githubusercontent.com/Bastionhh/iOS-IM-ProtocolBuffers/master/Image/image10.png)

数据确实非常的小。

以上就是关于ProtocolBuffers的配置与使用。

# GCDAsyncSocket实现即时通讯



