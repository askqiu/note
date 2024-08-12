# 可能会导致xss的接收器
- document.write()
- document.writeln()
- document.domain
- element.innerHTML
- element.outerHTML
- element.insertAdjacentHTML
- element.onevent
jquery中
- add()
- after()
- append()
- animate()
- insertAfter()
- insertBefore()
- before()
- html()
- prepend()
- replaceAll()
- replaceWith()
- wrap()
- wrapInner()
- wrapAll()
- has()
- constructor()
- init()
- index()
- jQuery.parseHTML()
- $.parseHTML()
# csp （content security policy）
CSP 是一种浏览器安全机制，旨在缓解 XSS 和其他一些攻击。它的工作原理是限制页面可以加载的资源（例如脚本和图像），并限制页面是否可以被其他页面框住。
**所以说，csp是可以防止或者减小xss产生后的灾害**

## 启用csp
若要启用 CSP，响应需要包含一个名为 Content-Security-Policy 的 HTTP 响应标头，其值包含策略。策略本身由一个或多个指令组成，用分号分隔。

以下指令仅允许从与页面本身相同的源加载脚本：
script-src 'self'
以下指令仅允许从特定域加载脚本：
script-src https://scripts.normal-website.com
以下指令仅允许来自同一源的其他页面构建该页面：
frame-ancestors 'self'
以下指令将完全阻止框架：
frame-ancestors 'none'

使用内容安全策略来防止点击劫持比使用 X-Frame-Options 标头更灵活，因为您可以指定多个域并使用通配符。例如：
frame-ancestors 'self' https://normal-website.com https://*.robust-website.com
CSP 还会验证父帧层次结构中的每个帧，而 X-Frame-Options 仅验证顶级帧。
建议使用 CSP 来防范点击劫持攻击。还可以将其与 X-Frame-Options 标头结合使用，以在不支持 CSP 的旧浏览器（如 Internet Explorer）上提供保护。

CSP 阻止脚本等资源是很常见的。但是，许多 CSP 允许图像请求。这意味着您通常可以使用 img 元素向外部服务器发出请求，例如，为了泄露 CSRF 令牌。
某些浏览器（例如 Chrome）具有内置的悬空标记缓解功能，可阻止包含特定字符的请求，例如原始的、未编码的新行或尖括号。
某些策略的限制性更强，可以阻止所有形式的外部请求。但是，仍然可以通过引发一些用户交互来绕过这些限制。要绕过这种形式的策略，您需要注入一个 HTML 元素，当单击该元素时，该元素将存储被注入元素包含的所有内容并将其发送到外部服务器。

### 外部域
具体含义

    外部脚本：
        网站开发者有时会从外部来源加载脚本（例如，通过CDN加载JavaScript库，如jQuery）。
        这些脚本通常位于外部域名下，而不是网站自己的服务器。

    信任问题：
        如果你从一个外部域名加载脚本，你实际上是在信任这个外部域提供的内容。
        一旦外部域被攻击、遭受恶意劫持、或者外部域的拥有者恶意更改了脚本，攻击者就可能通过你的网站执行恶意代码。
        例如，如果你的网站通过 ajax.googleapis.com 加载jQuery脚本，但 ajax.googleapis.com 的内容被恶意篡改，那么你的网页可能会执行恶意代码。

    不受信任的CDN：
        “不使用按客户URL的CDN”指的是，一些CDN可能允许任何人上传内容并通过它们的网络分发。
        如果攻击者能通过这种CDN上传恶意脚本，并诱导你的网站加载这个脚本，就可能对你的用户造成风险。

## 随机数和哈希
除了将特定域列入白名单外，内容安全策略还提供了另外两种指定受信任资源的方法：随机数和哈希：
CSP 指令可以指定随机数（随机值），并且必须在加载脚本的标记中使用相同的值。如果值不匹配，则脚本将不会执行。为了有效地作为控件，必须在每次页面加载时安全地生成随机数，并且攻击者不能猜到。
CSP 指令可以指定受信任脚本内容的哈希。如果实际脚本的哈希值与指令中指定的值不匹配，则脚本将不会执行。如果脚本的内容发生更改，则您当然需要更新指令中指定的哈希值。
## 绕过csp
您可能会遇到一个网站，该网站将输入反映到实际策略中，最有可能是在 report-uri 指令中。如果站点反映的是您可以控制的参数，则可以注入分号来添加自己的 CSP 指令。通常，此 report-uri 指令是列表中的最后一个指令。这意味着您需要覆盖现有指令才能利用此漏洞并绕过该策略。
通常，无法覆盖现有的 script-src 指令。但是，Chrome 最近引入了 script-src-elem 指令，该指令允许您控制脚本元素，但不能控制事件。至关重要的是，这个新指令允许您覆盖现有的 script-src 指令。
## xss防范
输出编码，在html中可以进行html编码
在JavaScript中可以进行unicode转义
对输入的数据进行验证，判断是不是服务器该业务下需要的字符
