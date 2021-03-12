---
layout: single
title:  "Protocol Buffer"
date:   2019-02-20 10:50:46 +0800
permalink: /golang/part1/protocol-buffer
toc: true
---



[TOC]

#### 简介
一套轻便高效的结构数据序列化机制，同类型的如JSON, XML。ProtoBuf序列化后所生成的二进制消息非常紧凑,一条数据，用protobuf序列化后比使用json，xml小的多，同时解析速度也更快。

可用于通讯协议、数据存储等领域的语言无关、平台无关。很适合做数据存储或 RPC 数据交换格式。

Protocol Buffers在谷歌被广泛用于各种结构化信息存储和交换。Protocol Buffers作为一个自定义的远程过程调用（RPC）系统，用于在谷歌几乎所有的设备间的通信。 protobuf 一个单纯的序列化工具。

proto2和proto3，两个版本之间差异较大，推荐使用proto3版本。

```
# XML
  <person>
    <name>John Doe</name>
    <email>jdoe@example.com</email>
  </person>
# Protocol buffers
person { name: "John Doe" ; email: "jdoe@example.com"}
```



#### 安装 Protocol Buffers
```python
# 依赖
yum -y install gcc  gcc-c++ automake autoconf libtool make
# 源码
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protobuf-all-3.6.1.tar.gz
tar zxvf protobuf-all-3.6.1.tar.gz
cd protobuf-3.6.1/
./autogen.sh
./configure && make && sudo make install
```
查看版本protoc --version

##### 使用ProtoBuf
编写.proto文件
```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
option java_outer_classname = "HelloWorldProto";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```


编译 .proto 文件
用 Protobuf 编译器将该文件编译成目标语言
编译为c++文件 `protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto`
编译为go 文件 `protoc --go_out=. *.proto` 或  `protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto`

```python
# protoc --go_out=. *.proto
protoc --go_out=plugins=grpc:. *.proto
```

需要处理的结构化数据由 .proto 文件描述

message是.proto文件最小的逻辑单元，由一系列name-value键值对构成。

```protobuf
syntax = "proto3";
package order;

//自定义一个订单类 order.proto
//每个字段名对应一个末尾数字标识
message Order {
    int32 id = 1;
    int32 userId = 2;
    float amount = 3;
    string date = 4;
}
//编译生成go文件
protoc order.proto --go_out=. *.proto
```


#### 和其他类似技术的比较
如 XML，JSON，Thrift 等.
同 XML 相比， Protobuf 的主要优点在于性能高。它以高效的二进制方式存储，比 XML 小 3 到 10 倍，快 20 到 100 倍。Protbuf 与 XML 相比也有不足之处。它功能简单，无法用来表示复杂的概念。XML通用性好。 XML可以直接读取编辑， Protobuf则 不行，它以二进制的方式存储，除非有 .proto 定义，否则没法直接读出 Protobuf 内容。

### Protocol Buffer 的 编码
Protobuf 序列化后所生成的二进制消息非常紧凑。消息经过序列化后会成为一个二进制数据流，

#### Key-Value 对
该流中的数据为一系列的 Key-Value 对。`key|value|key|value|key|value|key|value|`
采用这种 Key-Pair 结构无需使用分隔符来分割不同的 Field。对于可选的 Field，如果消息中不存在该 field，那么在最终的 Message Buffer 中就没有该 field，这些特性都有助于节约消息本身的大小。

##### Varint
Varint 是一种紧凑的表示数字的方法。它用一个或多个字节来表示一个数字，值越小的数字使用越少的字节数。这能减少用来表示数字的字节数。比如对于 int32 类型的数字，一般需要 4 个 byte 来表示。但是采用 Varint，对于很小的 int32 类型的数字，则可以用 1 个 byte 来表示。
看起来有点像数据库中的 varchar。

### Protocol Buffer 封解包的速度

####  XML 的封解包过程
XML 需要从文件中读取出字符串，再转换为 XML 文档对象结构模型。再从 XML 文档对象结构模型中读取指定节点的字符串，最后再将这个字符串转换成指定类型的变量。这个过程非常复杂，其中将 XML 文件转换为文档对象结构模型（`DOM`）的过程通常需要完成词法文法分析等大量消耗 CPU 的复杂计算。
`XML中处理DOM的过程就比较消耗CPU`

#### Protobuf
只要将一个二进制序列，按照指定的格式读取到对应的结构类型中就可以了。 decoding 解码过程也可以通过几个位移操作组成的表达式计算即可完成。速度非常快。（其实都是算法带来的差距）

#### 练习
```go
package main
import (
	"./order"
	"github.com/golang/protobuf/proto"
	"log"
	"io/ioutil"
	"fmt"
	"os"
)

var filename = "./order.data"

func orderWrite() {
	o1 := &order.Order{
		Id:10001,
		UserId:1325047,
		Amount:588,
		Date:"2018-08-17 17:13:08",
	}

	out, err := proto.Marshal(o1)
	if err != nil {
		log.Fatalln("Failed to encode address book:", err)
	}
	if err := ioutil.WriteFile(filename, out, os.ModePerm); err != nil {
		log.Fatalln("Failed to write address book:", err)
	}
	fmt.Printf("%s\n", out)
	fmt.Printf("%d\n", out)
	fmt.Printf("%b\n", out)
}

func orderRead() {
	//var orders []order.Order
	in, err := ioutil.ReadFile(filename)
	if err != nil {
		log.Fatalln("Error reading file:", err)
	}
	orders := &order.Order{}
	if err := proto.Unmarshal(in, orders); err != nil {
		log.Fatalln("Failed to parse address book:", err)
	}
	fmt.Println(orders)
}

func main() {
	orderWrite()
	orderRead()
}

//序列化的文本内容为 id:10001 userId:1325047 amount:588 date:"2018-08-17 17:13:08"
//十进制显示为 [8 145 78 16 247 239 80 29 0 0 19 68 34 19 50 48 49 56 45 48 56 45 49 55 32 49 55 58 49 51 58 48 56]
//对应protocol的二进制序列为：[1000 10010001 1001110 10000 11110111 11101111 1010000 11101 0 0 10011 1000100 100010 10011 110010 110000 110001 111000 101101 110000 111000 101101 110001 110111 100000 110001 110111 111010 110001 110011 111010 110000 111000] 
```
从二进制序列可以看出varint，存储字节数的确是随着数字大小变化的



参考：
https://developers.google.com/protocol-buffers/docs/overview 
https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/index.html


## Protobuf 使用
官方文档：https://developers.google.cn/protocol-buffers/docs/proto3#simple

Protobuf核心的工具集是C++语言开发的，在官方的protoc编译器中并不支持Go语言。要想基于上面的hello.proto文件生成相应的Go代码，需要安装相应的插件。可以通过go get github.com/golang/protobuf/protoc-gen-go命令安装。

`Protobuf用来作为接口的定义和描述语言，来保证RPC的接口规范和安全`。

注释: .proto文件注释，可以使用C/C++风格的 // 和 /* … */ 语法格式.

当用protocolbuffer编译器来运行.proto文件时，编译器将生成所选择语言的代码，这些代码可以操作在.proto文件中定义的消息类型，包括获取、设置字段值，将消息序列化到一个输出流中，以及从一个输入流中解析消息。对于 Go 语言，针对每一个定义的消息类型编译器会创建一个带类型的.pb.go 文件。

#### .proto文件生成Go代码

```
protoc --go_out=. *.proto #会在同目录下生成同名 .pb.go文件

#指定文件
protoc --go_out=. hello.proto
```


#### 标量类型
.proto文件中标量类型与go类型对应关系 

```go
//proto标量	  go标量
//double      float64
//float       float32
//int32       int32
//int64       int64
//uint32      uint32
//uint64      uint64
//sint32      int32  //使用可变长度编码。
//sint64      int64  //使用可变长度编码。
//fixed32     uint32  //使用可变长度编码。有符号的int值。int32相比，它们更有效地编码负数。
//fixed64     uint64  //使用可变长度编码。有符号的int值。int64相比，它们更有效地编码负数。
//sfixed32    int32   //始终占四个字节
//sfixed64    int64   //始终占八个字节
//bool        bool
//string      string
//bytes       []byte
```


#### message
Protobuf中最基本的数据单元是message，是类似Go语言中结构体。在message中可以嵌套message或其它的基础数据类型的成员。

```protobuf
syntax = "proto3"; //proto3 语法
message String {
    string value = 1;
}
```

生成的go结构体为：

```go
type String struct {
	Value                string   `protobuf:"bytes,1,opt,name=value,proto3" json:"value,omitempty"`
	XXX_NoUnkeyedLiteral struct{} `json:"-"`
	XXX_unrecognized     []byte   `json:"-"`
	XXX_sizecache        int32    `json:"-"`
}
//该对象还有其他方法，如Reset, GetValue, ProtoMessage, Descriptor等
```

#### map
`map<key_type, value_type> map_field = N`;

```protobuf
message M {
    map<string, string> map_a = 1;
    repeated int32 ids = 2;
}
```

转为go代码

```go
type M struct {
	MapA                 map[string]int32 
	Ids                  []int32         
}
```

#### service
如果要将 message 类型与 RPC（远程过程调用）系统一起使用，则可以在 .proto 文件中定义 RPC 服务接口，如果您想使用一种方法来定义RPC服务，该方法接受您的参数SearchRequest并返回SearchResponse，则可以在.proto文件中按以下方式进行定义：

```protobuf
syntax = "proto3";

service SearchService {
    rpc Search(SearchRequest) returns(SearchResponse);
}

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

与 ProtoBuf 直接搭配使用的 RPC 系统是 gRPC :一个 Google 开发的平台无关语言无关的开源 RPC 系统。gRPC 和 ProtoBuf 能够非常完美的配合，你可以使用专门的 ProtoBuf 编译插件直接从.proto 文件生成相关 RPC 代码。
如果你不想使用 gRPC，你也可以用自己的 RPC 来实现和 ProtoBuf 协作。

#### JSON 映射
Proto3 支持标准的 JSON 编码，使得在不同的系统直接共享数据变得简单。

#### option 
.proto 文件中的各个声明可以使用一些选项进行诠释。选项不会更改声明的含义，但可能会影响在特定上下文中处理它的方式。

#### 生成代码
要 由.proto 文件生成 Java, Python, C++, Go, Ruby, Objective-C, 或者r C#代码，你需要使用 .proto 文件中定义的 message 类型，你需要在 .proto 上运行 protocol buffer 编译器 protoc。
Protocol 编译器的调用如下：
`protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
`

IMPORT_PATH 指定在解析导入指令时查找 .proto 文件的目录。如果省略，则使用当前目录。可以通过多次传递 --proto_path 选项来指定多个导入目录；他们将按顺序搜索。-I = IMPORT_PATH 可以用作 --proto_path 的缩写形式。
如果输出存档已存在，则会被覆盖。
输出指令:
```
--cpp_out 在 DST_DIR 中生成 C++ 代码。
--java_out 在DST_DIR中生成 Java 代码。
--python_out 在 DST_DIR 中生成 Python 代码。
--go_out 在 DST_DIR 中 生成 Go 代码 。
--php_out 在 DST_DIR 中生成PHP 代码。
```

示例：

```
protoc --go_out=. *.proto #会在同目录下生成同名 .pb.go文件

#指定文件
protoc --go_out=. hello.proto
```



```bash
go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway
go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
go get -u github.com/golang/protobuf/protoc-gen-go
```

```bash
protoc  --proto_path=${GOPATH}/src \
    --proto_path=${GOPATH}/pkg/mod/github.com/golang/protobuf@v1.4.2
    --proto_path=. \
    --govalidators_out=. --go_out=plugins=grpc:.\
    *.proto
    


protoc --proto_path=.  -I /usr/local/include  -I ${GOPATH}/pkg/mod/github.com/grpc-ecosystem/grpc-gateway@v1.14.5/third_party/googleapis  --grpc-gateway_out=logtostderr=true:. --go_out=plugins=grpc:. *.proto


protoc --proto_path=. \
	-I /usr/local/include \
	-I ${GOPATH}/pkg/mod/github.com/grpc-ecosystem/grpc-gateway@v1.14.5/third_party/googleapis  \
	--proto_path=${GOPATH}/pkg/mod/github.com/golang/protobuf@v1.4.2 \
	--go_out=plugins=grpc:.--govalidators_out=. --go_out=plugins=grpc:. *.proto

# --plugin：grpc-安装的目录
# -I :项目的全目录
```











```
protoc --proto_path=. \
   -I ${GOPATH}/pkg/mod/github.com/grpc-ecosystem/grpc-gateway@v1.14.5/third_party/googleapis  \
   --proto_path=${GOPATH}/pkg/mod/github.com/golang/protobuf@v1.4.2 \
    --go_out=plugins=grpc:. *.proto
```












