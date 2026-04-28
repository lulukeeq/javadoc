# SQL 子查询学习笔记

## 一、子查询概述

### 1. 什么是子查询

子查询，也叫嵌套查询，是指在一条 SQL 语句中嵌套另一条 `SELECT` 查询语句。

简单理解：

> 一个查询的结果，可以作为另一个 SQL 的条件或数据来源。

基本语法：

```sql
SELECT *
FROM t1
WHERE column1 = (
    SELECT column1
    FROM t2
);
```

子查询外层语句可以是：

- `SELECT`
- `INSERT`
- `UPDATE`
- `DELETE`

其中最常见的是 `SELECT`。

### 2. 子查询分类

根据子查询返回结果的不同，可以分为 4 类：

| 类型 | 返回结果 | 说明 |
|------|----------|------|
| 标量子查询 | 一行一列 | 返回单个值 |
| 列子查询 | 一列多行 | 返回一列数据 |
| 行子查询 | 一行多列 | 返回一行数据 |
| 表子查询 | 多行多列 | 返回一张临时表 |

### 3. 子查询出现的位置

子查询常见位置：

| 位置 | 作用 |
|------|------|
| `WHERE` 后 | 作为查询条件 |
| `FROM` 后 | 作为临时表 |
| `SELECT` 后 | 作为查询字段 |

### 4. 子查询解题思路

写子查询时，不要一开始就直接写复杂 SQL。

推荐步骤：

1. 先分析最终要查询什么
2. 把复杂需求拆成几个小查询
3. 先分别写出每一步 SQL
4. 再把其中一个 SQL 嵌套到另一个 SQL 中
5. 最后合并成一条完整 SQL

核心记法：

> 先拆分，再合并。

---

## 二、标量子查询

### 1. 概念

标量子查询指的是：子查询返回的结果是单个值，也就是一行一列。

返回值可以是：

- 数字
- 字符串
- 日期

常用操作符：

- `=`
- `<>`
- `>`
- `>=`
- `<`
- `<=`

### 2. 案例：查询最早入职的员工信息

第一步：查询最早的入职时间。

```sql
SELECT MIN(entry_date)
FROM emp;
```

第二步：根据最早入职时间查询员工信息。

```sql
SELECT *
FROM emp
WHERE entry_date = '2000-01-01';
```

第三步：合并成子查询。

```sql
SELECT *
FROM emp
WHERE entry_date = (
    SELECT MIN(entry_date)
    FROM emp
);
```

说明：

- `SELECT MIN(entry_date) FROM emp` 返回一个日期值
- 这个日期值作为外层 SQL 的查询条件
- 所以这个子查询属于标量子查询

### 3. 案例：查询在“阮小五”入职之后入职的员工信息

第一步：查询“阮小五”的入职日期。

```sql
SELECT entry_date
FROM emp
WHERE name = '阮小五';
```

第二步：查询在该日期之后入职的员工。

```sql
SELECT *
FROM emp
WHERE entry_date > '2015-01-01';
```

第三步：合并成子查询。

```sql
SELECT *
FROM emp
WHERE entry_date > (
    SELECT entry_date
    FROM emp
    WHERE name = '阮小五'
);
```

说明：

- 子查询返回“阮小五”的入职日期
- 外层查询用这个日期作为比较条件
- `>` 表示查询更晚入职的员工

---

## 三、列子查询

### 1. 概念

列子查询指的是：子查询返回的结果是一列数据，可以是一行，也可以是多行。

常见操作符：

- `IN`
- `NOT IN`
- `ANY`
- `SOME`
- `ALL`

### 2. 案例：查询“教研部”和“咨询部”的所有员工信息

第一步：查询“教研部”和“咨询部”的部门 ID。

```sql
SELECT id
FROM dept
WHERE name = '教研部'
   OR name = '咨询部';
```

第二步：根据部门 ID 查询员工信息。

```sql
SELECT *
FROM emp
WHERE dept_id IN (3, 2);
```

第三步：合并成子查询。

```sql
SELECT *
FROM emp
WHERE dept_id IN (
    SELECT id
    FROM dept
    WHERE name = '教研部'
       OR name = '咨询部'
);
```

说明：

- 子查询返回的是多个部门 ID
- 外层查询使用 `IN` 判断员工的 `dept_id` 是否在这些 ID 中
- 所以这个子查询属于列子查询

---

## 四、行子查询

### 1. 概念

行子查询指的是：子查询返回的结果是一行数据，但可以包含多列。

常用操作符：

- `=`
- `<>`
- `IN`
- `NOT IN`

### 2. 案例：查询与“李忠”的薪资和职位都相同的员工信息

第一步：查询“李忠”的薪资和职位。

```sql
SELECT salary, job
FROM emp
WHERE name = '李忠';
```

第二步：根据薪资和职位查询员工信息。

```sql
SELECT *
FROM emp
WHERE (salary, job) = (5000, 5);
```

第三步：合并成子查询。

```sql
SELECT *
FROM emp
WHERE (salary, job) = (
    SELECT salary, job
    FROM emp
    WHERE name = '李忠'
);
```

说明：

- 子查询返回一行两列：`salary` 和 `job`
- 外层查询也用 `(salary, job)` 两个字段进行比较
- 所以这个子查询属于行子查询

---

## 五、表子查询

### 1. 概念

表子查询指的是：子查询返回多行多列，结果可以当成一张临时表使用。

表子查询常见位置：

- `FROM` 后面

### 2. 案例：获取每个部门中薪资最高的员工信息

第一步：查询每个部门的最高薪资。

```sql
SELECT dept_id, MAX(salary) AS max_sal
FROM emp
GROUP BY dept_id;
```

第二步：把上一步结果作为临时表，再和员工表关联。

```sql
SELECT *
FROM emp e,
     (
         SELECT dept_id, MAX(salary) AS max_sal
         FROM emp
         GROUP BY dept_id
     ) a
WHERE e.dept_id = a.dept_id
  AND e.salary = a.max_sal;
```

说明：

- 子查询先查出每个部门的最高薪资
- 子查询结果作为临时表 `a`
- 外层查询通过部门 ID 和薪资匹配，找出每个部门薪资最高的员工
- 所以这个子查询属于表子查询

---

## 六、综合练习

### 1. 查询“教研部”中性别为男，且在 2011-05-01 之后入职的员工信息

```sql
SELECT e.*
FROM emp AS e, dept AS d
WHERE e.dept_id = d.id
  AND d.name = '教研部'
  AND e.gender = 1
  AND e.entry_date > '2011-05-01';
```

说明：

- 这题主要是多表查询
- 员工表和部门表通过 `dept_id` 关联
- 再根据部门、性别、入职时间进行筛选

### 2. 查询工资低于公司平均工资，且性别为男的员工信息

```sql
SELECT e.*
FROM emp AS e, dept AS d
WHERE e.dept_id = d.id
  AND e.salary < (
      SELECT AVG(salary)
      FROM emp
  )
  AND e.gender = 1;
```

说明：

- `SELECT AVG(salary) FROM emp` 返回公司平均工资
- 外层查询筛选工资低于平均工资的员工
- 这个子查询属于标量子查询

### 3. 查询部门人数超过 10 人的部门名称

```sql
SELECT d.name, COUNT(*)
FROM emp AS e, dept AS d
WHERE e.dept_id = d.id
GROUP BY d.name
HAVING COUNT(*) > 10;
```

说明：

- 这题主要是多表查询和分组查询
- `GROUP BY d.name` 表示按部门名称分组
- `HAVING COUNT(*) > 10` 表示筛选人数超过 10 人的部门

### 4. 查询在 2010-05-01 后入职，薪资高于 10000 的“教研部”员工，并按薪资倒序排序

```sql
SELECT *
FROM emp e, dept d
WHERE e.dept_id = d.id
  AND e.entry_date > '2010-05-01'
  AND e.salary > 10000
  AND d.name = '教研部'
ORDER BY e.salary DESC;
```

说明：

- 这题主要是多条件筛选
- `ORDER BY e.salary DESC` 表示按薪资从高到低排序

### 5. 查询工资低于本部门平均工资的员工信息

第一步：查询每个部门的平均工资。

```sql
SELECT dept_id, AVG(salary) AS avg_sal
FROM emp
GROUP BY dept_id;
```

第二步：把每个部门的平均工资作为临时表，查询低于本部门平均工资的员工。

```sql
SELECT e.*
FROM emp e,
     (
         SELECT dept_id, AVG(salary) AS avg_sal
         FROM emp
         GROUP BY dept_id
     ) AS a
WHERE e.dept_id = a.dept_id
  AND e.salary < a.avg_sal;
```

说明：

- 子查询先查出每个部门的平均工资
- 外层查询把员工和自己部门的平均工资进行比较
- 这个子查询属于表子查询

---

## 七、子查询和连接查询的区别

### 1. 子查询可以转换成其他查询吗

很多子查询都可以转换成其他查询方式，常见转换方式有：

- 转换成连接查询：`JOIN`
- 转换成分组查询：`GROUP BY`
- 转换成表子查询：把子查询结果当成临时表

不是所有子查询都必须转换，主要看 SQL 是否清晰、数据量是否大、执行效率是否合适。

### 2. 子查询和连接查询的区别

| 对比项 | 子查询 | 连接查询 |
|--------|--------|----------|
| 写法思路 | 先查出一个结果，再给外层 SQL 使用 | 多张表直接关联后一起查询 |
| 适合场景 | 需求可以拆成多个步骤时 | 多表字段需要一起展示或筛选时 |
| 可读性 | 拆步骤时更容易理解 | 表关系清楚时更直接 |
| 性能 | 简单子查询通常问题不大，相关子查询要注意 | 通常更容易被数据库优化 |

简单理解：

> 子查询更像是“先算一个结果，再拿去用”；连接查询更像是“先把表关联起来，再一起筛选”。

### 3. 非相关子查询

非相关子查询指的是：子查询可以单独执行，不依赖外层 SQL 的字段。

示例：查询工资低于公司平均工资的员工信息。

```sql
SELECT *
FROM emp
WHERE salary < (
    SELECT AVG(salary)
    FROM emp
);
```

说明：

- 子查询 `SELECT AVG(salary) FROM emp` 可以单独执行
- 子查询只需要计算一次公司平均工资
- 外层 SQL 再拿这个平均工资进行比较

这种子查询一般比较容易理解，性能问题也通常不明显。

### 4. 相关子查询

相关子查询指的是：子查询依赖外层 SQL 的字段。

示例：查询工资低于本部门平均工资的员工信息。

```sql
SELECT *
FROM emp e
WHERE salary < (
    SELECT AVG(salary)
    FROM emp
    WHERE dept_id = e.dept_id
);
```

说明：

- 子查询中使用了外层表的 `e.dept_id`
- 可以理解为：每查询一个员工，都要根据这个员工的部门去计算一次平均工资
- 数据量较大时，相关子查询可能会影响性能

### 5. 相关子查询转换为表子查询

上面的相关子查询，可以改成先查询每个部门的平均工资，再和员工表关联。

```sql
SELECT e.*
FROM emp e,
     (
         SELECT dept_id, AVG(salary) AS avg_sal
         FROM emp
         GROUP BY dept_id
     ) a
WHERE e.dept_id = a.dept_id
  AND e.salary < a.avg_sal;
```

说明：

- 子查询先按部门分组，算出每个部门的平均工资
- 外层查询把员工表和平均工资临时表关联
- 这样思路更清楚，也更容易被优化

### 6. 学习阶段怎么判断

先记住下面几个判断：

1. 子查询可以单独执行，通常是非相关子查询
2. 子查询引用了外层表字段，就是相关子查询
3. 相关子查询不一定都很差，但数据量大时要注意性能
4. 能用连接查询或表子查询清楚表达时，可以优先考虑转换
5. 真正判断性能时，需要看数据库的执行计划，例如 `EXPLAIN`

---

## 八、学习进度追踪

- [x] 子查询概述
- [x] 标量子查询
- [x] 列子查询
- [x] 行子查询
- [x] 表子查询
- [x] 综合练习
- [x] 子查询和连接查询的区别
- [x] 相关子查询和非相关子查询
- [ ] 子查询关键字：`ANY` / `SOME` / `ALL`
