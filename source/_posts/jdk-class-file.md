---
title: class文件格式
date: 2020-03-17 22:03:52
tags: [jdk,java,源码]
categories: jdk-tutorial
---

> 学习Java的同学对class文件可能不会陌生，它是`.java`文件编译后生成的字节码文件(扩展名为`.class`)，它是Java语言`一次编译，处处运行`的基础，也是其他`jvm`语言运行在jvm上的基础。

### 1. class文件结构
一个class由以下各种属性构成
```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

#### 1. magic
魔数，用来标识一个class文件的格式，它的值是`0xCAFEBABE`(咖啡宝贝)，单位是无符号的4个字节(`u4`)。(通常识别一个文件格式比较严谨的方式是鉴别它的魔数而不是文件的扩展名)。jvm加载class文件时会首先检查这四个字节，如果不是`0xCAFEBABE`则会拒绝加载该文件避免浪费资源。

#### 2. minor_version，major_version
这个字段名是不是很熟悉，假如你的开发环境的jdk版本和部署环境的jdk版本不是一个主版本(如开发环境是jdk1.8，部署环境是jdk13)，很有可能会遇到`Unsupported major.minor version 57`，第一次遇到这个异常很多人都会很奇怪这个57是怎么来的。

|Java SE|对应的主版本号|兼容的主版本号|
|--|--|--|
|1.0.2|45|45|
|1.1|45|45|
|1.2|46|45 .. 46|
|1.3|47|45 .. 47|
|1.4|48|45 .. 48|
|5.0|49|45 .. 49|
|6|50|45 .. 50|
|7|51|45 .. 51|
|8|52|45 .. 52|
|9|53|45 .. 53|
|10|54|45 .. 54|
|11|55|45 .. 55|
|12|56|45 .. 56|
|13|57|45 .. 57|

主版本号(major version)和次版本号(minor version)共同决定class文件格式的版本。如果一个class文件的主版本号是M，次版本号是m，则将该class文件格式的版本定义为`M.m`
> 对于主版本号(major version)大于56的class文件，次版本号(minor version)必须是0或65535  
> 对于主版本号(major version)在[45,55]范围内的，次版本号(minor version)可以是任意值

#### 3. constant_pool_count
顾名思义，表示常量池的大小，等于常量池`constant_pool`的大小加1

#### 4. constant_pool[constant_pool_count-1]
紧跟在`constant_pool_count`后面的结构就是`constant_pool`，表示常量池，里面存储`constant_pool_count`个常量(主要包括字面常量，类和接口名，字段名，以及其他类型的常量)，常量池的索引值的范围是[1,constant_pool_count-1]。  
每个常量池的项(entry)使用`cp_info`类型表示，`cp_info`的结构为：
```
cp_info {
    u1 tag;
    u1 info[];
}
```
jvm根据`tag`的值来确定每个常量池的项表示什么类型的字面量，info[]表示的是该字面量的字节数组。  
jvm规定了不同的`tag`对应不同类型的字面量，对应关系如下表所示：

|tag|表示的字面量|对应的结构|
|--|--|--|
|1|表示字符串常量的值|CONSTANT_Utf8_info|
|3|表示4字节(int)数值常量|CONSTANT_Integer_info|
|4|表示4字节(float)数值常量|CONSTANT_Float_info|
|5|表示8字节(long)数值常量|CONSTANT_Long_info|
|6|表示8字节(double)数值常量|CONSTANT_Double_info|
|7|表示类或接口的完全限定名|CONSTANT_Class_info|
|8|表示java.lang.String类型的常量|CONSTANT_String_info|
|9|表示类的字段|CONSTANT_Fieldref_info|
|10|表示类中的方法|CONSTANT_Methodref_info|
|11|表示类所实现接口的方法|CONSTANT_InterfaceMethodref_info|
|12|表示字段或方法的名称和类型|CONSTANT_NameAndType_info|
|15|表示方法句柄|CONSTANT_MethodHandle_info|
|16|表示方法类型|CONSTANT_MethodType_info|
|18|表示invokedynamic指令所使用的引导方法及其调用名称，参数，请求返回类型以及可以选择性附加的静态参数的常量序列(参考`lambda`表达式)|CONSTANT_InvokeDynamic_info|


#### 5. access_flags
类访问标识修饰符

|flag|值|含义|
|--|--|--|
|ACC_PUBLIC|0x0001|允许被不在同一package的类访问|
|ACC_FINAL|0x0010|不允许有子类|
|ACC_SUPER|0x0020|当被`invokespecial`指令调用时，需要特别处理超类方法|
|ACC_INTERFACE|0x0200|表示是一个接口|
|ACC_ABSTRACT|0x0400|表示是抽象的类或方法，不允许被初始化|
|ACC_SYNTHETIC|0x1000|不会在源码中显示|
|ACC_ANNOTATION|0x2000|表示是一个注解类型|
|ACC_ENUM|0x4000|表示是一个枚举类型|
|ACC_MODULE|0x8000|表示是一个模块|

可以自己试着分析以下哪些标识可以一起被设置，哪些则不能(如`ACC_FINAL`和`ACC_ABSTRACT`不能同时被设置)

#### 5. this_class
表示当前类的全局限定名在常量池(`constant_pool`)中的索引，该索引对应的常量池项必须是`CONSTANT_Class_info`的结构，代表这个class文件定义的是一个类还是一个接口。

#### 6. super_class
对于一个类，它的`super_class`的值必须是0或者是常量池中的一个有效索引，如果`super_class`不为0，则其对应的常量池中的项必须是`CONSTANT_Class_info`的结构，代表在class文件中，是该类的直接超类。该直接超类和其他超类都不允许被`access_flags`为`ACC_FINAL`的修饰符修饰。

#### 7. interfaces_count
表示该类(或接口)直接实现(或继承)的接口数

#### 8. interfaces[]
表示的该类(或接口)是直接实现(或继承)的接口集合在常量池中的索引数组，数组的长度在[0,interfaces_count)之间。且常量池中对应的索引必须都是`CONSTANT_Class_info`的结构。

#### 9. fields_count
表示该类中定义的字段(包括静态变量和实例变量)的数量

#### 10. fields[]
表示字段数组，每一项的类型必须是`field_info`，以提供该类或接口中字段的完整描述。这个字段数组只包含该类或接口自己定义的字段，不包含从超类或超接口中继承的字段。

#### 11. methods_count
表示该类或接口中定义的方法数量

#### 12. methods[]
表示方法数组，每一项的类型必须是`method_info`，以提供该类或接口中字段的完成描述。方法数组包含这个类或接口声明的所有方法，但是不包含从超类或超接口中继承的方法。如果某些项中未设置`access_flags`为`ACC_NATIVE`或`ACC_ABSTRACT`的修饰符，还需要提供实现该方法的jvm指令。

#### 13. attributes_count
表示该类或接口中定义的属性的数量

#### 14. attributes[]
表示该class文件中定义的属性列表，其中的每一项必须是`attribute_info`类型。

### 2. 完全限定名
类或接口在class文件中始终以完全限定名表示，这类名称使用`CONSTANT_Utf8_info`结构进行表示。  
类和接口是从将这类名称作为他们描述符(descriptor)的一部分的`CONSTANT_NameAndType_info`和`CONSTANT_Class_info`的结构中引用的。  
由于历史原因，出现在class文件中的完全限定名和我们在程序中使用的完全限定名不一样，在class内部结构中，通常会将英文符号的`.`使用正斜杠`/`代替。
> 如`Thread`类的完全限定名是`java.lang.Thread`，在class文件格式的描述符中，`Thread`使用`CONSTANT_Utf8_info`结构的字符串`java/lang/Thread`来实现对类名的引用。

----------------------------
### 3. 非限定名称
方法，字段，局部变量和形参的名称都以非限定名称存储，非限定名称至少要包含一个Unicode`代码点`且不允许包含`. ; [ /`的字符。
方法名称还受到进一步的限制：除了特殊方法名称`<init>`和`<clinit>`，方法名称不得包含`< >`的字符。  
请注意，字段名称或接口方法名称可以是`<init>`或`<clinit>`，但是任何方法调用指令都不能引用`<clinit>`，而只有`invokespecial`指令可以引用`<init>`。

### 4. 描述符
> 描述符表示一个字段或方法类型的字符串。
#### 1. 字段描述符

|class文件中的字段类型|字段类型|解释|示例|
|--|--|--|--|
|V|void|void类型|V -> void|
|B|byte|有符号的byte类型|B -> byte|
|C|char|字符类型|C -> char|
|D|double|双精度浮点类型|D -> double|
|F|float|单精度浮点类型|F -> float|
|I|int|整型|I -> int|
|J|long|长整型|J -> long|
|L|className;|引用|一个类实例的引用|Object类的实例 -> Ljava/lang/Object;|
|S|short|有符号的短整型|S -> short|
|Z|boolean|布尔类型|Z -> boolean|
|[|一维数组|一维数组的引用|[[[D -> double[][][]|


#### 2. 方法描述符
> 方法描述符的表达式: `(参数描述符) 返回值描述符`

示例：
> void m(int i,double d) { ... } -> (ID)V   
> Object m(int i,double d,Thread t) { ... } -> (IDLjava/lang/Thread;)Ljava/lang/Object;

<!-- `this`关键字对对象的引用除了预期的参数外，并未反应在方法描述符中，实际上，对`this`的引用由调用实例方法的jvm指令隐式传递。 -->

### 5. 实战
下面将通过一个简单的例子来体会上述文档的含义。

#### 1. 创建项目，并引入依赖
使用ide创建一个java项目，并引入maven依赖(其实jdk里已经包含asm库了)
```xml
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm</artifactId>
    <version>7.3.1</version>
</dependency>
```

#### 2. 编写java文件
创建一个java文件，可以命名为`ClassWriterTest.java`，并输入如下代码：
```java
import org.objectweb.asm.ClassWriter;
import java.io.FileOutputStream;
import java.io.IOException;
import static org.objectweb.asm.Opcodes.*;

public class ClassWriterTest {

    public static void main(String[] args) throws IOException {

        ClassWriter classWriter = new ClassWriter(0);
        classWriter.visit(V1_8,ACC_PUBLIC+ACC_ABSTRACT+ACC_INTERFACE,
                "pkg/Comparable",null,"java/lang/Object",
                new String[]{"java/io/Serializable"});
        classWriter.visitField(ACC_PUBLIC+ACC_FINAL+ACC_STATIC,"LESS",
                "I",null, -1).visitEnd();
        classWriter.visitField(ACC_PUBLIC+ACC_FINAL+ACC_STATIC,"EQUAL",
                "Z",null, true).visitEnd();
        classWriter.visitField(ACC_PUBLIC+ACC_FINAL+ACC_STATIC,"GREATER",
                "J",null, -1).visitEnd();
        classWriter.visitMethod(ACC_PUBLIC+ACC_ABSTRACT,"compareTo",
                "(Ljava/lang/Object;)I",null,null).visitEnd();
        classWriter.visitEnd();
        byte[] bytes = classWriter.toByteArray();
        FileOutputStream fos = new FileOutputStream("./Comparable.class");
        fos.write(bytes);
        fos.close();
    }
}
```
执行代码的结果会生成一个`Comparable.class`文
```java
// Comparable.class
package pkg;

public interface Comparable extends java.io.Serializable {
    int LESS = -1;
    boolean EQUAL = true;
    long GREATER = -1;

    int compareTo(java.lang.Object o);
}
```
结合代码和生成的class文件，你对上面的文档有更深的了解了吗？


参考: [Java Virtual Machine Specification-ch4](https://docs.oracle.com/javase/specs/jvms/se13/html/jvms-4.htm)