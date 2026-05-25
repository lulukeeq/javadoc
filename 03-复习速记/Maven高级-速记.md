# Maven 高级速记

## 1. 主线

```text
分模块设计 -> 继承 -> 聚合 -> 私服
```

| 主题 | 解决的问题 |
|------|------------|
| 分模块设计 | 项目怎么按职责拆开 |
| 继承 | 公共配置和依赖版本怎么统一 |
| 聚合 | 多个模块怎么一次性构建 |
| 私服 | 公司内部 jar 包怎么共享 |

一句话：

> Maven 高级主要解决多模块项目管理和团队依赖共享。

---

## 2. 分模块设计

分模块设计就是把一个大项目拆成多个子模块。

常见结构：

```text
tlias-parent
├─ tlias-pojo
├─ tlias-utils
└─ tlias-web-management
```

核心记法：

```text
父工程负责管，子工程负责干。
```

重点：

1. 父工程通常不写业务代码
2. 子模块按职责拆分
3. 公共代码可以抽到公共模块
4. 模块之间要互相使用，仍然要写依赖

---

## 3. 继承

继承解决的是：

> 拆成多个模块后，公共配置不要重复写。

子工程通过 `parent` 继承父工程：

```xml
<parent>
    <groupId>com.itheima</groupId>
    <artifactId>tlias-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>
```

父工程适合统一管理：

1. `groupId`
2. `version`
3. Java 版本
4. 编码格式
5. 依赖版本
6. 插件版本

---

## 4. `dependencies` 和 `dependencyManagement`

| 配置 | 作用 |
|------|------|
| `dependencies` | 直接引入依赖，子工程会继承并使用 |
| `dependencyManagement` | 只管理版本，不会自动引入依赖 |

最短记法：

```text
dependencies：直接给依赖
dependencyManagement：只管版本，不给依赖
```

企业里更常用 `dependencyManagement`：

```text
父工程统一管版本，子模块按需引入。
```

---

## 5. 版本锁定

完整链路：

```text
properties 定义版本变量
-> dependencyManagement 统一锁版本
-> 子工程 dependencies 按需声明依赖
```

父工程：

```xml
<properties>
    <jjwt.version>0.9.1</jjwt.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

子工程：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
</dependency>
```

注意：子工程不写 `version`。

---

## 6. 聚合

聚合解决的是：

> 多模块项目怎么一键构建。

父工程中配置：

```xml
<packaging>pom</packaging>

<modules>
    <module>tlias-pojo</module>
    <module>tlias-utils</module>
    <module>tlias-web-management</module>
</modules>
```

关键点：

1. 聚合用 `modules`
2. 父工程通常是 `packaging=pom`
3. 聚合工程可以没有业务代码
4. Maven 会按模块依赖关系自动决定构建顺序

---

## 7. 继承和聚合的区别

| 对比项 | 继承 | 聚合 |
|--------|------|------|
| 关注点 | 复用配置 | 一起构建 |
| 配置位置 | 子工程写 `parent` | 父工程写 `modules` |
| 方向 | 子工程找父工程 | 父工程找子模块 |

最短记法：

```text
继承：子工程配 parent，向上拿配置
聚合：父工程配 modules，向下管构建
```

---

## 8. 私服

私服就是公司或团队内部搭建的 Maven 仓库服务器。

作用：

1. 保存公司内部 jar 包
2. 团队项目共享依赖
3. 缓存中央仓库中的第三方依赖
4. 统一管理企业内部依赖来源

依赖查找顺序：

```text
本地仓库 -> 私服 -> 中央仓库
```

---

## 9. release 和 snapshot

| 版本类型 | 示例 | 存放仓库 |
|----------|------|----------|
| RELEASE | `1.0.0` | `maven-releases` |
| SNAPSHOT | `1.0-SNAPSHOT` | `maven-snapshots` |

最短记法：

```text
带 SNAPSHOT -> snapshots
不带 SNAPSHOT -> releases
```

---

## 10. 上传和下载

`install` 和 `deploy`：

```text
install：安装到本地仓库
deploy：发布到远程私服
```

上传资源到私服：

```text
settings.xml 配 server 账号密码
-> pom.xml 配 distributionManagement 上传地址
-> 执行 mvn deploy
-> Nexus 私服中出现资源
```

从私服下载资源：

```text
settings.xml 配 mirror / profile
-> pom.xml 写 dependency 坐标
-> Maven 按本地仓库 -> 私服 -> 中央仓库查找
```

---

## 11. 私服配置速查

| 配置 | 常见位置 | 主要作用 |
|------|----------|----------|
| `server` | Maven `settings.xml` | 配置访问私服的账号密码 |
| `distributionManagement` | 项目 `pom.xml` | 配置项目发布到哪个私服仓库 |
| `mirror` | Maven `settings.xml` | 配置依赖下载时走哪个私服入口 |
| `profile` | Maven `settings.xml` | 配置 release / snapshot 等下载规则 |

最短记法：

```text
server 管账号
distributionManagement 管上传
mirror 管下载入口
profile 管下载规则
```

---

## 12. 高频问答

### `dependencyManagement` 会不会自动引入依赖？

不会。它只负责统一管理版本。子工程如果要用，仍然要在 `dependencies` 中声明。

### 聚合构建顺序由什么决定？

由模块之间的依赖关系决定，不是单纯由 `modules` 里的书写顺序决定。

### `1.0-SNAPSHOT` 执行 `deploy` 后上传到哪里？

上传到 `maven-snapshots`。

### `install` 和 `deploy` 有什么区别？

`install` 到本地仓库，`deploy` 到远程私服。

---

> 创建时间：2026-05-25
> 对应主笔记：`../02-学习笔记/Maven/Maven学习清单.md`
