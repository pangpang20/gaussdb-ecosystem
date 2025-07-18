# Quartz GaussDB User Guide

Quartz is a powerful open source job scheduling framework in the Java domain.

For details about the official development guide, see the following: [https://www.quartz-scheduler.org/documentation/quartz-2.3.0/](https://www.quartz-scheduler.org/documentation/quartz-2.3.0/)

This document describes how to connect Quartz to GaussDB.

### driver

Connect to GaussDB. You are advised to use the official driver.

 *  Connection string:`jdbc:gaussdb://host:port/database`
 *  Driver name:`com.huawei.gaussdb.jdbc.Driver`
 *  maven
    
    ```
    <dependency>
        <groupId>com.huaweicloud.gaussdb</groupId>
        <artifactId>gaussdbjdbc</artifactId>
        <version>506.0.0.b058</version>
    </dependency>
    ```

### Introducing Quartz 

```
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.5.0</version>
  </dependency>

  <dependency>
    <groupId>com.huaweicloud.gaussdb</groupId>
    <artifactId>gaussdb-quartz</artifactId>
    <version>2.5.0</version>
  </dependency>
```

### (Optional) HikariCP 

```
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>3.4.5</version>
  </dependency>
```

### Create the quartz.properties file in the resources folder. 

```
#Master Configuration File
org.quartz.scheduler.instanceName = GaussDBScheduler
org.quartz.scheduler.instanceId = AUTO

#Thread pool configuration
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 5
org.quartz.threadPool.threadPriority = 5

#Job Storage Configuration
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
#GaussDB Default Compatibility Mode
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.GaussDBDelegate=
#GaussDB m compatibility mode
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.useProperties = false
org.quartz.jobStore.dataSource = myDS
org.quartz.jobStore.tablePrefix = quartz.qrtz_
#org.quartz.jobStore.isClustered = false
org.quartz.jobStore.isClustered = true
org.quartz.jobStore.clusterCheckinInterval = 20000

#Data Source Configuration
org.quartz.dataSource.myDS.provider = hikaricp
org.quartz.dataSource.myDS.driver = com.huawei.gaussdb.jdbc.Driver
org.quartz.dataSource.myDS.URL = jdbc:gaussdb://ip:port/databaseName?currentSchema=quartz&preparedStatementCacheQueries=0&batchMode=off
org.quartz.dataSource.myDS.user = userName
org.quartz.dataSource.myDS.password = password
org.quartz.dataSource.myDS.maxConnections = 10
```

### Development Job 

```
public class SampleJob implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.out.println("Job Exectued: " + new Date());
        System.out.println("Job Parameters: " + context.getJobDetail().getJobDataMap().getString("param1"));
        
        //Simulate service processing.
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Development Trigger 

```
public class GaussDBQuartzDemo {
    public static void main(String[] args) throws Exception {
        //1. Initialize the scheduler.
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        
        //2. Define JobDetail.
        JobDetail job = JobBuilder.newJob(SampleJob.class)
                .withIdentity("sampleJob", "group1")
                .usingJobData("param1", "param1")
                .storeDurably()
                .build();
        
        //3. Define triggers.
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("cronTrigger", "group1")
                .withSchedule(CronScheduleBuilder.cronSchedule("0/2 * * * * ?"))
                .build();
        
        //4. Register tasks and triggers with the scheduler.
        scheduler.scheduleJob(job, trigger);

        //Registering a Listener
        scheduler.getListenerManager().addJobListener(new SampleJobListener());

        //5. Start the scheduler.
        scheduler.start();
        
        //6. Shut down after running for a period of time.
        Thread.sleep(60000);
        scheduler.shutdown(true);
        System.out.println("Closing ...");
    }
}
```

### Run the table creation statement. ###

Use the GaussDB table creation statement, which can also be obtained from the source code.

 *  Default compatibility mode: SQL statement: https://github.com/HuaweiCloudDeveloper/gaussdb-quartz/blob/main/src/main/resources/tables_gauss_default_compatibility.sql
 *  m Compatibility mode SQL statement: https://github.com/HuaweiCloudDeveloper/gaussdb-quartz/blob/main/src/main/resources/tables_gauss_m_compatibility.sql

