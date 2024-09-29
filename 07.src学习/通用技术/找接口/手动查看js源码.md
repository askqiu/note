# 简介
js文件存储客户端代码，可以充当网站支柱，随着技术发展，越来越多的网站会在.js文件中列出许多有趣的新消息。例如，在浏览网站源码时候，我们可能会看到对min.js和app.js的引用，也就是reactjs。这些网站严重依赖js和发出ajax请求。这些文件将包含有关次应用程序的几乎所有内容，单他们还将为每个断点使用特定的唯一JavaScript文件。

Javascript 文件可帮助网站执行某些功能，例如监控何时单击某个按钮，或者当用户将鼠标移动到图像上甚至代表用户发出请求（例如检索信息）时。有时开发人员可以混淆他们的 javascript 代码，使其无法被人眼读取，但是大多数类型的混淆都可以逆转，如下所述。

.js 文件通常很容易被忽视，因为它们包含许多看起来“奇怪”的代码，这没有太大意义，但是您只需要知道要搜索哪些关键字来提取有用的信息。随着时间的推移，你阅读javascript的越多，你就会开始将其拼凑起来并理解它是如何工作的。这就是理解代码和学习 javascript 将有助于您完成漏洞赏金之旅的地方。

有关视频
```
https://youtu.be/0jM8dDVifaI
```
# 如何查找.js文件
==手动黑客==：右键viewsource，找.js
某些 JavaScript 文件可能只用于当前访问的特定页面。例如，当你访问 https://www.bugbountyhunter.com/challenge 页面时，可能会发现一个名为 challenge.js 的 JavaScript 文件，它的功能是专门针对该页面的。这些文件可能包含你以前不知道的 API 端点（接口）。
使用burp爬虫

# 我们要找什么？
新的endpoints
新参数
隐藏功能
	（如何测试：
	`查看 JavaScript 文件：通过查看和分析 .js 文件，查找是否有隐藏的 API 调用或功能逻辑。`
	
	手动测试 API 端点：使用工具如 Burp Suite 或 Postman，手动向你发现的 API 端点发送请求，看看是否可以不通过高级用户身份访问相关资源或功能。
	
	修改客户端数据：通过浏览器的开发者工具（F12），尝试修改 JavaScript 变量、隐藏的 UI 元素，或直接调用与功能相关的函数，看看是否可以绕过限制。）
apikey

# 怎么找？（手动看）
通过在js中找上面的东西
关键字：url:    post    api    GET   setRequestHeader    send(     headers    onreadystatechange    var{xyz} =     getParameter()    parameter    .theirdomain.com    apiKey 

这些关键字将帮助我们扩大攻击面，但 postMessage 错误、DOM XSS 和重定向呢？以下是一些需要注意的示例关键字：
postMessage、messageListener、.innerHTML、document.write（、document.cookie、location.href、redirectUrl、window.hash。
您阅读某些 javascript 文件的次数越多，您就越能发现有关目标和 API 终端节点的某些模式（例如终端节点的版本或名称）的信息。

---
如您所见，我们已经设法开始通过读取他们的 .js 文件和查找特定关键字来增加此目标网站上的数据。由此，我们能够发现新的端点（这导致了更多的.js文件！）和更多关于我们目标的知识。需要注意的一点是==不要浪费时间浏览 jquery.js 等内容==。通常，这些文件不包含对您有用的任何内容。选择引用的自定义 .js 文件。

查找和读取.js文件的主要目的是通过发现端点、参数、键、接收器 （dom xss） 以及有关目标的更多总体知识来扩大攻击面。例如，如果您知道他们严重依赖 javascript 文件，那么您可以设置一个脚本来自动检测任何更改并立即在新端点上收到通知！

# 自动读取js
另一种方法是使用 003random 的名为 GetJS 的工具，可以在这里找到：https://github.com/003random/getJS。GetJS 将获取域列表并提取在每个域上找到的任何.js文件。（packerfuzzer也可以）如果您想大规模地对大量 .js 文件进行排序以获取新的 url/endpoints（我们将在下面解释），这将非常有用。如果您正在测试的网站确实包含每个功能/终端节点的特定 .js 文件，则可以开始扫描终端节点以发现新的 JS 文件。

除此之外，Jobert Abma 还发布了一个名为 Relative URL Extractor 的工具，可以在这里找到 - https://github.com/jobertabma/relative-url-extractor。另一种选择是 0xsha 的 GoLinkFinder，可以在这里找到 - https://github.com/0xsha/GoLinkFinder。如果您不习惯阅读 javascript，两者都将有助于从 JS 文件中提取有用的信息。

# 进行反混淆
有的js代码会被混淆，看不懂，可以通过在线工具尝试去混淆
它可以去混淆大多数类型的Javascript并使其可读性：https://lelinhtinh.github.io/de4js/

# 阅读js以获得真正的发现
这是有关真实调查结果的信息（网站已编辑）。在这个特定的网站上，当访问 https://www.example.com/login 时，我在 javascript 中注意到以下内容：
![[Pasted image 20240929090641.png]]
此代码将向 /login.aspx 发送 POST 请求，成功后，它将重定向到 /dashboard。访问 example.com/dashboard 会自动将我重定向到 example.com/login，但它是通过 META REFRESH 进行的，并且响应包含 HTML 和对更多.JS文件的引用。第一个错误！

浏览内容时，我再次注意到使用了更多的 JavaScript 代码，引用了更多的端点，只是这次它们 AppSpot.com。由于它现在正在调用外部域，我只知道不会有会话处理..哦，还有进行调用所需的 API-KEY 的事实也被引用了。让我们来测试一下！
![[Pasted image 20240929090714.png]]
在使用所需的 API-KEY 向 appspot URL 发送请求后，它使用看起来像加密字符串的内容进行响应。有趣的是，这是做什么的？我们越来越近了，但到目前为止没有什么有趣的报告。每当我看到正在使用有趣的加密、ID、参数名称等时，我都会记下来，我们这里有一个很好的指南。

通过阅读我的笔记，我记得我发现这种类型的值在其他地方使用，在这种情况下，一个完全不同的子域上还有另一个端点，它将这个加密的 ID 作为参数值，它将响应有关用户的 PII 信息。成功和 4 位数赏金！

仅仅因为一个页面会重定向，就找出如何以及为什么。是因为您没有经过身份验证吗？它是通过 302 标头加载，还是通过一些 javascript/meta 刷新重定向？如果是这样，页面重定向的代码中是否有任何有趣的内容？
# 例子

假设您正在使用 https://www.example.com，在查看源时发现了一个“app.js”文件。此外，在访问 https://www.example.com/settings 时，您会看到一个 settings.js 文件。![[Pasted image 20240929085014.png]]
然而，在这一点上app.js 是我们的重点。我们在寻找什么？新的终端节点、参数，可能还有 api 密钥。打开它后，我们看到的似乎是 “胡言乱语”![[Pasted image 20240929085042.png]]
第一步：美化。美化会将我们的代码变成可读的代码，你可以遵循和理解。复制代码内容并使用 Javascript 美化器，例如 https://beautifier.io/
![[Pasted image 20240929085055.png]]
现在我们的代码是可读的，让我们开始寻找新的端点、参数，也许还有 api 密钥。在下面的示例（这是一个真实的网站）中，通过搜索关键字路径名，我能够找到网站上使用的所有端点，这些端点扩大了攻击面以查找错误，并开始查找有关其 API 的信息。
![[Pasted image 20240929085120.png]]
![[Pasted image 20240929085125.png]]

