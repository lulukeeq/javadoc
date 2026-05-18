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

## 四、自动配置源码跟踪

### 1. @SpringBootApplication 三大组成

`@SpringBootApplication` 标在启动类上，是 SpringBoot **最最最重要**的注解，本质是三个注解的组合：

| 组成注解 | 作用 |
|---------|------|
| `@SpringBootConfiguration` | 等价于 `@Configuration`，声明启动类本身也是一个配置类 |
| `@ComponentScan` | 组件扫描，默认扫描启动类所在包及其子包 |
| `@EnableAutoConfiguration` | **自动配置的总开关**，核心注解 |

> 接上一章：`@ComponentScan` 解释了为什么自己的包能被扫到（也解释了第三方包扫不到）；`@EnableAutoConfiguration` 才是解决第三方 bean 自动装配的那一环。

### 2. @EnableAutoConfiguration 拆解

点进 `@EnableAutoConfiguration`，里面又是两个：

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)   // ← 又见 @Import + ImportSelector
public @interface EnableAutoConfiguration {
```

**和第三章完全呼应**：144 自己写的 `@EnableHeaderConfig` 内部是 `@Import(MyImportSelector.class)`；SpringBoot 官方就是同一套路，只是 Selector 换成了官方的 `AutoConfigurationImportSelector`。

### 3. AutoConfigurationImportSelector 干了什么

它实现了 `ImportSelector`，重写 `selectImports()` 返回 `String[]`（一堆要装配的类的全限定名）。

这些类名不是写死的，而是**读配置文件**得到的：

```text
spring-boot-autoconfigure-3.1.3.jar
  └─ META-INF/spring/
       └─ org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

文件内容就是一长串自动配置类的全限定名，例如：

```text
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration
org.springframework.boot.autoconfigure.jpa.JpaRepositoriesAutoConfiguration
...（几十上百个 XxxAutoConfiguration）
```

`selectImports()` 把这个文件里所有类名读出来返回 → Spring 把这些 `XxxAutoConfiguration` 配置类全部加载，配置类里的 `@Bean` 就进了 IOC 容器。

### 4. 版本差异（易错点）

| SpringBoot 版本 | 自动配置类清单文件 |
|----------------|------------------|
| 2.7.0 **及以后** | `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` |
| 2.7.0 **以前** | `META-INF/spring.factories` |

> 面试常问"自动配置类从哪来"——新版答 `.imports` 文件，老版答 `spring.factories`，最好两个都提。

### 5. @Conditional 按需装配

> ⚠️ **关键误区（课程红字强调）**
> `.imports` 文件里那上百个 `XxxAutoConfiguration` 会**全部注册为 IOC 容器的 bean 吗？**
> **NO！** `selectImports()` 只是把它们当成**候选清单**全读进来，并不代表全部生效。

清单里有上百个 `XxxAutoConfiguration`，但不可能全生效（没引 Redis 依赖就配 Redis bean 会报错）。靠 `@Conditional` 系列注解**按条件筛选**，只有满足条件的那几个才真正装配：

| 条件注解 | 生效条件 |
|---------|---------|
| `@ConditionalOnClass` | classpath 下存在某个类（即引了对应依赖）才生效 |
| `@ConditionalOnMissingBean` | 容器里没有该类型 bean 时才生效（**给用户留覆盖余地**） |
| `@ConditionalOnProperty` | 配置文件里某属性满足条件才生效 |

例（`@ConditionalOnMissingBean` 的意义）：

```java
@Bean
@ConditionalOnMissingBean      // 你没自己配 Gson，它才帮你配；你配了就用你的
public Gson gson(GsonBuilder gsonBuilder) {
    return gsonBuilder.create();
}
```

> 所以自动配置不是"全装"，而是"**有依赖才装 + 你没配它才配**"，既不报错也不覆盖用户。

### 6. 完整闭环（背这一段）

```text
@SpringBootApplication
  └─ @EnableAutoConfiguration                         自动配置总开关
       └─ @Import(AutoConfigurationImportSelector)    官方 ImportSelector
            └─ selectImports() 读 jar 包里的
               META-INF/spring/...AutoConfiguration.imports   （旧版：spring.factories）
                 └─ 拿到上百个 XxxAutoConfiguration 类名
                      └─ 逐个 @Conditional 判断 → 满足条件的才真正装配
                           └─ 配置类里的 @Bean 进 IOC → 用户 @Autowired 直接用
```

**一句话总结：**

> 引入 starter → `@EnableAutoConfiguration` 通过 `AutoConfigurationImportSelector` 读 `.imports` 文件拿到所有候选自动配置类 → 用 `@Conditional` 按需筛选 → 满足条件的配置类把 bean 装进容器，使用者完全无感。

---

## 五、@Conditional 条件装配

### 1. 定义

| 项 | 内容 |
|----|------|
| 作用 | 按一定**条件判断**，满足条件才把对应 bean 注册到 IOC 容器 |
| 位置 | 方法、类（标方法上控制单个 `@Bean`；标类上控制整个配置类） |
| 本质 | `@Conditional` 是**父注解**，派生出一大批 `@ConditionalOnXxx` 子注解 |

> 第四章说"自动配置类不是全装，靠 `@Conditional` 筛"——本章就是把这个筛选机制讲透。

### 2. 派生子注解一览（部分）

`@ConditionalOnBean` / `OnClass` / `OnMissingBean` / `OnMissingClass` / `OnProperty` / `OnExpression` / `OnResource` / `OnJava` / `OnWebApplication` / `OnNotWebApplication` / `OnSingleCandidate` …

**最常用三个（重点记）：**

| 子注解 | 生效条件 |
|--------|---------|
| `@ConditionalOnClass` | 环境中**存在**对应字节码文件（引了对应依赖），才注册 |
| `@ConditionalOnMissingBean` | 环境中**没有**对应的 bean（按类型 或 名称），才注册 |
| `@ConditionalOnProperty` | 配置文件中有对应**属性和值**，才注册 |

### 3. 三个实操演示（同一个 HeaderParser bean）

```java
@Configuration
public class HeaderConfig {

    @Bean
    @ConditionalOnClass(name = "io.jsonwebtoken.Jwts")
    // 环境中有 io.jsonwebtoken.Jwts 这个类（引了 jjwt 依赖）才创建 bean
    public HeaderParser headerParser() {
        return new HeaderParser();
    }
}
```

> 细节：参数用 `name = "全限定名字符串"` 而不是 `value = Jwts.class`。
> 因为如果直接写 `.class`，类不存在编译期就报错；用字符串则**延迟到运行期反射判断**，类不在也不报错——这正是自动配置能引用一堆可能没引入的类却不崩的原因。

```java
    @Bean
    @ConditionalOnMissingBean
    // 容器中没有 HeaderParser 类型的 bean 才创建
    public HeaderParser headerParser() {
        return new HeaderParser();
    }
```

> 不写参数时，按**方法返回值类型**判断（这里即 `HeaderParser`）。也可显式指定类型或名称。
> 用途：给自动配置兜底——用户自己配了就用用户的，没配框架才补默认（呼应第四章 Gson 的例子）。

```java
    @Bean
    @ConditionalOnProperty(name = "myname", havingValue = "itheima")
    // application.yml 中 myname 的值等于 itheima 才创建
    public HeaderParser headerParser() {
        return new HeaderParser();
    }
```

> 常用于"配置开关"：通过 `prefix`（前缀）+ `name`（属性名）+ `havingValue`（期望值）控制某功能 bean 是否启用。

### 4. 与自动配置的闭环

```text
@ConditionalOnClass        →  引了依赖才装（解决"没依赖也装会报错"）
@ConditionalOnMissingBean  →  用户没配它才装（解决"覆盖用户配置"）
@ConditionalOnProperty     →  配置开关控制装不装（解决"按需开关功能"）
```

> 一句话：`@Conditional` 是自动配置"按需、不冲突、可开关"的实现基础；上百个 `XxxAutoConfiguration` 之所以不会全生效，全靠这些子注解逐个把关。

---

## 六、核心问答（小结·考点）

课程小结的三连问，直接当面试答案背：

**Q1：`@Conditional` 及其衍生注解的作用是什么？**

> 满足给定条件后，才注册对应的 bean 对象到 Spring IOC 容器中。

**Q2：`@Conditional` 及其衍生注解可以作用在什么地方？**

> - 方法上：只针对当前这个方法声明的 bean
> - 类上：针对这个类中所有方法声明的 bean

**Q3：自己定义自动配置类的核心是什么？如何完成自动配置？（自定义 starter 的本质）**

> 1. 定义自动配置类（配置类 + `@Bean` + `@ConditionalOnXxx`）
> 2. 把自动配置类的全限定名写进
>    `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件
>
> 这样别人引入你的依赖后，`AutoConfigurationImportSelector` 会读这个文件加载你的配置类 → 满足 `@Conditional` 条件的 bean 自动进容器，使用者零配置。
> （旧版 ≤2.7.0 是写进 `META-INF/spring.factories`）

> 串起来：第三章讲"怎么把别人的 bean 装进来（@Import/ImportSelector）"，第四章讲"SpringBoot 怎么自动找到这些配置类（.imports）"，第五章讲"怎么按需筛选（@Conditional）"，Q3 就是把这三步反过来——**自己造一个 starter** 的完整步骤。

---

## 七、自定义 starter 实战（原理篇收官）

### 1. 场景

实际开发中常把公共组件封装成 SpringBoot starter，提供给各项目团队复用——别人**引一个依赖 + 配几行 yml** 就能用，不用关心内部实现。starter 同时含**起步依赖**和**自动配置**两块能力。

### 2. 通用结构：starter 一定是"双模块"

观察任何 starter（官方或第三方），都是成对出现：

| 模块 | 后缀 | 职责 |
|------|------|------|
| `xxx-spring-boot-starter` | `-starter` | **依赖管理**：空壳，只聚合依赖坐标，是别人引入的入口 |
| `xxx-spring-boot-autoconfigure` | `-autoconfigure` | **自动配置**：真正干活，放配置类、工具类、`@Conditional`、`.imports` |

> starter 模块依赖 autoconfigure 模块；使用者只引 starter，autoconfigure 靠依赖传递跟着进来。

实例对照（都遵守这个规律）：

```text
SpringBoot 官方   spring-boot-starter        + spring-boot-autoconfigure
MyBatis 提供      mybatis-spring-boot-starter + mybatis-spring-boot-autoconfigure
PageHelper 提供   pagehelper-spring-boot-starter + pagehelper-spring-boot-autoconfigure
```

> 呼应第二章"starter 本质=依赖清单 pom"：原来官方 starter 之所以引一个就够，是因为它内部依赖了对应的 autoconfigure。

### 3. 案例需求

- **需求**：自定义 `aliyun-oss-spring-boot-starter`，完成阿里云 OSS 操作工具类 `AliyunOSSOperator` 的自动配置。
- **目标**：使用方引入起步依赖后，直接 `@Autowired AliyunOSSOperator` 就能用，零手动配置。

### 4. 实现三步骤总览

```text
1. 创建 aliyun-oss-spring-boot-starter 模块（依赖管理：pom 聚合，引入 autoconfigure）
2. 创建 aliyun-oss-spring-boot-autoconfigure 模块，并在 starter 模块中引入它
3. 在 autoconfigure 模块中：
   ① 写配置属性类 AliyunOSSProperties（@ConfigurationProperties 接收 yml 配置）
   ② 写工具类 AliyunOSSOperator（真正干活）
   ③ 写自动配置类 AliyunOSSAutoConfiguration（@Bean 把工具类装进容器）
   ④ 把自动配置类全限定名写进
      META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

> 第三步④就是第四/六章 Q3 的亲手实现：`AutoConfigurationImportSelector` 去读这个 `.imports` 文件，发现并加载我们的配置类。

### 5. autoconfigure 模块代码

**① 配置属性类（接收使用者 yml 的配置）**

```java
@Data
@ConfigurationProperties(prefix = "aliyun.oss")   // 绑定 yml 中 aliyun.oss.* 的配置
public class AliyunOSSProperties {
    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
}
```

> `@ConfigurationProperties(prefix=...)` 把配置文件里 `aliyun.oss.endpoint` 等自动映射到字段。
> 单独写它**还不够**，必须靠下面的 `@EnableConfigurationProperties` 让它生效并进容器。

**② 工具类（真正干活的，依赖配置类）**

```java
public class AliyunOSSOperator {

    private final AliyunOSSProperties properties;

    public AliyunOSSOperator(AliyunOSSProperties properties) {
        this.properties = properties;            // 构造注入配置
    }

    public String upload(byte[] content, String originalFilename) throws Exception {
        String endpoint        = properties.getEndpoint();
        String bucketName      = properties.getBucketName();
        // 用阿里云 OSS SDK 上传，省略具体调用，返回访问 URL
        // OSS ossClient = new OSSClientBuilder().build(endpoint, ak, sk);
        // ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(content));
        return "https://" + bucketName + "." + endpoint + "/" + originalFilename;
    }
}
```

**③ 自动配置类（把工具类注册成 bean）**

```java
@Configuration
@EnableConfigurationProperties(AliyunOSSProperties.class)   // 让 @ConfigurationProperties 生效
public class AliyunOSSAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean              // 用户没自己配才用默认（兜底，呼应第五章）
    public AliyunOSSOperator aliyunOSSOperator(AliyunOSSProperties properties) {
        return new AliyunOSSOperator(properties);
    }
}
```

**④ 注册自动配置类**（resources 下，路径名一字不能错）：

```text
src/main/resources/
  └─ META-INF/spring/
       └─ org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

文件内容只有一行（全限定名）：

```text
com.aliyun.oss.AliyunOSSAutoConfiguration
```

### 6. starter 模块（只有 pom，无代码）

`aliyun-oss-spring-boot-starter` 的 `pom.xml` 里：引入 `aliyun-oss-spring-boot-autoconfigure` + 阿里云 OSS SDK 等真正要用的依赖。使用者只引 starter，这些靠依赖传递全进来。

### 7. 使用方怎么用（验证零配置）

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

```yaml
# application.yml —— 前缀对应 @ConfigurationProperties 的 prefix
aliyun:
  oss:
    endpoint: https://oss-cn-hangzhou.aliyuncs.com
    access-key-id: xxx
    access-key-secret: xxx
    bucket-name: my-bucket
```

```java
@RestController
public class UploadController {
    @Autowired
    private AliyunOSSOperator aliyunOSSOperator;   // 直接注入，无需任何手动配置
}
```

### 8. @EnableConfigurationProperties vs @ConfigurationProperties（易错点）

| 注解 | 作用 | 加在哪 |
|------|------|--------|
| `@ConfigurationProperties(prefix)` | 声明"我要绑定哪段配置" | 属性类（`AliyunOSSProperties`）上 |
| `@EnableConfigurationProperties(X.class)` | 让 X 这个属性类生效并注册成 bean | 自动配置类上 |

> 两者**必须配合**。在自定义 starter 里属性类一般不加 `@Component`（避免依赖组件扫描），改用 `@EnableConfigurationProperties` 显式启用——更可控，也符合 starter 不该依赖使用者扫描范围的原则。

### 9. 收官闭环

```text
起步依赖（二章）  → starter 模块聚合依赖
@Import/Selector（三章）+ .imports（四/六章） → autoconfigure 被自动发现加载
@Conditional（五章） → @ConditionalOnMissingBean 兜底，不覆盖用户
@ConfigurationProperties（七章） → 把使用者 yml 注入工具类
= 别人引依赖 + 配 yml + @Autowired 直接用，零配置
```

---

> 创建时间：2026-05-18
