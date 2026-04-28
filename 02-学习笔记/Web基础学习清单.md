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

---

> 创建时间：2026-04-21
> 最后更新：2026-04-22
> 学习进度：1 / 8
