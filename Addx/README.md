
# Addax-GaussDB 插件文档


___


## 1 快速介绍
Addax 是一个支持主流数据库的通用数据采集工具  
GaussDBReader插件实现了从GaussDB读取数据。在底层实现上，GaussDBReader通过JDBC连接远程GaussDB数据库，并执行相应的sql语句将数据从GaussDB库中SELECT出来。  
GaussDBWriter插件实现了写入数据到 GaussDB主库目的表的功能。在底层实现上，GaussDBWriter通过JDBC连接远程 GaussDB 数据库，并执行相应的 insert into ... sql 语句将数据写入 GaussDB，内部会分批次提交入库。 GaussDBWriter面向ETL开发工程师，他们使用GaussDBWriter从数仓导入数据到GaussDB。同时 GaussDBWriter亦可以作为数据迁移工具为DBA等用户提供服务。

## 2 实现原理

简而言之，GaussDBReader通过JDBC连接器连接到远程的GaussDB数据库，并根据用户配置的信息生成查询SELECT SQL语句并发送到远程GaussDB数据库，并将该SQL执行返回结果使用Addax自定义的数据类型拼装为抽象的数据集，并传递给下游Writer处理。
对于用户配置Table、Column、Where的信息，GaussDBReader将其拼接为SQL语句发送到GaussDB数据库；对于用户配置querySql信息，GaussDBReader直接将其发送到GaussDB数据库。  

GaussDBWriter通过 Addax 框架获取 Reader 生成的协议数据，根据你配置生成相应的SQL插入语句  
* `insert into...`(当主键/唯一性索引冲突时会写不进去冲突的行)
<br />  
    注意：  <br />
  		1. 目的表所在数据库必须是主库才能写入数据；整个任务至少需具备 insert into...的权限，是否需要其他权限，取决于你任务配置中在 preSql 和 postSql 中指定的语句。  <br />
      	2. GaussDBWriter和MysqlWriter不同，不支持配置writeMode参数。  


## 3 功能说明

### 3.1 配置样例
* 配置一个自定义SQL的数据库同步任务的作业：

```json
{
    "job": {
        "setting": {
            "speed": 1048576
        },
        "content": [
            {
                "reader": {
                    "name": "gaussdbreader",
                    "parameter": {
                        "username": "xx",
                        "password": "xxxx",
                        "where": "",
                        "connection": [
                            {
                                "querySql": [
                                    "select id,name,comment from schema.table_name where id < 10;"
                                ],
                                "jdbcUrl": [
                                    "jdbc:gaussdb://host:port/database"
                                ]
                            }
                        ]
                    }
                },
				"writer": {
					"name": "gaussdbwriter",
					"parameter": {
						"username": "xx",
						"password": "xxxx",
						"preSql": ["truncate table schema.table_name"],						
						"connection": [
							{
								"jdbcUrl": "jdbc:gaussdb://host:port/database",
								"table": [
									"schema.table_name"
								]
							}
						],
						"column": ["id","name","comment"],
						"postSql": []
					}
				}
            }
        ]
    }
}
```

### 3.2 参数说明

* **jdbcUrl**

	* 描述：描述的是数据库的JDBC连接信息，使用JSON的数组描述，并支持一个库填写多个连接地址。如果配置了多个，GaussDBReader可以依次探测ip的可连接性，直到选择一个合法的IP。如果全部连接失败，GaussDBReader报错。 注意，jdbcUrl必须包含在connection配置单元中。通常情况下 JSON数组填写一个JDBC连接即可。
		jdbcUrl按照GaussDB官方规范，并可以填写连接附件控制信息。具体请参看[GaussDB官方文档](https://docs.opengauss.org/zh/docs/3.1.0/docs/Developerguide/java-sql-Connection.html)。

	* 必选：是 <br />

	* 默认值：无 <br />

* **username**

	* 描述：数据库用户名 <br />

	* 必选：是 <br />

	* 默认值：无 <br />

* **password**

	* 描述：数据库指定用户名的密码 <br />

	* 必选：是 <br />

	* 默认值：无 <br />

* **table**

	* 描述：所选取的需要同步的表。使用JSON的数组描述，因此支持多张表同时抽取。当配置为多张表时，用户自己需保证多张表是同一schema结构，GaussDBReader不予检查表是否同一逻辑表。注意，table必须包含在connection配置单元中。<br />

	* 必选：是 <br />

	* 默认值：无 <br />

* **column**
  * reader 部分  
	* 描述：所配置的表中需要同步的列名集合，使用JSON的数组描述字段信息。用户使用\*代表默认使用所有列配置，例如['\*']。
	  支持列裁剪，即列可以挑选部分列进行导出。    
      支持列换序，即列可以不按照表schema信息进行导出。  
	  支持常量配置，用户需要按照GaussDB语法格式:  
	  ["id", "'hello'::varchar", "true", "2.5::real", "power(2,3)"]  
	  id为普通列名，'hello'::varchar为字符串常量，true为布尔值，2.5为浮点数, power(2,3)为函数。  
      coumn必须用户显示指定同步的列集合，不允许为空！  

  * writer 部分    
    * 描述：目的表需要写入数据的字段,字段之间用英文逗号分隔。例如: "column": ["id","name","age"]。如果要依次写入全部列，使用\*表示, 例如: "column": ["\*"]   
        注意：  
          1、我们强烈不推荐你这样配置，因为当你目的表字段个数、类型等有改动时，你的任务可能运行不正确或者失败  
          2、此处 column 不能配置任何常量值  

    * 必选：是 <br />

    * 默认值：无 <br />

* **splitPk**

	* 描述：GaussDBReader进行数据抽取时，如果指定splitPk，表示用户希望使用splitPk代表的字段进行数据分片，Addax因此会启动并发任务进行数据同步，这样可以大大提高数据同步的效能。

	  推荐splitPk用户使用表主键，因为表主键通常情况下比较均匀，因此切分出来的分片也不容易出现数据热点。

	  目前splitPk仅支持整形数据切分，`不支持浮点、字符串型、日期等其他类型`。如果用户指定其他非支持类型，GaussDBReader将报错！
      
      splitPk设置为空，底层将视作用户不允许对单表进行切分，因此使用单通道进行抽取。

	* 必选：否 <br />

	* 默认值：空 <br />

* **where**

	* 描述：筛选条件，GaussDBReader根据指定的column、table、where条件拼接SQL，并根据这个SQL进行数据抽取。在实际业务场景中，往往会选择当天的数据进行同步，可以将where条件指定为gmt_create > $bizdate 。注意：不可以将where条件指定为limit 10，limit不是SQL的合法where子句。<br />

          where条件可以有效地进行业务增量同步。		where条件不配置或者为空，视作全表同步数据。

	* 必选：否 <br />

	* 默认值：无 <br />

* **querySql**

	* 描述：在有些业务场景下，where这一配置项不足以描述所筛选的条件，用户可以通过该配置型来自定义筛选SQL。当用户配置了这一项之后，Addax系统就会忽略table，column这些配置型，直接使用这个配置项的内容对数据进行筛选，例如需要进行多表join后同步数据，使用select a,b from table_a join table_b on table_a.id = table_b.id <br />

	 `当用户配置querySql时，GaussDBReader直接忽略table、column、where条件的配置`。

	* 必选：否 <br />

	* 默认值：无 <br />

* **fetchSize**

	* 描述：该配置项定义了插件和数据库服务器端每次批量数据获取条数，该值决定了Addax和服务器端的网络交互次数，能够较大的提升数据抽取性能。<br />

	 `注意，该值过大(>2048)可能造成Addax进程OOM。`。

	* 必选：否 <br />

	* 默认值：1024 <br />


### 3.3 类型转换

目前Addax支持大部分GaussDB类型，但也存在部分个别类型没有支持的情况，请注意检查你的类型。

下面列出Addax针对GaussDB类型转换列表:


| Addax 内部类型| GaussDB 数据类型    |
| -------- | -----  |
| Long     |bigint, bigserial, integer, smallint, serial |
| Double   |double precision, money, numeric, real |
| String   |varchar, char, text, bit, inet|
| Date     |date, time, timestamp |
| Boolean  |bool|
| Bytes    |bytea|

请注意:

* `除上述罗列字段类型外，其他类型均不支持; money,inet,bit需用户使用a_inet::varchar类似的语法转换`。


## 4 案例分享

*  本案例实现场景为 从GaussDB的一张源表抽取数据写入到另外一张不同库的GaussDB目标表。        
配置如下: 
- OS  Huawei Cloud EulerOS 2.0  aarch64
- JDK 17 
- Python 3.3.9 
- Apache Maven 3.9.10

### 使用一键安装脚本(非必须)

```shell
/bin/bash -c "$(curl -fsSL https://github.com/Tyhoning/Addax/blob/master/install.sh)"
```
上述脚本会将 Addax 安装到预设的目录(`/opt/addax`)

### 编译及打包

目前编译需要 JDK 17+ 版本

```shell
git clone https://github.com/Tyhoning/Addax.git addax
cd addax
export MAVEN_OPTS="-DskipTests -Dmaven.javadoc.skip=true -Dmaven.source.skip=true -Dgpg.skip=true"
mvn clean package 
mvn package -Pdistribution
```

编译打包成功后，会在项目目录的`target` 目录下创建一个 `addax-<version>`的 文件夹，其中 `<version>` 表示版本。

### 冒烟测试

`job` 子目录包含了大量的任务样本，其中 `job.json` 可以作为冒烟测试

```shell
python bin/addax.py job/job.json
```
上述命令的输出大致如下：
<details>
<summary>点击展开</summary>

```shell
[root@hadoop2 addax-6.0.2-SNAPSHOT]# bin/addax.py job/job.json
/usr/bin/env: ‘python\r’: No such file or directory
/usr/bin/env: use -[v]S to pass options in shebang lines
[root@hadoop2 addax-6.0.2-SNAPSHOT]# python bin/addax.py job/job.json
==================== DEPRECATED WARNING ========================
addax.py is deprecated, It's going to be removed in future release.
As a replacement, you can use addax.sh to run job
==================== DEPRECATED WARNING ========================

2025-07-03 19:52:03.225 [        main] INFO  Engine               - 
  ___      _     _            
 / _ \    | |   | |           
/ /_\ \ __| | __| | __ ___  __
|  _  |/ _` |/ _` |/ _` \ \/ /
| | | | (_| | (_| | (_| |>  < 
\_| |_/\__,_|\__,_|\__,_/_/\_\
:: Addax version ::    (v6.0.2-SNAPSHOT)
2025-07-03 19:52:03.386 [        main] INFO  Engine               - 
{
	"setting":{
		"speed":{
			"byte":-1,
			"channel":1
		},
		"errorLimit":{
			"record":0,
			"percentage":0.02
		}
	},
	"content":{
		"reader":{
			"name":"streamreader",
			"parameter":{
				"column":[
					{
						"value":"addax",
						"type":"string"
					},
					{
						"value":19890604,
						"type":"long"
					},
					{
						"value":"1989-06-04 00:00:00",
						"type":"date"
					},
					{
						"value":true,
						"type":"bool"
					},
					{
						"value":"test",
						"type":"bytes"
					}
				],
				"sliceRecordCount":10
			}
		},
		"writer":{
			"name":"streamwriter",
			"parameter":{
				"print":true,
				"column":[
					"col1"
				],
				"encoding":"UTF-8"
			}
		}
	}
}

2025-07-03 19:52:03.410 [        main] INFO  JobContainer         - The jobContainer begins to process the job.
2025-07-03 19:52:03.418 [       job-0] INFO  JobContainer         - The Reader.Job [streamreader] perform prepare work .
2025-07-03 19:52:03.418 [       job-0] INFO  JobContainer         - The Writer.Job [streamwriter] perform prepare work .
2025-07-03 19:52:03.418 [       job-0] INFO  JobContainer         - Job set Channel-Number to 1 channel(s).
2025-07-03 19:52:03.419 [       job-0] INFO  JobContainer         - The Reader.Job [streamreader] is divided into [1] task(s).
2025-07-03 19:52:03.419 [       job-0] INFO  JobContainer         - The Writer.Job [streamwriter] is divided into [1] task(s).
2025-07-03 19:52:03.438 [       job-0] INFO  JobContainer         - The Scheduler launches [1] taskGroup(s).
2025-07-03 19:52:03.444 [ taskGroup-0] INFO  TaskGroupContainer   - The taskGroupId=[0] started [1] channels for [1] tasks.
2025-07-03 19:52:03.446 [ taskGroup-0] INFO  Channel              - The Channel set byte_speed_limit to -1, No bps activated.
2025-07-03 19:52:03.447 [ taskGroup-0] INFO  Channel              - The Channel set record_speed_limit to -1, No tps activated.
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
addax	19890604	1989-06-04 00:00:00	true	test
2025-07-03 19:52:06.450 [       job-0] INFO  AbstractScheduler    - The scheduler has completed all tasks.
2025-07-03 19:52:06.450 [       job-0] INFO  JobContainer         - The Writer.Job [streamwriter] perform post work.
2025-07-03 19:52:06.451 [       job-0] INFO  JobContainer         - The Reader.Job [streamreader] perform post work.
2025-07-03 19:52:06.455 [       job-0] INFO  StandAloneJobContainerCommunicator - Total 10 records, 260 bytes | Speed 86B/s, 3 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.000s |  All Task WaitReaderTime 0.000s | Percentage 100.00%
2025-07-03 19:52:06.456 [       job-0] INFO  JobContainer         - 
Job start  at             : 2025-07-03 19:52:03
Job end    at             : 2025-07-03 19:52:06
Job took secs             :                  3s
Average   bps             :               86B/s
Average   rps             :              3rec/s
Number of rec             :                  10
Failed record             :                   0

```
</details>

### 案例任务

```shell
python bin/addax.py job/gaussdb2gaussdb.json
```
上述命令的输出大致如下：
<details>
<summary>点击展开</summary>

```shell
[root@hadoop2 addax-6.0.2-SNAPSHOT]# python ./bin/addax.py ./job/gaussdb2gaussdb.json
==================== DEPRECATED WARNING ========================
addax.py is deprecated, It's going to be removed in future release.
As a replacement, you can use addax.sh to run job
==================== DEPRECATED WARNING ========================

2025-07-03 19:58:20.910 [        main] INFO  Engine               - 
  ___      _     _            
 / _ \    | |   | |           
/ /_\ \ __| | __| | __ ___  __
|  _  |/ _` |/ _` |/ _` \ \/ /
| | | | (_| | (_| | (_| |>  < 
\_| |_/\__,_|\__,_|\__,_/_/\_\
:: Addax version ::    (v6.0.2-SNAPSHOT)
2025-07-03 19:58:21.070 [        main] INFO  Engine               - 
{
	"content":{
		"reader":{
			"name":"gaussdbreader",
			"parameter":{
				"username":"******",
				"password":"******",
				"column":[
					"player_id",
					"team_id",
					"player_name",
					"height",
					"substring(current_timestamp,1,23)"
				],
				"connection":{
					"jdbcUrl":"jdbc:gaussdb://*.*.*.*:8000/bigdata",
					"table":[
						"players.addax_src1"
					]
				}
			}
		},
		"writer":{
			"name":"gaussdbwriter",
			"parameter":{
				"column":[
					"*"
				],
				"connection":{
					"jdbcUrl":"jdbc:gaussdb://*.*.*.*:8000/metastore",
					"table":[
						"players.addax_dst1"
					]
				},
				"username":"******",
				"password":"******",
				"preSql":[
					"truncate table players.addax_dst1"
				],
				"postSql":[]
			}
		}
	},
	"setting":{
		"speed":{
			"bytes":-1,
			"channel":1
		}
	}
}

2025-07-03 19:58:21.093 [        main] INFO  JobContainer         - The jobContainer begins to process the job.
2025-07-03 19:58:22.484 [       job-0] INFO  OriginalConfPretreatmentUtil - The table [players.addax_src1] has columns [player_id,team_id,player_name,height,update_time].
2025-07-03 19:58:23.198 [       job-0] INFO  OriginalConfPretreatmentUtil - The table [players.addax_dst1] has columns [player_id,team_id,player_name,height,update_time].
2025-07-03 19:58:23.198 [       job-0] WARN  OriginalConfPretreatmentUtil - There are some risks in the column configuration. Because you did not configure the columns to read the database table, changes in the number and types of fields in your table may affect the correctness of the task or even cause errors.
2025-07-03 19:58:23.200 [       job-0] INFO  OriginalConfPretreatmentUtil - Writing data using [INSERT INTO %s ( player_id,team_id,player_name,height,update_time) VALUES ( ?,?,?,?,? )].
2025-07-03 19:58:23.200 [       job-0] INFO  JobContainer         - The Reader.Job [gaussdbreader] perform prepare work .
2025-07-03 19:58:23.200 [       job-0] INFO  JobContainer         - The Writer.Job [gaussdbwriter] perform prepare work .
2025-07-03 19:58:23.726 [       job-0] INFO  CommonRdbmsWriter$Job - Begin to execute preSqls:[truncate table players.addax_dst1]. context info:jdbc:gaussdb://*.*.*.*:8000/metastore.
2025-07-03 19:58:23.759 [       job-0] INFO  JobContainer         - Job set Channel-Number to 1 channel(s).
2025-07-03 19:58:23.761 [       job-0] INFO  JobContainer         - The Reader.Job [gaussdbreader] is divided into [1] task(s).
2025-07-03 19:58:23.761 [       job-0] INFO  JobContainer         - The Writer.Job [gaussdbwriter] is divided into [1] task(s).
2025-07-03 19:58:23.777 [       job-0] INFO  JobContainer         - The Scheduler launches [1] taskGroup(s).
2025-07-03 19:58:23.785 [ taskGroup-0] INFO  TaskGroupContainer   - The taskGroupId=[0] started [1] channels for [1] tasks.
2025-07-03 19:58:23.787 [ taskGroup-0] INFO  Channel              - The Channel set byte_speed_limit to -1, No bps activated.
2025-07-03 19:58:23.787 [ taskGroup-0] INFO  Channel              - The Channel set record_speed_limit to -1, No tps activated.
2025-07-03 19:58:23.794 [  reader-0-0] INFO  CommonRdbmsReader$Task - Begin reading records by executing SQL query: [SELECT player_id,team_id,player_name,height,substring(current_timestamp,1,23) FROM players.addax_src1 ].
2025-07-03 19:58:24.973 [  reader-0-0] INFO  CommonRdbmsReader$Task - Finished reading records by executing SQL query: [SELECT player_id,team_id,player_name,height,substring(current_timestamp,1,23) FROM players.addax_src1 ].
2025-07-03 19:58:26.789 [       job-0] INFO  AbstractScheduler    - The scheduler has completed all tasks.
2025-07-03 19:58:26.790 [       job-0] INFO  JobContainer         - The Writer.Job [gaussdbwriter] perform post work.
2025-07-03 19:58:26.790 [       job-0] INFO  JobContainer         - The Reader.Job [gaussdbreader] perform post work.
2025-07-03 19:58:26.794 [       job-0] INFO  StandAloneJobContainerCommunicator - Total 9 records, 337 bytes | Speed 112B/s, 3 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.000s |  All Task WaitReaderTime 0.000s | Percentage 100.00%
2025-07-03 19:58:26.795 [       job-0] INFO  JobContainer         - 
Job start  at             : 2025-07-03 19:58:21
Job end    at             : 2025-07-03 19:58:26
Job took secs             :                  5s
Average   bps             :              112B/s
Average   rps             :              3rec/s
Number of rec             :                   9
Failed record             :                   0

```
</details>