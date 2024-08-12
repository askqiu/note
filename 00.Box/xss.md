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

## 启用csp
若要启用 CSP，响应需要包含一个名为 Content-Security-Policy 的 HTTP 响应标头，其值包含策略。策略本身由一个或多个指令组成，用分号分隔。

以下指令仅允许从与页面本身相同的源加载脚本：
script-src 'self'
以下指令仅允许从特定域加载脚本：
script-src https://scripts.normal-website.com