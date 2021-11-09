# Mybatis

环境 jdk, mysql, maven. idea

回顾 JDBC Mysql java基础 Maven junit



SSM框架：配置文件的。 最好的方式：看官方文档



## 1简介

### 1.1什么是Mybatis

#####  ![](C:\Users\Jack\AppData\Roaming\Typora\typora-user-images\image-20211101194803948.png)

- MyBatis 是一款优秀的持久层框架

- 它支持自定义 SQL、存储过程以及高级映射。

- MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。

- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。
- MyBatis 本是apache的一个[开源项目](https://baike.baidu.com/item/开源项目/3406069)iBatis, 2010年这个[项目](https://baike.baidu.com/item/项目/477803)由apache software foundation 迁移到了[google code](https://baike.baidu.com/item/google code/2346604)，并且改名为MyBatis 。
- 2013年11月迁移到[Github](https://baike.baidu.com/item/Github/10145341)。

如何得到Mybatis

1.maven仓库下载依赖

```java
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
<groupId>org.mybatis</groupId>
<artifactId>mybatis</artifactId>
<version>3.4.6</version>
</dependency>

```

2.Github [Releases · mybatis/mybatis-3 (github.com)](https://github.com/mybatis/mybatis-3/releases)

3.中文文档 https://mybatis.org/mybatis-3/zh/index.html



### 1.2持久化

​		数据持久化

- 持久化就是将程序的数据在持久状态和瞬时状态转化的过程

- 内存：断电就失

- 数据库（JDBC），io文件持久化

为什么需要持久化？

1.有一些对象，不能让他丢掉

2.内存太贵重了



### 1.3持久层

Dao层，Service层，Controller层

1.完成持久化的工作代码

2.层界限十分明显



### 1.4为何需要Mybatis

1.帮助我们将数据存入到数据库之中

2.方便

3.传统的JDBC太过于复杂了。简单，框架，自动化

4.不用Mybatis也可以，更容易上手，技术没有高低之分

5.有点：

​			简单，灵活，sql同代码分离提高可维护性，提供映射标签，支持对象与数据库的orm字段映射，提供对象关系的映射标签，支持对象关系组件维护，提供xml标签，支持动态编写sql



**tips：使用的人多**



## 2第一个Mybatis程序

思路：搭建环境->导入Mybatis->编写代码->测试

### 2.1搭建环境

搭建数据库

```java
CREATE DATABASE `mybatis`;

USE `mybatis`

CREATE TABLE `user`(
id INT(20) NOT NULL PRIMARY KEY,
`name` VARCHAR(32) DEFAULT NULL,
`pwd` VARCHAR(32) DEFAULT NULL
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user`(id, `name`, pwd)VALUES
(1, 'scy', 12345),
(2, '张三', 123),
(3, 'lisi', 123456)
```

新建项目

1.新建一个普通的maven项目

2.删除src目录

3.导入maven依赖

```XML
 <dependencies>
        <!--mysqlq驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.12</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.4</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

```

### 2.2创建一个模块

编写mybatis核心配置文件

```xml
<?xml version="1.0" encoding="utf8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?userSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="scy464658"/>
            </dataSource>
        </environment>
    </environments>
   // <mappers>
   //     <mapper resource="com/scy/dao/UserMapper.xml"/>
   //</mappers>
</configuration>

```

编写mybatis工具类

```java
static SqlSessionFactory sqlSessionFactory;

static {
try {
//使用Mybatis第一步 ：获取sqlSessionFactory对象
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
} catch (IOException e) {
e.printStackTrace();
}
}

//既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例.
// SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。
public static SqlSession getSqlSession(){
return sqlSessionFactory.openSession();
}

```

### 2.3编写代码

- 实体类

```JAVA
private int id;
private  String name;
private String pwd;

public User(){

}
public User(int id, String name, String pwd) {
this.id = id;
this.name = name;
this.pwd = pwd;
}

public int getId() {
return id;
}

public void setId(int id) {
this.id = id;
}

public String getName() {
return name;
}

public void setName(String name) {
this.name = name;
}

public String getPwd() {
return pwd;
}

public void setPwd(String pwd) {
this.pwd = pwd;
}
```



- Dao接口

```JAVA
public interface UserDao {
public List<User> getUserList();
}
```



- 接口实现类由原来的UserImpI转变成一个Mapper文件

```JAVA
<?xml version="1.0" encoding="utf8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace=绑定一个指定的Dao/Mapper接口-->
<mapper namespace="com.scy.dao.UserDao">
<select id="getUserList" resultType="com.scy.pojo.User">
select * from mybatis.user;
</select>
</mapper>

```



### 2.4测试代码

**MapperRegistry是什么？**

核心配置文件中注册mappers

--juit测试

```JAVA
public void test(){
//第一步：获得SqlSession对象
SqlSession sqlSession = MybatisUtils.getSqlSession();

//第二步：执行sql语句,通过UserDao.class获得其对应的实现类的对象mapper
UserDao mapper = sqlSession.getMapper(UserDao.class);

//第三步：通过mapper执行其getUserList()方法
List<User> userList = mapper.getUserList();

for(User user:userList){
System.out.println(user);
}

sqlSession.close();//用完后记者关闭session


}
```

注意点：

1.mapper映射（mybati-config.xml 中间的mapper映射）

2.约定大于配置

```JAVA
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

## 3.CRUD

###  	**1.namespace**

​		namespace中的包名要和Dao/mapper接口的包名一致

###      **2.select**

​	选择，查询语句

- id:就是对应的namespace中的方法名

- resultType：SQL语句执行的返回值

- parameterType:参数类型



Dao

```java
User getUserById(int id);
```

mapper.xml

```java
<select id="getUserById" parameterType="int" resultType="com.scy.pojo.User">
		select * from mybatis.user where id = #{id};
</select>
```

Test

```java
@Test
public void getUserById(){
	SqlSession sqlSession = MybatisUtils.getSqlSession();

	UserMapper mapper = sqlSession.getMapper(UserMapper.class);

	User user = mapper.getUserById(1);
	System.out.println(user);
	sqlSession.close();
}
```

###   3.insert

  Dao

  ```java
 int addUser(User user);
  ```

  mapper.xml

  ```java
<insert id="addUser" parameterType="com.scy.pojo.User">
        insert into mybatis.user(id, name, pwd)values (#{id}, #{name}, #{pwd});
    </insert>
  ```

  Test

  ```java
 @Test
    public void addUser() {

      SqlSession  sqlSession = MybatisUtils.getSqlSession();
       UserMapper mapper = sqlSession.getMapper(UserMapper.class);
       int res = mapper.addUser(new User(4, "hi", "1234567"));
       if(res > 0){
           System.out.println("成功");
       }
       sqlSession.commit();
       sqlSession.close();
    }
  ```

###   4.update

Dao

```JAVA
    int Update(User user);
```

mapper.xml

```java
<update id="updateUser" parameterType="com.scy.pojo.User">
        update mybatis.user set name = #{name},pwd=#{pwd} where id ={id};
    </update>
```

### 5.delete

Dao

```java
 int deleteUser(int id);
```

mapper.xml

```java
    <delete id="deleteUser" parameterType="int">
        delete from mybatis.user where id = #{id};
    </delete>
```

Test

```java
    @Test
    public void deleteUser(){
        SqlSession  sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(4);
        sqlSession.commit();
        sqlSession.close();
    }
```

## 4.配置解析

### 	4.1核心配置文件

- mybatis-config.xml

- Mybatis配置文件包含了会深深影响Mybatis行为的设置和属性信息

  ```XML
  configuration（配置）
  properties（属性）
  settings（设置）
  typeAliases（类型别名）
  typeHandlers（类型处理器）
  objectFactory（对象工厂）
  plugins（插件）
  environments（环境配置）
  environment（环境变量）
  transactionManager（事务管理器）
  dataSource（数据源）
  databaseIdProvider（数据库厂商标识）
  mappers（映射器）
  ```

  

### 4.2环境配置(environments)

Mybatis可以配置适应多种文件

不过要记住，尽管可以配置多个环境，但是每个SqlSessionFactory实例只能选择一个环境

会使用和配置环境

Mybatis默认的事务管理器就是JDBC还有一个无用的MANAGED

连接池POOLED



### 4.3属性（properties）

我们可以通过properties属性来实现引用配置文件

这些属性都是可以外部配置且可以动态替换的，既可以在典型的java属性文件中配置，亦可以通过properties元素的子元素来传递{db.properties}

```properties
driver = com.mysql.jdbc.Driver
url = jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8
username = root
password = 12345
```

在核心配置文件中引入

```JAVA
 <!-- 引入外部配置文件-->
    <properties resource="db.properties">
        <property name="username" value="root"/>
        <property name="pwd" value="11111"/>
    </properties>
```

- 可以直接引入外部文件
- 可以在其中增加一些属性配置
- 如果两个文件由同一个字段，优先使用外部配置文件的



### 4.4类型别名（typeAliases）

- 类型别名可为 Java 类型设置一个缩写名字。

- 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

  ```JAVA
   <!--可以给实体类起别名-->
      <typeAliases>
          <typeAlias type="com.scy.pojo" alias="User"></typeAlias>
      </typeAliases>
  ```

  也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：扫描实体类的包，它的默认别名就是这个类的类名，首字母小写

  ```JAVA
  <typeAliases>
    <package name="com.scy.pojo"/>
  </typeAliases>
  ```

  在实体类比较少的时候，使用第一种方式

  如果实体类十分多，建议使用第二种

  第一种可以DIY别名，第二种则不行，如果非要改，需要在实体类上添加注解

  @Alias("user")

  public class User{}

### 4.5设置

![](F:\邵辰宇\ideal\123456.png)

### 4.6其他设置

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)

- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)

- [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)

  ​		1.mybatis-generator-core

  ​		2.mybatis-plus

  ​		3.通用mapper





### 4.7映射器(mapper)

MapperRegistty:注册绑定我们的Mapper文件

​	方式一：[推荐使用]

```JAVA
<!-- 每一个Mapper.XML都需要在Mybatis核心文件中注册--> 
<mappers>
        <mapper resource="com/scy/dao/UserMapper.xml"/>
    </mappers>
```

方式二：使用class文件绑定注册

```XML
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="com.scy.dao.User"/>
</mappers>
```

注意点

- 接口和他的Mapper配置文件必须同名

- 接口和他的Mapper配置文件必须在同一个包下

  ```XML
  <mappers>
    <package name="com.scy.dao"/>
  </mappers>
  ```

注意点

- 接口和他的Mapper配置文件必须同名
- 接口和他的Mapper配置文件必须在同一个包下



### 4.8生命周期和作用域

生命周期和作用域是至关重要的，因为错误使用会导致严重的**并发问题**

![](F:\邵辰宇\1234567.png)

#### SqlSessionFactoryBuilder

- 一旦创建了 SqlSessionFactoryBuilder，就不再需要它了

- 局部变量



#### SqlSessionFactory

- 可以想象为，数据库连接池

- sqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**。

- 因此 SqlSessionFactory 的最佳作用域是应用作用域。

- 有很多方法可以做到，最简单的就是使用**单例模式**或者静态单例模式。



#### SqlSession

- 连接到连接池的一个请求

- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。

- 用完之后迅速关闭，否则资源被占用

  ![](F:\邵辰宇\12345678.png)

**每一个Mapper，对应一个业务**

## 5.解决属性名和字段名不一致的问题

### 5.1问题

数据库中的字段

​						![](F:\邵辰宇\ideal\QQ截图20211103164939.png)



新建一个项目，拷贝之前的，测试实体类字段不一致的情况

```JAVA
public class User {
    private int id;
    private  String name;
    private String password;}
```

![](F:\邵辰宇\ideal\结果.png)

```XML
//	select * from mybatis.user where id = #{id}
// 类型处理器
//	select id, name, pwd from mabtis.user where id = #{id}
```

解决方法

- 起别名

  ```JAVA
  <select id ="getUserById" resultType = "com.scy.pojo.User">
      select id, name, pwd as password from mybatis.user where id =#{id};
  </select>
     
  ```

### 5.2resultMap

```XML
id name pwd
id name password
```



```java
<resultMap id="UserMap" type="User">
        <!--column数据库中的字段，property实体类中的属性-->
        <result column="pwd" property="password"></result>
        <result column="id" property="id"></result>
    </resultMap>
   		<select id="getUserById"  resultMap="UserMap">
        select * from mybatis.user where id = #{id};
    </select>
```

- resultMap元素是Mybatis中最重要最强大的元素
- ResultMap的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句，只需要描述他们的关系就行了
- ResultMap最优秀的地方在于，虽然你已经对于它相当了解了，但是根本不用显示的调用他们
- **如果世界总是那么简单就好了**

## 6.日志

### 6.1日志工厂

如果一个数据库操作，出现了异常，我们需要排错，日志就是最好的助手！

曾经：sout，debug

现在：日志工厂

![](F:\邵辰宇\ideal\日志工厂.png)

- SLF4J  
- LOG4J √
- LOG4J2 
- JDK_LOGGING 
- COMMONS_LOGGING 
- STDOUT_LOGGING √
- NO_LOGGING

在Mybatis中具体使用日志实现，在设置中设定

```XML
 <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

![](F:\邵辰宇\ideal\日志工厂1.png)

### 6.2 Log4j

- Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995)的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件

- 我们也可以控制每一条日志的输出格式

- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。

- 这些可以通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置，而不需要修改应用的代码。

  1.先导入log4j的包

  ```XML
  <dependency>
              <groupId>log4j</groupId>
              <artifactId>log4j</artifactId>
              <version>1.2.12</version>
          </dependency>
  ```

  2.log4j.properties

  ```xml
  #将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
  log4j.rootLogger=DEBUG,console,file
  
  #控制台输出的相关设置
  log4j.appender.console = org.apache.log4j.ConsoleAppender
  log4j.appender.console.Target = System.out
  log4j.appender.console.Threshold=DEBUG
  log4j.appender.console.layout = org.apache.log4j.PatternLayout
  log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
  
  #文件输出的相关设置
  log4j.appender.file = org.apache.log4j.RollingFileAppender
  log4j.appender.file.File=./log/lin.log
  log4j.appender.file.MaxFileSize=10mb
  log4j.appender.file.Threshold=DEBUG
  log4j.appender.file.layout=org.apache.log4j.PatternLayout
  log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
  
  #日志输出级别
  log4j.logger.org.mybatis=DEBUG
  log4j.logger.java.sql=DEBUG
  log4j.logger.java.sql.Statement=DEBUG
  log4j.logger.java.sql.ResultSet=DEBUG
  log4j.logger.java.sql.PreparedStatement=DEBUG
  ```

  3.配置log4j为日志的实现

  ```XML
   <settings>
          <setting name="logImpl" value="LOG4J"/>
      </settings>
  ```

  4.输出结果

  ![](F:\邵辰宇\ideal\日志工厂2.png)

## 7.分页

​	**思考一个问题：为什么需要分页？**

​	减少数据的处理量



### 7.1使用Limit分页

```mysql
select * from user limit startIndex.pageSize;
SELECT * from user limit 3; #[0 n];
```



使用Mybatis实现分页，核心SQL

1.接口

```JAVA
 List<User> getUserLimit(Map<String,Integer> map);
```



2.Mapper.xml

```JAVA
 <select id="getUserLimit" parameterType="map" resultType="com.scy.pojo.User">
        select * from mybatis.user limit #{startIndex}, #{pageSize};
    </select>
```



3.测试

```JAVA
 @Test
    public void getByLimit(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        HashMap<String, Integer> map = new HashMap<>();
        map.put("startIndex",0);
        map.put("pageSize", 2);
        List<User> userList = mapper.getUserLimit(map);
        for (User user : userList) {
            System.out.println(user);
        }
        sqlSession.close();
    }
```

### 7.2RowBounds

自己了解即可，一般开发不常用



### 7.3分页插件

让人更懒的工具



## 8.使用注解开发

### 8.1 面向接口编程

之前学过面向对象编程，也学习过接口，但在真正的开发中，很多时候会选择面向接口编程。
根本原因：解耦，可拓展，提高复用，分层开发中，上层不用管具体的实现，大家都遵守共同的标准，使得开发变得容易，规范性更好
在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的。在这种情况下，各个对象内部是如何实现自己的，对系统设计人员来讲就不那么重要了；
而各个对象之间的协作关系则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是指按照这种思想来编程。


### 8.2 使用注解开发

1. 注解在UserMapper接口上实现，并删除UserMapper.xml文件

   ```JAVA
    @Select("select * from user")
       List<User> getUsers();
   ```

2. 需要在mybatis-config.xml核心配置文件中绑定接口

   ```JAVA
     <!--绑定接口！-->
       <mappers>
           <mapper class="com.kuang.dao.UserMapper" />
       </mappers>
   ```

3. 测试

   ```java
   @Test
       public void getUsers(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           List<User> users = mapper.getUsers();
   
           for (User user : users) {
               System.out.println(user);
           }
   
           sqlSession.close();
       }
   
   ```

   **本质：反射机制实现**
   **底层：动态代理！**

### 8.3* mybatis执行机制

自己后面复习吧，听不懂。。。

### 8.4CRUD

1.在MybatisUtils工具类创建的时候自动提交事务

```JAVA
 public interface UserMapper {

    @Select("select * from user")
    List<User> getUsers();

    //方法存在多个参数，所有参数前面必须加上@Param("id")注解
    @Select("select * from user where id=#{id}")
    User getUserById(@Param("id") int id);

    @Insert("insert into user (id,name,pwd) values(#{id},#{name},#{password})")
    int addUser(User user);

    @Update("update user set name=#{name},pwd=#{password} where id=#{id}")
    int updateUser(User user);

    @Delete("delete from user where id = #{uid}")
    int deleteUser(@Param("uid") int id);
    
}

```

2.编写借口，增加注解

```java
  <!--绑定接口！-->
    <mappers>
        <mapper class="com.kuang.dao.UserMapper" />
    </mappers>
```

3.测试

**必须将接口注册绑定到我们的核心配置文件中**

**关于@Param()注解**

- 基本类型的参数或者String类型，需要加上

- 引用类型不需要加

- 如果只有一个基本类型的话，可以忽略，但是建议都加上！

- 我们在SQL中引用的就是我们这里的@Param("")中设定的属性名！

  

  **#{}和${}区别**

  1）#{}是预编译处理，$ {}是字符串替换。

  2）MyBatis在处理#{}时，会将SQL中的#{}替换为?号，使用PreparedStatement的set方法来赋值；MyBatis在处理 $ { } 时，就是把 ${ } 替换成变量的值。

  3）使用 #{} 可以有效的防止SQL注入，提高系统安全性。

## 9.Lombok（偷懒的话可以使用）

使用步骤：

1. 在IDEA中安装Lombok插件！

2. 在项目pom.xml文件中导入Lombok的jar包

   ```JAVA
     <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <version>1.18.10</version>
           </dependency>
   
   ```

   

3.在实体类上加注解即可！

```JAVA
较为常用
@Data
@AllArgsConstructor
@NoArgsConstructor
```

```java
基本注解
@Getter and @Setter
@FieldNameConstants
@ToString
@EqualsAndHashCode
@AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
@Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
@Data
@Builder
@SuperBuilder
@Singular
@Delegate
@Value
@Accessors
@Wither
@With
@SneakyThrows
```

说明

```java
@Data:无参构造、get、set、toString、hashCode、equals
@AllArgsConstructor
@NoArgsConstructor
@EqualsAndHashCode
@ToString
@Getter and @Setter
```

## 10.多对一处理

多对一：

- 多个学生，对应一个老师
- 对于学生而言，**关联**–多个学生，关联一个老师【多对一】
- 对于老师而言，**集合**–一个老师，有很多个学生【一对多】

SQL语句：

```sql
CREATE TABLE `teacher` (
	`id` INT(10) NOT NULL,
	`name` VARCHAR(30) DEFAULT NULL,
	PRIMARY KEY (`id`)
)ENGINE = INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`,`name`) VALUES (1,'秦老师');

CREATE TABLE `student` (
	`id` INT(10) NOT NULL,
	`name` VARCHAR(30) DEFAULT NULL,
	`tid` INT(10) DEFAULT NULL,
	PRIMARY KEY (`id`),
	KEY `fktid`(`tid`),
	CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
)ENGINE = INNODB DEFAULT CHARSET=utf8

INSERT INTO `student`(`id`,`name`,`tid`) VALUES ('1','小明','1');
INSERT INTO `student`(`id`,`name`,`tid`) VALUES ('2','小红','1');
INSERT INTO `student`(`id`,`name`,`tid`) VALUES ('3','小张','1');
INSERT INTO `student`(`id`,`name`,`tid`) VALUES ('4','小李','1');
INSERT INTO `student`(`id`,`name`,`tid`) VALUES ('5','小王','1');

```

### 10.1测试环境搭建

1. 导入Lombok
2. 新建实体类Teacher，Student
3. 建立Mapper接口
4. 建立Mapper.XML文件
5. 在核心配置文件中绑定注册我们的Mapper接口或者文件！【方式很多,随心选】
6. 测试查询是否能够成功！

### 10.2按照结果嵌套处理

```JAVA
<!--按照结果嵌套处理    -->
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid,s.name sname,t.name tname
        from mybatis.student s,mybatis.teacher t
        where s.tid = t.id
    </select>

    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
        </association>
    </resultMap>

```

### 10.3按照查询嵌套处理

```JAVA
 <!--
      思路：
          1.查询所有的学生信息
          2.根据查询出来的学生的tid，寻找对应的老师！ 子查询-->
    <select id="getStudent" resultMap="StudentTeacher">
        select * from mybatis.student
    </select>

    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <!--  复杂的属性，我们需要单独处理 对象：association 集合：collection      -->
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>

    <select id="getTeacher" resultType="Teacher">
        select * from mybatis.teacher where id = #{id}
    </select>
```

回顾Mysql多对一查询方式：

- 子查询

- 联表查询

## 11 、一对多处理

比如：一个老师拥有多个学生！
对于老师而言，就是一对多的关系！

### 11.1环境搭建

1. 环境搭建，和刚才一样
   **实体类：**

```java
@Data
public class Student {
    private int id;
    private String name;
    private int tid;

}
```

```JAVA
@Data
public class Teacher {
    private int id;
    private String name;

    //一个老师拥有多个学生
    private List<Student> students;
}
```

### 11.2按照结果嵌套处理

```JAVA
 <!--    按结果嵌套查询-->
    <select id="getTeacher" resultMap="TeacherStudent">
            SELECT  s.id sid,s.name sname,t.name tname,t.id,tid
            from student s,teacher t
            where s.tid = t.id and t.id = #{tid}
    </select>

    <resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
        <!--  复杂的属性，我们需要单独处理 对象：association 集合：collection
             javaType="" 指定属性的类型！
             集合中的泛型信息，我们使用ofType获取
             -->
        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>
```

### 11.3按照查询嵌套处理

```JAVA
    <select id="getTeacher2" resultMap="TeacherStudent2">
        select * from mybatis.teacher where id = #{tid}
    </select>

    <resultMap id="TeacherStudent2" type="Teacher">
        <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
    </resultMap>

    <select id="getStudentByTeacherId" resultType="Student">
        select * from  mybatis.student where tid = #{tid}
    </select>

```

**小结**
1.关联-association【多对一】
2.集合-collection【一对多】
3.javaType & ofType
		javaType 用来指定实体类中属性的类型
		ofType 用来指定映射到List或者集合中的pojo类型，泛型中的约束类型！

**注意点：**

保证SQL的可读性，尽量保证通俗易懂
注意一对多和多对一中，属性名和字段的问题！
如果问题不好排查错误，可以使用日志，建议使用Log4j



**面试高频**

- Mysql引擎
- InnoDB底层原理
- 索引
- 索引优化

## 13. 缓存

### 13.1什么是缓存

- 存在内存中的临时数据。
- 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库查询文件）查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。

### 13.2为什么使用缓存

- 减少和数据库的交互次数，减少系统开销，提高系统效率。

什么样的数据能使用缓存

- 经常查询并且不经常改变的数据。【可以使用缓存】

### 13.3Mybatis缓存

MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。

MyBatis系统中默认定义了两级缓存：**一级缓存和二级缓存**

默认情况下，只有一级缓存开启。（SqlSession级别的缓存，也称为本地缓存）

二级缓存需要手动开启和配置，他是基于namespace级别的缓存。

为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

#### 一级缓存

一级缓存也叫本地缓存：

与数据库同一次会话期间查询到的数据会放在本地缓存中。

以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库；

测试

1. 在mybatis中加入日志，方便测试结果
2. 编写接口方法

```XML
//根据id查询用户
User queryUserById(@Param("id") int id);
```

3.接口对应的Mapper文件

```XML
<select id="queryUserById" resultType="user">
select * from user where id = #{id}
</select>
```

4.测试

```XML
@Test
public void testQueryUserById(){
SqlSession session = MybatisUtils.getSession();
UserMapper mapper = session.getMapper(UserMapper.class);

User user = mapper.queryUserById(1);
System.out.println(user);
User user2 = mapper.queryUserById(1);
System.out.println(user2);
System.out.println(user==user2);

session.close();
}
```

![](F:\邵辰宇\ideal\缓存测试图1.jpg)

> 级缓存失效的四种情况

一级缓存是SqlSession级别的缓存，是一直开启的，我们关闭不了它；

一级缓存失效情况：没有使用到当前的一级缓存，效果就是，还需要再向数据库中发起一次查询请求！

##### 1.sqlSession不同

```JAVA
@Test
public void testQueryUserById(){
SqlSession session = MybatisUtils.getSession();
SqlSession session2 = MybatisUtils.getSession();
UserMapper mapper = session.getMapper(UserMapper.class);
UserMapper mapper2 = session2.getMapper(UserMapper.class);

User user = mapper.queryUserById(1);
System.out.println(user);
User user2 = mapper2.queryUserById(1);
System.out.println(user2);
System.out.println(user==user2);

session.close();
session2.close();
}
```

观察结果：发现发送了两条SQL语句！

结论：**每个sqlSession中的缓存相互独立**

##### 2.sqlSession相同，查询条件不同

```XML
@Test
public void testQueryUserById(){
	SqlSession session = MybatisUtils.getSession();
	UserMapper mapper = session.getMapper(UserMapper.class);
	UserMapper mapper2 = session.getMapper(UserMapper.class);

	User user = mapper.queryUserById(1);
    System.out.println(user);
	User user2 = mapper2.queryUserById(2);
	System.out.println(user2);
	System.out.println(user==user2);

	session.close();
}
```

观察结果：发现发送了两条SQL语句！很正常的理解

结论：**当前缓存中，不存在这个数据**

##### 3.sqlSession相同，两次查询之间执行了增删改操作！

增加方法

```XML
//修改用户
int updateUser(Map map);
```

编写SQL

```XML
<update id="updateUser" parameterType="map">
  update user set name = #{name} where id = #{id}
</update>
```

测试

```JAVA
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);

   HashMap map = new HashMap();
   map.put("name","kuangshen");
   map.put("id",4);
   mapper.updateUser(map);

   User user2 = mapper.queryUserById(1);
   System.out.println(user2);

   System.out.println(user==user2);

   session.close();
}
```

观察结果：查询在中间执行了增删改操作后，重新执行了

结论：**因为增删改操作可能会对当前数据产生影响**

##### 4、sqlSession相同，手动清除一级缓存

```JAVA
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);

   session.clearCache();//手动清除缓存

   User user2 = mapper.queryUserById(1);
   System.out.println(user2);

   System.out.println(user==user2);

   session.close();
}
```

一级缓存就是一个map

#### 二级缓存

二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存

**基于namespace级别的缓存，一个名称空间，对应一个二级缓存**；

> 工作机制

一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；

如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中；

新的会话查询信息，就可以从二级缓存中获取内容；

不同的mapper查出的数据会放在自己对应的缓存（map）中；

> 使用步骤

1.开启全局缓存 【mybatis-config.xml】

```JAVA
<setting name="cacheEnabled" value="true"/>
```

2.去每个mapper.xml中配置使用二级缓存，这个配置非常简单；【xxxMapper.xml】

```JAVA
<cache/>

官方示例=====>查看官方文档
<cache
 eviction="FIFO"
 flushInterval="60000"
 size="512"
 readOnly="true"/>
这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突
```

3.代码测试

所有的实体类先实现序列化接口

测试代码

```JAVA
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   SqlSession session2 = MybatisUtils.getSession();

   UserMapper mapper = session.getMapper(UserMapper.class);
   UserMapper mapper2 = session2.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);
   session.close();

   User user2 = mapper2.queryUserById(1);
   System.out.println(user2);
   System.out.println(user==user2);

   session2.close();
}
```

**结论**

只要开启了二级缓存，我们在同一个Mapper中的查询，可以在二级缓存中拿到数据

查出的数据都会被默认先放在一级缓存中

只有会话提交或者关闭以后，一级缓存中的数据才会转到二级缓
