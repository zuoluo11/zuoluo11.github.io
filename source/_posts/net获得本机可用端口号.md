---
title: .net获得本机可用端口号
date: 2017-06-19 10:04:06
tags:
- ASP.Net
categories:
- C#基础
---
好久没有总结文字了，上周整理过一个IIS自动发布并绑定域名的demo，
其中有一部分需要读取未使用的服务器端口号用来发布网站，因此
整理出该方法。
实现逻辑思路：.net 通过命令提示符窗口获得当前已用端口，再筛选出可用端口。
<!--more-->
## 公用方法代码
``` bash
public string GetFreePort()
        {
            string result = "";
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
            p.StandardInput.WriteLine("exit"); //此处必须加退出命令，否则界面会卡死在cmd窗口无法继续执行
            string str;
            StreamReader reader=p.StandardOutput;
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
                    ports.Add(port);
                }
                Console.WriteLine(str);
                str = reader.ReadLine();
            }
            ////可用端口号范围1024－49151
            for (int i = 1024; i < 49151; i++)
            {
                if (!ports.Contains(i))
                {
                    result = i.ToString();
                    break;
                }
            }
            return result;
        }
```
