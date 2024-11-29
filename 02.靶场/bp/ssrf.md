# 针对本地服务器的基本 SSRF

通过点击该按钮
![[Pasted image 20240917155613.png]]
我们看到了这样的post请求，放到repeter，猜测一下让他访问本身的admin，于是看到了这样的画面
![[Pasted image 20240917160114.png]]
看起来删除是个链接，我们读一下源码，发现了触发的链接，并且同样让服务器本身访问那里
![[Pasted image 20240917160223.png]]![[Pasted image 20240917160318.png]]

# 带外带检测的盲 SSRF
Lab: Blind SSRF with out-of-band detection

漏洞点在referer，结合bp的dnslog