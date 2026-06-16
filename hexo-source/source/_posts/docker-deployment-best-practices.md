---
title: Docker 容器化部署最佳实践
date: 2026-06-08 14:30:00
categories:
  - Docker
tags:
  - Docker
  - 部署
  - DevOps
  - 运维
description: 详细介绍 Docker 在生产环境中的最佳实践，包括镜像优化、多阶段构建、健康检查、日志管理和安全加固等关键技巧。
---

## 前言

Docker 已成为现代应用部署的标准方式，但简单地"能跑起来"和"生产级部署"之间还有很长的距离。本文总结了在金融和能源行业后端开发中的 Docker 实践经验。

## 1. 镜像瘦身

### 选择合适的基础镜像

使用 `alpine` 版本替代完整发行版，镜像体积可以从 600MB 降到 100MB 以内：

```dockerfile
# 不推荐 - 完整 Ubuntu 镜像
FROM openjdk:11
# 镜像大小约 600MB

# 推荐 - Alpine 精简版
FROM openjdk:11-jre-slim
# 镜像大小约 200MB
```

### 多阶段构建

将构建环境和运行环境分离，最终镜像只包含运行时需要的文件：

```dockerfile
# 阶段1: 构建
FROM maven:3.8-openjdk-11 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# 阶段2: 运行
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## 2. 健康检查

为容器配置健康检查，让 Docker 和编排工具能感知服务状态：

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

## 3. 日志管理

避免将日志写入容器文件系统，将日志输出到标准输出：

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

Docker 日志驱动配置：

```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## 4. 资源限制

在生产环境中必须设置资源限制，防止单个容器耗尽宿主机资源：

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
```

## 5. 安全最佳实践

以非 root 用户运行容器：

```dockerfile
FROM openjdk:11-jre-slim
RUN groupadd -r app && useradd -r -g app app
USER app
COPY --chown=app:app target/*.jar app.jar
```

## 6. Docker Compose 多服务编排

典型的 SpringBoot + MySQL + Redis 编排示例：

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: app_db
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 5s
      retries: 10

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/app_db
      SPRING_REDIS_HOST: redis

volumes:
  mysql_data:
  redis_data:
```

## 总结

Docker 生产化部署的核心思路是轻量化、可观测、可限制、可编排。多阶段构建减少镜像体积，健康检查保证服务可用性，资源限制避免相互干扰，日志驱动便于集中采集。
