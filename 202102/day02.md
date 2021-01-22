# 1. 浅谈源码

## 1.1源码的目的：

（1）解决新问题和新需求

（2）真正理解理论的落地实现

（3）应对面试

## 1.2阅读源码的心态

（1）持久战

（2）锦上添花，如果是新领域，首要任务是读文档而不是读源码

（3）大量实操才是最根本的保障

（4）读源码需要理论先行

## 1.3读源码的步骤

（1）从官方仓库Fork到自己的仓库

（2）clone某个项目的代码到本地

（3）查看这个项目的release列表

（4）找到一个看的懂的Release版本

（5）读懂上一个版本的源码

（6）向后阅读大版本的代码

（7）读最新的代码

## 1.4月度源码的技巧

（1）调试追踪

（2）归类总结

（3）上下文整合

# 2. MyBatis源码：

## 2.1 Mybatis的核心概念

（1）SqlSession：代表和数据库的一次会话，向用户提供了操作数据库的方法

（2）MappedStatement：代表要发往数据库执行的指令，可以理解为是Sql的抽象表示

（3）Execute：具体用来和数据库交互的执行器，接受MappedStatement作为参数

（4）映射接口：在接口中会要执行的Sql用一个方法来表示，具体的Sql写在映射文件中

（5）映射文件：可以理解为MyBatis编写Sql的地方，通常来说每一张单表都会对应着一个映射文件，在该文件中会定义Sql语句入参和出参的形式

## 2.2 MyBatis的核心功能

（1）将复杂数据库操作语句解析为纯粹的SQL语句

（2）将数据库操作节点和映射接口的抽象方法进行绑定，在抽象方法被调用时执行数据库操作

（3）将输入参数对象转化为数据库操作语句中的参数

（4）将数据库操作语句的返回结果转化为对象

## 2.3 MyBatis操作的两大阶段

（1）MyBatis初始化阶段，只在启动时运行一次

（2）数据读写阶段

## 2.4源码结构

![image-20210121193857936](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210121193857936.png)

# 3 Mybatis基础包

## 3.1 exception包

![image-20210121194943511](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210121194943511.png)

+ Error及其子类：代表了JVM自身的异常。这一类异常发生时，无法通过程序来修正。最可靠的方式就是停止JVM的运行
+ Exception及其子类：代表程序运行中发生了意料之外的事情。这些意外的事情可以被java异常处理机制处理
  - RuntimeException及其子类：这一类异常其实是程序设计的错误，用过修正程序是可以避免的，如数组越界，数值异常
  - 非RuntimeException及其子类：这一类异常的发生通常由外部因素导致，是不可以预知和避免的。如IO异常，类型寻找异常

***

### 3.1.1异常处理的原则：

* 捕获异常是为了处理它，最外层的业务使用者必须处理异常

* 不能在finally块中使用return

* 防止NPE异常，注意NPE产生的场景

  - 返回类型为基本数据，return包装数据类型的对象时，自动拆箱有可能产生NPE
  - 数据库查询结果可能为null
  - 集合里的元素即使isNotEmpty，取出的数据元素也可能为null
  - 远程调用返回对象时，建议NPE检查
  - 级联调用obj.getA().getB().getC()；一连串调用，易产生NPE

* 抛异常还是错误码：

  - 对于公司外的http/api开放接口必须使用错误码
  - 而应用内部推荐异常抛出
  - 跨应用间的rpc调用有限考虑使用Result方式，封装isSuccess()方法、错误码、错误剑短信息。

* DAO/Service/Controller层异常处理原则：

  - DAO/Service层触发某些checked异常时，应该做异常处理，更多时候应该转换为uncheck exception抛出
  - Service层遇到无法处理的问题，应该抛出自定义业务异常
  - Controller层接收到异常后，应该将异常转换为对用户友好的提示

  ![image-20210121200555599](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210121200555599.png)

* 不要捕获顶层的Exception

  - 异常的粒度很重要，应该为一个基本操作定义一个try-catch快，不要将几百行代码放到一个try-catch

* NPE，数组越界异常不应该通过catch的方式来处理

* 异常不要用来做流程控制、条件控制

***

### 3.1.2序列化

+ 在希望类的版本间实现序列化二号反序列化的兼容时，保持serialVersionUID值不变
+ 在希望类的版本间实现序列化二号反序列化的不兼容时，确保serialVersionUID值改变

***

### 3.1.3 Mybatis的Exception包

* IbatisException
* PersistenceException
* TooManyResultsException

ExceptionFactory类：该类是负责生产Exception的工厂

## 3.2 reflection包

提供反射功能的基础包

#### 【设计模式】装饰器模式：

​		指能够在一个类的基础上增加一个装饰类，并在装饰类中增加一些新的特性和功能，这样，通过对原有类的包装，就可以在不改变原有类的情况下为原有类增加更多的功能

#### 【Java】反射：

​		![image-20210121203220147](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210121203220147.png)

![image-20210121203412990](C:\Users\baiheng.jiang\AppData\Roaming\Typora\typora-user-images\image-20210121203412990.png)



## 3.3 annotation包和lang包

## 3.4 type包

## 3.5 io包

## 3.6 logging包

## 3.7 parsing包

# 4. Java代码规范

## 4.1 共通开发规约；

### 4.1.1目录结构划分：

一个服务或项目大体分为四层Package：API，Service、DAO、Common

* Api：对外API接口，该包中的功能暴露到网关或其他模块调用
* Servcie：业务实现层，功能不对外提供
* Dao：数据访问接口及实现
* Common：公共模块，包括常量、工具类、通用模型
* Api-impl：对外API的实现，TO C（To Customer）
* model：实体对象
* web：TO B（To Business）

### 4.1.2 包引入规范：

```
import java.util.List
不推荐使用：
import java.util.*
```

### 4.1.3 Java源文件结构：

（1）版本和版权声明：

（2）包声明：

（3）包的引用：

（4）类注释：

（5）类实现：

		 1. 静态变量定义
   		 2. 实例变量定义
   		 3. 构造方法

### 4.1.4 实体类继承规范：

所有Entity对象必须继承BaseEntity对象，如下：

```
public class BaseEntity implements Serializable{
	
	private String createBy;
	private Date createDate;
	private String lastUpdateBy;
	private Date lastUpdateDate;
	private Integer rowVersion;
	private Integer isValid = 1;
}
```

### 4.1.5 代码排版：

+ 在每个类声明之后，每个方法定义结束之后加空行
+ 不同代码块按层次有缩进
+ 对于if,for，即使只有一行执行内容，也要写在大括号内
+ 表达式定义：在运算符两侧各加一个空格
+ 类、方法、块结构：
  - “{”和声明语句放在同一行
  - “{”之前加一个空格
  - “}”单独占一行
+ 类先继承，后实现

### 4.1.6 方法命名：

（1）应该描述出该方法大致的用途

（2）方法名一般以动词开头，采用完整的英文描述时候成员函数功能

（3）service中不用开启事务的方法在配置文件中添加read-only = "true"

### 4.1.7 变量命名：

（1）禁止使用 int aa;

（2）for循环用 i,j,k, 可用x,y,z

（3）避免使用下划线

（4）大于3个字符，小于15个字符

（5）常量全部大写，单词之间用下划线

### 4.1.8 错误码规范：

（1）错误码 5位或6位，总区间为：10000-999999，根据不同业务划分500-1000个错误码

### 4.1.9 MVC层、Servcice层：

（1）Controller类命名除符合普通类命名规则外，还需要以controller作为结尾

（2）Service类命名除符合普通类命名规则外，还需要以service作为结尾

（3）service的实现不该牵涉到DTO

## 4.2 接入A类型网关接口约定：

### 4.2.1 异常处理：

+ BaseDubboException：作为业务异常
+ BaseDubboServerException：为服务器异常

## 4.3 接入B类型网关接口约定：

## 4.4 其他规则：

# 5 数据库规范：

## 5.1 表名规范：

（1）普通表：TB_XXXX

（2）日志表：TL_XXXX

（3）关系表：TR_XXXX

（4）接口表：TI_XXXX

（5）系统表：TS_XXXX







