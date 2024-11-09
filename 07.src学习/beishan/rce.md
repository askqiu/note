aksk泄露，可能前端有加密，要解密
js，小程序反编译，app的java代码

常见组件漏洞导致rce，拿到的poc要新，绕waf，说白了就是在应急和安全研究员之后捡漏
- fastjson所有版本漏洞
- log4j
- mssql
- chrom
- springboot actuator env heapdump等路径

觉得经常可能碰到的新爆出来的cve就复现一下，其他的就看一下就行
敏感信息rce，aksk，redis，mysql

不需要懂原理
- 怎么找到这个漏洞
- 怎么利用漏洞rce
所以要提前复现

ssti
https://mp.weixin.qq.com/s/-wSFSweL1my-kUDB3gq0ng
不常见，一般需要用到html渲染的业务功能就很可能存在，比如有邮件相关的可能有，并且伴随xss
一些简单输出的地方（开发可能为了html渲染，懒得写html文件，简单写了几句直接渲染字符串）

ssrf
https://cloud.tencent.com/developer/article/1968655?areaSource=105001.3&traceld=FyZTpLc2ixMvGwJf2-pDn
国内服务器很难碰到aksk能访问，因为有未知变量
aws可以

ssrf无回显打redis，mysql，全回显

命令注入
文件名等，经过后端代码处理可能会使用命令执行的代码 