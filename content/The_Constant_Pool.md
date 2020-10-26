+++
title = "JVM常量池小解"
date = 2018-10-16

[taxonomies]
tags = ["JVM", "JAVA"]
categories = ["JVM"]
+++

## 前言
今日,同事问我一个问题,以下java代码输出什么:
```java
String s1 = "hello world";
String s2 = "hello" + " " + "world";
System.out.println(s1 == s2); //true
```

沉思了一下,依我的编码经验给出了结果为true,辣么他接着问这是为什么,于是乎我就说常量是在编译期间存放在常量池中的,他接着问什么是常量池,都有什么东西存放在里面?

<!-- more -->

*综上所述*,引出了今天我们的问题**什么是常量池**?

## 定义

### [Run-Time Constant Pool](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.5)
>A run-time constant pool is a per-class or per-interface run-time representation of the constant_pool table in a class file. It contains several kinds of constants, ranging from numeric literals known at compile-time to method and field references that must be resolved at run-time. The run-time constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table.

**运行时常量池(Runtime Constant Pool)是每一个类或接口的常量池(Constant_Pool)的运行时表示形式,它包括了若干种不同的常量:从编译期可知的数值字面量到必须运行 期解析后才能获得的方法或字段引用。运行时常量池扮演了类似传统语言中符号表(Symbol Table)的角色，不过它存储数据范围比通常意义上的符号表要更为广泛。**

### [The Constant Pool](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4)
>Java Virtual Machine instructions do not rely on the run-time layout of classes, interfaces, class instances, or arrays. Instead, instructions refer to symbolic information in the constant_pool table.

**Java 虚拟机指令执行时不依赖与类、接口,实例或数组的运行时布局,而是依赖常量池(constant_pool)表中的符号信息**

以上是jvm规范中定义的,简单点来说:
在程序运行期间有块区域叫运行时常量池,运行时常量池里面存储着一些常量池结构的数据(每个结构里面都是class文件中的某种类型的字面量)

## 结构
```cpp
cp_info {
    u1 tag;
    u1 info[];
}
```

jvm规定了不同的tag值和不同类型的字面量对应关系,如下图所示:

|Constant Type| Value |
| :------: |:------: |
| CONSTANT_Class | 7 |
| CONSTANT_Fieldref| 9 |
| CONSTANT_Methodref | 10 |
| CONSTANT_InterfaceMethodref| 11 |
| CONSTANT_String | 8 |
| CONSTANT_Integer | 3 |
| CONSTANT_Float | 4 | 
| CONSTANT_Long | 5 |
| CONSTANT_Double | 6 |
| CONSTANT_NameAndType | 12 |
| CONSTANT_Utf8 | 1 |
| CONSTANT_MethodHandle | 15 |
| CONSTANT_MethodType | 16 |
| CONSTANT_InvokeDynamic | 18 |

此次只分析基本类型与String:

### CONSTANT_String_info
```cpp
CONSTANT_String_info {
    u1 tag;
    u2 string_index;
}
```
u1表示1个无符号字节,u2表示2个无符号字节
tag: 8
string_index: 指向CONSTANT_Utf8_info结构体

### CONSTANT_Utf8_info
```cpp
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```
u1表示1个无符号字节,u2表示2个无符号字节
tag: 1
length: utf-8编码的字节数组的长度
bytes[lenth]: utf-8编码的字节数组

### CONSTANT_Integer_info
```cpp
CONSTANT_Integer_info {
    u1 tag;
    u4 bytes;
}
```
u1表示1个无符号字节,u4表示4个无符号字节
tag: 3
bytes: int常量的值

### CONSTANT_Float_info
```cpp
CONSTANT_Float_info {
    u1 tag;
    u4 bytes;
}
```
u1表示1个无符号字节,u4表示4个无符号字节
tag: 4
bytes: float常量的值

### CONSTANT_Long_info
```cpp
CONSTANT_Long_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}
```
u1表示1个无符号字节,u4表示4个无符号字节
tag: 5


### CONSTANT_Double_info
```
CONSTANT_Double_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}
```
u1表示1个无符号字节,u4表示4个无符号字节
tag: 6

### bytecode op
有人可能就奇怪了,怎么少了byte/short/char/boolean这几个基本类型的呢?
jvm在此处有对应的指令支持:

| 类型 | 范围 | 指令 |
| :------: | :------: | :------: |
| boolean | [0, 1] | iconst_n |
| byte | [-128, 127] | iconst_m1, iconst_n, bipush |
| short | [-32768, 32767] | iconst_m1, iconst_n, bipush, sipush|
| char | [0, 65535] | iconst_m1,iconst_n, bipush, sipush, ldc |
| int | [-2147483648, 2147483647] | iconst_m1, iconst_n, bipush, sipush, ldc|

>byte,short,char其实都是int与boolean类似.也是在编译时就已经变成了int,然后执行int的一些操作符。

## 测试

```java
public static void main(String[] args) {
    // boolean
    boolean booltrue = true;
    boolean boolfalse = false;

    // byte
    byte byte_2 = -2;
    byte byte_1 = -1; // -1是 iconst_m1
    byte byte0 = 0;
    byte byte1 = 1;
    byte byte2 = 2;
    byte byte3 = 3;
    byte byte4 = 4;
    byte byte5 = 5;
    byte byte6 = 6; // [0, 5]是 iconst_n 的范围,超出部分用bipush

    // short
    short short_2 = -2;
    short short_1 = -1;
    short short0 = 0;
    short short1 = 1;
    short short2 = 2;
    short short3 = 3;
    short short4 = 4;
    short short5 = 5;
    short short6 = 6;
    short short12 = 12;
    short short127 = 127;
    short short128 = 128; //从 128开始,超出push的能力,换用sipush
    short short_32768 = -32768;
    short short32766 = 32766;
    short short32767 = 32767; //sipush能表示的范围和 short 一样

    // char
    char char0 = 0;
    char char1 = 1;
    char char2 = 2;
    char char3 = 3;
    char char4 = 4;
    char char5 = 5;
    char char6 = 6;
    char char12 = 12;
    char char127 = 127;
    char char128 = 128;
    char char32766 = 32766;
    char char32767 = 32767;
    char char32768 = 32768; //超出 sipush 能表示的范围, 所以只能用ldc
    char char65535 = 65535;

    // int
    int int_2 = -2;
    int int_1 = -1;
    int int0 = 0;
    int int1 = 1;
    int int2 = 2;
    int int3 = 3;
    int int4 = 4;
    int int5 = 5;
    int int6 = 6;
    int int_129 = -129;
    int int_128 = -128;
    int int_127 = -127;
    int int127 = 127;
    int int128 = 128;
    int int255 = 255;
    int int256 = 256;
    int int257 = 257;
    int int_32769 = -32769;
    int int_32768 = -32768;
    int int32766 = 32766;
    int int32767 = 32767;
    int int32768 = 32768;
    int int65534 = 65534;
    int int65535 = 65535;

    // long (除了0,1 均是ldc(ldc2_w))
    long long_2 = -2;
    long long_1 = -1;
    long long0 = 0; // lconst_0
    long long1 = 1; // lconst_1
    long long2 = 2;
    long long3 = 3;
    long long4 = 4;
    long long5 = 5;
    long long6 = 6;
    long long_129 = -129;
    long long_128 = -128;
    long long_127 = -127;
    long long127 = 127;
    long long128 = 128;
    long long255 = 255;
    long long256 = 256;
    long long257 = 257;
    long long_32769 = -32769;
    long long_32768 = -32768;
    long long32766 = 32766;
    long long32767 = 32767;
    long long32768 = 32768;
    long long65534 = 65534;
    long long65535 = 65535;
    long long165536 = 165536;
  }
```

javap结果
```cpp
Classfile /Users/oatiz/IdeaProjects/xxx-daas/daas-test/src/main/java/com/xxx/daas/test/StringTest.class
  Last modified 2018-10-16; size 1230 bytes
  MD5 checksum 99b3a33d2192b639186f350eacc0aac2
  Compiled from "StringTest.java"
public class com.xxx.daas.test.StringTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #53.#62        // java/lang/Object."<init>":()V
   #2 = Integer            32768
   #3 = Integer            65535
   #4 = Integer            -32769
   #5 = Integer            65534
   #6 = Long               -2l
   #8 = Long               -1l
  #10 = Long               2l
  #12 = Long               3l
  #14 = Long               4l
  #16 = Long               5l
  #18 = Long               6l
  #20 = Long               -129l
  #22 = Long               -128l
  #24 = Long               -127l
  #26 = Long               127l
  #28 = Long               128l
  #30 = Long               255l
  #32 = Long               256l
  #34 = Long               257l
  #36 = Long               -32769l
  #38 = Long               -32768l
  #40 = Long               32766l
  #42 = Long               32767l
  #44 = Long               32768l
  #46 = Long               65534l
  #48 = Long               65535l
  #50 = Long               165536l
  #52 = Class              #63            // com/xxx/daas/test/StringTest
  #53 = Class              #64            // java/lang/Object
  #54 = Utf8               <init>
  #55 = Utf8               ()V
  #56 = Utf8               Code
  #57 = Utf8               LineNumberTable
  #58 = Utf8               main
  #59 = Utf8               ([Ljava/lang/String;)V
  #60 = Utf8               SourceFile
  #61 = Utf8               StringTest.java
  #62 = NameAndType        #54:#55        // "<init>":()V
  #63 = Utf8               com/xxx/daas/test/StringTest
  #64 = Utf8               java/lang/Object
{
  public com.xxx.daas.test.StringTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 6: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=115, args_size=1
         0: iconst_1
         1: istore_1
         2: iconst_0
         3: istore_2
         4: bipush        -2
         6: istore_3
         7: iconst_m1
         8: istore        4
        10: iconst_0
        11: istore        5
        13: iconst_1
        14: istore        6
        16: iconst_2
        17: istore        7
        19: iconst_3
        20: istore        8
        22: iconst_4
        23: istore        9
        25: iconst_5
        26: istore        10
        28: bipush        6
        30: istore        11
        32: bipush        -2
        34: istore        12
        36: iconst_m1
        37: istore        13
        39: iconst_0
        40: istore        14
        42: iconst_1
        43: istore        15
        45: iconst_2
        46: istore        16
        48: iconst_3
        49: istore        17
        51: iconst_4
        52: istore        18
        54: iconst_5
        55: istore        19
        57: bipush        6
        59: istore        20
        61: bipush        12
        63: istore        21
        65: bipush        127
        67: istore        22
        69: sipush        128
        72: istore        23
        74: sipush        -32768
        77: istore        24
        79: sipush        32766
        82: istore        25
        84: sipush        32767
        87: istore        26
        89: iconst_0
        90: istore        27
        92: iconst_1
        93: istore        28
        95: iconst_2
        96: istore        29
        98: iconst_3
        99: istore        30
       101: iconst_4
       102: istore        31
       104: iconst_5
       105: istore        32
       107: bipush        6
       109: istore        33
       111: bipush        12
       113: istore        34
       115: bipush        127
       117: istore        35
       119: sipush        128
       122: istore        36
       124: sipush        32766
       127: istore        37
       129: sipush        32767
       132: istore        38
       134: ldc           #2                  // int 32768
       136: istore        39
       138: ldc           #3                  // int 65535
       140: istore        40
       142: bipush        -2
       144: istore        41
       146: iconst_m1
       147: istore        42
       149: iconst_0
       150: istore        43
       152: iconst_1
       153: istore        44
       155: iconst_2
       156: istore        45
       158: iconst_3
       159: istore        46
       161: iconst_4
       162: istore        47
       164: iconst_5
       165: istore        48
       167: bipush        6
       169: istore        49
       171: sipush        -129
       174: istore        50
       176: bipush        -128
       178: istore        51
       180: bipush        -127
       182: istore        52
       184: bipush        127
       186: istore        53
       188: sipush        128
       191: istore        54
       193: sipush        255
       196: istore        55
       198: sipush        256
       201: istore        56
       203: sipush        257
       206: istore        57
       208: ldc           #4                  // int -32769
       210: istore        58
       212: sipush        -32768
       215: istore        59
       217: sipush        32766
       220: istore        60
       222: sipush        32767
       225: istore        61
       227: ldc           #2                  // int 32768
       229: istore        62
       231: ldc           #5                  // int 65534
       233: istore        63
       235: ldc           #3                  // int 65535
       237: istore        64
       239: ldc2_w        #6                  // long -2l
       242: lstore        65
       244: ldc2_w        #8                  // long -1l
       247: lstore        67
       249: lconst_0
       250: lstore        69
       252: lconst_1
       253: lstore        71
       255: ldc2_w        #10                 // long 2l
       258: lstore        73
       260: ldc2_w        #12                 // long 3l
       263: lstore        75
       265: ldc2_w        #14                 // long 4l
       268: lstore        77
       270: ldc2_w        #16                 // long 5l
       273: lstore        79
       275: ldc2_w        #18                 // long 6l
       278: lstore        81
       280: ldc2_w        #20                 // long -129l
       283: lstore        83
       285: ldc2_w        #22                 // long -128l
       288: lstore        85
       290: ldc2_w        #24                 // long -127l
       293: lstore        87
       295: ldc2_w        #26                 // long 127l
       298: lstore        89
       300: ldc2_w        #28                 // long 128l
       303: lstore        91
       305: ldc2_w        #30                 // long 255l
       308: lstore        93
       310: ldc2_w        #32                 // long 256l
       313: lstore        95
       315: ldc2_w        #34                 // long 257l
       318: lstore        97
       320: ldc2_w        #36                 // long -32769l
       323: lstore        99
       325: ldc2_w        #38                 // long -32768l
       328: lstore        101
       330: ldc2_w        #40                 // long 32766l
       333: lstore        103
       335: ldc2_w        #42                 // long 32767l
       338: lstore        105
       340: ldc2_w        #44                 // long 32768l
       343: lstore        107
       345: ldc2_w        #46                 // long 65534l
       348: lstore        109
       350: ldc2_w        #48                 // long 65535l
       353: lstore        111
       355: ldc2_w        #50                 // long 165536l
       358: lstore        113
       360: return
```
