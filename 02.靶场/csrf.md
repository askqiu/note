# 令牌验证取决于请求方法的 CSRF
有的时候，csrftoken只针对post请求，我们把请求变成get，并且把无法确认实际值的参数（csrftoken）删掉，然后构造csrf poc