# Addax GaussDB User Guide

## Introduction to Addax

Addax is an offline synchronization tool for heterogeneous data sources. It is dedicated to implementing stable and efficient data synchronization between heterogeneous data sources, such as relational databases (GaussDB, MySQL, and Oracle), HDFS, Hive, HBase, and FTP.
The GaussDBReader plug-in reads data from GaussDB. The GaussDBWriter plug-in writes data to GaussDB.

Official reference documents:https://wgzhao.github.io/Addax/latest/quickstart/

## Addax installation

You can directly use Docker images by running the following command:

```
docker run -it --rm quay.io/wgzhao/addax:latest /opt/addax/bin/addax.sh /opt/addax/job/job.json
```

## Case sharing

In this case, data is extracted from a source table in GaussDB and written to a target table in another GaussDB database.

 *  To configure a database synchronization task for a customized SQL statement, run the following command:

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

 *  Executing a Job

```
python bin/addax.py job/gaussdb2gaussdb.json
```

After the job execution is complete, you can check whether data is correctly written to the target table.

