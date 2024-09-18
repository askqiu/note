# 演示
waybackurls democenter.dell.com > urls.txt
筛选掉超出范围的子域名，只留下demo子域名
cat urls.txt | grep https://democenter.dell.com |tee real_urls.txt
获取路径字典，并去除静态url
cat urls.txt |unfurl paths |sed 's/^.//' |sort -u | egrep -iv "\.(jpg|swf|mp3|mp4|m3u8|ts|jpeg|gif|css|tif|tiff|png|ttf|woff|woff2|ico|pdf|svg|txt|js)" | tee paths.txt
得到很多路径字典，去除相似的，有规律的