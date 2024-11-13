# JWT authentication bypass via unverified signature
jwt里修改用户为administrator，然后看页面源代码，发送get请求到删除的接口

# WT authentication bypass via flawed signature verification
点击攻击，选择没有签名，这里主要是不提供签名应该就不验证
![[Pasted image 20241113190428.png]]
