---
title: Java虚拟机指令集
date: 2020-03-22 17:38:15
tags: [jvm,jdk,java]
categories: jvm-tutorial
---

### aaload (50,0x32)
> 从数组中装载引用类型

#### 1. 操作数栈
> pop: `arrayref`,`index`  
> push: `value`  

`arrayref`必须是引用类型`R[]`，并且必须指向元素为引用类型`R`的数组。索引必须是`int`类型。  
`arrayref`和`index`从操作数栈中弹出，数组中索引为`index`的引用值(`value`)被检索到并推入到操作数栈中。

### -1. 字节码解读示例
在main函数中声明一个长度为1的数组，并给索引为0的项赋值，然后根据索引0调用该值赋给某个变量。代码如下：
```java
package io.github.iamazy.asm.instructions.aaload;

import java.io.Serializable;

public class Aaload implements Serializable {

    String name = "aaload";

    public static void main(String[] args) {
        Object[] objects = new Object[10];
        objects[0] = "hello asm";
        Object object = objects[0];
        System.out.println(object);
    }
}
```
运行main函数或执行`javac Aaload.java`生成`Aaload.class`文件，并在`Aaload.class`文件同级目录执行`javap -verbose Aaload`，生成如下字节码信息。
```
Warning: File ./Aaload.class does not contain class Aaload
Classfile /Users/iamazy/Documents/GitHub/asm-tutorial/target/classes/io/github/iamazy/asm/instructions/aaload/Aaload.class
  Last modified Mar 22, 2020; size 804 bytes
  SHA-256 checksum 7ebc42d18bc4fc81d44ab57f7492aa00e500f1a0fb02c2f71f8be359ca78d102
  Compiled from "Aaload.java"
public class io.github.iamazy.asm.instructions.aaload.Aaload implements java.io.Serializable
  minor version: 0
  major version: 52
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #8                          // io/github/iamazy/asm/instructions/aaload/Aaload
  super_class: #4                         // java/lang/Object
  interfaces: 1, fields: 1, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #4.#29         // java/lang/Object."<init>":()V
   #2 = String             #30            // aaload
   #3 = Fieldref           #8.#31         // io/github/iamazy/asm/instructions/aaload/Aaload.name:Ljava/lang/String;
   #4 = Class              #32            // java/lang/Object
   #5 = String             #33            // hello asm
   #6 = Fieldref           #34.#35        // java/lang/System.out:Ljava/io/PrintStream;
   #7 = Methodref          #36.#37        // java/io/PrintStream.println:(Ljava/lang/Object;)V
   #8 = Class              #38            // io/github/iamazy/asm/instructions/aaload/Aaload
   #9 = Class              #39            // java/io/Serializable
  #10 = Utf8               name
  #11 = Utf8               Ljava/lang/String;
  #12 = Utf8               <init>
  #13 = Utf8               ()V
  #14 = Utf8               Code
  #15 = Utf8               LineNumberTable
  #16 = Utf8               LocalVariableTable
  #17 = Utf8               this
  #18 = Utf8               Lio/github/iamazy/asm/instructions/aaload/Aaload;
  #19 = Utf8               main
  #20 = Utf8               ([Ljava/lang/String;)V
  #21 = Utf8               args
  #22 = Utf8               [Ljava/lang/String;
  #23 = Utf8               objects
  #24 = Utf8               [Ljava/lang/Object;
  #25 = Utf8               object
  #26 = Utf8               Ljava/lang/Object;
  #27 = Utf8               SourceFile
  #28 = Utf8               Aaload.java
  #29 = NameAndType        #12:#13        // "<init>":()V
  #30 = Utf8               aaload
  #31 = NameAndType        #10:#11        // name:Ljava/lang/String;
  #32 = Utf8               java/lang/Object
  #33 = Utf8               hello asm
  #34 = Class              #40            // java/lang/System
  #35 = NameAndType        #41:#42        // out:Ljava/io/PrintStream;
  #36 = Class              #43            // java/io/PrintStream
  #37 = NameAndType        #44:#45        // println:(Ljava/lang/Object;)V
  #38 = Utf8               io/github/iamazy/asm/instructions/aaload/Aaload
  #39 = Utf8               java/io/Serializable
  #40 = Utf8               java/lang/System
  #41 = Utf8               out
  #42 = Utf8               Ljava/io/PrintStream;
  #43 = Utf8               java/io/PrintStream
  #44 = Utf8               println
  #45 = Utf8               (Ljava/lang/Object;)V
{
  java.lang.String name;
    descriptor: Ljava/lang/String;
    flags: (0x0000)

  public io.github.iamazy.asm.instructions.aaload.Aaload();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String aaload
         7: putfield      #3                  // Field name:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 5: 0
        line 7: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lio/github/iamazy/asm/instructions/aaload/Aaload;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: iconst_1
         1: anewarray     #4                  // class java/lang/Object
         4: astore_1
         5: aload_1
         6: iconst_0
         7: ldc           #5                  // String hello asm
         9: aastore
        10: aload_1
        11: iconst_0
        12: aaload
        13: astore_2
        14: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
        17: aload_2
        18: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
        21: return
      LineNumberTable:
        line 10: 0
        line 11: 5
        line 12: 10
        line 13: 14
        line 14: 21
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      22     0  args   [Ljava/lang/String;
            5      17     1 objects   [Ljava/lang/Object;
           14       8     2 object   Ljava/lang/Object;
}
SourceFile: "Aaload.java"
```

有上述信息可知：
##### 1. 指定class文件格式版本号
Aaload.class文件的class文件格式版本号是52.0  

##### 2. 显示类访问标记符的值
类的`access_flags`为`ACC_PUBLIC+ACC_SUPER`=`0x0001+0x0020`=`0x0021`  (ACC_XXX对应的值见[access_flags](https://iamazy.github.io/2020/03/17/jdk-class-file/))  
##### 3. 显示类的基本信息
interfaces: 1 (impl `Serializable`), fields: 1 (`name`), methods: 2 (`<init>`,`main`), attributes: 1 ()  

##### 4. 显示了常量池中具体的信息
如常量池第一行
> `#1 = Methodref          #4.#29         // java/lang/Object."<init>":()V`  

含义：常量池中第一项的类型为`CONSTANT_Methodref_info`，它的值是`#4.#29`(往下找第4项和第29项)，即`java/lang/Object."<init>":()V`

##### 5. 类函数的执行过程
在生成的`Aaload.class`文件中，有两个函数，一个是类`Aaload`的无参构造函数，一个是程序入口`main`函数。
1. `Aaload`的无参构造函数对应的字节码信息
```
public io.github.iamazy.asm.instructions.aaload.Aaload();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1    //可知操作数栈长度为2，本地变量表长度为1，参数列表长度为1
         0: aload_0                     //从本地变量表中加载索引为0变量的值，也即this的引用，压入栈
         1: invokespecial #1            //出栈，使用invokespecial指令调用构造函数(java/lang/Object."<init>":()V),并初始化对象
         4: aload_0                     //同上，调用this.name="aaload",并将this的引用压入栈
         5: ldc           #2            //将字符串"aaload"常量压入栈
         7: putfield      #3            //出栈前面压入的两个值(this引用，字符串"aaload")，并将字符串"aaload"赋值给this.name
        10: return
      LineNumberTable:
        line 5: 0
        line 7: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lio/github/iamazy/asm/instructions/aaload/Aaload;
```
其中：`LineNumberTable`表示代码行号与指令的对应关系，前面一个数字表示代码行号，后面一个数字表示前面code对应的指令代号。`LocalVariableTable`表示本地变量表，这里只存了一个`this`的引用，类型描述为`Lio/github/iamazy/asm/instructions/aaload/Aaload`，`start+length`表示这个变量在字节码中的生命周期起始和结束的偏移量(this声明周期为开始0到结束11)，slot表示这个变量在局部变量表中的槽位(槽位可以复用)。

2. `main`函数对应的字节码信息
```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: iconst_1            //将int类型的常量1压入操作数栈
         1: anewarray     #4    //从栈顶弹出数组长度(指定数组长度为1)，并实例化数组，并将其压栈
         4: astore_1            //将栈顶的引用型数值弹出栈，并保存到局部变量表中的1的位置
         5: aload_1             //从局部变量表中装载索引值为1(slot=1)的引用型数值，并压栈
         6: iconst_0            //将int类型的常量0压栈
         7: ldc           #5    //将字符串"hello asm"常量压栈
         9: aastore             //将栈顶的数值依次弹出，并引用型数值存入指定数组的指定索引位置
        10: aload_1             //同上，从局部变量表中装载索引值为1(slot=1)的引用型数值，并压栈
        11: iconst_0            //将int类型的常量0压栈
        12: aaload              //将引用型数组指定索引的值(0)推送至栈顶
        13: astore_2            //将栈顶的引用型数值(object[0])弹出栈，并保存到局部变量表中slot=2的位置
        14: getstatic     #6    //访问类的静态变量，类型为(java/lang/System.out:Ljava/io/PrintStream;)
        17: aload_2             //从局部变量表中装载slot=2的引用型数值，并压栈
        18: invokevirtual #7    //将栈顶的引用型数值弹出栈，并调用类成员方法，方法描述符为java/io/PrintStream.println:(Ljava/lang/Object;)V
        21: return
      LineNumberTable:
        line 10: 0
        line 11: 5
        line 12: 10
        line 13: 14
        line 14: 21
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      22     0  args   [Ljava/lang/String;
            5      17     1 objects   [Ljava/lang/Object;
           14       8     2 object   Ljava/lang/Object;
```





```
aaload
aastore
aconst_null
aload
aload_<n>
anewarray
areturn
arraylength
astore
astore_<n>
athrow
baload
bastore
bipush
caload
castore
checkcast
d2f
d2i
d2l
dadd
daload
dastore
dcmp<op>
dconst_<d>
ddiv
dload
dload_<n>
dmul
dneg
drem
dreturn
dstore
dstore_<n>
dsub
dup
dup_x1
dup_x2
dup2
dup2_x1
dup2_x2
f2d
f2i
f2l
fadd
faload
fastore
fcmp<op>
fconst_<f>
fdiv
fload
fload_<n>
fmul
fneg
frem
freturn
fstore
fstore_<n>
fsub
getfield
getstatic
goto
goto_w
i2b
i2c
i2d
i2f
i2l
i2s
iadd
iaload
iand
iastore
iconst_<i>
idiv
if_acmp<cond>
if_icmp<cond>
if<cond>
ifnonnull
ifnull
iinc
iload
iload_<n>
imul
ineg
instanceof
invokedynamic
invokeinterface
invokespecial
invokestatic
invokevirtual
ior
irem
ireturn
ishl
ishr
istore
istore_<n>
isub
iushr
ixor
jsr
jsr_w
l2d
l2f
l2i
ladd
laload
land
lastore
lcmp
lconst_<l>
ldc
ldc_w
ldc2_w
ldiv
lload
lload_<n>
lmul
lneg
lookupswitch
lor
lrem
lreturn
lshl
lshr
lstore
lstore_<n>
lsub
lushr
lxor
monitorenter
monitorexit
multianewarray
new
newarray
nop
pop
pop2
putfield
putstatic
ret
return
saload
sastore
sipush
swap
tableswitch
wide
```