# SpringBoot启动banner更改

> 可以通过向类路径中添加一个banner.txt文件或设置spring.banner来更改在console上打印的banner。属性指向此类文件的位置。如果文件的编码不是UTF-8，那么可以设置spring.banner.charset。除了文本文件，还可以添加横幅。将gif、banner.jpg或banner.png图像文件保存到类路径或设置spring.banner.image。位置属性。图像被转换成ASCII艺术形式，并打印在任何文本横幅上面。

<img style="display: block; margin: 0 auto;zoom: 50%;" src="blog/sprintboot/Others/picture/1610367914(1).jpg"/>



------

?>先介绍一个可以制作自定义banner的网站，传送门：[http://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20](http://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type Something )



新建banner.txt放入resources下，启动可以看到

另外spring提供了几种类型来设定banner：

1.`${AnsiColor.BRIGHT_CYAN}`来设定banner字体；

2.`${AnsiBackground.BRIGHT_CYAN}`来设定banner背景颜色；

3.`${AnsiStyle.UNDERLINE}`设定字体样式；

4.Application Version: ${xxx.version}  设定项目版本；

5.Spring Boot Version: ${spring-boot.version}  设定SpringBoot版本；



> 也在配置文件中加入配置

!> <a href="#/blog/sprintboot/Others/Banner - 样式.md"  target="_blank">Banner样式</a>