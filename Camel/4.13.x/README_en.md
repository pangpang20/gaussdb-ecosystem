# Guide to Connecting Camel to GaussDB #

Apache Camel is an excellent integration framework.

Official development guide Reference: [https://camel.apache.org/](https://camel.apache.org/)

This document describes how to use the JDBC component of Camel to connect to GaussDB.

## Introduction to Related Technical Components ##

### driver ###

Connect to GaussDB. You are advised to use the official driver.

 *  Connection string:`jdbc:gaussdb://host:port/database`
 *  Driver name:`com.huawei.gaussdb.jdbc.Driver`
 *  maven
    
    ```
    <dependency>
        <groupId>com.huaweicloud.gaussdb</groupId>
        <artifactId>gaussdbjdbc</artifactId>
        <version>506.0.0.b058</version>
    </dependency>
    ```

### Camel Spring Boot Starter ###

If the application uses Spring Boot, the Camel Spring Boot Starter is recommended to simplify the configuration of Camel.

 *  maven
    
    ```
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

## GaussDB Connection Guide ##

This example demonstrates how the CAMEL writes data into the database by running a simple insert statement. For details, see `Try it first` section run this example.

 *  Importing Dependency
    
    Before you start, you need to follow the `Introduction to Related Technical Components` introduce the relevant app package.

 *  Configuring Data Sources
    
    Use Spring DataSource to configure the data source. Add the following configuration to the application.yml file:
    
    ```
    spring:
      datasource:
        url: ${DB_URL:jdbc:gaussdb://127.0.0.1:8000/postgres?currentSchema=authentication_server_db}
        username: ${DB_USERNAME:GaussdbExamples}
        password: ${DB_PASSWORD:Gaussdb-Examples-123}
        driver-class-name: com.huawei.gaussdb.jdbc.Driver
    ```

 *  Defining a Route
    
    ```
    @Component
      public class GaussDBRouteBuilder extends RouteBuilder {
          @Override
          public void configure() throws Exception {
              from("direct:insertData")
                  .setBody(constant("INSERT INTO TBL_TEST VALUES (default,10001,'Beijing','Shanghai',false)"))
                  .to("jdbc:dataSource"); //spring default data source name
          }
      }
    ```
    
    Particularly important to note, here's`dataSource`is the name of the data source injected by Spring JDBC by default.

 *  Define Produce and trigger execution.
    
    ```
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
    
    ## Try it first. ##

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples)	is an example based on the ServiceComb Fence.

 *  Download code
    
    ```
    git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
    git checkout -B Camel/4.13.x origin/Camel/4.13.x
    ```

install Zookeeper and GaussDB are required before the run example. Individual developers can use the open source image to install ZooKeeper and OpenGauss.

 *  docker install OpenGauss
    
    ```
    docker run --name opengauss --privileged=true -d -e GS_USERNAME=GaussdbExamples -e GS_PASSWORD=Gaussdb-Examples-123 -e GS_PORT=8000 -p 8000:8000 opengauss/opengauss:7.0.0-RC1.B023
    ```
 *  docker install Zookeeper
    
    ```
    docker run --name zookeeper --restart always -d -p 2181:2181 zookeeper:3.9.3
    ```
 *  Creating a table
    
    Reference[Script for creating tables.](https://github.com/HuaweiCloudDeveloper/gaussdb-examples/tree/MyBatis/3.5.x/authentication-server/src/main/resources/sql/user.sql)	, you can use database tools such as DBeaver to connect to and run related SQL statement.

 *  Starting the run
    
    Reference[Java Chassis 3 Best Practice (1): Introduction to the Fence Project](https://bbs.huaweicloud.com/blogs/433423)	Or the[Open Source for Huawei Wiki](https://gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki) describes the run example in.
    
    After logging in to the, go to `System Maintainence`\- > `Interface Test`. Invoke the following interface: `GET /api/authentication/camel`. You can see that the Camel inserts a statement into the database.

