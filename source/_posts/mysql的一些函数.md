---
title: mysql的一些函数
date: 2018-01-23 20:03:45
tags: mysql
categories: 数据库
description: mysql的一些函数
---
##### 字符串函数
- `ascii(str)` 返回字符串str的第一个字符的ascii值(str是空串时返回0)
- `ord(str)` 返回 若字符串str的最左字符是一个多字节字符,则返回该字符的代码,代码的计算通过使用以下公式计算其组成字节的数值而得出: (1st byte code) + (2nd byte code × 256) + (3rd byte code × 256^2) ...
- `conv(n,from_base,to_base)` 对数字n进制转换,并转换为字串返回(任何参数为null时返回null,进制范围为2-36进制,当to_base是负数时n作为有符号数否则作无符号数,conv以64位点精度工作) 
```
mysql> select conv("a",16,2);  
　　-> '1010'   
mysql> select conv("6e",18,8);    
　　-> '172'   
mysql> select conv(-17,10,-18);  
　　-> '-h'   
```
- `bin(n)` 把n转为二进制值并以字串返回(n是bigint数字,等价于conv(n,10,2))  
```
mysql> select bin(12);     
　　-> '1100' 
```
- `oct(n)` 把n转为八进制值并以字串返回(n是bigint数字,等价于conv(n,10,8))
- `hex(n)` 把n转为十六进制并以字串返回(n是bigint数字,等价于conv(n,10,16))
- `char(n,...)` 返回由参数n,...对应的ascii代码字符组成的一个字串(参数是n,...是数字序列,null值被跳过)
```
mysql> select char(77,121,83,81,'76');  
　　-> 'mysql' 
```
- `concat(str1,str2,...)` 把参数连成一个长字符串并返回(任何参数是null时返回null)
- `length(str)`,`octet_length(str)`,`char_length(str)`,`character_length(str)` 返回字符串str的长度(对于多字节字符char_length仅计算一次)
- `locate(substr,str)`,`position(substr in str)` 返回字符串substr在字符串str第一次出现的位置(str不包含substr时返回0) 
```
mysql> select locate('bar', 'foobarbar');  
　　-> 4  
```
- `locate(substr,str,pos)` 返回字符串substr在字符串str的第pos个位置起第一次出现的位置(str不包含substr时返回0)
```
mysql> select locate('bar', 'foobarbar',5);  
　　-> 7 
```
- `instr(str,substr)` 返回字符串substr在字符串str第一次出现的位置(str不包含substr时返回0)
- `lpad(str,len,padstr)` 用字符串padstr填补str左端直到字串长度为len并返回
```
mysql> select lpad('hi',4,'??*');  
　　-> '??hi'  
```
- `rpad(str,len,padstr)` 用字符串padstr填补str右端直到字串长度为len并返回
- `left(str,len)` 返回字符串str的左端len个字符
- `right(str,len)` 返回字符串str的右端len个字符
- `substring(str,pos,len)`,`substring(str from pos for len)`,`mid(str,pos,len)` 返回字符串str的位置pos起len个字符
- `substring(str,pos)`,`substring(str from pos)` 返回字符串str的位置pos起的一个子串
- `substring_index(str,delim,count)` 返回从字符串str的第count个出现的分隔符delim之后的子串(count为正数时返回左端,否则返回右端子串)
```
mysql> select substring_index('www.mysql.com', '.', 2);  
　　-> 'www.mysql' 
mysql> select substring_index('www.mysql.com', '.', -2);  
　　-> 'mysql.com' 
```
- `ltrim(str)` 返回删除了左空格的字符串str
```
mysql> select ltrim('  barbar');  
　　-> 'barbar'   
```
- `rtrim(str) ` 返回删除了右空格的字符串str
- `trim([[both | leading | trailing] [remstr] from] str) ` 返回前缀或后缀remstr被删除了的字符串str(位置参数默认both,remstr默认值为空格)
```
mysql> select trim('  bar   ');  
　　-> 'bar' 
mysql> select trim(leading 'x' from 'xxxbarxxx');  
　　-> 'barxxx' 
mysql> select trim(both 'x' from 'xxxbarxxx');  
　　-> 'bar' 
mysql> select trim(trailing 'xyz' from 'barxxyz');  
　　-> 'barx' 
```
- `soundex(str)` 返回str的一个同音字符串(听起来“大致相同”字符串有相同的同音字符串,非数字字母字符被忽略,在a-z外的字母被当作元音)
```
mysql> select soundex('hello');  
　　-> 'h400' 
mysql> select soundex('quadratically');  
　　-> 'q36324' 
```
- `space(n)` 返回由n个空格字符组成的一个字符串
- `replace(str,from_str,to_str)` 用字符串to_str替换字符串str中的子串from_str并返回
- `reverse(str)` 颠倒字符串str的字符顺序并返回
- `insert(str,pos,len,newstr)` 把字符串str由位置pos起len个字符长的子串替换为字符串
newstr并返回 
- `elt(n,str1,str2,str3,...)` 返回第n个字符串(n小于1或大于参数个数返回null)  
- `field(str,str1,str2,str3,...)` 返回str等于其后的第n个字符串的序号(如果str没找到返回0)
- `find_in_set(str,strlist)` 返回str在字符串集strlist中的序号(任何参数是null则返回null,如果str没找到返回0,参数1包含","时工作异常)
- `make_set(bits,str1,str2,...)` 把参数bits的数字转为二进制,假如某个位置的二进制位等于bits,对应位置的字串选入字串集并返回(null串不添加到结果中) 
```
mysql> select make_set(1,'a','b','c');  
　　-> 'a' 
mysql> select make_set(1 | 4,'hello','nice','world');  
　　-> 'hello,world' 
mysql> select make_set(0,'a','b','c');  
　　-> '' 
```
- `export_set(bits,on,off,[separator,[number_of_bits]])` 按bits排列字符串集,只有当位等于1时插入字串on,否则插入off(separator默认值",",number_of_bits参数使用时长度不足补0而过长截断)
```
mysql> select export_set(5,'y','n',',',4)  
　　-> y,n,y,n   
```
- `lcase(str)`,`lower(str)` 返回小写的字符串str
- `ucase(str)`,`upper(str)` 返回大写的字符串str
- `load_file(file_name)` 读入文件并且作为一个字符串返回文件内容(文件无法找到,路径不完整,没有权限,长度大于max_allowed_packet会返回null)
```
mysql> update table_name set blob_column=load_file  
("/tmp/picture") where id=1;   
```
##### 数学函数
- `abs(n)` 返回n的绝对值 
- `sign(n)` 返回参数的符号(为-1、0或1)
- `mod(n,m)` 取模运算,返回n被m除的余数(同%操作符)
- `floor(n)` 返回不大于n的最大整数值
- `ceiling(n)` 返回不小于n的最小整数值
- `round(n,d)` 返回n的四舍五入值,保留d位小数(d的默认值为0)
- `exp(n)` 返回值e的n次方(自然对数的底)
- `log(n)` 返回n的自然对数
- `log10(n)` 返回n以10为底的对数
- `pow(x,y)`,`power(x,y)` 返回值x的y次幂
- `sqrt(n)` 返回非负数n的平方根
- `pi()` 返回圆周率
- `cos(n)` 返回n的余弦值
- `sin(n)` 返回n的正弦值
- `tan(n)` 返回n的正切值
- `acos(n)` 返回n反余弦(n是余弦值,在-1到1的范围,否则返回null)
- `asin(n)` 返回n反正弦值
- `atan(n)` 返回n的反正切值 
- `atan2(x,y)` 返回2个变量x和y的反正切(类似y/x的反正切,符号决定象限)
- `cot(n)` 返回x的余切 
- `rand()`,`rand(n)` 返回在范围0到1.0内的随机浮点值(可以使用数字n作为初始值)
- `degrees(n)` 把n从弧度变换为角度并返回
- `radians(n)` 把n从角度变换为弧度并返回
- `truncate(n,d)` 保留数字n的d位小数并返回
- `least(x,y,...)` 返回最小值(如果返回值被用在整数(实数或大小敏感字串)上下文或所有参数都是整数(实数或大小敏感字串)则他们作为整数(实数或大小敏感字串)比较,否则按忽略大小写的字符串被比较)
- `greatest(x,y,...)` 返回最大值(其余同least())

##### 时间日期函数
- `dayofweek(date)` 返回日期date是星期几(1=星期天,2=星期一,……7=星期六,odbc标准)
- `weekday(date)` 返回日期date是星期几(0=星期一,1=星期二,……6= 星期天)
- `dayofmonth(date)` 返回date是一月中的第几日(在1到31范围内)
- `dayofyear(date)` 返回date是一年中的第几日(在1到366范围内)
- `month(date)` 返回date中的月份数值
- `dayname(date)` 返回date是星期几(按英文名返回)
- `monthname(date)` 返回date是几月(按英文名返回) 
- `quarter(date)` 返回date是一年的第几个季度
- `week(date,first)` 返回date是一年的第几周(first默认值0,first取值1表示周一是周的开始,0从周日开始) 
- `year(date)` 返回date的年份(范围在1000到9999)
```
mysql> select year('98-02-03');    
　　-> 1998    
```
- `hour(time)` 返回time的小时数(范围是0到23)
```
mysql> select hour('10:05:03');    
　　-> 10   
```
- `minute(time)` 返回time的分钟数(范围是0到59)
- `second(time)` 返回time的秒数(范围是0到59)
- `period_add(p,n)` 增加n个月到时期p并返回(p的格式yymm或yyyymm)
```
mysql> select period_add(9801,2);    
　　-> 199803    
```
- `period_diff(p1,p2)` 返回在时期p1和p2之间月数(p1和p2的格式yymm或yyyymm)
- `date_add(date,interval expr type)`   
	`date_sub(date,interval expr type)` 
	`adddate(date,interval expr type)`  
	`subdate(date,interval expr type)`  
对日期时间进行加减法运算,adddate()和subdate()是date_add()和date_sub()的同义词,也
可以用运算符+和-而不是函数,date是一个datetime或date值,expr对date进行加减法的一个表
达式字符串,type指明表达式expr应该如何被解释 
　[type值 含义 期望的expr格式]:  
　second 秒 seconds    
　minute 分钟 minutes    
　hour 时间 hours    
　day 天 days    
　month 月 months    
　year 年 years    
　minute_second 分钟和秒 "minutes:seconds"    
　hour_minute 小时和分钟 "hours:minutes"    
　day_hour 天和小时 "days hours"    
　year_month 年和月 "years-months"    
　hour_second 小时, 分钟， "hours:minutes:seconds"    
　day_minute 天, 小时, 分钟 "days hours:minutes"    
　day_second 天, 小时, 分钟, 秒 "days
hours:minutes:seconds" 
　expr中允许任何标点做分隔符,如果所有是date值时结果是一个
date值,否则结果是一个datetime值
　如果type关键词不完整,则mysql从右端取值,day_second因为缺
少小时分钟等于minute_second
　如果增加month、year_month或year,天数大于结果月份的最大天
数则使用最大天数
```
mysql> select "1997-12-31 23:59:59" + interval 1 second;  
　　-> 1998-01-01 00:00:00    
mysql> select interval 1 day + "1997-12-31";    
　　-> 1998-01-01    
mysql> select "1998-01-01" - interval 1 second;    
　　-> 1997-12-31 23:59:59    
mysql> select date_add("1997-12-31 23:59:59",interval 1
second);    
　　-> 1998-01-01 00:00:00    
mysql> select date_add("1997-12-31 23:59:59",interval 1
day);    
　　-> 1998-01-01 23:59:59    
mysql> select date_add("1997-12-31 23:59:59",interval
"1:1" minute_second);    
　　-> 1998-01-01 00:01:00    
mysql> select date_sub("1998-01-01 00:00:00",interval "1
1:1:1" day_second);    
　　-> 1997-12-30 22:58:59    
mysql> select date_add("1998-01-01 00:00:00", interval "-1
10" day_hour);  
　　-> 1997-12-30 14:00:00    
mysql> select date_sub("1998-01-02", interval 31 day);    
　　-> 1997-12-02    
mysql> select extract(year from "1999-07-02");    
　　-> 1999    
mysql> select extract(year_month from "1999-07-02
01:02:03");    
　　-> 199907    
mysql> select extract(day_minute from "1999-07-02
01:02:03");    
　　-> 20102  
```
- `to_days(date)` 返回日期date是西元0年至今多少天(不计算1582年以前)
- `from_days(n)` 给出西元0年至今多少天返回date值(不计算1582年以前)   
- `date_format(date,format)` 根据format字符串格式化date值  
	- 在format字符串中可用标志符:  
			%m 月名字(january……december)    
			%w 星期名字(sunday……saturday)    
			%d 有英语前缀的月份的日期(1st, 2nd, 3rd, 等等。）    
			%y 年, 数字, 4 位    
			%y 年, 数字, 2 位    
			%a 缩写的星期名字(sun……sat)    
			%d 月份中的天数, 数字(00……31)    
			%e 月份中的天数, 数字(0……31)    
			%m 月, 数字(01……12)    
			%c 月, 数字(1……12)    
			%b 缩写的月份名字(jan……dec)    
			%j 一年中的天数(001……366)    
			%h 小时(00……23)    
			%k 小时(0……23)    
			%h 小时(01……12)    
			%i 小时(01……12)    
			%l 小时(1……12)    
			%i 分钟, 数字(00……59)    
			%r 时间,12 小时(hh:mm:ss [ap]m)    
			%t 时间,24 小时(hh:mm:ss)    
			%s 秒(00……59)    
			%s 秒(00……59)    
			%p am或pm    
			%w 一个星期中的天数(0=sunday ……6=saturday ）    
			%u 星期(0……52), 这里星期天是星期的第一天    
			%u 星期(0……52), 这里星期一是星期的第一天    
			%% 字符%
- `curdate()`,`current_date()` 以'yyyy-mm-dd'或yyyymmdd格式返回当前日期值(根据返回值所处上下文是字符串或数字)
- `curtime()`,`current_time()` 以'hh:mm:ss'或hhmmss格式返回当前时间值(根据返回值所处上下文是字符串或数字)
- `now()`,`sysdate()`,`current_timestamp()` 　以'yyyy-mm-dd hh:mm:ss'或yyyymmddhhmmss格式返回当前日期时间(根据返回值所处上下文是字符串或数字) 
- `unix_timestamp() `,`unix_timestamp(date)` 返回一个unix时间戳(从'1970-01-01 00:00:00'gmt开始的秒数,date默认值为当前时间)
- `from_unixtime(unix_timestamp)` 以'yyyy-mm-dd hh:mm:ss'或yyyymmddhhmmss格式返回时间戳的值(根据返回值所处上下文是字符串或数字)
- `from_unixtime(unix_timestamp,format)` 以format字符串格式返回时间戳的值
- `sec_to_time(seconds)` 以'hh:mm:ss'或hhmmss格式返回秒数转成的time值(根据返回值所处上下文是字符串或数字)
- `time_to_sec(time)` 返回time值有多少秒 

##### 转换函数
- `cast`
	- 用法：cast(字段 as 数据类型) [当然是否可以成功转换，还要看数据类型强制转化时注意的问题]
	- 实例：select cast(a as unsigned) as b from cardserver where order by b desc;
- `convert`
	- 用法：convert(字段,数据类型)
	- 实例：select convert(a ,unsigned) as b from cardserver where order by b desc;
