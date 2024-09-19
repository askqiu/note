```
pip3 install arjun
```
针对单个 URL 运行 Arjun。

arjun -u https://api.example.com/endpoint

默认情况下，Arjun 会查找 GET 方法参数。所有可用的方法为：GET/POST/JSON/XML

arjun -u https://api.example.com/endpoint -m POST

Arjun 支持从 BurpSuite、简单文本文件和原始请求文件导入目标。Arjun 可以自动识别输入文件类型，因此您只需指定路径即可。

arjun -i targets.txt

您可以使用相应的选项将结果导出到 BurpSuite 或 txt/JSON 文件。

arjun -u https://api.example.com/endpoint -oJ result.json
-oJ result.json
-oT result.txt
-oB 127.0.0.1:8080

默认情况下，Arjun 可以在使用 JSON 或 XML 方法参数时检测指定位置的参数。所有可用的方法为：GET/POST/JSON/XML

arjun -u https://api.example.com/endpoint -m JSON --include='{"root":{"a":"b",$arjun$}}'

Arjun 默认使用 2 个线程，但您可以根据网络连接和目标限额调整其性能。

arjun -u https://api.example.com/endpoint -t 10

默认情况下，Arjun 在请求中包含 500 个参数，这些参数有时会超过某些服务器的最大 URL 长度限制。您可以使用 -c 选项处理此类情况，方法是指定要一次发送的参数数。

arjun -u https://api.example.com/endpoint -c 250

Disable redirects 禁用重定向
Option: --disable-redirects