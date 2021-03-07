# 问题汇总

?>  **使用MyBatis可能会遇到的问题**



------

# 1 、MyBatisLearn

**可能会：**

1. `配置文件没有注册`
2. `绑定接口错误` （mapper.xml的namespace一致）
3. `方法名不对`（mapper.xml的CRUD标签的id一致）
4. `返回类型不对`
5. `Maven导出资源问题`
6. `增删改需要提交事务`
7. `xxxMapper.xml文件位置放置错误`
8. `map使用`



!>配置文件没有注册

> ```bash
> org.apache.ibatis.binding.BindingException: Type interface com.mmdz.dao.UserDao is not known to the MapperRegistry.
> 
> 	at org.apache.ibatis.binding.MapperRegistry.getMapper(MapperRegistry.java:47)
> 	at org.apache.ibatis.session.Configuration.getMapper(Configuration.java:779)
> ```

原因：实现UserDao的XML文件没有扫描到。（mybatis-config.xml缺少xml文件扫描路径）

```xml
    <!--  mapper文件扫描  -->
    <mappers>
<!--        <mapper resource="com.mmdz.dao/UserMapper.xml"/>-->
        <!--    包扫描    -->
<!--          <package name="com.mmdz.dao"/>-->
    </mappers>
```

------

!>xxxMapper.xml文件位置放置错误

> ```bash
> Caused by: java.io.IOException: Could not find resource com/mybatis/mapper/StudentMapper.xml
> 	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:108)
> 	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:95)
> ```

原因：IDEA是不会编译src的java目录的xml文件，所以在Mybatis的配置文件中找不到xml文件！

`解决方案1：`

> 不将xml放到src目录下面，将xxxMapper.xml放到Maven构建的resource目录下面！
>

`解决方案2：`

> 在Maven的pom文件中，添加下面代码：

> ```xml
> <build>
>         <resources>
>             <resource>
>                 <directory>src/main/java</directory>
>                 <includes>
>                     <include>**/*.xml</include>
>                 </includes>
>             </resource>
>         </resources>
>     </build>
> ```

`解决方案3：`

> 测试时候可能存在只有 mapper resource 这种方式加载不到资源，不过可以使用其他方式，即的url class和package都可以，如果想解决问题的话，可以不使用resource这种方式！

------

!>使用map参数类型

> ```bas
> java.lang.IllegalArgumentException: Parameter Maps collection does not contain value for com.mmdz.dao.UserDao.map
> 
> 	at org.apache.ibatis.session.Configuration$StrictMap.get(Configuration.java:964)
> 	at org.apache.ibatis.session.Configuration.getParameterMap(Configuration.java:694)
> ```

> `注意：`SQL映射的XML文件：mybatis官方已经将parameterMap废弃了，现在使用parameterType来处理。

------

