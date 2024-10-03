# 令牌验证取决于请求方法的 CSRF
有的时候，csrftoken只针对post请求，我们把请求变成get，并且把无法确认实际值的参数（csrftoken）删掉，然后构造csrf poc，从而绕过csrf token


# 令牌验证取决于令牌存在的 CSRF
直接删掉csrftoken就可以，但是修改令牌值不行
，也就是不传csrftoken就能绕过

# 令牌未绑定到用户会话的 CSRF
这里主要是使用自己的csrftoken（在还没发给服务端的情况下）来修改别人的请求