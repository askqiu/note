根据你提供的项目目录结构，我们将为你的 Spring Boot 项目添加代码来实现连接数据库的登录功能。下面是代码的详细布局，适应于你当前的目录结构。

### 项目结构

```
src/
 └── main/
     ├── java/
     │   └── com/
     │       └── example/
     │           └── scanplatform/
     │               ├── LoginController.java
     │               ├── ScanPlatformApplication.java
     │               ├── WebSecurityConfig.java
     │               ├── entity/
     │               │   └── User.java
     │               └── repository/
     │                   └── UserRepository.java
     └── resources/
         ├── templates/
         │   └── login.html
         └── application.properties
```

### 1. `application.properties`

在 `src/main/resources/application.properties` 中添加数据库连接信息：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/your_database_name
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### 2. `entity/User.java`

在 `src/main/java/com/example/scanplatform/entity/` 下创建 `User.java` 文件：

```java
package com.example.scanplatform.entity;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String password;

    // Getters and setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

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

### 3. `repository/UserRepository.java`

在 `src/main/java/com/example/scanplatform/repository/` 下创建 `UserRepository.java` 文件：

```java
package com.example.scanplatform.repository;

import com.example.scanplatform.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```

### 4. `LoginController.java`

在 `src/main/java/com/example/scanplatform/` 下的 `LoginController.java` 文件中添加以下代码：

```java
package com.example.scanplatform;

import com.example.scanplatform.entity.User;
import com.example.scanplatform.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class LoginController {

    @Autowired
    private UserRepository userRepository;

    @GetMapping("/login")
    public String login() {
        return "login"; // 返回登录页面
    }

    @PostMapping("/login")
    public String loginUser(String username, String password, Model model) {
        User user = userRepository.findByUsername(username);
        if (user != null && user.getPassword().equals(password)) {
            return "home"; // 登录成功，跳转到主页（需创建）
        } else {
            model.addAttribute("error", "Invalid username or password");
            return "login"; // 登录失败，返回登录页面
        }
    }
}
```

### 5. `WebSecurityConfig.java`

在 `src/main/java/com/example/scanplatform/` 下创建 `WebSecurityConfig.java` 文件，以配置 Spring Security（如果需要）：

```java
package com.example.scanplatform;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable() // 关闭 CSRF 保护
            .authorizeRequests()
                .antMatchers("/login").permitAll() // 允许所有用户访问登录页面
                .anyRequest().authenticated(); // 其他请求需要认证
    }
}
```

### 6. `templates/login.html`

在 `src/main/resources/templates/` 下的 `login.html` 文件中添加以下代码：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="#" th:action="@{/login}" method="post">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required><br>
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required><br>
        <button type="submit">Login</button>
        <div th:if="${error}" style="color: red;">[[${error}]]</div>
    </form>
</body>
</html>
```

### 7. 数据库表

确保你的数据库中有一个用户表，结构可以类似于：

```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

### 启动应用

- 启动 Spring Boot 应用，访问 `http://localhost:8080/login` 进行登录。
- 根据输入的用户名和密码进行身份验证。

以上是基于你提供的目录结构的完整代码示例。如果有任何问题或需要进一步的调整，请告诉我！
要将数据库与您的 Spring Boot 应用程序连接并进行用户身份验证，您需要执行以下步骤：

### 1. 安装数据库

首先，确保您已经安装了数据库（例如 MySQL、PostgreSQL 或其他）。在本例中，我们将以 MySQL 为例。

### 2. 创建数据库

1. **登录 MySQL**：

   在终端或命令提示符中输入以下命令：

   ```bash
   mysql -u your_username -p
   ```

   替换 `your_username` 为您的 MySQL 用户名，系统会提示您输入密码。

2. **创建数据库**：

   在 MySQL 提示符下运行以下 SQL 命令以创建数据库：

   ```sql
   CREATE DATABASE your_database_name;
   ```

   替换 `your_database_name` 为您想要的数据库名称。

### 3. 创建用户表

在同一 MySQL 会话中，选择刚创建的数据库，并创建用户表：

```sql
USE your_database_name;

CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);
```

### 4. 插入测试用户

为了能够测试登录功能，您可以插入一个或多个测试用户。使用以下 SQL 命令插入用户数据：

```sql
INSERT INTO users (username, password) VALUES ('testuser', 'testpassword');
```

请注意，这里存储的是明文密码。在实际应用中，您应该对密码进行哈希处理。

### 5. 配置 `application.properties`

确保在 `src/main/resources/application.properties` 文件中配置了正确的数据库连接信息：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/your_database_name
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

### 6. 启动 Spring Boot 应用

1. 在 IDE 中启动您的 Spring Boot 应用，确保没有任何错误。
2. 打开浏览器，访问 `http://localhost:8080/login`，输入您在数据库中插入的用户名和密码（例如，`testuser` 和 `testpassword`）进行测试。

### 7. 处理密码（可选）

在实际应用中，您应该使用密码哈希算法（如 BCrypt）来存储密码，而不是明文存储。可以在用户注册时进行哈希处理，并在登录时进行比对。您可以使用 Spring Security 来简化这一过程。

### 示例：使用 BCrypt 对密码进行哈希处理

在 `UserService.java` 中，您可以添加如下代码来处理密码哈希：

```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    private BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

    public void registerUser(String username, String password) {
        User user = new User();
        user.setUsername(username);
        user.setPassword(passwordEncoder.encode(password)); // 对密码进行哈希处理
        userRepository.save(user);
    }
}
```

您还需要在 `pom.xml` 中添加 Spring Security 依赖，以使用 BCrypt 编码器：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 总结

- 创建数据库和用户表。
- 插入测试用户。
- 确保应用程序与数据库的连接配置正确。
- 启动应用程序并测试登录功能。

如果您在这个过程中遇到任何问题，或需要更多具体的指导，请告诉我！