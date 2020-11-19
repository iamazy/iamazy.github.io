---
title: Aspect获取第三方jar包的切面
date: 2020-11-19 21:30:31
tags: [java,aspect,aop]
categories: aspect
---

> 在之前总以为`Aspect`的使用场景有限，只能获取项目内类的切面，原来并非如此。

## 1. 引入依赖

在项目中引用相关依赖

```xml
<dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.8.13</version>
    </dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.6</version>
    <scope>runtime</scope>
</dependency>


 <plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.10</version>
    <configuration>
        <complianceLevel>1.8</complianceLevel>
        <source>1.8</source>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
            </goals>
        </execution>
    </executions>
 </plugin>
```

## 2. 编写`Aspect`表达式

举例

```java
package io.github.iamazy.example;

@Aspect
public class LogAspect {

    @Around("annotation(io.github.iamazy.demo.Logger)")
    public void log(Proceedingjoinpoint joinpoint) {
        // log action
    }
}
```

其中`io.github.iamazy.demo.Logger`是第三方依赖中的注解，且第三方依赖依赖使用了这个注解

## 3. 编写aop.xml文件

```xml
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "http://www.eclipse.org/aspectj/dtd/aspect.dtd>
<aspectj>
    <weaver options="-verbose">
        <include within="io.github.iamazy.action.*" />
    </weaver>
    <aspects>
        <aspect name="io.github.iamazy.example.LogAspect" />
    </aspects>
</aspectj>
```

其中`io.github.iamazy.action.*`就是第三方依赖中使用`io.github.iamazy.demo.Logger`注解的路径，如果有多个，继续添加`<include/>`即可。

## 4. 启动项目

在启动项目时，需要指定`VM参数`，如果是部署项目，可以写在shell脚本中。
```shell
-javaagent: /path/aspectweaver-1.9.6.jar
```

最后你就会惊奇的发现，真的可以获取第三方依赖的切面，然后搞搞事情！！！