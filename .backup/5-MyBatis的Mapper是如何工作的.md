[MyBatis的Mapper是如何工作的](https://github.com/denglei1024/denglei1024.github.io/issues/5)

## 介绍

本文将介绍MyBatis的Mapper接口如何跟映射文件关联起来。

### 1、配置

在MyBatis的配置文件中，可以指定Mapper接口。示例中，`mapper`标签指定了Mapper接口的类或者具体的文件位置：

```xml
<configuration>
    <mappers>
    	<mapper class="com.github.denglei1024.UserMapper"/>
    	或者
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```

### 2、加载

加载资源有多种方式，入口方法在`XMLConfigBuilder`的`mappersElement`中，关键代码如下：

```java
private void mappersElement(XNode context) {
  for (XNode child : context.getChildren()) {
    if ("package".equals(child.getName())) {
      String mapperPackage = child.getStringAttribute("name");
      configuration.addMappers(mapperPackage);
    } else {
      String resource = child.getStringAttribute("resource");
      String url = child.getStringAttribute("url");
      String mapperClass = child.getStringAttribute("class");
      if (resource != null && url == null && mapperClass == null) {
        ErrorContext.instance().resource(resource);
        try (InputStream inputStream = Resources.getResourceAsStream(resource)) {
          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource,
              configuration.getSqlFragments());
          mapperParser.parse();
        }
      } else if (resource == null && url != null && mapperClass == null) {
        ErrorContext.instance().resource(url);
        try (InputStream inputStream = Resources.getUrlAsStream(url)) {
          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url,
              configuration.getSqlFragments());
          mapperParser.parse();
        }
      } else if (resource == null && url == null && mapperClass != null) {
        Class mapperInterface = Resources.classForName(mapperClass);
        configuration.addMapper(mapperInterface);
      } else {
        throw new BuilderException(
            "A mapper element may only specify a url, resource or class, but not more than one.");
      }
    }
  }
```

### 3、注册

解析完文件后，`MapperRegistry`类负责Mapper接口的注册和查找。

```java
  public class Configuration{
      protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
      public  void addMapper(Class type) {
        mapperRegistry.addMapper(type);
    }
  }
```

### 4、解析

MyBatis使用动态代理机制为Mapper接口生成实现类。调用`SqlSession.getMapper(Class type)`方法时，会创建Mapper接口的动态代理对象。

```java
 public class DefaultSqlSession{
    @Override
    public  T getMapper(Class type) {
      return configuration.getMapper(type, this);
    }
  }

  public class Configuration{
    public  T getMapper(Class type, SqlSession sqlSession) {
      return mapperRegistry.getMapper(type, sqlSession);
    }
  }
```

`MapperRegistry`会使用`MapperProxyFactory`为Mapper接口创建代理实例。

```java
public class MapperRegistry {
    public  T getMapper(Class type, SqlSession sqlSession) {
      final MapperProxyFactory mapperProxyFactory = (MapperProxyFactory) knownMappers.get(type);
      if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
      }
      try {
        return mapperProxyFactory.newInstance(sqlSession);
      } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
      }
    }
}  	  
```

### 5、执行

当调用Mapper接口的方法时，动态代理会拦截方法调用，并将其转换为SQL语句的执行。具体执行逻辑由`MapperMethod`类处理。

```java
public class MapperProxy{
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      try {
        if (Object.class.equals(method.getDeclaringClass())) {
          return method.invoke(this, args);
        }
        return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
  }
```

`MapperMethod`会根据方法和参数生成SQL语句，并通过`Executor`执行SQL语句，返回结果。

```java
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
  switch (command.getType()) {
    case INSERT: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
      break;
    }
    case UPDATE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
      break;
    }
    case DELETE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
      break;
    }
    case SELECT:
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) {
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) {
        result = executeForMap(sqlSession, args);
      } else if (method.returnsCursor()) {
        result = executeForCursor(sqlSession, args);
      } else {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
        if (method.returnsOptional() && (result == null || !method.getReturnType().equals(result.getClass()))) {
          result = Optional.ofNullable(result);
        }
      }
      break;
    case FLUSH:
      result = sqlSession.flushStatements();
      break;
    default:
      throw new BindingException("Unknown execution method for: " + command.getName());
  }
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName()
        + "' attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
 }
```

## 总结

本文介绍了MyBatis中Mapper从定义到使用的过程，从源码角度演示了Mapper的定义、加载、注册、查找和执行操作的一系列过程，希望对你有帮助。
