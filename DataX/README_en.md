# DataX GaussDB User Guide

## DataX Introduction

DataX is an offline data synchronization tool for heterogeneous data sources. Huawei GaussDB, MySQL, Oracle, OceanBase, SQL Server, Postgre, HDFS, Hive, ADS, HBase, TableStore (OTS), MaxCompute (ODPS), Hologres, and DRDS are implemented. Efficient data synchronization between heterogeneous data sources, such as databend. As a data synchronization framework, the DataX abstracts the synchronization of different data sources into the Reader plug-in for reading data from the source data source and the Writer plug-in for writing data to the target end. Theoretically, the DataX framework supports data synchronization of any data source type. In addition, the DataX plug-in system serves as an ecosystem. Each time a new data source is connected, the newly added data source can communicate with the existing data source.

Official reference documents: [https://github.com/alibaba/DataX/blob/master/introduction.md](https://github.com/alibaba/DataX/blob/master/introduction.md)

## DataX Installation

Directly download the DataX tool package:[DataX download address](https://datax-opensource.oss-cn-hangzhou.aliyuncs.com/202309/datax.tar.gz)	

Decompress the downloaded package to a local directory, go to the bin directory, and run the synchronization job.

```
$ cd  {YOUR_DATAX_HOME}/bin
$ python datax.py {YOUR_JOB.json}
```

Self-check script:

```
python {YOUR_DATAX_HOME}/bin/datax.py {YOUR_DATAX_HOME}/job/job.json
```

## DataX Use Cases

In this example, data is extracted from a source table in GaussDB offline and written to a target table in another GaussDB database.

 *  Configuration Example

To configure a database synchronization task for a customized SQL statement, run the following command:

```
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

 *  Executing a Job

```
python bin/DataX.py job/gaussdb2gaussdb.json
```

After the job execution is complete, you can check whether data is correctly written to the target table.

