# 演示
waybackurls democenter.dell.com > urls.txt
筛选掉超出范围的子域名，只留下demo子域名
cat urls.txt | grep https://democenter.dell.com |tee real_urls.txt
获取路径字典，并去除静态url
cat urls.txt |unfurl paths |sed 's/^.//' |sort -u | egrep -iv "\.(jpg|swf|mp3|mp4|m3u8|ts|jpeg|gif|css|tif|tiff|png|ttf|woff|woff2|ico|pdf|svg|txt|js)" | tee paths.txt
得到很多路径字典，去除相似的，有规律的
![[Pasted image 20240918134343.png]]
这种的，写个py脚本去除掉
然后获取参数名字典
cat urls.txt | unfurl keys |sort -u| tee params.txt
![[Pasted image 20240918134924.png]]
这个工具可以获取参数值？
参数置空可能会对测xss有影响
有脚本获取问号后面的部分？，提取参数？
生成payload
![[Pasted image 20240918164138.png]]
前面箭头是参数字典，后面箭头加生成的payload，两个地方都记得在爆破模块去掉url编码，速率调低一点，参考脚本中的xss参数，在grep match中加入这些
![[Pasted image 20240918165252.png]]
查看有没有反射带你，优先看尖括号
![[Pasted image 20240918165417.png]]
然后在response找z那玩意，然后看反射的序号（前面脚本有生成的），在回请求里面找对应序号的参数。