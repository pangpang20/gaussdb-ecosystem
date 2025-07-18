# Spring Data R2DBC Connection Guide to GaussDB

Spring Data R2DBC is an important part of Spring technical architecture. It applies the core Spring concepts to the development of JDBC database driver solutions using domain-driven design principles. It provides a "template" as a high-level abstraction for storing and querying aggregations. R2DBC is an asynchronous implementation of JDBC.

For details about the official development guide, see the following: [https://docs.spring.io/spring-data/relational/reference/r2dbc.html](https://docs.spring.io/spring-data/relational/reference/r2dbc.html)

This document describes how to connect Spring Data R2DBC to GaussDB.

## Introduction to Related Technical Components

### R2DBC Driver

If Spring Data R2DBC is used, the R2DBC driver is required.

 *  Connection string:`r2dbc:gaussdb://host:port/database`
 *  maven
    
    ```
    <dependency>
        <groupId>com.huaweicloud.gaussdb</groupId>
        <artifactId>gaussdb-r2dbc</artifactId>
        <version>1.0.0.RC1</version>
      </dependency>
    ```

### Spring Data R2DBC

Spring Data R2DBC can be easily integrated with Spring Boot, and only the dependency needs to be introduced.

 *  maven
    
    ```xml
      <dependency>
        <groupId>org.springframework.boot.starter</groupId>
        <artifactId>spring-boot-starter-data-r2dbc</artifactId>
        <exclusions>
          <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
        </exclusions>
        <version>3.5.1</version>
      </dependency>
    ```

### Spring Data R2DBC Dialect

Spring Data R2DBC does not contain the GaussDB Dialect. The dependency must be introduced.

 *  maven
    
    ```xml
      <dependency>
        <groupId>com.huaweicloud.gaussdb</groupId>
        <artifactId>gaussdb-spring-data-r2dbc</artifactId>
        <exclusions>
          <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
        </exclusions>
        <version>3.5.0</version>
      </dependency>
    ```

## GaussDB Connection Guide

In the development example, Spring Data R2DBC and GaussDB are used to implement user authentication and authorization. For details, see `Try it First` section to run this example.

 *  Importing Dependency
    
    Before you start, you need to follow the `Introduction to Related Technical Components` import related software packages.

 *  Configuring Data Sources
    
    Spring Data R2DBC uses Spring R2DBC to configure data sources. Add the following configuration to the application.yml file:
    
    ```
    spring:
      r2dbc:
          url: ${DB_URL:r2dbc:gaussdb://127.0.0.1:8000/postgres?currentSchema=authentication_server_db}
          username: ${DB_USERNAME:GaussdbExamples}
          password: ${DB_PASSWORD:Gaussdb-Examples-123}
    ```
    
    Spring R2DBC will be injected.`ConnectionFactory` Such objects are used by Spring Data R2DBC.
 *  Configuring Spring Data R2DBC
    
    ```
    @Configuration
     @EnableR2dbcRepositories
     public class SpringDataR2DBCConfig {
    
     }
    ```
 *  Configuring the Repository
    
    Spring Data R2DBC and the core concept is Repository, which is the aggregate root in DDD. UserRepository is used as an example.
    
    ```
    public interface UserRepository extends ReactiveCrudRepository<User, String> {
      @Query("select * from t_users where user_name = :userName")
      Mono<User> selectUserByUsername(String userName);
      }
    ```
 *  Using the Repository

You can use the Repository as you would a normal bean.

```
@Component(CommonConstants.BEAN_AUTH_USER_DETAILS_SERVICE)
  public class JDBCUserDetailsManager implements UserDetailsManager {

  @Autowired
  private UserRepository userRepository;

  @Autowired
  private RoleRepository roleRepository;

  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
      User info = userRepository.selectUserByUsername(username).block();
      if (info == null) {
      throw new UsernameNotFoundException("");
      }

      return new JDBCUserDetails(info, new HashSet<>(roleRepository.selectRolesByUsername(username)
          .collectList().block()));
  }

  ...

  }
```

> Spring Data R2DBC is generally used in asynchronous processing scenarios. For simplicity, Spring Data R2DBC is directly used in synchronous scenarios. It is not recommended that Spring Data R2DBC be used in real service scenarios.

 *  Configuring Spring R2DBC
    
    Spring Data R2DBC depends on Spring R2DBC. By default, Spring R2DBC does not contain the GaussDB extension. Therefore, you need to manually add the extension to the code.
    
    Provides extended classes:
    
    ```
    public class GaussDBBindMarkersFactoryProvider implements BindMarkersFactoryResolver.BindMarkerFactoryProvider {
          @Override
          public BindMarkersFactory getBindMarkers(ConnectionFactory connectionFactory) {
              ConnectionFactoryMetadata metadata = connectionFactory.getMetadata();
              if("GaussDB".equals(metadata.getName())) {
                  return BindMarkersFactory.indexed("$", 1);
              }
    
              return null;
          }
      }
    ```
    
    Load the extension and configure spring.factories.
    
    ```
    org.springframework.r2dbc.core.binding.BindMarkersFactoryResolver$BindMarkerFactoryProvider=org.apache.servicecomb.fence.authentication.GaussDBBindMarkersFactoryProvider
    ```

> Precautions: GaussDB Dialect uses the following rules when generating database identifiers (notations, column names, and so on): Use lowercase letters and use double quotation marks. Therefore, you need to ensure that the preceding constraints can work properly when creating tables by using the customized query. For more information, please refer to [GaussDB Questions About Identifiers and Quotation Marks](https://bbs.huaweicloud.com/forum/thread-0254182512348607062-1-1.html)	.

## Try it first. ##

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples)	is an example based on the ServiceComb Fence.

 *  Download Code
    
    ```
    git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
    git checkout -B SpringDataR2DBC/3.5.x origin/SpringDataR2DBC/3.5.x
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
    
    Reference[Script for creating tables.](https://github.com/HuaweiCloudDeveloper/gaussdb-examples/tree/SpringDataR2DBC/3.5.x/authentication-server/src/main/resources/sql/user.sql)	, you can use database tools such as DBeaver to connect to and run related SQL statements.
    
 *  Start the operation.
    
    Reference [Java Chassis 3 Best Practice (1): Introduction to the Fence Project](https://bbs.huaweicloud.com/blogs/433423)	Or the [Open Source for Huawei Wiki](https://gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki)	the running example is described in.

