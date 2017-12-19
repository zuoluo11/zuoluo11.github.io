---
title: 去除微信昵称中emoji字符
date: 2017-12-18 17:30:27
categories:
- 微信公众号
---
问题描述
1、微信公众号获得用户基础信息时，用户的昵称nickname中可能包含emoji表情，而该表情在保存数据库之后在前台页面中
使用（导出excel）会导致异常，因此需要该emoji表情去掉。
解决方法：替换所有表情字符。

### [参考地址](http://www.howtobuildsoftware.com/index.php/how-do/bouX/c-mysql-unicode-emoji-how-do-i-remove-emoji-characters-from-a-string)
{% asset_img 20171218174151.png 来源 %}
```C#
class Program
    {
        static void Main(string[] args)
        {

             string nickName = "特殊字符串";
             nickName = Regex.Replace(nickName, @"\p{Cs}", "*");
             Console.WriteLine(nickName);
             Console.ReadKey();
        }
    }
```