?> `Swagger:`       `Swagger 2.9.2` / `Swagger 3.0`

## 1、生产文档的注解

?>swagger通过注解表明该接口会生成文档，包括接口名、请求方法、参数、返回信息的等等。

- `@Api`：修饰整个类，描述Controller的作用
- `@ApiOperation`：描述一个类的一个方法，或者说一个接口
- `@ApiParam`：单个参数描述
- `@ApiModel`：用对象来接收参数
- `@ApiProperty`：用对象接收参数时，描述对象的一个字段
- `@ApiResponse`：HTTP响应其中1个描述
- `@ApiResponses`：HTTP响应整体描述
- `@ApiIgnore`：使用该注解忽略这个API
- `@ApiError` ：发生错误返回的信息
- `@ApiImplicitParam`：一个请求参数
- `@ApiImplicitParams`：多个请求参数

------

## 2、导入Swagger的包

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
		<!--  V3  -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
    
    	<!--  V2  -->
        <!--        <dependency>-->
        <!--            &lt;!&ndash;添加Swagger依赖 &ndash;&gt;-->
        <!--            <groupId>io.springfox</groupId>-->
        <!--            <artifactId>springfox-swagger2</artifactId>-->
        <!--            <version>2.9.2</version>-->
        <!--        </dependency>-->
        <!--        <dependency>-->
        <!--            &lt;!&ndash;添加Swagger-UI依赖 &ndash;&gt;-->
        <!--            <groupId>io.springfox</groupId>-->
        <!--            <artifactId>springfox-swagger-ui</artifactId>-->
        <!--            <version>2.9.2</version>-->
        <!--        </dependency>-->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.4.3</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

------

## 3、配置Swagger的Config

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
//@EnableSwagger2   // V2
@EnableOpenApi	    // V3
public class SwaggerConfig {

    @Value("${swagger.enable}")
    private Boolean enable; // 是否开启

    @Value("${swagger.author}")
    private String author; // 作者

    @Value("${swagger.version}")
    private String version; // 版本号

    @Value("${swagger.title}")
    private String title; // 标题

    @Bean
    public Docket createRestApi() {
        //return new Docket(DocumentationType.SWAGGER_2)  // V2
        return new Docket(DocumentationType.OAS_30)       // V3
                .enable(enable)
                .apiInfo(apiInfo())
                .select()
                //为当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.mmdz.swagger.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //页面标题
                .title(title)
                //创建人
                .contact(new Contact(author, "http://www.baidu.com", ""))
                //版本号
                .version(version)
                //描述
                .description("API 描述")
                .build();
    }
}
```

------

## 4、yml配置文件

```yml
server:
  port: 8000

swagger:
  #是否开启
  enable: true
  #作者
  author: mmdz
  #版本号
  version: 1.0.0
  #接口标题
  title: swagger

```

------

## 5、使用

```java
import com.mmdz.swagger.entity.User;
import io.swagger.annotations.*;
import org.springframework.web.bind.annotation.*;

@Api(tags = "用户管理")
@RestController
public class UserController {

    @ApiOperation("添加用户")
    @PostMapping("/add")
    public User add(@ApiParam("用户") User user){
        return new User();
    }

    @ApiOperation("修改用户")
    @PostMapping("/update")
    public String update() {
        return "修改";
    }

    @ApiOperation("删除用户")
    @GetMapping("/delete")
    public boolean delete(@ApiParam("用户编号") Integer id) {
        return true;
    }

    @ApiOperation("查询用户")
    @GetMapping("/query")
    @ApiResponses(value = { @ApiResponse(code = 1000, message = "成功"), @ApiResponse(code = 1001, message = "失败"),
            @ApiResponse(code = 1002,message = "缺少参数") })
    @ApiImplicitParams({
            @ApiImplicitParam(name = "name", value = "电影名", dataType = "String", paramType = "query", required = true),})
    public User query(@RequestParam String name) {
        User user = new User();
        user.setUsername("name");
        user.setPassword("password");
        return  user;
    }
}
```

> 省略实体 User ...

------

?> 新版本和老版本的区别主要体现在以下 4 个方面：

1. 依赖项的添加不同：新版本只需要添加一项，而老版本需要添加两项；
2. 启动 Swagger 的注解不同：新版本使用的是 `@EnableOpenApi`，而老版本是 `@EnableSwagger2`；
3. Docket（文档摘要信息）的文件类型配置不同：新版本配置的是 `OAS_3`，而老版本是`SWAGGER_2`；
4. Swagger UI 访问地址不同：新版本访问地址是 `http://localhost:8080/swagger-ui/`，而老版本访问地址是 `http://localhost:8080/swagger-ui.html`。

------

## 6、Swagger增强-Knife4j

> 配置好Swagger后，添加 Knife4j 的依赖即可

```xml
<!-- https://mvnrepository.com/artifact/com.github.xiaoymin/knife4j-spring-boot-starter -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```

?>我们可以通过 `http://localhost:8080/doc.html` 访问 Knife4j 的主页，如下图所示：

<img style="display: block; margin: 0 auto;zoom: 50%;" src="blog/sprintboot/常用maven包/picture/微信图片_20210322135256.png"/>