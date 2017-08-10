---
title: Spring相关
date: 2017-08-08 15:46:40
tags:
	- Spring
---
> ***注意：部分浏览器会拦截MathJax生成数学公式，如出现无法显示公式，请允许浏览器加载脚本***

本篇文章记录在使用Spring过程中遇到的一些问题，记下来用于参考。

1. 集成SpringMVC的时候的配置；
2. Controller不生效有谁来背锅；
3. 集成maven时无法找到spring.xsd文件；

## 1. 集成SpringMVC的时候的配置  
### a) web.xml的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <!-- 此处设置为/*会对所有的请求进行拦截，如果设置为/则不会拦截jsp的请求 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```
部分教程会给出`web.xml`配置中加入下面的配置：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext.xml</param-value>
    <!-- 默认是/WEB-INF/applicationContext.xml -->
 </context-param>
 
 <listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
 </listener>
```
其实这涉及到Spring中父子容器的问题，此处不再赘述，给出一篇参考博文：[Spring和SpringMVC父子容器关系初窥](http://www.cnblogs.com/hafiz/p/5875740.html)

简而言之就是：如果配置了父子容器的话，最好要分开Spring和Spring MVC的容器，MVC作为子容器可以访问Spring父容器的Bean，但是父容器无法访问子容器的Bean。

**个人推荐：**在业务不是很复杂的时候仅使用Spring MVC子容器，可以避免很多Bean无法注入的问题。

### b) applicationContext.xml配置

```xml
 <mvc:annotation-driven />
 <!-- 单个*匹配一层目录, 两个**匹配多层目录,推荐使用basepackage.**表示 -->
 <context:component-scan base-package="com.xxxx.**"/>
```
`<context:component-scan />`标签已经具备了`<context: annotation-driven />`的功能,请参考:[ &lt;context:component-scan &gt;详解](http://www.cnblogs.com/fightingcoding/p/component-scan.html)

`<mvc:annotation-driven />`标签则能够帮助我们实现mvc路由及其他一些功能，请参考：[spring &lt;context:component-scan &gt;及使用说明](http://blog.csdn.net/u011079274/article/details/51477287)

### c) pom.xml配置及RequestBody和ResponseBody
使用Json对象的话需要用到Jackson对应的依赖：

```xml
<!-- jackson 配置-->
<jackson.version>2.1.1</jackson.version>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
</dependency>
```
并对请求对象，在参数中使用`@RequestBody`以及返回值中使用`@ResponseBody`注解进行标注。

## 2. Controller不生效有谁来背锅
前段时间写代码遇到了配置好Controller怎么都无法生效的问题，经过一系列排查，问题终于定位在`<context:component-scan />`的身上。

Controller路径： `com.baidu.cpu.analyze.controller.XXXController`
配置：`<context:component-scan base-package="com.baidu.cpu.*.*"/>`

原因： 单个`*`代表着一层路径，因此在我的项目中最多扫描到了`com.baidu.cpu.analyze.XXXController`而无法到下一层真正的`Controller`中;

其实这个和base-package中的通配符有关，有如下规则：

a) 全路径：`<context:component-scan base-package="com.baidu.cpu.analyze.controller"/>`,该方式可以扫描controller包下的所有的Controller；

b) 单`*`通配符，有两种规则：

- `<context:component-scan base-package="com.baidu.cpu.*"/>`能够扫描cpu单层子包中的类，而无法匹配到cpu包下的类；  
- `<context:component-scan base-package="com.baidu.cpu.*Controller"/>`可以匹配cpu包下一Controller结尾的类；
	
c) 双`*`通配符，可以匹配0~n层包或类，但是会有些影响效率；

> 个人比较倾向于在服务化的小规模的工程中用`**`进行配置.

## 3.集成maven时无法找到spring.xsd文件；
使用jar包来运行Spring配置的时候，怎么都无法运行，报错找不到spring.xsd文件;
其实该文件是在`spring.schemas`制定，并且在`spring.handlers`中进行处理，因此需要打包的时候将这两个文件打包到classpath下。

此处借用maven-shade打包，配置为：

```xml
<!--  打可运行jar包  -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.4.3</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
     	<!-- 防止生成dependency-reduced-pom.xml文件 -->
        <createDependencyReducedPom>false</createDependencyReducedPom>

        <transformers>
            <transformer
                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.baidu.cpu.analyze.benchmark.JMHAnalyzeQuery</mainClass>
            </transformer>

            <transformer
                    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.handlers</resource>
            </transformer>
            <transformer
                    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.schemas</resource>
            </transformer>
        </transformers>

    </configuration>
</plugin>
            
```




