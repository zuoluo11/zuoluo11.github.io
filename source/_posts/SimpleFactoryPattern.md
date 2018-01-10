---
title: 设计模式-简单工厂模式
date: 2018-01-10 16:15:16
tags:
categories:
- 设计模式
---
简单工厂模式一句话：一个产品抽象类，一个工厂类，这个工厂生产多个产品。
<!--more-->
> 设计模式对编写代码的帮助是不言而喻的，有必要在学习的基础上做一些笔记。
> 工厂类应用场景一般为创建对象。根据场景需求的不同，可以一般分为简单工厂模式（静态工厂）、工厂方法模式、抽象工厂模式。
> 本篇内容只针对简单工厂模式。

一般情况下，如果需要实例化一个对象，那么就要new一个对象如下：
```JAVA
using System;

namespace SettingPattern4
{
    class Customer
    {
        static void Main(string[] args)
        {
            //实例化苹果和香蕉
            Apple apple = new Apple();
            apple.PrintName();
            Banana banana = new Banana();
            banana.PrintName();
            Console.Read();

        }
    }
    /// <summary>
    /// 苹果类
    /// </summary>
    public class Apple
    {
        public void PrintName()
        {
            Console.WriteLine("This is an apple.");
        }
    }
    /// <summary>
    /// 香蕉类
    /// </summary>
    public class Banana
    {
        public void PrintName()
        {
            Console.WriteLine("This is a banana");
        }
    }
}

```
十分简单的代码，顾客想要一个苹果和一个香蕉，那么他就自己初始化一个苹果和一个香蕉就好。

问题提出：顾客不关心水果是怎么创建的，只需要最后把水果给他就好，如何做能将顾客与水果关系解耦？

解决办法：
a.抽象水果类，所有的水果都继承该类;
b.工厂类，静态方法生成对应的水果对象；
c.顾客想要什么水果，给工厂类的方法传入水果名称就好；

```javascript
using System;

namespace SettingPattern4
{
    class Customer
    {
        static void Main(string[] args)
        {
            //实例化苹果和香蕉
            Fruit first = FruitFactory.CreateFruit("Apple");
            first.PrintName();
            Console.WriteLine("-------------");
            Fruit second = FruitFactory.CreateFruit("Banana");
            second.PrintName();
            Console.Read();

        }
    }
    /// <summary>
    /// 水果工厂类
    /// </summary>
    public class FruitFactory
    {
        /// <summary>
        /// 静态水果生产方法
        /// </summary>
        /// <param name="fruitName"></param>
        /// <returns></returns>
        public static Fruit CreateFruit(string fruitName)
        {
            Fruit result;
            switch (fruitName)
            {
                case "Apple": result = new Apple(); break;
                case "Banana": result = new Banana(); break;
                default:
                    result = new Apple();//未找到返回一个苹果
                    break;
            }
            return result;
        }
    }
    /// <summary>
    /// 水果抽象类
    /// </summary>
    public abstract class Fruit
    {
        /// <summary>
        /// 水果抽象方法
        /// </summary>
        public abstract void PrintName();
    }
    /// <summary>
    /// 苹果类
    /// </summary>
    public class Apple : Fruit
    {
        public override void PrintName()
        {
            Console.WriteLine("This is an apple.");
        }
    }
    /// <summary>
    /// 香蕉类
    /// </summary>
    public class Banana : Fruit
    {
        public override void PrintName()
        {
            Console.WriteLine("This is a banana");
        }
    }
}

```
 {% asset_img 20180110165111.jpg 运行结果%}

可以看出，此时顾客并不知道如何去创建具体的水果对象，他只需要调用工厂创建水果方法并传入名称就能够获得对应水果对象。

**总结**
**使用场景**
a.使用者不关心产品对象是如何创建的，尤其是该对象创建工作十分繁琐;
b.产品类别比较少的情况下;
那么可以通过简单工厂模式将创建工作移入到工厂的静态类中

**优点**
将产品与使用者进行一定程度的解耦。
**缺点**
不方便扩展，增加新的产品时，需要修改工厂类增加if-else产品分支，可能导致工厂类逻辑特别复杂。

针对此缺点引入设计模式-工厂方法模式。
