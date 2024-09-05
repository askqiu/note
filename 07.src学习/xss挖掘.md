注意一下那个国外白帽子的xss文档pdf，总结很好

参数提取urlfurl
# 账号创建
第一步创建一个账号，尽量用hackerone给的别名，比如说shadovv@wearehackerone.com
第一个账号可以当成受害者账号，探索功能
注册多个账号
shadovv+1@wearehackerone.com
shadovv+2@wearehackerone.com

# 同源策略
同源策略限制了不同源之间如何进行资源交互，是用于隔离潜在恶意文件的重要安全机制。 是否
同源由URL决定，URL由协议、域名、端口和路径组成，如果两个URL的协议、域名和端口相同，
则表示他们同源

http:www.baidu.com:8080与https:www.baidu.com:8080不同源（协议不同）

# 升级危害
cors，csrf，越权

## 场景
在sub.aaa.com发现一个xss，同时这地方可注册可登陆功能很多，这个xss可以如何来升级危害
1. 修改邮箱，然后重置密码（用邮箱登陆，或者重置密码和邮箱有关的情况下）
2. 修改密码
3. 添加一个sso登陆
4. 读取sessiontoken在某个页面
5. 给这个组织添加一个管理员
6. 读取apikey
7. 通过oauth账号接管

**可以升级危害的情况**：
与cors配置有关
第一种就是sub1.aaa.com或者某种比较核心的重要子域名存在cors 配置为任意子域名可以读取它的内容，那么sub2.aaa.com 就能读取这些域名的内容进行进一步升级危  害

第二种：与httponly有关（难搞的）
![[Pasted image 20240904211738.png]]
# 案例

## 1（绕过）
此处单引号双引号都过滤了，在url中尝试%2"2发现%22直接输出在了html上，就是说没解码，所以还是不行
蓝色部分其实是js部分，于是我们考虑用-alert()-来弹窗，这样的话我们就需要用单引号闭合前后的单引号，但是被过滤了，所以考虑用html编码来闭合前后的单引号，单引号的html编码是&#x27，要对特殊符号编码url，所以最终payload是xxxxx%26%23x27;-alert()-%26%23x27;(这里不加；好像也可以)
## 2（绕过，危害证明）
递归过滤的一个点，为啥11111<<img\> img/src='a'/onerror=alert(1)>2222荷载可以绕过过滤？
：因为第二个img前面有个空格
`荷载<<img> img 输入以后，过滤器检测到<<img> img, 过滤器首先过滤的是里面的<img>, 这`
`个时候还剩下< img, 按照前面的过滤情况来看 ， 因为是递归删除的，< img 应该也被删除才对，`
`但是实际上并没有，最后保留下来了< img， 注意前面带了一个空格`
`出现这种情况的原因是我朋友告诉我这是一个补充规则，因为他之前已经提交过一次了，一开始`
`用的荷载是< img 前面带上了一个空格，一开始的规则只是递归删除<后面跟在字符的模式(比如`
`<aaa, <bbb, <cccc)`
`所以删除< img 其实只是一个在我朋友提交第一次报告以后，开发人员偷懒补充的一个规则，`
`大概就是删除<空格跟着字符这种模式，没有加入递归删除的规则里面`
所以，可以猜测开发人员修复逻辑进行二次挖掘，但是这里有空格所以img不会触发，但是我们发现![[Pasted image 20240905145212.png]]
只能弹个1无法接受危害，如何升级危害呢![[Pasted image 20240905150044.png]]

## 3（账号接管）
已知下面的cookie 没有设置http only 标记，所以可以通过XSS 直接读取
WC-AUTHENTICATION-{UserId}
WCU-SERACTIVITY-{UserId}
MS-USER-COOKIE-{id}
但是__Secure-next-auth.session-token 设置了http only 标记，请问如何读取这个cookie？
最直接方法就是找到一个页面，
\__Secure-next-auth.session-token cookie的内容会出现在那个页面、
然后通过XSS 读取那个页面的内容即可
那个页面找起来也不难，随便点点点功能点，然后检查burp的历史记录
最后找到了 /api/auth/session 这么一个路径
怎么操作呢？
![[Pasted image 20240905152150.png]]
## glassdoor
```
信息收集：gau|waybackurls对www.glassdoor.com进行收集
得到百万路径，先不处理，
提取参数，用unfurl那工具），做成字典，初次提取有两千多字典，手动整理之后还剩900+（把看着完全不可能的参数手动删除掉，比如带*的，带！的，但是带有-和_的参数需要保留）
获取相关信息：查看glass的公开报告，发现大多数是关于反射xss的，存储xss是通过selfxss+缓存漏洞完成，打开xss相关报告，把对应站点的xsspayload记录一下
通过比对报告漏洞的路径与参数和我们收集到的路径与参数，看看覆盖率，从而判断一下我们的信息收集手法对于这个厂商来说对不对
比如说这里，通过gau和waybackurl收集到的和报告中的路径参数覆盖率还是比较高的


```
![[bacc6717ffbb64420f416d08c88a760 1.jpg]]
![[Pasted image 20240905212121.png]]
# 有关referer的xss
```
在进行xss 测试的时候adco 这个参数存在反射XSS，但是当我在浏览器访问的时候发现无法触发
经过在burp 反复对比数据包，发现是referer 影响了结果
Referer: 可以是任意域名/widget/可以是任意的
Referer 头需要带上widget 这个XSS才能成功利用
Referer 包含widget的时候，\ 输出还是\ , 但是’会变成\’, 两个\\一组合，刚好可以打破’被转义
Referer 不包含widget的时候，\的输出是\\, 加上’ 变成\’, 所以一共3个\, 最后的结果就是无法打破上
下文，所以不存在漏洞
```

在你的域名放xss.html这么一个文件，这个文件的内容如下
\<a href="https://{漏洞所在域名}/js/widget/?adco=\',1:alert(document.domain),//"
referrerpolicy="no-referrer-when-downgrade">XSS test</a>
访问https://{你的域名}/widget/xss.html
然后点击XSS test就会重定向到https://{漏洞所在域
名}/js/widget/?adco=\',1:alert(document.domain),//
此时Referer 被正确设置，所以XSS会触发（其实也有办法进行自动重定向）

这里下文还有继续缓存特性利用进行劫持登陆页面的，在part11
现实攻击场景：
1. 攻击者通过邮箱发送信息给受害者让他修改自己的booking 信息
2. 当受害者点击邮箱里面的链接，就会被自动重定向到sub1.aaa.com(这是漏洞所在的域名)
3. xss 荷载在重定向的过程中已经被缓存下来了，受害者无论访问sub1.aaa.com任意路径
都能触发，哪怕是访问sub1.aaa.com/lol123 也能触发
4. 现在受害者被重定向到sub1.aaa.com，在这个页面他输入自己的email和reference number，
出于对sub1.aaa.com域名本身的信任，他将这些信息输入进去，却不知道这些信息已经悄悄被攻击
者获取
# 基础的xss上下文
前闭合，后注释（后闭合），中间payload
有的标签是不需要闭合，尽可能先闭合

## 路径xss
在各级路径内容后添加测试用的payload，如xxxx"'<>,
可以在chatgpt写脚本，把路径字典整理好，然后让各级路径进行payload替换，生成新的包含payload的路径字典，设置intreder里的grepmatch为xxxx
不过这种很难升级危害，因为要引入js地址的话，在地址栏不好操作
用eval与hash，
# 过滤和waf
过滤一般指的是对某些关键词进行删除或者替换，比如检到\<script>将其删掉，检查到alert 将其删掉WAF一般值得是拦截，比如检查到某个关键词或者某种规则给你返回403

一些等价替换
![[Pasted image 20240905105412.png]]

在x和28之间加任意多个0，还是x28，加几个不行就多加一点
![[Pasted image 20240905110302.png]]
## cf绕过
基于<svg/onload=alert()>
可以变成<svg/onxyz=1/onload=alert()>因为匹配第一个
最新版可能已经修复

# 会话机制
一个个删cookie里的东西，然后看看还在不在会话状态，找到会话的cookie
# csp
什么情况下可以绕过，什么情况下直接跑路
# 注意
一般来说浏览器会对url的特殊字符进行编码，然后会自动解码，如果我们的攻击只有在浏览器不对url进行编码的情况下才能进行的话，那么这是不行的。 如果一切都没问题，但是还是不弹窗，那么考虑是csp的问题
批量的时候优先处理尖括号不过滤的

tips：大多数人会先挖爬取结果多的，我们反其道而行， 先找到爬取结果少点的，进行更深入收集（目录爆破等）找xss，路径短杆、下划线的也可能出洞

我们一般是对参数值进行反射xss测试，实际上参数本身也有可能反射
① 参数值与参数都有反射点 比如url处为a=xxxx，响应为 "a":"xxxx" 这样的，可以尝试在参数本身
插入payload，得到ayyyy=xxxx，得到 "ayyyy":"xxxx"
② 有时候waf或防护会防御参数值处的反射处 但有可能会在参数本身没有waf或防护，用url为a"-
alert()-"=xxxx 成功触发

xss有时候需要某些其他参数才会触发

还可以看别人的报告，然后进行二开

大多数报告的存储xss是self结合缓存造成的，如果一个页面被缓存了他会直接指出这个页面被缓存了，通过http header或者cookie查找selfxss再结合缓存漏洞升级成存储xss是某些厂商挖掘存储xss的正确打开方式