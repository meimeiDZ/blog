# Cglib代理

上面的静态代理和动态代理模式都是要求目标对象是实现一个接口的目标对象,但是有时候目标对象只是一个单独的对象,并没有实现任何的接口,这个时候就可以使用以目标对象子类的方式类实现代理,这种方法就叫做:Cglib代理

> Cglib代理,也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.
>
> - JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.
> - Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如Spring AOP和synaop,为他们提供方法的`interception(拦截)`
> - Cglib包的底层是通过使用一个小而快的字节码处理框架ASM来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉.

```
Cglib子类代理实现方法:

​	1.需要引入cglib的jar文件,但是Spring的核心包中已经包括了Cglib功能,所以直接引入`spring-core-3.2.5.jar`即可.

​	2.引入功能包后,就可以在内存中动态构建子类

​	3.代理的类不能为final,否则报错

​	4.目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.
```

> spring-core jar 下载   https://repo.spring.io/webapp/#/artifacts/browse/tree/General/libs-release-local/org/springframework/spring-core

------

`代码示例:`

**目标对象**

```java
public class BuyHouse {
    public void buyHouse(String s) {
        System.out.println(s + "成功买房！");
    }
}
```

**Cglib代理工厂**

```java
public class ProxyFactory implements MethodInterceptor {

    // 目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    // 给目标对象创建一个代理对象
    public Object getProxyInstance(){
        // 1.工具类
        Enhancer enhancer = new Enhancer();
        // 2.设置父类
        enhancer.setSuperclass(target.getClass());
        // 3.设置回调函数
        enhancer.setCallback(this);
        // 4.创建子类(代理对象)
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("开始操作...");

        //执行目标对象的方法
        Object returnValue = method.invoke(target, objects);

        System.out.println("结束操作...");

        return returnValue;
    }
}
```

**测试**

```java
public class Test {
    public static void main(String[] args) {
        //目标对象
        BuyHouse target = new BuyHouse();

        //代理对象
        BuyHouse proxy = (BuyHouse)new ProxyFactory(target).getProxyInstance();

        //执行代理对象的方法
        proxy.buyHouse("qwqwqwqw");
    }
}
```

**结果**

```bash
目标对象：class com.proxy.cglib.BuyHouse
代理对象：class com.proxy.cglib.BuyHouse$$EnhancerByCGLIB$$dc4148b3
开始操作...
qwqwqwqw成功买房！
结束操作...
```

------

`Cglib代理总结`

?>CGLIB创建的动态代理对象比JDK创建的动态代理对象的性能更高，
但是CGLIB创建代理对象时所花费的时间却比JDK多得多。所以对于单例的对象，
因为无需频繁创建对象，用CGLIB合适，反之使用JDK方式要更为合适一些。
同时`由于CGLib是采用动态创建子类的方法，对于final修饰的方法无法进行代理`。