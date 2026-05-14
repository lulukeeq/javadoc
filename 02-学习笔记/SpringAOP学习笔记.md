# Spring AOP 学习笔记

## 一、什么是 AOP

AOP：Aspect Oriented Programming，面向切面编程，也叫面向方法编程。

一句话理解：

> 不修改原始方法，在方法执行前后自动插入公共逻辑。

AOP 是一种思想，Spring AOP 是它在 Spring 框架中的实现。

---

## 二、为什么用 AOP

场景举例：统计所有业务方法的执行耗时。

如果每个方法都手动加计时代码，就是重复劳动。AOP 的优势：

- 减少重复代码
- 代码无侵入
- 提高开发效率
- 维护方便

常见应用场景：

- 记录系统操作日志
- 事务管理（`@Transactional` 底层就是 AOP）
- 权限控制
- 性能监控

---

## 三、快速入门：统计业务方法执行耗时

### 1. 引入依赖

```xml
<!-- AOP 起步依赖，无需指定版本，父工程已经管理 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 2. 编写切面类

```java
@Slf4j
@Aspect      // 标记这是切面类
@Component   // 交给 Spring 容器管理
public class RecordTimeAspect {

    @Around("execution(* com.itheima.service.impl.*.*(..))")
    public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {
        long beginTime = System.currentTimeMillis();  // 记录开始时间
        Object result = pjp.proceed();                // 执行原始方法
        long endTime = System.currentTimeMillis();    // 记录结束时间
        log.info("方法 {} 执行耗时：{} ms", pjp.getSignature(), endTime - beginTime);
        return result;
    }
}
```

### 3. 关键说明

| 注解 / 方法          | 作用                                      |
| -------------------- | ----------------------------------------- |
| `@Aspect`            | 标记当前类是切面类                        |
| `@Component`         | 将切面类交给 Spring 容器管理              |
| `@Around`            | 环绕通知，方法执行前后都能插入逻辑        |
| `pjp.proceed()`      | 执行原始方法，返回值必须 return 出去      |
| `pjp.getSignature()` | 获取被拦截方法的签名信息（类名 + 方法名） |

---

## 四、AOP 核心概念

AOP 中有五个核心概念，必须能对应到代码：

### 1. 连接点 JoinPoint

可以被 AOP 控制的方法，暗含方法执行时的相关信息。

简单理解：

> 目标对象里**所有可以**被拦截的方法，都是连接点。

### 2. 通知 Advice

指那些重复的逻辑，也就是共性功能，最终体现为一个方法。

例如计时代码就是通知。

### 3. 切入点 PointCut

匹配连接点的条件，通知仅在切入点方法执行时被应用。

简单理解：

> 用表达式从所有连接点中**筛选**出真正要拦截的那些。

### 4. 切面 Aspect

描述通知与切入点的对应关系（通知 + 切入点）。

简单理解：

> 切面 = 通知 + 切入点，最终落到一个切面类（如 `RecordTimeAspect`）。

### 5. 目标对象 Target

通知所应用的对象。

例如 `DeptServiceImpl` 这个被拦截方法所在的类，就是目标对象。

### 6. 概念与代码对应表

| 概念 | 代码中对应 |
|------|-----------|
| 目标对象 Target | `DeptServiceImpl` 这个类本身 |
| 连接点 JoinPoint | `list()`、`save()`、`delete()` 等方法 |
| 切入点 PointCut | `execution(* com.itheima.service.impl.*.*(..))` |
| 通知 Advice | `recordTime()` 方法里的计时逻辑 |
| 切面 Aspect | `RecordTimeAspect` 这个类 |

### 7. 连接点和切入点的区别

最容易混的两个概念：

- **连接点**：所有**可以**被拦截的方法（候选池）
- **切入点**：你**实际指定**要拦截的方法（用表达式过滤后的子集）

一句话理解：

> 连接点是"能拦的"，切入点是"要拦的"。

### 8. 五个概念的组合关系

这五个概念不是先后流程，而是**组合关系**：

```text
        通知（做什么）
            +              =  切面 ──应用到──> 目标类
        切入点（拦哪些）                          │
                                                 │
                                                 ▼
                                          连接点（被拦截的方法）
```

记忆口诀：

> **目标类**里有一堆方法（都是**连接点**），
> 我用**切入点**表达式挑出要拦的那几个，
> 写一段**通知**逻辑塞进去，
> 这俩合起来就是**切面**，
> 底层靠**动态代理**实现。

易错点：

- 目标类**包含**连接点，不是"目标类之后才有连接点"
- 切面 = 通知 + 切入点，不是单指切入点或单指通知

---

## 五、AOP 执行流程（动态代理）

AOP 能做到"不改原代码、却能在方法前后插入逻辑"，底层靠的是**动态代理**。

### 1. 正常情况（没有 AOP）

```text
Controller 注入 DeptService
        ↓
直接调到 DeptServiceImpl.list()
```

### 2. 有 AOP 时

Spring 偷偷做了一件事：

> Controller 里注入的不是 `DeptServiceImpl`，而是一个**代理对象 DeptServiceProxy**。

代理对象的 `list()` 方法长这样（伪代码）：

```java
public List<Dept> list() {
    long begin = System.currentTimeMillis();    // 通知前半段
    List<Dept> deptList = 目标对象.list();       // 调真正的 DeptServiceImpl.list()
    long end = System.currentTimeMillis();      // 通知后半段
    log.info("执行耗时：{} ms", end - begin);
    return deptList;
}
```

### 3. 完整流程

```text
Controller 调 deptService.list()
        ↓
实际调到代理对象 DeptServiceProxy.list()
        ↓
代理里先执行通知前半段（计时开始）
        ↓
代理里调用目标对象 DeptServiceImpl.list()
        ↓
代理里执行通知后半段（计时结束、打日志）
        ↓
返回结果给 Controller
```

### 4. 关键点

1. Controller 完全不知道自己用的是代理 —— 代理和目标对象都实现了同一个 `DeptService` 接口
2. 代理对象是 Spring 在**运行时动态生成**的，所以叫"动态代理"
3. 通知逻辑能"无侵入"地加到目标方法前后，靠的就是这层代理

一句话理解：

> AOP 的本质 = 给目标对象生成一个代理对象，所有调用都先过代理，代理再决定何时调用目标方法、前后做什么。

---

## 六、通知类型

根据通知方法执行时机的不同，分为以下五类：

### 1. 五种通知类型

| 注解 | 触发时机 | 异常时是否执行 |
|------|---------|---------------|
| `@Around` | 目标方法**前 + 后** | 看你怎么写 |
| `@Before` | 目标方法**前** | 不涉及（已经执行过了） |
| `@After` | 目标方法**后** | ✅ 执行（类似 finally） |
| `@AfterReturning` | 目标方法**正常返回后** | ❌ 不执行 |
| `@AfterThrowing` | 目标方法**抛异常后** | ✅ 执行 |

### 2. 执行流程图

```text
        ┌─ @Before（前置）
        │
   目标方法执行
        │
        ├─ 正常返回 ──> @AfterReturning（返回后）─┐
        │                                          ├─> @After（后置，无论如何都执行）
        └─ 抛出异常 ──> @AfterThrowing（异常后）──┘

   @Around 包在最外层，把上面整个流程包起来
```

### 3. 两个关键注意点

**注意 1：`@Around` 必须自己调用 `pjp.proceed()`**

原始方法才会执行；其他四种通知不需要，因为它们只在某个时机插入，不负责"决定要不要执行"目标方法。

**注意 2：`@Around` 的返回值必须是 `Object`**

因为它要接住原始方法的返回值再 return 出去：

```java
public Object xxx(ProceedingJoinPoint pjp) throws Throwable {
    Object result = pjp.proceed();  // 接住返回值
    return result;                   // 必须 return
}
```

其他四种通知（`@Before` 等）不接管目标方法的执行，所以返回值可以是 `void`。

### 4. 易记对比

- `@After` ≈ `try-finally` 里的 `finally`（无论如何都执行）
- `@AfterReturning` ≈ 没抛异常才走
- `@AfterThrowing` ≈ 抛了异常才走

### 5. 实际执行顺序（Spring 5.2.7+）

**无异常时：**

```text
@Around 前 → @Before → 目标方法 → @AfterReturning → @After → @Around 后
```

**有异常时：**

```text
@Around 前 → @Before → 目标方法抛异常 → @AfterThrowing → @After → ❌ @Around 后不执行
```

### 6. 为什么异常时 `@Around` 后半段不执行

`@Around` 的代码本质长这样：

```java
public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {
    log.info("before around");          // 前半段
    Object result = pjp.proceed();      // 这里抛异常，直接跳出方法
    log.info("after around");           // 后半段被跳过
    return result;
}
```

`pjp.proceed()` 抛异常后，下面的代码走不到，所以 `after around` 不打印。

### 7. 为什么异常时 `@After` 仍然执行

因为 `@After` 是 Spring 在更外层用 `try-finally` 自动包的一层，类似：

```java
try {
    // 调用代理链（@Around、@Before、目标方法、@AfterReturning/@AfterThrowing）
} finally {
    // @After 永远执行
}
```

所以 `@After` 行为就像 `finally`。

### 8. 一句话总结

> `@Around` 是用户自己控制的代码，能不能走完取决于你写没写 `try-catch`；
> `@After` 是 Spring 框架兜底保证的，永远会执行。

### 9. `@Pointcut`：抽取切入点表达式

**问题：** 五个通知都要写一遍切入点表达式，重复且难维护：

```java
@Before("execution(* com.itheima.service.impl.*.*(..))")
public void before() {...}

@After("execution(* com.itheima.service.impl.*.*(..))")
public void after() {...}
// ... 写五遍
```

**用 `@Pointcut` 抽取：**

```java
// 1. 定义一个切入点方法（方法体空着就行）
@Pointcut("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
public void pt() {}

// 2. 其他通知引用方法名
@Around("pt()")
public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {...}

@Before("pt()")
public void before() {...}
```

要改表达式只改一处。

**`private` vs `public` 修饰符：**

| 修饰符 | 作用范围 |
|--------|---------|
| `private` | 仅当前切面类内部能引用 |
| `public` | 其他切面类也能引用（跨类用 `类全名.方法名()`） |

**跨类引用写法：**

```java
@Around("com.itheima.aop.CommonPointcut.pt()")
```

**关键理解：**

> `@Pointcut` 标注的方法只是一个"表达式的别名"，方法体永远是空的，方法名是用来被引用的。

---

## 七、通知顺序

当多个切面类的切入点都匹配到同一个目标方法时，多个通知都会被执行，那它们按什么顺序？

### 1. 默认规则：按类名字母排序

| 时机 | 顺序 |
|------|------|
| 目标方法**前**的通知（`@Before`、`@Around` 前半段） | 字母靠前的**先**执行 |
| 目标方法**后**的通知（`@After`、`@AfterReturning` 等） | 字母靠前的**后**执行 |

### 2. 洋葱模型

把多个切面理解为层层包裹：

```text
@Order(1) Aspect1 前 ───┐
  @Order(2) Aspect2 前 ─┐ │
    目标方法            │ │
  @Order(2) Aspect2 后 ─┘ │
@Order(1) Aspect1 后 ───┘
```

一句话理解：

> 外层先进，内层最后进，再倒着出去——像洋葱、像栈（LIFO）。

### 3. 完整执行顺序示例

两个切面 Aspect1 和 Aspect2（字母 1 < 2），都对同一方法生效：

```text
b1 → b2 → 目标方法 → ar2 → a2 → ar1 → a1
```

记忆要点：

- 进的时候按字母**升序**进（b1 → b2）
- 出的时候按字母**降序**出（ar2 → a2 → ar1 → a1）
- 同一个切面内：`@AfterReturning` → `@After`

### 4. 用 `@Order` 手动控制顺序

加在切面类上，数字决定优先级：

```java
@Slf4j
@Order(1)        // 数字越小，越外层
@Aspect
@Component
public class RecordTimeAspect {
   ...
}
```

规则和默认字母排序对应得很整齐：

| 时机 | 顺序 |
|------|------|
| 目标方法**前** | 数字**小**的**先**执行 |
| 目标方法**后** | 数字**小**的**后**执行 |

### 5. `@Order` 和字母排序的关系

- 没加 `@Order` → 按类名字母排序
- 加了 `@Order` → 按 `@Order` 数字排序，**优先级高于字母排序**
- 同时存在加了的和没加的 → 加了 `@Order` 的优先决定顺序

---

## 八、切入点表达式

切入点表达式描述切入点方法的一种表达式，用来决定哪些方法需要加入通知。

### 1. 两种常见形式

| 形式 | 匹配依据 | 适合场景 |
|------|---------|---------|
| `execution(...)` | 按方法签名 | 拦"某个包/某个类下的方法" |
| `@annotation(...)` | 按方法上的注解 | 拦"想加日志的方法"（贴标签式） |

### 2. execution 完整语法

```text
execution(访问修饰符? 返回值 包名.类名.?方法名(方法参数) throws 异常?)
```

带 `?` 的部分可省略：

| 部分 | 是否可省略 |
|------|-----------|
| 访问修饰符（`public`、`protected`） | ✅ 可省 |
| 返回值 | ❌ 必填 |
| 包名 | ✅ 可省 |
| 类名 | ✅ 可省 |
| 方法名 | ❌ 必填 |
| 方法参数 | ❌ 必填 |
| `throws 异常` | ✅ 可省 |

⚠️ 注意：`throws 异常` 是方法签名上**声明**的异常，不是运行时实际抛出的异常。

### 3. 两个通配符

**`*`：匹配单个内容（一个层级、一个符号）**

```java
execution(* com.*.service.*.update*(*))
```

| 位置 | `*` 的含义 |
|------|-----------|
| 返回值的 `*` | 任意返回类型 |
| `com.*` | `com` 下的**一层**包 |
| `service.*` | `service` 下任意一个类 |
| `update*` | 方法名以 `update` 开头 |
| `(*)` | 一个任意类型的参数 |

**`..`：匹配多个连续内容**

```java
execution(* com.itheima..DeptService.*(..))
```

| 位置 | `..` 的含义 |
|------|------------|
| `com.itheima..` | `com.itheima` 下任意层级的包 |
| `(..)` | 任意类型、任意个数的参数（0/1/多 都行）|

一句话区分：

> `*` 是**一个**（一层包、一个类、一个方法、一个参数）
> `..` 是**多个**（多层包、任意个参数）

### 4. 实战最常用的两个 execution

```java
// 拦截某个包下所有类的所有方法（最常用）
execution(* com.itheima.service.impl.*.*(..))

// 拦截某个具体类的所有方法
execution(* com.itheima.service.impl.DeptServiceImpl.*(..))
```

### 5. 逻辑运算符组合

切入点表达式可以用 `&&`、`||`、`!` 组合：

```java
@Before("execution(* com.itheima.service.impl.DeptServiceImpl.list(..)) || " +
        "execution(* com.itheima.service.impl.DeptServiceImpl.delete(..))")
public void before() {...}
```

⚠️ 注意是写在字符串里的，多行表达式要用 Java 字符串拼接（`+`）。

**更推荐用 `@Pointcut` 组合：**

```java
@Pointcut("execution(* com.itheima.service.impl.DeptServiceImpl.list(..))")
public void listPt() {}

@Pointcut("execution(* com.itheima.service.impl.DeptServiceImpl.delete(..))")
public void deletePt() {}

@Before("listPt() || deletePt()")
public void before() {...}
```

### 6. @annotation 用法

**第一步：定义切面**

```java
@Around("@annotation(com.itheima.anno.LogOperation)")
public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
    long startTime = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    // ...
}
```

表达式里写**注解的全限定名**。

**第二步：在需要的方法上贴注解**

```java
@LogOperation                    // 想拦哪个方法就贴哪个
@DeleteMapping
public Result delete(Integer id) {...}

@LogOperation
@PostMapping
public Result save(@RequestBody Dept dept) {...}
```

**注意：** `@LogOperation` 是开发者**自定义的注解**，不是 Spring 自带的。

### 7. execution vs @annotation 对比

| | `execution` | `@annotation` |
|---|------------|---------------|
| 由谁决定拦谁 | 切面单方面决定（写表达式） | 目标方法主动声明（贴注解） |
| 拦截范围 | 按包/类/方法名匹配 | 按注解标记匹配 |
| 加新方法时 | 自动拦截（在匹配范围内） | 需要手动贴注解 |
| 跨包跨类时 | 难写表达式 | 简单，只看注解 |
| 典型场景 | 全包计时、统一权限校验 | 关键操作日志、特定方法增强 |

### 8. 书写建议（项目实践规范）

| 建议 | 含义 | 为什么 |
|------|------|--------|
| 方法名命名规范 | `findXxx`、`updateXxx`、`deleteXxx` | 表达式直接 `find*` / `update*` 就能精准匹配 |
| **基于接口描述，不是实现类** | 写 `DeptService` 而不是 `DeptServiceImpl` | 实现类换了名字，表达式不用改，增强拓展性 |
| 尽量缩小匹配范围 | 包名尽量不用 `..`，用 `*` 匹配单层包 | 避免误伤别的包下的方法 |

**基于接口对比：**

```java
// 推荐 ✅（基于接口）
execution(* com.itheima.service.DeptService.*(..))

// 不推荐 ❌（基于实现类）
execution(* com.itheima.service.impl.DeptServiceImpl.*(..))
```

---

## 九、连接点 JoinPoint

在 Spring 中用 `JoinPoint` 抽象了连接点，通过它可以获取方法执行时的相关信息：目标类名、方法名、方法参数等。

### 1. 两个相关类的关系

```text
JoinPoint（父接口）
  ├─ getSignature()
  ├─ getArgs()
  └─ getTarget()
         ↑
         ├─ 继承
         │
ProceedingJoinPoint（子接口，@Around 专用）
  └─ proceed()   // 多了这个能力
```

| 类 | 用在哪 | 是否能执行原方法 |
|----|--------|-----------------|
| `JoinPoint` | `@Before`、`@After`、`@AfterReturning`、`@AfterThrowing` | ❌ 不能 |
| `ProceedingJoinPoint` | `@Around` 专属 | ✅ 通过 `proceed()` |

一句话理解：

> `@Around` 需要主动控制目标方法的执行，所以它的参数必须是 `ProceedingJoinPoint`；其他四种通知不需要这个能力，用 `JoinPoint` 就够了。

### 2. 共用的四个常用方法

| 方法 | 作用 |
|------|------|
| `getTarget()` | 获取目标对象 |
| `getTarget().getClass().getName()` | 获取目标类全限定名 |
| `getSignature().getName()` | 获取目标方法名 |
| `getArgs()` | 获取方法参数（`Object[]`）|

### 3. 完整 demo

```java
@Before("execution(* com.itheima.service.*.*(..))")
public void before(JoinPoint joinPoint) {
    // 1. 获取目标对象
    Object target = joinPoint.getTarget();
    log.info("获取目标对象：{}", target);

    // 2. 获取目标类
    String className = joinPoint.getTarget().getClass().getName();
    log.info("获取目标类：{}", className);

    // 3. 获取目标方法
    String methodName = joinPoint.getSignature().getName();
    log.info("获取目标方法：{}", methodName);

    // 4. 获取目标方法参数
    Object[] args = joinPoint.getArgs();
    log.info("获取目标方法参数：{}", Arrays.toString(args));
}
```

### 4. 易错点：打印参数数组

```java
Object[] args = joinPoint.getArgs();

// ❌ 直接打印是数组地址
log.info("参数: {}", args);            // 输出 [Ljava.lang.Object;@xxxxx

// ✅ 必须用 Arrays.toString
log.info("参数: {}", Arrays.toString(args));   // 输出 [1, 2, 3]
```

### 5. `@Around` 独有的 `proceed()`

```java
@Around("execution(* com.itheima.service.*.*(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    Object res = joinPoint.proceed();  // 执行原方法，必须接住返回值
    return res;                         // 必须 return 出去
}
```

### 6. 实际应用场景

| 方法 | 常见用途 |
|------|---------|
| `getSignature()` + `getArgs()` | **记录操作日志**：是谁、调了什么方法、传了什么参数 |
| `getArgs()` | **权限校验**：拿到参数判断当前用户能否操作 |
| `proceed()` | **性能监控、事务、缓存**：在 `@Around` 里包裹目标方法 |

---

> 创建时间：2026-05-13
> 最近更新：2026-05-14
