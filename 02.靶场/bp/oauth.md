# Authentication bypass via OAuth implicit flow
要求，登陆到carlos的账号
我们先登陆自己账号wiener，分析历史数据包，发现
```
{"email":"wiener@hotdog.com","username":"wiener","token":"al3jN4i8FXtoiqutHxumyvgPnPCYEw5QbHl45rqk_HT"}
```
尝试直接重放修改email和username，发现还是在自己账号，于是我们选择通过拦截数据包修改，成功。这里有个点是，如果你登陆过wiener，点击退出登陆后，再点击登陆会自动认证登陆wiener，所以进行数据包拦截。