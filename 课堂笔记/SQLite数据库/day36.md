## SQLite

### sqlite命令

>打开数据库文件：**(没有则创建)**
>
>方式1：sqlite3 数据库文件名.db（进入sqlite终端并**打开**数据库文件）
>方式2：1.sqlite3 2.使用.open 数据库文件名

```sqlite
.help  打开帮助手册
.open  打开数据库文件
.exit  退出 .quit  退出  使用  .q 也可以
.tables 查看数据库中有哪些数据表
.schema 查看表结构(建表语句)
.headers on|off  查询结果是否显示字段名(on显示  off不显示)
```

### 操作语句

> sql语句不区分大小写

#### DDL-数据定义语言

1.创建表：(create table …)(primary key只有在建表是才能指定**主键**)

​		 (if not exists)(as)

```sqlite
CREATE TABLE 表名(字段 类型，...)；
类型：
整数  INT integer
字符串 CHAR text（需要用单引号或双引号括起来）
```

2.修改表:(alter table …)(add colume，rename to)

```sqlite
#添加一列 (add column)
alter table stu add column gender text;

#sqlite3不允许直接删除一列
1.先创建一张新表 (as)
create table temp as select id,name,score from stu;
2.删除原来的旧表
drop table stu;
3.对新表重命名 (rename to)
alter table temp rename to stu;
```

3.删除表（drop table …）

```sqlite
drop table stu;
```

#### DML-数据操作语言

1.插入数据：(insert into … values)

```sqlite
insert into 表名 values(123,"张三",100);//这种方式每个字段都要赋值
insert into student(id,name) values(123,"张三");//选择给指定字段赋值
```

2.修改数据：(update … set)(where and or)

```sqlite
update stu set score=93 where name="zhang";
update stu set name='liu',score=93 where name="zhang";
```

3.删除数据：(delete from …)(where and or)

```sqlite
delete from stu where id=1 or name='liu';
```

#### DQL-数据查询语言

1.查询数据：(select # from … )(where and or)(order by asc desc)

```sqlite
select * from 表名;
select id,score from stu;
select * from stu where score=97;//条件查询
select id from stu where gender='男' and score=95;
select id from stu where gender='男' or score=95;
select * from stu order by socer desc;//不写默认asc
```

## 常用API

> 编译时链接库 **-lsqlite3**

```c
//1.打开数据库文件的函数
int sqlite3_open(const char *filename,   /* 数据库文件名(路径) */
                         sqlite3 **ppdb		/* OUT: SQLite db handle */);
功能：
    打开一个数据库，如果存在则直接打开 不存在就新建并打开
参数：
    filename	数据库名字
    ppdb		操作数据库的指针，句柄。
返回值：
    成功	SQLITE_OK
    失败	错误码

//2.获取错误信息的函数
const char *sqlite3_errmsg(sqlite3* db);
功能：返回错误信息，获取的是最后一次出错的信息

//3.关闭数据库的函数
int sqlite3_close(sqlite3* db);
功能：关闭一个数据库

//4.执行sql语句的函数
int sqlite3_exec(sqlite3* db, const char *sql, 
           		 int (*callback)(void*,int,char**,char**), void *arg, char **errmsg);
功能：
    执行一条sql语句
参数：
    db		数据库的句柄指针
    sql		将要被执行sql语句
    callback 回调函数, 只有在查询语句时,才会使用回调函数
    arg		为callback 传参的，只有在查询语句时，才给回调函数传参
    errmsg	错误信息的地址
        一般不使用 如果使用了 需要用 sqlite3_free 释放空间
返回值：
    成功	SQLITE_OK
    出错	错误码 
int sqlite3_free(errmsg);
    
--------------------关于sqlite3_exec的回调函数------------------
int (*callback)(void *arg ,int ncolumn, char** f_value, char** f_name)
功能：
    得到查询结果
参数：
    arg		为回调函数传递参数使用的
    ncolumn	记录中包含的字段的数目
    f_value	包含每个字段值的指针数组
    f_name	包含每个字段名称的指针数组
返回值：
    成功	SQLITE_OK
    出错	错误码 

//5.查询数据库的函数
int sqlite3_get_table( sqlite3 *db, const char *zSql, char ***pazResult, 
              int *nRow, int *nColumn, char **pzErrmsg);
功能：
    查询数据库，它会创建一个新的内存区域来存放查询的结果信息
参数：
    db			数据库操作句柄
    sql			数据库的sql语句
    pazResult	查询的结果
    nRow		查询的结果中不包含表头的行数
    nColumn		列数
    pzErrmsg	错误消息  和 前面的errmsg使用方式一样
返回值：
    成功	SQLITE_OK
    出错	错误码

//6.释放sqlite3_get_table产生的结果集
void sqlite3_free_table(char **result);
```

