# MyBatis-Plus连接GaussDB使用指南

MyBatis-Plus 是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

官方开发指南参考：[https://baomidou.com/introduce/](https://baomidou.com/introduce/)

本文主要提供MyBatis-Plus连接GaussDB相关的使用指南。

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

### MyBatis-Plus Spring Boot Starter

应用程序如果使用Spring Boot，推荐使用MyBatis-Plus Spring Boot Starter, 可以简化MyBatis-Plus的配置。

  * maven

    ```xml
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

MyBatis-Plus Spring Boot Starter已经包含了MyBatis 3.5.19版本。

## 连接GaussDB使用指南

我们的开发示例使用MyBatis和GaussDB实现用户认证和鉴权功能, 参考`动手试试`章节运行本示例。

* 引入依赖

  开始之前，需要按照`相关技术组件介绍`引入相关软件包。

* 配置数据源

  MyBatis-Plus Spring Boot Starter使用Spring DataSource配置数据源，在application.yml中，增加如下配置：

  ```yml
  spring:
    datasource:
      url: ${DB_URL:jdbc:gaussdb://127.0.0.1:8000/postgres?currentSchema=authentication_server_db}
      username: ${DB_USERNAME:GaussdbExamples}
      password: ${DB_PASSWORD:Gaussdb-Examples-123}
      driver-class-name: com.huawei.gaussdb.jdbc.Driver
  ```

* 配置Mapper

  MyBatis通过Mapper定义SQL与Java Object之间的映射。以TokenMapper为例：

  ```java
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

* 使用Mapper

可以像使用普通Bean一样，使用Mapper。

  ```java
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

  ## 动手试试

[gaussdb-examples](https://github.com/HuaweiCloudDeveloper/gaussdb-examples) 是基于 ServiceComb Fence提供的示例. 

* 下载代码

  ```shell
  git clone https://github.com/HuaweiCloudDeveloper/gaussdb-examples.git
  git checkout -B MyBatis-Plus/3.5.x origin/MyBatis-Plus/3.5.x
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
