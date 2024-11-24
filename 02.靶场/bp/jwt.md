# JWT authentication bypass via unverified signature
jwt里修改用户为administrator，然后看页面源代码，发送get请求到删除的接口

# WT authentication bypass via flawed signature verification
点击攻击，选择没有签名，这里主要是不提供签名应该就不验证
后面点要留下
![[Pasted image 20241117223128.png]]
![[Pasted image 20241113190428.png]]


# ab: JWT authentication bypass via weak signing key
--使用弱密钥签名
使用工具爆破密钥
hashcat -a 0 -m 16500 ./jwt.txt ./jwt.secrets.list
把爆破出来的密钥放到图中，进行重新签名
https://jwt.io/#debugger-io
![[Pasted image 20241124145550.png]]