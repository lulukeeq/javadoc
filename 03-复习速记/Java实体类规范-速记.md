# Java 实体类规范 - 速记

## 1. 类名用大驼峰

- 类名和业务含义对应
- 使用大驼峰命名

例如：

```java
Emp
EmpExpr
Dept
```

## 2. 字段名用小驼峰

- 数据库字段通常用下划线命名
- Java 实体类字段用小驼峰命名

例如：

```sql
dept_id
entry_date
update_time
```

对应：

```java
private Integer deptId;
private LocalDate entryDate;
private LocalDateTime updateTime;
```

## 3. 字段类型要合理

常见写法：

- `id`：`Integer` 或 `Long`
- `gender`：`Integer`
- `deptId`：`Integer`
- `entryDate`：`LocalDate`
- `createTime` / `updateTime`：`LocalDateTime`

## 4. 类和字段最好有简短注释

重点注释：

- 类的作用
- 字段含义
- 特殊取值说明

## 5. 类结构尽量完整

至少要有：

- 成员变量
- getter / setter

如果项目用了 Lombok，也要能看懂：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Emp {
}
```

## 6. 字段语义要和数据库一致

- 不要求名字完全一样
- 但含义必须一致

例如：

```sql
update_time
```

对应：

```java
updateTime
```

## 7. 学习阶段先把实体类当成数据载体

- 当前阶段主要作用是和数据库字段映射
- 先不要在实体类里塞太多复杂业务逻辑

## 8. 检查清单

- [ ] 类名是不是大驼峰
- [ ] 字段名是不是小驼峰
- [ ] 字段类型是不是合理
- [ ] 有没有必要的注释
- [ ] getter / setter 是否完整
- [ ] 和数据库字段含义是否一致
