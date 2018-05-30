---
title: ASP MVC 微信公众号通用授权
date: 2018-05-30 14:37:13
tags:
categories:
- 微信公众号
---

使用ASP MVC的注解方式，简易的将微信公众号内的实现网页自动登录。

<!--more-->


问题描述：在微信公众号增加网页菜单，实现用户微信号与系统自动绑定并登录的功能。
## 实现原理

所谓网站的自动登录，其实就是将用户微信的openid与系统的账号进行绑定，然后进行登录的过程。

### 微信接口介绍
在公众号下每一个用户都有独一无二的openid，因此可以使用这个唯一标识来进行登录权限的认证。
要获得用户的openid，需要使用微信提供的接口，即官网名称：[微信网页授权](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)

该授权的类型有两种：
第一种：静默跳转模式（scope的值是snsapi_base）
优点：可以在用户没有感觉的情况下获得openid
缺点：用户需要***关注微信公众号***且后续如果需要获得用户的信息需要调用[获取用户基本信息接口](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140839)

第二种：非静默的跳转模式（scope的值是snsapi_userinfo）
优点：两步合为一步，直接获得用户的信息，***用户可以不关注公众号***
缺点：用户会弹出页面提示是否需要授权（已关注用户从公众号的会话或者自定义菜单进入本公众号的网页授权页，则是静默授权）
{% asset_img 20180530161151.jpg 非静默授权 %}

### 实现代码原理
使用ASP MVC 中的注解模式，将需要授权的登录页Action增加注解实现自动登录。

1.创建授权注解类：将注解类新见到文件夹App_Start下并继承自父类AuthorizeAttribute；
2.登录Action上增加注解并在内部判断session中的openid是否存在来自动跳转登录页；
{% asset_img 20180530161842.jpg Action增加注解 %}

## 实现过程 
### 微信公众号后台需要设置 
{% asset_img 20180530162310.jpg 1 %}
{% asset_img 20180530162445.jpg 2 %}
{% asset_img sethost.png 3 %}
{% asset_img 20180530162838.jpg 4 %}
{% asset_img 20180530162916.jpg 5 %}
{% asset_img 20180530163018.jpg 6 %}
{% asset_img 20180530163137.jpg 7 %}

完成上面的步骤后，准备工作做完，开始进行编码。
### 代码实现 

#### 静默方式授权 

注解类
[代码参考地址](https://www.cnblogs.com/yangkangIT/p/5913524.html)
```C
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;
namespace Learun.Application.Web
{
    /// <summary>
    /// 微信授权
    /// </summary>
    public class WXFilterAttribute : AuthorizeAttribute
    {
        protected override bool AuthorizeCore(HttpContextBase httpContext)
        {
            httpContext.Session.Remove("weixinInfo");

            string userAgent = httpContext.Request.UserAgent;
            if (userAgent.IndexOf("MicroMessenger") <= -1)//不是微信浏览器
            {
                httpContext.Response.StatusCode = 400001;
                return false;
            }

            if (!httpContext.User.Identity.IsAuthenticated)
            {
                //ApplicationSignInManager SignInManager = httpContext.GetOwinContext().Get<ApplicationSignInManager>();
                //ApplicationUserManager UserManager = httpContext.GetOwinContext().GetUserManager<ApplicationUserManager>();
                string appid = string.Empty;
                string secret = string.Empty;

                appid ="**********";
                secret = "##################";


                var code = httpContext.Request["Code"];
                string returnUrl = HttpUtility.UrlDecode(httpContext.Request["ReturnUrl"] ?? "/");


                if (string.IsNullOrEmpty(code))
                {
                    string host = httpContext.Request.Url.Host;
                    string path = httpContext.Request.Path;
                    string redirectUrl = "http://" + host + path + "?ReturnUrl=" + HttpUtility.UrlEncode(returnUrl);//重定向的url，这里不需要进行编码，在后面会自己编码
                    try
                    {
                        //todo:通过微信获取2.0授权的url
                        string url = GetAuthorizeUrl(appid, redirectUrl, "state", "snsapi_base");

                        httpContext.Response.Redirect(url);
                    }
                    catch (System.Exception ex)
                    {
#if DEBUG
                        httpContext.Response.Write("构造网页授权获取code的URL时出错，错误是：" + ex.Message);
                        httpContext.Response.End();
#endif
                    }
                }
                else
                {
                    var client = new System.Net.WebClient();
                    client.Encoding = System.Text.Encoding.UTF8;
                    string url = GetAccessTokenUrl(appid, secret, code);
                    var data = client.DownloadString(url);
                    var obj = JsonConvert.DeserializeObject<Dictionary<string, string>>(data);
                    if (obj.ContainsKey("errcode") && obj["errcode"] == "40163")
                    {
                        //刷新页面,重新进入
                        httpContext.Response.Redirect("/SFJD_WeixinModule/WeiXinLogin/Login");
                        return true;
                    }
                    string accessToken;
                    if (!obj.TryGetValue("access_token", out accessToken))
                    {
#if DEBUG
                        httpContext.Response.Write("构造网页授权获取access_token的URL时出错");
                        httpContext.Response.End();
#endif
                    }
                    var openid = obj["openid"];
                    httpContext.Session["weixinInfo"] = openid;
                    //Utils.WidgetCode.ServerInfo.SetCookies("WXopenid", openid, DateTime.MinValue);

                    //var existUser = UserManager.Users.FirstOrDefault(p => p.OpenId == openid);
                    //if (existUser != null)
                    //{
                    //    SignInManager.SignInAsync(existUser, false, false);
                    //    httpContext.Response.Redirect(returnUrl);
                    //}
                }
            }
            return true;
        }
        public override void OnAuthorization(AuthorizationContext filterContext)
        {

            base.OnAuthorization(filterContext);
            if (filterContext.HttpContext.Response.StatusCode == 401)
            {
                filterContext.Result = new RedirectResult("/SFJD_WeixinModule/WeiXinLogin/Error?errorMsg=微信授权登录异常");

            }
            else if (filterContext.HttpContext.Response.StatusCode == 400001)
            {
                filterContext.Result = new RedirectResult("/SFJD_WeixinModule/WeiXinLogin/Error?errorMsg=请在微信中打开该页面");
            }

        }

        //扩展
        public string GetAuthorizeUrl(string appId, string redirectUrl, string state, string scope, string responseType = "code")
        {
            if (!string.IsNullOrEmpty(redirectUrl))
            {
                redirectUrl = HttpUtility.UrlEncode(redirectUrl, System.Text.Encoding.UTF8);
            }
            else
            {
                redirectUrl = null;
            }
            object[] args = new object[] { appId, redirectUrl, responseType, scope, state };
            return string.Format("https://open.weixin.qq.com/connect/oauth2/authorize?appid={0}&redirect_uri={1}&response_type={2}&scope={3}&state={4}#wechat_redirect", args);
        }
        public string GetAccessTokenUrl(string appId, string secret, string code, string grantType = "authorization_code")
        {
            object[] args = new object[] { appId, secret, code, grantType };
            string requestUri = string.Format("https://api.weixin.qq.com/sns/oauth2/access_token?appid={0}&secret={1}&code={2}&grant_type={3}", args);
            //return GetAccessTokenInfo(_httpClient.GetAsync(requestUri).Result.Content.ReadAsStringAsync().Result);
            return requestUri;
        }
    }
}

```
登录控制器

```C
   public class WeiXinLoginController : MvcWXControllerBase
    {
        // GET: SFJD_WeixinModule/WeiXinLogin

        #region 视图

        /// <summary>
        /// 登录
        /// </summary>
        /// <returns></returns>
        [WXFilter]
        public ActionResult Index()
        {
            string openID = GetOpenID();

            if (!string.IsNullOrEmpty(openID))
            {
                using (MAction action = new MAction(TableNames.SFJD_Client))
                {
                    if (action.Fill(string.Format("OpenID='{0}'", openID)))
                    {
                        //TODO:设置用户信息
                        //跳转首页
                        return RedirectToAction("Index", "WeiXinMain");
                    }
                }
            }
            //第一次登录去注册界面
            return View();
        }

        /// <summary>
        /// 获得当前OpenID
        /// </summary>
        /// <returns></returns>
        public string GetOpenID()
        {
            if (Session["weixinInfo"] == null)
            {
                return "";
            }
            else
            {
                return Convert.ToString(Session["weixinInfo"]);
            }
        }
    }
```

## 后续 ##

之前遇到了一个问题，在本地测试公众号下没有发现系统有什么异常，
发布到远程服务器及正式公众号下时，跳出异常信息“输入字符串的格式不正确。”，
找了半天认为应该是哪里转字符串的地方出问题了，可是一开始没找到。
最后使用日志输出的方式将问题定位到了获得用户基本信息后，放置token过期时间上。
```
异常信息{"errcode":40164,"errmsg":"invalid ip 111.111.112.117, not in whitelist hint: [yYexhA05332994]"}
```
发现公众号未设置白名单导致。

在IP白名单内的IP来源，获取access_token接口才可调用成功。

设置完成后，问题解决。

获得微信用户信息的工具类
```c
using CYQ.Data.Cache;
using CYQ.Data.Tool;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Util.WeiXin
{
    /// <summary>
    /// 微信公众号工具类
    /// </summary>
    public class WeiXinTool
    {
        /// <summary>
        /// 获得已关注公众号用户信息
        /// </summary>
        /// <param name="openid"></param>
        /// <param name="appID"></param>
        /// <param name="appSecret"></param>
        /// <returns></returns>
        public static string GetVisiterInfo(string openid)
        {
            string appID = "#########";
            string appSecret = "******************";

            string access_token = GetAccess_token(appID, appSecret);
            string requestURL = "https://api.weixin.qq.com/cgi-bin/user/info?access_token={0}&openid={1}&lang=zh_CN ";
            string strJson = HttpRequestUtil.RequestUrl(string.Format(requestURL,
                 access_token, openid));
            return strJson;
        }
        /// <summary>
        /// 获得基础接口的access_token
        /// </summary>
        /// <param name="appID"></param>
        /// <param name="appSecret"></param>
        /// <returns></returns>
        public static string GetAccess_token(string appID, string appSecret)
        {
            string access_token = "";
            CacheManage cache = CacheManage.Instance;
            if (cache.Contains("access_token" + appID))
            {
                access_token = cache.Get("access_token" + appID) as string;
            }
            else
            {
                string strJson = "";
                string requestURL = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid={0}&secret={1}";
                strJson = HttpRequestUtil.RequestUrl(string.Format(requestURL,
                  appID, appSecret));
              
                access_token = JsonHelper.GetValue(strJson, "access_token");
                double cacheMinutes = (double)(Convert.ToInt32(JsonHelper.GetValue(strJson, "expires_in")) / 60);//将秒转换为分钟
                cache.Set("access_token" + appID, access_token, cacheMinutes);
            }
            return access_token;
        }
    }


}


```

