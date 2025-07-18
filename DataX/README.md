
# DataX 使用指南

___

## 1 DataX介绍
DataX 是一个异构数据源离线数据同步工具。实现了包括华为GaussDB、MySQL、Oracle、OceanBase、SqlServer、Postgre、HDFS、Hive、ADS、HBase、TableStore(OTS)、MaxCompute(ODPS)、Hologres、DRDS, databend 等各种异构数据源之间高效的数据同步功能。DataX本身作为数据同步框架，将不同数据源的同步抽象为从源头数据源读取数据的Reader插件，以及向目标端写入数据的Writer插件，理论上DataX框架可以支持任意数据源类型的数据同步工作。同时DataX插件体系作为一套生态系统, 每接入一套新数据源该新加入的数据源即可实现和现有的数据源互通。
* 参考文档: https://github.com/alibaba/DataX/blob/master/introduction.md   <br />

## 2 依赖系统环境及部署方式
* 环境依赖配置如下:
- OS Linux Huawei Cloud EulerOS 2.0  aarch64
- JDK (1.8以上，推荐1.8)
- Python (2或3都可以) 如: 3.3.9
- Apache Maven 3.x (Compile DataX) 如: 3.9.10
- git version 1.8.3.1 

* 工具部署

    * 方法一、直接下载DataX工具包：[DataX下载地址](https://datax-opensource.oss-cn-hangzhou.aliyuncs.com/202309/datax.tar.gz)

      下载后解压至本地某个目录，进入bin目录，即可运行同步作业：

      ``` shell
      $ cd  {YOUR_DATAX_HOME}/bin
      $ python datax.py {YOUR_JOB.json}
      ```
      
      自检脚本：
      ``` python 
      python {YOUR_DATAX_HOME}/bin/datax.py {YOUR_DATAX_HOME}/job/job.json
      ```
    上述命令的输出大致如下：    
    <details>    
    <summary>点击展开</summary>    
  
    ```shell
    [root@ecs-1d40 datax]# python ./bin/datax.py ./job/job.json
    
    DataX (DATAX-OPENSOURCE-3.0), From Alibaba !
    Copyright (C) 2010-2017, Alibaba Group. All Rights Reserved.
    
    
    2025-07-17 10:45:10.991 [main] INFO  MessageSource - JVM TimeZone: GMT+08:00, Locale: zh_CN
    2025-07-17 10:45:10.993 [main] INFO  MessageSource - use Locale: zh_CN timeZone: sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
    2025-07-17 10:45:11.037 [main] INFO  VMInfo - VMInfo# operatingSystem class => com.sun.management.internal.OperatingSystemImpl
    2025-07-17 10:45:11.042 [main] INFO  Engine - the machine info  =>

	osInfo:	Linux amd64 3.10.0-1160.119.1.el7.x86_64
	jvmInfo:	Oracle Corporation 11 11.0.27+8-LTS-232
	cpu num:	4

	totalPhysicalMemory:	-0.00G
	freePhysicalMemory:	-0.00G
	maxFileDescriptorCount:	-1
	currentOpenFileDescriptorCount:	-1

	GC Names	[G1 Young Generation, G1 Old Generation]

	MEMORY_NAME                    | allocation_size                | init_size                      
	CodeHeap 'profiled nmethods'   | 117.22MB                       | 2.44MB                         
	G1 Old Gen                     | 1,024.00MB                     | 970.00MB                       
	G1 Survivor Space              | -0.00MB                        | 0.00MB                         
	CodeHeap 'non-profiled nmethods' | 117.22MB                       | 2.44MB                         
	Compressed Class Space         | 1,024.00MB                     | 0.00MB                         
	Metaspace                      | -0.00MB                        | 0.00MB                         
	G1 Eden Space                  | -0.00MB                        | 54.00MB                        
	CodeHeap 'non-nmethods'        | 5.56MB                         | 2.44MB                         


    2025-07-17 10:45:11.050 [main] INFO  Engine -
    {
    "setting":{
    "speed":{
    "channel":1
    },
    "errorLimit":{
    "record":0,
    "percentage":0.02
    }
    },
    "content":[
    {
    "reader":{
    "name":"streamreader",
    "parameter":{
    "column":[
    {
    "value":"DataX",
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
    "sliceRecordCount":100000
    }
    },
    "writer":{
    "name":"streamwriter",
    "parameter":{
    "print":false,
    "encoding":"UTF-8"
    }
    }
    }
    ]
    }
    
    2025-07-17 10:45:11.071 [main] INFO  PerfTrace - PerfTrace traceId=job_-1, isEnable=false
    2025-07-17 10:45:11.071 [main] INFO  JobContainer - DataX jobContainer starts job.
    2025-07-17 10:45:11.072 [main] INFO  JobContainer - Set jobId = 0
    2025-07-17 10:45:11.084 [job-0] INFO  JobContainer - jobContainer starts to do prepare ...
    2025-07-17 10:45:11.084 [job-0] INFO  JobContainer - DataX Reader.Job [streamreader] do prepare work .
    2025-07-17 10:45:11.084 [job-0] INFO  JobContainer - DataX Writer.Job [streamwriter] do prepare work .
    2025-07-17 10:45:11.084 [job-0] INFO  JobContainer - jobContainer starts to do split ...
    2025-07-17 10:45:11.084 [job-0] INFO  JobContainer - Job set Channel-Number to 1 channels.
    2025-07-17 10:45:11.085 [job-0] INFO  JobContainer - DataX Reader.Job [streamreader] splits to [1] tasks.
    2025-07-17 10:45:11.085 [job-0] INFO  JobContainer - DataX Writer.Job [streamwriter] splits to [1] tasks.
    2025-07-17 10:45:11.107 [job-0] INFO  JobContainer - jobContainer starts to do schedule ...
    2025-07-17 10:45:11.110 [job-0] INFO  JobContainer - Scheduler starts [1] taskGroups.
    2025-07-17 10:45:11.112 [job-0] INFO  JobContainer - Running by standalone Mode.
    2025-07-17 10:45:11.118 [taskGroup-0] INFO  TaskGroupContainer - taskGroupId=[0] start [1] channels for [1] tasks.
    2025-07-17 10:45:11.125 [taskGroup-0] INFO  Channel - Channel set byte_speed_limit to -1, No bps activated.
    2025-07-17 10:45:11.125 [taskGroup-0] INFO  Channel - Channel set record_speed_limit to -1, No tps activated.
    2025-07-17 10:45:11.136 [taskGroup-0] INFO  TaskGroupContainer - taskGroup[0] taskId[0] attemptCount[1] is started
    2025-07-17 10:45:11.437 [taskGroup-0] INFO  TaskGroupContainer - taskGroup[0] taskId[0] is successed, used[301]ms
    2025-07-17 10:45:11.437 [taskGroup-0] INFO  TaskGroupContainer - taskGroup[0] completed it's tasks.
    2025-07-17 10:45:21.128 [job-0] INFO  StandAloneJobContainerCommunicator - Total 100000 records, 2600000 bytes | Speed 253.91KB/s, 10000 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.023s |  All Task WaitReaderTime 0.036s | Percentage 100.00%
    2025-07-17 10:45:21.128 [job-0] INFO  AbstractScheduler - Scheduler accomplished all tasks.
    2025-07-17 10:45:21.128 [job-0] INFO  JobContainer - DataX Writer.Job [streamwriter] do post work.
    2025-07-17 10:45:21.128 [job-0] INFO  JobContainer - DataX Reader.Job [streamreader] do post work.
    2025-07-17 10:45:21.128 [job-0] INFO  JobContainer - DataX jobId [0] completed successfully.
    2025-07-17 10:45:21.129 [job-0] INFO  HookInvoker - No hook invoked, because base dir not exists or is a file: /home/datax/hook
    2025-07-17 10:45:21.130 [job-0] INFO  JobContainer -
    [total cpu info] =>
    averageCpu                     | maxDeltaCpu                    | minDeltaCpu                    
    -1.00%                         | -1.00%                         | -1.00%
    
    
    	 [total gc info] => 
    		 NAME                 | totalGCCount       | maxDeltaGCCount    | minDeltaGCCount    | totalGCTime        | maxDeltaGCTime     | minDeltaGCTime     
    		 G1 Young Generation  | 0                  | 0                  | 0                  | 0.000s             | 0.000s             | 0.000s             
    		 G1 Old Generation    | 0                  | 0                  | 0                  | 0.000s             | 0.000s             | 0.000s             
    
    2025-07-17 10:45:21.131 [job-0] INFO  JobContainer - PerfTrace not enable!
    2025-07-17 10:45:21.131 [job-0] INFO  StandAloneJobContainerCommunicator - Total 100000 records, 2600000 bytes | Speed 253.91KB/s, 10000 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.023s |  All Task WaitReaderTime 0.036s | Percentage 100.00%
    2025-07-17 10:45:21.132 [job-0] INFO  JobContainer -
    任务启动时刻                    : 2025-07-17 10:45:11
    任务结束时刻                    : 2025-07-17 10:45:21
    任务总计耗时                    :                 10s
    任务平均流量                    :          253.91KB/s
    记录写入速度                    :          10000rec/s
    读出记录总数                    :              100000
    读写失败总数                    :                   0

   ```
   </details>

    * 方法二、下载DataX源码，自己编译：[DataX源码](https://github.com/alibaba/DataX)

      (1)、下载DataX源码：

      ``` shell
      $ git clone git@github.com:alibaba/DataX.git
      ```

      (2)、通过maven打包：

      ``` shell
      $ cd  {DataX_source_code_home}
      $ mvn -U clean package assembly:assembly -Dmaven.test.skip=true
      ```

      打包成功，日志显示如下：

      ```
      [INFO] ------------------------------------------------------------------------
      [INFO] BUILD SUCCESS
      [INFO] ------------------------------------------------------------------------
      [INFO] Total time: 17:48.659s
      [INFO] Finished at: Thu Jul 17 12:07:19 CST 2025
      [INFO] Final Memory: 273M/571M
      [INFO] ------------------------------------------------------------------------
      ```

      打包成功后的DataX包位于 {DataX_source_code_home}/target/datax/datax/ ，结构如下：

      ``` shell
      $ cd  {DataX_source_code_home}
      $ ls ./target/datax/datax/
      bin		conf		job		lib		log		log_perf	plugin		script		tmp
      ```

## 3  DataX使用案例
*  本案例采用方法二实现场景: 从GaussDB的一张源表离线抽取数据写入到另外一张不同库的GaussDB目标表。

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

### 3.2 执行作业      

```shell
python bin/DataX.py job/gaussdb2gaussdb.json
```
上述命令的输出大致如下：
<details>
<summary>点击展开</summary>

```shell
[root@ecs-1d40 datax]# python ./bin/datax.py ./job/gaussdb2gaussdb.json

DataX (DATAX-OPENSOURCE-3.0), From Alibaba !
Copyright (C) 2010-2017, Alibaba Group. All Rights Reserved.


2025-07-17 18:28:33.814 [main] INFO  MessageSource - JVM TimeZone: GMT+08:00, Locale: zh_CN
2025-07-17 18:28:33.816 [main] INFO  MessageSource - use Locale: zh_CN timeZone: sun.util.calendar.ZoneInfo[id="GMT+08:00",offset=28800000,dstSavings=0,useDaylight=false,transitions=0,lastRule=null]
2025-07-17 18:28:33.843 [main] INFO  VMInfo - VMInfo# operatingSystem class => sun.management.OperatingSystemImpl
2025-07-17 18:28:33.848 [main] INFO  Engine - the machine info  => 

	osInfo:	Linux amd64 3.10.0-1160.119.1.el7.x86_64
	jvmInfo:	Red Hat, Inc. 1.8 25.412-b08
	cpu num:	4

	totalPhysicalMemory:	-0.00G
	freePhysicalMemory:	-0.00G
	maxFileDescriptorCount:	-1
	currentOpenFileDescriptorCount:	-1

	GC Names	[PS MarkSweep, PS Scavenge]

	MEMORY_NAME                    | allocation_size                | init_size                      
	PS Eden Space                  | 256.00MB                       | 256.00MB                       
	Code Cache                     | 240.00MB                       | 2.44MB                         
	Compressed Class Space         | 1,024.00MB                     | 0.00MB                         
	PS Survivor Space              | 42.50MB                        | 42.50MB                        
	PS Old Gen                     | 683.00MB                       | 683.00MB                       
	Metaspace                      | -0.00MB                        | 0.00MB                         


2025-07-17 18:28:33.859 [main] INFO  Engine - 
{
	"content":[
		{
			"reader":{
				"name":"gaussdbreader",
				"parameter":{
					"username":"username",
					"password":"****************",
					"column":[
						"player_id",
						"team_id",
						"player_name",
						"height",
						"substring(current_timestamp,1,23)"
					],
					"connection":[
						{
							"jdbcUrl":[
								"jdbc:opengauss://1.1.1.1:8000/database"
							],
							"table":[
								"players.table_src1"
							]
						}
					]
				}
			},
			"writer":{
				"name":"gaussdbwriter",
				"parameter":{
					"column":[
						"*"
					],
					"connection":[
						{
							"jdbcUrl":"jdbc:opengauss://1.1.1.1:8000/database",
							"table":[
								"players.table_dst1"
							]
						}
					],
					"username":"username",
					"password":"****************",
					"preSql":[
						"truncate table players.table_dst1"
					],
					"postSql":[
						
					]
				}
			}
		}
	],
	"setting":{
		"speed":{
			"bytes":-1,
			"channel":10
		}
	}
}

2025-07-17 18:28:33.874 [main] INFO  PerfTrace - PerfTrace traceId=job_-1, isEnable=false
2025-07-17 18:28:33.874 [main] INFO  JobContainer - DataX jobContainer starts job.
2025-07-17 18:28:33.875 [main] INFO  JobContainer - Set jobId = 0
七月 17, 2025 6:28:33 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [c3bd9137-b112-44bc-be4d-69f699f4b8d0] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35552/1.1.1.1:8000] Connection is established. ID: c3bd9137-b112-44bc-be4d-69f699f4b8d0
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: c3bd9137-b112-44bc-be4d-69f699f4b8d0
2025-07-17 18:28:34.336 [job-0] INFO  OriginalConfPretreatmentUtil - Available jdbcUrl:jdbc:opengauss://1.1.1.1:8000/database.
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [4b1d70d5-e64b-4d2f-a67f-431d78863531] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35554/1.1.1.1:8000] Connection is established. ID: 4b1d70d5-e64b-4d2f-a67f-431d78863531
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: 4b1d70d5-e64b-4d2f-a67f-431d78863531
2025-07-17 18:28:34.681 [job-0] INFO  OriginalConfPretreatmentUtil - table:[players.table_src1] has columns:[player_id,team_id,player_name,height,update_time].
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [06cfb009-f80c-4750-9cea-d948833f5c89] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35556/1.1.1.1:8000] Connection is established. ID: 06cfb009-f80c-4750-9cea-d948833f5c89
七月 17, 2025 6:28:34 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: 06cfb009-f80c-4750-9cea-d948833f5c89
2025-07-17 18:28:35.044 [job-0] INFO  OriginalConfPretreatmentUtil - table:[players.table_dst1] all columns:[
player_id,team_id,player_name,height,update_time
].
2025-07-17 18:28:35.045 [job-0] INFO  OriginalConfPretreatmentUtil - Write data [
INSERT INTO %s (player_id,team_id,player_name,height,update_time) VALUES(?,?,?,?,?)
], which jdbcUrl like:[jdbc:opengauss://1.1.1.1:8000/database]
2025-07-17 18:28:35.046 [job-0] INFO  JobContainer - jobContainer starts to do prepare ...
2025-07-17 18:28:35.046 [job-0] INFO  JobContainer - DataX Reader.Job [gaussdbreader] do prepare work .
2025-07-17 18:28:35.046 [job-0] INFO  JobContainer - DataX Writer.Job [gaussdbwriter] do prepare work .
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [26dbc9ae-2ae8-4ca8-8a20-e35eef6559b2] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35558/1.1.1.1:8000] Connection is established. ID: 26dbc9ae-2ae8-4ca8-8a20-e35eef6559b2
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: 26dbc9ae-2ae8-4ca8-8a20-e35eef6559b2
2025-07-17 18:28:35.368 [job-0] INFO  CommonRdbmsWriter$Job - Begin to execute preSqls:[truncate table players.table_dst1]. context info:jdbc:opengauss://1.1.1.1:8000/database.
2025-07-17 18:28:35.418 [job-0] INFO  JobContainer - jobContainer starts to do split ...
2025-07-17 18:28:35.419 [job-0] INFO  JobContainer - Job set Channel-Number to 10 channels.
2025-07-17 18:28:35.422 [job-0] INFO  JobContainer - DataX Reader.Job [gaussdbreader] splits to [1] tasks.
2025-07-17 18:28:35.422 [job-0] INFO  JobContainer - DataX Writer.Job [gaussdbwriter] splits to [1] tasks.
2025-07-17 18:28:35.441 [job-0] INFO  JobContainer - jobContainer starts to do schedule ...
2025-07-17 18:28:35.443 [job-0] INFO  JobContainer - Scheduler starts [1] taskGroups.
2025-07-17 18:28:35.444 [job-0] INFO  JobContainer - Running by standalone Mode.
2025-07-17 18:28:35.456 [taskGroup-0] INFO  TaskGroupContainer - taskGroupId=[0] start [1] channels for [1] tasks.
2025-07-17 18:28:35.459 [taskGroup-0] INFO  Channel - Channel set byte_speed_limit to -1, No bps activated.
2025-07-17 18:28:35.459 [taskGroup-0] INFO  Channel - Channel set record_speed_limit to -1, No tps activated.
2025-07-17 18:28:35.465 [taskGroup-0] INFO  TaskGroupContainer - taskGroup[0] taskId[0] attemptCount[1] is started
2025-07-17 18:28:35.468 [0-0-0-reader] INFO  CommonRdbmsReader$Task - Begin to read record by Sql: [select player_id,team_id,player_name,height,substring(current_timestamp,1,23) from players.table_src1 
] jdbcUrl:[jdbc:opengauss://1.1.1.1:8000/database].
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [6b3f7177-5f9a-4362-b04b-f706b2907853] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [b8e2f189-ac76-4b09-9851-86e1cecd98c5] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35562/1.1.1.1:8000] Connection is established. ID: b8e2f189-ac76-4b09-9851-86e1cecd98c5
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35560/1.1.1.1:8000] Connection is established. ID: 6b3f7177-5f9a-4362-b04b-f706b2907853
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: b8e2f189-ac76-4b09-9851-86e1cecd98c5
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: 6b3f7177-5f9a-4362-b04b-f706b2907853
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [60a96fe3-f1a6-4ea7-93c4-faea7ae7f844] Try to connect. IP: 1.1.1.1:8000
七月 17, 2025 6:28:35 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: [192.168.0.32:35564/1.1.1.1:8000] Connection is established. ID: 60a96fe3-f1a6-4ea7-93c4-faea7ae7f844
七月 17, 2025 6:28:36 下午 org.opengauss.core.v3.ConnectionFactoryImpl openConnectionImpl
信息: Connect complete. ID: 60a96fe3-f1a6-4ea7-93c4-faea7ae7f844
2025-07-17 18:28:45.460 [job-0] INFO  StandAloneJobContainerCommunicator - Total 0 records, 0 bytes | Speed 0B/s, 0 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 0.000s |  All Task WaitReaderTime 0.000s | Percentage 0.00%
2025-07-17 18:28:55.461 [job-0] INFO  StandAloneJobContainerCommunicator - Total 27136 records, 1227179 bytes | Speed 119.84KB/s, 2713 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 7.833s |  All Task WaitReaderTime 1.145s | Percentage 0.00%
2025-07-17 18:29:05.462 [job-0] INFO  StandAloneJobContainerCommunicator - Total 47616 records, 2169259 bytes | Speed 92.00KB/s, 2048 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 16.789s |  All Task WaitReaderTime 2.172s | Percentage 0.00%
2025-07-17 18:29:15.463 [job-0] INFO  StandAloneJobContainerCommunicator - Total 68096 records, 3111339 bytes | Speed 92.00KB/s, 2048 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 25.606s |  All Task WaitReaderTime 3.314s | Percentage 0.00%
2025-07-17 18:29:25.464 [job-0] INFO  StandAloneJobContainerCommunicator - Total 88576 records, 4053419 bytes | Speed 92.00KB/s, 2048 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 34.427s |  All Task WaitReaderTime 4.297s | Percentage 0.00%
2025-07-17 18:29:35.465 [job-0] INFO  StandAloneJobContainerCommunicator - Total 107008 records, 4931288 bytes | Speed 85.73KB/s, 1843 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 42.617s |  All Task WaitReaderTime 5.406s | Percentage 0.00%
2025-07-17 18:29:45.466 [job-0] INFO  StandAloneJobContainerCommunicator - Total 127488 records, 5934808 bytes | Speed 98.00KB/s, 2048 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 52.129s |  All Task WaitReaderTime 6.324s | Percentage 0.00%
2025-07-17 18:29:55.467 [job-0] INFO  StandAloneJobContainerCommunicator - Total 145920 records, 6837976 bytes | Speed 88.20KB/s, 1843 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 60.848s |  All Task WaitReaderTime 7.433s | Percentage 0.00%
2025-07-17 18:30:05.468 [job-0] INFO  StandAloneJobContainerCommunicator - Total 164352 records, 7741144 bytes | Speed 88.20KB/s, 1843 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 69.051s |  All Task WaitReaderTime 8.261s | Percentage 0.00%
2025-07-17 18:30:15.469 [job-0] INFO  StandAloneJobContainerCommunicator - Total 184832 records, 8744664 bytes | Speed 98.00KB/s, 2048 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 78.524s |  All Task WaitReaderTime 9.399s | Percentage 0.00%
2025-07-17 18:30:25.470 [job-0] INFO  StandAloneJobContainerCommunicator - Total 203264 records, 9647832 bytes | Speed 88.20KB/s, 1843 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 86.752s |  All Task WaitReaderTime 10.289s | Percentage 0.00%
2025-07-17 18:30:35.471 [job-0] INFO  StandAloneJobContainerCommunicator - Total 221696 records, 10551000 bytes | Speed 88.20KB/s, 1843 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 95.412s |  All Task WaitReaderTime 11.179s | Percentage 0.00%
2025-07-17 18:30:45.472 [job-0] INFO  StandAloneJobContainerCommunicator - Total 242176 records, 11554520 bytes | Speed 98.00KB/s, 2048 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 105.038s |  All Task WaitReaderTime 12.255s | Percentage 0.00%
2025-07-17 18:31:05.473 [job-0] INFO  StandAloneJobContainerCommunicator - Total 281088 records, 13461208 bytes | Speed 93.10KB/s, 1945 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 122.888s |  All Task WaitReaderTime 14.315s | Percentage 0.00%
2025-07-17 18:31:05.974 [0-0-0-reader] INFO  CommonRdbmsReader$Task - Finished read record by Sql: [select player_id,team_id,player_name,height,substring(current_timestamp,1,23) from players.table_src1 
] jdbcUrl:[jdbc:opengauss://1.1.1.1:8000/database].
2025-07-17 18:31:06.589 [taskGroup-0] INFO  TaskGroupContainer - taskGroup[0] taskId[0] is successed, used[151125]ms
2025-07-17 18:31:06.589 [taskGroup-0] INFO  TaskGroupContainer - taskGroup[0] completed it's tasks.
2025-07-17 18:31:15.474 [job-0] INFO  StandAloneJobContainerCommunicator - Total 300010 records, 14388386 bytes | Speed 90.54KB/s, 1892 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 132.305s |  All Task WaitReaderTime 15.172s | Percentage 100.00%
2025-07-17 18:31:15.474 [job-0] INFO  AbstractScheduler - Scheduler accomplished all tasks.
2025-07-17 18:31:15.475 [job-0] INFO  JobContainer - DataX Writer.Job [gaussdbwriter] do post work.
2025-07-17 18:31:15.475 [job-0] INFO  JobContainer - DataX Reader.Job [gaussdbreader] do post work.
2025-07-17 18:31:15.475 [job-0] INFO  JobContainer - DataX jobId [0] completed successfully.
2025-07-17 18:31:15.476 [job-0] INFO  HookInvoker - No hook invoked, because base dir not exists or is a file: /opt/datax-og/DataX/target/datax/datax/hook
2025-07-17 18:31:15.477 [job-0] INFO  JobContainer - 
	 [total cpu info] => 
		averageCpu                     | maxDeltaCpu                    | minDeltaCpu                    
		-1.00%                         | -1.00%                         | -1.00%
                        

	 [total gc info] => 
		 NAME                 | totalGCCount       | maxDeltaGCCount    | minDeltaGCCount    | totalGCTime        | maxDeltaGCTime     | minDeltaGCTime     
		 PS MarkSweep         | 0                  | 0                  | 0                  | 0.000s             | 0.000s             | 0.000s             
		 PS Scavenge          | 5                  | 5                  | 5                  | 0.022s             | 0.022s             | 0.022s             

2025-07-17 18:31:15.477 [job-0] INFO  JobContainer - PerfTrace not enable!
2025-07-17 18:31:15.477 [job-0] INFO  StandAloneJobContainerCommunicator - Total 300010 records, 14388386 bytes | Speed 87.82KB/s, 1875 records/s | Error 0 records, 0 bytes |  All Task WaitWriterTime 132.305s |  All Task WaitReaderTime 15.172s | Percentage 100.00%
2025-07-17 18:31:15.478 [job-0] INFO  JobContainer - 
任务启动时刻                    : 2025-07-17 18:28:33
任务结束时刻                    : 2025-07-17 18:31:15
任务总计耗时                    :                161s
任务平均流量                    :           87.82KB/s
记录写入速度                    :           1875rec/s
读出记录总数                    :              300010
读写失败总数                    :                   0
```
</details>