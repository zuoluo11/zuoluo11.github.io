---
title: .NET防止SQL注入
date: 2017-12-05 11:07:19
tags:
- ASP.NET
categories:
- C#基础
---

问题描述：测试人员通过IBM appscan工具进行网站系统扫描，发现SQL注入漏洞，
如何能进行全局性的修改，将所有请求的内容进行过滤？

解决办法：通过全局应用程序类，将所有的请求进行敏感字过滤。


{% asset_img Global.png Global Application Class 全局应用程序类 %}
{% asset_img SafeHelper.png 过滤敏感字符工具类 %}

<!--more-->

1、Global.asax
```C
using MxWeiXinPF.Common;
using System;

namespace MxWeiXinPF.Web
{
    public class Global : System.Web.HttpApplication
    {

        protected void Application_Start(object sender, EventArgs e)
        {

        }

        protected void Session_Start(object sender, EventArgs e)
        {

        }

        protected void Application_BeginRequest(object sender, EventArgs e)
        {
            if (Request.Cookies != null)
            {
                if (SafeHelper.CookieData())
                {
                    Response.Write("您提交的Cookie数据有恶意字符！");
                    Response.End();

                }


            }

            if (Request.UrlReferrer != null)
            {
                if (SafeHelper.referer())
                {
                    Response.Write("您提交的Referrer数据有恶意字符！");
                    Response.End();
                }
            }

            if (Request.RequestType.ToUpper() == "POST")
            {
                if (SafeHelper.PostData())
                {

                    Response.Write("您提交的Post数据有恶意字符！");
                    Response.End();
                }
            }
            if (Request.RequestType.ToUpper() == "GET")
            {
                if (SafeHelper.GetData())
                {
                    Response.Write("您提交的Get数据有恶意字符！");
                    Response.End();
                }
            }

        }

        protected void Application_AuthenticateRequest(object sender, EventArgs e)
        {

        }

        protected void Application_Error(object sender, EventArgs e)
        {

        }

        protected void Session_End(object sender, EventArgs e)
        {

        }

        protected void Application_End(object sender, EventArgs e)
        {

        }
        
    }
}


```

2、SafeHelper.cs
```C
using System.Text.RegularExpressions;
using System.Web;

namespace MxWeiXinPF.Common
{
    public class SafeHelper
    {
        private const string StrRegex = @"\b(alert|confirm|prompt)\b|^\+/v(8|9)|\b(and|or)\b.{1,6}?(=|>|<|\bin\b|\blike\b)|/\*.+?\*/|<\s*script\b|<\s*img\b|\bEXEC\b|UNION.+?SELECT|UPDATE.+?SET|INSERT\s+INTO.+?VALUES|(SELECT|DELETE).+?FROM|(CREATE|ALTER|DROP|TRUNCATE)\s+(TABLE|DATABASE)";

        public static bool PostData()
        {
            bool result = false;

            for (int i = 0; i < HttpContext.Current.Request.Form.Count; i++)
            {
                result = CheckData(HttpContext.Current.Request.Form[i].ToString());
                if (result)
                {
                    break;
                }
            }
            return result;
        }


        public static bool GetData()
        {
            bool result = false;

            for (int i = 0; i < HttpContext.Current.Request.QueryString.Count; i++)
            {
                result = CheckData(HttpContext.Current.Request.QueryString[i].ToString());
                if (result)
                {
                    break;
                }
            }
            return result;
        }


        public static bool CookieData()
        {
            bool result = false;
            for (int i = 0; i < HttpContext.Current.Request.Cookies.Count; i++)
            {
                result = CheckData(HttpContext.Current.Request.Cookies[i].Value.ToLower());
                if (result)
                {
                    break;
                }
            }
            return result;

        }
        public static bool referer()
        {
            bool result = false;
            return result = CheckData(HttpContext.Current.Request.UrlReferrer.ToString());
        }

        public static bool CheckData(string inputData)
        {
            if (Regex.IsMatch(inputData, StrRegex))
            {
                return true;
            }
            else
            {
                return false;
            }
        }
    }
}

```