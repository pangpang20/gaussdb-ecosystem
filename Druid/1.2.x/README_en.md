# Guide Connecting GaussDB With Druid

Druid is a JDBC component library, including the database connection pool and SQL Parser.

For details about the official development guide, see the following: [https://github.com/alibaba/druid/wiki/](https://github.com/alibaba/druid/wiki/)

Developers usually use the Druid through Spring. For details, see the official document. [https://docs.spring.io/spring-boot/3.3/reference/data/sql.html\#data.sql.datasource.connection-pool](https://docs.spring.io/spring-boot/3.3/reference/data/sql.html#data.sql.datasource.connection-pool)	.

This article focuses on how to use Druid in Spring Data JPA.

> Note: The WallFilter and StatFilter functions depend on or partially on the AST component, which is equivalent to a lightweight SQL parsing engine on the client. Currently, this engine does not support GaussDB. If developers use Druid to access GaussDB, you are advised not to enable the two functions to prevent unknown errors. Alternatively, use the HikariCP connection pool carried by Spring Data JPA by default.

## Introduction to Related Technical Components

### driver

You are advised to use the official driver to connect to GaussDB.

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

### Spring Data JPA

Spring Data JPA can be easily integrated with Spring Boot by introducing dependencies. By default, Spring Data JPA uses HikariCP as the connection pool. Therefore, you need to exclude the dependency.

 *  maven
    
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <exclusions>
          <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
          <exclusion>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
          </exclusion>
        </exclusions>
        <version>3.3.12</version>
      </dependency>
    ```

### Hibernate

Spring Data JPA depends on Hibernate by default. In this example, Hibernate 6.6.15.Final is used.

 *  maven
    
    ```
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.6.15.Final</version>
        <scope>compile</scope>
      </dependency>
    ```

In addition, the implementation depends on the Hibernate Dialect provided by GaussDB.

```xml
  <dependency>
    <groupId>com.huaweicloud.gaussdb</groupId>
    <artifactId>gaussdb-hibernate-dialect</artifactId>
    <version>6.0.0</version>
    <scope>compile</scope>
  </dependency>
```

### Druid

 *  maven
    
    ```
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-3-starter</artifactId>
        <version>1.2.23</version>
      </dependency>
    ```

## GaussDB Connection Guide

In our development example, Spring Data JPA and GaussDB are used to implement user authentication and authorization. For details, see `Try It Out` section to run this example.

 *  Importing Dependency
    
    Before you start, you need to follow the `Introduction to Related Technical Components` import related software packages.

 *  Configuring Data Sources
    
    Spring Data JPA uses Spring DataSource to configure data sources. Add the following configuration to the application.yml file:
    
    ```
    spring:
          datasource:
              url: ${DB_URL:jdbc:gaussdb://127.0.0.1:8000/postgres?currentSchema=authentication_server_db}
              username: ${DB_USERNAME:GaussdbExamples}
              password: ${DB_PASSWORD:Gaussdb-Examples-123}
              driver-class-name: com.huawei.gaussdb.jdbc.Driver
          jpa:
              hibernate:
                  ddl-auto: none
    ```
    
    In this example, a script is used to manually create a table. Therefore, hibernate ddl-auto is set to none.

 *  Configuring Spring Data JPA
    
    ```
    @Configuration
     @EnableJpaRepositories
     @EnableTransactionManagement
     public class SpringDataJPAConfig {
    
     }
    ```

 *  Configuring the Repository
    
    Spring Data JPA and the core concept is Repository, which is the aggregate root in DDD. UserRepository is used as an example.
    
    ```
    public interface UserRepository extends JpaRepository<User, String> {
          @Query("select u from User u where userName = ?1")
          User selectUserByUsername(String userName);
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
      User info = userRepository.selectUserByUsername(username);
      if (info == null) {
      throw new UsernameNotFoundException("");
      }

      return new JDBCUserDetails(info, roleRepository.selectRolesByUsername(username));
  }

  ...

  }
```

> Precautions: GaussDB Dialect uses the following rules when generating database identifiers (names, column names, and so on): Use lowercase letters and use double quotation marks. Therefore, you need to ensure that the preceding constraints can work properly when creating a table by using the customized query. For more information, please refer to: [GaussDB Questions About Identifiers and Quotation Marks](https://bbs.huaweicloud.com/forum/thread-0254182512348607062-1-1.html)	.

## Try it. ##

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples)	is based on the example provided by the ServiceComb Fence.

 *  Download code
    
    ```
    git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
    git checkout -B Druid/1.2.x origin/Druid/1.2.x
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
    
    Reference[Script for creating tables.](https://github.com/HuaweiCloudDeveloper/gaussdb-examples/tree/Druid/1.2.x/authentication-server/src/main/resources/sql/user.sql)	, you can use database tools such as DBeaver to connect to and run related SQL statements.
 *  Start the operation.
    
    Reference [Java Chassis 3 Best Practice (1): Introduction to the Fence Project](https://bbs.huaweicloud.com/blogs/433423)	Or the [Open Source for Huawei Wiki](https://gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki)	the running example is described in.

