# _MYSQL_SQL_BYPASS_ 

----------


待更新

----------

大家都时间可以逛逛圈子社区，土司论坛，目前论坛已经很少了其中能学到不少知识。

本文1.x属于科普文章,其中讲到一些注入的细节问题，核心还是bypass，由于时间问题我只选了3个waf来讲解，后期可能有补充。


这个教程的目的是让大家对SQL注入和bypass 有一定的了解，攻防一体，熟悉攻击手法才能更好的防御，文章可能会有笔误的地方，如有发现还请斧正



暂定目录


```
注入基础

1. 基本语句
2. 默认表名解读
3. 符号
4. 注入的产生
5. 数据库信息收集
6. 初识注入bypass
7. 报错注入
8. 盲注
9. insert,delete,update注入
10. 二次注入与宽字节注入
11. order,limit和from后的注入
12. 再谈万能密码登陆
13. 读写文件与堆叠查询

常见WAF bypass

1. 我有一千种 联合注入过狗
2. 我有一千种 盲注过狗
3. 我有一千种 报错过狗
4. 矛与云盾
5. 我与云锁有个约会
6. 拓展Bypass的常用思路

拓展 正则绕过

1. 金典正则过滤下的绕过(一)
2. 金典正则过滤下的绕过(二)

```

----------


本人撰写的文章，仅供学习和研究使用，请勿使用文中的技术源码用于非法用途，任何人造成的任何负面影响，与本人无关。

![](./img/404.png)