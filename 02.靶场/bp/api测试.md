# Exploiting an API endpoint using documentation
发现一个api请求，更改方法为delete，删除carlos
DELETE /api/user/carlos HTTP/2

# 利用查询字符串中的服务器端参数污染
Lab: Exploiting server-side parameter pollution in a query string
要求：登陆管理员账号删除carlos
忘记密码功能点输入administrator
![[Pasted image 20241127141510.png]]
看见有个forgotpassword的js
![[Pasted image 20241127141810.png]]
/forgot-password?reset_token=${resetToken}看js内容，有这个路径

```POST请求体
csrf=VcOTyYojxc25BBOOiu5Tx2MM9khcuFD1&username=administrator
```
截断发现响应不一样，通过爆破参数，看返回包等操作
构造这个
```
username=administrator%26field=reset_token%23
```
再去访问/forgot-password?reset_token=123456789，这里的功能点是带着这个token就能该密码

# 查找并利用未使用的 API 端点
Lab: Finding and exploiting an unused API endpoint
+++
要求：购买夹克，0元购
先options请求，发现patch请求方法可以用，patch可以更新服务器的资源
用patch请求方法更新商品信息
要根据返回包构造json，参数

# 利用大量赋值漏洞
Lab: Exploiting a mass assignment vulnerability
++++
要求也是购买夹克
get请求api发现响应有chosen_discount参数
而post请求请求体没有这个参数，猜测这个参数控制折扣百分比
我们给他加上，赋值100，直接拿下
这里post请求是结账用的

# 利用 REST URL 中的服务器端参数污染
Lab: Exploiting server-side parameter pollution in a REST URL
要求登陆管理员账号

username=administrator返回result":"*****@normal-user.net
username=./administrator返回一样
username=administrator%3F返回"result": "Invalid route. Please refer to the API definition"
username=administrator#返回和%3F一样
username=administrator/../同上
这表明请求可能访问了一个无效的 URL 路径。
说明这个参数在服务端请求中的url上

username=../../../../openapi.json返回/api/internal/v1/users/{username}/field/{field}

后面根据返回包继续思考，重点是上面判断参数拼接在服务端请求resturl