---
title: 定时任务组件FluentScheduler
date: 2017-12-19 16:38:52
tags:
- ASP.NET
categories:
- 插件
+description: "定时任务组件FluentScheduler"
---

> * 定时任务组件FluentScheduler的基础使用方法
> * 动态修改任务调用的间隔时间

### [FluentScheduler Github地址](https://github.com/fluentscheduler/FluentScheduler)
一般常用的方法，在它的github上面就可以找到api，这篇文章的demo主要实现的功能是通过修改appconfig文件内的配置信息，达到修改正在运行的任务时间，当然也可以增加自己的配置代码，如启动，暂停等等。


> 有一处要注意，修改appconfig文件后，读取时必须调用
ConfigurationManager.RefreshSection("appSettings");
否则会出现读取的值是缓存的情况，导致出错。




<!--more-->

### AppConfig文件如下

``` bash
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
  </startup>
  <appSettings>   
    <add key="ChangeStatus" value="1"/>  <!--修改状态-->
    <add key="Interval" value="5"/>  <!--间隔值-->
    <add key="IntervalType" value="Seconds"/>  <!--间隔类型年、月、日、时、分、秒-->
    <add key="HoursOfDay" value="23"/>  <!--间隔类型是天时的小时 取值0-23-->
    <add key="MinutesOfDay" value="59"/>  <!--间隔类型是天时的分钟 取值0-59-->
  </appSettings>
</configuration>
```
### 控制台应用程序源码
``` bash
using System;
using FluentScheduler;
using System.Configuration;

namespace FluentSchedulerDemo
{
    class Program
    {

        static void Main(string[] args)
        {
            //首次允许业务逻辑
            JobManager.AddJob(ServiceLogic, t =>
            {
                t.WithName("ServiceLogic").ToRunEvery(1).Seconds();
            });
            //允许检查
            JobManager.AddJob(CheckSchedule, t =>
            {
                t.WithName("CheckSchedule").ToRunEvery(10).Seconds();
            });
            Console.ReadLine();
        }

        /// <summary>
        /// 定时检查配置文件，动态修改当前工作安排
        /// </summary>
        static void CheckSchedule()
        {
            #region 读取appconfig配置文件
            ConfigurationManager.RefreshSection("appSettings");//非常重要！！！刷新后才能读取到更改后的配置信息
            int changeStatus = Convert.ToInt32(ConfigurationManager.AppSettings["ChangeStatus"]);
            int interval = Convert.ToInt32(ConfigurationManager.AppSettings["Interval"]);
            int hoursOfDay = Convert.ToInt32(ConfigurationManager.AppSettings["HoursOfDay"]);
            int minutesOfDay = Convert.ToInt32(ConfigurationManager.AppSettings["MinutesOfDay"]);
            string intervalType = ConfigurationManager.AppSettings["IntervalType"]; 
            #endregion

            if (changeStatus == 1)
            {
                //修改业务逻辑定时任务
                JobManager.RemoveJob("ServiceLogic");
                Console.WriteLine("CheckSchedule：" + DateTime.Now);
                #region 根据间隔的类型，修改任务计划时间
                switch (intervalType)
                {
                    case "Seconds":
                        JobManager.AddJob(ServiceLogic, t =>
                        {
                            t.WithName("ServiceLogic").ToRunEvery(interval).Seconds();
                        });
                        break;
                    case "Minutes":
                        JobManager.AddJob(ServiceLogic, t =>
                        {
                            t.WithName("ServiceLogic").ToRunEvery(interval).Minutes();
                        });
                        break;
                    case "Hours":
                        JobManager.AddJob(ServiceLogic, t =>
                        {
                            t.WithName("ServiceLogic").ToRunEvery(interval).Hours();
                        });
                        break;
                    case "Days":
                        JobManager.AddJob(ServiceLogic, t =>
                        {
                            t.WithName("ServiceLogic").ToRunEvery(interval).Days().At(hoursOfDay, minutesOfDay);
                        });
                        break;
                    case "Months":
                        JobManager.AddJob(ServiceLogic, t =>
                        {
                            t.WithName("ServiceLogic").ToRunEvery(interval).Months();
                        });
                        break;
                    case "Years":
                        JobManager.AddJob(ServiceLogic, t =>
                        {
                            t.WithName("ServiceLogic").ToRunEvery(interval).Years();
                        });
                        break;
                }
                #endregion

                //更新修改状态
                Configuration configuration = ConfigurationManager.OpenExeConfiguration(ConfigurationUserLevel.None);
                configuration.AppSettings.Settings["ChangeStatus"].Value = "0";
                configuration.Save();

            }
        }
        /// <summary>
        /// 业务逻辑
        /// </summary>
        static void ServiceLogic()
        {
            Console.WriteLine("ServiceLogic：" + DateTime.Now);
            //TODO: 实际的业务逻辑
        }
    }

}


```
### 程序运行截图
{% asset_img 20171219164453.png demo运行图 %}

总的来说一句话：使用两个任务计划，一个用来检查配置文件的状态，随时进行更改另一个业务任务计划的运行状态。