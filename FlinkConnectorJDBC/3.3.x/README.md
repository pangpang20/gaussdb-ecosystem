‌[Apache Flink ‌](https://flink.apache.org/) 是一个开源的分布式流处理框架，专为大规模数据处理设计，支持流处理和批处理一体化。它能够在无界和有界数据流上进行有状态计算，具有高吞吐量、低延迟的特性，广泛应用于实时数据处理场景。
本文档主要介绍 [Flink-connector-jdbc-gaussdb](https://github.com/HuaweiCloudDeveloper/gaussdb-flink-connector-jdbc) 的使用。[Flink-connector-jdbc-gaussdb](https://github.com/HuaweiCloudDeveloper/gaussdb-flink-connector-jdbc)是一个开源的GaussDB连接器，专为Flink写入数据(sink)到GaussDB场景下使用。

**核心特性：**
1. 新增数据：对应insert功能, 新增增量数据。
2. 更新数据：对应update功能, 更新存量数据。
3. ‌删除数据：对应delete功能, 删除存量数据。
4. 合并数据：对应upsert功能, 根据主键判断目标数据库(GaussDB)是否已存在增量数据，不存在则新增(insert)，存在则更新(update)。

## 前置条件
本项目提供的连接器使用前需预先安装 Flink集群及其相关运行环境，并下载对应jar包放置Flink集群各节点的Flink 安装目录 ~/lib 下。
需要下载的jar包 <strong>GaussDB驱动</strong> + <strong>flink-connector-jdbc-gaussdb</strong> + <strong>flink-connector-jdbc-core</strong>
相关依赖jar包下载地址(3.3.0-1.20版本):
[**GaussDB驱动**](https://repo1.maven.org/maven2/com/huaweicloud/gaussdb/gaussdbjdbc/506.0.0.b058-jdk7/gaussdbjdbc-506.0.0.b058-jdk7.jar)
[**flink-connector-jdbc-gaussdb**](https://repo.maven.apache.org/maven2/com/huaweicloud/gaussdb/flink/flink-connector-jdbc-gaussdb/3.3.0-1.20/)
[**flink-connector-jdbc-core**](https://repo1.maven.org/maven2/org/apache/flink/flink-connector-jdbc-core/3.3.0-1.20/)


> **系统要求如下：**
> - CPU: 2GHz 或更高
> - RAM: 4GB 或更大
> - Disk: 至少 40GB

## 使用说明

| jar包版本                                                                                                                                                 | Flink集群系统版本(推荐)                                | 备注                   |
|--------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------|----------------------|
| [flink-connector-jdbc-gaussdb-3.3.0-1.20](https://repo.maven.apache.org/maven2/com/huaweicloud/gaussdb/flink/flink-connector-jdbc-gaussdb/3.3.0-1.20/) | 基于 鲲鹏服务器 + Huawei Cloud EulerOS 2.0 标准版 64位 安装部署 | Flink1.20及以下集群环境适用     |
| [flink-connector-jdbc-core-3.3.0-1.20](https://repo.maven.apache.org/maven2/com/huaweicloud/gaussdb/flink/flink-connector-jdbc-gaussdb/3.3.0-1.20/)    | 基于 鲲鹏服务器 + Huawei Cloud EulerOS 2.0 标准版 64位 安装部署 | Flink1.20及以下集群环境适用     |
| [GaussDB驱动1](https://repo1.maven.org/maven2/com/huaweicloud/gaussdb/gaussdbjdbc/506.0.0.b058-jdk7/gaussdbjdbc-506.0.0.b058-jdk7.jar)                   | 基于 鲲鹏服务器 + Huawei Cloud EulerOS 2.0 标准版 64位 安装部署 |Flink集群环境JDK 17以下使用  如 JDK 8、11 |	
| [GaussDB驱动2](https://repo1.maven.org/maven2/com/huaweicloud/gaussdb/gaussdbjdbc/506.0.0.b058/gaussdbjdbc-506.0.0.b058.jar)                             | 基于 鲲鹏服务器 + Huawei Cloud EulerOS 2.0 标准版 64位 安装部署 |Flink集群环境JDK 17及以上使用    |

## 使用案例
1. gaussdb 数据库环境准备  
   gaussdb 数据库实例购买 以及库表创建。(以下为测试代码示例)  <br />
   在gaussdb 数据库创建db test_db,创建schema player, 创建表 players <br />
``` gaussdb sql 
create database test_db;
use test_db;
create schema player;

CREATE TABLE player.players (
player_id INT NOT NULL,
team_id INT,
player_name VARCHAR(255),
height VARCHAR(255),
update_time timestamp,
PRIMARY KEY (player_id)
);
```

2. flink sql 使用场景
*  做源表   <br />
   场景介绍: Flink sql 读取gaussdb players数据后将数据写入printsink表  <br />

```flink sql
CREATE TABLE source (
player_id INT,
team_id INT,
player_name VARCHAR,
height VARCHAR,
update_time timestamp,
PRIMARY KEY (player_id) NOT ENFORCED
) WITH (
'connector' = 'jdbc',
'url' = 'jdbc:gaussdb://1.1.1.1:8000/test_db?currentSchema=player',
'username' = 'user',
'password' = '123456',
'table-name' = 'players');

CREATE TABLE printsink (
player_id INT,
team_id INT,
player_name VARCHAR,
height VARCHAR ,
update_time timestamp
) WITH (
'connector' = 'print'
);

insert into printsink select * from source;
```
**执行成功：**  <br />
![img.png](docs/image/scenario01.png)


*  做结果表  <br />
   场景介绍: Flink sql 读取gaussdb players_source表的数据后将数据写入gaussdb players表  <br />

```flink sql
CREATE TABLE source (
player_id INT,
team_id INT,
player_name VARCHAR,
height VARCHAR,
update_time timestamp,
PRIMARY KEY (player_id) NOT ENFORCED
) WITH (
'connector' = 'jdbc',
'url' = 'jdbc:gaussdb://1.1.1.1:8000/test_db?currentSchema=player',
'username' = 'user',
'password' = '123456',
'table-name' = 'players_source');

CREATE TABLE sink (
player_id INT,
team_id INT,
player_name VARCHAR,
height VARCHAR,
update_time timestamp,
PRIMARY KEY (player_id) NOT ENFORCED
) WITH (
'connector' = 'jdbc',
'url' = 'jdbc:gaussdb://1.1.1.1:8000/test_db?currentSchema=player',
'username' = 'user',
'password' = '123456',
'table-name' = 'players');

insert into sink  select * from source;
```
**执行成功：** <br />
![img.png](docs/image/scenario02.png)


*  做维表  <br />
   场景介绍:  Flink sql 读取gaussdb players_info表数据并join gaussdb维表 players后 将数据写入players_detail 结果表 <br />

```flink sql
CREATE TABLE players (
player_id INT,
team_id INT,
player_name VARCHAR,
height VARCHAR,
update_time timestamp,
PRIMARY KEY (player_id) NOT ENFORCED
) WITH (
'connector' = 'jdbc',
'url' = 'jdbc:gaussdb://1.1.1.1:8000/test_db?currentSchema=player',
'username' = 'user',
'password' = '123456',
'table-name' = 'players');

CREATE TABLE players_info (
player_id INT,
player_name VARCHAR,
phone VARCHAR ,
address VARCHAR ,
email VARCHAR ,
update_time timestamp,
PRIMARY KEY (player_id) NOT ENFORCED
) WITH (
'connector' = 'jdbc',
'url' = 'jdbc:gaussdb://1.1.1.1:8000/test_db?currentSchema=player',
'username' = 'user',
'password' = '123456',
'table-name' = 'players_info');

CREATE TABLE players_detail (
player_id INT,
player_name VARCHAR,
phone VARCHAR ,
address VARCHAR ,
email VARCHAR ,
update_time timestamp,
PRIMARY KEY (player_id) NOT ENFORCED
) WITH (
'connector' = 'print'
);

insert into players_detail   
select ps.player_id, ps.player_name, pi.phone ,pi.address ,pi.email ,pi.update_time from players_info  as pi
inner join  players  as ps  on  ps.player_id = pi.player_id ;
```
**执行成功：** <br />
![img.png](docs/image/scenario03.png)


**注意事项:**
1. GaussDB驱动的选择: 依照Flink集群环境的JDK版本选择对应的驱动版本，否则可能出现低版本的JDK无法识别高版本的情况,导致任务提交失败。
2. 建议直接下载jar包后放置Flink安装目录 ~/lib 下使用,如要使用源码在本地重新打包后使用，注意对应连接器和JDK版本。
3. 连接器的选择: flink-connector-jdbc-gaussdb和flink-connector-jdbc-core需要与Flink的版本对应匹配 不支持混合版本使用。
4. 源码仓库地址: [**flink-connector-jdbc-gaussdb**](https://github.com/HuaweiCloudDeveloper/gaussdb-flink-connector-jdbc) , [**flink-connector-jdbc-core**](https://github.com/apache/flink-connector-jdbc)
5. 本文档的内容介绍是对 Flink [Table API Connectors](https://nightlies.apache.org/flink/flink-docs-release-1.20/zh/docs/connectors/table/jdbc/) 方式的实现, Flink [DataStream Connectors](https://nightlies.apache.org/flink/flink-docs-release-1.20/zh/docs/connectors/datastream/jdbc/) 方式请参考官网链接。