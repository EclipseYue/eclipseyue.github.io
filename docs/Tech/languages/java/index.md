# Java 学习路线

这里记录 Java 基础、Java Web 和项目实践。当前目标是通过一个完整业务项目，把语言、框架、数据库和部署链路串起来。

## Java 基础

需要补齐的基础能力：

- 语法：类、接口、继承、多态、泛型、异常。
- 集合：List、Set、Map、Queue，以及常见实现类。
- 并发：线程、线程池、锁、并发容器。
- JVM：内存区域、类加载、垃圾回收。
- IO：BIO、NIO、序列化。

## Java Web

- HTTP 请求与响应。
- Servlet / Filter / Interceptor。
- Spring、Spring MVC、Spring Boot。
- 参数校验、异常处理、日志。
- RESTful API 设计。

## 项目方向：苍穹外卖

想通过“苍穹外卖”项目补齐后端开发链路。前置知识包括 Java SE、Java Web、Spring Boot 和数据库基础。

### 开发流程

1. 需求分析和产品原型。
2. UI 设计、数据库设计、接口设计。
3. 编码和单元测试。
4. 联调、测试和 bug 修复。
5. 上线、部署和维护。

### 技术栈

- 前端：Vue。
- 后端：Spring Boot。
- 数据库：MySQL。
- 缓存：Redis。
- 消息队列：RabbitMQ。
- 网关/部署：Nginx、Docker。
- 接口文档：Swagger / Knife4j。
- 接口调试：Apifox。

### 系统链路

```text
浏览器 / 客户端
    -> Nginx
    -> Spring Boot
    -> MySQL / Redis / RabbitMQ
```

Nginx 主要用于静态资源缓存、反向代理和负载均衡。

## 后续记录

- 项目环境搭建。
- 数据库表设计。
- 登录鉴权流程。
- 订单流程。
- Redis 缓存设计。
- Docker 部署过程。

