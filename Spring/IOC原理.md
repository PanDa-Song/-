# IOC 原理



## IOC 概述

Spring 框架的核心是 Spring 容器。容器创建对象，将它们装配在一起，配置它们并管理它们的完整生命周期。Spring 容器使用依赖注入来管理组成应用程序的组件。

容器通过读取提供的配置元数据来接收对象进行实例化，配置和组装的指令。该元数据可以通过 XML，Java 注解或 Java 代码提供。

在依赖注入中，您不必创建对象，但必须描述如何创建它们。您不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。由 IoC 容器将它们装配在一起。





## IOC 的实现机制

IOC 本质上实现原理就是依靠**工厂模式**加上 Java 的**反射机制**

```java
interface Fruit {
    public abstract void eat();
}

class Apple implements Fruit {
    public void eat() {
        System.out,println("Apple");
    }
}
class Orange implements Fruit {
    public void eat() {
        System.out,println("Orange");
    }
}
class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f = null;
        try {
            f = (Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
```

 

