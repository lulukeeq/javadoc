# 05-常用片段 说明

`05-常用片段/` 放的是可以反复复用的模板，不是完整学习笔记，也不是练习代码。

适合放这里的内容：

- 常用 SQL 模板
- `try-catch-finally` 模板
- JDBC 连接模板
- Maven 常用命令
- 以后经常复制再改的代码骨架

不适合放这里的内容：

- 长篇知识解释
- 当天学习计划
- 完整练习项目
- 老师原始资料

可以把这个目录理解成：

> 以后再遇到类似题目时，可以先来这里抄一个靠谱起手式。

举例：

```sql
-- 内连接模板
SELECT a.xx, b.yy
FROM table_a a
INNER JOIN table_b b
ON a.id = b.a_id;

-- 左连接模板
SELECT a.xx, b.yy
FROM table_a a
LEFT JOIN table_b b
ON a.id = b.a_id;

-- 分组统计模板
SELECT a.type, COUNT(b.id)
FROM table_a a
LEFT JOIN table_b b
ON a.id = b.a_id
GROUP BY a.id, a.type;
```
