# SpringBoot 原理 - 速记

> 配套笔记：`../02-学习笔记/SpringBoot原理学习笔记.md`
> 两大核心：起步依赖（依赖怎么来）+ 自动配置（bean 怎么自动装好）

---

## 一、起步依赖

**一句话：** 一个 pom，靠 Maven 依赖传递（多层嵌套），一次引入一整套依赖 + 自动配置包。

starter 命名规律：

- 官方：`spring-boot-starter-xxx`（前缀打头）
- 第三方：`xxx-spring-boot-starter`（名字打头）

---

## 二、自动配置完整链路（背这张图）

```text
@SpringBootApplication
 ├─ @SpringBootConfiguration   = @Configuration，启动类也是配置类
 ├─ @ComponentScan             默认扫启动类所在包及子包（→第三方包扫不到）
 └─ @EnableAutoConfiguration   自动配置总开关
      └─ @Import(AutoConfigurationImportSelector.class)
           └─ selectImports() 读 jar 里的
              META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
              （旧版 ≤2.7.0：META-INF/spring.factories）
                └─ 拿到上百个 XxxAutoConfiguration（只是候选清单，不是全装！）
                     └─ 逐个 @Conditional 判断 → 命中的才注册 bean
                          └─ 用户 @Autowired 直接用，零配置
```

---

## 三、@Conditional 三大常用（速查）

| 注解 | 生效条件 | 解决 |
|------|---------|------|
| `@ConditionalOnClass` | classpath 有指定类（引了依赖） | 没依赖也装会报错 |
| `@ConditionalOnMissingBean` | 容器无该 bean（按类型/名称） | 不覆盖用户自己的配置 |
| `@ConditionalOnProperty` | 配置文件属性值匹配（prefix+name+havingValue） | 功能开关 |

作用位置：方法上（管单个 @Bean）/ 类上（管整个配置类的所有 @Bean）。

---

## 四、高频面试问答

**Q：自动配置类从哪来？**
> 新版（≥2.7.0）读 `META-INF/spring/...AutoConfiguration.imports`；旧版读 `META-INF/spring.factories`。由 `AutoConfigurationImportSelector.selectImports()` 加载。

**Q：清单里上百个配置类都注册成 bean 吗？**
> 不会。那只是候选清单，每个配置类/方法上的 `@Conditional` 逐个把关，满足条件的才装。

**Q：`@ConditionalOnClass` 引用的类没引入为什么不报错？**
> 演示用 `name = "全限定名字符串"` 而非 `value = X.class`；字符串延迟到运行期反射判断，类不在也不崩。

**Q：第三方包用 @Component 声明的 bean 为啥不生效？**
> @SpringBootApplication 的 @ComponentScan 默认只扫启动类包及子包，第三方包不在范围。解决：@ComponentScan 指定包，或 @Import（4 形式：普通类 / 配置类 / ImportSelector / @EnableXxx）。

**Q：怎么自己写一个 starter（自定义自动配置）？**
> ① 写自动配置类（@Configuration + @Bean + @ConditionalOnXxx）；② 全限定名写进 `META-INF/spring/...AutoConfiguration.imports`。别人引依赖即零配置可用。

---

## 五、自定义 starter（实战收官）

**铁律：starter 一定是双模块**

| 模块 | 职责 |
|------|------|
| `xxx-spring-boot-starter` | 依赖管理，空壳 pom，引入 autoconfigure |
| `xxx-spring-boot-autoconfigure` | 自动配置实现，干活 |

**四步（aliyun-oss 案例）：**

1. `AliyunOSSProperties` —— `@ConfigurationProperties(prefix="aliyun.oss")` 接 yml
2. `AliyunOSSOperator` —— 工具类，构造注入 Properties
3. `AliyunOSSAutoConfiguration` —— `@Configuration` + `@EnableConfigurationProperties(AliyunOSSProperties.class)` + `@Bean`（配 `@ConditionalOnMissingBean` 兜底）
4. 全限定名写进 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

**易错对比：**

- `@ConfigurationProperties(prefix)` 加属性类上：声明绑定哪段配置
- `@EnableConfigurationProperties(X.class)` 加自动配置类上：让 X 生效并进容器
- 二者**必须配合**；starter 里属性类不加 `@Component`（不依赖使用者扫描范围）

使用方：引 starter → 配 yml（前缀对上 prefix）→ `@Autowired` 直接用，零配置。

---

## 六、易错点

- starter 本身几乎没代码，就是一个"依赖清单 pom"
- 起步依赖只负责"jar 进来"，**不负责"bean 注册"**——后者是自动配置的事
- `@EnableXxx` 本质 = 封装了 `@Import` 的语义化注解（@EnableScheduling/@EnableCaching/@EnableAsync/@EnableTransactionManagement 同理）
- 新旧版自动配置清单文件不同，面试两个都要提
