---
title: AOP & java动态代理
date: 2017-05-11 11:21:20
tags:
- 动态代理
- 反射
- 代理
- AOP

categories:
- java
---

面向切面的程序设计（aspect-oriented programming，AOP，又译作面向方面的程序设计、观点导向编程、剖面导向程序设计）是计算机科学中的一个术语，指一种程序设计范型。该范型以一种称为侧面（aspect，又译作方面）的语言构造为基础，侧面是一种新的模块化机制，用来描述分散在对象、类或函数中的横切关注点（crosscutting concern）。
侧面的概念源于对面向对象的程序设计的改进，但并不只限于此，它还可以用来改进传统的函数。与侧面相关的编程概念还包括元对象协议、主题（subject）、混入（mixin）和委托。

面向切面的程序设计主要是实现思想就是动态代理,了解动态代理之前先看一下代理模式:

## 代理模式
定义:代理模式(Proxy Pattern) ：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。代理模式的英文叫做Proxy或Surrogate，它是一种对象结构型模式。
实现上是代理类中维护队原对象的应用(通过接口引用,面向接口编程)

接口定义:
```
public interface IHello {
    /**
     * 假设这是一个业务方法
     * @param name
     */
    void sayHello(String name);
}
```

被代理者实现接口:
```
public class Hello implements IHello {

    public void sayHello(String name) {
        System.out.println("Hello " + name);
    }

}
```

代理类维护代理者的引用,实现代理:

```
public class HelloProxy implements IHello {
     private IHello hello;  //接口引用被代理者

     public HelloProxy(IHello hello) {
         this.hello = hello;
     }

    public void sayHello(String name) {
        Logger.logging(Level.DEBUGE, "sayHello method start.");
        hello.sayHello(name);
        Logger.logging(Level.INFO, "sayHello method end!");

    }
}
```


## 动态代理模式
简单的说就是如何动态生成类似HelloProxy代理类:在java的动态代理机制中，有两个重要的类或接口，一个是 在java的动态代理机制中，有两个重要的类或接口，一个是 InvocationHandler(Interface)、另一个则是 Proxy(Class)，这一个类和接口是实现我们动态代理所必须用到的。(Interface)、另一个则是 Proxy(Class)，这一个类和接口是实现我们动态代理所必须用到的。
1.InvocationHandler 唯一一个方法 invoke 方法,当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。
2.Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法.利用java的反射原理生成代理者;

## 实例解析

1.接口定义
```
public interface Calculator {

    public int calculate( int a , int b);
}
```

2.被代理者实现接口
```
public class CalculatorImpl implements Calculator{

    @Override
    public int calculate(int a, int b) {
        return a/b;
    }
}
```
3.动态代理生成
```
public class ProxyFactory implements InvocationHandler {

    private Object opt;

    public  ProxyFactory(Object opt) {
        this.opt = opt;
    }

    /**
        * 要处理的对象中的每个方法会被此方法送去JVM调用,也就是说,要处理的对象的方法只能通过此方法调用
        * 此方法是动态的,不是手动调用的
        */

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        try {
            //JVM通过这条语句执行原来的方法(反射机制),可以在invoke前后做一些处理,就是代理的前后处理的事情.
            result = method.invoke(this.opt, args);
            //执行原来的方法之后记录日志

        } catch (Exception e) {
            e.printStackTrace();
        }
        //返回方法返回值给调用者
        return result;
    }
}
```
4.测试运行
```
public class AopTest {

    public static void main(String[] args){

        /**被代理的真实对象 */
        CalculatorImpl calcImpl = new CalculatorImpl();

        /** 实际调用方法的handler  实际上这貌似就是代理对象*/
        InvocationHandler handler = new ProxyFactory(calcImpl);

        /**
         * 参数1 用来定义代理类的 loader
         * 参数2 代理类要实现的接口
         * 参数3 分发方法调用的handler*/
        Calculator proxy =(Calculator) Proxy.newProxyInstance(handler.getClass().getClassLoader(),calcImpl.getClass().getInterfaces(),handler);

        int result = proxy.calculate(20, 10);
        System.out.print("FInal Result :::" + result);
    }

}
```





