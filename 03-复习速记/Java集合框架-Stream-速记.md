# Java集合框架 - Stream - 速记

## 1. Stream 是什么

`Stream` 是 JDK 8 开始新增的一套 API，主要在：

```java
java.util.stream.*
```

它的作用不是存数据，而是：

> 用来操作集合或者数组中的数据。

也就是说：

- `Collection / Map / 数组`：负责存数据
- `Stream`：负责处理数据

最短记法：

```text
集合存数据
Stream 处理数据
```

---

## 2. Stream 的优势

`Stream` 大量结合了 `Lambda` 的写法风格。

常见优势：

- 功能强
- 写法简洁
- 可读性较好
- 很适合做筛选、转换、统计、收集

一句话：

> `Stream` 让“处理一批数据”这件事，写起来比传统 `for` 循环更简洁。

---

## 3. 当前阶段先这样理解

如果以前我们处理集合数据，常常会写很多：

```java
for (...)
if (...)
```

而 `Stream` 更常见的写法会变成：

- `filter(...)`
- `map(...)`
- `forEach(...)`
- `collect(...)`

所以可以先把 `Stream` 理解成：

> 面向“数据处理流程”的写法。

---

## 4. 一个典型场景

例如有一个姓名集合：

```java
List<String> list = new ArrayList<>();
list.add("张无忌");
list.add("周芷若");
list.add("赵敏");
list.add("张强");
list.add("张三丰");
```

需求是：

> 把集合中所有以“张”开头，并且是 3 个字的元素，存到一个新的集合里。

这个例子非常适合后面用来体验 `Stream` 的筛选能力。

---

## 5. 一句话总结

> `Stream` 是 JDK 8 提供的一套数据处理 API，适合对集合或数组进行筛选、转换、遍历和收集。

---

## 6. Stream 的使用步骤

`Stream` 的使用过程可以先简单记成 3 步：

### 1. 获取 `Stream` 流

先从数据源中拿到一条流。

常见数据源有：

- 集合
- 数组
- 其他可生成流的数据源

这一步的本质是：

> 让数据源和 `Stream` 建立连接。

### 2. 调用中间方法处理数据

在流上继续做各种处理，例如：

- 过滤
- 排序
- 去重
- 转换

也就是你图里这条处理链：

```text
数据源 -> 过滤 -> 排序 -> 去重 -> ...
```

这一阶段的特点是：

> 主要负责“加工数据”，通常还没有真正拿到最终结果。

### 3. 调用终结方法获取结果

最后再通过终结操作拿到处理后的结果，例如：

- 遍历
- 统计
- 收集到新集合

这一步才真正把前面的处理结果落下来。

最短记法：

```text
1. 获取流
2. 中间处理
3. 终结取结果
```

一句话理解：

> `Stream` 的核心流程就是：先拿到流，再沿着流水线处理数据，最后通过终结操作拿到结果。

---

## 7. 怎么获取 `Stream`

`Stream` 常见的来源主要有三类：

1. 集合
2. `Map`
3. 数组

### 1. 获取集合的 `Stream`

`Collection` 提供了：

```java
default Stream<E> stream()
```

作用：

> 获取当前集合对象的 `Stream` 流。

例如：

```java
List<String> list = new ArrayList<>();
Stream<String> s = list.stream();
```

最短记法：

```text
集合拿流：集合对象.stream()
```

### 2. 获取数组的 `Stream`

数组常见有两种拿流方式。

#### 方式 A：`Arrays.stream(...)`

```java
public static <T> Stream<T> stream(T[] array)
```

例如：

```java
String[] arr = {"张无忌", "周芷若", "赵敏"};
Stream<String> s = Arrays.stream(arr);
```

#### 方式 B：`Stream.of(...)`

```java
public static<T> Stream<T> of(T... values)
```

例如：

```java
Stream<String> s = Stream.of("张无忌", "周芷若", "赵敏");
```

也可以直接接数组：

```java
String[] arr = {"张无忌", "周芷若", "赵敏"};
Stream<String> s = Stream.of(arr);
```

最短记法：

```text
数组拿流：Arrays.stream(arr) / Stream.of(arr)
```

### 3. 获取 `Map` 的 `Stream`

`Map` 本身不能直接调用：

```java
map.stream()
```

因为 `Map` 不是 `Collection`，所以通常要先拿它的视图集合，再转成流。

#### 方式 A：对键拿流

```java
Stream<String> s = map.keySet().stream();
```

适合：

- 只关心所有 `key`

#### 方式 B：对值拿流

```java
Stream<Integer> s = map.values().stream();
```

适合：

- 只关心所有 `value`

#### 方式 C：对键值对拿流

```java
Stream<Map.Entry<String, Integer>> s = map.entrySet().stream();
```

适合：

- 同时要处理 `key` 和 `value`

这也是 `Map` 场景里最常见、最完整的一种拿流方式。

最短记法：

```text
Map 拿流：keySet().stream() / values().stream() / entrySet().stream()
```

---

## 8. 当前阶段的速记小结

```text
Stream 是处理数据的 API，结合 Lambda
使用步骤：获取流 -> 中间处理 -> 终结取结果
Map 拿流：keySet().stream() / values().stream() / entrySet().stream()
集合拿流：stream()
数组拿流：Arrays.stream() / Stream.of()
```

---

## 9. Stream 的常用中间方法

中间方法的共同特点是：

- 调用后不会立刻拿到最终结果
- 返回的还是新的 `Stream`
- 可以继续链式调用

也就是说，它们主要负责：

> 对流中的数据继续加工处理。

### 1. `filter`

```java
Stream<T> filter(Predicate<? super T> predicate)
```

作用：

> 对流中的数据进行过滤，保留满足条件的元素。

例如：

```java
list.stream().filter(name -> name.startsWith("张"));
```

### 2. `sorted()`

```java
Stream<T> sorted()
```

作用：

> 对元素进行默认排序。

前提通常是元素本身支持比较。

### 3. `sorted(Comparator)`

```java
Stream<T> sorted(Comparator<? super T> comparator)
```

作用：

> 按照指定规则排序。

例如：

```java
list.stream().sorted((a, b) -> a.length() - b.length());
```

### 4. `limit`

```java
Stream<T> limit(long maxSize)
```

作用：

> 获取前几个元素。

### 5. `skip`

```java
Stream<T> skip(long n)
```

作用：

> 跳过前几个元素。

### 6. `distinct`

```java
Stream<T> distinct()
```

作用：

> 对流中元素去重。

### 7. `map`

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```

作用：

> 对元素进行加工或转换，并返回对应的新流。

例如：

```java
list.stream().map(String::length);
```

### 8. `concat`

```java
static <T> Stream<T> concat(Stream a, Stream b)
```

作用：

> 合并两个流为一个流。

---

## 10. 当前阶段的速记小结

```text
中间方法：处理完后返回新的 Stream，可继续链式调用
filter：过滤
sorted：排序
limit：取前几个
skip：跳过前几个
distinct：去重
map：转换
concat：合并两个流
```

---

## 11. 中间方法的链式示例

`Stream` 的一个很大特点就是：

> 可以把多个中间方法像流水线一样串起来。

例如有一组成绩：

```java
List<Double> scores = new ArrayList<>();
scores.add(88.8);
scores.add(66.6);
scores.add(66.6);
scores.add(77.6);
scores.add(77.6);
scores.add(99.6);
```

### 1. 默认升序排序

```java
scores.stream()
        .sorted()
        .forEach(System.out::println);
```

### 2. 自定义降序排序

```java
scores.stream()
        .sorted((s1, s2) -> Double.compare(s2, s1))
        .forEach(System.out::println);
```

### 3. 先降序，再取前 2 个

```java
scores.stream()
        .sorted((s1, s2) -> Double.compare(s2, s1))
        .limit(2)
        .forEach(System.out::println);
```

### 4. 先降序，再跳过前 2 个

```java
scores.stream()
        .sorted((s1, s2) -> Double.compare(s2, s1))
        .skip(2)
        .forEach(System.out::println);
```

### 5. 先排序，再去重

```java
scores.stream()
        .sorted((s1, s2) -> Double.compare(s2, s1))
        .distinct()
        .forEach(System.out::println);
```

这里也能顺手看出一个典型链路：

```text
排序 -> 截取 / 跳过 / 去重 -> 终结遍历
```

---

## 12. `map` 与 `concat` 的直观例子

### 1. `map`：把元素加工成新的结果

例如把成绩统一加 10 分并拼接提示：

```java
scores.stream()
        .map(s -> "加10分后：" + (s + 10))
        .forEach(System.out::println);
```

这里的重点是：

> `map` 会把原来的元素转换成新的结果，并返回一个新的流。

它既可以：

- 数值转换
- 字符串加工
- 对象字段提取

### 2. `concat`：把两个流合成一个流

例如：

```java
Stream<String> s1 = Stream.of("张三丰", "张无忌", "张翠山", "张良", "张安友");
Stream<Integer> s2 = Stream.of(11, 22, 33, 44);
Stream<Object> s3 = Stream.concat(s1, s2);

System.out.println(s3.count());
```

这里表示：

- `s1` 是一个字符串流
- `s2` 是一个整数流
- `concat(s1, s2)` 把两个流合成一个新流

注意：

> 合并后的流元素类型通常要兼容，因此这里用了 `Stream<Object>` 来接。

---

## 13. 这一段最短记法

```text
Stream 常常是链式写法
sorted 后可以继续 limit / skip / distinct
map：把元素转换成新结果
concat：合并两个流
中间方法负责加工，forEach/count/collect 这类终结方法才真正拿结果
```

---

## 14. `Stream` 的终结方法

终结方法的特点是：

> 调用完成后，不会再返回新的 `Stream`，所以不能继续往后链式处理。

也就是说：

- 中间方法：返回 `Stream`
- 终结方法：返回结果，或者直接执行动作

### 1. `forEach`

```java
void forEach(Consumer<? super T> action)
```

作用：

- 对流中的每个元素执行遍历操作

例如：

```java
scores.stream()
        .forEach(System.out::println);
```

### 2. `count`

```java
long count()
```

作用：

- 统计当前流中元素的个数

例如：

```java
long count = scores.stream().count();
```

### 3. `max`

```java
Optional<T> max(Comparator<? super T> comparator)
```

作用：

- 获取流中最大值元素

例如：

```java
Optional<Double> max = scores.stream()
        .max((s1, s2) -> Double.compare(s1, s2));
```

### 4. `min`

```java
Optional<T> min(Comparator<? super T> comparator)
```

作用：

- 获取流中最小值元素

例如：

```java
Optional<Double> min = scores.stream()
        .min((s1, s2) -> Double.compare(s1, s2));
```

### 为什么 `max` / `min` 返回 `Optional`

因为流有可能是空的。

所以 `Stream` 不会直接返回一个确定存在的元素，而是返回：

```java
Optional<T>
```

表示：

- 可能有值
- 也可能没值

---

## 15. 终结方法最短记法

```text
forEach：遍历每个元素
count：统计个数
max：取最大值
min：取最小值
终结方法一旦调用，流就结束了
```

---

## 16. 收集 `Stream` 流

收集 `Stream`，就是把流处理后的结果：

- 收回到集合中
- 或收回到数组中

也就是说：

> `Stream` 是处理数据的手段，集合/数组才是很多业务里最终真正要用的结果。

### 先记一个很重要的小点

> **同一条 `Stream` 一般只能终结一次。**

也就是说：

- 你对一条流调用了 `collect()` 之后
- 就不要再继续对这条同一个流对象调用别的终结方法

例如下面这样是两条新流，各自独立：

```java
List<String> list1 = list.stream()
        .filter(s -> s.startsWith("张"))
        .collect(Collectors.toList());

Set<String> set1 = list.stream()
        .filter(s -> s.startsWith("张"))
        .collect(Collectors.toSet());
```

而不是拿同一个 `Stream` 反复终结。

### 1. `collect(Collector collector)`

```java
R collect(Collector collector)
```

作用：

- 把流处理后的结果收集到指定集合结构中

常见写法都要配合 `Collectors` 工具类。

### 2. `toArray()`

```java
Object[] toArray()
```

作用：

- 把流处理后的结果收集到数组中

例如：

```java
Object[] arr = names.stream()
        .filter(name -> name.startsWith("张"))
        .toArray();
```

---

## 17. `Collectors` 常见收集方式

### 1. 收集到 `List`

```java
Collectors.toList()
```

例如：

```java
List<String> list = names.stream()
        .filter(name -> name.startsWith("张"))
        .collect(Collectors.toList());
```

### 2. 收集到 `Set`

```java
Collectors.toSet()
```

例如：

```java
Set<String> set = names.stream()
        .filter(name -> name.startsWith("张"))
        .collect(Collectors.toSet());
```

### 3. 收集到 `Map`

```java
Collectors.toMap(keyMapper, valueMapper)
```

例如：

```java
Map<String, Integer> map = names.stream()
        .collect(Collectors.toMap(name -> name, String::length));
```

这里表示：

- `key`：姓名本身
- `value`：姓名长度

如果是对象集合，也很常见写成：

```java
Map<String, Double> map = teachers.stream()
        .collect(Collectors.toMap(Teacher::getName, Teacher::getSalary));
```

这里表示：

- `key`：老师姓名
- `value`：老师薪水

---

## 18. 收集阶段最短记法

```text
collect：把流结果收回集合
toArray：把流结果收回数组
Collectors.toList()：收成 List
Collectors.toSet()：收成 Set
Collectors.toMap()：收成 Map
```

---

## 19. Stream 一整条主线

```text
认识 Stream
-> 获取 Stream
-> 中间方法处理数据
-> 终结方法获取结果
-> collect / toArray 收回集合或数组
```

---

## 20. `Collections` 工具类补充

`Collections` 是一个**操作集合的工具类**。

注意区分：

- `Collection`：集合体系中的接口
- `Collections`：操作集合的工具类

### 常见静态方法

#### 1. `addAll`

```java
public static <T> boolean addAll(Collection<? super T> c, T... elements)
```

作用：

- 给集合批量添加元素

例如：

```java
Collections.addAll(list, "张无忌", "周芷若", "赵敏", "张强", "张三丰", "张翠山");
```

#### 2. `shuffle`

```java
public static void shuffle(List<?> list)
```

作用：

- 打乱 `List` 集合中的元素顺序

例如：

```java
Collections.shuffle(list);
```

#### 3. `sort(List<T> list)`

```java
public static <T> void sort(List<T> list)
```

作用：

- 按默认规则对 `List` 排序

注意：

> 如果是自定义对象，通常需要实现 `Comparable` 接口，才能直接按默认规则排序。

#### 4. `sort(List<T> list, Comparator<? super T> c)`

```java
public static <T> void sort(List<T> list, Comparator<? super T> c)
```

作用：

- 按比较器指定的规则对 `List` 排序

例如：

```java
Collections.sort(list, (o1, o2) -> o2 - o1);
```

### 一个小提醒

> `Collections` 里的排序方法，当前阶段先重点记住它主要是对 `List` 进行排序。

---

## 21. 集合框架补充最短记法

```text
Collection：接口
Collections：工具类
addAll：批量加元素
shuffle：打乱顺序
sort(list)：按默认规则排序
sort(list, comparator)：按指定规则排序
```
