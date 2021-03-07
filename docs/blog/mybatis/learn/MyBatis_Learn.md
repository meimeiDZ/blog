# MyBatis

?> 根据狂神回忆 [MyBatis3](https://mybatis.org/mybatis-3/zh/index.html)

> `MyBatis 是一款优秀的持久层框架`
>
> 作用：完成持久化工作；帮助程序员将数据存入到数据库中；
>
> !> 数据持久化
>
> 1. 持久化就是将程序的数据在持久状态和瞬时状态转化的过程
> 2. 内存：**断电即失**
> 3. 数据库（Jdbc）,io文件持久化。



!>  使用 MyBatis 的准备：搭建环境 ---> JDK1.8 / Mysql5.7 / Maven3.6.1



## 1 、入门CRUD

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>MyBatis</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>mybatis-01-select</module>
    </modules>
    <!--  父工程  -->

    <dependencies>
        <!--mysql驱动-->
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
</project>
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <!--  多环境  default默认-->
    <environments default="development">
        <!--    开发    -->
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatis?userSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
        <!--    测试    -->
        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatis?userSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!--  mapper文件扫描  -->
    <mappers>
        <mapper resource="com/mmdz/dao/UserMapper.xml"/>
        <!--    包扫描    -->
<!--          <package name="com.mmdz.dao"/>-->
    </mappers>
</configuration>
```

MybatisUtils 工具类

```java
package com.mmdz.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

/**
 * 从 XML 中构建 SqlSessionFactory
 */
public class MybatisUtils {

    static SqlSessionFactory sqlSessionFactory = null;

    // 构建 SqlSessionFactory
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 获取sqlSession
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

Dao

```java
package com.mmdz.dao;

import com.mmdz.entity.User;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;

import java.util.List;

public interface UserDao {
    // Select
    List<User> getUserListByMapper();
    @Select("select * from user")
    List<User> getUserListByAnnotation();

    // Insert
    int AddUserByMapper(User user);
    @Insert("  insert into user(id,username,password) values (#{id},#{username},#{password})")
    int AddUserByAnnotation(User user);

    // Update
    int UpdateUserByMapper(User user);
    @Update("update user set username=#{username},password=#{password} where id=#{id}")
    int UpdateUserByAnnotation(User user);

    // Update
    int DeleteUserByMapper(int userId);
    @Delete("delete from user where id=#{userId}")
    int DeleteUserByAnnotation(int userId);
}
```

Mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mmdz.dao.UserDao">
    <select id="getUserListByMapper" resultType="com.mmdz.entity.User">
        select * from user
    </select>

    <insert id="AddUserByMapper" parameterType="com.mmdz.entity.User">
        insert into user(id,username,password) values (#{id},#{username},#{password})
    </insert>

    <update id="UpdateUserByMapper" parameterType="com.mmdz.entity.User">
        update user set username=#{username},password=#{password} where id=#{id}
    </update>

    <delete id="DeleteUserByMapper" parameterType="int">
        delete from user where id=#{userId}
    </delete>
</mapper>
```

!>注意：`增删改需要提交事务`

Test

```java
package com.mmdz;

import com.mmdz.dao.UserDao;
import com.mmdz.entity.User;
import com.mmdz.util.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.HashMap;
import java.util.Map;

public class test {

    @Test
    public void testDelete(){
        // 获取SqlSession
        SqlSession session = MybatisUtils.getSqlSession();
        UserDao mapper = session.getMapper(UserDao.class);
        mapper.DeleteUserByMapper(6);

        //增删改一定要提交事务
        session.commit();
        //关闭sqlSession
        session.close();
        System.out.println("Delete Success");
    }

    @Test
    public void testUpdate(){
        // 获取SqlSession
        SqlSession session = MybatisUtils.getSqlSession();
        UserDao mapper = session.getMapper(UserDao.class);
        User user1  = new User(5,"InsertByMapper","666");
        User user2  = new User(5,"InsertByMapper","666");
        mapper.UpdateUserByMapper(user2);

        //增删改一定要提交事务
        session.commit();
        //关闭sqlSession
        session.close();
        System.out.println("Update Success");
    }

    @Test
    public void testInsert(){
        // 获取SqlSession
        SqlSession session = MybatisUtils.getSqlSession();
        UserDao mapper = session.getMapper(UserDao.class);
        User user1  = new User(5,"InsertByMapper","666");
        User user2  = new User(7,"InsertByMap","666");
        Map map = new HashMap<>();
        map.put("id",7);
        map.put("name","InsertByMap");
        map.put("pwd","666");
        mapper.AddUserByMap(map);

        //增删改一定要提交事务
        session.commit();
        //关闭sqlSession
        session.close();
        System.out.println("Insert Success");
    }

    @Test
    public void testSelect(){
        // 获取SqlSession
        SqlSession session = MybatisUtils.getSqlSession();
        UserDao mapper = session.getMapper(UserDao.class);
        User user  = new User(5,"黑子","666");
        mapper.AddUserByMapper(user);
//        List<User> list1 = mapper.getUserListByMapper();
//        List<User> list2 = mapper.getUserListByAnnotation();
//        list2.stream().forEach(e -> System.out.println(e));
        //关闭sqlSession
        session.close();
    }
}
```



------

`Map使用`

> 我们的实体类，或者数据库中的表，字段或者参数过多，我们应该考虑使用Map!

```java
// dao
int AddUserByMap(Map<String,Object> mpa);

// xml
<!--对象中的属性可以直接取出来 传递map的key-->
<insert id="addUser2" parameterType="map">
    insert into user (id,name,password) values (#{userid},#{username},#{userpassword})
</insert>

// test
    @Test
    public void test3(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        HashMap<String, Object> map = new HashMap<String, Object>();
        map.put("userid",4);
        map.put("username","王虎");
        map.put("userpassword",789);
        mapper.addUser2(map);
        //提交事务
        sqlSession.commit();
        //关闭资源
        sqlSession.close();
    }
```

> Map传递参数，直接在sql中取出key即可！ 【parameter=“map”】对象传递参数，直接在sql中取出对象的属性即可！ 【parameter=“Object”】只有一个基本类型参数的情况下，可以直接在sql中取到多个参数用Map , **或者注解！**