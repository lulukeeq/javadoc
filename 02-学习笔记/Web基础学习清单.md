# Web 基础学习清单

## 学习目录

### 一、SpringBoot Web 入门

### 二、HTTP 协议

### 三、SpringBoot Web 案例

### 四、分层解耦

- 三层结构
- 分层解耦
- IOC & DI 入门
- IOC 详解
- DI 详解

### 五、Web 后端开发总结

---

## 学习笔记

### 四、分层解耦

#### 4.1 三层结构

**三层架构：**

| 层             | 名称                                      | 说明                                 |
| -------------- | ----------------------------------------- | ------------------------------------ |
| **Controller** | 控制层                                    | 接受请求、处理请求、响应数据         |
| **Service**    | 业务逻辑层                                | 处理具体业务逻辑                     |
| **Dao**        | 数据访问层（Data Access Object / 持久层） | 负责数据访问操作，对数据进行增删改查 |

> 💡 三层架构的好处：遵守**单一职责原则**，便于复用、后期维护。

#### 4.2 分层解耦

**控制反转（IOC）：** Inversion Of Control，简称 IOC。对象的创建控制权由程序本身转移到外部（容器），这种思想称为控制反转。

**依赖注入（DI）：** Dependency Injection，简称 DI。容器为应用程序提供运行时所依赖的资源，称之为依赖注入。

**Bean 对象：** IOC 容器中创建、管理的对象，称之为 Bean。

#### 4.3 IOC & DI 入门

1. 将 Dao 及 Service 层的实现类，交给 IOC 容器管理
2. 为 Controller 及 Service 注入运行时所依赖的对象

**常用注解：**

| 注解              | 说明                                                                |
| ----------------- | ------------------------------------------------------------------- |
| `@Component`      | 将类产生的对象交给 IOC 容器处理（不属于三层架构时使用，如配置类等） |
| `@Autowired`      | 运行程序时，会自动查询该类型的 Bean 对象，并赋值给该成员变量，即 DI |
| `@RestController` | 等同于 `@Controller` + `@ResponseBody`                              |

#### 4.4 IOC 详解

**IOC 注解（衍生注解）：**

| 注解          | 说明                                          |
| ------------- | --------------------------------------------- |
| `@Component`  | 通用注解，不属于三层架构时使用（如配置类等）  |
| `@Controller` | 用于控制层类                                  |
| `@Service`    | 用于业务逻辑层                                |
| `@Repository` | 用于数据访问层（由于 MyBatis 整合，用的较少） |

> 💡 Bean 的名字默认是类首字母小写。

> 💡 可以在 IOC 注解中提供 `value` 属性来指定名字，通常情况下不需要指定名字。

> 💡 可以通过 Actuator 来显示所有的 Bean。

**组件扫描：**

> 使用 IOC 注解想要生效，还需要被组件扫描注解 `@ComponentScan` 扫描。该注解虽然没有显式配置，但实际上已经包含在启动类声明注解 `@SpringBootApplication` 中。默认扫描的范围是**启动类所在包及其子包**。

#### 4.5 DI 详解

**`@Autowired` 进行依赖注入的三种方式：**

**1. 属性注入：**

```java
@RestController
public class UserController {
    @Autowired
    private UserService userService;
}
```

**2. 构造函数注入：**

```java
@RestController
public class UserController {
    private final UserService userService;

    @Autowired // 如果只有一个构造函数，@Autowired 可省略
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

**3. Setter 注入：**

```java
@RestController
public class UserController {
    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

**三种注入方式对比：**

| 方式 | 优点 | 缺点 |
|------|------|------|
| 属性注入 | 简洁、方便、快速 | 隐藏了类之间的依赖关系，可能会破坏类的封装性 |
| 构造函数注入 | 能清晰的看到类的依赖关系，提高了代码的安全性（final） | 代码繁琐，如果构造参数过多，可能导致构造函数臃肿 |
| Setter 注入 | 保持了类的封装性，依赖关系更清晰 | 需要额外编写 setter 方法，增加了代码量 |

> 💡 **如何选择：** 企业中一般选择**属性注入**；Spring 官方推荐**构造函数注入**方式；Setter 注入不推荐。

> 💡 `@Autowired` 注解，默认是按照**类型**进行注入的。如果存在多个相同类型的 Bean，将会报错误。

**解决多个同类型 Bean 的方案：**

| 方案 | 说明 |
|------|------|
| `@Primary` | 配合 IOC 注解使用，标记优先注入的 Bean |
| `@Qualifier` | 配合 `@Autowired` 使用，指定要注入的 Bean 名称 |
| `@Resource` | 通过传入名称来确定使用哪个 Bean（JavaEE 规范） |

> 💡 `@Autowired` 是 Spring 框架提供的，默认按照**类型**注入；`@Resource` 是 JavaEE 规范的，默认按照**名称**注入。

---

### 五、Web 后端开发总结

#### 5.1 一次请求的完整链路

Web 后端开发可以先抓住一条主线：

```text
浏览器
-> Filter 过滤器
-> Interceptor 拦截器
-> Controller
-> Service
-> Dao / Mapper
-> MySQL
```

也就是：

> 请求进来，先经过统一拦截处理，再进入三层架构，最后访问数据库并返回响应。

各层职责：

| 位置 | 主要作用 |
|------|----------|
| 浏览器 | 发起请求，接收响应 |
| Filter 过滤器 | 在进入 SpringMVC 之前做统一过滤处理 |
| Interceptor 拦截器 | 在进入 Controller 前后做拦截处理 |
| Controller | 接收请求参数，调用业务层，返回响应数据 |
| Service | 处理业务逻辑，控制事务 |
| Dao / Mapper | 操作数据库，执行 SQL |
| MySQL | 保存真实业务数据 |

最短记法：

```text
请求 -> 拦截处理 -> Controller -> Service -> Mapper -> 数据库 -> 响应
```

#### 5.2 三层架构的分工

三层架构是 Web 后端开发的核心骨架：

| 层 | 职责 |
|----|------|
| Controller | 接收请求、解析参数、返回结果 |
| Service | 编排业务逻辑、处理事务 |
| Mapper / Dao | 访问数据库 |

开发时要避免把所有代码都写在 Controller 里。

更推荐的分工是：

```text
Controller 不写复杂业务
Service 承担业务逻辑
Mapper 只负责数据访问
```

#### 5.3 关键技术点放在哪里

##### JavaWeb

主要提供底层 Web 能力：

- `Filter`
- `Cookie`
- `Session`

其中 `Filter` 常用于登录校验、编码处理、统一过滤等场景。

##### SpringMVC

主要负责 Web 层：

- 接收请求
- 响应数据
- 拦截器
- 全局异常处理

可以理解为：

> SpringMVC 负责“请求进来”和“响应出去”。

##### Spring Framework

主要提供核心容器与通用增强能力：

- IOC
- DI
- AOP
- 事务管理

其中：

| 技术 | 作用 |
|------|------|
| IOC | 对象交给 Spring 容器管理 |
| DI | 由容器自动注入依赖对象 |
| AOP | 抽取公共逻辑，统一增强 |
| 事务管理 | 保证一组数据库操作要么都成功，要么都失败 |

##### MyBatis

主要负责持久层：

- Mapper 接口
- SQL 映射
- 动态 SQL
- 数据库增删改查

一句话：

> MyBatis 负责 Java 程序和数据库之间的数据访问。

##### SpringBoot

SpringBoot 不是替代 SpringMVC、Spring、MyBatis，而是把它们整合起来。

它的作用是：

- 简化配置
- 自动配置常用组件
- 快速启动项目
- 统一整合后端技术栈

可以这样理解：

```text
SpringBoot 是整合平台
SpringMVC 管 Web 请求响应
Spring Framework 管对象、增强、事务
MyBatis 管数据库
JavaWeb 提供底层 Web 能力
```

#### 5.4 项目能力回顾

这一阶段的 Web 项目，已经覆盖了常见后端开发能力：

1. 接口开发
2. 统一响应结果
3. 分页查询
4. 条件查询
5. 新增、修改、删除
6. 动态 SQL
7. 多表关系与多表查询
8. 文件上传
9. 登录认证
10. Filter / Interceptor 登录校验
11. 全局异常处理
12. 事务管理
13. AOP 公共逻辑抽取
14. SpringBoot 原理
15. Maven 多模块、继承、聚合、私服

这些内容组合起来，就是一个 Java Web 后端项目的基础闭环。

#### 5.5 阶段总结

Web 后端开发不是单纯“写接口”，而是在完成一整条请求处理链路：

```text
接收请求
-> 参数处理
-> 登录校验 / 权限校验
-> 业务逻辑
-> 数据库操作
-> 异常兜底
-> 统一响应
```

这一阶段最该记住的两句话：

```text
Web 后端开发主线 = 请求 -> 业务 -> 数据库 -> 响应
```

```text
技术归属 = JavaWeb 打底，SpringMVC 管 Web，Spring 管容器和增强，MyBatis 管数据库，SpringBoot 负责整合。
```

下一步如果跳过前端工程化主线，非前端方向可以继续进入：

```text
Linux -> 项目部署 -> Docker
```

---

## 学习进度追踪

- [ ] SpringBoot Web 入门
- [ ] HTTP 协议
- [ ] SpringBoot Web 案例
- [ ] 分层解耦
  - [x] 三层结构
  - [ ] 分层解耦
  - [ ] IOC & DI 入门
  - [ ] IOC 详解
  - [ ] DI 详解
- [x] Web 后端开发总结

---

> 创建时间：2026-04-21
> 最后更新：2026-05-25
> 学习进度：已完成阶段总结
