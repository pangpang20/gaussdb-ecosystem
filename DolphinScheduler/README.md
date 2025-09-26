
# DolphinScheduler--GaussDB使用指南

## DolphinScheduler介绍
ApacheDolphinScheduler 是一个分布式和可扩展的开源工作流协调平台，具有强大的DAG可视化界面，致力于解决数据管道中复杂的任务依赖关系，并提供“开箱即用”的各种类型的作业

官方参考文档: [DolphinScheduler](https://github.com/apache/dolphinscheduler) 

## DolphinScheduler 安装
* DolphinScheduler 依赖jdk环境 需提前安装好jdk 推荐jdk11
* DolphinScheduler 后端依赖存储任务数据的数据库 推荐mysql
* 直接解压下载的DolphinScheduler安装包， 进入安装目录。
* 启动/停止 DolphinScheduler环境:   
./bin/dolphinscheduler-daemon.sh start standalone-server   
./bin/dolphinscheduler-daemon.sh stop standalone-server   

* 推荐华为云商店体验使用[DolphinScheduler](https://marketplace.huaweicloud.com/contents/31496fe8-a3c9-402a-863f-4b786940a410#productid=OFFI1121281606683963392)
## 案例分享

本案例实现场景为 从GaussDB的一张源表抽取数据写入到另外一张GaussDB目标表。          

* 在UI界面配置一个DolphinScheduler作业：  
<details>
<summary>点击展开</summary>

1. 创建数据源 
![img.png](image/img.png)
2. 创建项目
![img_1.png](image/img_1.png)
3. 创建工作流
![img_2.png](image/img_2.png)
4. 执行工作流(配置调度策略 可选)
![img_3.png](image/img_3.png)

</details>  
   
作业执行完成，可以查看目标表是否正确写入数据。   
