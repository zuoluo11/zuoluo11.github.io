---
title: 设计模式-工厂方法模式
date: 2018-01-10 17:30:26
tags:
categories:
- 设计模式
---
 {% asset_img 20180116090202.jpg UML%}
<!--more-->
工厂方法模式一句话：一个产品抽象类，一个工厂抽象类，一个工厂生产一个产品。
> {% post_link SimpleFactoryPattern 简单工厂模式%}如果扩展多个产品，需要修改工厂类中的静态方法，明显不符合开放关闭原则（新增开放修改关闭）,引入工厂方法模式。
> 本篇内容只针对工厂方法模式（Factory Method Pattern）。

简单工厂模式基础代码
```C
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

在上面的代码基础上进行增加一个橘子类，需要做哪些修改呢？
a.创建橘子类继承水果并实现抽象方法PrintName；
b.修改水果工厂CreateFruit方法，增加case分支语句；
c.修改客户类，增加获得橘子的语句；
代码如下：

```C
	/// <summary>
    /// 橘子类
    /// </summary>
    public class Orange : Fruit
    {
        public override void PrintName()
        {
            Console.WriteLine("This is a orange");
        }
    }

```
```C
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
            case "Orange": result = new Orange(); break;//增加橘子分支
            default:
                result = new Apple();//未找到返回一个苹果
                break;
        }
        return result;
    }

```
```C
 	static void Main(string[] args)
    {
        //实例化苹果和香蕉
        Fruit first = FruitFactory.CreateFruit("Apple");
        first.PrintName();
        Console.WriteLine("-------------");
        Fruit second = FruitFactory.CreateFruit("Banana");
        second.PrintName();
        Console.WriteLine("-------------");
        //增加实例化橘子
        Fruit third = FruitFactory.CreateFruit("Orange");
        third.PrintName();
        Console.Read();

    }

```
 {% asset_img 20180110175538.jpg 运行结果%}

做完上面的工作后，可以发现增加一个产品需要修改工厂类方法，明显违背开放关闭原则。
因此将上面的代码进一步进行整理（修改成为工厂方法模式）。
a.新增抽象工厂类;
b.针对每个产品都增加一个对应的工厂类；
c.顾客使用时，使用对应的工厂类来创建产品；

```C
using System;

namespace SettingPattern4
{
    class Customer
    {
        static void Main(string[] args)
        {
            //实例化苹果和香蕉
            Fruit first = new AppleFactory().CreateFruit();
            first.PrintName();
            Console.WriteLine("-------------");
            Fruit second = new BananaFactory().CreateFruit();
            second.PrintName();
            Console.WriteLine("-------------");
            //实例化橘子
            Fruit third = new OrangeFactory().CreateFruit();
            third.PrintName();
            Console.Read();

        }
    }
    /// <summary>
    /// 水果工厂类抽象类
    /// </summary>
    public abstract class FruitFactory
    {
        public abstract Fruit CreateFruit();
    }
    /// <summary>
    /// 苹果工厂类
    /// </summary>
    public class AppleFactory:FruitFactory
    {
        public override Fruit CreateFruit()
        {
            return new Apple();
        }
    }
    /// <summary>
    /// 香蕉工厂类
    /// </summary>
    public class BananaFactory : FruitFactory
    {
        public override Fruit CreateFruit()
        {
            return new Banana();
        }
    }
    /// <summary>
    /// 橘子工厂类
    /// </summary>
    public class OrangeFactory : FruitFactory
    {
        public override Fruit CreateFruit()
        {
            return new Orange();
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
    /// <summary>
    /// 橘子类
    /// </summary>
    public class Orange : Fruit
    {
        public override void PrintName()
        {
            Console.WriteLine("This is a orange");
        }
    }

}


```
通过上面的修改可以发现，当再次增加一个新的产品时，原来的代码不需要进行修改，只需要新增该产品类和对应的工厂类即可。

**总结**
**使用场景**
a. 针对同一种类多个产品的生产与消费的解耦，满足开放关闭原则。

**优点**
避免简单工厂类不易扩展与工厂类业务逻辑复杂的缺陷
**缺点**
当多个产品打包后的产品族需要进行扩展时，则显无能为力。
**即横向单个产品的扩展方便，纵向产品族扩展复杂**

针对此缺点引入{% post_link AbstractFactoryPattern 设计模式-抽象工厂模式 %}。
