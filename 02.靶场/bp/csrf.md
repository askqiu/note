# 令牌验证取决于请求方法的 CSRF
有的时候，csrftoken只针对post请求，我们把请求变成get，并且把无法确认实际值的参数（csrftoken）删掉，然后构造csrf poc，从而绕过csrf token


# 令牌验证取决于令牌存在的 CSRF
直接删掉csrftoken就可以，但是修改令牌值不行
，也就是不传csrftoken就能绕过

# 令牌未绑定到用户会话的 CSRF
这里主要是使用自己的csrftoken（在还没发给服务端的情况下）来修改别人的请求

# 令牌与非会话 Cookie 绑定的 CSRF（没成功）
cookie中的csrfkey与csrftoken绑定，但是csrf可以不属于会话cookie，于是我可以使用我的csrfkey和我的csrftoken去对别人进行csrf攻击，这里我们发现搜索功能会把用户输入字段插入到cookie里面
Set-Cookie: LastSearchTerm=xxx; Secure; HttpOnly
我们使用crlf攻击，设置受害者的csrfkey（setcookie响应中有两个的话后代替前，没有字段的话会合体）

# 令牌在 cookie 中重复的 CSRF
用上面的方法同时提交自己的csrftoken就行，在cookie中

# 绕过samesite=lax
`GET /my-account/change-email?email=test5%40qq.com&_method=POST HTTP/2`
