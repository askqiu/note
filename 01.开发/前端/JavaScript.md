# 简介
js是一种 脚本语言，需要配合其他语言，不能独立运行
所有浏览器都有内置js引擎
js是基于bs的
弱类型语言（直接用var定义变量就行，自动识别变量类型）
# 外部文件引入
我们有时候为了避免重复写代码，可能会直接引入js文件到当前的网页
.js文件的内容不用\<script>标签框着，直接写各种函数就行
```javascript
<script type="text/javascript" src="./jisuan.js"></script>
# 引入当前目录下的jisuan.js文件
然后直接使用
<script>alert("结果是："+jisuan(num1,num2,oper))</script>
```
jisuan.js的内容为
```javascript
function jisuan(num1, num2, oper) {
    num1 = parseFloat(num1);
    num2 = parseFloat(num2);

    var res = 0;

    switch (oper) {
        case "+":
            res = num1 + num2;
            break;
        case "-":
            res = num1 - num2;
            break;
        case "*":
            res = num1 * num2;
            break;
        case "/":
            res = num1 / num2;
            break;
        default:
            alert("不知道什么符号，重新输入");
            break;
    }
    return res;
}

```
# 函数
递归
```javascript
function abc(num1){
	if(num1>3){
	abc(--num1);
	}
	document.writeln(num1);
}//所以、abc(5)的结果是3
```
# 数组
数组定义var arr1=[100,33,44,55];
arr1.length返回数组的大小4
# 面向对象
其实和java差不多，包括this用法之类
```javascript
    <script type="text/javascript">
        function Person() {
            this.name = "我认为名字";
            this.age = 1;

            this.show = function() {
                document.write(this.name + "---" + this.age);
            };
        }

        var p1 = new Person();
        p1.name = "haha";
        p1.age = 100;
        p1.show();
    </script>

```
# uri和eval
大多数情况下uri和url差不多
eval()  会把字符串当作js代码来执行
```javascript
var str="alert(man)";
eval(str);//这串代码会弹窗
```
# 事件
事件源----事件对象----事件处理程序
什么是事件，点击等操作
事件对象呢？点击了哪里
```html
<body>
	<script>
		function test(event){
			alert("man")
			}
	</script>
	<input type="button" value="clickme" onclick="test(event)"/>
</body>
```
上述代码中，按钮被点击这个事件发生之后就会通过onclick调用test
		事件都是用event进行接收的    ps:挖洞xss思考
# dom
是js的精髓 document object model文档对象模型
html包围起来的就是dom，其外是bom，浏览器对象模型
**DOM（Document Object Model，文档对象模型）是一种用于访问和操作HTML和XML文档内容的API。通过DOM，开发者可以动态地创建、修改、删除页面内容和结构。**
```html
<html>
<head>
    <title>DOM 示例</title>
</head>
<body>
    <h1 id="title">原始标题</h1>
    <button id="changeTitleButton">更改标题</button>

    <script>
        // 获取DOM元素
        var titleElement = document.getElementById("title");
        var buttonElement = document.getElementById("changeTitleButton");

        // 定义按钮点击事件的处理函数
        buttonElement.onclick = function() {
            // 修改标题文本
            titleElement.textContent = "新标题";
        };
    </script>
</body>
</html>

```
观察如上代码，就是通过获取不同的节点去动态修改页面内容结构
### DOM概念

- **节点（Node）**：HTML文档的每个部分都是一个节点，比如元素节点、文本节点、属性节点等。
- **树结构（Tree Structure）**：DOM将文档表示为树结构，节点之间的关系像一个树形图。
- **访问和操作**：通过DOM API，可以访问和操作节点，包括创建、删除、修改节点。

这个示例展示了如何通过JavaScript访问和修改HTML文档的内容，这是DOM操作的基本应用。如果你有更多问题或需要进一步的示例，请告诉我！
# open
JavaScript 的 `window.open()` 函数用于打开一个新的浏览器窗口或标签页，并可以指定 URL、窗口名称和一些特性。
```
window.open(url, windowName, [windowFeatures]);
```
还有很多别的用法，挂广告等
```html
<!DOCTYPE html>
<html>
<head>
    <title>Window Open 示例</title>
</head>
<body>
    <button id="openWindowButton">打开新窗口</button>

    <script>
        document.getElementById("openWindowButton").onclick = function() {
            window.open("https://www.example.com", "exampleWindow", "width=800,height=600,resizable=yes");
        };
    </script>
</body>
</html>

```
点击按钮就会打开一个指定url的窗口
# history和location
history可以用于回退等操作
window.location.reload();可以刷新页面
window.location.href="url";可以跳转网址
这些都可以和别的操作一起结合使用，结合button等去触发
# event对象
动态绑定
```html
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript 示例</title>
</head>
<body>
    <script type="text/javascript">
        // 定义一个简化的选择器函数，用于通过ID获取元素
        function $(id) {
            return document.getElementById(id);
        }

        // 定义test1函数，当按钮1被点击时触发
        function test1() {
            // 弹出一个警告对话框，显示"test1"
            alert("test1");
            // 将按钮2的点击事件绑定到test2函数
            $("btn2").onclick = test2;
            // 或者使用下面的代码来绑定点击事件
            // document.getElementById("btn2").onclick = test2;
        }

        // 定义test2函数，当按钮2被点击时触发
        function test2() {
            // 弹出一个警告对话框，显示"test2"
            alert("test2");
        }
    </script>

    <!-- 创建一个按钮，当点击时调用test1函数 -->
    <input type="button" id="btn1" onclick="test1()" value="测试1" />
    <!-- 创建另一个按钮，初始时没有绑定点击事件 -->
    <input type="button" id="btn2" value="测试2" />
</body>
</html>

```
# document
属性
- **`document.title`**: 获取或设置文档的标题
- **`document.URL`**: 返回文档的完整 URL。
- **`document.body`**: 访问文档的 `<body>` 元素。
- **`document.documentElement`**: 访问文档的根元素（即 `<html>` 元素）。
- **`document.cookie`**: 读写页面的 cookie。
方法
1. **`document.getElementById(id)`**: 根据元素的 ID 获取元素。
2. **`document.getElementsByClassName(className)`**: 根据元素的类名获取元素列表。
3. **`document.getElementsByTagName(tagName)`**: 根据标签名获取元素列表。
4. **`document.querySelector(selector)`**: 根据 CSS 选择器获取第一个匹配的元素
5. **`document.querySelectorAll(selector)`**: 根据 CSS 选择器获取所有匹配的元素。
6. **`document.createElement(tagName)`**: 创建一个新的元素。
# 补充
const params = new URLSearchParams(window.location.search);这一句具体是做什么的
window.location.search:

    这个部分获取的是URL中问号 ? 后面的查询字符串部分，也就是URL中的参数部分。
    new URLSearchParams():

    这是一个构造函数，用来创建一个URLSearchParams对象。这个对象可以让你很方便地操作查询字符串中的参数，比如获取参数值、设置新参数、删除参数等等。

params:

    这就是你创建的URLSearchParams对象。通过这个对象，你可以使用各种方法来处理查询参数。
    这段代码 `const params = new URLSearchParams(window.location.search);` 在JavaScript中用于提取和操作当前网页URL中的查询参数。

以下是每个部分的具体作用：

1. **`window.location.search`**:
   - 这个部分获取的是URL中问号 `?` 后面的查询字符串部分，也就是URL中的参数部分。

2. **`new URLSearchParams()`**:
   - 这是一个构造函数，用来创建一个`URLSearchParams`对象。这个对象可以让你很方便地操作查询字符串中的参数，比如获取参数值、设置新参数、删除参数等等。

3. **`params`**:
   - 这就是你创建的`URLSearchParams`对象。通过这个对象，你可以使用各种方法来处理查询参数。

例如，如果你的URL是 `https://example.com/page?name=John&age=30`，那么 `window.location.search` 的值将是 `"?name=John&age=30"`。创建 `URLSearchParams` 对象后，你可以通过 `params.get('name')` 获取 `name` 参数的值，即 "John"。