---
title: Log4Net生成空日志文件的解决方法
date: 2016-09-10 20:26:04
tags:
- Log4Net
- ASP.NET日志
- 空日志删除
categories:
- 插件
+description: "Log4Net配置对于空日志文件的解决"
---

设置Log4Net并自动删除空日志
<!--more-->
1、根据网上的配置说明，该配置将记录Error 级别的错误，按照月份分文件夹，按照天来分文件进行日志的记录，
完成了配置如下：
``` bash
<configuration>
  <configSections>
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net, Version=1.2.11.0, Culture=neutral, PublicKeyToken=1B44E1D426115821" />
  </configSections>
  <log4net>
    <root>
      <!--<level value="DEBUG"/>-->
      <level value="ERROR"/>
      <!--根据log级别记录到不同的日志文件-->
      <!--<appender-ref ref="DebugLog" />-->
      <appender-ref ref="ErrorLog" />
    </root>
    <appender name="ErrorLog" type="log4net.Appender.RollingFileAppender">
      <!-- 最后放开注释-->
      <!--<lockingModel type="命名空间.MinimalLockDeleteEmpty" />-->
      <param name="File" value="Log\" />
      <param name="AppendToFile" value="true" />
      <param name="RollingStyle" value="Date" />
      <param name="DatePattern" value="yyyy-MM\\yyyy-MM-dd.'log'" />
      <param name="StaticLogFileName" value="false" />
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%date [%thread] %-5level %logger - %message%newline" />
      </layout>
      <securityContext type="log4net.Util.WindowsSecurityContext">
        <credentials value="Process" />
      </securityContext>
      <filter type="log4net.Filter.LevelRangeFilter">
        <levelMin value="ERROR" />
        <levelMax value="ERROR" />
      </filter>
    </appender>
  </log4net>
</configuration>
```
2、运行上面的（除红色）的设置后，发现就算没有抛出异常，log4net每天同样会创建一个空的日志文件。
3、查找搜索了下国内网站，未发现解决的方法，只是想到如果log4net不支持的化，可以在网站运行后
创建一个定时器，每隔一天检查一下对应的日志文件是否有空，有则删除；
4、google搜索到国外的网站，发现可以继承**FileAppender.MinimalLock**类 重写**ReleaseLock** 方法 来实现写日志完成后检查空文件并删除的功能。
[引用地址](http://stackoverflow.com/questions/2533403/how-to-disable-creation-of-empty-log-file-on-app-start)
``` bash
using log4net.Appender;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;

namespace 命名空间
{
    public class MinimalLockDeleteEmpty : FileAppender.MinimalLock
    {
        public override void ReleaseLock()
        {
            base.ReleaseLock();

            var logFile = new FileInfo(CurrentAppender.File);
            if (logFile.Exists && logFile.Length <= 0)
            {
                logFile.Delete();
            }
        }
    }
}
```
5、最后在配置文件中将类插入完成调用
``` bash
<lockingModel type="命名空间.MinimalLockDeleteEmpty" />
```

6、最后Log4Net效果就是记录中没有空日志文件且存放服务器时按照月份建文件夹，文件夹内按照日期建日志文件。