
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