---
title: 'c#基础-扩展方法'
date: 2016-09-12 22:58:48
categories: 
- C#基础
tags:
- 扩展方法

---
## 扩展方法的讲解
C#扩展方法，爱你在心口难开
转自[无风听海](http://www.cnblogs.com/wufengtinghai/archive/2011/08/05/2128110.html) 

<!--more-->
什么是扩展方法?好几天了打算记录一下，今天我们来深入研究一下，探究一下扩展方法的实现机制；那么到底什么是扩展方法呢？
扩展方法使您能够向现有类型“添加”方法，而无需创建新的派生类型、重新编译或以其他方式修改原始类型。扩展方法是一种特殊的静态方法，但可以像扩展类型上的实例方法一样进行调用。对于用 C# 和 Visual Basic 编写的客户端代码，调用扩展方法与调用在类型中实际定义的方法之间没有明显的差异。
也许你并不明白以上的意思，那一点都没有关系，如果我们平时一定经常使用linq标准查询，那么我们就一直在使用扩展方法啦！
微软为枚举的集合扩展了很多的标准查询方法，极大的方便了我们的使用！请看下面的例子没有使用扩展方法

   
	
	``` bash
using System;
using System.Collections.Generic;
using System.Text;
using wuFengTingHai.Person;

namespace ExtendMethod
{
    public class LinqExtend
    {
        private IList<Person> persons = new List<Person>();
        public IList<Person> Persons
        {
            get {
                //删除集合中名称为无风听海的记录，没有引入system.linq,所以不能使用扩展方法
                foreach(Person person in this.persons)
                {
                    if (person.Name.Equals("无风听海"))
                    {
                        this.persons.Remove(person);
                    }
                }                
                return this.persons;
            }
            set { this.persons = value; }
        }        
    }   
}
	```
	
使用扩展方法
	
	``` bash
	using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace wuFengTingHai.Person.Extend
{
    public class LinqExtend
    {
        private IList<Person> persons = new List<Person>();
        public IList<Person> Persons
        {
            get
            {
                //删除集合中名称为无风听海的记录，引入system.linq的扩展方法
                this.persons = this.persons.Where(m => !m.Name.Equals("无风听海")).ToList<Person>();
                return this.persons;
            }
            set { this.persons = value; }
        }
    }
}
	```
引入system.linq后,扩展方法的智能提示
 从上面的例子中我们可以学到扩展方法的使用方法。那么扩展方法是怎么定义的呢？查看一下Where扩展方法的定义  

``` bash
 #region Assembly System.Core.dll, v2.0.50727
// C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.5\System.Core.dll
#endregion

using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.CompilerServices;

namespace System.Linq
{    
    //为了方便展示，将其他的扩展方法进行了删除精简
    public static class Enumerable
    {
        public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate);

        public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, int, bool> predicate);
    }
}
``` 
  虽然扩展方法通过实例方法语法进行调用的，但是他们却被定义为静态方法。从定义中我们可以看到，它们的第一个参数指定该方法作用于哪个类型，并且该参数以 this 修饰符为前缀。

    下面我们自己定义一个Person类和PersonExtend类来扩展方法，来看看编译器究竟做了什么！
	``` bash
	using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace wuFengTingHai.Person
{
    public class Person
    {
        public Person(string name,string sex,string age)
        {
            this.name = name;
            this.sex = sex;
            this.age = age;
        }

        private string name;
        public string Name
        {
            set { this.name = value; }
            get { return this.name; }
        }

        private string sex;
        public string Sex
        {
            set { this.sex = value; }
            get { return this.sex; }
        }

        private string age;
        public string Age
        {
            set { this.age = value; }
            get { return this.age; }
        }

         public override string  ToString()
        {
            return string.Format("{0}","类本身的方法覆盖了同名扩展方法");
        }            

    }
}
	
	```
	 PersonExtend，对Person类进行方法扩展
	 
	 
	 
	 
	 ``` bash
	 using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace wuFengTingHai.Person.Extend
{
     public static  class PersonExtend
    {
         public static string  GetDescription(this Person person,string msg)
         {             
             return string.Format("{0}      我是{1},性别{2},今年{3}岁",msg,person.Name,person.Sex,person.Age);
         }
        
    }
}
	 ```
	 客户端调用结果
	 
	 让我们看一下客户端main方法的IL代码，我们就知道编译器到底干什么了！
	 ``` bash 
	 .method private hidebysig static void  Main(string[] args) cil managed
{
  .entrypoint
  // Code size       63 (0x3f)
  .maxstack  4
  .locals init ([0] class [wuFengTingHai.Person]wuFengTingHai.Person.Person person)
  IL_0000:  nop
  IL_0001:  ldstr      bytearray (E0 65 CE 98 2C 54 77 6D )                         // .e..,Twm
  IL_0006:  ldstr      bytearray (37 75 )                                           // 7u
  IL_000b:  ldstr      "20"
  IL_0010:  newobj     instance void [wuFengTingHai.Person]wuFengTingHai.Person.Person::.ctor(string,
                                                                                              string,
                                                                                              string)
  IL_0015:  stloc.0
  IL_0016:  ldloc.0
  IL_0017:  ldstr      bytearray (F4 76 A5 63 1A 90 C7 8F 59 97 01 60 7B 7C 50 00   // .v.c....Y..`{|P.
                                  65 00 72 00 73 00 6F 00 6E 00 45 00 78 00 74 00   // e.r.s.o.n.E.x.t.
                                  65 00 6E 00 64 00 03 8C 28 75 2C 00 )             // e.n.d...(u,.
                       //直接使用静态类PersonExtend调用
  IL_001c:  call       string [wuFengTingHai.Person.Extend]wuFengTingHai.Person.Extend.PersonExtend::GetDescription(class [wuFengTingHai.Person]wuFengTingHai.Person.Person,
                                                                                                                    string)
  IL_0021:  call       void [mscorlib]System.Console::WriteLine(string)
  IL_0026:  nop
  IL_0027:  ldloc.0
  IL_0028:  ldstr      bytearray (F4 76 A5 63 1A 90 C7 8F 50 00 65 00 72 00 73 00   // .v.c....P.e.r.s.
                                  6F 00 6E 00 84 76 9E 5B 8B 4F 03 8C 28 75 2C 00 ) // o.n..v.[.O..(u,.
                       //直接使用Person的扩展方法调用
  IL_002d:  call       string [wuFengTingHai.Person.Extend]wuFengTingHai.Person.Extend.PersonExtend::GetDescription(class [wuFengTingHai.Person]wuFengTingHai.Person.Person,
                                                                                                                    string)
  IL_0032:  call       void [mscorlib]System.Console::WriteLine(string)
  IL_0037:  nop
  IL_0038:  call       string [mscorlib]System.Console::ReadLine()
  IL_003d:  pop
  IL_003e:  ret
} // end of method Program::Main
	 
	 ```
	 
	 从IL中我们可以看到扩展方法与其扩展的类之间并没有什么本质的联系，只是编译器跟我们玩的把戏罢了，最终编译器还是将扩展方法转化成静态类的静态方法调用，所以扩展方法不能访问相应类的私有字段和私有方法；至于为什么使用静态类的静态方法，我考虑可能是这样效率相对较高，同时扩展方法作为其他类的扩展，本身类的实例化没有什么意义；     

      如果扩展方法和被扩展类中的方法相同，会怎么样？
	  
	  ``` bash
	  using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace wuFengTingHai.Person.Extend
{
     public static  class PersonExtend
    {
         public static string  GetDescription(this Person person,string msg)
         {             
             return string.Format("{0}      我是{1},性别{2},今年{3}岁",msg,person.Name,person.Sex,person.Age);
         }

       
         public static string ToString(this Person person)
         {
             return "扩展方法能够覆盖原始类的同名方法";
         }
    }
}
	  
	  ```
	  客户端调用代码
	  
	  ``` bash
	  using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using wuFengTingHai.Person.Extend;
namespace ExtendMethod
{
    class Program
    {
        static void Main(string[] args)
        {            
            wuFengTingHai.Person.Person person = new wuFengTingHai.Person.Person("无风听海", "男", "20");
            //Console.WriteLine(wuFengTingHai.Person.Extend.PersonExtend.GetDescription(person, "直接通过静态类PersonExtend调用,"));
            //Console.WriteLine(person.GetDescription("直接通过Person的实例调用,"));
            Console.WriteLine(person.ToString());
           

            Console.ReadLine();
        }
    }
}
	  ```
	  
调用结果

   

以上我们可以看到编译时，扩展方法的优先级总是比类型本身中定义的实例方法低，所以与接口或类方法具有相同名称和签名的扩展方法永远不会被调用。

综上进行总结
  扩展方法不改变被扩展类的代码，不用重新编译、修改、派生被扩展类
  扩展方法不能访问被扩展类的私有成员
  扩展方法会被被扩展类的同名方法覆盖，所以实现扩展方法我们需要承担随时被覆盖的风险
  扩展方法看似实现了面向对象中扩展对修改说不的特性，但是也违背了面向对象的继承原则，被扩展类的派生类是不能继承扩展扩展方法的，从而又违背了面向对象的多态性。
  在我们稳定的引用同一个版本的类库，但是我们没有该类库的源代码，那么我们可以使用扩展方法；但是从项目的可扩展、可维护和版本控制方面来说，都不建议

  使用扩展方法进行类的扩展。