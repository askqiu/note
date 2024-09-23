### 使用 Java 和 Spring Boot 创建一个前后端登录页面并部署到 Tomcat 的完整指南

本指南将引导你使用 **Spring Boot** 创建一个简单的前后端登录页面，并将其部署到 **Apache Tomcat** 服务器上。整个过程包括项目创建、编写前端页面和后端逻辑、打包应用程序以及部署到 Tomcat。

---

## 目录

1. [前置条件](#1-前置条件)
2. [创建 Spring Boot 项目](#2-创建-spring-boot-项目)
3. [编写前端登录页面](#3-编写前端登录页面)
4. [编写后端登录逻辑](#4-编写后端登录逻辑)
5. [配置 Spring Boot 生成 WAR 文件](#5-配置-spring-boot-生成-war-文件)
6. [构建项目](#6-构建项目)
7. [部署到 Tomcat](#7-部署到-tomcat)
8. [验证部署](#8-验证部署)
9. [总结](#9-总结)

---

## 1. 前置条件

在开始之前，请确保你的开发环境具备以下条件：

- **Java Development Kit (JDK)**：建议使用 JDK 11 或更高版本。检查是否已安装：

  ```bash
  java -version
  ```

  如果未安装，可以使用以下命令安装 OpenJDK 11：

  ```bash
  sudo apt update
  sudo apt install openjdk-11-jdk -y
  ```

- **Maven**：用于项目构建。检查是否已安装：

  ```bash
  mvn -version
  ```

  如果未安装，可以使用以下命令安装：

  ```bash
  sudo apt install maven -y
  ```

- **Apache Tomcat**：确保已在 Ubuntu 上安装并运行。默认情况下，Tomcat 使用 8080 端口。

- **文本编辑器或 IDE**：推荐使用 [IntelliJ IDEA](https://www.jetbrains.com/idea/)、[Eclipse](https://www.eclipse.org/) 或 [VS Code](https://code.visualstudio.com/) 进行开发。

---

## 2. 创建 Spring Boot 项目

我们将使用 **Spring Initializr** 创建一个 Spring Boot 项目，并配置为生成 WAR 文件，以便部署到外部 Tomcat 服务器。

### 步骤 1：访问 Spring Initializr

打开浏览器，访问 [Spring Initializr](https://start.spring.io/)。

### 步骤 2：配置项目元数据

在 Spring Initializr 页面上，填写以下信息：

- **Project**: Maven Project
- **Language**: Java
- **Spring Boot**: 选择最新的稳定版本（如 3.0.0）
- **Project Metadata**:
  - **Group**: `com.example`
  - **Artifact**: `loginapp`
  - **Name**: `loginapp`
  - **Description**: `A simple Spring Boot login application`
  - **Package name**: `com.example.loginapp`
  - **Packaging**: `War`
  - **Java**: `11` 或更高版本

### 步骤 3：选择依赖

点击 “Add Dependencies” 并添加以下依赖：

- **Spring Web**: 用于构建 Web 应用，包括 RESTful 服务。
- **Thymeleaf**: 模板引擎，用于渲染 HTML 页面。
- **Spring Boot DevTools**（可选）：用于开发时的自动重启和其他开发工具。

> **注意**: 不添加 Spring Security 依赖，因为我们将实现一个简单的登录逻辑。如果需要更复杂的安全性，可以在后续步骤中添加。

### 步骤 4：生成项目

点击 “Generate” 按钮，下载生成的项目压缩包（`loginapp.zip`）。

### 步骤 5：解压项目

在终端中运行以下命令解压项目并进入项目目录：

```bash
unzip loginapp.zip
cd loginapp
```

---

## 3. 编写前端登录页面

我们将使用 **Thymeleaf** 创建一个简单的登录页面。

### 步骤 1：创建登录页面模板

在项目目录中，导航到 `src/main/resources/templates/` 并创建一个名为 `login.html` 的文件：

```bash
nano src/main/resources/templates/login.html
```

### 步骤 2：编写 `login.html` 内容

将以下 HTML 代码粘贴到 `login.html` 中：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f2f2f2;
        }
        .login-container {
            width: 300px;
            padding: 16px;
            background-color: white;
            margin: 100px auto;
            border-radius: 5px;
            box-shadow: 0px 0px 10px 0px #0000001a;
        }
        input[type=text], input[type=password] {
            width: 100%;
            padding: 12px 8px;
            margin: 8px 0;
            display: inline-block;
            border: 1px solid #ccc;
            box-sizing: border-box;
            border-radius: 4px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 12px 8px;
            margin: 8px 0;
            border: none;
            cursor: pointer;
            width: 100%;
            border-radius: 4px;
        }
        .error {
            color: red;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>

<div class="login-container">
    <h2>登录</h2>
    <div th:if="${error}" class="error" th:text="${error}"></div>
    <form th:action="@{/login}" method="post">
        <label for="username"><b>用户名</b></label>
        <input type="text" placeholder="输入用户名" name="username" required>

        <label for="password"><b>密码</b></label>
        <input type="password" placeholder="输入密码" name="password" required>

        <button type="submit">登录</button>
    </form>
</div>

</body>
</html>
```

> **说明**:
>
> - 使用了简单的 CSS 来美化登录表单。
> - 如果存在错误信息，将显示在表单上方。

---

## 4. 编写后端登录逻辑

我们将创建一个控制器来处理登录请求，并进行简单的身份验证。

### 步骤 1：创建控制器类

在 `src/main/java/com/example/loginapp/` 目录下创建一个名为 `LoginController.java` 的类：

```bash
nano src/main/java/com/example/loginapp/LoginController.java
```

### 步骤 2：编写 `LoginController.java` 内容

将以下 Java 代码粘贴到 `LoginController.java` 中：

```java
package com.example.loginapp;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class LoginController {

    // 显示登录页面
    @GetMapping("/login")
    public String showLoginPage() {
        return "login";
    }

    // 处理登录请求
    @PostMapping("/login")
    public String handleLogin(@RequestParam String username,
                              @RequestParam String password,
                              Model model) {
        // 简单的用户名和密码验证
        if ("admin".equals(username) && "password".equals(password)) {
            model.addAttribute("message", "登录成功，欢迎 " + username + "!");
            return "success";
        } else {
            model.addAttribute("error", "用户名或密码错误，请重试。");
            return "login";
        }
    }

    // 显示成功页面
    @GetMapping("/success")
    public String showSuccessPage() {
        return "success";
    }
}
```

> **说明**:
>
> - `@Controller`: 标识这是一个 Spring MVC 控制器。
> - `@GetMapping("/login")`: 处理 GET 请求，显示登录页面。
> - `@PostMapping("/login")`: 处理 POST 请求，处理登录逻辑。
> - 简单验证用户名为 "admin" 且密码为 "password"。
> - 根据验证结果，返回成功页面或重新显示登录页面并显示错误信息。

### 步骤 3：创建成功页面模板

在 `src/main/resources/templates/` 目录下创建一个名为 `success.html` 的文件：

```bash
nano src/main/resources/templates/success.html
```

### 步骤 4：编写 `success.html` 内容

将以下 HTML 代码粘贴到 `success.html` 中：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>登录成功</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #e2f0d9;
            text-align: center;
            padding-top: 100px;
        }
        .message {
            background-color: #ffffff;
            padding: 20px;
            display: inline-block;
            border-radius: 5px;
            box-shadow: 0px 0px 10px 0px #0000001a;
        }
        a {
            display: block;
            margin-top: 20px;
            text-decoration: none;
            color: #4CAF50;
        }
    </style>
</head>
<body>

<div class="message">
    <h2 th:text="${message}">登录成功！</h2>
    <a href="/login">返回登录页面</a>
</div>

</body>
</html>
```

> **说明**:
>
> - 显示登录成功的消息。
> - 提供一个链接返回登录页面。

---

## 5. 配置 Spring Boot 生成 WAR 文件

为了将 Spring Boot 应用部署到外部 Tomcat 服务器，我们需要配置项目生成 WAR 文件。

### 步骤 1：修改 `pom.xml`

打开 `pom.xml` 文件：

```bash
nano pom.xml
```

将以下内容替换或添加到 `pom.xml` 中：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>loginapp</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>loginapp</name>
    <description>A simple Spring Boot login application</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Thymeleaf -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <!-- Servlet API (provided by Tomcat) -->
        <dependency>
            <groupId>jakarta.servlet</groupId>
            <artifactId>jakarta.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludeDevtools>true</excludeDevtools>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

> **说明**:
>
> - **`<packaging>war</packaging>`**：将项目打包类型设置为 WAR。
> - **`jakarta.servlet-api`**：提供 Servlet API，作用域设置为 `provided`，因为 Tomcat 已经提供了这些类库。
> - **Spring Boot Maven Plugin**：用于打包 Spring Boot 应用。

### 步骤 2：修改主应用类

为了支持在外部 Servlet 容器（如 Tomcat）中部署，需修改主应用类，使其继承 `SpringBootServletInitializer` 并重写 `configure` 方法。

打开 `LoginappApplication.java` 文件：

```bash
nano src/main/java/com/example/loginapp/LoginappApplication.java
```

修改内容如下：

```java
package com.example.loginapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class LoginappApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(LoginappApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(LoginappApplication.class);
    }
}
```

> **说明**:
>
> - **继承 `SpringBootServletInitializer`**：以支持在外部 Servlet 容器中部署。
> - **重写 `configure` 方法**：指定应用的主类。

---

## 6. 构建项目

使用 Maven 打包项目为 WAR 文件。

### 步骤 1：清理和打包项目

在项目根目录下运行以下命令：

```bash
mvn clean package
```

### 步骤 2：验证生成的 WAR 文件

构建成功后，你将在 `target/` 目录下找到生成的 WAR 文件，如 `loginapp-0.0.1-SNAPSHOT.war`。

```bash
ls target/
```

你应该看到类似以下的输出：

```
loginapp-0.0.1-SNAPSHOT.jar
loginapp-0.0.1-SNAPSHOT.war
classes/
generated-sources/抽的
...
```

---

## 7. 部署到 Tomcat

将生成的 WAR 文件部署到 Apache Tomcat 服务器。

### 方法 1：直接复制 WAR 文件到 `webapps` 目录

#### 步骤 1：上传 WAR 文件到服务器

如果你在本地开发，可以使用 `scp` 将 WAR 文件上传到 Ubuntu 服务器。例如，将 WAR 文件上传到 `/tmp/` 目录：

```bash
scp target/loginapp-0.0.1-SNAPSHOT.war your-username@your-server-ip:/tmp/
```

#### 步骤 2：移动 WAR 文件到 Tomcat 的 `webapps` 目录

SSH 登录到 Ubuntu 服务器：

```bash
ssh your-username@your-server-ip
```

移动 WAR 文件到 Tomcat 的 `webapps` 目录：

```bash
sudo mv /tmp/loginapp-0.0.1-SNAPSHOT.war /opt/tomcat/webapps/
```

> **注意**:
>
> - Tomcat 的安装目录可能不同，默认通常在 `/opt/tomcat`。如果你的 Tomcat 安装在其他位置，请相应调整路径。

#### 步骤 3：设置正确的权限

确保 Tomcat 用户对 WAR 文件有读取权限：

```bash
sudo chown tomcat:tomcat /opt/tomcat/webapps/loginapp-0.0.1-SNAPSHOT.war
sudo chmod 644 /opt/tomcat/webapps/loginapp-0.0.1-SNAPSHOT.war
```

> **说明**:
>
> - **`tomcat`**：假设 Tomcat 运行在 `tomcat` 用户下。根据实际情况调整。

#### 步骤 4：Tomcat 自动部署

Tomcat 会自动检测到新的 WAR 文件，并进行解压和部署。你可以查看 Tomcat 的日志以确认部署状态：

```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

你应该看到类似以下的日志输出，表示应用已成功部署：

```
INFO: Deploying web application archive /opt/tomcat/webapps/loginapp-0.0.1-SNAPSHOT.war
INFO: Deployment of web application archive /opt/tomcat/webapps/loginapp-0.0.1-SNAPSHOT.war has finished in 1234 ms
```

### 方法 2：使用 Tomcat 管理界面部署

#### 步骤 1：配置 Tomcat 管理用户

编辑 `tomcat-users.xml` 文件，添加管理用户：

```bash
sudo nano /opt/tomcat/conf/tomcat-users.xml
```

在 `<tomcat-users>` 标签内添加以下内容：

```xml
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="admin" password="123" roles="manager-gui,admin-gui"/>
```

> **注意**: 
>
> - 将 `adminpassword` 替换为你希望设置的强密码。
> - **安全性**: 仅在开发环境中使用简单的用户名和密码，生产环境中请采用更安全的认证方式。

#### 步骤 2：重新启动 Tomcat

保存并关闭文件后，重新启动 Tomcat：

```bash
sudo systemctl restart tomcat
```

#### 步骤 3：访问 Tomcat 管理界面

`在浏览器中访问 [http://192.168.206.130:8080/manager/html](http://your-server-ip:8080/manager/html)`。

1. **登录**: 使用刚刚在 `tomcat-users.xml` 中配置的用户名和密码（如 `admin/adminpassword`）。
2. **部署应用**:
   - 在 “**Deploy**” 部分下的 “**WAR file to deploy**” 部分，点击 “**Choose File**” 并选择你的 WAR 文件。
   - 点击 “**Deploy**” 按钮，Tomcat 会自动上传并部署应用。

#### 步骤 4：验证部署

部署成功后，你可以在 “**Applications**” 部分看到你的应用。访问 [http://your-server-ip:8080/loginapp-0.0.1-SNAPSHOT/](http://your-server-ip:8080/loginapp-0.0.1-SNAPSHOT/) 或根据 `pom.xml` 中的 `<finalName>` 设置访问应用。

---

## 8. 验证部署

确保应用已成功部署并能正常工作。

### 步骤 1：访问登录页面

在浏览器中输入以下 URL 访问登录页面：

```
http://your-server-ip:8080/loginapp-0.0.1-SNAPSHOT/login
```

> **注意**: 根据实际的上下文路径和应用名称调整 URL。

### 步骤 2：测试登录功能

1. **正确登录**:
   - **用户名**: `admin`
   - **密码**: `password`
   
   输入正确的用户名和密码后，应该会看到登录成功的页面，显示欢迎消息。

2. **错误登录**:
   - 输入错误的用户名或密码，应该会重新显示登录页面并显示错误信息。

### 步骤 3：查看 Tomcat 日志

如果遇到问题，可以查看 Tomcat 的日志文件以获取详细信息：

```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

---

## 9. 总结

通过本指南，你已经完成了以下步骤：

1. **创建 Spring Boot 项目**：使用 Spring Initializr 创建并配置项目生成 WAR 文件。
2. **编写前端页面**：使用 Thymeleaf 创建简单的登录页面。
3. **编写后端逻辑**：创建控制器处理登录请求并进行简单的身份验证。
4. **构建项目**：使用 Maven 打包项目为 WAR 文件。
5. **部署到 Tomcat**：将 WAR 文件上传到 Tomcat 的 `webapps` 目录或使用 Tomcat 管理界面进行部署。
6. **验证部署**：通过浏览器访问应用并测试登录功能。

### 进一步的改进

- **增强安全性**:
  - 使用 **Spring Security** 进行更强大的身份验证和授权。
  - 加密存储用户密码。
  - 实现用户注册和密码重置功能。

- **数据库集成**:
  - 将用户信息存储在数据库中（如 MySQL、PostgreSQL）。
  - 使用 **Spring Data JPA** 管理数据库操作。

- **前端优化**:
  - 使用前端框架如 **Bootstrap** 美化登录页面。
  - 实现前端验证，提高用户体验。

- **部署优化**:
  - 配置 **HTTPS** 以确保数据传输的安全性。
  - 设置反向代理（如 Nginx）以管理流量和提供额外的安全性。
  - 使用 **CI/CD** 工具（如 Jenkins、GitLab CI）自动化构建和部署流程。

### 资源参考

- [Spring Boot 官方文档](https://spring.io/projects/spring-boot)
- [Thymeleaf 官方文档](https://www.thymeleaf.org/documentation.html)
- [Apache Tomcat 官方文档](https://tomcat.apache.org/tomcat-10.0-doc/index.html)

如果在开发或部署过程中遇到任何问题，欢迎随时提问！祝你成功构建和部署你的 Java Web 应用程序！