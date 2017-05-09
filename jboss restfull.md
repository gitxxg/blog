# jboss resteasy搭建简单的restful web services项目

项目上要用到webservice，鉴于现在restful webservice比较流行，用jboss resteasy搭建一个简单restful webservice。mark！

### 1、首先是用idea创建一个maven webapp工程，这个很简单，不用多说。

### 2、配置web.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         id="WebApp_ID" version="2.5">
    <display-name>httpserver</display-name>

    <!-- Auto scan rest service -->
    <context-param>
        <param-name>resteasy.resources</param-name>
        <param-value>cn.ghl.httpserver.MyHttpServer</param-value>
    </context-param>

    <context-param>
        <param-name>resteasy.servlet.mapping.prefix</param-name>
        <param-value>/rest</param-value>
    </context-param>

    <listener>
        <listener-class>
            org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap
        </listener-class>
    </listener>

    <servlet>
        <servlet-name>resteasy-servlet</servlet-name>
        <servlet-class>
            org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
        </servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>resteasy-servlet</servlet-name>
        <url-pattern>/rest/*</url-pattern>
    </servlet-mapping>
</web-app>
```
其中<param-value>cn.ghl.httpserver.MyHttpServer</param-value>为你创建的类。

### 3、配置pom.xml文件，引入相关jar包。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.ghl</groupId>
    <artifactId>httpserver</artifactId>
    <packaging>war</packaging>
    <name>httpserver Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jaxrs</artifactId>
        </dependency>
    </dependencies>

    <build>
        <finalName>httpserver</finalName>
    </build>
</project>

```

### 4、代码实现

```
package cn.ghl.httpserver;

import javax.ws.rs.*;

@Path("/")
@Produces({ "application/json" })
@Consumes({ "application/json" })
public class MyHttpServer {

    @GET
    @Path("/{id}")
    public String get(@PathParam("id") String id) {
        String str = "GET [" + id + "]";
        System.out.println(str);
        return str;
    }

    @PUT
    @Path("/{id}")
    public String put(@PathParam("id") String id) {
        String str = "PUT [" + id + "]";
        System.out.println(str);
        return str;
    }

    @POST
    @Path("/{id}")
    public String post(@PathParam("id") String id) {
        String str = "POST [" + id + "]";
        System.out.println(str);
        return str;
    }

    @DELETE
    @Path("/{id}")
    public String delete(@PathParam("id") String id) {
        String str = "DELETE [" + id + "]";
        System.out.println(str);
        return str;
    }

}

```

### 4、OK搞定，用tomcat或者jetty运行项目
测试示例：http://localhost:8080/httpserver/rest/id1223
