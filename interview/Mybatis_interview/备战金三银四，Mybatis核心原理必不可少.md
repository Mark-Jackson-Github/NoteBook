# 备战金三银四，Mybatis核心原理必不可少

## 带着问题思考本次分享

2020 下半年在公司组织了一场关于 Mybatis 核心原理的分享，掌握底层源码不仅能够更好的排查问题，同时也能够借鉴其中优秀的设计。正好赶上金三银四面试季，这里就梳理出以下几个问题供大家参考

- Mybatis 与 JDBC 的关系
- .xml 文件定义 SQL 语句如何解析
- Mybatis 中 Mapper 接口的存储与实现
- Mybatis SQL 的执行过程
- Mybatis 中分页如何实现

> 本次分享内容依据 Mybatis-3-3.4.x 源码

## 持久层的那些事

#### 什么是 JDBC

JDBC（JavaDataBase Connectivity）就是 Java 数据库连接, 说的直白点就是 **使用 Java 语言操作数据库**

本来我们是通过控制台或客户端操作的数据库, JDBC 是用 Java 语言来发送 SQL 语句

#### JDBC 原理

最初 SUN 公司希望提供一套 **能够适用所有数据库的 API**, 但是在实际操作中却发现这是项基本不可能完成的任务

因为各个厂商所提供的 **数据库差异实在太大**, 所以 SUN 公司与数据库厂商讨论出的就是：由 **SUN 公司提供出一套访问数据库的规范 API**, 并提供相对应的连接数据库协议标准, 然后各厂商根据规范提供一套访问自家数据库的 API 接口

最终：SUN 公司提供的规范 API 称之为 **JDBC**, 各厂商提供的自家数据库 API 接口称之为 **驱动**

![JDBC 架构图](/Users/mark/typora_workspace/NoteBook/images/46cd0f1cc80a452eb5b513f5321f5d71~tplv-k3u1fbpfcp-watermark.image)

### 什么是 Mybatis

Mybatis 是一款优秀的 ORM（持久层）框架，使用 Java 语言 编写

前身是 apache 的一个开源项目 iBatis，2010 年迁移到 google code 并正式改名为 Mybatis

ORM 持久层 指的是 : **将业务数据存储到磁盘，也具备长期存储能力，只要磁盘不损坏，如果在断电情况下，重启系统仍然可以读取数据**

### Mybatis 与 JDBC 的关系

在没有持久层框架之前, 想要代码中操作数据库都必须通过 JDBC 来操作, 接下来一个例子来说明两者之间的关系

**JDBC 操作数据库**

![JDBC](/Users/mark/typora_workspace/NoteBook/images/5737e5778db642b5a6f2788957b360b4~tplv-k3u1fbpfcp-watermark.image)

相信大家都在实际项目中使用过 Mybatis, 可以联想一下, 平常我们工作中, 是否做过以下事情：

- 是否装载过数据库驱动？
- 是否从驱动中获取数据库连接？
- 是否创建过执行 SQL 的 Statement？
- 是否自行将数据库返回结果转换成 Java 对象？
- 是否关闭过 finally 块中的三个对象？

看到上面的灵魂拷问, 就可以对本次分享的第一个问题作出解答:

**Mybatis 针对 JDBC 中重复操作做了封装, 同时扩展并优化部分功能**

## Mybatis 关键词说明

> 📖 如果在阅读文章前没有接触过 Mybatis 源码相关的内容, 建议将下述名词多看几遍再向下阅读

### SqlSession

负责执行 **select、insert、update、delete** 等命令, 同时负责获取映射器和管理事务; 其底层封装了与 JDBC 的交互, 可以说是 mybatis 最核心的接口之一

### SqlSessionFactory

负责创建 **SqlSession** 的工厂, 一旦被创建就应该在应用运行期间一直存在, **不需要额外再进行创建**

### SqlSessionFactoryBuilder

主要是负责创建 **SqlSessionFactory** 的构造器类, 其中使用到了构建者设计模式; 仅负责创建 **SqlSessionFactory**

### Configuration

Mybatis 最重要的配置类, 没有之一, 存储了大量的对象配置, 可以看源码感受一下

### MappedStatement

MappedStatement 是保存 SQL 语句的数据结构, 其中的类属性都是由解析 .xml 文件中的 SQL 标签转化而成

### Executor

SqlSession 对象对应一个 Executor, Executor 对象作用于 **增删改查方法** 以及 **事务、缓存** 等操作

### ParameterHandler

Mybatis 中的 **参数处理器**, 类关系比较简单

### StatementHandler

StatementHandler 是 Mybatis 负责 **创建 Statement 的处理器**, 根据不同的业务创建不同功能的 Statement

### ResultSetHandler

ResultSetHandler 是 Mybatis 负责将 JDBC 返回数据进行解析, 并包装为 Java 中对应数据结构的处理器

### Interceptor

Interceptor 为 Mybatis 中定义公共拦截器的接口, 其中定义了相关实现方法

## Mybatis 架构设计

### 架构图

![Mybatis 分层架构图](/Users/mark/typora_workspace/NoteBook/images/d48fb3b5d8f9476a999312f6e7e22f67~tplv-k3u1fbpfcp-watermark.image)

### 基础支持层

#### 反射模块

反射在 Java 中的应用可以说是相当广泛了, 同时也是一把双刃剑。 Mybatis 框架本身 **封装出了反射模块**, 提供了比原生反射更 **简洁易用的 API 接口**, 以及对 **类的元数据增加缓存, 提高反射的性能**

#### 类型转换

类型转换模块最重要的功能就是在为 SQL 语句绑定实参时, 将 **Java 类型转为 JDBC 类型**, 在映射结果集时再由 **JDBC 类型转为 Java 类型**

另外一个功能就是提供别名机制, 简化了配置文件的定义

#### 日志模块

日志对于系统的作用不言而喻, 尤其是测试、生产环境上查看信息及排查错误等都非常重要。主流的日志框架包括 Log4j、Log4j2、S l f4j 等, Mybatis 的日志模块作用就是 **集成这些日志框架**

#### 资源加载

Mybatis 对类加载器进行了封装, 用来确定类加载器的使用顺序, 用来记载类文件以及其它资源文件, 感兴趣可以参考 **ClassLoaderWrapper**

#### 解析器模块

解析器模块主要提供了两个功能, 一个是封装了 XPath 类, 在 Mybatis 初始化时解析 Mybatis-config.xml 配置文件以及映射配置文件提供功能, 另一点就是处理动态 SQL 语句的占位符提供帮助

#### ...

### 核心处理层

#### 配置解析

在 Mybatis 初始化时, 会加载 Mybatis-config.xml 文件中的配置信息, 解析后的配置信息会 **转换成 Java 对象添加到 Configuration 对象**

> 📖 比如说在 .xml 中定义的 resultMap 标签, 会被解析为 ResultMap 对象

#### SQL 解析

大家如果手动拼写过复杂 SQL 语句, 就会明白会有多痛苦。Mybatis 提供出了动态 SQL, 加入了许多判断循环型标签, 比如 : if、where、foreach、set 等, 帮助开发者节约了大量的 SQL 拼写时间

SQL 解析模块的作用就是将 Mybatis 提供的动态 SQL 标签解析为带占位符的 SQL 语句, 并在后期将实参对占位符进行替换

#### SQL 执行

SQL 的执行过程涉及几个比较重要的对象, **Executor、StatementHandler、ParameterHandler、ResultSetHandler**

Executor 负责维护 **一级、二级缓存以及事务提交回滚操作**, 举个查询的例子, 查询请求会由 Executor 交给 StatementHandler 完成

StatementHandler 通过 ParameterHandler 完成 **SQL 语句的实参绑定**, 通过 java.sql.Statement 执行 SQL 语句并拿到对应的 **结果集映射**

最后交由 ResultSetHandler 对结果集进行解析, 将 JDBC 类型转换为程序自定义的对象

#### 插件

插件模块是 Mybatis 提供的一层扩展, 可以针对 SQL 执行的四大对象进行 **拦截并执行自定义插件**

插件编写需要很熟悉 Mybatis 运行机制, 这样才能控制编写的插件安全、高效

### 接口层

接口层只是 Mybatis **提供给调用端的一个接口 SqlSession**, 调用端在进行调用接口中方法时, 会调用核心处理层相对应的模块来完成数据库操作

## 问题答疑

### .xml 文件定义 Sql 语句如何解析

Mybatis 在创建 SqlSessionFactory 时, XMLConfigBuilder 会解析 Mybatis-config.xml 配置文件

#### Mybatis 相关解析器

Mybatis 解析器模块中定义了相关解析器的抽象类 BaseBuilder, 不同的子类负责实现解析不同的功能, 使用了 Builder 设计模式

![BaseBuilder](/Users/mark/typora_workspace/NoteBook/images/83603426138d45b080675ad437e4e4fa~tplv-k3u1fbpfcp-watermark.image)

XMLConfigBuilder 负责解析 mybatis-config.xml 配置文件

XMLMapperBuilder 负责解析业务产生的 xxxMapper.xml

...

#### mybatis-config.xml 解析

XMLConfigBuilder 解析 mybatis-config.xml 内容参考代码 :

![parseConfiguration](/Users/mark/typora_workspace/NoteBook/images/eef7daa43e6c4306b8e73823ea3e1858~tplv-k3u1fbpfcp-watermark.image)

**XMLConfifigBuilder#parseConfiguration()** 方法将 mybatis-config.xml 中定义的标签进行相关解析并填充到 Configuration 对象中

#### xxxMapper.xml 解析

**XMLConfifigBuilder#mapperElement()** 中解析配置的 mappers 标签, 找到具体的 .xml 文件, 并将其中的 select、insert、update、delete、resultMap 等标签解析为 Java 中的对象信息

具体解析 xxxMapper.xml 的对象为 XMLMapperBuilder, 具体的解析方法为 parse()

![parse](/Users/mark/typora_workspace/NoteBook/images/a28eced384d841ed9e10f1c6247e8cd6~tplv-k3u1fbpfcp-watermark.image)

到这里就可以对当前问题作出答复了

Mybatis 创建 **SqlSessionFactory** 会解析 **mybatis-config.xml**, 然后 **解析 configuration 标签下的子标签**, 解析 mappers 标签时, 会根据相关配置读取到 .xml 文件, 继而解析 .xml 中各个标签

具体的 select、insert、update、delete 标签定义为 **MappedStatement** 对象, .xml 文件中的其余标签也会根据不同映射解析为 Java 对象

#### MappedStatement

这里重点说明下 MappedStatement 对象, 一起看一下类中的属性和 SQL 有何关联呢

![MappedStatement](/Users/mark/typora_workspace/NoteBook/images/7bb2d67aafb04015a56aca983e4fb86e~tplv-k3u1fbpfcp-watermark.image)

MappedStatement 对象中 **提供的属性与 .xml 文件中定义的 SQL 语句** 是能够对应上的, 用来 **控制每条 SQL 语句的执行行为**

### Mapper 接口的存储与实现

在平常我们写的 SSM 框架中, 定义了 Mapper 接口与 .xml 对应的 SQL 文件, 在 Service 层直接注入 xxxMapper 就可以了

也没有看到像 JDBC 操作数据库的操作, Mybatis 在中间是如何为我们省略下这些重复繁琐的操作呢

这里使用 Mybatis 源码中的测试类进行验证, 首先定义 Mapper 接口, 省事直接注解定义 SQL

![AutoConstructorMapper](/Users/mark/typora_workspace/NoteBook/images/325cb12dea0c429d885167f51e4120f0~tplv-k3u1fbpfcp-watermark.image)

这里使用 SqlSession 来获取 Mapper 操作数据库, 测试方法如下

![primitiveSubjects](/Users/mark/typora_workspace/NoteBook/images/8ad9375f42c2425dacf8e7ccf8147ff0~tplv-k3u1fbpfcp-watermark.image)

#### 创建 SqlSession

\#1 从 SqlSessionFactory 中打开一个 新的 SqlSession

#### 获取 Mapper 实例

\#2 就存在一个疑问点, 定义的 AutoConstructorMapper 明明是个接口, **为什么可以实例化为对象?**

#### 动态代理方法调用

\#3 通过创建的对象调用类中具体的方法, 这里具体聊一下 #2 操作

SqlSession 是一个接口, 有一个 **默认的实现类 DefaultSqlSession**, 类中包含了 Configuration 属性

Mapper 接口的信息以及 .xml 中 SQL 语句是在 Mybatis **初始化时添加** 到 Configuration 的 **MapperRegistry** 属性中的

![MapperRegistry#addMapper](/Users/mark/typora_workspace/NoteBook/images/c18bf30550df45768e26c57662ab979a~tplv-k3u1fbpfcp-watermark.image)

\#2 中的 getMapper 就是从 MapperRegistry 中获取 Mapper

看一下 MapperRegistry 的类属性都有什么

![MapperRegistry](/Users/mark/typora_workspace/NoteBook/images/133d9a2800e44c47951bef4a72486993~tplv-k3u1fbpfcp-watermark.image)

config 为 **保持全局唯一** 的 Configuration 对象引用

**knownMappers** 中 Key-Class 是 Mapper 对象, Value-MapperProxyFactory 是通过 Mapper 对象衍生出的 **Mapper 代理工厂**

再看一下 MapperProxyFactory 类的结构信息

![MapperProxyFactory](/Users/mark/typora_workspace/NoteBook/images/6dd6b9a20bf64c48bf549f99e8247055~tplv-k3u1fbpfcp-watermark.image)

mapperInterface 属性是 Mapper 对象的引用, methodCache 的 key 是 Mapper 中的方法, value 是 Mapper 解析对应 SQL 产生的 MapperMethod

> 📖 Mybatis 设计 methodCache 属性时使用到了 **懒加载机制**, 在初始化时不会增加对应 Method, 而是在 **第一次调用时新增**

![cachedMapperMethod](/Users/mark/typora_workspace/NoteBook/images/69b81a21a8bb42778957debf59f66563~tplv-k3u1fbpfcp-watermark.image)

MapperMethod 运行时数据如下, 比较容易理解

![MapperMethod 运行状态](/Users/mark/typora_workspace/NoteBook/images/c68f39f5734d4e11b8f1309765b8fbb3~tplv-k3u1fbpfcp-watermark.image)

通过一个实际例子帮忙理解一下 MapperRegistry 类关系, Mapper 初始化第一次调用的对象状态, 可以看到 methodCache 容量为0

![MapperRegistry 运行状态](/Users/mark/typora_workspace/NoteBook/images/6d692ce05c4346cd839532957c9c6fc0~tplv-k3u1fbpfcp-watermark.image)

我们目前已经知道 MapperRegistry 的类关系, 回头继续看一下第二步的 **MapperRegistry#getMapper**() 处理步骤

![getMapper](/Users/mark/typora_workspace/NoteBook/images/aadf5cb424d94edb9095cc6d7a0b837e~tplv-k3u1fbpfcp-watermark.image)

核心处理在 MapperProxyFactory#newInstance() 方法中, 继续跟进

![newInstance](/Users/mark/typora_workspace/NoteBook/images/0f0a61dea40f451690345e2765818cb1~tplv-k3u1fbpfcp-watermark.image)

MapperProxy **继承了 InvocationHandler 接口**, 通过 newInstance() 最终返回的是由 **Java Proxy 动态代理返回的动态代理实现类**

看到这里就清楚了步骤二中接口为什么能够被实例化, 返回的是 **接口的动态代理实现类**

### Mybatis Sql 的执行过程

根据 Mybatis SQL 执行流程图进一步了解

![Mybatis-SQL执行流程](/Users/mark/typora_workspace/NoteBook/images/edb7e271711d4a18a8840341c3b978b8~tplv-k3u1fbpfcp-watermark.image)

大致可以分为以下几步操作:

> 📖 在前面的内容中, 知道了 Mybatis Mapper 是动态代理的实现, 查看 SQL 执行过程, 就需要紧跟实现了 InvocationHandler 的 MapperProxy 类

#### 执行增删改查

```java
@Select(" SELECT * FROM SUBJECT WHERE ID = #{id}")
PrimitiveSubject getSubject(@Param("id") final int id);
复制代码
```

我们以上述方法举例, 调用方通过 SqlSession 获取 Mapper 动态代理对象, 执行 Mapper 方法时会通过 **InvocationHandler 进行代理**

![MapperProxy](/Users/mark/typora_workspace/NoteBook/images/cdedcc7def7c4dbd86a08fc5fc9370ab~tplv-k3u1fbpfcp-watermark.image)

在 MapperMethod#execute 中, 根据 MapperMethod -> SqlCommand -> **SqlCommandType** 来确定增、删、改、查方法

> 📖 SqlCommandType 是一个枚举类型, 对应五种类型 UNKNOWN、INSERT、UPDATE、DELETE、SELECT、FLUSH

![execute](/Users/mark/typora_workspace/NoteBook/images/72c0a6ce958e48adb5e28437f1a35d42~tplv-k3u1fbpfcp-watermark.image)

#### 参数处理

查询操作对应 SELECT 枚举值, if else 中判断为返回值是否集合、无返回值、单条查询等, 这里以查询单条记录作为入口

```java
Object param = method.convertArgsToSqlCommandParam(args);
result = sqlSession.selectOne(command.getName(), param);
复制代码
```

![convertArgsToSqlCommandParam_new](/Users/mark/typora_workspace/NoteBook/images/50b7fb79178f47d28a95f1a58e915e49~tplv-k3u1fbpfcp-watermark.image)

![参数解析](/Users/mark/typora_workspace/NoteBook/images/deb5cad58bdd4111abd7480d94feadcf~tplv-k3u1fbpfcp-watermark.image)

> 📖 这里能够解释一个之前困扰我的问题, 那就是为什么方法入参只有单个 `@Param("id")`, 但是参数 param 对象会存在两个键值对

继续查看 **SqlSession#selectOne** 方法, sqlSession 是一个接口, 具体还是要看实现类 **DefaultSqlSession**

![selectOne](/Users/mark/typora_workspace/NoteBook/images/17dd9d667ab9479f95b1cbf8a8ca9a41~tplv-k3u1fbpfcp-watermark.image)

因为单条和查询多条以及分页查询都是走的一个方法, 所以在查询的过程中, 会将分页的参数进行添加

![selectList](/Users/mark/typora_workspace/NoteBook/images/e361f7cde22542eb9a9f9b07e9b1c779~tplv-k3u1fbpfcp-watermark.image)

#### 执行器处理

在 Mybatis 源码中, 创建的执行器默认是 **CachingExecutor,** 使用了装饰者模式, 在类中保持了 **Executor** 接口的引用, **CachingExecutor** 在持有的执行器基础上增加了缓存的功能

![CachingExecutor#query](/Users/mark/typora_workspace/NoteBook/images/423acfcf191a4e31a4d84882a60cec23~tplv-k3u1fbpfcp-watermark.image)

**delegate.query** 就是在具体的执行器了, 默认 **SimpleExecutor,** query 方法统一在抽象父类 **BaseExecutor** 中维护

![BaseExecutor#query](/Users/mark/typora_workspace/NoteBook/images/a1c043b8664d411ab57b388c138c5169~tplv-k3u1fbpfcp-watermark.image)

**BaseExecutor#queryFromDatabase** 方法执行了缓存占位符以及执行具体方法, 并将查询返回数据添加至缓存

![queryFromDatabase](/Users/mark/typora_workspace/NoteBook/images/56cfc89fb9684668a667597305316e9e~tplv-k3u1fbpfcp-watermark.image)

**BaseExecutor#doQuery** 方法是由具体的 SimpleExecutor 实现

![doQuery](/Users/mark/typora_workspace/NoteBook/images/d04d2d59516843728bb9fd9830575500~tplv-k3u1fbpfcp-watermark.image)

#### 执行 SQL

因为我们 SQL 中使用了参数占位符, 使用的是 **PreparedStatementHandler** 对象, 执行预编译SQL的 Handler, 实际使用 **PreparedStatement** 进行 SQL 调用

![PreparedStatementHandler_query](/Users/mark/typora_workspace/NoteBook/images/17b141325bb349d2b8a227e47410f236~tplv-k3u1fbpfcp-watermark.image)

#### 返回数据解析

将 JDBC 返回类型转换为 Java 类型, 根据 resultSets 和 resultMap 进行转换

![handleResultSets](/Users/mark/typora_workspace/NoteBook/images/7d6bb34000ee4506b28e03bdd5247e12~tplv-k3u1fbpfcp-watermark.image)

### 5.4 Mybatis 中分页如何实现

通过 Mybatis 执行分页 SQL 有两种实现方式, 一种是编写 SQL 时添加 LIMIT, 一种是全局处理

#### SQL 分页

```sql
<select id="getSubjectByPage" resultMap="resultAutoMap">
    SELECT * FROM SUBJECT LIMIT #{CURRINDEX} , #{PAGESIZE}
</select>
复制代码
```

#### 拦截器分页

上文说到, Mybatis 支持了插件扩展机制, 可以拦截到具体对象的方法以及对应入参级别

我们添加插件时需要实现 **Interceptor** 接口, 然后将插件写在 mybatis-config.xml 配置文件中或者添加相关注解, Mybatis 初始化时解析才能在项目启动时添加到插件容器中

![pluginElement](/Users/mark/typora_workspace/NoteBook/images/8affa3b4dd9f4944ba892d3a5eaeb64c~tplv-k3u1fbpfcp-watermark.image)

由一个 List 结构存储项目中全部拦截器, 通过 **Configuration#addInterceptor** 方法添加

![InterceptorChain](/Users/mark/typora_workspace/NoteBook/images/f90f97319f1f47fca35b450c1a0cd86c~tplv-k3u1fbpfcp-watermark.image)

重点需要关注 **Interceptor#pluginAll** 中 plugin 方法, Interceptor 只是一个接口, plugin 方法只能由其实现类完成

![ExamplePlugin](/Users/mark/typora_workspace/NoteBook/images/226ec4a0154948ad8112e73274befc21~tplv-k3u1fbpfcp-watermark.image)

Plugin 可以理解为是一个工具类, **Plugin#wrap** 返回的是一个动态代理类 

![wrap](/Users/mark/typora_workspace/NoteBook/images/4810b915651e446aa36b0a5ea90836e1~tplv-k3u1fbpfcp-watermark.image)

这里使用一个测试的 Demo 看一下方法运行时的参数

![AlwaysMapPlugin](/Users/mark/typora_workspace/NoteBook/images/df9732f94a654d378ac0b73ff1168ba2~tplv-k3u1fbpfcp-watermark.image)

虽然是随便写的 Demo, 但是与正式使用的插件并无实际区别

![插件运行状态](/Users/mark/typora_workspace/NoteBook/images/10c063fc192647f48be04462aad2e297~tplv-k3u1fbpfcp-watermark.image)

## 结言

相对于 Spring 而言，Mybatis 足够的轻巧，属于入门级的框架源码，但是里面用到的设计模式却不少，可以借鉴其中的设计进行套用业务代码。同时，掌握了 Mybatis 之后对阅读 SpringCloud、Dubbo 源码提供了不小的帮助，这里也希望看过文章的小伙伴对 Mybatis 的理解能加深印象