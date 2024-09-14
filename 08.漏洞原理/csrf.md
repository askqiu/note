# csrftoken
通常通过表单把csrftoken嵌入到请求里，由于受到同源策略限制，攻击者构造的网站无法获取表单里的csrftoken并且把他加到请求头里
攻击者的攻击方式：构造一个恶意网站，然后可以在里面插入恶意的js代码，比如说
`<img src="https://www.siteA.com/transfer?amount=1000&to_account=attacker_account" />`
受害者加载这张不存在的图片的时候，会请求这个url，然后触发转账的功能
因为这个恶意网站无法获取原本网站的csrftoken，所以不会在请求里加上csrftoken这个头，请求无效
==但是如果是xss+csrf的话，就可以直接在原本网站触发恶意url功能，带上csrftoken==

