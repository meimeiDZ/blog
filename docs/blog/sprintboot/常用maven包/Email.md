## 1、设置163邮箱获取授权

<img style="display: block; margin: 0 auto;zoom: 50%;" src="_images/zh-cn/springboot/1606111551(1).jpg" />



> 勾选后安装提示会叫你设置授权密码之类的：记住授权的密码
>
> 163 ： https://mail.163.com/
>
> QQ :  https://mail.qq.com/



## 2、配置文件

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <lombok.version>1.18.6</lombok.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
        <!-- lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <!-- 插件管理 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

#### application.yml

```yml
spring:
  mail:
    host: smtp.163.com #设置邮箱主机
    username: XXXXXXXXXXXXXX #发送邮件的发送者	
    password: XXXXXXXXXXXXXX #163邮箱获取授权码
    default-encoding: utf-8
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true

server:
  port: 8000
```



## 3、目录结构

```
|-- java
|   |-- com.example.demo
|   |   |-- controller
|   |   |   	   EmailController
|   |   |-- dto
|   |   |   	   RegisterDto
|   |   |-- entity
|   |   |   	   User
|   |   |-- sevice
|   |   |   IMailService
|   |   |	|-- impl
|   |   |   	   MailServiceImpl
|   |   |-- utils
|   |   |   	   EmailCode
|   |   |   	   Result
|   |   |   	   ResultEnum
|   |   | DemoApplication
|-- resources
|       |-- application.yml
```



#### Controller

```java
package com.example.demo.controller;

import com.example.demo.dto.RegisterDto;
import com.example.demo.service.IMailService;
import com.example.demo.utils.Result;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Resource
    private IMailService emailService ;

    /**
     * 发送简单的文本
     */
    @GetMapping("/sendText")
    public Result sendEmail (){
        return emailService.send();
    }

    /**
     * 发送验证码到指定邮箱
     */
    @GetMapping("/verCode")
    public Result verCode(String receiver) {
        return emailService.getCode(receiver);
    }

    /**
     * 注册
     */
    @PostMapping("/addUser")
    public Result addUser (@RequestBody RegisterDto registerDto){
        return emailService.addUser(registerDto.getUser(),registerDto.getReceiver(),registerDto.getVerCode());
    }
}
```



#### Service

```java
// IMailService
package com.example.demo.service;

import com.example.demo.entity.User;
import com.example.demo.utils.Result;

public interface IMailService {

    /**
     * 发送验证码到指定邮箱
     * @param receiver 接受地址
     */
    Result getCode(String receiver);

    /**
     * 注册用户
     */
    Result addUser(User user,String receiver, String verCode);

    Result send();
}
// ----------------------------------------------------------------------------------------------------
package com.example.demo.service.impl;

import com.example.demo.entity.User;
import com.example.demo.service.IMailService;
import com.example.demo.utils.Result;
import org.springframework.mail.MailSender;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.Date;

import static com.example.demo.utils.EmailCode.generateVerCode;
import static com.example.demo.utils.EmailCode.getMinute;

/**
 * 邮箱发送实现类
 */
@Service
@Component
public class MailServiceImpl implements IMailService {

    /**
     * 验证码
     */
    private String code;
    /**
     * 发送时间
     */
    private Date sendTime;
    /**
     * Email
     */
    private String emailAddress;

    @Resource
    private MailSender mailSender;

    public Result send() {
        // new 一个简单邮件消息对象
        SimpleMailMessage message = new SimpleMailMessage();
        // 和配置文件中的的username相同，相当于发送方
        message.setFrom("发件人邮箱地址");
        // 收件人邮箱
        message.setTo("收件人邮箱地址");
        // 标题
        message.setSubject("欢迎访问");
        // 正文
        message.setText("注册码：" + generateVerCode());
        // 发送
        mailSender.send(message);
        return Result.success();
    }

    @Override
    @Async
    public Result getCode(String receiver) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setSubject("注册码");//设置邮件标题
        code = generateVerCode();
        emailAddress = receiver;
        sendTime = new Date();
        message.setFrom("发件人邮箱地址");//发件人
        message.setTo(receiver);//收件人
        message.setText("尊敬的用户，您好：\n"
                + "\n本次请求的邮件验证码为：" + code + "，本验证码5分钟内有效，请及时输入。（请勿泄露此验证码）\n"
                + "\n如非本人操作，请忽略该邮件。\n(这是一封自动发送的邮件，请不要直接回复）");    //设置邮件正文
        mailSender.send(message);//发送邮件
        return Result.success();
    }

    @Override
    public Result addUser(User user, String receiver, String verCode) {
        Date date = new Date();
        if (!receiver.equals(emailAddress)) {
            return Result.error("邮箱地址错误！！！");
        }
        //判断验证码
        if (getMinute(sendTime, date) > 5) {
            return Result.error("验证码已经失效！！！");
        }
        if (!verCode.equals(code)) {
            return Result.error("验证码不正确！！！");
        }
        user.setCreateTime(new Date());
        code = null;
        emailAddress = null;
        return Result.success(user);
    }
}
```

> 注：Service 中，getCode() 方法上有一个 @Async 表示异步，可以在邮件未发送完成时就返回，而不必等待太长时间
> ，必须在总配置类上加 @EnableAsync 注解才可以生效。



#### Dto & Entity

```java
// Dto 
package com.example.demo.dto;

import com.example.demo.entity.User;
import lombok.Data;

@Data
public class RegisterDto {

    // 注册人员
    private User user;

    // 人员邮箱地址
    private String receiver;

    // 注册码
    private String verCode;
}
// ----------------------------------------------------------------------------------------------------
// Entity
package com.example.demo.entity;

import lombok.Data;
import java.util.Date;

@Data
public class User {
    private String username;
    private String password;
    private Date createTime;
}
```



#### Utils

```java
package com.example.demo.utils;

import java.security.SecureRandom;
import java.util.Date;
import java.util.Random;

/**
 * 验证码的工具类
 */
public class EmailCode {

    private static final String SYMBOLS = "0123456789";
    /**
     * Math.random生成的是一般随机数，采用的是类似于统计学的随机数生成规则，其输出结果很容易预测，因此可能导致被攻击者击中。
     * 而SecureRandom是真随机数，采用的是类似于密码学的随机数生成规则，其输出结果较难预测，若想要预防被攻击者攻击，最好做到使攻击者根本无法，或不可能鉴别生成的随机值和真正的随机值。
     */
    private static final Random RANDOM = new SecureRandom();

    public static String generateVerCode() {
        char[] nonceChars = new char[6];
        for (int i = 0; i < nonceChars.length; i++) {
            nonceChars[i] = SYMBOLS.charAt(RANDOM.nextInt(nonceChars.length));
        }
        return new String(nonceChars);
    }

    /**
     *计算两个日期的分钟差
     */
    public static int getMinute(Date fromDate, Date toDate) {
        return (int) (toDate.getTime() - fromDate.getTime()) / (60 * 1000);
    }
}
// ----------------------------------------------------------------------------------------------------
package com.example.demo.utils;

import lombok.Getter;

/**
 * 返回数据工具类
 */
@Getter
public class Result<T> {
    /**
     * 错误码
     */
    private Integer code;

    /**
     * 提示信息
     */
    private String msg;

    /**
     * 具体的内容
     */
    private T data;

    /**
     * 成功
     * @param object 需要返回的数据
     * @return Result
     */
    public static Result success(Object object) {
        return new Result(object);
    }

    /**
     * 成功
     * @return Result
     */
    public static Result success() {
        return success(null);
    }

    /**
     * 错误
     * @param code 状态码
     * @param msg 消息
     * @return Result
     */
    public static Result error(Integer code, String msg) {
        return new Result(code, msg);
    }

    /**
     * 错误
     * @param resultEnum 错误枚举类
     * @return Result
     */
    public static Result error(ResultEnum resultEnum) {
        return new Result(resultEnum);
    }

    /**
     * 错误
     * @param msg 错误信息
     * @return Result
     */
    public static Result error(String msg) {
        return new Result(msg);
    }

    /**
     * 私有化工具类 防止被实例化
     */
    private Result() {}

    /**
     * 成功
     * @param data
     */
    private Result(T data) {
        this.code = ResultEnum.SUCCESS.getCode();
        this.msg = ResultEnum.SUCCESS.getMsg();
        this.data = data;
    }

    /**
     * 失败
     * @param msg
     */
    private Result(String msg) {
        this.code = -9999;
        this.msg = msg;
    }

    /**
     * 失败
     * @param resultEnum
     */
    private Result(ResultEnum resultEnum) {
        this.code = resultEnum.getCode();
        this.msg = resultEnum.getMsg();
    }

    /**
     * 失败
     * @param code
     * @param msg
     */
    private Result(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }
}
// ----------------------------------------------------------------------------------------------------
package com.example.demo.utils;

import lombok.Getter;

@Getter
public enum ResultEnum {

    /**
     * 成功
     */
    SUCCESS(666, "成功"),

    /**
     * 未知异常
     */
    UNKNOWN_EXCEPTION(100, "未知异常"),

    /**
     * 格式错误
     */
    FORMAT_ERROR(101, "参数格式错误"),

    /**
     * 超时
     */
    TIME_OUT(102, "超时"),

    /**
     * 添加失败
     */
    ADD_ERROR(103, "添加失败"),

    /**
     * 更新失败
     */
    UPDATE_ERROR(104, "更新失败"),

    /**
     * 删除失败
     */
    DELETE_ERROR(105, "删除失败"),

    /**
     * 查找失败
     */
    GET_ERROR(106, "查找失败"),

    /**
     * 参数类型不匹配
     */
    ARGUMENT_TYPE_MISMATCH(107, "参数类型不匹配"),

    /**
     * 请求方式不支持
     */
    REQ_METHOD_NOT_SUPPORT(110,"请求方式不支持"),
    ;

    private Integer code;

    private String msg;

    ResultEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    /**
     * 通过状态码获取枚举对象
     * @param code 状态码
     * @return 枚举对象
     */
    public static ResultEnum getByCode(int code){
        for (ResultEnum resultEnum : ResultEnum.values()) {
            if(code == resultEnum.getCode()){
                return resultEnum;
            }
        }
        return null;
    }
}
```

#### Application

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@EnableAsync // 异步
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```



## 4、运行结果

#### 1.发送简单的文本

<img style="display: block; margin: 0 auto;zoom: 50%;" src="_images/zh-cn/springboot/1606124130.png" />



#### 2.验证码注册

获取验证码

<img style="display: block; margin: 0 auto;zoom: 50%;" src="_images/zh-cn/springboot/1606124450(1).jpg" />

验证

<img style="display: block; margin: 0 auto;zoom: 50%;" src="_images/zh-cn/springboot/1606124570(1).jpg" />