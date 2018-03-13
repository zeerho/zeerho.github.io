---
title: Quartz
date: 2017-01-01 09:00:00
tags: [Java]
---

# 1.JobDetail 
`JobDetail`的结构如下：

`JobDetail`中有三个关键字段：`JobKey`、`Job`、`JobDataMap`。

- `JobKey`：包含`group`和`name`两个字段，标识了唯一的一个`JobKey`。其中任一`group`内所有`name`不得重复。
- `Job`：所有实现`Job`接口的类必须重写`void execute(JobExecutionContext context)`方法，该方法包含了触发任务后要执行的代码。
- `JobDataMap`：包含了任务所需的数据。

Quartz推荐使用`JobBuilder`来构造`JobDetail`（实际上是`JobDetail`的实现类`JobDetailImpl`）。方式是静态引入`JobBuilder`的静态方法`JobBuilder.newJob()`。

可以自定义任务类，只需实现`Job`接口。
```java
package com.demo.quartz;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class JobImpl implements Job{
    //必须提供无参构造
    public JobImpl(){
    }
    
    public void execute(JobExecutionContext context) throws JobExecutionException{
        public void doSomething(){
        }
    }
}
```

# 2.Trigger
常用的触发器有 **SimpleTrigger** 和 **CronTrigger**。
**SimpleTrigger** 适用于在一个固定时间点和重复次数或固定的时间间隔的情况。
**CronTrigger** 适用于比较复杂的作业调度。需要 cron 表达式。
|字段名|允许的值|允许的特殊字符|
|:-:|:-:|:-:|
|秒|0-59|, - * /|
|分|0-59|, - * /|
|时|0-23|, - * /|
|日|1-31|, - * ? / L W C|
|月|1-12或JAN-DEC|, - * /|
|周几|1-7或SUN-SAT（1表示周日）|, - * ? / L C # |
|年（可选）|empty，1970-2099|, - * /|


- `*`：“每一”。
- `?`：“不确定的值”。由于“日”和“周几”之间存在联系，所以不能用`* * *`表示“每月每日，不论周几”，而要用`* * ?`。
- `-`：指定一个值的范围。
- `,`：指定若干个值。
- `/`：指定一个值的增加幅度，`初值/步长`，在该字段的取值范围内**不会**循环增长。在`/`前加`*`表示从0开始。
- `L`：在“日”字段中表示该月最后一天（lastday）。在“周几”字段中表示周六，`5L`表示该月最后一个周四。
- `W`：表示离该日最近的一个工作日（workday），既可向前也可向后取，但是不能跨月。可与`L`组合成`LW`表示“该月最后一个工作日”。
- `#`：`y#x`表示该月“第x个周y-1”。若该日期不存在则不会触发。
- `C`：表示基于相关日历（calendar）所计算出的值，具体用法存疑。
- 备注：对于“月”和“周几”字段的合法字符来说大小写不敏感。

# 3.实例
## 简单构造方式
```Java
public static void main(String[] args){
    /*********************Scheduler************************/
    //创建调度器工厂
    SchedulerFactory schedulerFactory = new StdSchedulerFactory();
    //获取一个调度器实例
    Scheduler scheduler = schedulerFactory.getScheduler();
    
    /*********************JobDetail************************/
    //创建JobDetail实例
    JobDetail jobDetail = new JobDetail("job_name","job_group",JobImpl.class);
    
    /**********************Trigger*************************/    
    //创建简单触发器实例
    SimpleTrigger simpleTrigger = new SimpleTrigger("trigger_name","trigger_group");
    //设置触发器的开始时间
    Long ctime = System.currentTimeMillis();
    simpleTrigger.setStartTime(new Date(ctime));
    //设置触发周期，单位毫秒
    simpleTrigger.setRepeatInterval(1000);
    //设置触发次数
    simpleTrigger.setRepeatCount(10);
    //设置触发器的结束时间（到达结束时间后即使触发次数未到零也会停止触发）
    simpleTrigger.setEndTime(ctime+60000L);
    //设置优先级，默认5
    simpleTrigger.setPriority(1);
    
    //将jobDetail和simpleTrigger关联起来
    scheduler.scheduleJob(jobDetail,simpleTrigger);
    
    //启动scheduler
    scheduler.start();
}
```

## Quartz推荐构造方式
Quartz提供了一组“Builder-style API”来构造相关的对象。具体方法是通过静态引用对应Builder类中的静态方法来进行对象的构造。这些类包括：`ScheduleBuilder`的各种实现类以及`TriggerBuilder`,`JobBuilder`,
`DateBuilder`,`JobKey`,`TriggerKey`。具体的方式如下：
```java
JobDetail job = newJob(MyJob.class)
                .withIdentity("myJob")
                .build();
            
Trigger trigger = newTrigger() 
                    .withIdentity(triggerKey("myTrigger", "myTriggerGroup"))
                    .withSchedule(simpleSchedule()
                        .withIntervalInHours(1)
                        .repeatForever())
                    .startAt(futureDate(10, MINUTES))
                    .build();

scheduler.scheduleJob(job, trigger);
```

# 4.与Spring整合
## 方式1.作业类继承QuartzJobBean
```Java
package com.demo;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;

public class myJob1 extends QuartzJobBean{
    public void executeInternal(JobExecutionContext context) throws JobExecutionException{
    }
}
```
```xml
<!--这是第一种配置JobDetail的方法-->
<bean id="myjob" class="org.springframework.scheduling.quartz.JobDetailBean">
    <property name="name" value="myJobName"></property>
    <property name="jobClass" value="com.demo.myJob1"></property>
    <property name="jobDataAsMap">
        <map>
            <entry key="blabla"><value>blabla</value>
        </map>
    </property>
</bean>
```
## 方式2.在配置中定义作业类和执行方法
```Java
package com.demo;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;

public class myJob2{
    public void execute(){
    }
}
```
```xml
<!--这是第二种配置JobDetail的方法-->
<bean id="myJob" class="com.demo.myJob2"/>
<bean id="SpringJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <property name="targetObject" ref="myJob2"></property>
    <property name="targetMethod">
        <value>execute</value>
    </property>
</bean>
```

```xml
<!--这是两种方法共用的触发器和调度器工厂的配置-->
<!--触发器-->
<bean id="CronTriggerBean" class="org.springframework.scheduling.quartz.CronTriggerBean">
    <property name="jobDetail" ref="myJob"></property>
    <property name="cronExpression" value="这里填cron表达式"></property>
</bean>

<!--调度工厂-->
<bean id="SpringJobSchedulerFactoryBean" class="org.springframework.scheduling.quartz.SchedulerFactoryBean>
    <property name="triggers">
        <list>
            <ref bean="CronTriggerBean"/>
        </list>
    </property>
</bean>
```



