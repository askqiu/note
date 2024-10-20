如何用浏览器进行网站源代码的静态分析—赏金猎人入门手册
@bl4de 迪哥讲事
 2023年05月21日 12:02 江苏
在这个手册中，我将展示如何用web浏览器的内置工具去分析客户端的源码。这可能就会有一些奇怪的声音，可能浏览器不是执行这个任务最好的选择，但是在你更深入之前，我们打开Burp Suite来拦截一下http的请求，或者在这里或者用alert(1)去寻找无尽的xss，首先去了解你的目标总是很好的主意

这篇文章主要面向的是那些对HTML和JavaScript代码没有经验或经验很少的赏金猎人，但是我希望更有经验的黑客也能发现其中有趣的东西。
我最近的一篇介绍基本操作的推文获得了社区很多的关注后，我就觉得应该写一篇这样的文章了。

![[Pasted image 20240909214158.png]]



这个简单的想法其实冰山的一角，如果我把这些小技巧全都发到推特上，那么其他人会很容易错过，所以我决定收集这些小技巧，然后把他们写成博客。我希望你们能找到一些有用的东西。
好了，让我们开始吧

工具集
每一个现代浏览器都会内置开发者工具，为了启动他们，你可以使用Ctrl+Shift+I, CMD+Option+I (macOS)，F12键或者在浏览器右边的菜单选项--这依靠你所使用的操作系统和浏览器。虽然在这篇文章中，我使用的是最新版本的Chromium，如果你使用Firefox, Safari, Chrome or Edge，他们除了UI，其他的没有什么不同。你可以随便选择你喜欢的浏览器，但是你会发现Chrome 开发者工具是最强大（Chrome开发者工具或者轻量级开发工具可以兼容Chrome, Chromium, Brave, Opera 或者其他基于Chromium 内核的浏览器）
你要安装好IDE（集成开发环境）或者任何一款带html和JavaScript代码高亮的编辑器。这些都是基于你自己的喜好，但是我发现Visual Studio Code特别好用（顺便说明，我用VSCode做所有的事情，包含我在我的工作中也会使用）。你可以用下面这个链接来下载适合你系统的VSCode
https://code.visualstudio.com/

安装NodeJS也是一个很不错的主意(只要经常用它就会越来越熟悉的--在互联网确实有成千上万的资源)。比较好用的在 https://nodejs.org/en/

python对我来说也是一个必备工具（如果你使用基于NIX的系统，你就有机会去使用它，并且它已经安装好了。如果你是windows用户，你必须自己手动安装Python）。能用Python写代码的能力是无价的，并且我建议那些从来没有写过代码的人可以试着使用一下Python

对于在终端中运行和测试JavaScript代码NodeJS是非常有用的（你也可以在浏览器中实现，但是我们稍后会谈论到它们的优点和缺点）。你可以用Python创建你自己的脚本工具，这些工具可以很快的验证漏洞也可以实际的去利用它--我也会在这篇文章中展示我自己的工具。如果你解释其他的解释型语言（像 Ruby, PHP, Perl, Bash等），你也可以使用它们。上面这些语言的主要好处在于它们不用编译就可以运行,也可以直接用命令行把它们执行起来。它们可以百分之百的跨平台，而且你可以使用网络上的很多库和模块。
OK，现在终于都弄清楚了

查看HTML源码
让我们回到刚才我引用的那个推文上去。你可能会注意，截屏的网页似乎没有内容，似乎仅仅是空白页面。
但是你要看网页的源代码（用CTRL+U 或者在mac上用CMD+Option+U）你会看到大量的代码（不幸的是，我不能提供截屏中的那个网站的url，因为那是一个众测项目的私有项目）。为什么那些元素不会展现在浏览器中？
重要的事情是，有些HTML标签不会在页面中展现任何东西，HTML中有很多这样的标签，我在这里举一些基本的例子<html>, <head>, <body>, <style> or<script>。并且，css也可以因此一些元素（比如，通过设置元素的高和宽都为0，或者设置display为none)
比如下面这个例子：

<html>
<head>
    <title>Move along, nothing to see here!</title>
    <style>
    /* note to myself: add CSS from Bob's repo: https://verysecurecompany.com/__internal__/repo/bob/specs.git */
    * {
        font-size:16px;
        color: #c0c0c0;
    }
    </style>
</head>
<body>
    <iframe src="https://verysecurecompany.com/__internal__/loginframe.html" style="width:0;height:0" frameborder="0" id="you-cant-see-me"></iframe>
    <script>
        // a hidden feature
        console.log('Diagnostic message: username is admin and password is password :)');    
    </script>
</body>
</html>
如果你在浏览器中打开这样的html页面，它不会显示任何东西并且你也不会从中看到任何东西。但是当你查看源码的时候，你会发现很多有趣的东西。

![[Pasted image 20240909214250.png]]


这里面有很多有价值的信息：urls指向了内部的资源，带有登录框页面的隐藏框架，甚至诊断信息中带有认证信息，而这些信息可以显示开发者工具的console中。虽然这个页面中没有显示任何东西。当然，你不要指望你会在每一个网站上面发现这些信息，可是在常见的情况中，很多JavaScript代码是被注释掉的，有时你能通过这些代码发现那些仍然可以被访问的服务器端api接口。
但是如果只查看源代码的话不会看到所有的东西，因为它只会呈现当前的HTML文档，被<iframe>,<script>等类似的标签加载的外部资源会包含更多有趣的东西。你会在Chrome开发者工具中看到这些资源的源码：
![[Pasted image 20240909214316.png]]



树状图最底部的那个按钮是主HTML文档，你可以用“查看源”这个选项去查看它们。所有的资源都会以标准的文件夹和文件树的形式所呈现。如果你点开这些文件，它们的内容就会显示在右边。在上面那个截图中，就是jquery.min.js的文件内容，并且你会经常发现这些JavaScript文件的压缩版本（从web应用程序的性能角度来看，这是很好的习惯）。但是如果你点击最下面那个小图标{}，开发者工具将会“解压”这些代码，让这些代码变的可读。
![[Pasted image 20240909214329.png]]


一些网站会使用一种特殊的功能去安排源码（变形后，真正代码中的变量名，函数名，对象名会被替换掉，而这个也会被用于代码压缩中--你会在 https://developers.google.com/web/tools/chrome-devtools/javascript/source-maps 找到关于源码映射的资料。通过给对象提供有意义的名字，同时替换掉那些被压缩的JavaScript变量，可以让格式化后的代码更易读。

另一个更强大的功能是tab中的全局搜索。假设，你发现了一个有趣的函数，你想要找到它在哪里调用的。可能这个函数中包含eval函数，它的参数来自于url，这样你就可以用这个url来执行任意JavaScript代码了。你可以使用CTRL+Shift+F（在mac系统中你可以用CMD+Option+F）去使用全局搜索这样功能。在下图这个例子中，我试图在 AppMeasurement.js中寻找所有引用getAccount函数的地方。你会看到这个函数仅仅被调用了一次，还是在同一个文件中，如果在其他文件中找到这样的字符串，它就会被显示在结果列表中:
![[Pasted image 20240909214341.png]]



有时，你会发现搜索结果在非常，非常长的字符串之中（尤其是那种经过压缩后的JavaScript文件）。你用开发者工具打开这个文件，点下面的{}图标，之后就会在右边展示解压之后的代码了，即使这个文件有好几千行都没有问题。
开发者工具的第二个tab被称为Elements。如你所见，对于在(index)中的源码来说（或者你以源码模式查看网页源代码），Elements这个tab有一点非常大的不同，虽然在Elements中也提供了内容。

前者显示从服务器端加载的HTML文件，Elements则会显示你当前的dom树，包括通过JavaScript代码创建和添加的元素。为了明白这点的不同，我会提供一个小的例子，但是首先，我要先介绍一点原理。
DOM(文档对象模型)实际代表了所有的html节点，dom树有一个根节点(<html>)，还有两个重要的子节点<head>和<body>,所有的其他元素要么是<head> 的子节点（像<title>或 <meta>,要么是<body>的子节点（<div>, <p>, <img>等）
当你在你的浏览器中个打开一个url时，HTML文件首先会被加载进来，然后代码会被浏览器引擎所解析。当浏览器发现<script>或者<style>标签时（或者其他带src属性的标签时，像image 文件或者 video 文件时），它会停止解析HTML并且加载那些文件。如果要执行JavaScript代码时，这些代码也会被马上的执行。如果有样式表的话，css解析器也会把css代码解析成css样式规则。所有的事情原理就像下面这张图一样（这个图非常简单，但是足够说明这些基本的概念）

![[Pasted image 20240909214352.png]]

Elements所包含的内容和源码所包含的内容不同之处是什么呢？

像下面这个例子，JavaScript添加一个元素到DOM中去：



<html>
<head>
    <title>Dynamic P Application</title>
    <style>
    * {
        font-size:18px;
        font-weight:bold;
        color: #2e2e2e;
    }
    </style>
</head>
<body>
    <div id="container">

    `</div>`
    `<script>`
        `const el = document.getElementById('container')`
        `const dynamic_paragraph = document.createElement('p')`
        `const dp_content = document.createTextNode('Hello from dynamically added <P>aragraph!')`

        `dynamic_paragraph.appendChild(dp_content)`
        `el.appendChild(dynamic_paragraph)`
    `</script>`
</body>
</html>
当你打开浏览器并查看源码时，你会发现所呈现的内容和上面的代码是一样的。
图片
![[Pasted image 20240909214426.png]]

在这个非常简单的例子中，JavaScript添加一个元素到DOM树中。

为了看清这样的不同，在开发者工具中使用Elements tab去查看

![[Pasted image 20240909214405.png]]


当你对比Elements标签的中的内容和查看源代码中的内容，你会很容易发现它们的不同点。

在Elements标签页中，你可以看到<p> 元素之间的内容，它被添加到了<div id=”container”>元素的子节点中。

你源码模式中不会看到这些元素，因为它们不存在于源代码中。



如果你用一些框架处理这些单页应用，例如AngularJS, React, Vue.js, Ember.js等。你会看到大量的动态内容被添加到标签之中。这些内容包含变量，还有表单，带分页，排序搜索属性的动态表格或列表。这些元素会造成大量DOM XSS的产生，或者前端模板会解析用户的输入（像AngularJS会解析{{ }}）
漏洞的原因在于，应用常常使用来自GET请求，POST请求中，保存在cookie中的数据或者浏览器存储中的数据去渲染网站应用中的内容。并且应用自己也会创建很多东西，所以总是会有机会去发现各种各样的漏洞。

在我们进入到JavaScript这一章节之前，还有一点非常重要的事情要说，你要注意网页源代码中那些没有被渲染的注释。你会发现非常多有价值的东西。

查看cookie和浏览器存储
你用开发者工具做的另外一件事情就是去检查那些存储在客户端上的信息。网站应用经常会用到两个地方。其中最常见的就是cookie--通过名称来识别的一小片数据（其实cookie就是简单的键-值对数据），通过http请求包和返回包，cookie会在客户端和服务器端来回交换。

浏览器存储是另一个地方，你会在其中发现很多有价值的东西。它们有两种存储形式：本地存储和session存储。这两种存储方式的不同点在于，当你关闭应用时，session会消失（当你关闭浏览器的tab时或者关闭整个浏览器时）。而如果你没有指定时间的话，本地存储会保存相当一段时间（数据本身没有过期时间）

你可以使用开发者工具中的Application tab去查看所有存储在本地的信息。

![[Pasted image 20240909214508.png]]

使用Application tab你不仅仅能看到这些数据的内容，你还可以去修改，删除，和增加你自己所需要的键以及对应的值，修改这些值之后，应用可能会发生一些不可思议的现象，有时甚至会触发漏洞。通过这样的方式去修改session token，看会不会导致越权的产生--只要改变维持会话的cookie值就可以了（这仅仅是一个例子，现代web应用程序使用几种不同的方式去识别用户并且仅仅改变单个cookie不足以冒充为其他的用户）
![[Pasted image 20240909214521.png]]

这个标签页上还有一个位置，你可以在那个上面发现JavaScript源码和web应用程序的关系：

Service Workers。

你可以在下面这个网站中找到关于Service Workers的介绍 -- https://developers.google.com/web/fundamentals/primers/service-workers/



这里还是有不少新的东西，不仅许多web应用程序会用到，还可以用它分析web应用是如何工作的，尤其在web应用离线时。

分析JavaScript
现在我们来到代码这一章节，这些代码会运行在整个web应用之中（HTML和css仅仅只会作为展示，它们不会包含任何逻辑。但还是有一小部分css的表达式可以运行JavaScript代码，这种特性会导致xss漏洞--但是在纯HTML和css组成的网页中，这样的机会不是很多）
有几种方式去分析JavaScript代码，我们先用浏览器中的工具试试。我已经介绍了关于Sources 标签和如何使用{}这个功能来让压缩后的代码变的可读。但是你可以用开发者工具做更多的事情其中最好用的一个就是JavaScript的debugger

使用DevTools debugger
如果你不了解debug是什么，那么通俗来说，就是让程序停在某一行代码上。这让你可以看到实际的变量值，实际所执行的函数和函数怎么样被调用的（这个优点主要得益于调用栈--debugger展示了函数的调用顺序，像函数a被函数b调用，在此之前，函数b被另一个函数c调用）。并且debugger允许你单步运行代码（一条指令），这可以让你有机会跟踪程序的每一次改变和其中的状态。最后一点，debugger可以修改运行时的程序，这意味着，当你修改变量时甚至程序自身的逻辑时，程序将会发生什么。高效的使用debugger是一种非常好的方式，我认为这是每一个优秀程序员都应该具备的重要技能。

从赏金猎人的角度来看，debugging可以让你更好的明白程序是怎样工作的，你也可以直接测试你的payload。你也可以很轻松的直接从程序中分离出有漏洞的代码，并且可以用debugger给予你的有力工具去测试这些东西。例如，想象一下，你发现了一个有任意重定向漏洞的函数，你想了解这个函数每一次到底做了什么，这个函数被调用之后浏览器就被重定向到了一个外部的资源，当你被重定向之后，你不会重定向之后的页面里面看到上一步页面的代码。

在重定向函数之前设置一个设置断点，浏览器运行到重定向函数之前就可以停下来，现在你去读取函数的源代码，去了解这个函数是如何工作的，想清楚你要如何注入你的payload，是否需要将你的payload进行编码或者去做其他的事情。

了解完原理之后，就去练习一下。
下面这个代码实现了一个简单重定向的功能

<html>
<head>
    <title>Redirection</title>
</head>
<body>
    </div>
    <script>

        `// imagine that read url from  GET parameter routine goes here...`
        `// but we just hardcode it for now :)`
        `const url = 'https://hackerone.com'`


        `function redirect() {`
            `// I will redirect you! Now!`
            `if (url) {`
                `location.href = url`
            `} else {`
                `location.href = 'https://company.com/__internal__/supersecretadminpanel'`
            `}`
        `}`


        `setTimeout( redirect, 10000 )`
    `</script>`
</body>
</html>
当你用浏览器打开这个网站，十秒后，你就会重定向到HackerOne 这个网站中，之后你就再也看不到原始的代码了，因此，你可以不能看到刚才发生的事情。
在浏览器中打开开发者工具，然后选择源代码标签，然后打开上面的HTML文件。现在你有10秒钟的时间在源码的第16行设置断点(if (url) {)。你只要在左边那个带数字的框框上点16那个数字就好了。当十秒过去之后，浏览器就会调用redirect()，然后马上会停到你下断点的那一行：

![[Pasted image 20240909214535.png]]



当我们向下走一步之后，蓝色的那一行就会被执行（蓝色那一行现在还没有执行！这是非常重要的）。你看debugger面板的左侧--那里会展示你现在在哪个位置（调用栈）然后，你如果打开script那个节点，你会看到在执行过程中所有被定义的变量的值（在这个例子中只有url）

现在你需要花费一些时间去阅读和理解这些代码。如你所知，如果16行的那个条件语句为TRUE的时候，我们将会跳转到HackerOne那个网站
让我们修改url这个值，我们去修改一些东西，那么JavaScript中什么样的值会让逻辑判断语句认为这是一个false呢（可能是空的字符串，0，布尔 false，或者其他代表false的表达式）。让我们把他变成false吧（只要点击变量，然后输入你想要的值，就可以改变这个变量的值了）


![[Pasted image 20240909214548.png]]

现在让我们看看发生了什么样的改变。我们将继续走入到下一步，看看debugger面板最顶部上面的图标：
![[Pasted image 20240909214556.png]]



第一个图标将让你继续运行这个程序，第二个图标允许你单步运行（我们一会再说这个），第三个图标允许你跳转进到被调用的函数之中（只要那里没有设置断点，debugger不会进入到函数中去的，它会执行那一行的函数，然后移到下一行去执行），这里有一个叫“跳出”的图标，这个图标可以让函数继续执行，然后跳转到被调用的那一个点
现在点击第二个图标（你会看到url的值已经变成了false），然后你会注意到执行的下一行代码变成了19行（这是因为我们在16行判断语句中设置了false）
![[Pasted image 20240909214610.png]]


如果你按下debugger工具栏上第一个按钮之后（“Play”按钮），这次你会注意到应用会跳转到一个company.com中的一个链接里面去。



你会想到url这个参数可能会是一个漏洞，所以你现在尝试去利用这个问题去发现任意跳转漏洞或者反射型xss漏洞，或者更深一步，你会怀疑这个url会存储在服务器端的某个位置（如果你进一步探索程序内部的逻辑，就能发现你的想法是否是正确的）

用Snippets执行JavaScript
有时，你想执行应用中的一部分代码。这可能比较困难，特别是遇到要做大量前置准备工作之后才能触发的代码，就会特别耗时。在这种情况下，你可以使用Snippets去运行你的代码。但是要记住，每一次测试都不会很顺利，比如当你运行一些需要其他依赖的代码时，通常像一些变量会来自其他部分的代码或者代码片段中包含来自其他文件中函数。
但是让我们假设，你要检测一个函数，以检查它所提供的值是否是正确的，并且你希望你只关注代码的逻辑部分。
在源码标签，你会发现一个叫Snippets的面板
![[Pasted image 20240909214619.png]]


当你点击它的时候，你会发现一些片段列表（如果你已经创建了所有的东西）然后可以点击创建new去创建一个新的片段。

点击这个选项，然后在控制面板中间的位置，你会看到一些简单的示例代码，这些代码是可被编辑的而且带有代码高亮。

你可以把所有的JavaScript代码放上去运行，当你点击位于控制面板左下底部的play按钮（或者你可以用CMD+Enter，在mac系统上你可以用CTRL+Enter）后，你会发现结果会马上打印到下面的控制台面板上去。


你可以无限制的修改和运行你的代码，但是我刚才提到，代码在这里运行时会遇到依赖问题。



Snippets中代码运行于你打开开发者工具的那个页面，你可以在snippet创建和运行你的代码。并且，每一次代码的运行环境都是相同的，这就意味者你之前定义的所有变量依然没有被改变，并且会一直保留那些值。
为什么这很重要？
想想下面的这个例子：
![[Pasted image 20240909214627.png]]


在JavaScript中，当你使用const关键字定义一个常量的时候，你会初始化它的值，并且不会去改变他。

如你所见，代码片段按照预期的行为去工作，但是你如果试图改变SOME_CONST的值，再次运行snippet，你会受到下面的这个语法错误：

![[Pasted image 20240909214632.png]]

这个错误是因为SOME_CONST这个静态变量已经被初始化。

开发者工具就想，你要在同一个执行环境中继续执行,你编辑的代码已经变的不重要了。



所以你如果用debugger停止正在运行的程序（所有通过程序代码定义的变量，类和函数将会存在于执行环境之中）。如果你试图使用现有的标志在同一个标签页中创建snippet--有时你要么会覆盖原来网页的代码，要么在 重新初始化某些变量时会让页面报错（比如常量）。为了能够重新运行snippet，首先你要通过刷新浏览器来重新加载页面，清空执行环境（浏览器不会记住web应用的状态的，只要刷新之后，整个进程会重新加载资源，重建DOM树，等等）

为了避免上面的问题，不要使用Snippets（还有有一种比较好的办法，就是你新开一个浏览器标签页，然后打开开发者工具并在那里创建一个新的snippet），你可以使用NodeJS来运行你的JavaScript代码
你可以把你的代码放入一个新的JavaScript文件中，然后用在终端中用NodeJS（你要确定你已经安装好了）运行它：
![[Pasted image 20240909214639.png]]



我将运行三次这个代码，每一次我都会修改SOME_CONST的值。如你所见，没有任何报错，而且每一次执行都会成功，输出的结果也都是正确的。
这是因为，你用NodeJS运行这个代码的时候，它都会创建一个新的执行环境，所以，它不会把一份代码在相同执行环境中运行两次。

Sources 和 execution sinks
当你看JavaScript代码时，首先你要着重注意两个地方

首先第一个是sources，这个术语描述了，用户输入的每一个点都应该被应用程序所处理。GET请求中的url里面的参数，能被应用程序读取到的cookie或者应用程序使用到的本地存储。

第二个被称为execution sink，这个术语意味着，所有的JavaScript语法元素或者那些可以执行代码的HTML API。一个很明显的例子就是JavaScript中的eval(code_to_evaluate)，这个函数可以通过参数来执行代码。另一个例子是setTimeout(function_to_execute, timeout_in_miliseconds),可以通过这个函数的第一个参数去执行一个函数，前提是要等到第二个参数的时间到了之后才可以执行。
在应用程序中发现漏洞的过程就是寻找source和处理此source的execution sink之间的连接。在上面这个例子中，我将展示如何使用debugger，url作为参数(source) 被直接放入location.href (execution sink) 中。另一个例子是，一个函数会获取HTML输入框中用户输入的数据（JavaScript能通过DOM API读取到这些数据，例如document.getElementById(‘input_id’).value,然后把这些值传递给一个变量，这也可能会是一个source。）然后这个值会被放入到innerHTML()函数中，之后这个函数会更新浏览器中DOM（这就会成为一个execution sink）
https://www.youtube.com/watch?v=ZaOtY4i5w_U
这里有一个非常棒的视频，它的作者是@LiveOverflow。我建议你马上去看看这个视频以熟悉它概念（这个视频大概8分钟长）
在web应用程序中，由于其复杂的业务逻辑，会存在非常多的sources 和 execution sinks（想想那些输入框，url参数，cookie，浏览器存储，WebSockets等）。但是重要的东西是那些可以被作为execution sinks的函数。它们也有很多，像location属性：href or hash, window.open(), document.write()，或者DOM函数：innerHTML 或者 appendChild。它们都可以执行任意代码，任意跳转，或者执行其他类型的注入。

为了识别上面这些代码特征，我写了一个工具nodestructor，这个工具可以检查JavaScript文件（参数是单个文件或者包含所有JavaScript文件的文件夹），这个工具可以根据特征寻找execution sinks（或者sources）。不要指望nodestructor会识别每一行代码或者更容易的去利用--所有的东西都应该依赖于source 和 execution sink（过滤，编码，解析，将数据转换为object，字符串操作等）。这个工具的主要目的是更容易和更快速的在大量代码中通过规则来寻找有漏洞的代码。

让我们用一个例子来快速的展示这个工具吧。首先，我需要一个JavaScript文件。我将检查来自GM.com 网站的AppMeasurement.js。然后我把这个文件从浏览器中复制下来（首先要解压，并格式化好），然后粘贴到代码编辑器中，然后后保存在一个临时文件夹中。

在终端中，我在AppMeasurement.js文件上运行nodestructor（-H参数是让这个工具去搜索各种HTML5的APIs）

![[Pasted image 20240909214646.png]]



如你所见，这个工具识别了大量潜在的execution sinks。它们中大多数都是误报，但是让我们集中精力去看看报告结果中的第二个，它看起来像是直接用location.hostname.toLowerCase() 这个函数的结果来初始化domain这个变量。

如果有可能在这个文件中去跟踪这些变量所有的引用，看看它是否被用在了execution sink中。你也可以使用Visual Studio Code内置工具像Find all references去寻找所有的domain字符串。

我想进一步来挖掘更多的漏洞，所以不久前，我已经开始用我的工具去做这样的事情了--这个是一个简单的JavaScript静态分析工具，所以它没有很花哨的名字。这个工具仍处于开发的早期阶段（它虽然现在还只是个poc，但其实已经很好用了），当我运行这个工具去查找domain这个变量时，它会很快的给出结果的预览（到目前为止，这是唯一可用的选项，正如我所提到的一样，这个工具仅仅只是一个雏形，所以例如文件名这样的东西，看上去就像硬编码在代码里面一样）

![[Pasted image 20240909214655.png]]



如你所见，工具能识别变量在哪里被定义的，在什么时候或者什么地方被怎样使用的。我希望工具能够执行更复杂的分析，像是在不同范围内寻找变量（例如，变量是否是一个函数的参数或者变量是否被当成execution sink中的参数）

总结
web浏览器十一个非常强大的工具。有时，你仅仅会用这个工具去读源代码和了解应用程序是如果工作的，还可以用浏览器寻找漏洞，测试你发现的漏洞，或者仅仅是去了解它是如何工作的并且去了解一些新的东西。

我希望我的文章能让你了解到如何使用浏览器开发者工具这一强大功能。你能通过下面的这个链接找到更多强大的工具。
https://developers.google.com/web/tools/chrome-devtools/

如果你有任何问题或者对这篇文章有好的建议--不要犹豫，来推特上找我 https://twitter.com/_bl4de

感谢你对阅读，我希望你能找到更多有价值的漏洞

原文:https://medium.com/@_bl4de/how-to-perform-the-static-analysis-of-website-source-code-with-the-browser-the-beginners-bug-d674828c8d9a
