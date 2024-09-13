https://portswigger.net/web-security/cross-site-scripting#how-does-xss-work
1.闭合h1标签
```html
</h1><script>alert(1)</script>\\
```
3.dom型xss，在检查里查看，查看页面源代码看不见的
```html
xxx" onload="alert(1)  \\构造前后闭合
```


```
情景：<a id="backLink" href="/xxx">Back</a>
```
令href=javascript:alert(document.cookie)

## Lab: DOM XSS in jQuery selector sink using a hashchange event
`$(window).on('hashchange', function() {`
    `var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');`
    `if (post) post.get(0).scrollIntoView();`
`});`
逐行解释

    $(window).on('hashchange', function() {...}):
        这行代码使用 jQuery 给 window 对象添加了一个 hashchange 事件监听器。当 URL 中的哈希值（即 # 后面的部分）发生变化时，这个监听器会被触发，执行其中的回调函数。

    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');:
        这一行代码做了两件事：
            获取当前 URL 的哈希值：window.location.hash.slice(1) 获取 URL 中 # 后面的部分，并通过 slice(1) 去掉 #，得到纯文本部分。
            解码哈希值：decodeURIComponent 解码哈希值，因为 URL 中的特殊字符会被编码为 %XX 形式。
            查找匹配的元素：$('section.blog-list h2:contains(...)') 使用 jQuery 选择器在页面上查找位于 section.blog-list 内的所有 <h2> 元素，并筛选出包含解码后的哈希值作为文本内容的 <h2> 元素。如果找到了这样的元素，就将它存储在变量 post 中。

    if (post) post.get(0).scrollIntoView();:
        如果找到了符合条件的 post（即 post 不是空的），则将这个 h2 元素滚动到视图中（scrollIntoView()）。这意味着页面将自动滚动到与哈希值匹配的 <h2> 元素所在的位置。

这段代码通常用于博客或文章列表页面。其作用是当用户点击链接或手动更改浏览器地址栏中的哈希部分时，页面会自动滚动到与哈希值对应的文章标题（\<h2> 标签）。例如，如果 URL 中的哈希值是 My-Article-Title，代码会在 section.blog-list 内查找包含 My Article Title 的 \<h2> 标题，然后自动滚动到这个标题的位置。
可能存在的风险

使用 :contains() 选择器并直接插入用户控制的内容（例如 URL 的哈希值）可能存在一定的安全风险，特别是 XSS（跨站脚本攻击）的风险。因此，在实际开发中，使用这类选择器时应注意对输入内容进行充分的验证和过滤，以防止恶意代码注入。

在这个lab的关键点是，jquery的工作原理：无论是否有任何匹配项，他都会返回jquery的搜索对象
payload
```
<iframe src="https://exploit-0ac6000704c9ae5b80ac431f01ad008b.exploit-server.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

## Lab: Reflected XSS into attribute with angle brackets HTML-encoded
```
" onmouseover="alert(1)
```
该属性会在用户将鼠标移到输入框时触发 alert(1)，从而弹出一个 JavaScript 警告框。
## Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded
在这里我犯错了，新手常见的靶场错误，以为反射xss只会存在评论里，于是我找半天没找到答案里的href，后来发现反射点在名字的超链接哈哈哈
href="javascript:alert(1)"
## Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded
payload是
```html
'-alert(1)-'
```
反射点的代码
```html
var searchTerms = 'xxx'-alert(1)-'';
document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
```
'xxx' - alert(1) - '' 这段代码会被 JavaScript 引擎解析并执行。
在 JavaScript 中，使用 - 运算符时，会尝试将两边的操作数转换为数字进行数学运算。
由于 alert(1) 是一个有效的 JavaScript 函数，它会弹出一个带有数字 1 的警告框。
然后 alert(1) 的返回值是 undefined。
'xxx' - undefined - ''：尝试将字符串和 undefined 进行减法运算，结果会是 NaN (Not a Number)，但是这并不影响弹窗的触发。然后nan-''，这里是什么都没有，转换为数字就是0，所以结果还是nan
总结：==弹窗的原因并不是因为 encodeURIComponent() 或 document.write()，而是因为 alert(1) 在表达式计算过程中被直接调用并执行。因此，尽管代码的最终输出不会插入恶意代码，但在计算过程中弹窗已经触发。==
# Lab: DOM XSS in document.write sink using source location.search inside a select element
domxss，我们先打开页面代码，找找script代码，看看有没有我们能控制dom的地方。在这个lab里我们发现storeid可以操纵页面的dom，于是用他来构造闭合
需要注意的是：这里要闭合option之后再闭合select标签
?productId=1&storeId=ppp</option></select><img%20src=1%20onerror=alert(1)>

# Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
首先通过f12的网络发现有个叫angular的js 文件，判断他用了该框架
`{{$on.constructor('alert(1)')()}}` 这个 XSS payload 生效的原因与 JavaScript 处理函数和构造函数的方式有关。具体来说，它利用了 AngularJS 或其他双大括号模板语法中的注入点，并结合了 JavaScript 的 `Function` 构造函数来执行任意代码。

### 解释过程：

1. **双大括号表达式 (`{{}}`)**:
   - 在许多前端框架（如 AngularJS）中，双大括号语法被用于模板表达式。这种表达式可以直接插入并执行用户输入的内容。
   - 如果没有正确的安全过滤，这种表达式可以成为 XSS 攻击的入口。

2. **`$on` 对象**:
   - 在 AngularJS 中，`$on` 是一个标准的事件注册函数。虽然 `{{$on}}` 的具体内容可以视上下文而定，但它通常是存在的。

3. **`constructor`**:
   - `constructor` 是 JavaScript 中每个对象的属性，指向该对象的构造函数。
   - 在这个例子中，`$on.constructor` 实际上是对 `Function` 构造函数的引用。

4. **`Function` 构造函数**:
   - `Function` 构造函数允许动态创建新的函数。`Function('alert(1)')` 相当于定义了一个包含 `alert(1)` 的新函数。Function 是 JavaScript 的内建构造函数，用于创建函数对象，意思是他就是用来创建对象的一个函数。
   - 在JavaScript中，Function('alert(1)') 能够触发弹窗是因为 Function 构造函数允许你动态创建一个新的函数，并在这个过程中执行其中的代码。

5. **立即调用 `()`**:
   - 在 `{{$on.constructor('alert(1)')()}}` 中，创建了一个新的函数，并立即调用它。
   - 因为这个函数中包含了 `alert(1)`，所以会弹出警告框。

### 攻击流程:
- 用户输入 `{{$on.constructor('alert(1)')()}}`，前端框架将其解析为 JavaScript 表达式并执行。
- `constructor('alert(1)')` 创建一个新函数，包含 `alert(1)`。
- 最后，通过 `()` 立即调用这个函数，从而触发 `alert(1)` 弹窗。

### 生效的前提条件：
- **模板注入**：应用程序必须允许未经过滤的用户输入进入模板并被解析。
- **对象存在**：`$on` 对象以及其 `constructor` 属性必须存在且可访问。

这种方式的攻击主要是利用了前端框架和 JavaScript 本身的灵活性，如果应用没有进行严格的输入验证和模板转义，类似的 payload 就能生效并执行恶意代码。
# Lab: Reflected DOM XSS 实验室：反射式 DOM XSS
打开站点地图我们发现有一个返回是
```json
{"results":[],"searchTerm":"xxxx"}
```
然而当我们想逃离时候，输入xxxx",原本xxxx的地方变成了xxxx\"，也就是说这个双引号失去了原有的特殊意义，导致我们逃离失败，所以我们把\也给转义了就行，
xxxx\"alert()，但是我发现这样的话就会导致{(这里省略前面)"searchTerm":"xxxx\\"-alert"}我们多了一个单引号，导致无法构成有效的js代码，所以最终payload
```js
xxxx\"-alert()}//
```
把后面的注释掉，自己重构js
为什么说eval容易受到攻击，那是因为他不需要特定的json，只要复合JavaScript的执行逻辑就能执行，而json数据的话，在攻击过程中很容易就破坏了json 的数据结构导致我们的攻击失效。
![[图片.png]]
这段 JSON 数据看起来不符合标准的 JSON 格式，但之所以它能够触发 XSS，是因为它并没有完全遵循 JSON 格式在特定场景下被 JavaScript 解析。（其实这个lab会触发xss完全是因为他使用eval函数进行数据输出）

### 关键点：
1. **双重解析**：尽管在纯粹的 JSON 格式中，这样的字符串并不合法，但在某些 Web 应用中，开发者可能会直接将服务器返回的数据插入到 HTML 页面中，或者通过不安全的方式将数据插入到 DOM 中。例如，使用 `document.write()`、`innerHTML` 等操作可能会将该数据直接执行，而不是按照 JSON 来处理。

2. **插入到 JavaScript 中**：假设服务器返回的数据被插入到 JavaScript 代码中，而这个数据被未经处理地执行，那么这些包含恶意 JavaScript 代码的字符串就有可能会被执行。例如：
   ```javascript
   var searchTerm = "z3r0sh3ll\\\"-alert()"; // 直接被插入到 JS 代码中
   ```
   这里，如果开发者没有正确地处理输入数据，攻击者就可以利用这个途径来执行 JavaScript 代码。

### 触发 XSS 的原因：
- **不安全的字符串插入**：当字符串没有被正确转义，并被插入到 HTML 或 JavaScript 代码中时，攻击者可以通过构造特殊的输入，逃逸出字符串范围并注入自己的代码。
- **双重反斜杠问题**：在你的截图中，反斜杠在 JSON 中看似用于转义，但实际在浏览器执行时可能会被忽略或视作普通字符，这取决于执行环境。这样会让构造的 payload 得以运行。

### 防范措施：
- **严格 JSON 解析**：确保数据始终按照 JSON 格式处理，避免直接将数据插入到 HTML 或 JavaScript 中。
- **转义与过滤**：在将用户输入的数据插入到 DOM 或 JavaScript 代码中之前，一定要进行适当的转义与过滤。

综上，尽管这段数据不符合标准 JSON 格式，但由于其在特定场景下被不安全地解析和执行，最终导致了 XSS 的发生。
## 有关json和普通数据

1. **`eval()` 不限制输入**：`eval()` 可以执行任何符合 JavaScript 语法的代码，而不局限于特定的数据格式。这意味着如果 `eval()` 处理用户输入，攻击者可以构造任意的 JavaScript 代码，并通过 `eval()` 执行。这种灵活性带来了安全风险，因为恶意输入可能包含破坏性或有害的代码。

2. **JSON 解析更严格**：相比之下，JSON 是一种严格的数据格式，只允许数据（如对象、数组、字符串、数字、布尔值等），而不能包含函数或其他可执行代码。在解析 JSON 时，使用 `JSON.parse()` 只会解析合法的 JSON 数据，而不会执行任何代码。因此，攻击者很难通过破坏 JSON 数据结构来执行任意代码。如果 JSON 数据被破坏，解析过程会失败，导致攻击无效。

3. **攻击容易成功 vs. 攻击容易失败**：由于 `eval()` 可以直接执行任何 JavaScript 代码，所以即使输入稍有改变，也可能导致恶意代码执行成功。相反，JSON 格式的严格性使得攻击者很难通过构造合法的 JSON 数据来注入和执行恶意代码。稍有不当，攻击就会失败。

因此，`eval()` 容易受到攻击的原因，它不需要特定的 JSON 格式，只要输入符合 JavaScript 的执行逻辑即可。而 JSON 在攻击过程中更容易因为结构破坏而导致攻击失败，这就是为什么在处理不受信任的数据时，不建议使用 `eval()` 的原因。

**JSON** 常用于在客户端和服务器之间传输数据。通过 JSON 传输的数据通常包括：

1. **API 请求和响应**：前端与后端通信时，使用 JSON 格式来发送和接收数据。例如，前端发起一个包含用户信息的 API 请求，服务器返回一个 JSON 格式的响应，其中包含查询结果、状态信息等。

2. **表单数据**：前端表单提交时，可以将表单数据转换为 JSON 格式并通过 AJAX 或 Fetch API 发送给服务器。服务器端会解析 JSON 数据并处理相应的业务逻辑。

3. **配置数据**：前端应用在初始化时，可能需要从服务器获取一些配置数据，例如用户设置、界面布局等，这些数据通常以 JSON 格式传输。

4. **动态内容加载**：当页面需要加载动态内容时，例如评论、文章列表、商品信息等，服务器通常会返回 JSON 格式的数据，然后前端解析并渲染到页面上。

5. **用户会话和状态信息**：前端应用通过 JSON 格式存储和传输用户的会话信息或状态信息，例如用户是否登录、当前所处的界面等。

在这些场景中，JSON 是一种通用的、轻量级的数据格式，用于在网络上传输结构化数据。相比于 `eval()`，使用 `JSON.parse()` 来解析这些数据更安全，因为 JSON 格式严格，不允许包含可执行的 JavaScript 代码。

### 例如：
```json
{
  "username": "john_doe",
  "email": "john.doe@example.com",
  "loggedIn": true,
  "preferences": {
    "theme": "dark",
    "notifications": false
  }
}
```

这种数据在前后端之间传输时，前端可以使用 `JSON.parse()` 将字符串解析为 JavaScript 对象，而不会执行其中任何代码。这就是为什么 JSON 在传输数据时被广泛使用，并且比 `eval()` 更安全。
# Lab: Stored DOM XSS
这里存在漏洞的原因是这里的replace只是将字符替换成html编码，没有使用正则表达式替换内容里的所有有害符号（唯一的办法），所以
```
<><img src=2 onerror=alert(1)>
```
c此处的防御方法应该是，把传入的数据的危险符号用html存储在数据库中，而不是从数据库拿出来之后再进行编码输出 

# 允许使用一些 SVG 标记的反射型 XSS
<svg onbegin=alert(1)>无法触发
这样可以触发<svg><animatetransform onbegin=alert(1)>

# 实验：规范链接标签中的反射型 XSS
url反射到页面源码的head里，在head里不能插入<img>之类的标签
然后我们输入一个问号?,使得页面不会跳转 
payload:https://0a8600b903a47c9e80ca5841004c00e2.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
在这个例子我发现，属性值'之后不用空格
![[1726226526783.png]]

# 实验：将 XSS 反射为 JavaScript 字符串，并转义单引号和反斜杠
思维局限了，只想着逃出var的参数
先逃出'，虽然有\转义但是可以如图直接用<script\>隔开
![[Pasted image 20240913193458.png]]
![[Pasted image 20240913193741.png]]