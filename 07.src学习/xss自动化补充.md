paramspider爬取url
-d dell.com
waybackurls 爬取url
去除url特多的路径
waybackurls dell.com>>1.txt
命令去重排序，
用uro（github下载）
cat 1.txt|uro |tee 2.txt
可能会把有用的也去掉

grep -v 'img'这样慢慢筛选
还有信息收集里面目录爆破的一个知识点，去除静态文件的url