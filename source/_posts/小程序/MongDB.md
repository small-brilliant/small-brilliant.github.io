---
title: MongDB
tags: 学习记录
categories: 
- 学习记录
- 后端
cover: https://images.pexels.com/photos/38537/woodland-road-falling-leaf-natural-38537.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
date: 2022-03-02 12:52:17
updated: 2022-03-02 12:52:20
keywords: MongDB
---
## MongoDB的启动

```
docker pull mongo
```

```
docker run -p 18000:18000 -p 27017:27017 mongo
```

安装插件<img src="https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203012119442.png" alt="image-20220301211949410" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203012122207.png" alt="image-20220301212226138" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203012130969.png" alt="image-20220301213046930" style="zoom:50%;" />

地址为：mongodb://localhost:27017/?readPreference=primary&ssl=false

删除目前这个连接，用下面的方式连接

<img src="https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202203012134743.png" alt="image-20220301213445710" style="zoom: 50%;" />

我们可以修改成：mongodb://localhost:27017/coolcar?readPreference=primary&ssl=false

这样操作的时候，都会默认在coolcar数据库进行，如果没有这个数据库，也会默认创建

## CRUD

```json
user('coolcar') // 建立一个coolcar数据库
db.sales.drop() //删除sales表

db.account.insert({
	open_id: "123",
	login_count: 0,
}) // 会创建一个account表，插入数据并且帮我们自动创建一个“_id”的属性并赋值唯一id，我们也可以自己设置这个属性

db.account.insertMany([{
	open_id: "123",
	login_count: 0,
},{
	open_id: "124",
	login_count: 2,
}) // insertMany可以一次性加更多的数据

db.account.find() //查询表account的所有数据
db.account.find({
	_id: ObjectId(""),
})//查询id符合的记录，返回的是一个数组，

db.account.update({
	id: ObjectId("") // 查找的条件
},{
	$set: {
		login_count: 1
	}// 修改的属性
})
db.account.deletOne({
	_id:"",
})//指定id删除

a: {$gt: 4}//表示a大于4
a: {$gte: 4}//表示a大于等于4
a: {$lte: 4}//表示a小于等于4
$or:[],//里面是条件数组

//建立索引
db.account.createIndex({
	"age": 1 //1表示从小到大，-1表示从大到小
},{
    unique: true // true,如果用一个属性作为了索引，插入如果有重复的数值，则会失败。
})

db.account.findAndModify{
	query:{
		opend_id: opend_id
	},
	update:{
		$set:{opend_id:opend_id}
	}
	upsert: true,
	new: false,// false会查询原有的记录，不会本次更新的数据查找
}//查询并更新
```
