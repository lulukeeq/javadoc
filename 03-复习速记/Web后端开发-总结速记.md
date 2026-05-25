# Web 后端开发 - 总结速记

## 1. 一次请求的完整链路

```text
浏览器
-> Filter 过滤器
-> Interceptor 拦截器
-> Controller
-> Service
-> Mapper / Dao
-> MySQL
```

最短记法：

```text
请求 -> 拦截处理 -> Controller -> Service -> Mapper -> 数据库 -> 响应
```

---

## 2. 各层职责

| 位置 | 作用 |
|------|------|
| Filter | 进入 SpringMVC 前做统一过滤 |
| Interceptor | 进入 Controller 前后做拦截 |
| Controller | 接请求、收参数、还响应 |
| Service | 写业务逻辑、控制事务 |
| Mapper / Dao | 操作数据库 |
| MySQL | 保存真实数据 |

核心分工：

```text
Controller 不写复杂业务
Service 承担业务逻辑
Mapper 只负责数据访问
```

---

## 3. 技术归属

| 技术体系 | 主要负责 |
|----------|----------|
| JavaWeb | Filter、Cookie、Session |
| SpringMVC | 请求接收、响应数据、拦截器、全局异常处理 |
| Spring Framework | IOC、DI、AOP、事务管理 |
| MyBatis | Mapper、SQL、数据库访问 |
| SpringBoot | 整合技术栈、自动配置、快速启动 |

最短记法：

```text
JavaWeb 打底
SpringMVC 管 Web
Spring 管容器、增强、事务
MyBatis 管数据库
SpringBoot 管整合和启动
```

---

## 4. 常见项目能力

这一阶段已经覆盖：

1. 接口开发
2. 统一响应结果
3. 分页查询、条件查询
4. 新增、修改、删除
5. 动态 SQL
6. 多表关系、多表查询
7. 文件上传
8. 登录认证
9. Filter / Interceptor 登录校验
10. 全局异常处理
11. 事务管理
12. AOP
13. SpringBoot 原理
14. Maven 高级

---

## 5. 最终主线

Web 后端开发不是只写接口，而是在完成一条完整链路：

```text
接收请求
-> 参数处理
-> 登录校验 / 权限校验
-> 业务逻辑
-> 数据库操作
-> 异常兜底
-> 统一响应
```

一句话：

```text
Web 后端开发主线 = 请求 -> 业务 -> 数据库 -> 响应
```

---

> 创建时间：2026-05-25
> 对应主笔记：`../02-学习笔记/Web基础学习清单.md`
