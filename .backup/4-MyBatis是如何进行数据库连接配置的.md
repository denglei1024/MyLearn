[MyBatis是如何进行数据库连接配置的](https://github.com/denglei1024/denglei1024.github.io/issues/4)


MyBatis 是一个流行的持久层框架，它通过使用XML或注解的方式将SQL语句、存储过程和Java方法进行绑定，从而避免了手写大量的JDBC代码和手动设置参数与结果集。以下是 MyBatis 连接数据库的基本步骤和机制：

> MyBatis源码版本：3.5.16

**MyBatis-config.xml 示例**

```xml
<environments default="development">  
    <environment id="development">  
        <transactionManager type="JDBC"/>  
        <dataSource type="POOLED">  
            <property name="driver" value="${db.driver}"/>  
            <property name="url" value="${db.url}"/>  
            <property name="username" value="${db.username}"/>  
            <property name="password" value="${db.password}"/>  
        </dataSource>  
    </environment>  
</environments>
```


### 1. 读取配置文件

MyBatis 通过 `Resources` 类读取配置文件。配置文件通常是 XML 格式，如 `mybatis-config.xml`。

#### 关键代码

```java
String resource = "mybatis-config.xml";
Reader reader = Resources.getResourceAsReader(resource)
```

### 2. 解析配置文件

`SqlSessionFactoryBuilder` 使用 `XMLConfigBuilder` 解析配置文件，将 XML 配置转换为 Java 对象。

#### 关键代码

```java
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
```

#### `XMLConfigBuilder` 源码片段

```
private void parseConfiguration(XNode root) {  
  try {  
    // issue #117 先读取属性 
    propertiesElement(root.evalNode("properties"));  
    Properties settings = settingsAsProperties(root.evalNode("settings"));  
    loadCustomVfsImpl(settings);  
    loadCustomLogImpl(settings);  
    typeAliasesElement(root.evalNode("typeAliases"));  
    pluginsElement(root.evalNode("plugins"));  
    objectFactoryElement(root.evalNode("objectFactory"));  
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));  
    reflectorFactoryElement(root.evalNode("reflectorFactory"));  
    settingsElement(settings);  
    // read it after objectFactory and objectWrapperFactory issue #631  
    environmentsElement(root.evalNode("environments"));  
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));  
    typeHandlersElement(root.evalNode("typeHandlers"));  
    mappersElement(root.evalNode("mappers"));  
  } catch (Exception e) {  
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);  
  }  
}
```

`XMLConfigBuilder`解析XML配置，并生成`Configuration`对象。

### 3. 创建 SqlSessionFactory

`Configuration` 对象包含了所有配置信息，包括环境配置、数据源配置等。`SqlSessionFactoryBuilder` 使用 `Configuration` 创建 `DefaultSqlSessionFactory`。

#### 关键代码

```java
public SqlSessionFactory build(Configuration config) {  
  return new DefaultSqlSessionFactory(config);  
}
```

### 4. 建立数据库连接

`DefaultSqlSessionFactory` 创建 `SqlSession` 时，通过 `Environment` 配置和 `DataSource` 获取数据库连接。

#### `DefaultSqlSessionFactory` 源码片段

```java
public SqlSession openSession() {  
  return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);  
}
```

`openSessionFromDataSource` 方法中，`DataSource` 被用于获取数据库连接。

#### 关键代码

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level,  
    boolean autoCommit) {  
  //环境配置  
  final Environment environment = configuration.getEnvironment();  
  //事务工厂  
  final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);  
  //创建事务  
  Transaction tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);  
  //创建执行器  
  final Executor executor = configuration.newExecutor(tx, execType);  
  //返回SqlSession  
  return new DefaultSqlSession(configuration, executor, autoCommit);
}
```

### 5. 数据源配置

`DataSource` 配置在 `mybatis-config.xml` 文件中，通过 `DataSourceFactory` 创建具体的数据源实例。

#### 关键代码

```java
// 获取数据源工厂
String type = context.getStringAttribute("type");  
Properties props = context.getChildrenAsProperties();  
DataSourceFactory factory = (DataSourceFactory) resolveClass(type).getDeclaredConstructor().newInstance();  
factory.setProperties(props);    
// 获取数据源
DataSource dataSource = factory.getDataSource();
```

### 总结

MyBatis连接数据库的步骤包括：

1. **读取配置文件**：通过 `Resources` 类读取 XML 文件。
2. **解析配置文件**：使用 `XMLConfigBuilder` 解析 XML，生成 `Configuration` 对象。
3. **创建 SqlSessionFactory**：使用 `SqlSessionFactoryBuilder` 和 `Configuration` 创建 `DefaultSqlSessionFactory`。
4. **建立数据库连接**：`DefaultSqlSessionFactory` 创建 `SqlSession`，使用 `DataSource` 获取数据库连接。
