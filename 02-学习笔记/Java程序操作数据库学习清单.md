# Java 程序操作数据库学习清单

## 学习目录

### 一、JDBC（基础规范）

- 入门程序

### 二、MyBatis

### 三、MyBatisPlus

> 💡 JDBC 是基础规范，MyBatis 和 MyBatisPlus 等框架都是基于 JDBC 的封装。

---

## 学习笔记

### 一、JDBC（基础规范）

**JDBC（Java DataBase Connectivity）：** 就是使用 Java 语言操作关系型数据库的一套 API。

**本质：**

- Sun 公司定义的一套操作所有关系型数据库的规范，即接口
- 各个数据库厂商去实现这套接口，提供数据库驱动 jar 包
- 可以使用这套接口（JDBC）编程，真正执行的代码是驱动 jar 包中的实现类

#### 1.1 入门程序

**步骤：**

1. 注册驱动
2. 获取连接
3. 获取 SQL 语句执行对象
4. 执行 SQL
5. 释放资源

**代码案例：**

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

public class JdbcDemo {
    public static void main(String[] args) throws Exception {
        // 1. 注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        // 2. 获取连接
        String url = "jdbc:mysql://localhost:3306/mydb";
        String username = "root";
        String password = "123456";
        Connection conn = DriverManager.getConnection(url, username, password);

        // 3. 获取 SQL 语句执行对象
        Statement stmt = conn.createStatement();

        // 4. 执行 SQL
        String sql = "SELECT * FROM user";
        stmt.executeQuery(sql);

        // 5. 释放资源
        stmt.close();
        conn.close();
    }
}
```

#### 1.2 查询数据

**Statement 执行 SQL 的两种方法：**

| 方法                 | 适用场景                           | 返回值                |
| -------------------- | ---------------------------------- | --------------------- |
| `executeUpdate(sql)` | DML 语句（INSERT、UPDATE、DELETE） | `int`（受影响的行数） |
| `executeQuery(sql)`  | DQL 语句（SELECT）                 | `ResultSet`（结果集） |

**ResultSet（结果集对象）：**

- `next()`：将光标从当前位置向下移动一行，返回 boolean 表示是否有数据
- `getXXX()`：获取当前行中某列的值，如 `getInt()`、`getString()` 等

**查询代码案例：**

```java
import java.sql.*;

public class JdbcQueryDemo {
    public static void main(String[] args) throws Exception {
        // 1. 注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        // 2. 获取连接
        String url = "jdbc:mysql://localhost:3306/mydb";
        Connection conn = DriverManager.getConnection(url, "root", "123456");

        // 3. 获取 SQL 语句执行对象
        Statement stmt = conn.createStatement();

        // 4. 执行查询 SQL
        String sql = "SELECT id, name, age FROM user";
        ResultSet rs = stmt.executeQuery(sql);

        // 5. 遍历结果集
        while (rs.next()) {
            int id = rs.getInt("id");
            String name = rs.getString("name");
            int age = rs.getInt("age");
            System.out.println("id=" + id + ", name=" + name + ", age=" + age);
        }

        // 6. 释放资源
        rs.close();
        stmt.close();
        conn.close();
    }
}
```

#### 1.3 预编译 SQL（PreparedStatement）

**预编译 SQL 的优势：**

1. **防止 SQL 注入**，更安全
2. **性能更高**

**SQL 执行流程：**

```
SQL语法解析检测 → 优化SQL → 编译SQL → 执行SQL
```

> 预编译的 SQL 会被缓存下来，缓存的 SQL 不会再进行语法解析检测、优化、编译，直接执行，从而提高性能。

**PreparedStatement 代码案例：**

```java
import java.sql.*;

public class JdbcPreparedStatementDemo {
    public static void main(String[] args) throws Exception {
        // 1. 注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        // 2. 获取连接
        String url = "jdbc:mysql://localhost:3306/mydb";
        Connection conn = DriverManager.getConnection(url, "root", "123456");

        // 3. 获取预编译 SQL 对象（使用 ? 占位符）
        String sql = "SELECT * FROM user WHERE name = ? AND age = ?";
        PreparedStatement pstmt = conn.prepareStatement(sql);

        // 4. 设置参数（索引从 1 开始）
        pstmt.setString(1, "张三");
        pstmt.setInt(2, 25);

        // 5. 执行查询
        ResultSet rs = pstmt.executeQuery();
        while (rs.next()) {
            int id = rs.getInt("id");
            String name = rs.getString("name");
            System.out.println("id=" + id + ", name=" + name);
        }

        // 6. 释放资源
        rs.close();
        pstmt.close();
        conn.close();
    }
}
```

---

### 二、MyBatis

**MyBatis** 是一款优秀的持久层框架，用于简化 JDBC 开发。

> （待补充）

---

## 学习进度追踪

- [ ] JDBC（基础规范）
- [ ] MyBatis
- [ ] MyBatisPlus

---

> 创建时间：2026-04-22
> 最后更新：2026-04-22
> 学习进度：0 / 3
