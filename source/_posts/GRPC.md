---
title: GRPC(一)
tags: 学习记录
categories: 
- 学习记录
- 想要车小程序开发
- 后端
cover: https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241936988.png
date: 2022-02-24 19:38:00
keywords: Go
---
[toc]
![image-20220224193606899](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241936988.png)

# ProtoBuf使用

## 安装

- 二进制流
- 语言无关描述来定义结构

安装ProtoBuf，然后需要配置环境变量

protoc帮助命令，运行成功表示安装成功。

去github搜索`grpc-gateway`有下面的安装教程

```
go install \
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway \
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2 \
    google.golang.org/protobuf/cmd/protoc-gen-go \
    google.golang.org/grpc/cmd/protoc-gen-go-grpc
```

如果不行，就一个一个安装。

## hello world

 创建`proto`文件夹，创建`trip.proto`文件

在vscode安装proto3插件。

在`trip.proto`文件中写如下代码：

```protobuf
syntax = "proto3";
package coolcar;
option go_package="coolcar/proto/gen/g0;trippb";
//定义数据格式
message Trip{
	//1数字表示第一个字段
	string start = 1;
	string end = 2;
	int64 duration_sec = 3;
	int64 fee_cent = 4;
}
```

进入`proto`文件夹目录，命令行操作

```
-I表示输入目录=.表示当前目录go_out表示生成go语言的代码paths=source_relative:gen/go表示生成文件的存储路径，最后加上源文件trip.proto
protoc -I=. --go_out=paths=source_relative:gen/go trip.proto
```

![image-20220224162105653](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241621689.png)

![image-20220224162157782](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241621808.png)

```go
import(
    trippb "coolcar/proto/gen/go"
    "fmt"
)
func main(){
    trip:= trippb.Trip{
        Start: "abc",
        End: "def",
        DurationSec: 3600,
        FeeCent: 10000,
    }
    //不加&表示复制，但是trip的一些状态和锁无法复制，所以不能复制。取地址就可以。
    fmt.Println(&trip)
}
```

![image-20220224163059214](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241630240.png)

## 序列化,反序列化二进制流

### 序列化

```go
b, err := proto.Marshal(&trip)
if err !=nil {
	panic(err)
}
fmt.Printf("%X\n",b)
```

![image-20220224163315031](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241633056.png)

### 反序列化

```go
var trip2 trippb.Trip
err = protp.Unmarshal(b, &trip2)
if err!= nil{
	panic(err)
}
fmt.Println(&trip2)
```

这也就是服务器和客户端通信的过程：通过`Marshal`出来的二进制流通过`grpc`服务的TCP端口发给客户端，客户端`Unmarshal`就可以获得数据

## json.Marshal

转json数据

```go
b,err = json.Marshal(&trip2)
```

## 补充

## 更多的数据字段

bytes就是和穿多媒体内容

```protobuf
syntax = "proto3";
package coolcar;
option go_package="coolcar/proto/gen/g0;trippb";
//定义数据格式
message Location{
	double latitude=1;
	double longtitude=2;
}
// 枚举类型
enum TripStatus{
	TS_NOT_SPECIFIDE = 0
	NOT_STARTED = 1;
	IN_PROGRESS = 2;
	FINISHED = 3;
	PAID = 4;
}
message Trip{
	//1数字表示第一个字段
	string start = 1;
	string end = 2;
	int64 duration_sec = 3;
	int64 fee_cent = 4;
	// 复合类型
	Location start_pos = 5;
	//repeated 被翻译成切片
	repeated Location path_locations= 7;
	TripStatus status = 8;
}
```

```go
func main(){
    trip:= trippb.Trip{
        Start: "abc",
        End: "def",
        DurationSec: 3600,
        FeeCent: 10000,
        StartPos: &trippb.Location{
            Latitude: 32,
            Longtitude: 120,
        },
        EndPos: &trippb.Location{
            Latitude: 35,
            Longtitude: 111,
        },
        PathLocations: []*trippb.Location{
            {
                Latitude: 23,
            	Longtitude: 111,
            },
            {
                Latitude: 15,
            	Longtitude: 131,
            },
        },
        Status: trippb.TripStatus_IN_PROGRESS,
    }
    fmt.Println(&trip)
}
```

## ProtoBuf字段的可选性

每个字段都是可填可不填的。ProtoBuf规定了数据在网络上传输的具体格式，但是随着系统的更新，格式是不固定的，需要增加的。新老系统同时存在是非常普遍的问题。新老系统的交替，导致字段不一致的问题。

ProtoBuf的处理方式：

- 新系统数据到老系统，出现的新字段，ProtoBuf不理。


- 老系统数据到新系统，老系统没有的字段数值为零值（比如空字符串，false，0）。


当出现零值之后，序列化就不会有零值的字段。

**所以在定义字段的时候，一定要考虑零值代表的意思**。 