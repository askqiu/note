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

# 基于黑名单输入过滤器的 SSRF
Lab: SSRF with blacklist-based input filter

要求通过ssrf访问localhost/admin
不过要绕过黑名单防护
这里我的payload是
stockApi=http://%31%32%37%2e%31/%25%36%31dmin
其中对字母a进行了双重url编码

# Ssrf结合重定向进行绕过
SSRF with filter bypass via open redirection vulnerability

这个路径下存在重定向漏洞，/product/nextProduct?currentProductId=4%26path=
我们让存在ssrf的点去访问具有开放重定向漏洞的点，从而绕过只能访问自己的防御
最终poc
stockApi=/product/nextProduct?currentProductId=4%26path=http://192.168.0.12:8080/admin/delete?username=carlos
