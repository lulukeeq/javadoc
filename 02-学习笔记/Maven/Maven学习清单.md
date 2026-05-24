# Maven 学习清单

## 学习目录

### 一、Maven 核心

- Maven 概述（介绍、安装）
- IDEA 集成 Maven
- 依赖管理
- 单元测试

### 二、Maven 进阶

- 分模块设计
- 集成
- 聚合
- 私服

---

## 学习笔记

### 1. Maven 概述

#### 1.1 介绍

**Maven 是什么？**

> Maven 是一款项目管理和构建工具，基于项目对象模型（POM）。

**Maven 的三大作用：**

| 序号 | 作用         | 说明                                         |
| ---- | ------------ | -------------------------------------------- |
| 1    | 依赖管理     | 管理项目所需的第三方库和依赖                 |
| 2    | 项目构建     | 提供标准化的、跨平台的、自动化的项目构建方式 |
| 3    | 统一项目结构 | 提供统一的项目目录结构规范                   |

**Maven 的配置文件：**

> `pom.xml`（Project Object Model）

**Maven 仓库：**

> 仓库用于存储资源，管理各种 jar 包。分为三种：
>
> - **本地仓库**：本机上的仓库目录
> - **中央仓库**：Maven 社区维护的公共仓库
> - **远程仓库（私服）**：企业或团队内部搭建的仓库

> 📎 流程图参考：[`maven_doc.png`](maven_doc.png)

#### 1.2 安装

**Maven 安装步骤：**

1. **解压 zip 包** — 下载 Maven 安装包并解压到指定目录
2. **配置本地仓库** — 修改 `conf/settings.xml` 中的 `<localRepository>` 指定本地仓库路径
3. **配置阿里云私服** — 在 `settings.xml` 的 `<mirrors>` 中添加阿里云镜像地址，加速依赖下载
4. **配置环境变量** — 设置 `MAVEN_HOME` 并将 `%MAVEN_HOME%/bin` 添加到 `PATH`

### 2. IDEA 集成 Maven

**先决条件：** IDEA 全局配置 Maven 环境（全局）

**主要内容：**

1. **创建 Maven 项目** — 在 IDEA 中通过 Maven Archetype 创建新项目
2. **Maven 坐标** — 坐标是资源（jar）的唯一标识，通过该坐标可以唯一定位资源位置，使用坐标来定义项目或引入项目中需要的依赖。由三个元素组成：
   - `groupId`：定义当前 Maven 项目隶属组织名称（如 `com.example`）
   - `artifactId`：定义当前 Maven 项目名称（如 `my-project`）
   - `version`：定义当前项目版本号（如 `1.0.0`）
     - **SNAPSHOT**：开发中的版本，即快照版本
     - **RELEASE**：发布版本
3. **导入 Maven 项目** — 将已有 Maven 项目导入 IDEA，自动识别 POM 并下载依赖
   - 方式一：`File → Project Structure → Modules → Import Module → 选择 Maven 项目的 pom.xml`
   - 方式二：`Maven 面板 → +(Add Maven Projects) → 选择 Maven 项目的 pom.xml`

---

### 3. 依赖管理

#### 3.1 依赖配置

**依赖：** 指当前项目运行所需要的 jar 包，一个项目中可以引入多个依赖。

**配置步骤：**

1. 在 `pom.xml` 中编写 `<dependencies>` 标签
2. 在标签中使用 `<dependency>` 引入坐标
3. 定义坐标的 `groupId` / `artifactId` / `version`
4. 点击刷新按钮，引入最新加入的坐标

> 💡 **注意：** 如果不知道依赖的坐标信息，可以从 [Maven 中央仓库官网](https://mvnrepository.com/) 搜索。

#### 3.2 依赖传递

**Maven 有依赖传递：** 当项目依赖某个 jar 包时，该 jar 包所依赖的其他 jar 包也会被自动引入。

#### 3.3 排除依赖

如果不需要某个依赖时可以**排除依赖**：主动断开依赖的资源。

- 被排除的资源无需指定版本
- 使用 `<exclusions>` 和 `<exclusion>` 来排除依赖

#### 3.4 生命周期

**Maven 生命周期：** 是为了对所有的 Maven 项目构建过程中进行抽象和统一。

**Maven 中有 3 套独立的生命周期：**

| 生命周期    | 说明                                         |
| ----------- | -------------------------------------------- |
| **clean**   | 清理工作                                     |
| **default** | 核心工作，如：编译、测试、打包、安装、部署等 |
| **site**    | 生成报告、发布站点等                         |

**每套生命周期包含一些阶段（phase），阶段是有顺序的，后面的阶段依赖于前面的阶段。**

> ⚠️ **重要：** 在同一套生命周期中，当运行后面的阶段时，前面的阶段都会运行。

**常用阶段说明：**

| 生命周期 | 阶段      | 说明                               |
| -------- | --------- | ---------------------------------- |
| clean    | `clean`   | 移除上一次构建生成的文件           |
| default  | `compile` | 编译项目源代码                     |
| default  | `test`    | 使用合适的单元测试框架运行测试     |
| default  | `package` | 将编译后的文件打包，如 jar、war 等 |
| default  | `install` | 安装项目到本地仓库                 |

**运行阶段的方式：**

1. 使用 IDEA 面板点击运行
2. 使用命令行命令运行

---

### 4. 单元测试

**测试：** 是一种用来促进鉴定软件的正确性、完整性、安全性和质量的过程。

**阶段划分：**

| 阶段     | 说明                                                                  |
| -------- | --------------------------------------------------------------------- |
| 单元测试 | 目的：检验软件基本组成单位的正确性，由开发人员测试。使用 **白盒测试** |
| 集成测试 | 使用 **灰盒测试**                                                     |
| 系统测试 | 使用 **黑盒测试**                                                     |
| 验收测试 | 使用 **黑盒测试**                                                     |

**测试方法：**

| 方法     | 说明       |
| -------- | ---------- |
| 白盒测试 | （待补充） |
| 黑盒测试 | （待补充） |
| 灰盒测试 | （待补充） |

#### 4.1 单元测试内容

**单元测试：** 针对最小的功能单元（方法），编写测试代码对其正确性进行测试。

**JUnit：** 最流行的 Java 测试框架之一，提供了一些功能，方便程序进行单元测试。

**基于 JUnit 单元测试的好处：**

1. 测试代码与源代码分开，便于维护
2. 可根据需求进行自动化测试
3. 可自动分析测试结果，产出测试报告

- 快速入门
- 断言

> 通过断言可以检测方法运行结果是否和预期一致，从而判断业务方法的正确性。

**常用断言方法（`Assertions`）：**

| 方法              | 说明                           |
| ----------------- | ------------------------------ |
| `assertEquals`    | 检查两个值是否相等，不相等报错 |
| `assertNotEquals` | 检查两个值是否不相等，相等报错 |
| `assertNull`      | 检查值是否为 null              |
| `assertNotNull`   | 检查值是否不为 null            |
| `assertTrue`      | 检查条件是否为 true            |
| `assertFalse`     | 检查条件是否为 false           |
| `assertThrows`    | 检查是否抛出指定异常           |

- 常见注解

**JUnit 常见注解：**

| 注解                 | 说明                                                                 |
| -------------------- | -------------------------------------------------------------------- |
| `@Test`              | 标记一个方法为测试方法                                               |
| `@ParameterizedTest` | 标记一个方法为参数化测试（使用后不需要 `@Test`）                     |
| `@ValueSource`       | 为参数化测试提供参数来源，与 `@ParameterizedTest` 配合使用           |
| `@DisplayName`       | 为测试类或测试方法设置自定义显示名称，可以友好地提示当前测试的是什么 |
| `@BeforeEach`        | 在每个测试方法执行前执行（初始化）                                   |
| `@AfterEach`         | 在每个测试方法执行后执行（清理）                                     |
| `@BeforeAll`         | 在所有测试方法执行前执行一次（静态初始化），修饰**静态方法**         |
| `@AfterAll`          | 在所有测试方法执行后执行一次（静态清理），修饰**静态方法**           |

- 依赖范围

**Maven 依赖范围（scope）：**

> 依赖的 jar 包，默认情况下，可以在任何地方使用，可以通过 `<scope>` 设置作用范围。

**作用范围的三种维度：**

1. **主程序范围有效**（main 目录下的代码）
2. **测试程序范围有效**（test 目录下的代码）
3. **是否参与打包运行**（package 指令范围内）

**scope 取值说明：**

| scope 值   | 主程序(main) | 测试程序(test) | 打包(package) | 说明                                         |
| ---------- | :----------: | :------------: | :-----------: | -------------------------------------------- |
| `compile`  |      ✅      |       ✅       |      ✅       | 默认值，所有范围都有效                       |
| `test`     |      ❌      |       ✅       |      ❌       | 仅测试程序有效，如 JUnit                     |
| `provided` |      ✅      |       ✅       |      ❌       | 主程序和测试有效，不参与打包，如 servlet-api |
| `runtime`  |      ❌      |       ✅       |      ✅       | 测试和打包有效，如 JDBC 驱动                 |

#### 4.2 企业开发规范

1. **原则：** 编写测试方法时，要尽可能的覆盖业务方法中可能的情况
2. **统计测试覆盖率：** 使用 Coverage 工具统计
3. **配置统计范围：** 可以通过 Code Coverage 中配置统计的业务类范围
4. **使用 AI 编写单元测试：** 提高开发效率
5. **代码规范：** 在 Maven 项目中，test 是默认存放测试代码的目录，如果放到 main/ 下可以生效，但是不规范

---

### 5. Maven 常见问题

**问题：依赖变红，重新下载后还是失败**

**原因：** 可能是由于网络不好导致依赖下载不完整，在 Maven 仓库中残留了不完整的依赖文件（`*.lastUpdated` 文件）。

**解决方式：** 删除变红的 `**/*.lastUpdated` 文件，可以使用以下指令：

```bash
del /s *.lastUpdated
```

> 💡 如果删除后还报错，可以重新打开 IDEA。

> 💡 可以把 `del` 指令封装成 `del.bat` 脚本，方便重复使用。

---

### 6. 分模块设计

#### 6.1 为什么要分模块

当项目越来越大时，如果所有代码都堆在一个工程里，通常会出现这些问题：

- 代码职责混在一起，不容易维护
- 公共代码难复用
- 不同功能改动时互相影响
- 依赖越来越多，工程越来越臃肿

所以分模块设计的核心目标可以理解为：

> 按职责拆分工程，让每个模块只负责一类事情，再通过 Maven 统一管理它们之间的关系。

#### 6.2 什么是分模块设计

分模块设计，就是把一个完整项目拆成多个子模块。

例如一个简单 Web 项目，常见可以拆成：

- `pojo`：实体类、DTO、VO 等公共数据模型
- `utils`：工具类
- `dao` / `mapper`：数据访问层
- `service`：业务逻辑层
- `web`：Controller、启动类、接口层

这样做的本质不是“为了拆而拆”，而是为了让：

- 公共部分独立出来
- 业务层次更清晰
- 模块之间依赖关系更明确

#### 6.3 一个典型拆分例子

```text
tlias-parent
├─ tlias-pojo
├─ tlias-utils
└─ tlias-web-management
```

可以这样理解：

- `tlias-parent`：父工程，自己通常不写业务代码，主要做统一管理
- `tlias-pojo`：存放实体类
- `tlias-utils`：存放工具类
- `tlias-web-management`：真正运行的 Web 工程

一句话：

> 父工程负责“管”，子工程负责“干”。

#### 6.4 父工程主要管什么

父工程通常负责统一配置：

1. 统一依赖版本
2. 统一插件版本
3. 统一 Java 版本
4. 统一编码方式
5. 统一管理各个子模块

父工程最重要的两个标签通常是：

```xml
<packaging>pom</packaging>
```

表示父工程本身主要用于管理，不打成普通 jar。

```xml
<modules>
    <module>tlias-pojo</module>
    <module>tlias-utils</module>
    <module>tlias-web-management</module>
</modules>
```

表示当前父工程要管理哪些子模块。

#### 6.5 子工程和父工程的关系

子工程通常会通过 `parent` 标签继承父工程配置：

```xml
<parent>
    <groupId>com.itheima</groupId>
    <artifactId>tlias-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>
```

继承后，子工程就可以直接复用父工程里统一定义的内容。

例如：

- Java 版本
- 编码配置
- 依赖版本
- 插件版本

这就是“继承”的意义。

#### 6.6 分模块之后，模块之间怎么用

如果 `web` 模块要使用 `pojo` 模块里的实体类，或者使用 `utils` 模块里的工具类，本质上仍然是：

> 在当前模块的 `pom.xml` 中引入对应子模块依赖。

例如：

```xml
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>tlias-pojo</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

也就是说，拆成多个模块后，模块之间并不是自动可见的，仍然要显式声明依赖。

#### 6.7 分模块设计的常见好处

1. **便于维护**：每个模块职责更单一
2. **便于复用**：公共模块可以被多个业务模块重复使用
3. **便于协作**：多人开发时边界更清晰
4. **便于扩展**：新增功能时不容易把整个工程越改越乱

#### 6.8 分模块设计的常见原则

##### 按职责拆，不按随意感觉拆

要按“实体、工具、业务、接口”等职责拆分，不要拆成没有清晰边界的模块。

##### 公共部分优先抽取

只要多个模块都会用到，就适合抽到公共模块里。

##### 不要过度拆分

如果项目本来很小，却拆出很多模块，反而会增加维护成本。

一句话：

> 分模块设计追求的是“清晰”，不是“越多越高级”。

#### 6.9 当前阶段最该抓住的点

1. 父工程主要负责统一管理
2. 子工程主要承载具体业务代码
3. 父工程用 `packaging=pom`
4. 父工程用 `modules` 管理子模块
5. 子工程用 `parent` 继承父工程
6. 模块之间如果要互相使用，仍然要写依赖

#### 6.10 和后面“聚合”有什么关系

当前这一步“分模块设计”，更偏向：

> 把工程拆开，明确结构和职责。

后面的“聚合”更偏向：

> 在父工程上一键构建所有模块。

也就是说：

- 分模块设计：解决“怎么拆”
- 聚合：解决“怎么一起管、一起构建”

这两个概念相关，但不是一回事。

---

### 7. 继承

分模块设计之后，一个很自然的问题是：

> 如果每个子模块都要重复写一遍 Java 版本、依赖版本、插件配置，那分模块之后不是更麻烦了吗？

这时候就要用到 Maven 里的继承。

#### 7.1 为什么需要继承

在多模块项目里，很多配置其实是共通的：

- `groupId`
- `version`
- Java 版本
- 编码格式
- 依赖版本
- 插件版本

如果每个子模块都各写一遍，会出现这些问题：

1. 重复配置很多
2. 后续改版本很麻烦
3. 容易改漏，导致模块之间不一致

所以继承要解决的核心问题是：

> 把公共配置抽到父工程里，子工程统一复用。

#### 7.2 Maven 继承的基本写法

子工程通过 `parent` 标签继承父工程：

```xml
<parent>
    <groupId>com.itheima</groupId>
    <artifactId>tlias-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>
```

其中：

- `groupId`：父工程组织名
- `artifactId`：父工程项目名
- `version`：父工程版本号
- `relativePath`：父工程 `pom.xml` 的相对路径

一句话：

> 子工程认一个“父亲”，就能把父工程里的公共配置继承下来。

#### 7.3 继承之后通常能省掉什么

继承父工程后，子工程里很多重复信息就可以不写了。

例如子工程通常可以省掉：

- 自己重复写的 `groupId`
- 自己重复写的 `version`
- 重复的 Java 配置
- 重复的依赖版本

这样子工程的 `pom.xml` 会更短，也更聚焦自己真正需要的内容。

#### 7.4 父工程里最适合放什么

父工程最适合放“公共规则”，例如：

##### 统一属性

```xml
<properties>
    <java.version>17</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

##### 统一依赖版本

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.32</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

##### 统一插件配置

例如编译插件、SpringBoot 打包插件等，也适合在父工程里统一管理。

#### 7.5 版本锁定是什么

版本锁定，就是在父工程里统一规定依赖的版本。

这样子工程在使用依赖时，就不用每个模块都重复写版本号。

简单理解：

> 父工程先把版本定死，子工程只负责声明“我要用谁”。

例如父工程统一锁定 `jjwt` 的版本：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

子工程真正要用时，只需要写：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
</dependency>
```

注意这里没有写 `<version>`。

因为版本已经被父工程锁定了。

也就是说，子工程只需要指定：

- 组织名称：`groupId`
- 模块名称：`artifactId`

版本号交给父工程统一管理。

#### 7.6 为什么要版本锁定

多模块项目里，假设有三个子模块都要用同一个依赖。

如果每个模块自己写版本，可能会变成：

```text
tlias-web-management 用 0.9.1
tlias-utils          用 0.9.0
tlias-service        用 0.10.0
```

这样项目后期就很容易出现版本不一致的问题。

版本锁定的作用就是：

- 防止不同模块使用不同版本
- 后续升级依赖时只改父工程一处
- 让子模块的 `pom.xml` 更干净
- 方便团队统一依赖规范

一句话：

> 版本锁定不是为了少写几个字，而是为了让多模块项目的依赖版本保持一致。

#### 7.7 `dependencyManagement` 的真正作用

Maven 中通常用 `dependencyManagement` 做版本锁定。

它有两个核心特点：

1. 只负责管理版本
2. 不会自动把依赖引入子模块

这点非常重要。

也就是说，父工程里写了：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

并不代表所有子模块都能直接使用 `jjwt`。

哪个子模块要用，哪个子模块还要自己写：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
</dependency>
```

只是子模块不用再写版本号。

用课堂里的项目结构理解：

```text
tlias-parent
├─ tlias-pojo
├─ tlias-utils
├─ tlias-web-management
├─ tlias-web-system
└─ tlias-web-report
```

如果这几个子模块都需要使用 `jjwt`，不建议每个子模块各写一个版本号。

更合适的做法是：

1. 在 `tlias-parent` 的 `dependencyManagement` 中统一锁定 `jjwt` 版本
2. 哪个子模块要用 `jjwt`，哪个子模块在 `dependencies` 中声明 `groupId + artifactId`
3. 所有子模块最终都会使用父工程锁定的同一个版本

这样就实现了“统一管理多个依赖版本”。

#### 7.8 `dependencies` 和 `dependencyManagement` 的区别

这是 Maven 继承里最容易混的点之一。

##### `dependencies`

写在父工程里的 `dependencies`，子工程通常会直接继承并实际可用。

也就是说，它更像：

> 父工程不但规定了版本，还直接把依赖带给你了。

##### `dependencyManagement`

写在父工程里的 `dependencyManagement`，主要作用是：

> 统一“版本管理”，但不会自动让子工程真正引入这个依赖。

子工程如果想用，还得自己写：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

只是这里可以不用再写 `<version>`，因为版本由父工程统一管了。

一句话区分：

- `dependencies`：直接继承并参与使用
- `dependencyManagement`：只管版本，不自动引入

#### 7.9 为什么企业里更常用 `dependencyManagement`

因为它更灵活。

如果父工程把很多依赖都直接写进 `dependencies`，那子工程即使不用，也可能被一股脑带上。

而 `dependencyManagement` 的好处是：

- 版本统一
- 子模块按需引入
- 不容易让模块变臃肿

所以企业里更常见的思路是：

> 父工程统一管版本，子工程按需声明使用。

#### 7.10 版本锁定的完整理解

把版本锁定完整串起来，就是：

```text
父工程 dependencyManagement 锁版本
        ↓
子工程 dependencies 声明要使用哪个依赖
        ↓
子工程不写 version，自动使用父工程锁定的版本
```

这个过程里，父工程像一个“版本清单”，子工程像一个“点菜的人”。

父工程说：

> 这些菜分别用哪个版本。

子工程说：

> 我要其中哪几道。

最终子工程只会拿自己声明过的依赖。

#### 7.11 自定义属性和引用属性

版本锁定还可以继续优化。

如果版本号都直接写在依赖里，后面版本多了以后，父工程的 `pom.xml` 也会越来越难维护。

所以 Maven 中经常会先在 `properties` 中定义版本号，再在依赖里引用这个属性。

例如：

```xml
<properties>
    <lombok.version>1.18.30</lombok.version>
    <jjwt.version>0.9.1</jjwt.version>
</properties>
```

这里的：

- `lombok.version`
- `jjwt.version`

都是自定义属性名。

后面使用版本号时，就可以通过 `${属性名}` 引用。

例如在 `dependencyManagement` 中引用 `jjwt.version`：

```xml
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

例如在普通依赖中引用 `lombok.version`：

```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
    </dependency>
</dependencies>
```

一句话：

> `properties` 负责定义变量，`${...}` 负责引用变量。

#### 7.12 为什么要用属性引用版本号

直接写版本号也能用，但属性引用更适合大型项目。

好处是：

- 版本集中在 `properties` 中，一眼能看清
- 升级版本时只改一处
- 多个地方引用同一个版本时不容易改漏
- 父工程的依赖管理更整洁

例如以后想升级 `jjwt`：

```xml
<jjwt.version>0.9.1</jjwt.version>
```

只要改成：

```xml
<jjwt.version>0.9.2</jjwt.version>
```

所有引用 `${jjwt.version}` 的地方都会跟着变化。

#### 7.13 版本锁定的三层写法

把这节课里的写法按层次整理一下：

```text
第一层：properties 定义版本变量
第二层：dependencyManagement 使用变量锁定依赖版本
第三层：子工程 dependencies 声明要使用哪个依赖
```

对应关系：

```text
properties
  ↓ 提供版本号
dependencyManagement
  ↓ 统一锁版本
子工程 dependencies
  ↓ 按需使用依赖
```

所以完整理解应该是：

> 父工程用 `properties` 管版本号，用 `dependencyManagement` 锁依赖版本；子工程只声明自己要用哪些依赖。

#### 7.14 高频小结：`dependencyManagement` 与 `dependencies` 的区别

这是 149 里最容易被问到的问题。

##### 问题

`dependencyManagement` 和 `dependencies` 的区别是什么？

##### 答案

`dependencies` 是直接依赖。

如果在父工程中配置了 `dependencies`，子工程会直接继承这些依赖。

也就是说：

> 父工程写了依赖，子工程会直接拿到并使用。

`dependencyManagement` 是依赖版本管理。

如果在父工程中配置了 `dependencyManagement`，它只负责统一管理依赖版本，不会让子工程直接拥有这个依赖。

子工程如果需要使用，仍然要自己在 `dependencies` 中引入，只是不用再指定版本。

也就是说：

> 父工程只管版本，子工程按需引入。

##### 最短记法

```text
dependencies：直接给依赖
dependencyManagement：只管版本，不给依赖
```

##### 子工程使用方式

父工程锁版本：

```xml
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

子工程按需引入：

```xml
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
    </dependency>
</dependencies>
```

注意子工程这里不写 `<version>`。

#### 7.15 分模块设计和继承是什么关系

这两个概念经常一起出现，但它们解决的问题不一样：

- 分模块设计：解决“项目怎么拆”
- 继承：解决“拆完之后公共配置怎么统一管理”

简单理解：

> 分模块是结构拆分，继承是配置复用。

#### 7.16 当前阶段最该抓住的点

1. 继承的目的，是消除多模块项目里的重复配置
2. 子工程通过 `parent` 继承父工程
3. 父工程适合放公共属性、依赖版本、插件配置
4. `dependencyManagement` 只管版本，不会自动引入依赖
5. 版本锁定的核心是“父工程管版本，子工程按需使用”
6. `properties` 定义版本变量，`${...}` 引用版本变量
7. 分模块和继承通常一起出现，但不是一回事

#### 7.17 和明天的“聚合”怎么衔接

前面两节主线可以压成：

- 148 分模块设计：先把工程拆开
- 149 继承：再把公共配置统一起来

150“聚合”会继续解决：

> 这些拆开的模块，怎么一键一起构建。

所以顺序非常自然：

```text
分模块设计
-> 继承
-> 聚合
-> 私服
```

---

### 8. 聚合

学完分模块和继承后，项目已经被拆成多个子模块了。

接下来会遇到一个新问题：

> 模块拆开以后，如果每个模块都要单独执行 `clean`、`package`、`install`，会很麻烦。

这时候就需要 Maven 的聚合。

#### 8.1 聚合解决什么问题

聚合解决的是：

> 在父工程中一次性构建所有子模块。

比如项目结构是：

```text
tlias-parent
├─ tlias-pojo
├─ tlias-utils
├─ tlias-web-management
├─ tlias-web-system
└─ tlias-web-report
```

如果没有聚合，你可能要分别构建这些模块。

有了聚合后，只需要在父工程上执行 Maven 命令，Maven 就会按照 `modules` 中声明的模块统一构建。

更正式一点说：

> 聚合就是将多个模块组织成一个整体，同时进行项目构建。

#### 8.2 聚合的基本写法

聚合通常写在父工程 `pom.xml` 中：

```xml
<modules>
    <module>tlias-pojo</module>
    <module>tlias-utils</module>
    <module>tlias-web-management</module>
    <module>tlias-web-system</module>
    <module>tlias-web-report</module>
</modules>
```

这里的每一个 `module`，对应一个子模块路径。

如果子模块在父工程内部，通常直接写目录名。

如果子模块和聚合工程是同级目录，也可能写成：

```xml
<modules>
    <module>../tlias-pojo</module>
    <module>../tlias-utils</module>
    <module>../tlias-web-management</module>
</modules>
```

一句话：

> `modules` 告诉 Maven：当前父工程下面有哪些模块需要一起构建。

#### 8.3 什么是聚合工程

聚合工程通常是一个不具有业务功能的“空工程”。

它可以没有 `src` 目录，但需要有一个 `pom.xml`。

它的主要职责是：

- 组织多个子模块
- 统一触发多个模块构建
- 作为整个多模块项目的构建入口

#### 8.4 父工程为什么通常是 `pom` 打包

聚合工程本身一般不写业务代码，它主要负责管理子模块。

所以父工程通常配置：

```xml
<packaging>pom</packaging>
```

这表示：

> 当前工程不是普通 jar / war，而是一个负责管理和聚合的父工程。

#### 8.5 聚合和继承的区别

这两个特别容易混。

##### 继承

继承关注的是：

> 子工程从父工程那里继承公共配置。

关键词：

- `parent`
- 复用配置
- 统一版本

##### 聚合

聚合关注的是：

> 父工程一次性管理并构建多个子模块。

关键词：

- `modules`
- 一键构建
- 批量管理

最短记法：

```text
继承：子工程找父工程要配置
聚合：父工程找子工程一起构建
```

#### 8.6 继承和聚合可以同时存在

实际多模块项目里，父工程通常同时承担两个角色：

1. 被子模块继承
2. 聚合多个子模块

也就是说，同一个 `tlias-parent` 可以同时写：

```xml
<packaging>pom</packaging>

<modules>
    <module>tlias-pojo</module>
    <module>tlias-utils</module>
    <module>tlias-web-management</module>
</modules>
```

子模块里再写：

```xml
<parent>
    <groupId>com.itheima</groupId>
    <artifactId>tlias-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
</parent>
```

这样父工程既能统一管理配置，也能统一构建子模块。

#### 8.7 聚合的构建顺序

聚合构建时，Maven 会根据模块之间的依赖关系，自动决定合理的构建顺序。

例如：

```text
tlias-web-management 依赖 tlias-pojo
```

那么 Maven 会先构建 `tlias-pojo`，再构建 `tlias-web-management`。

也就是说：

> 聚合不是简单按肉眼顺序乱构建，Maven 会结合依赖关系处理顺序。

注意：

> 聚合工程中包含的模块，在构建时会自动根据模块之间的依赖关系设置构建顺序，与聚合工程中模块的配置书写位置无关。

#### 8.8 当前阶段最该抓住的点

1. 聚合解决“一次性构建多个模块”的问题
2. 聚合使用父工程中的 `modules`
3. `module` 写的是子模块路径，常见是子模块目录名
4. 父工程通常是 `<packaging>pom</packaging>`
5. 聚合工程通常没有业务功能，可以只有一个 `pom.xml`
6. 继承解决配置复用，聚合解决统一构建
7. 构建顺序由模块依赖关系决定，不由 `modules` 的书写顺序决定
8. 一个父工程可以同时承担继承和聚合两个角色

#### 8.9 高频小结：聚合怎么实现

##### 问题

Maven 中如何实现聚合关系？

##### 答案

在聚合工程的 `pom.xml` 中配置：

```xml
<modules>
    <module>tlias-pojo</module>
    <module>tlias-utils</module>
    <module>tlias-web-management</module>
</modules>
```

也就是说：

> Maven 通过 `<modules>...</modules>` 实现聚合关系。

#### 8.10 高频小结：继承与聚合的联系和区别

##### 联系

继承和聚合都属于 Maven 多模块项目中的设计方式。

父工程通常使用：

```xml
<packaging>pom</packaging>
```

实际开发中，继承关系和聚合关系经常写在同一个父工程 `pom.xml` 中。

##### 区别

继承用于简化依赖配置、统一管理依赖版本。

配置位置：

> 子工程中配置 `parent`，表示继承哪个父工程。

聚合用于快速构建项目。

配置位置：

> 父工程，也就是聚合工程中配置 `modules`，表示要聚合哪些子模块。

最短记法：

```text
继承：子工程配 parent，向上找父工程拿配置
聚合：父工程配 modules，向下找子模块一起构建
```

---

### 9. 私服

学完分模块、继承、聚合后，项目内部模块之间已经能比较好地组织起来。

接下来会遇到另一个企业开发里的问题：

> 公司内部自己开发的 jar 包，应该放在哪里，其他项目又该怎么使用？

这时候就要用到 Maven 私服。

#### 9.1 什么是私服

私服就是公司或团队内部搭建的 Maven 仓库服务器。

它不是 Maven 中央仓库，而是企业内部自己的远程仓库。

一句话：

> 私服就是公司内部的 Maven 资源仓库。

#### 9.2 为什么需要私服

如果没有私服，公司内部公共模块想被其他项目使用，会比较麻烦。

例如：

- 公共工具包
- 公共实体类
- 统一认证模块
- 公司内部 starter

这些 jar 包不适合上传到 Maven 中央仓库，但又希望团队内部多个项目可以复用。

私服的作用就是：

- 保存公司内部 jar 包
- 让团队项目共享依赖
- 缓存中央仓库中的第三方依赖
- 统一管理企业内部依赖来源

#### 9.3 Maven 仓库回顾

Maven 仓库可以分为：

| 仓库 | 说明 |
|------|------|
| 本地仓库 | 当前电脑上的 Maven 仓库 |
| 中央仓库 | Maven 官方公共仓库 |
| 远程仓库 / 私服 | 公司或团队内部搭建的仓库 |

查找依赖时，通常可以理解成：

```text
先找本地仓库
找不到再找远程仓库 / 私服
私服没有时，再去中央仓库下载并缓存
```

#### 9.4 私服里的常见仓库类型

企业私服里通常会分几类仓库。

##### release 仓库

用于保存正式发布版本。

例如：

```text
1.0.0
1.2.3
```

这类版本相对稳定，发布后通常不随意改。

##### snapshot 仓库

用于保存开发中的快照版本。

例如：

```text
1.0-SNAPSHOT
```

这类版本还在开发中，可能会频繁更新。

##### central 代理仓库

用于代理 Maven 中央仓库。

项目下载第三方依赖时，可以先从公司私服拿。

如果私服没有，再由私服去中央仓库下载并缓存下来。

#### 9.5 私服的上传和下载

私服使用时，通常涉及两个方向：

##### 下载依赖

项目需要某个依赖时：

```text
项目
-> 本地仓库
-> 私服
-> 中央仓库
```

如果私服已经缓存过这个依赖，后续团队成员下载会更快。

##### 上传依赖

公司内部开发的公共模块，打包后可以上传到私服。

其他项目再通过 Maven 坐标引入。

例如：

```xml
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>company-utils</artifactId>
    <version>1.0.0</version>
</dependency>
```

#### 9.6 当前阶段先抓住的点

1. 私服是公司内部 Maven 仓库
2. 私服可以保存公司内部 jar 包
3. 私服也可以缓存中央仓库的第三方依赖
4. `RELEASE` 版本通常放正式仓库
5. `SNAPSHOT` 版本通常放快照仓库
6. 私服解决的是“团队内部依赖共享和统一管理”的问题

---

## 学习进度追踪

### 一、Maven 核心

- [x] Maven 概述 — 介绍
- [x] Maven 概述 — 安装
- [x] IDEA 集成 Maven
- [x] 依赖管理
- [x] 单元测试

### 二、Maven 进阶

- [x] 分模块设计
- [x] 继承
- [x] 聚合
- [ ] 私服

---

> 创建时间：2026-04-21
> 最后更新：2026-05-24
> 学习进度：3 / 9
