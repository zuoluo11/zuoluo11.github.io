---
title: 动态调用webservice
date: 2016-12-24 10:20:50
tags:
- c#基础
categories:
- 插件
+description: "动态调用webservice demo"
---
 关于webservice的调用方法
 之前使用或了解的只是通过vs的操作界面直接引入web服务而调用的，现在才知道调用的方式也可以使用反射的机制来实现，
 网上找到的代码本地测试可以使用，留存。

<!--more-->
```C
using System;
using System.Web.Services.Description;
using System.CodeDom;
using Microsoft.CSharp;
using System.CodeDom.Compiler;
using System.Net;
using System.IO;

namespace DirectInvoke
{
    class Program
    {
        static void Main(string[] args)
        {
            string serviceUrl = "http://www.webxml.com.cn/WebServices/ChinaZipSearchWebService.asmx";
            object o = Webservice.InvokeWebService(serviceUrl, "getSupportProvince", new object[] { });
        }
        //反射获得方法及成员
        public void PrintInstanceInfor(Type t)
        {
            //获取所有方法 
            System.Reflection.MethodInfo[] methods = t.GetMethods();

            //获取所有成员 
            System.Reflection.MemberInfo[] members = t.GetMembers();

            //获取所有属性 
            System.Reflection.PropertyInfo[] properties = t.GetProperties();
        }
    }
    public class Webservice
    {
        /// <summary>
        /// 实例化WebServices
        /// </summary>
        /// <param name="url">WebServices地址</param>
        /// <param name="methodname">调用的方法</param>
        /// <param name="args">把webservices里需要的参数按顺序放到这个object[]里</param>
        public static object InvokeWebService(string url, string methodname, object[] args)
        {

            //这里的namespace是需引用的webservices的命名空间，我没有改过，也可以使用。也可以加一个参数从外面传进来。
            string @namespace = "client";
            try
            {
                //获取WSDL
                WebClient wc = new WebClient();
                Stream stream = wc.OpenRead(url + "?WSDL");
                ServiceDescription sd = ServiceDescription.Read(stream);
                string classname = sd.Services[0].Name;
                ServiceDescriptionImporter sdi = new ServiceDescriptionImporter();
                sdi.AddServiceDescription(sd, "", "");
                CodeNamespace cn = new CodeNamespace(@namespace);

                //生成客户端代理类代码
                CodeCompileUnit ccu = new CodeCompileUnit();
                ccu.Namespaces.Add(cn);
                sdi.Import(cn, ccu);
                CSharpCodeProvider csc = new CSharpCodeProvider();
                //ICodeCompiler icc = csc.CreateCompiler();

                //设定编译参数
                CompilerParameters cplist = new CompilerParameters();
                cplist.GenerateExecutable = false;
                cplist.GenerateInMemory = true;
                cplist.ReferencedAssemblies.Add("System.dll");
                cplist.ReferencedAssemblies.Add("System.xml.dll");
                cplist.ReferencedAssemblies.Add("System.Web.Services.dll");
                cplist.ReferencedAssemblies.Add("System.Data.dll");

                //编译代理类
                CompilerResults cr = csc.CompileAssemblyFromDom(cplist, ccu);
                if (true == cr.Errors.HasErrors)
                {
                    System.Text.StringBuilder sb = new System.Text.StringBuilder();
                    foreach (System.CodeDom.Compiler.CompilerError ce in cr.Errors)
                    {
                        sb.Append(ce.ToString());
                        sb.Append(System.Environment.NewLine);
                    }
                    throw new Exception(sb.ToString());
                }

                //生成代理实例，并调用方法
                System.Reflection.Assembly assembly = cr.CompiledAssembly;
                Type t = assembly.GetType(@namespace + "." + classname, true, true);
                object obj = Activator.CreateInstance(t);
                System.Reflection.MethodInfo mi = t.GetMethod(methodname);

                return mi.Invoke(obj, args);
            }
            catch
            {
                return null;
            }
        }
    }


}
```
[核心代码转自](http://www.knowsky.com/898239.html)