# Guide to Connecting MyBatis-Plus to GaussDB

MyBatis-Plus is an enhancement tool of MyBatis. It is developed to simplify development and improve efficiency.

For details about the official development guide, see the following [https://baomidou.com/en/introduce/](https://baomidou.com/en/introduce/). 

This document describes how to connect MyBatis-Plus to GaussDB.

## Introduction to Related Technical Components

### JDBC driver ###

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

### MyBatis-Plus Spring Boot Starter ###

If the application uses Spring Boot, MyBatis-Plus Spring Boot Starter is recommended to simplify the configuration of MyBatis-Plus.

 *  maven
    
    ```
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
        <exclusions>
          <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
        </exclusions>
        <version>3.5.13</version>
      </dependency
    ```

MyBatis-Plus Spring Boot Starter already includes MyBatis 3.5.19.

## GaussDB Connection Guide

Our development example uses MyBatis and GaussDB to implement user authentication and authorization. For details, see.`Try it first`section to run this example.

 *  Importing Dependency
    
    Before you start, you need to follow the`Introduction to Related Technical Components` import related software packages.

 *  Configuring Data Sources
    
    MyBatis-Plus Spring Boot Starter uses Spring DataSource to configure data sources. Add the following configuration to the application.yml file:
    
    ```
    spring:
      datasource:
        url: ${DB_URL:jdbc:gaussdb://127.0.0.1:8000/postgres?currentSchema=authentication_server_db}
        username: ${DB_USERNAME:GaussdbExamples}
        password: ${DB_PASSWORD:Gaussdb-Examples-123}
        driver-class-name: com.huawei.gaussdb.jdbc.Driver
    ```

 *  Configuring the Mapper
    
    MyBatis uses Mapper to define the mapping between SQL and Java objects. TokenMapper is used as an example:
    
    ```
    @Mapper
      public interface TokenMapper {
          @Insert("""
              insert into
              T_TOKENS(ACCESS_TOKEN_VALUE,REFRESH_TOKEN_VALUE,TOKEN)
              values(#{accessTokenId},#{refreshTokenId},#{tokenInfo})""")
          void insertNewToken(@Param("accessTokenId") String accessTokenId,
              @Param("refreshTokenId") String refreshTokenId,
              @Param("tokenInfo") String tokenInfo);
    
          @Select("""
              select TOKEN
              from T_TOKENS where ACCESS_TOKEN_VALUE =
              #{accessTokenId}""")
          String getTokenInfoByAccessTokenId(@Param("accessTokenId") String accessTokenId);
    
          @Select("""
              select TOKEN
              from T_TOKENS where REFRESH_TOKEN_VALUE =
              #{refreshTokenId}""")
          String getTokenInfoByRefreshTokenId(@Param("refreshTokenId") String refreshTokenId);
          }
    ```
    
 *  Using Mapper

You can use Mapper just like you would with a regular bean.

```
@Component(CommonConstants.BEAN_AUTH_OPEN_ID_TOKEN_STORE)
  public class JDBCOpenIDTokenStore extends AbstractOpenIDTokenStore {
      @Autowired
      private TokenMapper tokenMapper;

      @Override
      public CompletableFuture<OpenIDToken> readTokenByAccessToken(String value) {
          CompletableFuture<OpenIDToken> result = new CompletableFuture<>();

          String tokenInfo = tokenMapper.getTokenInfoByAccessTokenId(value);
          if (tokenInfo != null) {
          result.complete(JsonParser.parse(tokenInfo, OpenIDToken.class));
          }
          result.complete(null);
          return result;
      }

      @Override
      public OpenIDToken readTokenByRefreshToken(String refreshTokenValue) {
          String tokenInfo = tokenMapper.getTokenInfoByRefreshTokenId(refreshTokenValue);
          if (tokenInfo != null) {
          return JsonParser.parse(tokenInfo, OpenIDToken.class);
          }
          return null;
      }

      @Override
      public void saveToken(OpenIDToken token) {
          tokenMapper.insertNewToken(token.getValue(),
              token.getRefreshToken().getValue(),
              JsonParser.unparse(token));
      }

  }
```

## Try it first. ##

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples)	is based on the example provided by ServiceComb Fence.

 *  Download Code
    
    ```
    git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
    git checkout -B MyBatis-Plus/3.5.x origin/MyBatis-Plus/3.5.x
    ```

Before running the sample, you need to install the ZooKeeper and GaussDB. Individual developers can install ZooKeeper and OpenGauss using open-source images.

 *  Installing OpenGauss Using Docker
    
    ```
    docker run --name opengauss --privileged=true -d -e GS_USERNAME=GaussdbExamples -e GS_PASSWORD=Gaussdb-Examples-123 -e GS_PORT=8000 -p 8000:8000 opengauss/opengauss:7.0.0-RC1.B023
    ```
 *  Installing ZooKeeper on Docker
    
    ```
    docker run --name zookeeper --restart always -d -p 2181:2181 zookeeper:3.9.3
    ```
 *  Creating a table
    
    Reference [Script for creating tables](https://github.com/HuaweiCloudDeveloper/gaussdb-examples/tree/MyBatis/3.5.x/authentication-server/src/main/resources/sql/user.sql)	, you can use database tools such as DBeaver to connect to and run related SQL statements.
 
 *  Start the application
    
    Reference [Java Chassis 3 Best Practice : Introduction to the Fence Project](https://bbs.huaweicloud.com/blogs/433423)	Or the [Open Source for Huawei Wiki](https://gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki)	where the running example is described in.

