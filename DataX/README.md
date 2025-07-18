
# DataX GaussDB 使用指南

## DataX介绍
DataX 是一个异构数据源离线数据同步工具。实现了包括华为GaussDB、MySQL、Oracle、OceanBase、SqlServer、Postgre、HDFS、Hive、ADS、HBase、TableStore(OTS)、MaxCompute(ODPS)、Hologres、DRDS, databend 等各种异构数据源之间高效的数据同步功能。DataX本身作为数据同步框架，将不同数据源的同步抽象为从源头数据源读取数据的Reader插件，以及向目标端写入数据的Writer插件，理论上DataX框架可以支持任意数据源类型的数据同步工作。同时DataX插件体系作为一套生态系统, 每接入一套新数据源该新加入的数据源即可实现和现有的数据源互通。

官方参考文档: [https://github.com/alibaba/DataX/blob/master/introduction.md](https://github.com/alibaba/DataX/blob/master/introduction.md)


## DataX 安装

直接下载DataX工具包：[DataX下载地址](https://datax-opensource.oss-cn-hangzhou.aliyuncs.com/202309/datax.tar.gz)

下载后解压至本地某个目录，进入bin目录，即可运行同步作业：

``` shell
$ cd  {YOUR_DATAX_HOME}/bin
$ python datax.py {YOUR_JOB.json}
```
      
自检脚本：

``` python 
python {YOUR_DATAX_HOME}/bin/datax.py {YOUR_DATAX_HOME}/job/job.json
```

## DataX使用案例

本案例从GaussDB的一张源表离线抽取数据写入到另外一张不同库的GaussDB目标表。

* 配置样例

配置一个自定义SQL的数据库同步任务的作业：  

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
                                    "jdbc:opengauss://host:port/database"
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
								"jdbcUrl": "jdbc:opengauss://host:port/database",
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

* 执行作业      

```shell
python bin/DataX.py job/gaussdb2gaussdb.json
```

作业执行完成，可以查看目标表是否正确写入数据。 
