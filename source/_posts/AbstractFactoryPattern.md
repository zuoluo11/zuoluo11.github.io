---
title: 设计模式-抽象工厂模式
date: 2018-01-11 09:31:49
tags:
categories:
- 设计模式
---
 {% asset_img 20180116090042.jpg UML%}
<!--more-->
抽象工厂模式一句话：一个工厂抽象类，多个产品抽象类，一个工厂生产一个产品族（多个有关系的产品组合）。
> {% post_link SimpleFactoryPattern 简单工厂模式： %}一个工厂生产多个产品
> {% post_link FactoryMethodPattern 工厂方法模式： %}一个工厂生产一个产品
> 抽象工厂模式：一个工厂生产一个产品族

首先下面有一个关于电脑的工厂方法模式，稍后将它改造为抽象工厂模式。

业务逻辑分析：
a.电脑配件有鼠标、键盘、屏幕。
b.顾客拿到配件后进行组装使用。

```C
using System;

namespace SettingPattern4
{
    class Customer
    {
        static void Main(string[] args)
        {

            Parts mouse = new MouseFactory().CreateFruit();
            mouse.PrintName();
            Console.WriteLine("-------------");
            Parts keyboard = new KeyboardFactory().CreateFruit();
            keyboard.PrintName();
            Console.WriteLine("-------------");
            Parts screen = new ScreenFactory().CreateFruit();
            screen.PrintName();
            Console.Read();

        }
    }
    /// <summary>
    /// 工厂抽象类
    /// </summary>
    public abstract class Factory
    {
        public abstract Parts CreateFruit();
    }
    /// <summary>
    /// 鼠标工厂类
    /// </summary>
    public class MouseFactory : Factory
    {
        public override Parts CreateFruit()
        {
            return new Mouse();
        }
    }
    /// <summary>
    /// 键盘工厂类
    /// </summary>
    public class KeyboardFactory : Factory
    {
        public override Parts CreateFruit()
        {
            return new KeyBoard();
        }
    }
    /// <summary>
    /// 屏幕工厂类
    /// </summary>
    public class ScreenFactory : Factory
    {
        public override Parts CreateFruit()
        {
            return new Screen();
        }
    }
    /// <summary>
    /// 配件抽象类
    /// </summary>
    public abstract class Parts
    {
        /// <summary>
        /// 配件抽象方法
        /// </summary>
        public abstract void PrintName();
    }
    /// <summary>
    /// 鼠标类
    /// </summary>
    public class Mouse : Parts
    {
        public override void PrintName()
        {
            Console.WriteLine("This is an mouse.");
        }
    }
    /// <summary>
    /// 键盘类
    /// </summary>
    public class KeyBoard : Parts
    {
        public override void PrintName()
        {
            Console.WriteLine("This is a keyboard");
        }
    }
    /// <summary>
    /// 屏幕类
    /// </summary>
    public class Screen : Parts
    {
        public override void PrintName()
        {
            Console.WriteLine("This is a screen");
        }
    }

}


```
问题提出：顾客认为自己配置电脑太累，打算直接购买品牌机。
很明显：上面的代码中没有体现出各个配件之间的关系且最后由用户进行组装，现在配件内部存在关联关系且由工厂组装好后直接给用户使用。
此处可以看出横向扩展配件，如加入一个耳机比较容易，如果要加入一个纵向的产品族则很复杂。

业务描述：
a.品牌机的配件都只有该品牌自己的标识，因此各个配件内部都有关系，不会买惠普的电脑里出现联想的鼠标；
b.顾客现在可以买惠普的也可以买联想的；


```C
using System;

namespace SettingPattern4
{
    class Customer
    {
        static void Main(string[] args)
        {
            Factory factory = new LenovoFactory();
            factory.CreateKeyboard().PrintName();
            factory.CreateMouse().PrintName();
            factory.CreateScreen().PrintName();
            Console.WriteLine("-----切换厂家-----");
            factory = new HPFactory();
            factory.CreateKeyboard().PrintName();
            factory.CreateMouse().PrintName();
            factory.CreateScreen().PrintName();
            Console.Read();

        }
    }
    /// <summary>
    /// 工厂抽象类
    /// </summary>
    public abstract class Factory
    {
        public abstract MouseBase CreateMouse();
        public abstract KeyboardBase CreateKeyboard();
        public abstract ScreenBase CreateScreen();
    }

    public abstract class MouseBase
    {
        public abstract void PrintName();
    }
    public abstract class KeyboardBase
    {
        public abstract void PrintName();
    }
    public abstract class ScreenBase
    {
        public abstract void PrintName();
    }

    /// <summary>
    /// HP工厂类
    /// </summary>
    public class HPFactory : Factory
    {
        public override KeyboardBase CreateKeyboard()
        {
            return new HPKeyBoard();
        }

        public override MouseBase CreateMouse()
        {
            return new HPMouse();

        }

        public override ScreenBase CreateScreen()
        {
            return new HPScreen();

        }
    }

    /// <summary>
    /// Lenovo工厂类
    /// </summary>
    public class LenovoFactory : Factory
    {
        public override KeyboardBase CreateKeyboard()
        {
            return new LenovoKeyBoard();
        }

        public override MouseBase CreateMouse()
        {
            return new LenovoMouse();

        }

        public override ScreenBase CreateScreen()
        {
            return new LenovoScreen();

        }
    }
   

    /// <summary>
    /// HP鼠标类
    /// </summary>
    public class HPMouse : MouseBase
    {
        public override void PrintName()
        {
            Console.WriteLine("This is HP鼠标.");
        }
    }
    /// <summary>
    /// lenovo鼠标类
    /// </summary>
    public class LenovoMouse : MouseBase
    {
        public override void PrintName()
        {
            Console.WriteLine("This is lenovo鼠标.");
        }
    }
    /// <summary>
    /// HP键盘类
    /// </summary>
    public class HPKeyBoard : KeyboardBase
    {
        public override void PrintName()
        {
            Console.WriteLine("This is HP键盘");
        }
    }
    /// <summary>
    /// Lenovo键盘类
    /// </summary>
    public class LenovoKeyBoard : KeyboardBase
    {
        public override void PrintName()
        {
            Console.WriteLine("This is a Lenovo键盘");
        }
    }
    /// <summary>
    /// HP屏幕类
    /// </summary>
    public class HPScreen : ScreenBase
    {
        public override void PrintName()
        {
            Console.WriteLine("This is HP屏幕");
        }
    }
    /// <summary>
    /// Lenovo屏幕类
    /// </summary>
    public class LenovoScreen : ScreenBase
    {
        public override void PrintName()
        {
            Console.WriteLine("This is Lenovo屏幕");
        }
    }
}


```
 {% asset_img 20180111104640.jpg 运行结果%}

此时，如果想添加一个华硕的计算机，需要做如下修改：
a.创建华硕的配件每个配件都继承该配件的抽象类；
b.创建华硕的工厂类继承工厂抽象类；
c.顾客直接创建华硕工厂获得计算机；

[这里的总结的很好：](https://jackal007.gitbooks.io/design_pattern/content/abstract-factory.html)
>工厂创建一种产品，抽象工厂创建的是一组产品，是一个产品系列。这里要注意的是“系列”的意思，也就是说，抽象工厂创建出的一组产品是成套的。
>当你发现，有一个接口可以有多种实现的时候，可以考虑使用工厂方法来创建实例。
>当你返现，有一组接口可以有多种实现方案的时候，可以考虑使用抽象工厂创建实例组。