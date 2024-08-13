https://portswigger.net/web-security/cross-site-scripting#how-does-xss-work
1.闭合h1标签
```html
</h1><script>alert(1)</script>\\
```
3.dom型xss，在检查里查看，查看页面源代码看不见的
```html
xxx" onload="alert(1)  \\构造前后闭合
```
情景：<a id="backLink" href="/xxx">Back</a>