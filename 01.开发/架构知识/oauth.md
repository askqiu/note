- （A）客户端（Client）向资源所有者（Resource Owner）请求资源授权。授权请求可以直接向资源所有者（Resource Owner）发起，不过最好是通过授权服务器（Authorization Server）间接发起。
- （B)  客户端（Client）得到资源所有者（Resoure Owner）的授权，这通常是一个凭据；授权的形式和凭据可以有不同的类型。RFC 6749 定义了四种主要的授权类型（下文进一步介绍）
- （C）客户端（Client）向授权服务器（Authorization Server）出示授权（来自Resource Owenr的）凭据进行身份认证；并申请用于访问资源授权的访问令牌（Access Token）
- （D) 授权服务器（Authorization Server）对客户端（Client）进行身份验证并验证授权授予，如果通过验证，则颁发访问令牌（Access Token）。（
- E）客户端（Client）通过向资源服务器（Resource Server）发起令牌（Access Token）验证，请求被保护的资源。
- （F）资源服务器（Resource Server)验证访问令牌（Access Token）；如果通过认证，则返回请求的资源。

客户端必须得到用户的授权（前面的步骤B），才能获得访问令牌（Access Token）。
oauth2.0有四种授权方式
1. 授权码
2. 隐式授权
3. 密码
4. 客户端

授权码
1. （A）Client先将页面重定向Authorization Server的授权页；重定向是需要携带授权完毕后要重新打开的页面（携带RedirectURI）。
2. （B）Resource Owner在授权也进行授权。
3. （C）授权后，Authorization Server将页面重定向会Client的页面（在A步骤中指定的RedirectURI）。同时会在URI中携带授权码Code。授权码Code会经UserAgent最终传递给Client的后端。
4. （D）Client（后端）利用授权码向Authorization Server请求访问令牌（Access Token），这里需要指定请求访问的访问Scope等信息。
5. （E）Authorization Server 校验授权码通过后，返回访问令牌Access Token和刷新令牌Refresh Token。
```

GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com
```


隐式授权
主要用于纯前端应用，如JavaScript spa
在隐式授权模式中，不是向客户端颁发授权码，而是直接向客户端颁发访问令牌（作为资源所有者授权的结果）。省去了颁发中间凭据（例如授权代码）的过程。
1. （A）用户代理（通常是浏览器）向认证服务器发送授权请求。这通常通过将用户重定向到认证服务器的授权端点来完成，请求中包含了客户端ID、请求的权限范围、重定向URI和状态。
2. （B） 认证服务器对用户进行身份验证，通常是通过要求用户输入用户名和密码。认证服务器向用户显示一个授权页面，让用户决定是否授予客户端请求的权限。
3. （C）如果用户同意授予权限，认证服务器将用户代理重定向回客户端的重定向URI，并在重定向URI的片段部分（fragment）中包含访问令牌和状态。注意，由于这是在用户代理中完成的，所以访问令牌从未通过服务器端的应用代码。

```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```