---
title: 'RSA加密解密(c#与js版)'
date: 2017-11-29 10:41:20
tags:
- 加密解密
categories:
- C#基础
+description: "为保证网络中数据安全进行RSA加密解密"
---

业务描述：前端js使用公有秘钥将数据进行加密，后端C#在一般处理程序中使用私钥对加密数据进行解密。
两个版本的实现，版本一可直接使用，版本二可用来参考如何将XML秘钥与PEM秘钥的转换。

版本一描述：
1、该版本实现用户登录过程中RSA加密验证的过程。
2、前台JS进行加密，后台从session中获得私钥进行解密。
3、RSA秘钥保存在session中，刷新登录页面后自动更新。

版本二描述：
1、C#自有类RSACryptoServiceProvider生成RSA的秘钥与公钥的方法ToXmlString(true)秘钥，ToXmlString(false)公钥，结果是XML形式的，js无法直接使用，js所使用的是PEM形式的秘钥。
解决方法：使用工具类bouncycastle直接生成PEM版的秘钥和公钥，后台使用时，再将秘钥转换为XML版的。
引用地址：https://www.bouncycastle.org/
插件下载地址：https://downloads.bouncycastle.org/csharp/bccrypto-csharp-1.8.1-bin.zip


2、js加密数据后base64位的数据在传输中“+”号会变为空格，导致后台C#解密出错。
解决方法：js先加号转后，然后后台再将加号转回。

3、网页版的加密解密demo以固定的秘钥形式来实现，实际案例中可以考虑更改为使用session保存针对单个
用户来生成的公钥和秘钥。

参考地址：
http://www.sufeinet.com/forum.php?mod=viewthread&tid=5484&highlight=RSA
https://www.cnblogs.com/Leo_wl/p/5763243.html


------

<!--more-->

### 控制台应用程序RSA基础代码演示
```C 
using Org.BouncyCastle.Asn1.Pkcs;
using Org.BouncyCastle.Asn1.X509;
using Org.BouncyCastle.Crypto;
using Org.BouncyCastle.Crypto.Generators;
using Org.BouncyCastle.Crypto.Parameters;
using Org.BouncyCastle.Pkcs;
using Org.BouncyCastle.Security;
using Org.BouncyCastle.X509;
using System;
using System.Security.Cryptography;
using System.Text;

namespace RSAGenerateKey
{
    class Program
    {
        static void Main(string[] args)
        {
            RsaKeyPairGenerator g = new RsaKeyPairGenerator();
            g.Init(new KeyGenerationParameters(new SecureRandom(), 1024));
            var pair = g.GenerateKeyPair();
            PrivateKeyInfo privateKeyInfo = PrivateKeyInfoFactory.CreatePrivateKeyInfo(pair.Private);
            byte[] serializedPrivateBytes = privateKeyInfo.ToAsn1Object().GetDerEncoded();
            string serializedPrivate = Convert.ToBase64String(serializedPrivateBytes);//PEM秘钥

            SubjectPublicKeyInfo publicKeyInfo = SubjectPublicKeyInfoFactory.CreateSubjectPublicKeyInfo(pair.Public);
            byte[] serializedPublicBytes = publicKeyInfo.ToAsn1Object().GetDerEncoded();
            string serializedPublic = Convert.ToBase64String(serializedPublicBytes);//PEM公钥

            RsaPrivateCrtKeyParameters privateKey = (RsaPrivateCrtKeyParameters)PrivateKeyFactory.CreateKey(Convert.FromBase64String(serializedPrivate));
            RsaKeyParameters publicKey = (RsaKeyParameters)PublicKeyFactory.CreateKey(Convert.FromBase64String(serializedPublic));


            RSACryptoServiceProvider rcsp = new RSACryptoServiceProvider();
            RSAParameters parms = new RSAParameters();

            //So the thing changed is offcourse the ToByteArrayUnsigned() instead of
            //ToByteArray()
            parms.Modulus = privateKey.Modulus.ToByteArrayUnsigned();
            parms.P = privateKey.P.ToByteArrayUnsigned();
            parms.Q = privateKey.Q.ToByteArrayUnsigned();
            parms.DP = privateKey.DP.ToByteArrayUnsigned();
            parms.DQ = privateKey.DQ.ToByteArrayUnsigned();
            parms.InverseQ = privateKey.QInv.ToByteArrayUnsigned();
            parms.D = privateKey.Exponent.ToByteArrayUnsigned();
            parms.Exponent = privateKey.PublicExponent.ToByteArrayUnsigned();

            //So now this now appears to work.
            rcsp.ImportParameters(parms);


            string s = rcsp.ToXmlString(true);

            string privateKeyXmlText = rcsp.ToXmlString(true);//XML秘钥
            string publicKeyXmlText = rcsp.ToXmlString(false);//XML公钥

            //加密解密
            string texta1 = "abc",texta2="",textb1="";
            byte[] cipherbytes;

            cipherbytes = rcsp.Encrypt(Encoding.UTF8.GetBytes(texta1), false);

            texta2 = Convert.ToBase64String(cipherbytes);

            cipherbytes = rcsp.Decrypt(Convert.FromBase64String(texta2), false);

            textb1= Encoding.UTF8.GetString(cipherbytes);

            Console.Read();

        }
    }
}
```




### 版本一
> 版本一： 直接使用RSACryptoServiceProvider类生成xml的公钥和秘钥，放入session中
该版本用到了四个js。下载地址：https://pan.baidu.com/s/1boQox8V
```html
<script src="Scripts/jquery-1.4.1.js"></script>
<script src="Scripts/Barrett.js"></script>
<script src="Scripts/BigInt.js"></script>
<script src="Scripts/RSA.js"></script>
```

#### 前台代码
```html
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Login.aspx.cs" Inherits="RSAWeb.Login" %>

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>登录页RSA加密解密demo</title>
    <script src="Scripts/jquery-1.4.1.js"></script>
    <script src="Scripts/Barrett.js"></script>
    <script src="Scripts/BigInt.js"></script>
    <script src="Scripts/RSA.js"></script>
     <script type="text/javascript">
        function cmdEncrypt() {
            //关键步骤
            setMaxDigits(129);
            var key = new RSAKeyPair("<%=strPublicKeyExponent%>", "", "<%=strPublicKeyModulus%>");
            var pwdMD5Twice = $("#txtPassword").attr("value");
            var pwdRtn = encryptedString(key, pwdMD5Twice);
            //关键步骤
            $("#encrypted_pwd").attr("value", pwdRtn);
            $("#formLogin").submit();
            return;
        }
    </script>
</head>
<body>
    <form action="Login.aspx" id="formLogin" method="post">
    <div>
        <div>
            User Name:
        </div>
        <div>
            <input id="txtUserName" name="txtUserName" value="<%=postbackUserName%>" type="text" maxlength="16" />
        </div>
        <div>
            Password:
        </div>
        <div>
            <input id="txtPassword" type="password" maxlength="16" />
        </div>
        <div>
            <input id="btnLogin" type="button" value="Login" onclick="return cmdEncrypt()" />
        </div>
    </div>
    <div>
        <input type="hidden" name="encrypted_pwd" id="encrypted_pwd" />
    </div>
    </form>
    <div>
        <%=LoginResult%>
    </div>
</body>
</html>

```

#### 后台代码
```C
using System;
using System.Security.Cryptography;
using System.Text;

namespace RSAWeb
{
    public partial class Login : System.Web.UI.Page
    {
        protected string strPublicKeyExponent = "";
        protected string strPublicKeyModulus = "";
        protected string LoginResult = "";
        protected string postbackUserName = "";


        protected void Page_Load(object sender, EventArgs e)
        {
            LoginResult = "";
            RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();
            if (string.Compare(Request.RequestType, "get", true) == 0)
            {
                //将私钥存Session中
                Session["private_key"] = rsa.ToXmlString(true);
                CYQ.Data.Log.WriteLogToTxt(Convert.ToString(Session["private_key"]), CYQ.Data.LogType.Debug);
            }
            else
            {
                bool bLoginSucceed = false;
                try
                {
                    string strUserName = Request.Form["txtUserName"];
                    postbackUserName = strUserName;
                    string strPwdToDecrypt = Request.Form["encrypted_pwd"];
                    rsa.FromXmlString((string)Session["private_key"]);

                    byte[] result = rsa.Decrypt(HexStringToBytes(strPwdToDecrypt), false);
                    System.Text.ASCIIEncoding enc = new ASCIIEncoding();
                    string strPwdMD5 = enc.GetString(result);
                    if (string.Compare(strUserName, "admin", true) == 0 && string.Compare(strPwdMD5, "admin", true) == 0)//14e1b600b1fd579f47433b88e8d85291
                        bLoginSucceed = true;
                }
                catch (Exception)
                {

                }
                if (bLoginSucceed)
                    LoginResult = "登录成功";
                else
                    LoginResult = "登录失败";
            }

            //把公钥适当转换，准备发往客户端
            RSAParameters parameter = rsa.ExportParameters(true);
            strPublicKeyExponent = BytesToHexString(parameter.Exponent);
            strPublicKeyModulus = BytesToHexString(parameter.Modulus);
        }
        private string BytesToHexString(byte[] input)
        {
            StringBuilder hexString = new StringBuilder(64);

            for (int i = 0; i < input.Length; i++)
            {
                hexString.Append(String.Format("{0:X2}", input[i]));
            }
            return hexString.ToString();
        }

        public static byte[] HexStringToBytes(string hex)
        {
            if (hex.Length == 0)
            {
                return new byte[] { 0 };
            }

            if (hex.Length % 2 == 1)
            {
                hex = "0" + hex;
            }

            byte[] result = new byte[hex.Length / 2];

            for (int i = 0; i < hex.Length / 2; i++)
            {
                result[i] = byte.Parse(hex.Substring(2 * i, 2), System.Globalization.NumberStyles.AllowHexSpecifier);
            }

            return result;
        }

    }
}
```



### 版本二
> 版本二： 未能解决问题：在使用过程中偶尔会出现无法解密的异常，
用户
a8176a4c-5b7e-4f1d-94d8-04e744332f76
进行分数
/H544nYvvBRPZQAIQfDGHMySy/svCbS8/uUwwv5Fc6hKlTed9XvwuEYeAKv22cdPXDV%2B6/
tmduEuVJDCP7G8Jb2TxJdN9A%2Bwou%2BGnOTO7%2BTo6yA2KX4Uvriof5yCt1ONmkLbsPRVh0/cnjnogJwLk2U/FVggjIhcOP6Rq%2BzLOw==
的提交操作,发生异常不正确的数据。
本地直接使用测试demo可以解密，所以无法找到出问题的原因。！！！！

2、网页版本webform

#### 前台网页
```html
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="login.aspx.cs" Inherits="RSADemo.login" %>

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title></title>
    <script src="js/jquery.js"></script>
    <script src="http://passport.cnblogs.com/scripts/jsencrypt.min.js"></script>
    <script src="https://unpkg.com/node-forge@0.7.0/dist/forge.min.js"></script>
    <script type="text/javascript">
        $(function () {
             //pem公钥
            var pemPublicKey = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDHCkaSCVy6sHB/5rAKS/1EEtyzWuy30gLyrpNbFI3GtpsdFGdsqQ/uwiscGD+pZ7Mxj1ZumPs4jHvPpcAeCb8gKsqP/f5+pputTMuTkhTQqlDT1plHR7w3REQI8MaJ8KTA/pJiPo6iWToFynQeJNWjicxXxNURSZQ7nmC2rl4uPQIDAQAB";
            $("#tra").val(pemPublicKey);


            var encrypt = new JSEncrypt();
            encrypt.setPublicKey($("#tra").val());
            var data = encrypt.encrypt("123456789");
            alert(data);
            $("#btn").click(function () {
                $.ajax({
                    url: 'Handler1.ashx',
                    data: "pwd=" +encodeURI(data).replace(/\+/g, '%2B'),  //+号的处理：因为数据在网络上传输时，非字母数字字符都将被替换成百分号（%）后跟两位十六进制数，而base64编码在传输到后端的时候，+会变成空格，因此先替换掉。后端再替换回来
                    type: 'post',
                    success: function (msg) {
                        alert(msg);
                    }
                });
            });

        });


    </script>
</head>
<body>
    <form id="form1" runat="server">
        <div>
            <input type="button" id="btn" value="点我" />
            <textarea id="tra" rows="15" cols="65">
                
        </textarea>
            <hr />
            注意+好的处理
        </div>
    </form>
</body>
</html>

```

#### 后台ashx一般处理程序

```C
using System;
using System.Security.Cryptography;
using System.Text;
using System.Web;

namespace RSADemo
{
    /// <summary>
    /// Summary description for Handler1
    /// </summary>
    public class Handler1 : IHttpHandler
    {

        public void ProcessRequest(HttpContext context)
        {
            string result = "";
            //XML密钥
            string privateKey = @"<RSAKeyValue><Modulus>xwpGkglcurBwf+awCkv9RBLcs1rst9IC8q6TWxSNxrabHRRnbKkP7sIrHBg/qWezMY9Wbpj7OIx7z6XAHgm/ICrKj/3+fqabrUzLk5IU0KpQ09aZR0e8N0RECPDGifCkwP6SYj6Oolk6Bcp0HiTVo4nMV8TVEUmUO55gtq5eLj0=</Modulus><Exponent>AQAB</Exponent><P>/LnrFrusoNklOl6d0zSWZ1aCdQC2l3XXU8SSgYNLuSmgVwl7wQ2w0Jn9SQqyiVRmbcp1SX28/bH6TaA2v9ejHQ==</P><Q>yZ5TTtLOfTqrikYjy/fyktTk977y2GG2R9sgNPHtnH5EIIC9CoJETDwfSu40YUlHeUUXHQ1nG1WmWbEOC76toQ==</Q><DP>5UfC+YfgkLkQJklqxA+EmFIK3x17iiO16+B9zhQQ4fba6bvH05iZHldmTBrxaNfyaY7xI3B4wmzymfRNV3TKHQ==</DP><DQ>k7EvRaaXLJU14+zNfDT9tSHPOMzgCDJL3Qdf6GjwrpqwPT8RPAmBDndcVP95z2pmuScrb1TKGvP7D+jraR8dAQ==</DQ><InverseQ>4gbDpW7ca3dn0XXPkYsVmIl7SBqU8lq9X2xji/Nyg1M0pjDcpdQm0bqOm+/5usQl+kRotpIoK+Yf6J++zbmNjg==</InverseQ><D>Ccxfc356/mTDsQQv+93ISsLb8wdhml4AD6bY8bWmEhd4tNqFieObuW79FM27ypDkkSbDhD/LNDo0OSFpfwEPU8VxnEMzFnVw7MIWGSVKWocZHIhsclkHtHNtHaKS0LNEie2q0PGMiIYty/QG5k3bJeA8R42teXv3nARYEgzuNmE=</D></RSAKeyValue>";

       
            try
            {
                string pwd = context.Request["pwd"];

                RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();
                byte[] cipherbytes;
                rsa.FromXmlString(privateKey);
                //把+号，再替换回来
                cipherbytes = rsa.Decrypt(Convert.FromBase64String(pwd.Replace("%2B", "+")), false);

                result = Encoding.UTF8.GetString(cipherbytes);
            }
            catch (Exception exception)
            {


            }

         

            context.Response.Write(result);
        }
     
        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}

```

