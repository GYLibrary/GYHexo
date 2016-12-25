---
title: Sqlite
date: 2015-06-02 15:30:16
categories: OC
tags: [数据库] 
description: 本章记录sqlite3的相关操作步骤与使用。
---

# 数据持久化
## 常用接口
```
sqlite3               *pdb, 数据库句柄，跟文件句柄 FILE 很类似
sqlite3_stmt      *stmt, 这个相当于 ODBC 的 Command 对象，用于保存编译好的 SQL 语句

sqlite3_open(),   打开数据库
sqlite3_exec(),   执行非查询的 sql 语句
sqlite3_prepare(), 准备 sql 语句，执行 select 语句或者要使用 parameter bind 时（封装了 sqlite3_exec ） .
Sqlite3_step(), 在调用 sqlite3_prepare 后，使用这个函数在记录集中移动。
Sqlite3_close(), 关闭数据库文件

还有一系列的函数，用于从记录集字段中获取数据，如
sqlite3_column_text(), 取 text 类型的数据。
sqlite3_column_blob （），取 blob 类型的数据

sqlite3_column_int(), 取 int 类型的数据
```

## Sqlite3操作步骤

1 首先获取iPhone上Sqlite 3的数据库文件的地址
2 打开Sqlite 3的数据库文件
3 定义SQL文
4 邦定执行SQL所需要的参数
5 执行SQL文，并获取结果
6 释放资源
7 关闭Sqlite 3数据库。
## 导入库

在工程中的Frameworks中导入相应的`libsqlite3.dylib`文件，也许在相应的目录下存在多个以libsqlite3开头的文件，务必选择`libsqlite3.dylib`，它始终指向最新版的SQLite3库的别名。

### *SQLite数据库执行SQL语句的过程*

1. 调用sqlite3_prepare()将SQL语句编译为sqlite内部一个结构体(sqlite3_stmt).该结构体中包含了将要执行的的SQL语句的信息. 
2. 如果需要传入参数,在SQL语句中用'?'作为占位符,再调用sqlite3_bind_XXX()函数将对应的参数传入.
3. 调用sqlite3_step(),这时候SQL语句才真正执行.注意该函数的返回值,SQLITE_DONE和SQLITE_ROW都是表示执行成功, 不同的是SQLITE_DONE表示没有查询结果,象UPDATE,INSERT这些SQL语句都是返回SQLITE_DONE,SELECT查询语句在 查询结果不为空的时候返回SQLITE_ROW,在查询结果为空的时候返回SQLITE_DONE. 
4. 每次调用sqlite3_step()的时候,只返回一行数据,使用sqlite3_column_XXX()函数来取出这些数据.要取出全部的数据需要 反复调用sqlite3_step(). (注意, 在bind参数的时候,参数列表的index从1开始,而取出数据的时候,列的index是从0开始). 
5. 在SQL语句使用完了之后要调用sqlite3_finalize()来释放stmt占用的内存.该内存是在sqlite3_prepare()时分配的. 
6. 如果SQL语句要重复使用,可以调用sqlite3_reset()来清楚已经绑定的参数

```
#import "HMTSqlDBManager.h"
#import <sqlite3.h>          //--------记得导入libsqlite3.0.dylib库
@implementation HMTSqlDBManager

static  HMTSqlDBManager * _dbManager = nil;

+ (HMTSqlDBManager *)shareInstance{

    @synchronized(self){
        if (!_dbManager) {

            _dbManager = [[HMTSqlDBManager alloc]init];
        }
    }

    return _dbManager;
}

#pragma mark - 获取沙盒下Documents路径
- (NSString *)documentsPath{

    return [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject];

}

#pragma mark - 先声明一个数据库类型的变量
static sqlite3 * dbSqlite = nil;

#pragma mark - 打开数据库操作
- (void)openDB{

    // (3)如果数据库存在并且已经打开,直接返回
    if (dbSqlite) {
        return;
    }

    // 拼接出数据库全路径
    NSString * filePath = [[self documentsPath] stringByAppendingPathComponent:@"Lanou.sqlite"];

    // 创建一个文件管理器
    NSFileManager * fileManager = [NSFileManager defaultManager];

    // (1)判断是否存在数据库文件
    if (![fileManager fileExistsAtPath:filePath]) {

        // 打开数据库(这里已经会创建data.sqlite数据库了,但是,里面为空)
        int result = sqlite3_open([filePath UTF8String], &dbSqlite);
        if (result == SQLITE_OK) {

            // 设置创建表语句CREATE
            NSString * sql = @"CREATE TABLE Student (number INTEGER PRIMARY KEY  AUTOINCREMENT  NOT NULL , name TEXT DEFAULT 18, age INTEGER NOT NULL  DEFAULT 18,sex TEXT DEFAULT 不男不女,icon BLOB);";

            // 执行
            sqlite3_exec(dbSqlite, [sql UTF8String], NULL, NULL, NULL);
        }
    }

    // (2)打开数据库
    sqlite3_open([filePath UTF8String], &dbSqlite);

}

#pragma mark - 关闭数据库操作
- (void)closeDB{

    int result = sqlite3_close(dbSqlite);
    if (result == SQLITE_OK) {
        dbSqlite = nil;
    }

}

#pragma mark - **********增-----删-----改-----查 数据,最好封装成对象,直接对对象进行操作**********

#pragma mark - 插入数据操作
- (void)insertDataWithNumber:(int)number Name:(NSString *)name Sex:(NSString *)sex Age:(int)age Image:(UIImage *)image{

    // 1.打开数据库
    [self openDB];

    // 2.跟随指针,记录数据库的每步操作
    sqlite3_stmt * stmt = nil;

    // 3.绑定数据,执行sql语句(插入列表,用"?"占位符表示)
    NSString * sql = @"INSERT INTO Student(number,name,sex,age,icon) VALUES(?,?,?,?,?)";

    // 4.编译,验证SQL语句是否正确
    int result = sqlite3_prepare_v2(dbSqlite, [sql UTF8String], -1, &stmt, NULL);
    if (result == SQLITE_OK) {

        // 5.绑定sql语句中得值(替换"?")----图像存入数据库,自断类型为blob,将图像转为二进制数据NSData存入
        sqlite3_bind_int(stmt, 1, number);
        sqlite3_bind_text(stmt, 2, [name UTF8String], -1, NULL);
        sqlite3_bind_text(stmt, 3, [sex UTF8String], -1, NULL);
        sqlite3_bind_int(stmt, 4, age);
        sqlite3_bind_blob(stmt, 5, [UIImagePNGRepresentation(image) bytes], (int)[UIImagePNGRepresentation(image) length], NULL);
    }

    // 6.执行
    sqlite3_step(stmt);

    // 7.释放
    sqlite3_finalize(stmt);

    // 8.关闭数据库
    [self closeDB];
}

#pragma mark - 修改数据操作
- (void)updateNumber:(int)number Name:(NSString *)name{

    // 1.
    [self openDB];

    // 2.
    sqlite3_stmt * stmt= nil;

    // 3.
    NSString * sql = @"UPDATE Student SET name = ? WHERE number = ?";

    // 4.
    int result = sqlite3_prepare_v2(dbSqlite, [sql UTF8String], -1, &stmt, NULL);
    if (result == SQLITE_OK) {

        // 5.
        sqlite3_bind_text(stmt, 1, [name UTF8String], -1, NULL);
        sqlite3_bind_int(stmt, 2, number);

        // 6.
        sqlite3_step(stmt);
    }

    // 7.
    sqlite3_finalize(stmt);

    // 8.
    [self closeDB];

}

#pragma mark - 删除数据操作
- (void)deleteNumber:(int)number{

    // 1.
    [self openDB];

    // 2.stmt跟随指针(数据库句柄)
    sqlite3_stmt * stmt= nil;

    // 3.
    NSString * sql = @"DELETE FROM Student WHERE number = ?";

    // 4.
    int result = sqlite3_prepare_v2(dbSqlite, [sql UTF8String], -1, &stmt, NULL);
    if (result == SQLITE_OK) {

        // 5.
        sqlite3_bind_int(stmt,1, number);

        // 6.
        sqlite3_step(stmt);
    }

    // 7.
    sqlite3_finalize(stmt);

    // 8.
    [self closeDB];

}

#pragma mark - 查询数据操作
- (NSArray *)selectAllData{

    NSMutableArray * allDataArray = [NSMutableArray array];

    // 1.
    dbSqlite = [self openDB];

    // 2.类似JAVA句柄?
    sqlite3_stmt * stmt = nil;

    // 3.设置查询数据语句SELECT
    NSString * sql = @"SELECT name FROM Student";

    // 4.编译
    int result = sqlite3_prepare_v2(dbSqlite, [sql UTF8String], -1, &stmt, NULL);
    if (result == SQLITE_OK) {

        // 5.查询数据
        while (sqlite3_step(stmt) == SQLITE_ROW) {

            // 6.查询的索引下标从0开始(每一次循环后,下标+1)
            const unsigned char * name = sqlite3_column_text(stmt, 0);
            //NSString * nameString = [NSString stringWithUTF8String:(const char *)name];
            NSString * nameString = [NSString stringWithCString:(const char *)name encoding:NSUTF8StringEncoding];
            [allDataArray addObject:nameString];
        }

        // 8.关闭数据库句柄
        sqlite3_finalize(stmt);

        // 9.
        [self closeDB];

        return allDataArray;
    }

    return nil;
}

- (NSMutableDictionary *)selectOneDataWithNumber:(int)number{

    sqlite3 * _dbSqlite = [self openDB];

    sqlite3_stmt * stmt = nil;

    NSString * sql = @"SELECT * FROM Student WHERE number = ?";

    int result = sqlite3_prepare_v2(_dbSqlite, [sql UTF8String], -1, &stmt, NULL);

    if (result == SQLITE_OK) {

        sqlite3_bind_int(stmt, 1, number);
        while (sqlite3_step(stmt) == SQLITE_ROW) {

            int number = sqlite3_column_int(stmt, 0);
            NSString * name = [NSString stringWithUTF8String:(const char *)sqlite3_column_text(stmt, 1)];
            int age = sqlite3_column_int(stmt, 2);
            NSString * sex = [NSString stringWithUTF8String:(const char *)sqlite3_column_text(stmt, 3)];
            UIImage * image = [UIImage imageWithData:[NSData dataWithBytes:sqlite3_column_blob(stmt, 4) length:sqlite3_column_bytes(stmt, 4)]];

            NSMutableDictionary * dic = [NSMutableDictionary dictionaryWithObjectsAndKeys:[NSNumber numberWithInt:number],@"number",name,@"name",[NSNumber numberWithInt:age],@"age",sex,@"sex",image,@"image", nil];

            // 关闭数据库句柄
            sqlite3_finalize(stmt);
            return dic;
        }

    }

    return nil;
}

@end


@SQLite支持哪些数据类型些？
NULL 值为NULL
INTEGER 值为带符号的整型，根据类别用1，2，3，4，6，8字节存储
REAL 值为浮点型，8字节存储
TEXT 值为text字符串，使用数据库编码(UTF-8, UTF-16BE or UTF-16-LE)存储
BLOB 值为二进制数据，具体看实际输入
---------但实际上，sqlite3也接受如下的数据类型：
smallint  16 位元的整数
interger  32 位元的整数
decimal(p,s)  p 精确值和 s 大小的十进位整数，精确值p是指全部有几个数(digits)大小值 ，s是指小数点後有几位数。如果没有特别指定，则系统会设为 p=5; s=0 。
float   32位元的实数。
double   64位元的实数。
char(n)   n 长度的字串，n不能超过 254。
varchar(n)  长度不固定且其最大长度为 n 的字串，n不能超过 4000。
graphic(n)  和 char(n) 一样，不过其单位是两个字元 double-bytes， n不能超过127。 这个形态是为了支援两个字元长度的字体，例如中文字。
vargraphic(n)  可变长度且其最大长度为 n 的双字元字串，n不能超过 2000。
date   包含了 年份、月份、日期。
time   包含了 小时、分钟、秒。
timestamp  包含了 年、月、日、时、分、秒、千分之一秒。

@如果将某个字段设置为INTEGER PRIMARY KEY属性，有什么特性？
如果将声明表的一列设置为 INTEGER PRIMARY KEY，则具有：
1．每当你在该列上插入一NULL值时， NULL自动被转换为一个比该列中最大值大1的一个整数；
2．如果表是空的， 将会是1；
---------注意该整数会比表中该列上的插入之前的最大值大1。 该键值在当前的表中是唯一的。但有可能与已从表中删除的值重叠。要
想建立在整个表的生命周期中唯一的键值，需要在 INTEGER PRIMARY KEY 上增加AUTOINCREMENT声明。那么，新的键值将会比该表中曾能存在过的最大值大1。

@字段声明中有AUTOINCREMENT属性，有什么与众不同的含义？
要想建立在整个表的生命周期中唯一的键值，需要在 INTEGER PRIMARY KEY 上增加AUTOINCREMENT声明。那么，新的键值将会比该表中曾能存在过的最大值大1。


@SQL常用命令使用方法：

(1) 数据记录筛选：

sql="select * from 数据表 where 字段名=字段值 order by 字段名 [desc]"

sql="select * from 数据表 where 字段名 like %字段值% order by 字段名 [desc]"

sql="select top 10 * from 数据表 where 字段名 order by 字段名 [desc]"

sql="select * from 数据表 where 字段名 in (值1,值2,值3)"

sql="select * from 数据表 where 字段名 between 值1 and 值2"

(2) 更新数据记录：

sql="update 数据表 set 字段名=字段值 where 条件表达式"

sql="update 数据表 set 字段1=值1,字段2=值2 …… 字段n=值n where 条件表达式"

(3) 删除数据记录：

sql="delete from 数据表 where 条件表达式"

sql="delete from 数据表" (将数据表所有记录删除)

(4) 添加数据记录：

sql="insert into 数据表 (字段1,字段2,字段3 …) valuess (值1,值2,值3 …)"

sql="insert into 目标数据表 select * from 源数据表" (把源数据表的记录添加到目标数据表)

(5) 数据记录统计函数：

AVG(字段名) 得出一个表格栏平均值
COUNT(*|字段名) 对数据行数的统计或对某一栏有值的数据行数统计
MAX(字段名) 取得一个表格栏最大的值
MIN(字段名) 取得一个表格栏最小的值
SUM(字段名) 把数据栏的值相加

引用以上函数的方法：

sql="select sum(字段名) as 别名 from 数据表 where 条件表达式"
set rs=conn.excute(sql)

用 rs("别名") 获取统的计值，其它函数运用同上。

(6) 数据表的建立和删除：

CREATE TABLE 数据表名称(字段1 类型1(长度),字段2 类型2(长度) …… )

例：CREATE TABLE tab01(name varchar(50),datetime default now())

DROP TABLE 数据表名称 (永久性删除一个数据表)

@sqlite中用LIMIT返回前2行，语句如下： 
select * from aa order by ids desc LIMIT 2

```



