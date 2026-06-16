---
title: SpringBoot 微服务快速入门指南
date: 2026-06-12 10:00:00
categories:
  - SpringBoot
tags:
  - SpringBoot
  - 微服务
  - Java
  - 后端开发
description: 从零开始构建一个SpringBoot微服务项目，涵盖项目初始化、RESTful API设计、数据持久化、异常处理与单元测试等核心内容。
---

## 前言

SpringBoot 是目前 Java 生态系统中最流行的微服务开发框架。它通过自动配置和起步依赖大幅简化了 Spring 应用的搭建和开发过程。本文将通过一个实际的用户管理微服务示例，带你快速掌握 SpringBoot 的核心开发流程。

## 1. 项目初始化

使用 Spring Initializr 创建项目，选择以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

## 2. 设计 RESTful API

遵循 RESTful 设计原则，用户管理模块的 API 设计如下：

| 方法   | 路径            | 说明         |
| ------ | --------------- | ------------ |
| GET    | /api/users      | 获取用户列表 |
| GET    | /api/users/{id} | 获取单个用户 |
| POST   | /api/users      | 创建用户     |
| PUT    | /api/users/{id} | 更新用户     |
| DELETE | /api/users/{id} | 删除用户     |

## 3. 数据模型与持久化

定义 User 实体类，使用 JPA 注解完成对象关系映射：

```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    @NotBlank(message = "用户名不能为空")
    private String username;

    @Column(nullable = false, unique = true)
    @Email(message = "邮箱格式不正确")
    private String email;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
```

## 4. 统一响应格式

定义统一的 API 响应结构，避免每个接口返回格式不一致：

```java
@Data
@AllArgsConstructor
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "success", data);
    }

    public static <T> ApiResponse<T> error(int code, String message) {
        return new ApiResponse<>(code, message, null);
    }
}
```

## 5. 全局异常处理

使用 `@ControllerAdvice` 实现统一的异常处理，让代码更简洁：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiResponse<Void> handleNotFound(ResourceNotFoundException ex) {
        return ApiResponse.error(404, ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors()
                .stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining(", "));
        return ApiResponse.error(400, message);
    }
}
```

## 6. 单元测试

SpringBoot 提供了完善的测试支持，使用 `@WebMvcTest` 测试 Controller 层：

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUserList() throws Exception {
        when(userService.findAll()).thenReturn(List.of(new User(1L, "test", "test@example.com", null)));

        mockMvc.perform(get("/api/users"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data[0].username").value("test"));
    }
}
```

## 总结

通过本文的实践，我们完成了一个 SpringBoot 微服务的基础搭建，涵盖了项目初始化、API 设计、数据持久化、异常处理和测试等关键环节。这些都是实际开发中必不可少的基础设施，掌握之后可以快速投入到业务开发中。
