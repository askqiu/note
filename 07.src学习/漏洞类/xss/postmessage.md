
主题是postMessage未验证消息来源origin，导致恶意代码注入的dom-xss
![[Pasted image 20241023222842.png]]
参考靶场1
https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages
```

<iframe src="https://YOUR-LAB-ID.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
```
当iframe加载时， postMessage()方法会向主页发送一条 Web 消息。旨在提供广告服务的事件侦听器获取网络消息的内容并将其插入 ID 为ads的div中。然而，在这种情况下，它插入了我们的img标签，其中包含无效的src属性。这会抛出一个错误，导致onerror事件处理程序执行我们的有效负载。
![[Pasted image 20241023231109.png]]