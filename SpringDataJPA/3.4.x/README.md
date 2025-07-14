# Spring Data JPA连接GaussDB使用指南

Spring Data JPA是Spring技术体系的重要组成部分。它将核心的 Spring 概念应用于使用符合领域驱动设计原则的 JDBC 数据库驱动程序解决方案的开发中。它提供了一个 “模板”，作为用于存储和查询聚合的高级抽象。

官方开发指南参考：[https://docs.spring.io/spring-data/jpa/reference/3.4/index.html](https://docs.spring.io/spring-data/jpa/reference/3.4/index.html)

本文主要提供Spring Data JPA连接GaussDB相关的使用指南。

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

### Spring Data JPA

Spring Data JPA可以方便的与Spring Boot集成，只需要引入依赖。

  * maven

    ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <exclusions>
          <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
        </exclusions>
        <version>3.4.5</version>
      </dependency>
    ```

### Hibernate

Spring Data JPA默认依赖Hibernate，本例子使用了Hibernate 7.0.0.Final版本：

  * maven

    ```xml
      <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>7.0.0.Final</version>
        <scope>compile</scope>
      </dependency>
    ```

同时还需要依赖GaussDB提供的Hibernate Dialect实现：

    ```xml
      <dependency>
        <groupId>com.huaweicloud.gaussdb</groupId>
        <artifactId>gaussdb-hibernate-dialect</artifactId>
        <version>7.0.1</version>
        <scope>compile</scope>
      </dependency>
    ```

## 连接GaussDB使用指南

我们的开发示例使用Spring Data JPA和GaussDB实现用户认证和鉴权功能, 参考`动手试试`章节运行本示例。

* 引入依赖

  开始之前，需要按照`相关技术组件介绍`引入相关软件包。

* 配置数据源

  Spring Data JPA使用Spring DataSource配置数据源，在application.yml中，增加如下配置：

  ```yml
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
  
  本例子使用脚本手工建表，因此设置hibernate ddl-auto为none

* 配置Spring Data JPA

   ```java
    @Configuration
    @EnableJpaRepositories
    @EnableTransactionManagement
    public class SpringDataJPAConfig {

    }
   ```


* 配置Repository

  Spring Data JPA和核心概念是Repository，它是DDD里面的聚合根（aggregate root）。 以UserRepository为例：

  ```java
    public interface UserRepository extends JpaRepository<User, String> {
        @Query("select u from User u where userName = ?1")
        User selectUserByUsername(String userName);
    }
  ```

* 使用Repository

可以像使用普通Bean一样，使用Repository。

  ```java
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

* 注意事项

  GaussDB Dialect在生成数据库标识符（表明、列名等）的时候采用如下规则：使用小写，并且使用双引号。因此在使用自定义Query，建表的时候，需要保证上述约束能够正常工作。更多信息可以参考：[GaussDB关于标识符加引号的疑问](https://bbs.huaweicloud.com/forum/thread-0254182512348607062-1-1.html) 。

## 动手试试

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples) 是基于 ServiceComb Fence提供的示例. 

* 下载代码

  ```shell
  git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
  git checkout -B SpringDataJPA/3.4.x origin/SpringDataJPA/3.4.x
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

  参考[建表脚本](https://github.com/HuaweiCloudDeveloper/gaussdb-examples/tree/SpringDataJPA/3.4.x/authentication-server/src/main/resources/sql/user.sql)，可以使用DBeaver等数据库工具连接，并执行相关SQL语句。

* 启动运行

  参考[Java Chassis 3最佳实践（一）：Fence项目介绍](https://bbs.huaweicloud.com/blogs/433423) 或者[Open Source for Huawei Wiki](https://gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki)的介绍运行示例。
