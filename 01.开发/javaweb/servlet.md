是sun公司开发动态web的一门技术
把实现了servlet接口的java程序叫做servlet


1.maven父子项目
maven父项目的jar包子项目可以直接使用

2.编写一个servlet程序
	编写一个普通类
	实现servlet接口，这里直接继承HttpServlet

![[20241226221649.km]]

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
     PrintWriter writer= resp.getWriter();
    writer.println("我草饲你的马");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       doGet(req, resp);
    }
}

```

3.编写Servlet的映射
为什么需要映射？我们要通过浏览器访问我们的java程序，而浏览器需要连接web服务器，所以我们要在web服务中注册我们写的servlet，还要给他一个李凌秋能够访问的路径

```xml
<!-- 注册serv -->
    <servlet-name>hello</servlet-name>
    <servlet-class>com.fever.servlet.HelloServlet</servlet-class>
  </servlet>
  <!-- servlet 的请求路径-->
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
```


4.配置tomcat
配置项目发布的路径，在右上角绿色三角形旁边编辑
![[Pasted image 20241226230239.png]]
5.访问
http://localhost:8080/s1/hello
这里报错500了，不知道为啥