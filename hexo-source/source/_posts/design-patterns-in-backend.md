---
title: 后端开发必备的设计模式实战
date: 2026-06-02 16:00:00
categories:
  - Java
tags:
  - Java
  - 设计模式
  - 后端开发
description: 结合实际业务场景，讲解策略模式、模板方法、责任链模式、观察者模式在后端开发中的典型应用。
---

## 前言

设计模式是软件开发中反复出现的问题的通用解决方案。在后端开发中，合理运用设计模式能让代码结构更清晰、扩展性更强。本文基于金融和能源行业的实际项目经验，分享四个最常用的设计模式。

## 1. 策略模式 - 支付路由

电商系统中，不同支付渠道的处理逻辑各不相同，策略模式可以将每种支付方式封装成独立的策略：

```java
// 策略接口
public interface PaymentStrategy {
    PaymentResult pay(PaymentRequest request);
    boolean supports(PaymentChannel channel);
}

// 微信支付
@Component
public class WechatPaymentStrategy implements PaymentStrategy {
    @Override
    public PaymentResult pay(PaymentRequest request) {
        // 调用微信支付 API
    }

    @Override
    public boolean supports(PaymentChannel channel) {
        return channel == PaymentChannel.WECHAT;
    }
}

// 支付上下文
@Component
public class PaymentContext {
    private final List<PaymentStrategy> strategies;

    public PaymentContext(List<PaymentStrategy> strategies) {
        this.strategies = strategies;
    }

    public PaymentResult executePayment(PaymentRequest request) {
        return strategies.stream()
                .filter(s -> s.supports(request.getChannel()))
                .findFirst()
                .orElseThrow(() -> new UnsupportedOperationException("不支持的支付渠道"))
                .pay(request);
    }
}
```

新增支付渠道只需添加一个新的策略类，无需修改现有代码。

## 2. 模板方法模式 - 数据导出

不同的报表导出(PDF/Excel/CSV)有相似的流程：查询数据、格式化、写入文件。模板方法定义了算法骨架，子类实现具体步骤：

```java
public abstract class ReportExporter {
    public final byte[] export(ReportQuery query) {
        List<ReportData> data = fetchData(query);
        validateData(data);
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        writeToStream(formatData(data), output);
        return output.toByteArray();
    }

    protected abstract List<ReportData> fetchData(ReportQuery query);
    protected abstract byte[] formatData(List<ReportData> data);

    protected void validateData(List<ReportData> data) {
        if (data == null || data.isEmpty()) {
            throw new BusinessException("导出数据为空");
        }
    }

    protected abstract void writeToStream(byte[] content, ByteArrayOutputStream output);
}
```

## 3. 责任链模式 - 请求校验

在 Controller 层中，经常需要对请求进行多层校验(参数校验、权限校验、限流校验等)，责任链模式可以灵活组合：

```java
public abstract class ValidationHandler {
    protected ValidationHandler next;

    public ValidationHandler setNext(ValidationHandler next) {
        this.next = next;
        return next;
    }

    public void validate(RequestContext context) {
        doValidate(context);
        if (next != null) {
            next.validate(context);
        }
    }

    protected abstract void doValidate(RequestContext context);
}
```

## 4. 观察者模式 - 事件驱动

使用 Spring 的事件机制解耦业务逻辑：

```java
// 定义事件
public class OrderCreatedEvent extends ApplicationEvent {
    private final Long orderId;
    public OrderCreatedEvent(Object source, Long orderId) {
        super(source);
        this.orderId = orderId;
    }
}

// 发布事件
@Service
public class OrderService {
    @Autowired
    private ApplicationEventPublisher publisher;

    @Transactional
    public void createOrder(OrderDTO dto) {
        Order order = saveOrder(dto);
        publisher.publishEvent(new OrderCreatedEvent(this, order.getId()));
    }
}

// 监听者: 发送通知
@Component
public class NotificationListener {
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        // 发送短信/邮件通知
    }
}
```

## 总结

后端开发中，设计模式的核心价值在于应对变化。策略模式处理算法族的变化，模板方法处理流程步骤的变化，责任链模式处理处理器的组合变化，观察者模式处理事件的响应变化。在实际项目中，不要为了模式而模式，只有当代码出现"坏味道"时才考虑引入。
