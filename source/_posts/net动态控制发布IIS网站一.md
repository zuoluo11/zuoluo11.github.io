---
title: .net动态控制发布IIS网站（一）
date: 2017-06-20 16:22:31
tags:
- ASP.Net
- IIS
categories:
- C#基础
+description: "变相实现SAAS，.net控制动态发布网站"
---
# 资料
1.[Microsoft.Web.Administration  IIS操作API](https://msdn.microsoft.com/en-us/library/microsoft.web.administration)

# 代码简介
主要逻辑功能 拷贝网站发布文件夹->IIS创建站点(获得空闲端口)->绑定域名

涉及到两个工具类 
1、IISManager IIS管理类
2、DirFileHelper 文件及文件夹操作类
<!--more-->
## 代码详情
```C
using CommonTools;
using System;
using System.Collections;
namespace IIS
{
    class Program
    {

        static void Main(string[] args)
        {
            string folderName = "D:\\WebSiteTest";//发布网站目录
            string physicsPath = "D:\\WebSite0421";//模板网站目录



            ArrayList ports = IISManager.GetAllUsePorts();
            for (int i = 0; i < 30; i++)
            {
                string port = IISManager.GetFreePort(ref ports);
                //Console.WriteLine(port);
                string currentWebsiteName = "WebSite" + i;
                string currentPath = folderName + "\\" + currentWebsiteName;
                DirFileHelper.CopyFolder(physicsPath, currentPath);
                IISManager.CreateWebSite(currentWebsiteName, currentPath, port);
                IISManager.BindHost(currentWebsiteName, "www.baidu.com" + i);

                System.Console.WriteLine("网站名：" + currentWebsiteName + "端口号：" + port + "路径：" + currentPath + " 域名:" + "www.baidu.com" + i);
            }
            //for (int i = 0; i < 30; i++)
            //{
            //    IISManager.DeleteWebSite("WebSite" + i);
                
            //}

            Console.ReadKey();
        }


    }
}

```
```C
using Microsoft.Web.Administration;
using System;
using System.Collections;
using System.Diagnostics;
using System.IO;
using System.Text.RegularExpressions;

namespace CommonTools
{
    public static class IISManager
    {
        private static ServerManager serverManager = new ServerManager();
        /// <summary>
        /// 创建站点
        /// </summary>
        /// <param name="websiteName">站点名</param>
        /// <param name="physicsPath">物理路径</param>
        /// <param name="ip">ip地址</param>
        /// <param name="port">端口号</param>
        /// <param name="poolver">应用程序池的DOTNET FRAMEWORK版本</param>
        /// <param name="binding">使用的协议</param>
        public static void CreateWebSite(string websiteName, string physicsPath,
            string port, string ip = "*", string poolver = "v4.0", string binding = "http")
        {
            try
            {
                string fullIP = ip + ":" + port + ":";

                ApplicationPool newPool = serverManager.ApplicationPools.Add(websiteName);
                newPool.ManagedRuntimeVersion = poolver;

                Site mySide = serverManager.Sites.Add(websiteName, binding, fullIP, physicsPath);
                mySide.Applications[0].ApplicationPoolName = websiteName;

                serverManager.CommitChanges();
            }
            catch (Exception error)
            {

            }

        }


        /// <summary>
        /// 删除站点
        /// </summary>
        /// <param name="websiteName">站点名称</param>
        public static void DeleteWebSite(string websiteName)
        {

            serverManager.Sites.Remove(serverManager.Sites[websiteName]);
            serverManager.ApplicationPools.Remove(serverManager.ApplicationPools[websiteName]);
            serverManager.CommitChanges();
        }


        /// <summary>
        /// 获得空闲端口
        /// </summary>
        /// <returns></returns>
        public static ArrayList GetAllUsePorts()
        {
            
            Process p = new Process();
            p.StartInfo.FileName = "cmd.exe";//设置启动的应用程序    
            p.StartInfo.UseShellExecute = false;//禁止使用操作系统外壳程序启动进程    
            p.StartInfo.RedirectStandardInput = true;//应用程序的输入从流中读取    
            p.StartInfo.RedirectStandardOutput = true;//应用程序的输出写入流中    
            p.StartInfo.RedirectStandardError = true;//将错误信息写入流    
            p.StartInfo.CreateNoWindow = true;//是否在新窗口中启动进程    
            p.Start();
            //p.StandardInput.WriteLine(@"netstat -a -n>c:\port.txt");//将字符串写入文本流    
            p.StandardInput.WriteLine(@"netstat -a -n");
            p.StandardInput.WriteLine("exit");
            string str;
            StreamReader reader = p.StandardOutput;
            str = reader.ReadLine();
            ArrayList ports = new ArrayList();
            ////匹配出端口号
            string pattern = @":\d+"; //正则表达式字符串
            Regex regex = new Regex(pattern);
            while (!reader.EndOfStream)
            {
                Match match = regex.Match(str);
                if (match.Success)
                {
                    string port = match.Groups[0].Value.Substring(1);
                    ports.Add(int.Parse(port));
                }
                str = reader.ReadLine();
            }
           
            return ports;
        }


        /// <summary>
        /// 追加绑定域名
        /// </summary>
        /// <param name="websiteName">站点名</param>
        /// <param name="hostName">域名</param>
        public static void BindHost(string websiteName, string hostName)
        {
            BindingCollection bindings = serverManager.Sites[websiteName].Bindings;
            for (int i = 1; i < bindings.Count; i++)
            {
                serverManager.Sites[websiteName].Bindings.RemoveAt(i);
            }
            serverManager.Sites[websiteName].Bindings.Add("*:80:" + hostName, "http");
            serverManager.CommitChanges();
        }


        /// <summary>
        /// 获得当前IIS网站运行列表
        /// </summary>
        public static void GetListOfIIS()
        {

            string StateStr = "";
            for (int i = 0; i < serverManager.Sites.Count; i++)
            {

                switch (serverManager.Sites[i].State)
                {
                    case ObjectState.Started:
                        {
                            StateStr = "正常"; break;
                        }
                    case ObjectState.Starting:
                        {
                            StateStr = "正在启动"; break;
                        }
                    case ObjectState.Stopping:
                        {
                            StateStr = "正在关闭"; break;
                        }
                    case ObjectState.Stopped:
                        {
                            StateStr = "关闭"; break;
                        }
                }
                Console.WriteLine(serverManager.Sites[i].Name + "【" + StateStr + "】");
            }
        }

        /// <summary>
        /// 获得当前IIS站点状态
        /// </summary>
        public static void ReadConfig()
        {


            System.Console.WriteLine("应用程序池默认设置：");
            System.Console.WriteLine("\t常规：");
            System.Console.WriteLine("\t\t.NET Framework 版本：{0}", serverManager.ApplicationPoolDefaults.ManagedRuntimeVersion);
            System.Console.WriteLine("\t\t队列长度：{0}", serverManager.ApplicationPoolDefaults.QueueLength);
            System.Console.WriteLine("\t\t托管管道模式：{0}", serverManager.ApplicationPoolDefaults.ManagedPipelineMode.ToString());
            System.Console.WriteLine("\t\t自动启动：{0}", serverManager.ApplicationPoolDefaults.AutoStart);

            System.Console.WriteLine("\tCPU：");
            System.Console.WriteLine("\t\t处理器关联掩码：{0}", serverManager.ApplicationPoolDefaults.Cpu.SmpProcessorAffinityMask);
            System.Console.WriteLine("\t\t限制：{0}", serverManager.ApplicationPoolDefaults.Cpu.Limit);
            System.Console.WriteLine("\t\t限制操作：{0}", serverManager.ApplicationPoolDefaults.Cpu.Action.ToString());
            System.Console.WriteLine("\t\t限制间隔（分钟）：{0}", serverManager.ApplicationPoolDefaults.Cpu.ResetInterval.TotalMinutes);
            System.Console.WriteLine("\t\t已启用处理器关联：{0}", serverManager.ApplicationPoolDefaults.Cpu.SmpAffinitized);

            System.Console.WriteLine("\t回收：");
            System.Console.WriteLine("\t\t发生配置更改时禁止回收：{0}", serverManager.ApplicationPoolDefaults.Recycling.DisallowRotationOnConfigChange);
            System.Console.WriteLine("\t\t固定时间间隔（分钟）：{0}", serverManager.ApplicationPoolDefaults.Recycling.PeriodicRestart.Time.TotalMinutes);
            System.Console.WriteLine("\t\t禁用重叠回收：{0}", serverManager.ApplicationPoolDefaults.Recycling.DisallowOverlappingRotation);
            System.Console.WriteLine("\t\t请求限制：{0}", serverManager.ApplicationPoolDefaults.Recycling.PeriodicRestart.Requests);
            System.Console.WriteLine("\t\t虚拟内存限制（KB）：{0}", serverManager.ApplicationPoolDefaults.Recycling.PeriodicRestart.Memory);
            System.Console.WriteLine("\t\t专用内存限制（KB）：{0}", serverManager.ApplicationPoolDefaults.Recycling.PeriodicRestart.PrivateMemory);
            System.Console.WriteLine("\t\t特定时间：{0}", serverManager.ApplicationPoolDefaults.Recycling.PeriodicRestart.Schedule.ToString());
            System.Console.WriteLine("\t\t生成回收事件日志条目：{0}", serverManager.ApplicationPoolDefaults.Recycling.LogEventOnRecycle.ToString());

            System.Console.WriteLine("\t进程孤立：");
            System.Console.WriteLine("\t\t可执行文件：{0}", serverManager.ApplicationPoolDefaults.Failure.OrphanActionExe);
            System.Console.WriteLine("\t\t可执行文件参数：{0}", serverManager.ApplicationPoolDefaults.Failure.OrphanActionParams);
            System.Console.WriteLine("\t\t已启用：{0}", serverManager.ApplicationPoolDefaults.Failure.OrphanWorkerProcess);

            System.Console.WriteLine("\t进程模型：");
            System.Console.WriteLine("\t\tPing 间隔（秒）：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.PingInterval.TotalSeconds);
            System.Console.WriteLine("\t\tPing 最大响应时间（秒）：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.PingResponseTime.TotalSeconds);
            System.Console.WriteLine("\t\t标识：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.IdentityType);
            System.Console.WriteLine("\t\t用户名：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.UserName);
            System.Console.WriteLine("\t\t密码：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.Password);
            System.Console.WriteLine("\t\t关闭时间限制（秒）：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.ShutdownTimeLimit.TotalSeconds);
            System.Console.WriteLine("\t\t加载用户配置文件：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.LoadUserProfile);
            System.Console.WriteLine("\t\t启动时间限制（秒）：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.StartupTimeLimit.TotalSeconds);
            System.Console.WriteLine("\t\t允许 Ping：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.PingingEnabled);
            System.Console.WriteLine("\t\t闲置超时（分钟）：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.IdleTimeout.TotalMinutes);
            System.Console.WriteLine("\t\t最大工作进程数：{0}", serverManager.ApplicationPoolDefaults.ProcessModel.MaxProcesses);

            System.Console.WriteLine("\t快速故障防护：");
            System.Console.WriteLine("\t\t“服务不可用”响应类型：{0}", serverManager.ApplicationPoolDefaults.Failure.LoadBalancerCapabilities.ToString());
            System.Console.WriteLine("\t\t故障间隔（分钟）：{0}", serverManager.ApplicationPoolDefaults.Failure.RapidFailProtectionInterval.TotalMinutes);
            System.Console.WriteLine("\t\t关闭可执行文件：{0}", serverManager.ApplicationPoolDefaults.Failure.AutoShutdownExe);
            System.Console.WriteLine("\t\t关闭可执行文件参数：{0}", serverManager.ApplicationPoolDefaults.Failure.AutoShutdownParams);
            System.Console.WriteLine("\t\t已启用：{0}", serverManager.ApplicationPoolDefaults.Failure.RapidFailProtection);
            System.Console.WriteLine("\t\t最大故障数：{0}", serverManager.ApplicationPoolDefaults.Failure.RapidFailProtectionMaxCrashes);
            System.Console.WriteLine("\t\t允许32位应用程序运行在64位 Windows 上：{0}", serverManager.ApplicationPoolDefaults.Enable32BitAppOnWin64);

            System.Console.WriteLine();
            System.Console.WriteLine("网站默认设置：");
            System.Console.WriteLine("\t常规：");
            System.Console.WriteLine("\t\t物理路径凭据：UserName={0}, Password={1}", serverManager.VirtualDirectoryDefaults.UserName, serverManager.VirtualDirectoryDefaults.Password);
            System.Console.WriteLine("\t\t物理路径凭据登录类型：{0}", serverManager.VirtualDirectoryDefaults.LogonMethod.ToString());
            System.Console.WriteLine("\t\t应用程序池：{0}", serverManager.ApplicationDefaults.ApplicationPoolName);
            System.Console.WriteLine("\t\t自动启动：{0}", serverManager.SiteDefaults.ServerAutoStart);
            System.Console.WriteLine("\t行为：");
            System.Console.WriteLine("\t\t连接限制：");
            System.Console.WriteLine("\t\t\t连接超时（秒）：{0}", serverManager.SiteDefaults.Limits.ConnectionTimeout.TotalSeconds);
            System.Console.WriteLine("\t\t\t最大并发连接数：{0}", serverManager.SiteDefaults.Limits.MaxConnections);
            System.Console.WriteLine("\t\t\t最大带宽（字节/秒）：{0}", serverManager.SiteDefaults.Limits.MaxBandwidth);
            System.Console.WriteLine("\t\t失败请求跟踪：");
            System.Console.WriteLine("\t\t\t跟踪文件的最大数量：{0}", serverManager.SiteDefaults.TraceFailedRequestsLogging.MaxLogFiles);
            System.Console.WriteLine("\t\t\t目录：{0}", serverManager.SiteDefaults.TraceFailedRequestsLogging.Directory);
            System.Console.WriteLine("\t\t\t已启用：{0}", serverManager.SiteDefaults.TraceFailedRequestsLogging.Enabled);
            System.Console.WriteLine("\t\t已启用的协议：{0}", serverManager.ApplicationDefaults.EnabledProtocols);

            foreach (var s in serverManager.Sites)//遍历网站
            {
                System.Console.WriteLine();
                System.Console.WriteLine("模式名：{0}", s.Schema.Name);
                System.Console.WriteLine("编号：{0}", s.Id);
                System.Console.WriteLine("网站名称：{0}", s.Name);
                System.Console.WriteLine("物理路径：{0}", s.Applications["/"].VirtualDirectories["/"].PhysicalPath);
                System.Console.WriteLine("物理路径凭据：{0}", s.Methods.ToString());
                System.Console.WriteLine("应用程序池：{0}", s.Applications["/"].ApplicationPoolName);
                System.Console.WriteLine("已启用的协议：{0}", s.Applications["/"].EnabledProtocols);
                System.Console.WriteLine("自动启动：{0}", s.ServerAutoStart);
                System.Console.WriteLine("运行状态：{0}", s.State.ToString());

                System.Console.WriteLine("网站绑定：");
                foreach (var tmp in s.Bindings)
                {
                    System.Console.WriteLine("\t类型：{0}", tmp.Protocol);
                    //System.Console.WriteLine("\tIP 地址：{0}", tmp.EndPoint.Address.ToString());
                    //System.Console.WriteLine("\t端口：{0}", tmp.EndPoint.Port.ToString());
                    System.Console.WriteLine("\t主机名：{0}", tmp.Host);
                    //System.Console.WriteLine(tmp.BindingInformation);
                    //System.Console.WriteLine(tmp.CertificateStoreName);
                    //System.Console.WriteLine(tmp.IsIPPortHostBinding);
                    //System.Console.WriteLine(tmp.IsLocallyStored);
                    //System.Console.WriteLine(tmp.UseDsMapper);
                }

                System.Console.WriteLine("连接限制：");
                System.Console.WriteLine("\t连接超时（秒）：{0}", s.Limits.ConnectionTimeout.TotalSeconds);
                System.Console.WriteLine("\t最大并发连接数：{0}", s.Limits.MaxConnections);
                System.Console.WriteLine("\t最大带宽（字节/秒）：{0}", s.Limits.MaxBandwidth);

                System.Console.WriteLine("失败请求跟踪：");
                System.Console.WriteLine("\t跟踪文件的最大数量：{0}", s.TraceFailedRequestsLogging.MaxLogFiles);
                System.Console.WriteLine("\t目录：{0}", s.TraceFailedRequestsLogging.Directory);
                System.Console.WriteLine("\t已启用：{0}", s.TraceFailedRequestsLogging.Enabled);

                System.Console.WriteLine("日志：");
                //System.Console.WriteLine("\t启用日志服务：{0}", s.LogFile.Enabled);
                System.Console.WriteLine("\t格式：{0}", s.LogFile.LogFormat.ToString());
                System.Console.WriteLine("\t目录：{0}", s.LogFile.Directory);
                System.Console.WriteLine("\t文件包含字段：{0}", s.LogFile.LogExtFileFlags.ToString());
                System.Console.WriteLine("\t计划：{0}", s.LogFile.Period.ToString());
                System.Console.WriteLine("\t最大文件大小（字节）：{0}", s.LogFile.TruncateSize);
                System.Console.WriteLine("\t使用本地时间进行文件命名和滚动更新：{0}", s.LogFile.LocalTimeRollover);

                System.Console.WriteLine("----应用程序的默认应用程序池：{0}", s.ApplicationDefaults.ApplicationPoolName);
                System.Console.WriteLine("----应用程序的默认已启用的协议：{0}", s.ApplicationDefaults.EnabledProtocols);
                //System.Console.WriteLine("----应用程序的默认物理路径凭据：{0}", s.ApplicationDefaults.Methods.ToString());
                //System.Console.WriteLine("----虚拟目录的默认物理路径凭据：{0}", s.VirtualDirectoryDefaults.Methods.ToString());
                System.Console.WriteLine("----虚拟目录的默认物理路径凭据登录类型：{0}", s.VirtualDirectoryDefaults.LogonMethod.ToString());
                System.Console.WriteLine("----虚拟目录的默认用户名：{0}", s.VirtualDirectoryDefaults.UserName);
                System.Console.WriteLine("----虚拟目录的默认用户密码：{0}", s.VirtualDirectoryDefaults.Password);
                System.Console.WriteLine("应用程序 列表：");
                foreach (var tmp in s.Applications)
                {
                    if (tmp.Path != "/")
                    {
                        System.Console.WriteLine("\t模式名：{0}", tmp.Schema.Name);
                        System.Console.WriteLine("\t虚拟路径：{0}", tmp.Path);
                        System.Console.WriteLine("\t物理路径：{0}", tmp.VirtualDirectories["/"].PhysicalPath);
                        //System.Console.WriteLine("\t物理路径凭据：{0}", tmp.Methods.ToString());
                        System.Console.WriteLine("\t应用程序池：{0}", tmp.ApplicationPoolName);
                        System.Console.WriteLine("\t已启用的协议：{0}", tmp.EnabledProtocols);
                    }
                    System.Console.WriteLine("\t虚拟目录 列表：");
                    foreach (var tmp2 in tmp.VirtualDirectories)
                    {
                        if (tmp2.Path != "/")
                        {
                            System.Console.WriteLine("\t\t模式名：{0}", tmp2.Schema.Name);
                            System.Console.WriteLine("\t\t虚拟路径：{0}", tmp2.Path);
                            System.Console.WriteLine("\t\t物理路径：{0}", tmp2.PhysicalPath);
                            //System.Console.WriteLine("\t\t物理路径凭据：{0}", tmp2.Methods.ToString());
                            System.Console.WriteLine("\t\t物理路径凭据登录类型：{0}", tmp2.LogonMethod.ToString());
                        }
                    }
                }
            }
        }
        /// <summary>
        /// 获得可用端口号
        /// </summary>
        /// <param name="ports"></param>
        /// <returns></returns>
        public static string GetFreePort(ref ArrayList ports)
        {
            ////1024－49151
            string result = "";
            for (int i = 1024; i < 49151; i++)
            {
                if (!ports.Contains(i))
                {
                    ports.Add(i);
                    result = i.ToString();
                    break;
                }
            }
            return result;
        }
    }
}

```

```C
using System;
using System.Text;
using System.IO;


namespace CommonTools
{
    /// <summary>
    /// 文件操作夹
    /// </summary>
    public static class DirFileHelper
    {
        #region 检测指定目录是否存在
        /// <summary>
        /// 检测指定目录是否存在
        /// </summary>
        /// <param name="directoryPath">目录的绝对路径</param>
        /// <returns></returns>
        public static bool IsExistDirectory(string directoryPath)
        {
            return Directory.Exists(directoryPath);
        }
        #endregion

        #region 检测指定文件是否存在,如果存在返回true
        /// <summary>
        /// 检测指定文件是否存在,如果存在则返回true。
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>        
        public static bool IsExistFile(string filePath)
        {
            return File.Exists(filePath);
        }
        #endregion

        #region 获取指定目录中的文件列表
        /// <summary>
        /// 获取指定目录中所有文件列表
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>        
        public static string[] GetFileNames(string directoryPath)
        {
            //如果目录不存在，则抛出异常
            if (!IsExistDirectory(directoryPath))
            {
                throw new FileNotFoundException();
            }

            //获取文件列表
            return Directory.GetFiles(directoryPath);
        }
        #endregion

        #region 获取指定目录中所有子目录列表,若要搜索嵌套的子目录列表,请使用重载方法.
        /// <summary>
        /// 获取指定目录中所有子目录列表,若要搜索嵌套的子目录列表,请使用重载方法.
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>        
        public static string[] GetDirectories(string directoryPath)
        {
            try
            {
                return Directory.GetDirectories(directoryPath);
            }
            catch (IOException ex)
            {
                throw ex;
            }
        }
        #endregion

        #region 获取指定目录及子目录中所有文件列表
        /// <summary>
        /// 获取指定目录及子目录中所有文件列表
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>
        /// <param name="searchPattern">模式字符串，"*"代表0或N个字符，"?"代表1个字符。
        /// 范例："Log*.xml"表示搜索所有以Log开头的Xml文件。</param>
        /// <param name="isSearchChild">是否搜索子目录</param>
        public static string[] GetFileNames(string directoryPath, string searchPattern, bool isSearchChild)
        {
            //如果目录不存在，则抛出异常
            if (!IsExistDirectory(directoryPath))
            {
                throw new FileNotFoundException();
            }

            try
            {
                if (isSearchChild)
                {
                    return Directory.GetFiles(directoryPath, searchPattern, SearchOption.AllDirectories);
                }
                else
                {
                    return Directory.GetFiles(directoryPath, searchPattern, SearchOption.TopDirectoryOnly);
                }
            }
            catch (IOException ex)
            {
                throw ex;
            }
        }
        #endregion

        #region 检测指定目录是否为空
        /// <summary>
        /// 检测指定目录是否为空
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>        
        public static bool IsEmptyDirectory(string directoryPath)
        {
            try
            {
                //判断是否存在文件
                string[] fileNames = GetFileNames(directoryPath);
                if (fileNames.Length > 0)
                {
                    return false;
                }

                //判断是否存在文件夹
                string[] directoryNames = GetDirectories(directoryPath);
                if (directoryNames.Length > 0)
                {
                    return false;
                }

                return true;
            }
            catch
            {
                //这里记录日志
                //LogHelper.WriteTraceLog(TraceLogLevel.Error, ex.Message);
                return true;
            }
        }
        #endregion

        #region 检测指定目录中是否存在指定的文件
        /// <summary>
        /// 检测指定目录中是否存在指定的文件,若要搜索子目录请使用重载方法.
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>
        /// <param name="searchPattern">模式字符串，"*"代表0或N个字符，"?"代表1个字符。
        /// 范例："Log*.xml"表示搜索所有以Log开头的Xml文件。</param>        
        public static bool Contains(string directoryPath, string searchPattern)
        {
            try
            {
                //获取指定的文件列表
                string[] fileNames = GetFileNames(directoryPath, searchPattern, false);

                //判断指定文件是否存在
                if (fileNames.Length == 0)
                {
                    return false;
                }
                else
                {
                    return true;
                }
            }
            catch (Exception ex)
            {
                throw new Exception(ex.Message);
                //LogHelper.WriteTraceLog(TraceLogLevel.Error, ex.Message);
            }
        }

        /// <summary>
        /// 检测指定目录中是否存在指定的文件
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>
        /// <param name="searchPattern">模式字符串，"*"代表0或N个字符，"?"代表1个字符。
        /// 范例："Log*.xml"表示搜索所有以Log开头的Xml文件。</param> 
        /// <param name="isSearchChild">是否搜索子目录</param>
        public static bool Contains(string directoryPath, string searchPattern, bool isSearchChild)
        {
            try
            {
                //获取指定的文件列表
                string[] fileNames = GetFileNames(directoryPath, searchPattern, true);

                //判断指定文件是否存在
                if (fileNames.Length == 0)
                {
                    return false;
                }
                else
                {
                    return true;
                }
            }
            catch (Exception ex)
            {
                throw new Exception(ex.Message);
                //LogHelper.WriteTraceLog(TraceLogLevel.Error, ex.Message);
            }
        }
        #endregion

        #region 创建目录
        /// <summary>
        /// 创建目录
        /// </summary>
        /// <param name="dir">要创建的目录路径包括目录名</param>
        public static void CreateDir(string dir)
        {
            if (dir.Length == 0) return;
            if (!Directory.Exists(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir))
                Directory.CreateDirectory(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir);
        }
        #endregion

        #region 删除目录
        /// <summary>
        /// 删除目录
        /// </summary>
        /// <param name="dir">要删除的目录路径和名称</param>
        public static void DeleteDir(string dir)
        {
            if (dir.Length == 0) return;
            if (Directory.Exists(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir))
                Directory.Delete(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir);
        }
        #endregion

        #region 删除文件
        /// <summary>
        /// 删除文件
        /// </summary>
        /// <param name="file">要删除的文件路径和名称</param>
        public static void DeleteFile(string file)
        {
            if (File.Exists(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + file))
                File.Delete(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + file);
        }
        #endregion

        #region 创建文件
        /// <summary>
        /// 创建文件
        /// </summary>
        /// <param name="dir">带后缀的文件名</param>
        /// <param name="pagestr">文件内容</param>
        public static void CreateFile(string dir, string pagestr)
        {
            dir = dir.Replace("/", "\\");
            if (dir.IndexOf("\\") > -1)
                CreateDir(dir.Substring(0, dir.LastIndexOf("\\")));
            System.IO.StreamWriter sw = new System.IO.StreamWriter(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir, false, System.Text.Encoding.GetEncoding("GB2312"));
            sw.Write(pagestr);
            sw.Close();
        }
        #endregion

        #region 移动文件(剪贴--粘贴)
        /// <summary>
        /// 移动文件(剪贴--粘贴)
        /// </summary>
        /// <param name="dir1">要移动的文件的路径及全名(包括后缀)</param>
        /// <param name="dir2">文件移动到新的位置,并指定新的文件名</param>
        public static void MoveFile(string dir1, string dir2)
        {
            dir1 = dir1.Replace("/", "\\");
            dir2 = dir2.Replace("/", "\\");
            if (File.Exists(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir1))
                File.Move(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir1, System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir2);
        }
        #endregion

        #region 复制文件
        /// <summary>
        /// 复制文件
        /// </summary>
        /// <param name="dir1">要复制的文件的路径已经全名(包括后缀)</param>
        /// <param name="dir2">目标位置,并指定新的文件名</param>
        public static void CopyFile(string dir1, string dir2)
        {
            dir1 = dir1.Replace("/", "\\");
            dir2 = dir2.Replace("/", "\\");
            if (File.Exists(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir1))
            {
                File.Copy(System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir1, System.Web.HttpContext.Current.Request.PhysicalApplicationPath + "\\" + dir2, true);
            }
        }
        #endregion

        #region 根据时间得到目录名 / 格式:yyyyMMdd 或者 HHmmssff
        /// <summary>
        /// 根据时间得到目录名yyyyMMdd
        /// </summary>
        /// <returns></returns>
        public static string GetDateDir()
        {
            return DateTime.Now.ToString("yyyyMMdd");
        }
        /// <summary>
        /// 根据时间得到文件名HHmmssff
        /// </summary>
        /// <returns></returns>
        public static string GetDateFile()
        {
            return DateTime.Now.ToString("HHmmssff");
        }
        #endregion

        #region 复制文件夹
        /// <summary>
        /// 复制文件夹(递归)
        /// </summary>
        /// <param name="varFromDirectory">源文件夹路径</param>
        /// <param name="varToDirectory">目标文件夹路径</param>
        public static void CopyFolder(string varFromDirectory, string varToDirectory)
        {
            Directory.CreateDirectory(varToDirectory);

            if (!Directory.Exists(varFromDirectory)) return;

            string[] directories = Directory.GetDirectories(varFromDirectory);

            if (directories.Length > 0)
            {
                foreach (string d in directories)
                {
                    CopyFolder(d, varToDirectory + d.Substring(d.LastIndexOf("\\")));
                }
            }
            string[] files = Directory.GetFiles(varFromDirectory);
            if (files.Length > 0)
            {
                foreach (string s in files)
                {
                    File.Copy(s, varToDirectory + s.Substring(s.LastIndexOf("\\")), true);
                }
            }
        }
        #endregion

        #region 检查文件,如果文件不存在则创建
        /// <summary>
        /// 检查文件,如果文件不存在则创建  
        /// </summary>
        /// <param name="FilePath">路径,包括文件名</param>
        public static void ExistsFile(string FilePath)
        {
            //if(!File.Exists(FilePath))    
            //File.Create(FilePath);    
            //以上写法会报错,详细解释请看下文.........   
            if (!File.Exists(FilePath))
            {
                FileStream fs = File.Create(FilePath);
                fs.Close();
            }
        }
        #endregion

        #region 删除指定文件夹对应其他文件夹里的文件
        /// <summary>
        /// 删除指定文件夹对应其他文件夹里的文件
        /// </summary>
        /// <param name="varFromDirectory">指定文件夹路径</param>
        /// <param name="varToDirectory">对应其他文件夹路径</param>
        public static void DeleteFolderFiles(string varFromDirectory, string varToDirectory)
        {
            Directory.CreateDirectory(varToDirectory);

            if (!Directory.Exists(varFromDirectory)) return;

            string[] directories = Directory.GetDirectories(varFromDirectory);

            if (directories.Length > 0)
            {
                foreach (string d in directories)
                {
                    DeleteFolderFiles(d, varToDirectory + d.Substring(d.LastIndexOf("\\")));
                }
            }


            string[] files = Directory.GetFiles(varFromDirectory);

            if (files.Length > 0)
            {
                foreach (string s in files)
                {
                    File.Delete(varToDirectory + s.Substring(s.LastIndexOf("\\")));
                }
            }
        }
        #endregion

        #region 从文件的绝对路径中获取文件名( 包含扩展名 )
        /// <summary>
        /// 从文件的绝对路径中获取文件名( 包含扩展名 )
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>        
        public static string GetFileName(string filePath)
        {
            //获取文件的名称
            FileInfo fi = new FileInfo(filePath);
            return fi.Name;
        }
        #endregion

        /// <summary>
        /// 复制文件参考方法,页面中引用
        /// </summary>
        /// <param name="cDir">新路径</param>
        /// <param name="TempId">模板引擎替换编号</param>
        public static void CopyFiles(string cDir, string TempId)
        {
            //if (Directory.Exists(Request.PhysicalApplicationPath + "\\Controls"))
            //{
            //    string TempStr = string.Empty;
            //    StreamWriter sw;
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\Default.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\Default.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\Default.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\Column.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\Column.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\List.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\Content.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\Content.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\View.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\MoreDiss.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\MoreDiss.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\DissList.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\ShowDiss.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\ShowDiss.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\Diss.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\Review.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\Review.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\Review.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //    if (File.Exists(Request.PhysicalApplicationPath + "\\Controls\\Search.aspx"))
            //    {
            //        TempStr = File.ReadAllText(Request.PhysicalApplicationPath + "\\Controls\\Search.aspx");
            //        TempStr = TempStr.Replace("{$ChannelId$}", TempId);

            //        sw = new StreamWriter(Request.PhysicalApplicationPath + "\\" + cDir + "\\Search.aspx", false, System.Text.Encoding.GetEncoding("GB2312"));
            //        sw.Write(TempStr);
            //        sw.Close();
            //    }
            //}
        }

        #region 创建一个目录
        /// <summary>
        /// 创建一个目录
        /// </summary>
        /// <param name="directoryPath">目录的绝对路径</param>
        public static void CreateDirectory(string directoryPath)
        {
            //如果目录不存在则创建该目录
            if (!IsExistDirectory(directoryPath))
            {
                Directory.CreateDirectory(directoryPath);
            }
        }
        #endregion

        #region 创建一个文件
        /// <summary>
        /// 创建一个文件。
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>
        public static void CreateFile(string filePath)
        {
            try
            {
                //如果文件不存在则创建该文件
                if (!IsExistFile(filePath))
                {
                    //创建一个FileInfo对象
                    FileInfo file = new FileInfo(filePath);

                    //创建文件
                    FileStream fs = file.Create();

                    //关闭文件流
                    fs.Close();
                }
            }
            catch (Exception ex)
            {
                //LogHelper.WriteTraceLog(TraceLogLevel.Error, ex.Message);
                throw ex;
            }
        }

        /// <summary>
        /// 创建一个文件,并将字节流写入文件。
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>
        /// <param name="buffer">二进制流数据</param>
        public static void CreateFile(string filePath, byte[] buffer)
        {
            try
            {
                //如果文件不存在则创建该文件
                if (!IsExistFile(filePath))
                {
                    //创建一个FileInfo对象
                    FileInfo file = new FileInfo(filePath);

                    //创建文件
                    FileStream fs = file.Create();

                    //写入二进制流
                    fs.Write(buffer, 0, buffer.Length);

                    //关闭文件流
                    fs.Close();
                }
            }
            catch (Exception ex)
            {
                //LogHelper.WriteTraceLog(TraceLogLevel.Error, ex.Message);
                throw ex;
            }
        }
        #endregion

        #region 获取文本文件的行数
        /// <summary>
        /// 获取文本文件的行数
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>        
        public static int GetLineCount(string filePath)
        {
            //将文本文件的各行读到一个字符串数组中
            string[] rows = File.ReadAllLines(filePath);

            //返回行数
            return rows.Length;
        }
        #endregion

        #region 获取一个文件的长度
        /// <summary>
        /// 获取一个文件的长度,单位为Byte
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>        
        public static int GetFileSize(string filePath)
        {
            //创建一个文件对象
            FileInfo fi = new FileInfo(filePath);

            //获取文件的大小
            return (int)fi.Length;
        }
        #endregion

        #region 获取指定目录中的子目录列表
        /// <summary>
        /// 获取指定目录及子目录中所有子目录列表
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>
        /// <param name="searchPattern">模式字符串，"*"代表0或N个字符，"?"代表1个字符。
        /// 范例："Log*.xml"表示搜索所有以Log开头的Xml文件。</param>
        /// <param name="isSearchChild">是否搜索子目录</param>
        public static string[] GetDirectories(string directoryPath, string searchPattern, bool isSearchChild)
        {
            try
            {
                if (isSearchChild)
                {
                    return Directory.GetDirectories(directoryPath, searchPattern, SearchOption.AllDirectories);
                }
                else
                {
                    return Directory.GetDirectories(directoryPath, searchPattern, SearchOption.TopDirectoryOnly);
                }
            }
            catch (IOException ex)
            {
                throw ex;
            }
        }
        #endregion

        #region 向文本文件写入内容

        /// <summary>
        /// 向文本文件中写入内容
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>
        /// <param name="text">写入的内容</param>
        /// <param name="encoding">编码</param>
        public static void WriteText(string filePath, string text, Encoding encoding)
        {
            //向文件写入内容
            File.WriteAllText(filePath, text, encoding);
        }
        #endregion

        #region 向文本文件的尾部追加内容
        /// <summary>
        /// 向文本文件的尾部追加内容
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>
        /// <param name="content">写入的内容</param>
        public static void AppendText(string filePath, string content)
        {
            File.AppendAllText(filePath, content);
        }
        #endregion

        #region 将现有文件的内容复制到新文件中
        /// <summary>
        /// 将源文件的内容复制到目标文件中
        /// </summary>
        /// <param name="sourceFilePath">源文件的绝对路径</param>
        /// <param name="destFilePath">目标文件的绝对路径</param>
        public static void Copy(string sourceFilePath, string destFilePath)
        {
            File.Copy(sourceFilePath, destFilePath, true);
        }
        #endregion

        #region 将文件移动到指定目录
        /// <summary>
        /// 将文件移动到指定目录
        /// </summary>
        /// <param name="sourceFilePath">需要移动的源文件的绝对路径</param>
        /// <param name="descDirectoryPath">移动到的目录的绝对路径</param>
        public static void Move(string sourceFilePath, string descDirectoryPath)
        {
            //获取源文件的名称
            string sourceFileName = GetFileName(sourceFilePath);

            if (IsExistDirectory(descDirectoryPath))
            {
                //如果目标中存在同名文件,则删除
                if (IsExistFile(descDirectoryPath + "\\" + sourceFileName))
                {
                    DeleteFile(descDirectoryPath + "\\" + sourceFileName);
                }
                //将文件移动到指定目录
                File.Move(sourceFilePath, descDirectoryPath + "\\" + sourceFileName);
            }
        }
        #endregion

        #region 从文件的绝对路径中获取文件名( 不包含扩展名 )
        /// <summary>
        /// 从文件的绝对路径中获取文件名( 不包含扩展名 )
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>        
        public static string GetFileNameNoExtension(string filePath)
        {
            //获取文件的名称
            FileInfo fi = new FileInfo(filePath);
            return fi.Name.Split('.')[0];
        }
        #endregion

        #region 从文件的绝对路径中获取扩展名
        /// <summary>
        /// 从文件的绝对路径中获取扩展名
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>        
        public static string GetExtension(string filePath)
        {
            //获取文件的名称
            FileInfo fi = new FileInfo(filePath);
            return fi.Extension;
        }
        #endregion

        #region 清空指定目录
        /// <summary>
        /// 清空指定目录下所有文件及子目录,但该目录依然保存.
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>
        public static void ClearDirectory(string directoryPath)
        {
            if (IsExistDirectory(directoryPath))
            {
                //删除目录中所有的文件
                string[] fileNames = GetFileNames(directoryPath);
                for (int i = 0; i < fileNames.Length; i++)
                {
                    DeleteFile(fileNames[i]);
                }

                //删除目录中所有的子目录
                string[] directoryNames = GetDirectories(directoryPath);
                for (int i = 0; i < directoryNames.Length; i++)
                {
                    DeleteDirectory(directoryNames[i]);
                }
            }
        }
        #endregion

        #region 清空文件内容
        /// <summary>
        /// 清空文件内容
        /// </summary>
        /// <param name="filePath">文件的绝对路径</param>
        public static void ClearFile(string filePath)
        {
            //删除文件
            File.Delete(filePath);

            //重新创建该文件
            CreateFile(filePath);
        }
        #endregion

        #region 删除指定目录
        /// <summary>
        /// 删除指定目录及其所有子目录
        /// </summary>
        /// <param name="directoryPath">指定目录的绝对路径</param>
        public static void DeleteDirectory(string directoryPath)
        {
            if (IsExistDirectory(directoryPath))
            {
                Directory.Delete(directoryPath, true);
            }
        }
        #endregion
    }
}

```
