---
title: xml序列化
date: 2017-08-16 20:51:17
categories:
- C#基础
---

  可能目前系统之间接口多数以JSON或者XML形式进行数据的传输，
项目中用到了XML，但是没有使用下面介绍的以序列化的形式将XML与对象
进行互转，当时为了多个实现方案所以做了一些准备，
先总结XML序列化与反序列化的使用基础，备用吧。



> * 基础类及基础方法
> * 代码实现
> * 总结及出现的问题

<!--more-->
### 基础类及基础方法
   基础类主要是：[System.Xml.Serialization.XmlSerializer][XmlSerializer]
XmlSerializer类主要功能是将对象和XML文档之间进行序列化和发序列化。
XmlSerializer可以方便将对象编码为XML。

1、构造器创建XmlSerializer对象
``` bash
	//创建新实例并指定改实例可以用来序列化和反序列化的类的类型
	XmlSerializer xmlSerializer=new XmlSerializer(Type);
```
2、XmlSerializer对象序列化对象
``` bash
	//将对象Object序列化到TextWriter中
	xmlSerializer.Serialize(TextWriter,Object);
```
TextWriter是一个有序序列字符的编写器，它是一个抽象类，常用的有
两个子类即：StreamWriter（写入流） 和 StringWriter（写入字符串）.
因此在实际使用时，这里选择[System.IO.StringWriter][StringWriter]。

这里需要注意，如果要指定序列化后XML的编码类型，StringWriter有一个属性[Encoding][Encoding],
但是该属性只读，无法进行设置，因此需要写一个子类来覆盖Encoding的属性。

``` bash
	public sealed class Utf8StringWriter : StringWriter
    {
    	//指定编码类型是UTF-8
        public override Encoding Encoding { get { return Encoding.UTF8; } }
    }


    //将对象Object序列化到TextWriter的子类Utf8StringWriter中
	xmlSerializer.Serialize(Utf8StringWriter,Object);
```
3、获得序列化后的结果
``` bash
	string XMLResult=Utf8StringWriter.ToString();
```

4、通过特性Attritube设置自定义类的序列化结果
[System.Xml.Serialization][Serialization]命名空间中有一系列的特性类，用来控制复杂类型序列化的控制。例如XmlElementAttribute、XmlAttributeAttribute、XmlArrayAttribute、XmlArrayItemAttribute、XmlRootAttribute等等。
[序列化帮助][序列化帮助]

### 代码实现
完整代码如下：
```C
using System;
using System.IO;
using System.Reflection;
using System.Text;
using System.Xml;
using System.Xml.Serialization;

namespace Test
{
    class Program
    {
       

        static void Main(string[] args)
        {
            SERVICE service = new SERVICE();
            HEAD head = service.HEAD;
            head.nsrsbh = "140115728183815";
            head.serviceversion = "1.3";
            head.serviceid = "jy.dzptfpkj";
            head.iszip = "N";
            head.encryptcode = "0";
            head.issyn = "Y";
            service.Cmd = new Cmd[] {new Cmd(),new Cmd() };

            //序列化这个对象
            XmlSerializer serializer = new XmlSerializer(service.GetType());
            
            using (Utf8StringWriter writer = new Utf8StringWriter())
            {
                serializer.Serialize(writer, service);
                Console.WriteLine(writer.ToString());
            }  

            Console.Read();
        }
    }


    public class BaseObject
    {
        public BaseObject()
        {
            Type t = this.GetType();//构造器获得当前类型
            PropertyInfo[] list=t.GetProperties();//获得所有属性

            foreach (PropertyInfo item in list)
            {
                object[] objAttrs = item.GetCustomAttributes(typeof(XmlElementAttribute), true);//获得所有XmlElement特性
                if (objAttrs != null && objAttrs.Length > 0)
                {
                    XmlElementAttribute attr = objAttrs[0] as XmlElementAttribute;
                    //Console.WriteLine(attr.ElementName);
                    attr.IsNullable = true;//将特性设置为可空
                    
                }

                if (IsType(item.PropertyType, "System.String"))
                {
                    item.SetValue(this, string.Empty, null);//将字符串类型属性的值设置为空字符串
                }
            }
        }
        /// <summary>
        /// 类型匹配
        /// </summary>
        /// <param name="type"></param>
        /// <param name="typeName"></param>
        /// <returns></returns>
        public static bool IsType(Type type, string typeName)
        {
            if (type.ToString() == typeName)
                return true;
            if (type.ToString() == "System.Object")
                return false;

            return IsType(type.BaseType, typeName);
        }
    }

    [XmlRoot("SERVICE")]
    public class SERVICE: BaseObject
    {
        public SERVICE() {
            this.HEAD = new HEAD();
        }
        [XmlElement("HEAD")]
        public HEAD HEAD { get; set; }

        [XmlArray("BODY")]
        [XmlArrayItem("cmd")]
        public Cmd[] Cmd { get; set; }

       
    }

    public class HEAD : BaseObject
    {
        public HEAD()
        {
            this.RTNINF = new RTNINF();
        }
        [XmlElement("nsrsbh")]
        public string nsrsbh { get; set; }
        [XmlElement("serviceversion")]
        public string serviceversion { get; set; }
        [XmlElement("serviceid")]
        public string serviceid { get; set; }
        [XmlElement("iszip")]
        public string iszip { get; set; }
        [XmlElement("encryptcode")]
        public string encryptcode { get; set; }
        [XmlElement("issyn")]
        public string issyn { get; set; }
        [XmlElement("RTNINF")]
        public RTNINF RTNINF { get; set; }
    }

    public class RTNINF : BaseObject
    {
        [XmlElement("rtn_code")]
        public string rtn_code { get; set; }
        [XmlElement("rtn_reason")]
        public string rtn_reason { get; set; }
    }
    public sealed class Utf8StringWriter : StringWriter
    {
        public override Encoding Encoding { get { return Encoding.UTF8; } }
    }
    
    public class Cmd : BaseObject
    {
        [XmlElement("productid")]
        public string ProductId { get; set; }

        [XmlElement("price")]
        public string Price { get; set; }

        [XmlElement("date")]
        public string Date { get; set; }

        [XmlElement("state")]
        public string State { get; set; }

        [XmlElement("type")]
        public string Type { get; set; }
    }
   
}

```

### 总结问题
1、项目需要，所有的对象属性都要生成XML，即如果属性是空值，默认序列化XML后
不会显示，所以把所有对象都继承自BaseObject类下，再使用BaseObject类的构造器
通过反射子类中特性的方法，在初始化子类时，将空字符串写入，这样最后生成XML
文件时，空字符串的属性也会显示了。

反射时使用了的类及方法包括Object.GetType()根据对象获得类型,Type.GetProperties()根据类型获得说有属性,
PropertyInfo.GetCustomAttributes根据属性获得所有特性，
父类BaseObject 代码如下：

```C
	    public class BaseObject
    {
        public BaseObject()
        {
            Type t = this.GetType();//构造器获得当前类型
            PropertyInfo[] list=t.GetProperties();//反射获得所有属性

            foreach (PropertyInfo item in list)
            {
                object[] objAttrs = item.GetCustomAttributes(typeof(XmlElementAttribute), true);//获得所有XmlElement特性
                if (objAttrs != null && objAttrs.Length > 0)
                {
                    XmlElementAttribute attr = objAttrs[0] as XmlElementAttribute;
                    //Console.WriteLine(attr.ElementName);
                    attr.IsNullable = true;//将特性设置为可空
                    
                }

                if (IsType(item.PropertyType, "System.String"))
                {
                    item.SetValue(this, string.Empty, null);//将字符串类型属性的值设置为空字符串
                }
            }
        }
        /// <summary>
        /// 类型匹配
        /// </summary>
        /// <param name="type"></param>
        /// <param name="typeName"></param>
        /// <returns></returns>
        public static bool IsType(Type type, string typeName)
        {
            if (type.ToString() == typeName)
                return true;
            if (type.ToString() == "System.Object")
                return false;

            return IsType(type.BaseType, typeName);
        }
    }


```
 2、SERVICE类下的Cmd属性比较特殊，它是一个数组，通过XmlArray特性和XmlArrayItem特性配合使用后，如下：
    	 [XmlArray("BODY")]
        [XmlArrayItem("cmd")]
        public Cmd[] Cmd { get; set; }
        序列化XML后生成的效果如下：


``` bash
	<BODY>
		<cmd></cmd>
		<cmd></cmd>
		<cmd></cmd>
		<cmd></cmd>
	</BODY>
```

[XmlSerializer]:https://msdn.microsoft.com/zh-cn/library/system.xml.serialization.xmlserializer(v=vs.110).aspx
[StringWriter]:https://msdn.microsoft.com/zh-cn/library/system.io.stringwriter(v=vs.110).aspx
[Encoding]:https://msdn.microsoft.com/zh-cn/library/system.io.stringwriter.encoding(v=vs.110).aspx
[Serialization]:https://msdn.microsoft.com/zh-cn/library/system.xml.serialization(v=vs.110).aspx
[序列化帮助]:https://msdn.microsoft.com/zh-cn/library/2baksw0z(v=vs.110).aspx

