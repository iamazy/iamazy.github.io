---
title: SPI机制简单示例
date: 2020-03-18 23:13:23
tags: [spi,java]
categories: java
---
> spi(Service Provider Interface)是一种服务发现机制，主要对接口进行解耦，实现对装配类的动态加载。本文只讲如何使用`spi`，不去分析它的源码。


### 1. 在classpath下创建`META-INF/services`文件夹
`ServiceLoader`将会扫描`META-INF/services`下的文件

### 2. 创建以父类或接口的完全限定名为名的文件
比如创建一个接口名为`Animal`，它的完全限定名为`io.github.iamazy.asm.spi.Animal`，则在`META-INF/services`文件夹下创建文件`io.github.iamazy.asm.spi.Animal`。

Animal接口内容如下：
```java
package io.github.iamazy.asm.spi;

public interface Animal {
    String name();
}
```

### 3. 在`io.github.iamazy.asm.spi.Animal`文件中添加子类的完全限定名
创建`Animal`的子类`Dog`和`Cat`，类文件内容如下：
```java
// Dog.java
package io.github.iamazy.asm.spi;

public class Dog implements Animal {
    @Override
    public String name() {
        return "DOG";
    }
}

// Cat.java
package io.github.iamazy.asm.spi;

public class Cat implements Animal {
    @Override
    public String name() {
        return "CAT";
    }
}
```
则`io.github.iamazy.asm.spi.Animal`文件内容应为：
```
io.github.iamazy.asm.spi.Dog
io.github.iamazy.asm.spi.Cat
```

### 4. 使用`ServiceLoader`加载
在main方法中使用`ServiceLoader`加载`Animal`接口
```java
public static void main(String[] args) {
    ServiceLoader<Animal> animals = ServiceLoader.load(Animal.class);
    for(Animal animal:animals){
        System.out.println(animal.name());
    }
}
```


