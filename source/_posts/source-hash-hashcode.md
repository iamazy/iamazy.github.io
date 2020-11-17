---
title: HashCode
date: 2020-03-15 08:38:58
tags: [jdk,java,源码]
categories: jdk-tutorial
---

# HashCode

### 1. hashCode声明
```java
/**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     *
     * @implSpec
     * As far as is reasonably practical, the {@code hashCode} method defined
     * by class {@code Object} returns distinct integers for distinct objects.
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
    @HotSpotIntrinsicCandidate
    public native int hashCode();
```
> 根据注释我们可以知道几点：(已知存在x，y两个对象)
```
1. Object类的hashCode函数是本地方法
2. 如果x.equals(y)==true，则x，y的hashCode一定相等
3. 如果x.equals(y)==false,则x，y的hashCode有可能相等
4. 如果x.equals(y)==false,且x,y的hashCode不相等会提高hash表查找性能
5. 如果子类重写equals方法，建议同时重写hashCode方法，反之亦然
```
### 2. 常见生成hashCode的方式
```java
1. Objects.hash(Object... values);
2. System.identityHashCode(Object x);
3. HashCodeBuilder.reflectionHashCode(Object x);
```
> 示例：生成字符串"Aa","BB"的hashCode
```java
"Aa".hashCode()                                  // hashCode -> 2112
"BB".hashCode()                                  // hashCode -> 2112
Objects.hash("Aa")                               // hash -> 2143
Objects.hash("BB")                               // hash -> 2143
System.identityHashCode("Aa")                    // hashCode -> 1528902577
System.identityHashCode("BB")                    // hashCode -> 1927950199
HashCodeBuilder.reflectionHashCode("Aa")         // hashCode -> 1305660456
HashCodeBuilder.reflectionHashCode("BB")         // hashCode -> 1305964374
```

### 3. String类的hashCode实现
```java
/**
     * Returns a hash code for this string. The hash code for a
     * {@code String} object is computed as
     * <blockquote><pre>
     * s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
     * </pre></blockquote>
     * using {@code int} arithmetic, where {@code s[i]} is the
     * <i>i</i>th character of the string, {@code n} is the length of
     * the string, and {@code ^} indicates exponentiation.
     * (The hash value of the empty string is zero.)
     *
     * @return  a hash code value for this object.
     */
    public int hashCode() {
        // The hash or hashIsZero fields are subject to a benign data race,
        // making it crucial to ensure that any observable result of the
        // calculation in this method stays correct under any possible read of
        // these fields. Necessary restrictions to allow this to be correct
        // without explicit memory fences or similar concurrency primitives is
        // that we can ever only write to one of these two fields for a given
        // String instance, and that the computation is idempotent and derived
        // from immutable state
        int h = hash;
        if (h == 0 && !hashIsZero) {
            h = isLatin1() ? StringLatin1.hashCode(value)
                           : StringUTF16.hashCode(value);
            if (h == 0) {
                hashIsZero = true;
            } else {
                hash = h;
            }
        }
        return h;
    }
```
> 根据注释可知
```
1. String的hashCode计算公式：s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```
> StringLatin1.hashCode(byte[] value)实现
```java
public static int hashCode(byte[] value) {
    int h = 0;
    for (byte v : value) {
        h = 31 * h + (v & 0xff);
    }
    return h;
}
```
>选择数字31是因为它是一个奇质数，如果选择一个偶数会在乘法运算中产生溢出，导致数值信息丢失，因为乘二相当于移位运算。选择质数的优势并不是特别的明显，但这是一个传统。同时，数字31有一个很好的特性，即乘法运算可以被移位和减法运算取代，来获取更好的性能：`31 * i == (i << 5) - i`，现代的 Java 虚拟机可以自动的完成这个优化。 
31是质子数中一个“不大不小”的存在，如果你使用的是一个如2的较小质数，那么得出的乘积会在一个很小的范围，很容易造成哈希值的冲突。而如果选择一个100以上的质数，得出的哈希值会超出int的最大范围，这两种都不合适。如果你对超过50000个英文单词（由两个不同版本的Unix字典合并而成进行hashcode运算，并使用常数`31`,`33`,`37`,`39`和`41`作为乘子，每个常数算出的哈希值冲突数都小于7个，所以在上面几个常数中，常数`31`被 Java实现所选用也就不足为奇了
### 4. 0xff问题

> 计算机中存储的数据都是以补码的形式存储的
>> 原码
>>> 正数：（十进制）正数转换成二进制就是该正数的原码</br> 负数：（十进制）负数的绝对值转换成二进制然后在高位补1就是该负数的原码

>> 反码
>>> 正数：（十进制）正数的反码与原码相同</br>负数：（十进制）负数的反码等于原码除符号位以外所有的位取反

>> 补码
>>> 正数：（十进制）正数的补码与原码相同</br>负数：（十进制）负数的补码等于负数的原码最低位加1

|num|原码|反码|补码|
|---|---|---|---|
|128|1000 0000|1000 0000|1000 0000|
|-127|1111 1111|1000 0000|1000 0001|

#### a. 为什么要v&0xff？</br>
> &表示按位与，只有两个数同时为1，才能得到1</br>0xff表示的二进制数是`1111 1111`,占一个字节，和其进行&操作的数，最低8位，不会发生变化
##### 1. 示例
当byte类型的数字（如-127）转换为int类型的时候，其补码将提升到32位，补码的高位补1
> (byte)-127 -> (int)-127 时
```
1000 0001
转
1111 1111 1111 1111 1111 1111 1000 0001
```
> 负数的补码转原码，符号位不变，其他位取反，然后最低位加1
```
1111 1111 1111 1111 1111 1111 1000 0001 (-127补码高24位补1)
转
1000 0000 0000 0000 0000 0000 0111 1111 (int类型的32位-127的原码)
```
可以看出当byte->int时可以保证十进制不变

但是有时候如文件流转为byte数组的时候，我们不关心对应的十进制数有没有变，而是关心对应的补码有没有变，这时候就需要加上&0xff。

上例中，byte->int高24位必将补1，此时补码显然发生变化，再&0xff，将高24位重新置0，这样就能保证补码的一致性，由于符号位发生变化，表示的十进制数也会改变。

```
1111 1111 1111 1111 1111 1111 1000 0001
&
0000 0000 0000 0000 0000 0000 1111 1111 
结果：
0000 0000 0000 0000 0000 0000 1000 0001 -> 正数的补码==原码 -> 129
```
和原来的补码一致，但是显然符号位变了，表示的十进制发生变化，变为129
##### 2.总结
> 取得低8位 
> 保持补码的一致性，但是表示的十进制数可能会变

### 5. 为什么hashCode可能会相同

> 哈希表结合了**直接寻址**和**链式寻址**两种方式，所需要的就是将需要加入哈希表的数据首先计算哈希值，预先分组，然后再将数据挂到分组后的链表中，随着添加的数据越来越多，分组链上会挂接更多的数据，同一个分组链上的数据必定具有相同的哈希值，因为java的hash函数返回值是int类型，也就是说，最多允许存在2^32个分组，也是有限的，所以很容易出现相同的hash值。
