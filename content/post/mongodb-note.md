---
layout: post
title: MongoDB学习笔记
date: 2017-06-25
author: opso
tags:
- mongo
cover: "/images/mongodb_logo.jpg"
---

MongoDB  是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。


在接触了 `Redis`、`Memcache`、`MySQL`这些数据库之后，由于工作需要，最近在学习另外一个常见的 **NoSQL数据库** ： `MongoDB`，下面是整理的资料和笔记。

<!--more-->

| 对比项  | Mongodb      | MySQL等     |
| :--- | :----------- | :--------- |
| 数据库  | database     | database   |
| 表    | 集合collection | 二维表table   |
| 一行   | 文档document   | 一条记录record |
| 字段   | 键key         | 列column    |

## 主要功能特性

1. 文件存储格式`BSON`（一种json的扩展） 
2. 模式自由 数据格式不受限了表的结构 
3. 支持动态查询 
4. 支持完全索引 
5. 支持复制（其主从复制）和故障恢复 
6. 使用高效的二进制数据存储，包括大型对象 
7. 自动处理碎片，以支持云计算层次的扩展。 
8. 支持`Java`、`Ruby`、`Python`、`C++`、`PHP`等多种语言 
9. 内部支持`JavaScript`

## 安装

linux下安装过程这里简单说明下

1. 下载适合当前系统、编译好的压缩包
2. 解压缩并移动到 `usr/local/mongodb`
3. 创建`/data/mongo_data`和`/data/mongo_log`,创建`mongodb:mongodb`用户，修改以上文件夹权限
4. 将mongodb添加到`PATH`环境中

这里遇到一个问题，如果直接使用`mongodb`指定配置文件启动，默认情况下，`mongodb`会以`root`用户运行主进程和子进程，这时`mongo`命令行会警告不安全的root用户权限:

> [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.

找到一个自启动脚本（**ubuntu**）

https://github.com/mongodb/mongo/blob/master/debian/init.d

研究了下，关键命令是：

> start-stop-daemon --start --quiet --pidfile /var/run/mongodb.pid --chuid mongodb:mongodb --exec /usr/local/mongodb/bin/mongod -- -f /usr/local/mongodb/mongo.conf

这里的 `--chuid mongodb:mongodb` 参数表示会将子进程以`mongodb` 用户运行，确保了数据的安全性。

## PHP扩展

1. [mongo扩展](http://pecl.php.net/package/mongo)
2. [mongodb扩展](http://pecl.php.net/package/mongodb)

这里PHP7和PHP5将会有所不同，`mongo`扩展不支持PHP7，得安装`mongodb`，支持`php5.5`及以上，安装过程就不赘述了 : )

## 索引说明

mongodb索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。

实际项目中基本不允许不带索引的查询；索引大小写敏感，索引分两种 复合索引和单索引，任何类型，包括文档(document)都能作为索引，默认会建立 `_id` 单索引。

索引也会在插入和删除的时候增加一些系统的负担，所以建立索引更适合读取较多的场景，也是大多数应用的场景。

```js
db.stu.ensureIndex({KEY:1}) // 建立索引
db.stu.getIndexs() // 获取索引
db.stu.reIndex()//通常这是不必要的，但是在集合的大小变动很大及集合在磁盘空间上占用很多空间时重建索引才有用。对于大数据量的集合来说，重建索引可能会很慢
```

## 命令操作

### 数据库

- `show dbs` 展示数据库 类似SQL的`SHOW DATABASES`
- `use xxx`  创建/切换数据库
  以下皆以 `stu` 作为举例

### 文档(document)操作

以 Collection_name = stu 为例

####  1.插入

```js
db.stu.insert(Document);	// 插入一条或多组数据
db.stu.insertOne(Document); // 插入一条数据
db.stu.insertMany(Document);// 插入多条数据
// 例：
db.stu.insertOne({"name":"xiaoming", "age":20, "height": 170});
db.stu.insertMany([
	{"name":"xiaoming", "age":20, "height": 170},
	{"name":"billy", "age":21, "height": 172},
	{"name":"xiaohong", "age":18, "height": 164},
	{"name":"xiaohua", "age":23, "height": 181},
	{"name":"lisi", "age":18, "height": 171, "extra": {"favorite": "eat", "sport": "running"}},
	{"name":"wangwu", "age":20, "height": 170, "extra": {"favorite": "sleep", "sport": "swimming"}},
]);
```

#### 2.删除

```js
db.stu.remove({}); // 删除所有数据
db.stu.drop(); // 同上
db.stu.remove(query <,options>);
// query: 查询条件（数据索引或名字）
// options: 可选条件
// 		{
//			justOne: <boolean>,     //默认false，删除所有匹配到的。
// 			writeConcern: <document>//抛出异常的级别。
// 		}
db.col.deleteOne(query <,options>)   // 同上，无justOne参数，只删除第一条
db.col.deleteMany(query <,options>)  // 同上，无justOne参数，只删除多条
```

#### 3.更新

```js
db.stu.update(query, update, <,options>);
//	query:  查询条件(数据索引或名字)
//	update: 更新的内容，语法：{$set:query}
//	options:三个可选参数
//		{
//			upsert: <boolean>,      //如果不存在update的记录，是否插入新数据，默认:false。
//			multi: <boolean>,       //只更新找到的第一条记录，默认是false,如果为true,多条记录全部更新。
//     		writeConcern: <document>//#抛出异常的级别。
//    	}
// 例如：
db.stu.update(
	{"name":"billy"}, 
	{$set: {"age": 18}},
	{upsert: true, multi: true}
);
db.stu.updateOne();	// 同上，无multi参数，只更新第一条
db.stu.updateMany();// 同上，无multi参数
db.stu.replaceOne();// 同updateOne
db.stu.save(document <,writeConcern>)	// 通过传入的文档整个替换
// 例 增加billy age加4
db.stu.update({"name":"billy"},{$inc:{"age":4}});
```

**insert 与 save 的区别**
如果插入的数据`_id`相同，`save`将会更新改文档，而`insert`将会报错

| 操作符          | 说明                                       |
| :----------- | :--------------------------------------- |
| $set         | 当文档中包含该字段的时候,更新该字段,如果该文档中没有该字段,则为本文档添加一个字段 |
| $unset       | 当文档中包含该字段的时候,更新该字段,如果该文档中没有该字段,则为本文档添加一个字段 |
| $rename      | 重命名某个列                                   |
| $inc         | 增长某个列                                    |
| $setOnInsert | 当upsert为true时,并且发生了insert操作时,可以补充的字段     |
| $push        | 将一个数字存入一个数组,分为三种情况,如果该字段存在,则直接将数字存入数组.如果该字段不存在,创建字段并且将数字插入该数组.如果更新的字段不是数组,会报错的 |
| $pushAll     | 将多个数值一次存入数组.上面的push只能一个一个的存入             |
| $addToSet    | 与push功能相同将一个数字存入数组,不同的是如果数组中有这个数字,将不会插入,只会插入新的数据,同样也会有三种情况,与push相同 |
| $pop         | 删除数组最后一个元素                               |
| $pull        | 删除数组中的指定的元素,如果删除的字段不是数组,会报错              |
| $pullAll     | 删除数组中的多个值,跟pushAll与push的关系类似             |

#### 4.查询

```js
db.stu.find({})          // 查询所有文档
db.stu.find().pretty()   // 以格式化输出数据
db.collection.find(query, projection)
//	query：查询条件(数据索引或名字)
//	projection：可选。指定返回的字段。
//	只显示_id,name字段
db.stu.find({}, {name: 1});
```

以`age`字段查询：

```js
db.stu.find({age: value});
```

| value                                    | 说明                  |
| ---------------------------------------- | ------------------- |
| {$ne: 20}                                | 不等于                 |
| {$nin: [18,19,20]}                       | 不包含后面给定的值           |
| {$all: [18,19,20]}                       | 必须包含后面给定的值          |
| {$in: [18,19,20]}                        | 包含一个或多个             |
| {$exists: true}                          | 是否存在，true或1存在，反之不存在 |
| {$mod: [10, 0]}                          | 取模操作（模10余0）         |
| {$or: [{filed1: vulue1}, {filed2: vulue2}]} | 或，满足其中一个即可          |
| {$nor: [{filed1: vulue1}, {filed2: vulue2}]} | 异或，排除这两种情况的其他情况     |
| {$where: function(){ return ... }}       | 回调，符合条件才返回          |
| {$lt: 20}                                | <                   |
| {$gt: 20}                                | >                   |
| {$lte: 20}                               | <=                  |
| {$gte: 20}                               | >=                  |

另外，`find` 还支持正则表达式查询（类似mysql里的`like`），但效率并不高：

```js
db.stu.find({name: /user.*/i}); // 查询filed以user开头不区分大小写
```

其他常见的还有`skip`、`limit`、`sort`、`count` 等方法：

```js
// 读取年龄最大的3-8名学生信息
db.stu.find().sort({age: -1}).skip(2).limit(5);
// 获取学生人数
db.stu.find().count();
```

**distinct** 去重：

```js
db.stu.distinct("age"); // 特殊,传入的参数直接是字符串,而不是对象; [ 20, 25, 18, 23 ]
```

#### 5.子文档查询 $elemMatch

elemMatch投影操作符将限制查询返回的数组字段的内容只包含匹配elemMatch条件的数组元素。
注意：
(1) 数组中元素是内嵌文档。
(2) 如果多个元素匹配$elemMatch条件，操作符返回数组中第一个匹配条件的元素。

```js
// 1 常规方法
db.stu.find({'extra.sport': 'swimming'});
// 2 使用操作符
db.stu.find({"extra": {
	$elemMatch: {"sport": "swimming"}}
});
```

### 存储过程

```js
db.system.js.save({_id:"mypro",value:function(){
     var total = 0;
     var arr = db.student.find().toArray();
     for(var i=arr.length-1;i>=0;i--){
         total += arr[i].age;
     }
     return total;
}});
//查询存储过程
db.system.js.find();
//执行存储过程
db.eval("mypro()");
```

### 资料

[mongoDB 4.3中文文档](http://docs.mongoing.com/manual-zh)