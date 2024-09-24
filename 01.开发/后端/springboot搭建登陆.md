搭建一个使用 Spring Boot 框架的前后端登录页面，而不依赖数据库，可以通过以下步骤在 Visual Studio Code (VS Code) 中实现。本指南将详细介绍从项目初始化到实现登录功能的全过程。

## 目录

1. [前提条件](#前提条件)
2. [项目初始化](#项目初始化)
3. [项目结构](#项目结构)
4. [配置依赖](#配置依赖)
5. [编写后端代码](#编写后端代码)
    - [用户模型](#用户模型)
    - [用户服务](#用户服务)
    - [控制器](#控制器)
    - [安全配置](#安全配置)
6. [编写前端代码](#编写前端代码)
    - [模板文件](#模板文件)
    - [静态资源](#静态资源)
7. [运行项目](#运行项目)
8. [总结](#总结)

---

## 前提条件

在开始之前，请确保您已经安装和配置了以下环境：

- **Java Development Kit (JDK) 11 或更高版本**: [下载链接](https://www.oracle.com/java/technologies/javase-downloads.html)
- **Visual Studio Code (VS Code)**: [下载链接](https://code.visualstudio.com/)
- **Maven**: 通常与 JDK 一起安装，确保 Maven 在环境变量中可用
- **Spring Boot 插件（可选）**: 在 VS Code 中安装相关插件以增强开发体验
- **Git**（可选）：用于版本控制

## 项目初始化

我们将使用 [Spring Initializr](https://start.spring.io/) 来生成项目骨架。

1. **访问 Spring Initializr**：

   打开浏览器，访问 [https://start.spring.io/](https://start.spring.io/)。

2. **项目设置**：

   - **Project**: Maven Project
   - **Language**: Java
   - **Spring Boot**: 选择最新稳定版本（如 3.1.x）
   - **Project Metadata**:
     - **Group**: `com.example`
     - **Artifact**: `loginapp`
     - **Name**: `loginapp`
     - **Description**: `A simple login application without database`
     - **Package name**: `com.example.loginapp`
     - **Packaging**: Jar
     - **Java**: 11 或更高
   - **Dependencies**:
     - **Spring Web**
     - **Thymeleaf**
     - **Spring Security**
     - **Spring Boot DevTools**（可选，方便开发时热重载）

3. **生成项目**：

   点击 **Generate** 按钮，下载生成的压缩包。将其解压到您希望的位置。

4. **在 VS Code 中打开项目**：

   启动 VS Code，选择 **File > Open Folder**，然后选择刚刚解压的项目文件夹。

## 项目结构

解压后的项目结构大致如下：

```
loginapp/
├── src/
│   ├── main/
│   │   ├── java/com/example/loginapp/
│   │   │   └── LoginappApplication.java
│   │   ├── resources/
│   │   │   ├── static/
│   │   │   ├── templates/
│   │   │   └── application.properties
│   └── test/
│       └── java/com/example/loginapp/
│           └── LoginappApplicationTests.java
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md
```

## 配置依赖

确保 `pom.xml` 包含以下必要的依赖。Spring Initializr 已经为我们添加了部分依赖，我们将进一步检查和添加所需的内容。

打开 `pom.xml`，确保包含以下依赖：

```xml
<dependencies>
    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Thymeleaf -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Spring Boot DevTools (optional) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>

    <!-- Lombok (optional, for reducing boilerplate code) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Test Dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

> **注**：如果您不打算使用 Lombok，可以忽略该依赖。

保存 `pom.xml`，Maven 将自动下载所需的依赖。

## 编写后端代码

### 用户模型

由于不使用数据库，我们可以创建一个简单的 `User` 类来表示用户信息，并在内存中存储用户数据。

1. **创建 `User` 类**：

在 `src/main/java/com/example/loginapp/model` 目录下创建 `User.java`。

```java
package com.example.loginapp.model;

public class User {
    private String username;
    private String password;

    // 构造器
    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    // Getter 和 Setter
    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### 用户服务

创建一个服务类，用于管理用户信息。在此示例中，我们将使用内存中的用户存储。

2. **创建 `UserService` 类**：

在 `src/main/java/com/example/loginapp/service` 目录下创建 `UserService.java`。

```java
package com.example.loginapp.service;

import com.example.loginapp.model.User;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class UserService {
    private Map<String, String> users = new HashMap<>();

    public UserService() {
        // 初始化默认用户
        users.put("user1", "password1");
        users.put("user2", "password2");
    }

    public boolean authenticate(String username, String password) {
        if (users.containsKey(username)) {
            String storedPassword = users.get(username);
            return storedPassword.equals(password);
        }
        return false;
    }

    public void addUser(String username, String password) {
        users.put(username, password);
    }

    public boolean userExists(String username) {
        return users.containsKey(username);
    }
}
```

### 控制器

创建控制器处理前端请求。

3. **创建 `AuthController` 类**：

在 `src/main/java/com/example/loginapp/controller` 目录下创建 `AuthController.java`。

```java
package com.example.loginapp.controller;

import com.example.loginapp.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class AuthController {

    @Autowired
    private UserService userService;

    @GetMapping("/")
    public String home(Authentication authentication, Model model) {
        if (authentication != null && authentication.isAuthenticated()) {
            model.addAttribute("username", authentication.getName());
            return "welcome";
        }
        return "login";
    }

    @GetMapping("/login")
    public String login() {
        return "login";
    }

    // 可选的注册页面
    @GetMapping("/register")
    public String register() {
        return "register";
    }
}
```

### 安全配置

配置 Spring Security，以使用我们自定义的用户服务进行身份验证。

4. **创建 `WebSecurityConfig` 类**：

在 `src/main/java/com/example/loginapp/config` 目录下创建 `WebSecurityConfig.java`。

```java
package com.example.loginapp.config;

import com.example.loginapp.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
// import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder; // Deprecated
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
// import org.springframework.security.core.userdetails.User.UserBuilder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
// import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
// import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class WebSecurityConfig {

    @Autowired
    private UserService userService;

    // 定义用户详情服务
    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        // 初始化用户
        // 这里使用未加密的密码，在生产环境中应使用加密的密码
        manager.createUser(User.withUsername("user1").password("{noop}password1").roles("USER").build());
        manager.createUser(User.withUsername("user2").password("{noop}password2").roles("USER").build());
        return manager;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/register", "/css/**", "/js/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/", true)
                .permitAll()
            )
            .logout(logout -> logout
                .permitAll()
            );

        return http.build();
    }
}
```

> **说明**：
>
> - 我们使用 `InMemoryUserDetailsManager` 来管理用户，预先添加了两个用户 `user1` 和 `user2`。
> - 密码前缀 `{noop}` 表示不使用加密。在实际应用中，应使用加密密码（如 BCrypt）。
> - 配置了自定义的登录页面 `/login`，并允许访问 `/register` 页面（如果实现注册功能）。

## 编写前端代码

我们将使用 Thymeleaf 模板引擎来创建前端页面。

### 模板文件

1. **登录页面 (`login.html`)**：

在 `src/main/resources/templates` 目录下创建 `login.html`。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>登录</title>
    <link rel="stylesheet" th:href="@{/css/style.css}">
</head>
<body>
    <h2>登录</h2>
    <form th:action="@{/login}" method="post">
        <div>
            <label>用户名:</label>
            <input type="text" name="username" required />
        </div>
        <div>
            <label>密码:</label>
            <input type="password" name="password" required />
        </div>
        <div>
            <button type="submit">登录</button>
        </div>
        <div>
            <a th:href="@{/register}">注册</a>
        </div>
    </form>
    <div th:if="${param.error}">
        <p style="color:red;">用户名或密码错误</p>
    </div>
    <div th:if="${param.logout}">
        <p style="color:green;">您已成功注销</p>
    </div>
</body>
</html>
```

2. **欢迎页面 (`welcome.html`)**：

在 `src/main/resources/templates` 目录下创建 `welcome.html`。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>欢迎</title>
    <link rel="stylesheet" th:href="@{/css/style.css}">
</head>
<body>
    <h2>欢迎, <span th:text="${username}">用户</span>!</h2>
    <form th:action="@{/logout}" method="post">
        <button type="submit">注销</button>
    </form>
</body>
</html>
```

3. **注册页面 (`register.html`)**（可选）：

如果需要注册功能，可以创建 `register.html`。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>注册</title>
    <link rel="stylesheet" th:href="@{/css/style.css}">
</head>
<body>
    <h2>注册</h2>
    <form th:action="@{/register}" method="post">
        <div>
            <label>用户名:</label>
            <input type="text" name="username" required />
        </div>
        <div>
            <label>密码:</label>
            <input type="password" name="password" required />
        </div>
        <div>
            <button type="submit">注册</button>
        </div>
    </form>
    <div th:if="${message}">
        <p th:text="${message}"></p>
    </div>
</body>
</html>
```

4. **更新 `AuthController` 以支持注册（可选）**：

如果添加了注册页面，还需要在 `AuthController` 中添加处理注册请求的代码。

```java
@PostMapping("/register")
public String registerUser(@RequestParam String username, @RequestParam String password, Model model) {
    if (userService.userExists(username)) {
        model.addAttribute("message", "用户名已存在");
    } else {
        userService.addUser(username, password);
        model.addAttribute("message", "注册成功，请登录");
    }
    return "register";
}
```

5. **修改 `WebSecurityConfig` 以允许注册后的用户登录**：

由于我们使用 `InMemoryUserDetailsManager`，需要在注册时动态添加用户。为了简化，以下示例将跳过动态添加，而仅提供静态用户。

> **注意**：如果需要动态添加用户，请自定义 `UserDetailsService`，这超出了本教程的范围。

### 静态资源

1. **创建 CSS 文件**：

在 `src/main/resources/static/css` 目录下创建 `style.css`。

```css
body {
    font-family: Arial, sans-serif;
    margin: 50px;
}

h2 {
    color: #333;
}

form div {
    margin-bottom: 15px;
}

label {
    display: inline-block;
    width: 80px;
}

input {
    padding: 5px;
    width: 200px;
}

button {
    padding: 5px 10px;
}
```

## 运行项目（没做）

1. **构建项目**：

在 VS Code 的终端中，运行以下命令以构建项目：

```bash
./mvnw clean install
```

> **注**：Windows 用户可以使用 `mvnw.cmd`。

2. **运行应用程序**：

在 VS Code 的终端中，运行以下命令启动 Spring Boot 应用：

```bash
./mvnw spring-boot:run
```

应用启动后，您应该会看到类似如下的日志输出：

```
...
Tomcat started on port(s): 8080 (http) with context path ''
Started LoginappApplication in X.XXX seconds (JVM running for X.XXX)
```

3. **访问应用程序**：

打开浏览器，访问 [http://localhost:8080](http://localhost:8080)。

您应该会看到登录页面。使用预定义的用户进行登录：

- **用户名**: `user1`
- **密码**: `password1`

成功登录后，将重定向到欢迎页面。

## 总结

通过以上步骤，您已经在 VS Code 中使用 Spring Boot 框架搭建了一个简单的前后端登录页面，而无需依赖数据库。以下是关键要点：

- **项目初始化**：使用 Spring Initializr 快速生成项目骨架。
- **后端实现**：利用 Spring Security 配置内存中的用户存储，并处理身份验证。
- **前端实现**：使用 Thymeleaf 创建动态的登录和欢迎页面。
- **无数据库依赖**：通过 `InMemoryUserDetailsManager` 管理用户，无需配置数据库。

> **注意**：此示例适用于学习和演示目的。在生产环境中，建议使用加密的密码存储机制，并连接到持久化数据库以管理用户数据。此外，考虑使用更复杂的用户管理和权限控制策略以提高安全性。

希望本指南对您有所帮助！如果有任何问题或需要进一步的帮助，请随时提问。