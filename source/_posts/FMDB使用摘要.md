title: FMDB使用摘要
tags:
  - fmdb
  - sqlite
categories: []
date: 2016-03-09 16:47:00
---
最近项目中需要加入持久化数据缓存的需求，在比较了[EGOCache](https://github.com/enormego/EGOCache)与[FMDB](https://github.com/ccgus/fmdb)之后，决定采用基于SQLite3的FMDB来实现，结合[AFNetworking](https://github.com/AFNetworking/AFNetworking)封装了一套API缓存方案，在此简单介绍下具体逻辑与注意事项。

####1，使用FMDatabaseQueue

FMDB的每一次数据读写操作，都可以理解为一个Operation，数据库是每个Operation对应的数据源，所以需要保证操作的线程安全，这就引入了FMDatabaseQueue的概念。每次Operation顺序加入队列中，保证了数据访问的有序进行，互补干扰。

`+ (instancetype)databaseQueueWithPath:(NSString*)aPath`

通过dispatch_queue_create创建并发队列，不同的dataPath通过FMDatabaseQueue来维护，多个queue之间并发执行任务，保证了数据读写的效率。

`- (void)inDatabase:(void (^)(FMDatabase *db))block`

每次Operation通过inDatabase执行，dispatch_sync方法来添加串行任务，保证了每个FMDatabaseQueue数据的安全与有序。在添加任务前，调用**dispatch_get_specific**获得**FMDatabaseQueue**关联的线程，检查是否是同一个线程，因为串行队列里面同步执行**dispatch_sync**会造成死锁。

####2，存储json数据

在GET到API返回json数据时候，常常会遇到字典或者数组嵌套的情况，使用FMDB写入数据时，如果Key-Value一一对应会比较麻烦，所以，把整条记录转换为string对象来存储，在不降低性能的情况下对数据的写入与读取十分便捷。在此需要注意sql语句中的string要作为参数写入，以免引起syntax error，例如：

``` objc
// 正确：
NSString  *insertSQl=[NSString stringWithFormat:@"insert or replace into '%@' (ID,JSONString) values (?,?) ",tableName];
  [_queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
     if(![db executeUpdate:insertSQl,objId,jsonStr]){
          BDLog(@"rollback Table %@",tableName);
          *rollback = YES;
          return ;
      }
   }];

// 错误：
NSString  *insertSQl=[NSString stringWithFormat:@"insert or replace into '%@' (ID,JSONString) values (%@,%@) ",tableName,objId,jsonStr];   
  [_queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
      if(![db executeUpdate:insertSQl]){
          BDLog(@"rollback Table %@",tableName);
          *rollback = YES;
          return ;
      }
  }];
```

如果是大量的数据保存操作可以通过dispatch_group_t创建并发线程来执行，在并发线程中加入串行任务，避免对主线程造成阻塞，来提高代码效率。示例如下：

``` objc
    // 并发线程中串行执行
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t serialQueue = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
    
    dispatch_group_async(group, serialQueue, ^{
      // 先执行清理工作
    });
    
    dispatch_group_notify(group, serialQueue, ^{
      // 保存数据
    });
```

####3，sqlite事务

与关系数据库进行交互的标准 SQLite 命令类似于 SQL。命令包括 CREATE、SELECT、INSERT、UPDATE、DELETE 和 DROP。这些命令基于它们的操作性质可分为以下几种：

- DDL - 数据定义语言(CREATE、ALTER、DROP)
- DML - 数据操作语言(INSERT、UPDATE、DELETE)
- DQL - 数据查询语言(SELECT)

事务控制命令只与 DML 命令 INSERT、UPDATE 和 DELETE 一起使用。他们不能在创建表或删除表时使用，因为这些操作在数据库中是自动提交的。

SQLite中使用下面的命令来控制事务：BEGIN TRANSACTION：开始事务处理；COMMIT：保存更改，或者可以使用 END TRANSACTION 命令；ROLLBACK：回滚所做的更改。如果iOS的sqlite同时插入或者查询10000条数据，你该怎么办？减少数据库的开关操作，通过事务命令，把10000条语句封装成一个事务，只需一次COMMIT transaction即可完成。示例如下：
``` objc
// 开始事务
-(int)beginService{
     char *errmsg;
     int rc = sqlite3_exec(database, "BEGIN transaction", NULL, NULL, &errmsg);
     return rc;
 }
 
// 提交事务  
-(int)commitService{
     char *errmsg;
     int rc = sqlite3_exec(database, "COMMIT transaction", NULL, NULL, &errmsg);
     return rc;
}

- (void)execSQL
{
   [database open]
   [self beginService];
   for (i=0; i<100000; i++) {
          //想要执行的sql语句
       }
   [self commitService]
   [database close]
}

```

FMDB对事务也有很好的支持，调用`- (void)inTransaction:(void (^)(FMDatabase *db, BOOL *rollback))block`方法即可实现，如果数据写入或读取失败，可以指定rollback参数来设置是否需要回滚，撤销本次操作。

####4，FMDB性能测试

设备环境iphone6s , xcode7.2 ，FMDB分别读写100、500、1000条数据，测试结果如下：

- 100条数据：写入用时0.655秒，读取0.012秒
- 500条数据，写入用时2.112秒，读取0.043秒
- 1000条数据，写入用时6.2秒，读取0.036秒

可见FMDB的读取性能还是很不错的，大量写入数据的时候比较耗时，可以通过切换至后台线程执行，保证主线程的任务可以立即响应。

参考：

[http://www.runoob.com/sqlite/sqlite-intro.html](http://www.runoob.com/sqlite/sqlite-intro.html)