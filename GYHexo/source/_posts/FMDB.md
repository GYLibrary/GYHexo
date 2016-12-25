---
title: FMDB
date: 2015-10-22 20:30:16
categories: OC
tags: [数据库] 
description: SQLite是一种小型的轻量级的关系型数据库，在移动设备上使用是非常好的选择，无论是Android还是IOS，都内置了SQLite数据库，现在的版本都是SQLite3。在IOS中使用SQLite如果使用SDK提供的方法，感觉使用很不方便，今天就讲讲一个针对IOS的SQlite API封装的第三方库FMDB，FMDB对SDK中的API做了一层封装，使之使用OC来访问，使用方便而且更熟悉。
---

# 数据持久化 使用第三方库FMDB
[FMDB](https://github.com/ccgus/fmdb)主要涉及两个类，FMDatabase和FMResultSet 下载完FMDB源码后把文件拖到工程中，并导入SQLite支持        库,libsqlite3.0.dylib
### 相关代码封装
```
#import "GYDataBaseHandle.h"
#import "FMDB.h"
#import "HMTPerson.h"


@implementation GYDataBaseHandle

static GYDataBaseHandle * _dbHandle = nil;
+ (GYDataBaseHandle *)shareInstance{

    @synchronized(self){

        if (!_dbHandle) {

            _dbHandle = [[GYDataBaseHandle alloc]init];

            [_dbHandle openDataBase];

            [_dbHandle createTable];
        }
    }

    return _dbHandle;
}

#pragma mark - 定义一个 FMDatabase 对象
static FMDatabase * database = nil;

#pragma mark - 获得沙盒文件下Documents路径
- (NSString *)getDocumentsPath{

    NSString * documents = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];

    return documents;
}

#pragma mark - 打开数据库操作------ databaseWithPath   open
- (void)openDataBase{

    if (database) {
        return;
    }

    NSString * DataBasePath = [[self getDocumentsPath] stringByAppendingPathComponent:@"test.sqlite"];

    database = [FMDatabase databaseWithPath:DataBasePath];
    if (![database open]) {

        NSLog(@"打开数据库失败");
    }

    // 为数据库设置缓存,提高查询效率
    database.shouldCacheStatements = YES;

    NSLog(@"打开数据库成功");
}

#pragma mark - 关闭数据库操作
- (void)closeDataBase{

    if (![database close]) {
        NSLog(@"关闭数据库失败");
        return;
    }

    database = nil;
    NSLog(@"关闭数据库成功");

}

#pragma mark - 管理创建表的操作
- (void)createTable{

    [self openDataBase];

    // 判断表是否存在,不存在就创建------ tableExists
    if (![database tableExists:@"Person"]) {

        [database executeUpdate:@"CREATE TABLE Person(id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, age INTEGER)"];

        NSLog(@"创建表成功");
    }

    [self closeDataBase];
}


#pragma mark - 增加数据操作----- executeUpdate
- (void)insertIntoDataBase:(HMTPerson *)person{

    [self openDataBase];

    [database executeUpdate:@" INSERT INTO Person (id,name,age) VALUES (?,?,?)",[NSString stringWithFormat:@"%i",person.personId],person.personName,[NSString stringWithFormat:@"%i",person.personAge]];

    [self closeDataBase];
}

#pragma mark - 删除数据操作----- executeUpdate
- (void)deleteDataFromDataBase:(HMTPerson *)person{

    [self openDataBase];

    [database executeUpdate:@" DELETE FROM Person WHERE id = ?",[NSString stringWithFormat:@"%i",person.personId]];

    [self closeDataBase];
}

#pragma mark - 更新数据操作----- executeUpdate
- (void)updateFromDataBase:(HMTPerson *)person{

    [self openDataBase];

    [database executeUpdate:@" UPDATE Person SET name = ? WHERE id = ?",person.personName,[NSString stringWithFormat:@"%i",person.personId]];

    [self closeDataBase];
}

#pragma mark - 查询数据操作(与其他的都不一样,查询是调用executeQuery,切记切记!!!!!!)
- (void)selectAllDataFromDataBase{

    [self openDataBase];

    FMResultSet * resultSet = [database executeQuery:@" SELECT * FROM Person"];
    while ([resultSet next]) {

        int Id = [resultSet intForColumn:@"id"];
        NSString * name = [resultSet stringForColumn:@"name"];
        int age = [resultSet intForColumn:@"age"];

        NSLog(@"personId:%i,personName:%@,personAge:%i",Id,name,age);
    }

    [self closeDataBase];
}


@end
```
### FMDB提供如下多个方法来获取不同类型的数据
1.intForColumn:
2.longForColumn:
3.longLongIntForColumn:
4.boolForColumn:
5.doubleForColumn:
6.stringForColumn:
7.dateForColumn:
8.dataForColumn:
9.dataNoCopyForColumn:
10.UTF8StringForColumnIndex:
11.objectForColumn:

### 注意事项
如果我们的app需要多线程操作数据库，那么就需要使用FMDatabaseQueue来保证线程安全了。切记不能在多个线程中共同一个FMDatabase对象并且在多个线程中同时使用，这个类本身不是线程安全的，这样使用会造成数据混乱等问题。

使用FMDatabaseQueue很简单，首先用一个数据库文件地址来初使化FMDatabaseQueue，然后就可以将一个闭包(block)传入inDatabase方法中。在闭包中操作数据库，而不直接参与FMDatabase的管理。

```
-(void)executeDBOperation  
{  
    //获取Document文件夹下的数据库文件，没有则创建  
    NSString *docPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];  
    NSString *dbPath = [docPath stringByAppendingPathComponent:@"user.db"];  

    FMDatabaseQueue *databaseQueue = [FMDatabaseQueue databaseQueueWithPath:dbPath];  
    [databaseQueue inDatabase:^(FMDatabase *db){  
    [db executeUpdate:@"insert into user values (?,?,?)",@"Ren",@"Male",[NSNumber numberWithInt:20]];  
    }];  
    [databaseQueue close];  
}  
```



