

# Mybatis源码分析---1

## 1 源码的技术本质：

![image-20210123201510496](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123201510496.png)

![image-20210123224122157](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123224122157.png)



## 2 Mybatis如何获取数据源：

### 2.1 数据源获取步骤：

​	![image-20210123201158618](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123201158618.png)

![image-20210123201344287](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123201344287.png)

![image-20210123200346781](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123200346781.png)



```java
public Configuration parse() {
        // 若已解析，抛出 BuilderException 异常
    if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    // 标记已解析
    parsed = true;
    // 解析 XML configuration 节点
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
}
```



```xml
<!-- root的值 -->
<configuration>
    <!-- 存储具体参数的值，不可以放到后面，因为源码解析是放在前面 -->
    <properties resource="jdbc.properties"/>
    <environments default="development">
        <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
        </dataSource>
        </environment>
    </environments>
    <mappers>
    	<mapper resource="mapper/BlogMapper.xml"/>
    </mappers>
</configuration>
```

=======================================================================================

```java
 private void environmentsElement(XNode context) throws Exception {
     if (context != null) {
         // environment 属性非空，从 default 属性获得
         if (environment == null) {
             environment = context.getStringAttribute("default");
         }
         // 遍历 XNode 节点
         for (XNode child : context.getChildren()) {
             // 判断 environment 是否匹配
             String id = child.getStringAttribute("id");
             if (isSpecifiedEnvironment(id)) {
                 // 解析 `<transactionManager />` 标签，返回 TransactionFactory 对象
                 TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                 //***重点 解析 `<dataSource />` 标签，返回 DataSourceFactory 对象
                 DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                 DataSource dataSource = dsFactory.getDataSource();
                 // 创建 Environment.Builder 对象
                 Environment.Builder environmentBuilder = new Environment.Builder(id)
                     .transactionFactory(txFactory)
                     .dataSource(dataSource);
                 // 构造 Environment 对象，并设置到 configuration 中
                 configuration.setEnvironment(environmentBuilder.build());
             }
         }
     }
 }
```

+ 具体内容从jdbc.properties里面读取

```xml
<!--environments的值 -->
<environments default="development">
    <environment id="development">
    <transactionManager type="JDBC"/>
    <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3366/mybatis?serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </dataSource>
    </environment>
</environments>
```

## 3. Mybatis如何获取SQL语句：

+ 因为sql语句在/mapper/* xml文件中，谁能够解析/mapper/*xml文件，谁就获取了sql语句。

这个文件被mybatis-config.xml文件中的 <mappers>标签包含

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"></properties>
    <!-- ...... -->
    <mappers>
        <mapper resource="mapper/BlogMapper.xml"/>
    </mappers>
</configuration>
```

+ mapper文件的几种形式：
  - package
  - resource
  - url
  - mapperclass

![image-20210123204214981](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123204214981.png)

=======================================================================================

```java
 private void configurationElement(XNode context) {
     try {
         // 获得 namespace 属性
         String namespace = context.getStringAttribute("namespace");
         if (namespace == null || namespace.equals("")) {
             throw new BuilderException("Mapper's namespace cannot be empty");
         }
         // 设置 namespace 属性
         builderAssistant.setCurrentNamespace(namespace);
         // 解析 <cache-ref /> 节点
         cacheRefElement(context.evalNode("cache-ref"));
         // 解析 <cache /> 节点
         cacheElement(context.evalNode("cache"));
         // 已废弃！老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除，这里不会记录。
         parameterMapElement(context.evalNodes("/mapper/parameterMap"));
         // 解析 <resultMap /> 节点们
         resultMapElements(context.evalNodes("/mapper/resultMap"));
         // 解析 <sql /> 节点们
         sqlElement(context.evalNodes("/mapper/sql"));
         // 解析 <select /> <insert /> <update /> <delete /> 节点们
         buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
     } catch (Exception e) {
         throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
     }
 }
```

+ context节点的内容：

  ```xml
  <mapper namespace="org.mybatis.example.BlogMapper">
      <select id="selectBlog" resultType="org.apache.ibatis.demo.pojo.Blog">
     		select * from Blog where id = #{id}
      </select>
  </mapper>
  ```

  ### 3.1 获取SQL语句的步骤：

  ![image-20210123212435250](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123212435250.png)

  ![image-20210123212634514](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123212634514.png)

  ## 4. Mybatis操作数据库：

  ![image-20210123223847041](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123223847041.png)

  ### 4.1 openSession()

  ![image-20210123213727214](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123213727214.png)

  ### 4.2 openSessionFromDataSource()

  ![image-20210123214213749](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123214213749.png)

  ### 4.3 newExecutor()

  #### 三种枚举类型：

  + SIMPLE
  + BATCH
  + REUSE

  ![image-20210123214603327](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210123214603327.png)

  

  

  