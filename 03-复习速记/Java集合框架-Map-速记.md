# Java集合 Map - 速记

## 1. Map 是什么

`Map` 用来存 **键值对**：

```text
key -> value
```

特点：

- `key` 不能重复
- `value` 可以重复
- 它和 `Collection` 是并列体系，不是 `Collection` 的子接口

一句话：

> `Map` 适合做“通过一个键快速找到对应值”的数据存储。

---

## 2. 常见实现类

### `HashMap`

- 最常用
- 无序
- 查询和插入效率高
- 允许一个 `null` 键、多个 `null` 值

### `LinkedHashMap`

- 在 `HashMap` 基础上保留插入顺序
- 适合既想按键查值，又想保留遍历顺序的场景

底层理解：

- 本质上还是基于哈希表
- 只是每个节点额外维护了**双向链表**
- 通过双向链表把元素串起来，所以能记录顺序

一句话：

> `LinkedHashMap` = `HashMap` + 双向链表记录顺序

补充：

- 前面学过的 `LinkedHashSet`，底层本质上也是借助 `LinkedHashMap`
- 所以 `LinkedHashSet` 能保证有序，根源也在这里

### `TreeMap`

- 会按 `key` 排序
- 适合需要有序键的场景

补充理解：

- `TreeMap` 只对 **key** 排序
- `value` 不参与排序
- 仍然是 `key` 不重复、无索引

一句话：

> `TreeMap` 的核心特征是“按 key 排序”，不是按 value 排序。

最短记法：

```text
HashMap：常用、无序
LinkedHashMap：有插入顺序
TreeMap：按 key 排序
```

### 实现类底层原理补充

#### `HashMap`

- 底层是哈希表
- JDK 8 之前：`数组 + 链表`
- JDK 8 开始：`数组 + 链表 + 红黑树`

补充理解：

- `HashSet` 底层其实就是 `HashMap`
- `HashSet` 只使用键，不使用值

一句话：

> `HashMap` 查询快，核心原因是底层基于哈希表。

#### `LinkedHashMap`

- 底层仍然是哈希表
- 在哈希表节点基础上额外增加双向链表机制
- 常见能看到类似：
  - `head`
  - `tail`
  这样的头尾节点引用

它们的作用是：

> 记录元素的前后顺序，从而保证遍历时有序。

一句话：

> `LinkedHashMap` 之所以有序，不是因为哈希表会排序，而是因为它额外维护了双向链表。

#### `TreeMap`

- 底层通常基于红黑树
- 因此 key 会自动按规则排序

一句话：

> `TreeMap` 之所以有序，是因为它底层本来就是排序型结构。

### `TreeMap` 的排序规则

`TreeMap` 和前面学过的 `TreeSet` 很像，也支持两种指定排序规则的方式：

#### 方式 1：让 key 实现 `Comparable`

也就是让键对象自己具备比较规则。

适合：

- 这个类本身就有一个默认、稳定的排序标准

#### 方式 2：创建时传入 `Comparator`

例如：

```java
Map<Teacher, String> map = new TreeMap<>((o1, o2) -> Double.compare(o2.getSalary(), o1.getSalary()));
```

这表示：

- 在创建 `TreeMap` 时直接指定比较器
- 后续所有 key 都按这个比较规则排序

适合：

- 不方便改类源码
- 或者希望同一个类在不同场景下按不同规则排序

最短记法：

```text
TreeMap 排序两种方式：
1. Comparable
2. Comparator
```

一句话理解：

> `TreeMap` 的有序性来自红黑树，而“按什么规则排”可以通过 `Comparable` 或 `Comparator` 来决定。

---

## 3. 常用方法

### 添加

```java
map.put("name", "Tom");
```

### 获取

```java
map.get("name");
```

### 删除

```java
map.remove("name");
```

### 判断 key 是否存在

```java
map.containsKey("name");
```

### 判断 value 是否存在

```java
map.containsValue("Tom");
```

### 获取大小

```java
map.size();
```

### 判空

```java
map.isEmpty();
```

---

## 4. 遍历方式

### 方式 1：遍历 key

核心方法：

```java
map.keySet();   // 获取所有键的集合
map.get(key);   // 根据键获取对应值
```

思路：

1. 先通过 `keySet()` 拿到所有键
2. 再遍历每个 `key`
3. 最后通过 `get(key)` 找到对应的 `value`

```java
Set<String> keys = map.keySet();

for (String key : map.keySet()) {
    System.out.println(key + "=" + map.get(key));
}
```

也可以写成更完整的形式：

```java
Set<String> keys = map.keySet();
for (String key : keys) {
    Integer value = map.get(key);
    System.out.println(key + "=" + value);
}
```

特点：

- 好理解
- 很适合初学时掌握 `Map` 的“键找值”思路
- 但每次遍历时都要再调用一次 `get(key)`

### 方式 2：遍历 entry

核心方法：

```java
map.entrySet();   // 获取所有“键值对对象”的集合
```

这里要先理解：

> `Map` 中的每一组 `key-value`，都可以看成一个 `Map.Entry<K, V>` 对象。

也就是说：

- `key`
- `value`

这一对会被看成一个整体来遍历。

```java
for (Map.Entry<String, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + "=" + entry.getValue());
}
```

更完整的形式：

```java
Set<Map.Entry<String, Double>> entries = map.entrySet();
for (Map.Entry<String, Double> entry : entries) {
    String key = entry.getKey();
    Double value = entry.getValue();
    System.out.println(key + "=" + value);
}
```

特点：

- 直接把键和值作为一对来遍历
- 不需要再通过 `get(key)` 取值
- 是传统写法里更常用、更推荐的一种方式

### 方式 3：Lambda

```java
map.forEach((key, value) -> {
    System.out.println(key + "=" + value);
});
```

对应的方法本质上是：

```java
default void forEach(BiConsumer<? super K, ? super V> action)
```

也就是说，`Map` 会把每一组：

- `key`
- `value`

直接交给 Lambda 表达式来处理。

更常见的写法是：

```java
map.forEach((k, v) -> {
    System.out.println(k + "----->" + v);
});
```

特点：

- 写法最简洁
- 可读性好
- JDK 8 之后很常见
- 很适合日常快速遍历

最短记法：

```text
Lambda：forEach((k, v) -> ...)
```

最常用、也最顺手的通常是：

```text
entrySet 遍历
```

因为不用再次 `get(key)`。

如果从“代码最简洁”的角度看：

```text
Lambda 最简洁
```

---

## 5. `put` 的一个重要特点

如果 `key` 已经存在，再次 `put`：

```java
map.put("name", "Tom");
map.put("name", "Jerry");
```

结果是：

```text
后面的值覆盖前面的值
```

所以：

> `Map` 的“去重”是对 `key` 来说的，不是对 `value`。

---

## 6. 和今天业务代码的联系

今天员工信息统计里，我们看到过：

```java
List<Map<String, Object>>
```

它的意思是：

- `List`：多行数据
- `Map<String, Object>`：每一行里是“字段名 -> 字段值”

例如一行职位统计数据可以是：

```text
{pos=班主任, num=7}
```

然后再通过：

```java
dataMap.get("pos");
dataMap.get("num");
```

把数据取出来。

一句话：

> MyBatis 查询返回 `List<Map<String,Object>>` 时，可以把它理解成“多行结果，每行一个字段名到字段值的映射”。

---

## 7. 当前最该记住的点

```text
Map：存键值对，key 不能重复
HashMap：最常用，无序
LinkedHashMap：保留插入顺序
TreeMap：按 key 排序
常用遍历：entrySet
业务里 List<Map<String,Object>> = 多行结果 + 每行字段映射
```

---

## 8. 一句话总结

> `Map` 是 Java 里专门存键值对的数据结构，日常最常用的是 `HashMap`；在业务开发里，尤其数据库查询场景中，`List<Map<String,Object>>` 很常见。
