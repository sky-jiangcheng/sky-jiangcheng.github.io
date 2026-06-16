---
title: Node.js 后端中间件开发实战
date: 2026-05-25 08:30:00
categories:
  - Node.js
tags:
  - Node.js
  - 后端开发
  - Express
  - 中间件
description: 讲解 Node.js 中间件的工作原理，以及如何开发认证、日志、限流、错误处理等常用中间件。
---

## 前言

中间件是 Node.js Web 框架(Express/Koa)的核心概念。理解中间件的工作原理和开发模式，是成为合格 Node.js 后端开发者的关键一步。

## 1. 中间件工作原理

Express 中间件的本质是一个函数，按注册顺序依次执行：

```javascript
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next(); // 传递给下一个中间件
});
```

Koa 中间件基于 async/await，采用洋葱模型：

```javascript
app.use(async (ctx, next) => {
  const start = Date.now();
  await next(); // 执行后续中间件
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
```

## 2. 认证中间件

JWT 认证中间件的实现：

```javascript
const jwt = require('jsonwebtoken');

function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ code: 401, message: '未提供认证令牌' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ code: 401, message: '令牌已过期' });
    }
    return res.status(401).json({ code: 401, message: '无效的令牌' });
  }
}

app.use('/api/admin', authMiddleware);
```

## 3. 请求日志中间件

使用 `morgan` 记录 HTTP 请求日志：

```javascript
const morgan = require('morgan');
const fs = require('fs');
const path = require('path');

const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'logs', 'access.log'),
  { flags: 'a' }
);

morgan.token('body', (req) => JSON.stringify(req.body));

app.use(morgan(':date[iso] :method :url :status :response-time ms - :body', {
  stream: accessLogStream
}));
```

## 4. 限流中间件

基于 Token Bucket 算法实现 API 限流：

```javascript
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    code: 429,
    message: '请求过于频繁，请稍后再试'
  }
});

app.use('/api', apiLimiter);
```

针对不同接口设置不同的限制：

```javascript
const authLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true
});

app.use('/api/login', authLimiter);
```

## 5. 统一错误处理中间件

Express 中通过四个参数的中间件处理错误：

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

function errorHandler(err, req, res, next) {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : '服务器内部错误';

  if (!err.isOperational) {
    console.error('非预期错误:', err);
  }

  res.status(statusCode).json({
    code: statusCode,
    message: message
  });
}

app.use(errorHandler);
```

## 6. CORS 中间件

处理跨域请求的中间件配置：

```javascript
const cors = require('cors');

const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('不允许的跨域请求'));
    }
  },
  credentials: true,
  maxAge: 86400
};

app.use(cors(corsOptions));
```

## 总结

中间件是 Node.js 后端开发的基石，合理组合认证、日志、限流和错误处理中间件可以构建一个健壮的后端服务。开发时注意中间件的执行顺序：安全相关(认证/限流/CORS)放在前面，业务逻辑在中间，错误处理在最后。
