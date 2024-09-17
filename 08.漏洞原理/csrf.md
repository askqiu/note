# 漏洞简介
实际上就是操纵用户发送他们不打算发出的http请求，
用户已经进行了身份验证，发送请求会自动带上他们的cookie等会话验证信息
# csrftoken
通常通过表单把csrftoken嵌入到请求里，由于受到同源策略限制，攻击者构造的网站无法获取表单里的csrftoken并且把他加到请求头里
攻击者的攻击方式：构造一个恶意网站，然后可以在里面插入恶意的js代码，比如说
`<img src="https://www.siteA.com/transfer?amount=1000&to_account=attacker_account" />`
受害者加载这张不存在的图片的时候，会请求这个url，然后触发转账的功能
因为这个恶意网站无法获取原本网站的csrftoken，所以不会在请求里加上csrftoken这个头，请求无效
csrf是js加载到dom上的，所以不会立马加载出来，如果过早加载xss账号接管恶意js，可能会获取不到csrftoken，
==但是如果是xss+csrf的话，就可以直接在原本网站触发恶意url功能，带上csrftoken==

## 防御：
- 双重提交Cookie（Double Submit Cookie）：除了通过表单提交csrftoken，还可以在Cookie中存储一个相同的令牌。服务器会检查请求中的令牌和Cookie中的令牌是否匹配，以进一步增强安全性。
- SameSite Cookie：通过设置Cookie的SameSite属性为Strict或Lax，可以防止跨站点的请求自动带上会话Cookie，从而防止CSRF攻击。
- 