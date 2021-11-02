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

