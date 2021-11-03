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

![](C:\Users\Jack\Desktop\123456.png)

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

