# SpringBoot 原理学习笔记

## 一、SpringBoot 原理总览

SpringBoot 原理这一块就两大核心：

| 原理 | 解决什么问题 |
|------|-------------|
| 起步依赖 | 依赖怎么来——一次引入一整套相关依赖 |
| 自动配置 | Bean 怎么自动装好——不用手动配置就能用 |

这两个是一对：起步依赖把依赖（含自动配置包）引进来，自动配置才能生效。

---

## 二、起步依赖原理

### 1. 问题引入

pom.xml 里只写一行：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

项目里却能用 SpringMVC、Tomcat、Jackson(JSON)、日志……一堆东西。为什么？

### 2. 核心知识点一：Maven 依赖传递

```text
你的项目
  └─ 依赖 A
        └─ A 依赖 B
              └─ B 依赖 C
```

结果：只写了 A，B、C 也被自动引入——这叫**依赖传递**（间接依赖），而且是**多层嵌套**传递。

### 3. 核心知识点二：starter 的本质

`spring-boot-starter-web` 本身几乎没有代码，它就是一个"依赖清单" pom，里面声明了一堆 web 开发要用的依赖：

```text
spring-boot-starter-web : 3.2.6
  ├─ spring-boot-starter : 3.2.6          ← 核心 starter
  │    ├─ spring-boot
  │    ├─ spring-boot-autoconfigure       ← 自动配置（144/145 主角）
  │    └─ spring-boot-starter-logging ─→ slf4j（日志）
  ├─ spring-boot-starter-json : 3.2.6     ← JSON（Jackson）
  ├─ spring-boot-starter-tomcat : 3.2.6   ← 内嵌 Tomcat
  │    ├─ tomcat-embed-core
  │    ├─ tomcat-embed-el
  │    └─ tomcat-embed-websocket
  ├─ spring-web : 6.1.8
  └─ spring-webmvc : 6.1.8
```

引入 starter → 靠依赖传递，这些全自动进来。

### 4. starter 命名规律

| 命名格式 | 谁提供的 | 例子 |
|---------|---------|------|
| `spring-boot-starter-*` | **官方**（前缀在前） | `spring-boot-starter-web`、`-aop`、`-test` |
| `*-spring-boot-starter` | **第三方**（名字在前） | `mybatis-spring-boot-starter`、`pagehelper-spring-boot-starter` |

记忆点：

> 官方的：`spring-boot-starter-` 打头
> 第三方的：自己名字打头，`-spring-boot-starter` 结尾

### 5. 关键认知

1. **嵌套传递，不止一层**：`starter-web → starter-tomcat → tomcat-embed-core`，层层传下来，不是平铺
2. **起步依赖和自动配置的关联**：起步依赖把 `spring-boot-autoconfigure` 也传进来了，自动配置才能生效
3. **版本号大多不用写**：父工程 `spring-boot-starter-parent` 统一管理了版本

### 6. 一句话总结

> 起步依赖的本质 = 一个 pom，靠 Maven 依赖传递（含多层嵌套），一次引入一整套相关依赖 + 自动配置包。

---

## 三、自动配置原理

### 1. 什么是自动配置

> SpringBoot 项目启动后，一些**配置类、bean 对象**就自动存入 IOC 容器，不需要手动声明，从而简化开发。

**对比理解：**

没有自动配置时，要用一个第三方类（如 Gson）得自己写：

```java
@Configuration
public class GsonConfig {
    @Bean
    public Gson gson() { return new Gson(); }
}
```

有自动配置 → 这段不用写，引入依赖即可 `@Autowired` 使用。

### 2. 自动配置要解决的核心矛盾

| 现象 | 原因 |
|------|------|
| jar 包在了（能 `new XXX()`） | Maven 依赖传递做到了（起步依赖） |
| 但 `@Autowired XXX` 报错 | 没人把 XXX 注册成 bean |

> 起步依赖只负责"把 jar 弄进来"，**不负责**"把 bean 注册进 IOC 容器"。后者才是自动配置要解决的。

**根因：** `@SpringBootApplication` 默认只扫描**启动类所在包及其子包**。第三方包（如 `com.example`）不在 `com.itheima` 下，扫不到，bean 进不了容器。

### 3. 实现方案一：@ComponentScan

```java
@ComponentScan(basePackages = {"com.example", "com.itheima"})
@SpringBootApplication
public class SpringbootWebConfigApplication {
```

手动把第三方包名加入扫描范围。

**问题：**

- 麻烦：每用一个第三方包就得手动加包名
- 不通用：得知道人家内部包名叫什么
- 不优雅：框架级的东西不该让使用者改启动类

### 4. 实现方案二：@Import

思路：不靠"扫描包"，而是**直接点名**要哪些类，加载进 IOC 容器。

`@Import` 四种导入形式：

| # | 形式 | 写法 | 效果 |
|---|------|------|------|
| 1 | 普通类 | `@Import(TokenParser.class)` | 直接注册成一个 bean |
| 2 | 配置类 | `@Import(HeaderConfig.class)` | 配置类里所有 `@Bean` 一起进容器 |
| 3 | ImportSelector 实现类 | `@Import(MyImportSelector.class)` | 代码动态返回要导入的类名 |
| 4 | `@EnableXxx` 注解 | `@EnableHeaderConfig` | 封装 `@Import`，方便、优雅 ✅ |

第 4 种本质是把前三种包一层注解，是实际框架最终采用的形式（详见第 6 节）。

**ImportSelector 实现：**

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.example.HeaderConfig"};
    }
}
```

关键：返回的是**类的全限定名字符串**（不是 `.class`），所以能用代码动态拼，最灵活。

### 5. 完整演进

| 方案 | 写法 | 问题 |
|------|------|------|
| 方案一 | `@ComponentScan` 指定包 | 要知道内部包名，麻烦不通用 |
| 方案二·普通类 | `@Import(A.class)` | 一次一个，多了啰嗦 |
| 方案二·配置类 | `@Import(Config.class)` | 一次一批，但还得手写每个类 |
| 方案二·ImportSelector | `@Import(MySelector.class)` | 代码动态返回类名，最灵活 |
| 方案二·@EnableXxx | `@EnableXxx` | 封装 @Import，使用者无感，最优雅 ✅ |

### 5.1 本节核心问答（考点）

**Q1：为什么第三方依赖里用 `@Component` 及衍生注解声明的 bean 不生效？**

> 因为基于 `@Component`（及 `@Service`/`@Repository`/`@Controller`）声明的 bean 要生效，**必须被组件扫描注解扫描到**。而 `@SpringBootApplication` 默认只扫描启动类所在包及子包，第三方包不在范围内。

**Q2：有哪些方案能让它生效？**

> a. `@ComponentScan` 扫描指定的包
> b. `@Import` 把类导入 IOC 容器（四种方式：普通类、配置类、ImportSelector 实现类、`@EnableXxx`）

一句话锁住整节：

> bean 不生效 = 没被扫到；解决 = 要么扩大扫描范围（`@ComponentScan`），要么直接点名导入（`@Import` 四形式）。

### 6. 用自定义注解封装 @Import

方案二还有个缺点：`@Import(MyImportSelector.class)` **暴露了第三方包的内部类名**，使用者还得知道它存在。

解决：第三方提供一个语义化的 `@EnableXxx` 注解，把 `@Import` 藏进去。

```java
@Retention(RetentionPolicy.RUNTIME)   // 运行时保留，Spring 才能反射读到
@Target(ElementType.TYPE)             // 只能加在类上
@Import(MyImportSelector.class)       // 核心：把 @Import 封装进来
public @interface EnableHeaderConfig {
}
```

**封装前后对比：**

```java
// 封装前（使用者得知道 MyImportSelector）
@Import(MyImportSelector.class)
@SpringBootApplication
public class XxxApplication {

// 封装后（使用者只写一行语义化注解）
@EnableHeaderConfig
@SpringBootApplication
public class XxxApplication {
```

好处：内部实现（`MyImportSelector`、`HeaderConfig`）全藏起来，第三方改内部，使用者无感。

**这就是真实框架的套路，早见过：**

| 框架 | `@EnableXxx` 注解 |
|------|------------------|
| 定时任务 | `@EnableScheduling` |
| 缓存 | `@EnableCaching` |
| 异步 | `@EnableAsync` |
| 事务 | `@EnableTransactionManagement` |

点进去看，内部全是 `@Import(某个 Selector/Configuration.class)`。

### 7. 完整链路闭环

```text
第三方写 @EnableHeaderConfig（内含 @Import(MyImportSelector.class)）
        ↓
使用者在启动类加一行 @EnableHeaderConfig
        ↓
@Import 触发 MyImportSelector.selectImports()
        ↓
返回要装配的类名 → Spring 注册成 bean
        ↓
使用者 @Autowired 直接能用
```

### 8. 和真实 SpringBoot 的连接

> SpringBoot 自动配置底层用的就是 **ImportSelector**。第三方 starter 内部写好 selector，返回自己要装配的类名，所以引入 starter 后**啥都不用写**，bean 自动进容器。

最后一步（145 源码跟踪解决）：真实 SpringBoot 连 `@EnableXxx` 都不用写——`@SpringBootApplication` 里自带 `@EnableAutoConfiguration`，配合 `META-INF/.../AutoConfiguration.imports` 文件自动找到所有 starter 的配置类，做到使用者完全无感。

---

> 创建时间：2026-05-18
