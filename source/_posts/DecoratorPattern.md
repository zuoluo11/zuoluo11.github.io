---
title: 设计模式-装饰模式
date: 2018-01-16 08:53:10
tags:
categories:
- 设计模式
---
 {% asset_img 20180116085436.jpg 装饰模式%}
 <!--more-->
1、 装饰者模式，动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更加有弹性的替代方案。

2、组合和继承的区别

继承。继承是给一个类添加行为的比较有效的途径。通过使用继承，可以使得子类在拥有自身方法的同时，还可以拥有父类的方法。但是使用继承是静态的，在编译的时候就已经决定了子类的行为，我们不便于控制增加行为的方式和时机。

组合。组合即将一个对象嵌入到另一个对象中，由另一个对象来决定是否引用该对象来扩展自己的行为。这是一种动态的方式，我们可以在应用程序中动态的控制。

与继承相比，组合关系的优势就在于不会破坏类的封装性，且具有较好的松耦合性，可以使系统更加容易维护。但是它的缺点就在于要创建比继承更多的对象。

3、装饰者模式的优缺点

优点

a.装饰者模式可以提供比继承更多的灵活性
b.可以通过一种动态的方式来扩展一个对象的功能，在运行时选择不同的装饰器，从而实现不同的行为。
c.通过使用不同的具体装饰类以及这些装饰类的排列组合，可以创造出很多不同行为的组合。可以使用多个具体装饰类来装饰同一对象，得到功能更为强大的对象。
d.具体构件类与具体装饰类可以独立变化，用户可以根据需要增加新的具体构件类和具体装饰类，在使用时再对其进行组合，原有代码无须改变，符合“开闭原则”。

缺点

a.会产生很多的小对象，增加了系统的复杂性
b.这种比继承更加灵活机动的特性，也同时意味着装饰模式比继承更加易于出错，排错也很困难，对于多次装饰的对象，调试时寻找错误可能需要逐级排查，较为烦琐。

4、装饰者的使用场景

a.在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
b.需要动态地给一个对象增加功能，这些功能也可以动态地被撤销。  
c.当不能采用继承的方式对系统进行扩充或者采用继承不利于系统扩展和维护时。

 [以上内容来自网络](https://www.cnblogs.com/xinye/p/3910149.html)
 ```C
using System;

namespace SettingPattern4
{
    /// <summary>
    /// Decorator Pattern 装饰模式
    /// 优点：通过动态的方式为类附加功能
    /// </summary>
    class Customer
    {
        static void Main(string[] args)
        {
            Person p = new Student();
            ConcreteStudent1 c1 = new ConcreteStudent1();
            ConcreteStudent2 c2 = new ConcreteStudent2();

            c1.SetPerson(p);
            c2.SetPerson(c1);
            c2.Operate();
           

            Console.Read();
        }
    }

    public abstract class Person
    {
        public abstract void Operate();
    }

    public class Student : Person
    {
        public override void Operate()
        {
            Console.WriteLine("原始Student操作");
        }
    }

    public abstract class Decorator : Person
    {
        private Person person;
        public void SetPerson(Person person)
        {
            this.person = person;
        }
        public override void Operate()
        {
            if (person != null)
            {
                person.Operate();
            }
        }
    }

    public class ConcreteStudent1 : Decorator
    {
        public override void Operate()
        {
            base.Operate();
      
            Console.WriteLine("ConcreteStudent1添加操作");
        }

        public void Print()
        {
            Console.WriteLine("ConcreteStudent1 Print");
        }
    }
    public class ConcreteStudent2 : Decorator
    {

        public override void Operate()
        {
            base.Operate();
            Console.WriteLine("ConcreteStudent2添加操作");
        }
    }
}



 ```