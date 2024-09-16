# 简介
XML 外部实体注入（也称为 XXE）是一种 Web 安全漏洞，允许攻击者干扰应用程序对 XML 数据的处理。它通常允许攻击者查看应用程序服务器文件系统上的文件，并与应用程序本身可以访问的任何后端或外部系统进行交互。
在某些情况下，攻击者可以通过利用 XXE 漏洞执行服务器端请求伪造 （SSRF） 攻击，升级 XXE 攻击以破坏底层服务器或其他后端基础设施。
# XML
## 什么是xml
XML 是一种用于存储和传输数据的语言。与 HTML 一样，XML 使用标签和数据的树状结构。与 HTML 不同，XML 不使用预定义的标记，因此可以为标记指定描述数据的名称。在 Web 历史的早期，XML 作为一种数据传输格式很流行（“AJAX”中的“X”代表“XML”）。
## 什么是xml实体
XML 实体是一种在 XML 文档中表示数据项的方法，而不是使用数据本身。XML 语言的规范中内置了各种实体。例如，实体 &lt;和 &gt; 表示字符 < 和 >。这些是用于表示 XML 标记的元字符，因此当它们出现在数据中时，通常必须使用其实体来表示。
## 什么是文档类型定义？(DTD)
DTD（Document Type Definition，文档类型定义）是一种定义 XML 文档结构的标准。它规定了 XML 文档的合法元素、属性、子元素以及它们之间的关系。DTD 可以用来验证 XML 文档的格式是否符合规定的结构。
DTD 的作用
- 定义 XML 文档中的元素及其内容类型。
- 限制元素的嵌套关系。
- 定义元素的属性及其取值范围。
- 可以在 XML 文档内部或外部定义。
 **DTD**（Document Type Definition，文档类型定义）是一种定义 XML 文档结构的标准。它规定了 XML 文档的合法元素、属性、子元素以及它们之间的关系。DTD 可以用来验证 XML 文档的格式是否符合规定的结构。

### 一个 DTD 示例

#### 1. 内部 DTD 示例：
内部 DTD 是在 XML 文档中直接定义的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
    <!ELEMENT note (to, from, heading, body)>  <!-- 定义 note 元素，包含 to, from, heading, body 四个子元素 -->
    <!ELEMENT to (#PCDATA)>                    <!-- 定义 to 元素的内容为可解析的字符数据 -->
    <!ELEMENT from (#PCDATA)>                  <!-- 定义 from 元素的内容为可解析的字符数据 -->
    <!ELEMENT heading (#PCDATA)>               <!-- 定义 heading 元素的内容为可解析的字符数据 -->
    <!ELEMENT body (#PCDATA)>                  <!-- 定义 body 元素的内容为可解析的字符数据 -->
]>
<note>
    <to>John</to>
    <from>Jane</from>
    <heading>Reminder</heading>
    <body>Don't forget our meeting!</body>
</note>
```

在这个例子中：
- `<!DOCTYPE note>` 定义了文档的根元素是 `note`。
- `<!ELEMENT note (to, from, heading, body)>` 指定 `note` 元素必须包含 `to`、`from`、`heading` 和 `body` 四个子元素。
- `<!ELEMENT to (#PCDATA)>` 指定 `to` 元素的内容是文本数据（`#PCDATA` 表示可解析字符数据，即普通文本）。

#### 2. 外部 DTD 示例：
外部 DTD 是将 DTD 定义放在一个单独的文件中，并通过 XML 文档中的 `DOCTYPE` 引用该文件。

**DTD 文件 (note.dtd)**:
```xml
<!ELEMENT note (to, from, heading, body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
```

**XML 文件 (note.xml)**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
    <to>John</to>
    <from>Jane</from>
    <heading>Reminder</heading>
    <body>Don't forget our meeting!</body>
</note>
```

在这个例子中：
- `<!DOCTYPE note SYSTEM "note.dtd">` 指定了外部 DTD 文件的路径 `note.dtd`，XML 解析器会根据该文件来验证文档结构。
  
### DTD 中常见的声明
1. **元素声明**：
   - `<!ELEMENT 元素名 (子元素或内容)>`：定义元素的内容类型。内容可以是文本（`#PCDATA`）或其他子元素。

   示例：
   ```xml
   <!ELEMENT note (to, from, heading, body)>
   <!ELEMENT to (#PCDATA)>
   ```

2. **属性声明**：
   - `<!ATTLIST 元素名 属性名 属性类型 默认值>`：定义元素的属性及其默认值。

   示例：
   ```xml
   <!ATTLIST note date CDATA #REQUIRED>  <!-- 定义 note 元素必须有一个 date 属性 -->
   ```

3. **常见的内容模型**：
   - `(子元素)`：必须按顺序包含的子元素。
   - `(#PCDATA)`：表示文本数据。
   - `EMPTY`：该元素不能包含任何内容。
   - `ANY`：该元素可以包含任何内容。
   - `|`：表示选择其中一个。
   - `+`：表示至少出现一次。
   - `*`：表示可以出现零次或多次。

### DTD 的优缺点：
- **优点**：
  - 可以为 XML 文档提供结构验证，确保文档的合法性。
  - 简单易用，适用于小型 XML 文档的结构定义。

- **缺点**：
  - DTD 的语法比较简单，无法表示复杂的数据结构。
  - 不支持命名空间，这在复杂的 XML 应用中是个问题。
  - 与 XML 的语法不一致，导致学习曲线稍有差异。

总之，DTD 是早期 XML 数据验证和描述文档结构的一种机制，尽管它已经被更强大的 XML Schema（XSD）所取代，但在一些简单的 XML 应用场景中仍然有使用。
# 用xxe读取文件
例如，假设购物应用程序通过向服务器提交以下 XML 来检查产品的库存水平：

```
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```
该应用程序对 XXE 攻击没有特别的防御，因此您可以利用 XXE 漏洞通过提交以下 XXE 负载来获取 /etc/passwd 文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```
此 XXE 有效负载定义一个外部实体 &xxe;，其值是 /etc/passwd 文件的内容，并使用 productId 值中的实体。这会导致应用程序的响应包含文件的内容：
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
对于现实世界的 XXE 漏洞，提交的 XML 中通常会有大量数据值，其中任何一个都可能在应用程序的响应中使用。要系统地测试 XXE 漏洞，您通常需要单独测试 XML 中的每个数据节点，方法是使用您定义的实体并查看它是否出现在响应中。
# 利用XXE进行SSRF攻击
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
要利用 XXE 漏洞执行 SSRF 攻击，您需要使用要定位的 URL 定义外部 XML 实体，并在数据值中使用定义的实体。如果您可以在应用程序响应中返回的数据值中使用定义的实体，那么您将能够在应用程序的响应中查看来自 URL 的响应，从而获得与后端系统的双向交互。否则，您将只能执行盲目的 SSRF 攻击（这仍然可能产生严重后果）。比如说，xxe打ssrf获取云服务器的元数据
# 盲xxe
在许多情况下，XXE 注入漏洞的攻击面是显而易见的，因为应用程序的正常 HTTP 流量包括包含 XML 格式数据的请求。在其他情况下，攻击面不太明显。但是，如果您查看正确的地方，您会在不包含任何 XML 的请求中发现 XXE 攻击面。
某些应用程序接收客户端提交的数据，将其嵌入服务器端的 XML 文档中，然后解析文档。例如，将客户端提交的数据放入后端 SOAP 请求中，然后由后端 SOAP 服务处理该请求。
在这种情况下，您无法执行经典的 XXE 攻击，因为您无法控制整个 XML 文档，因此无法定义或修改 DOCTYPE 元素。但是，您或许可以改用 XInclude。XInclude 是 XML 规范的一部分，它允许从子文档构建 XML 文档。您可以在 XML 文档的任何数据值内放置 XInclude 攻击，因此，在您只控制放置在服务器端 XML 文档中的单个数据项的情况下，可以执行该攻击。
要执行 XInclude 攻击，您需要引用 XInclude 命名空间并提供要包含的文件的路径。例如：

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```
- xmlns:xi="http://www.w3.org/2001/XInclude"：
- 这是定义 XInclude 的命名空间，用来声明 XML 文档可以包含其他文档或文本文件。
- <xi:include>：
- 这是一个 XInclude 元素，用于将外部资源包含到 XML 文档中。这里它尝试包含一个文件。
- parse="text"：
- 该属性表示要以纯文本的形式解析外部文件（而不是解析为 XML）。在这个例子中，它尝试以纯文本的方式读取并包含 /etc/passwd 文件。
- href="file:///etc/passwd"：
- 这个 href 属性指向本地文件系统中的 /etc/passwd 文件。/etc/passwd 是 Linux/Unix 系统中的文件，包含用户的基本信息。
# 文件上传打xxe
某些应用程序允许用户上传文件，然后在服务器端进行处理。一些常见的文件格式使用 XML 或包含 XML 子组件。基于 XML 的格式示例包括 DOCX 等办公文档格式和 SVG 等图像格式。
例如，应用程序可能允许用户上传图像，并在上传图像后在服务器上处理或验证这些图像。即使应用程序希望接收 PNG 或 JPEG 等格式，正在使用的图像处理库也可能支持 SVG 图像。由于 SVG 格式使用 XML，攻击者可以提交恶意 SVG 图像，从而到达 XXE 漏洞的隐藏攻击面。
# 修改content type打xxe
大多数 POST 请求都使用由 HTML 表单生成的默认内容类型，例如 application/x-www-form-urlencoded .某些 Web 站点希望接收此格式的请求，但可以容忍其他内容类型，包括 XML。
例如，如果普通请求包含以下内容：

```
POST /action HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=bar
```
然后，您或许可以提交以下请求，结果相同：

```
POST /action HTTP/1.0
Content-Type: text/xml
Content-Length: 52

<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```
如果应用程序容忍消息正文中包含 XML 的请求，并将正文内容解析为 XML，则只需重新格式化请求以使用 XML 格式即可到达隐藏的 XXE 攻击面。
# xxe防御
几乎所有 XXE 漏洞的出现都是因为应用程序的 XML 解析库支持应用程序不需要或不打算使用的潜在危险 XML 功能。防止 XXE 攻击的最简单、最有效的方法是禁用这些功能。
通常，禁用外部实体的解析并禁用对 XInclude 的支持就足够了。这通常可以通过配置选项或以编程方式覆盖默认行为来完成。有关如何禁用不必要功能的详细信息，请参阅 XML 解析库或 API 的文档。