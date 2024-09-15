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