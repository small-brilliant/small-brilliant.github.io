---
title: GRPC实践
tags: 学习记录
categories: 
- 学习记录
- 想要车小程序开发
- 后端
cover: https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251345232.png
date: 2022-02-25 19:42:25
keywords: Go
---
# GRPC实践

在`trip.proto`中

```protobuf
// 请求数据格式
message GetTripRequest{
	string id = 1;
}
// 返回数据格式
message GetTripResponse{
	string id = 1;
	Trip trip = 2;
}
// 服务
service TripService {
	rpc GetTrip (GetTripRequest) returns (GetTripResponse);
}
```

生成代码，为了生成service的代码框架，需要加参数`plugins=grpc`

```
protoc -I=. --go_out=plugins=grpc,paths=source_relative:gen/go trip.proto
```

命令出错：

`-go_out: protoc-gen-go: plugins are not supported; use 'protoc --go-grpc_out=...' to generate gRPC`

版本问题解决办法：重新下载`grpc`和`protoc-gen-go`两个包

```
go get google.golang.org/grpc
go get github.com/golang/protobuf/protoc-gen-go
```

运行成功后，在trip.pb.go里面有客服端和服务端的接口

![image-20220225140946052](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251409079.png)

![image-20220225140922882](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251409917.png)

在server下面创建目录`tripservice`，创建文件`trip.go`来实现`Service`接口

服务端：

```go
package trip

import (
	"context"
	trippb "coolcar/proto/gen/go"
)

type Service struct{}

func (*Service) GetTrip(c context.Context, req *trippb.GetTripRequest) (*trippb.GetTripResponse, error) {
	return &trippb.GetTripResponse{
		Id: req.Id,
		Trip: &trippb.Trip{
			Start:       "abc",
			End:         "dfe",
			DurationSec: 3600,
			FeeCent:     10000,
		},
	}, nil
}

```

在server下创建main.go

```go
package main

import (
	trippb "coolcar/proto/gen/go"
	trip "coolcar/tripservice"
	"log"
	"net"

	"google.golang.org/grpc"
)

func main() {
    l, err := net.Listen("tcp", ":8081")
	if err != nil {
		log.Fatalf("fail to listen: %v", err)
	}

	s := grpc.NewServer()
	trippb.RegisterTripServiceServer(s, &trip.Service{})
	log.Fatal(s.Serve(l))
}

```

运行即可启动服务：

![image-20220225145956411](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251459441.png)

模拟客户端：创建`client/main.go`

```go
package main

import (
	"context"
	trippb "coolcar/proto/gen/go"
	"fmt"
	"log"

	"google.golang.org/grpc"
)

func main() {
	cc, err := grpc.Dial("localhost:8081", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("cannot connect server:%v", err)
	}
	tsc := trippb.NewTripServiceClient(cc)
	r, err := tsc.GetTrip(context.Background(), &trippb.GetTripRequest{
		Id: "trip423",
	})
	if err != nil {
		log.Fatalf("cannot call GetTrip:%v", err)
	}
	fmt.Println(r)
}

```

在TERMINAL运行：

```
go run client/main.go
```

![image-20220225150134494](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251501522.png)

服务器内部GRPC都用TCP连接，要将GRPC暴露给外部使用，下图有两种方法。web转二进制其实不太方便，JavaScript和HTTP设计之初都是为了文本所考虑的。

<img src="https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251345232.png" alt="image-20220225134502047" style="zoom:50%;" />

# GRPC Gateway实现

![image-20220225155044128](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251550162.png)

gen.bat文件是生成命令:

```
protoc -I=. --go_out=plugins=grpc,paths=source_relative:gen/go trip.proto

protoc -I=. --grpc-gateway_out=paths=source_relative,grpc_api_configuration=trip.yaml:gen/go trip.proto
```

`trip.yaml`是配置文件:

```go
type: google.api.Service
config_version: 3

http:
  rules:
  - selector: coolar.TripService.GetTrip
    get: /trip/{id}
```

运行`gen.bat`

![image-20220225155335185](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251553226.png)

生成`trip.pb.gw.go`

![image-20220225155416071](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251554098.png)

main.go代码：

```go
package main

import (
	"context"
	trippb "coolcar/proto/gen/go"
	trip "coolcar/tripservice"
	"log"
	"net"
	"net/http"

	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
	"google.golang.org/grpc"
)

func main() {
	log.SetFlags(log.Lshortfile)
	go startGRPCGateway()
	l, err := net.Listen("tcp", ":8081")
	if err != nil {
		log.Fatalf("fail to listen: %v", err)
	}

	s := grpc.NewServer()
	trippb.RegisterTripServiceServer(s, &trip.Service{})
	log.Fatal(s.Serve(l))
}

func startGRPCGateway() {
	c := context.Background()
	c, cancel := context.WithCancel(c)
	// 调用cancel内部就和gateway断开
	defer cancel()
    //配置传出的数据格式，可以改为false测试不同。
	mux := runtime.NewServeMux(runtime.WithMarshalerOption(
		runtime.MIMEWildcard, &runtime.JSONPb{
			MarshalOptions: protojson.MarshalOptions{
                UseEnumNumbers: true, 
                UseProtoNames: true}},
	))
	err := trippb.RegisterTripServiceHandlerFromEndpoint(
		c,
		//注册位置
		mux,
		":8081",
		//连接方式
		[]grpc.DialOption{grpc.WithInsecure()},
	)
	if err != nil {
		log.Fatalf("cannot start gateway: %v", err)
	}

	err2 := http.ListenAndServe(":8080", mux)
	if err2 != nil {
		log.Fatalf("err2:%v", err2)
	}
}

```

运行`main.go`启动服务，之后客户端调用：

![image-20220225162646745](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251626776.png)

在网页上

![image-20220225162818169](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202251628204.png)