# Java 异常学习笔记

## 一、异常概述

### 1. 什么是异常

异常就是程序在运行过程中出现的不正常情况。

简单理解：

> 程序本来应该按正常流程执行，但中途出现了错误或特殊情况，就会产生异常。

例如：

- 空指针
- 数组下标越界
- 类型转换错误
- 文件不存在
- 数据库连接失败

异常处理的目的不是让错误消失，而是让程序在出现问题时能够有明确的处理方式。

---

## 二、Java 异常体系

Java 中所有异常和错误的顶层父类是 `Throwable`。

异常体系可以这样理解：

```text
Throwable
├── Error
└── Exception
    ├── RuntimeException
    └── 其他编译时异常
```

### 1. Error

`Error` 表示严重错误，通常不是程序员能处理的。

常见例子：

- `OutOfMemoryError`：内存溢出
- `StackOverflowError`：栈溢出

这类问题一般不建议在代码中捕获，而是应该从系统配置、代码逻辑或运行环境上解决。

### 2. Exception

`Exception` 表示程序可以处理的异常。

它又可以分为：

- 编译时异常
- 运行时异常

### 3. RuntimeException

`RuntimeException` 是运行时异常的父类。

常见运行时异常：

| 异常 | 说明 |
|------|------|
| `NullPointerException` | 空指针异常 |
| `ArrayIndexOutOfBoundsException` | 数组下标越界 |
| `ClassCastException` | 类型转换异常 |
| `NumberFormatException` | 数字格式转换异常 |
| `IllegalArgumentException` | 参数不合法 |

---

## 三、编译时异常和运行时异常

### 1. 编译时异常

编译时异常也叫受检异常。

特点：

- 编译阶段就会检查
- 必须处理，否则代码无法通过编译
- 可以使用 `try-catch` 捕获
- 也可以使用 `throws` 抛出

常见例子：

- `IOException`
- `SQLException`
- `ClassNotFoundException`

示例：

```java
public void readFile() throws IOException {
    FileReader reader = new FileReader("a.txt");
}
```

### 2. 运行时异常

运行时异常也叫非受检异常。

特点：

- 编译阶段不会强制处理
- 程序运行时才可能出现
- 通常是代码逻辑问题导致的

示例：

```java
String name = null;
System.out.println(name.length());
```

这里会出现 `NullPointerException`。

### 3. 区别总结

| 对比项 | 编译时异常 | 运行时异常 |
|--------|------------|------------|
| 是否强制处理 | 是 | 否 |
| 出现阶段 | 编译阶段检查 | 运行阶段出现 |
| 常见原因 | 外部资源问题，如文件、数据库、网络 | 代码逻辑问题 |
| 处理方式 | `try-catch` 或 `throws` | 通常先修正代码逻辑 |

---

## 四、异常处理方式

### 1. try-catch

`try-catch` 用来捕获并处理异常。

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("除数不能为 0");
}
```

说明：

- `try` 中写可能出现异常的代码
- `catch` 中写异常出现后的处理逻辑

### 2. try-catch-finally

`finally` 中的代码通常用于释放资源。

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("出现异常：" + e.getMessage());
} finally {
    System.out.println("finally 执行");
}
```

说明：

- 不管是否出现异常，`finally` 通常都会执行
- 常用于关闭文件、关闭连接、释放资源

### 3. 多个 catch

一段代码可能出现多种异常，可以写多个 `catch`。

```java
try {
    String str = null;
    System.out.println(str.length());
} catch (NullPointerException e) {
    System.out.println("空指针异常");
} catch (Exception e) {
    System.out.println("其他异常");
}
```

注意：

> 多个 `catch` 时，子类异常要写在前面，父类异常要写在后面。

### 4. throw

`throw` 用在方法内部，表示主动抛出一个异常对象。

```java
public void checkAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("年龄不能小于 0");
    }
}
```

### 5. throws

`throws` 用在方法声明上，表示这个方法可能会抛出异常。

```java
public void readFile() throws IOException {
    FileReader reader = new FileReader("a.txt");
}
```

### 6. throw 和 throws 的区别

| 对比项 | `throw` | `throws` |
|--------|---------|----------|
| 使用位置 | 方法内部 | 方法声明上 |
| 作用 | 主动抛出一个异常对象 | 声明方法可能抛出异常 |
| 后面跟什么 | 异常对象 | 异常类型 |
| 数量 | 一次只能抛出一个异常对象 | 可以声明多个异常类型 |

---

## 五、自定义异常

### 1. 为什么要自定义异常

Java 内置异常不一定能表达具体业务含义。

例如：

- 用户不存在
- 余额不足
- 订单状态错误
- 参数不符合业务规则

这时可以定义自己的异常，让错误更清楚。

### 2. 自定义运行时异常

业务开发中，自定义异常通常继承 `RuntimeException`。

```java
public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}
```

使用示例：

```java
public void pay(int money) {
    if (money <= 0) {
        throw new BusinessException("支付金额必须大于 0");
    }
}
```

### 3. 自定义编译时异常

如果继承 `Exception`，就是编译时异常。

```java
public class MyCheckedException extends Exception {
    public MyCheckedException(String message) {
        super(message);
    }
}
```

使用时必须处理：

```java
public void test() throws MyCheckedException {
    throw new MyCheckedException("自定义编译时异常");
}
```

---

## 六、常见面试题

### 1. final、finally、finalize 的区别

| 名称 | 说明 |
|------|------|
| `final` | Java 关键字，可以修饰类、方法、变量 |
| `finally` | 异常处理中的代码块，通常用于释放资源 |
| `finalize` | Object 类中的方法，垃圾回收前可能调用，已经不推荐使用 |

### 2. finally 一定会执行吗

通常情况下会执行。

但下面情况可能不会执行：

- JVM 直接退出，例如 `System.exit(0)`
- 程序崩溃
- 机器断电

### 3. 自定义异常为什么通常继承 RuntimeException

原因：

- 不需要强制调用方处理
- 更适合表达业务规则错误
- 代码使用起来更简洁

但如果异常必须被调用方明确处理，也可以继承 `Exception`。

---

## 七、复习完成标准

- [ ] 能画出 Java 异常体系结构
- [ ] 能说清 `Error` 和 `Exception` 的区别
- [ ] 能说清编译时异常和运行时异常的区别
- [ ] 能写出 `try-catch-finally` 示例
- [ ] 能区分 `throw` 和 `throws`
- [ ] 能写一个简单自定义异常
- [ ] 能回答 `final`、`finally`、`finalize` 的区别

---

> 创建时间：2026-04-28
