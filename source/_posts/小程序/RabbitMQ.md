---
title: RabbitMQ简单上手
tags: 想要车小程序
categories: 
- 学习记录
- 想要车小程序开发
- 后端
cover: https://images.pexels.com/photos/3112898/pexels-photo-3112898.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
date: 2022-05-05 17:47:19
updated: 2022-05-05 17:47:22
keywords: RabbitMQ
---

# 1.用Docker启动RabbitMQ

1. 拉取镜像

```
docker pull rabbitmq:management
```

2. 运行

```
docker run -d --name myrabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```

端口映射：

- 15672：控制台端口号
- 5672：应用访问端口号

3. 本地访问的网站http://127.0.0.1:15672/

   账号：guest

   密码：guest

# 2.RabbitMQ的使用模式

## 2.1收发

![image-20220503163405470](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205031634508.png)

## 2.2工作分配

![image-20220503163423785](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205031634818.png)

## 2.3pub/sub

x表示exchange，只要绑定了这个exchange的队列，就都收到同样的信息。

![image-20220503163905169](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205031639207.png)

## 2.4路由

![image-20220503163936531](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205031639586.png)

![image-20220503164129626](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205031641672.png)

## 2.5topic

routing_key = quick.orange.rabit**发送到Q1和Q2**

routing_key = lazy.black.dog发送到Q2

![image-20220503164502884](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205031645947.png)

# 3.Go语言使用RabbitMQ

拉客户端

```
go get github.com/streadway/amqp
```

## 3.1收发

```go
package main

import (
	"fmt"
	"time"

	"github.com/streadway/amqp"
)

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		panic(err)
	}
	c, err := conn.Channel()
	if err != nil {
		panic(err)
	}

	q, err := c.QueueDeclare(
		"go_q1",
		true,  // durable
		false, //autoDelete
		false, // exclusive
		false, //nowait
		nil,   // args
	)
	if err != nil {
		panic(err)
	}
	go consume("c1", conn, q.Name)
	go consume("c2", conn, q.Name)
	i := 0
	for {
		i++
		err = c.Publish(
			"",
			q.Name,
			false,
			false,
			amqp.Publishing{
				Body: []byte(fmt.Sprintf("message %d", i)),
			},
		)
		if err != nil {
			fmt.Println(err)
		}

		time.Sleep(time.Millisecond * 200)
	}
}

func consume(consume string, conn *amqp.Connection, q string) {
	c, err := conn.Channel()
	if err != nil {
		panic(err)
	}
	msgs, err := c.Consume(
		q,
		consume,
		true,
		false,
		false,
		false,
		nil,
	)
	if err != nil {
		fmt.Println(err)
	}
	for msg := range msgs {
		fmt.Printf("%s: %s\n", consume, msg.Body)
	}
}
```

## 2.2pub/sub

```go
package main

import (
	"fmt"
	"time"

	"github.com/streadway/amqp"
)

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		panic(err)
	}
	c, err := conn.Channel()
	if err != nil {
		panic(err)
	}
	c.ExchangeDeclare(
		"go_ex",
		"fanout",
		true,
		false,
		false,
		false,
		nil,
	)
	go subscrib(conn, "go_ex")
	go subscrib(conn, "go_ex")
	i := 0
	for {
		i++
		err = c.Publish(
			"go_ex",
			"", //routing key
			false,
			false,
			amqp.Publishing{
				Body: []byte(fmt.Sprintf("message %d", i)),
			},
		)
		if err != nil {
			fmt.Println(err)
		}

		time.Sleep(time.Millisecond * 200)
	}
}
func subscrib(conn *amqp.Connection, ex string) {
	c, err := conn.Channel()
	if err != nil {
		panic(err)
	}
	q, err := c.QueueDeclare(
		"",
		false, // durable
		true,  //autoDelete
		false, // exclusive
		false, //nowait
		nil,   // args
	)
	if err != nil {
		panic(err)
	}
	defer c.QueueDelete(
		q.Name,
		false,
		false,
		false,
	)
	c.QueueBind(
		q.Name,
		"",
		"go_ex",
		false,
		nil,
	)
	consume("c", c, q.Name)
}
func consume(consume string, c *amqp.Channel, q string) {
	msgs, err := c.Consume(
		q,
		consume,
		true,
		false,
		false,
		false,
		nil,
	)
	if err != nil {
		fmt.Println(err)
	}
	for msg := range msgs {
		fmt.Printf("%s: %s\n", consume, msg.Body)
	}
}
```

