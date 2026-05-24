# SpringAOP - 速记

> 配套笔记：`../02-学习笔记/SpringAOP学习笔记.md`
> 当前覆盖：133 ~ 140，含基础、进阶与案例

---

## 一、AOP 是什么

一句话：

> 不改原始方法，在方法执行前后统一织入公共逻辑。

常见场景：

- 记录日志
- 统计耗时
- 权限校验
- 事务管理

---

## 二、5 个核心概念

| 概念 | 解释 |
|------|------|
| 连接点 `JoinPoint` | 可以被拦截的方法 |
| 切入点 `Pointcut` | 真正要拦截的那一批方法 |
| 通知 `Advice` | 拦截后要执行的增强逻辑 |
| 切面 `Aspect` | 切入点 + 通知 组成的整体 |
| 目标对象 `Target` | 被增强的原始对象 |

记忆：

> 先找“拦谁”（切入点），再写“干什么”（通知），合起来就是切面。

---

## 三、5 种通知

| 通知 | 时机 | 能不能拿异常 | 能不能改返回值 |
|------|------|-------------|---------------|
| `@Before` | 方法执行前 | ❌ | ❌ |
| `@After` | 方法执行后（无论成败） | ❌ | ❌ |
| `@AfterReturning` | 方法正常返回后 | ❌ | ✅ 可拿返回值 |
| `@AfterThrowing` | 方法抛异常后 | ✅ | ❌ |
| `@Around` | 前后都能包住 | ✅ | ✅ 最强 |

一句话：

> 真正常用、能力最强的是 `@Around`。

---

## 四、`@Around` 最关键的 3 句

```java
@Around("execution(* com.itheima.service.*.*(..))")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    Object result = pjp.proceed();
    return result;
}
```

必须记住：

1. 参数必须是 `ProceedingJoinPoint`
2. 必须调用 `proceed()` 才会执行原方法
3. 必须把原方法返回值 `return` 出去

---

## 五、切入点表达式 `execution`

通用写法：

```java
execution(访问修饰符 返回值 包名.类名.方法名(参数))
```

最常见：

```java
execution(* com.itheima.service.*.*(..))
```

通配规则：

- `*`：单个任意
- `..`：任意层包 / 任意个参数

---

## 六、`@Pointcut` 的作用

作用：

> 把重复的切入点表达式抽出来复用。

示例：

```java
@Pointcut("execution(* com.itheima.service.*.*(..))")
public void pt() {}
```

然后通知里写：

```java
@Before("pt()")
```

---

## 七、通知顺序

多个切面都拦同一个方法时：

- 数字越小，优先级越高
- `@Order(1)` 先于 `@Order(2)`

记忆：

> `@Before` 先执行优先级高的；`@After` 类通知执行顺序反过来更像“出栈”。

---

## 八、JoinPoint 常用方法

| 方法 | 作用 |
|------|------|
| `getTarget()` | 获取目标对象 |
| `getSignature().getName()` | 获取方法名 |
| `getArgs()` | 获取参数数组 |

易错点：

```java
Arrays.toString(joinPoint.getArgs())
```

不要直接打印数组对象地址。

---

## 九、表达式匹配 vs 注解匹配

| 方式 | 特点 |
|------|------|
| 表达式匹配 | 适合整包、整类统一增强 |
| 注解匹配 | 适合关键操作、指定方法增强 |

记忆：

> “全局规则”偏表达式；“指定动作”偏注解。

---

## 十、今天先背这 6 句

1. AOP 就是在不改原方法的前提下织入公共逻辑
2. 切面 = 切入点 + 通知
3. 最常用最强的通知是 `@Around`
4. `ProceedingJoinPoint.proceed()` 不调用，原方法就不会执行
5. `@Pointcut` 是为了复用表达式
6. `@Order` 数字越小，优先级越高

---

## 十一、案例落点：操作日志

一句话：

> 新增、修改、删除这类关键操作，非常适合用 AOP 统一记日志。

为什么：

- 到处都要记
- 不是核心业务
- 写在业务方法里会很重复

最常见搭配：

```java
@annotation(...)
+ @Around
+ ProceedingJoinPoint
```

记忆：

> 全局统一规则偏 `execution`，关键操作日志偏 `@annotation`。

## 十二、案例里最常见的 4 个动作

1. 自定义一个方法注解，比如 `@LogOperation`
2. 在需要记录日志的方法上贴注解
3. 切面用 `@annotation` 拦截
4. 在通知里拿方法名、参数、耗时并统一记录

## 十三、AOP 适合放什么，不适合放什么

适合：

- 日志
- 耗时统计
- 权限校验
- 事务

不适合：

- 订单金额计算
- 工资条生成
- 业务规则判断本身

记忆：

> AOP 适合横切公共逻辑，不适合承载主业务。
