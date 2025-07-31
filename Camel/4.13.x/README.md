# Camel连接GaussDB使用指南

Apache Camel 是一款优秀的系统集成开发框架。

官方开发指南参考：[https://camel.apache.org/](https://camel.apache.org/)

本文提供使用Camel的JDBC组件连接GaussDB使用指南。

## 相关技术组件介绍

### JDBC驱动

连接GaussDB，推荐使用官方驱动。 

  * 连接串：`jdbc:gaussdb://host:port/database`

  * Driver名称：`com.huawei.gaussdb.jdbc.Driver`

  * maven

    ```xml
    <dependency>
        <groupId>com.huaweicloud.gaussdb</groupId>
        <artifactId>gaussdbjdbc</artifactId>
        <version>506.0.0.b058</version>
    </dependency>
    ```

### Camel Spring Boot Starter

应用程序如果使用Spring Boot，推荐使用Camel Spring Boot Starter, 可以简化Camel的配置。

  * maven

    ```xml
    <dependency>
      <groupId>org.apache.camel.springboot</groupId>
      <artifactId>camel-spring-boot-starter</artifactId>
      <version>4.13.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.camel.springboot</groupId>
      <artifactId>camel-spring-jdbc-starter</artifactId>
      <version>4.13.0</version>
    </dependency>
    ```


## 连接GaussDB使用指南

本示例通过执行简单的插入语句，演示Camel写入数据库的操作。参考`动手试试`章节运行本示例。

* 引入依赖

  开始之前，需要按照`相关技术组件介绍`引入相关软件包。

* 配置数据源

  使用Spring DataSource配置数据源，在application.yml中，增加如下配置：

  ```yml
  spring:
    datasource:
      url: ${DB_URL:jdbc:gaussdb://127.0.0.1:8000/postgres?currentSchema=authentication_server_db}
      username: ${DB_USERNAME:GaussdbExamples}
      password: ${DB_PASSWORD:Gaussdb-Examples-123}
      driver-class-name: com.huawei.gaussdb.jdbc.Driver
  ```

* 定义Route

  ```java
    @Component
    public class GaussDBRouteBuilder extends RouteBuilder {
        @Override
        public void configure() throws Exception {
            from("direct:insertData")
                .setBody(constant("INSERT INTO TBL_TEST VALUES (default,10001,'Beijing','Shanghai',false)"))
                .to("jdbc:dataSource"); // spring default data source name
        }
    }
  ```

  特别需要注意，这里的 `dataSource` 是Spring JDBC默认注入的数据源名称。 

* 定义Produce，触发执行

  ```java
    @RestController
    public class RestService {
        @Autowired
        private ProducerTemplate producerTemplate;

        @GetMapping("/camel")
        public void camel() {
            producerTemplate.sendBody("direct:insertData", null);
        }
    }
  ```


  ## 动手试试

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples) 是基于 ServiceComb Fence提供的示例. 

* 下载代码

  ```shell
  git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
  git checkout -B Camel/4.13.x origin/Camel/4.13.x
  ```

运行示例前，需要先安装Zookeeper和GaussDB。 个人开发者可以通过开源镜像安装Zookeeper和OpenGauss。

* docker安装OpenGauss

  ```shell
  docker run --name opengauss --privileged=true -d -e GS_USERNAME=GaussdbExamples -e GS_PASSWORD=Gaussdb-Examples-123 -e GS_PORT=8000 -p 8000:8000 opengauss/opengauss:7.0.0-RC1.B023
  ```

  
* docker安装Zookeeper

  ```shell
  docker run --name zookeeper --restart always -d -p 2181:2181 zookeeper:3.9.3
  ```

* 建表

  参考[建表脚本](https://github.com/HuaweiCloudDeveloper/gaussdb-examples/tree/MyBatis/3.5.x/authentication-server/src/main/resources/sql/user.sql)，可以使用DBeaver等数据库工具连接，并执行相关SQL语句。

* 启动运行

  参考[Java Chassis 3最佳实践（一）：Fence项目介绍](https://bbs.huaweicloud.com/blogs/433423) 或者[Open Source for Huawei Wiki](https://gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki)的介绍运行示例。

  登录后，进入`系统运维` -> `接口测试`。 调用接口： `GET /api/authentication/camel` ， 就可以看到Camel往数据库插入了一条语句。 


