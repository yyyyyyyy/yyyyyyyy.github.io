---
layout: post
title:  "【介绍】Java ClassFile Structure"
date:   2015-05-19 00:00:00 +0800
categories: 
comments: true
excerpt: 

---


一个编译后的class文件包含下面的结构：

```
ClassFile {
    u4:magic;             //魔数
    u2:minor_version;     //副版本号
    u2:major_version;     //主版本号
    u2:constant_pool_count;       //常量池计数器
    cp_info:constant_pool[constant_pool_count -1];     //常量池
    u2:access_flags;      //访问标识
    u2:this_class;        //类索引
    u2:super_class;       //父类索引
    u2:interfaces_count;   //接口计数器
    u2:interfaces[interfaces_count];  //接口表
    u2:fields_count;       //字段计数器
    field_info:fields[fields_count];  //字段表
    u2:methods_count;      //方法计数器
    method_info:methods[methods_count];  //方法表
    u2:attributes_count;   //属性计数器
    attribute_info:attributes[attributes_count];   //属性表
}
```

#### magic u4

魔数，固定值为0xCAFEBABE。作用是确定此Class文件是否可以被虚拟机所接受。

#### minor_version u2、magor_version u2

副版本号、主版本号  00 00 00 33 代表副版本号是0，主版本号51 JDK1.7。
Class文件能够被相同或高版本加载，不能被低版本加载。jdk中版本号是从45开始的，所以51代表的是1.7版本。

#### constant_pool_count u2、cp_info:constant_pool[constant_pool_count -1] 

常量池计数器，因为常量池中的数据不是固定的，所以需要一个计数器。
计数器是u2（2个byte），所以constant_pool_count不能超过16的四次幂-1，即65535。
常量池计数器默认从1开始，而不是0，即当constant_pool_count = 1时，常量池中cp_info的个数为0；当cp_info的个数为n时，常量池中的cp_info个数为n-1.
在指定Class文件规范的时候，将索引#0项常量空出来是有特殊考虑的，使用索引#0表达“不引用任何一个常量池项”的意思。

constant_pool中主要存放着两种常量，字面量和符号引用。

- 字面量：文本字符、声明为final的常量值，基础数据类型的值。
- 符号引用：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。
  - 全限定名中的“.”会被“/”替代，存储进常量池中。
  - 符号引用&直接引用：符号引用在javac编译时产生，完全符合Java虚拟机规范；编译产生Class文件，而Class文件此时还没有被JVM加载，此时的符号引用仅仅是一个标识，符号引用并不知道其对应的数据放在哪一处内存；当JVM加载Class文件时，会将具体的信息提取出来放入内存，进而就知道了哪一条信息处于哪一处内存，由于符号引用完全符合java虚拟机规范，所以这时根据每一处信息的内容，逆推出该信息对应的符号引用，然后再使用该信息的实际的内存地址来替代符号引用。

cp_info又可以分为14种结构类型：

| 类型                             | tag (u1) | 说明                                                         |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| CONSTANT_Utf8_info               | 1        | utf-8编码的字符串占用的字节数length u2\|字符串 length        |
| CONSTANT_Integer_info            | 3        | 按照高位在前存储的int值 u4                                   |
| CONSTANT_Float_info              | 4        | 按照高位在前存储的float值 u4                                 |
| CONSTANT_Long_info               | 5        | 按照高位在前存储的long值 high_bytes u4 low_types u4          |
| CONSTANT_Long_info               | 6        | 按照高位在前存储的double值 high_bytes u4 low_types u4        |
| CONSTANT_Class_info              | 7        | index（指向全限定名常量项的索引） u2                         |
| CONSTANT_String_info             | 8        | index（指向字符串字面量的索引的索引） u2                     |
| CONSTANT_Fieldref_info           | 9        | index（指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项） u2，index（指向字段描述符CONSTANT_NameAndType_info的索引项） u2 |
| CONSTANT_Methodref_info          | 10       | index（指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项） u2，index（指向字段描述符CONSTANT_NameAndType_info的索引项） u2 |
| CONSTANT_InterfaceMethodref_info | 11       | index（指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项） u2，index（指向字段描述符CONSTANT_NameAndType_info的索引项） u2 |
| CONSTANT_NameAndType_info        | 12       | index（指向该字段活方法名称常量项的索引） u2，index（指向该字段或方法描述符的索引） u2 |
| CONSTANT_MethodHandle_info       | 15       | reference_kind（值必须在[1,9]之间，它决定了方法句柄的类型，方法句柄类型的值表示方法句柄的字节码行为） u2，reference_index（值必须是对常量池的有效引用） u2 |
| CONSTANT_MethodType_info         | 16       | descriptor_index（值必须是对常量池的有效引用，常量池在索引处的项必须是CONSTANT_Utf8_info结构，表示方法的描述符） u2 |
| CONSTANT_InvokeDynamic_info      | 18       | bootstrap_method_attr_index（值必须是对当前Class文件中引导方法表的bootstrap methods[]数据的有效索引） u2， name_and_type_index（值必须是对当前常量池的有效索引，常量池在该处的索引必须是CONSTANT_NameAndType_info结构，表示方法名和方法描述符） |

#### access_flags u2

| 类型          | 值     | 说明                                          |
| ------------- | ------ | --------------------------------------------- |
| ACC_PUBLIC    | 0x0001 | 是否为public类型                              |
| ACC_FINAL     | 0x0010 | 是否声明final                                 |
| ACC_SUPER     | 0x0020 | 是否允许使用invokespicial字节码指令的新语意。 |
| ACC_INTERFACE | 0x0200 | 是否为接口                                    |
| ACC_ABSTRACT  | 0x0400 | 是否为抽象类                                  |
| ACC_SYNTHETIC | 0x1000 | 标识这个类并非由用户代码生成访问标志          |

多个标识符access_flags的值累加。

#### this_class u2

类索引，确定当前类的全限定名

#### super_class u2

父类索引，确定当前类的父类全限定名

#### interfaces_count， interfaces[interfaces_count]

接口索引计数器、接口索引集合

#### fields_count u2|field_info:fields[fields_count]

field_info用于描述接口或者类中声明的变量。字段包括静态变量和实例变量，不包括方法内部的局部变量。
field_info的结构为：access_flags u2|name_index u2|descriptor_index u2|attributes_count u2|attributes attributes_count.

##### access_flags 

| 类型          | 值     | 说明                     |
| ------------- | ------ | ------------------------ |
| ACC_PUBLIC    | 0x0001 | 字段是否public           |
| ACC_PRIVATE   | 0x0002 | 字段是否private          |
| ACC_PROTECTED | 0x0004 | 字段是否protected        |
| ACC_STATIC    | 0x0008 | 字段是否static           |
| ACC_FINAL     | 0x0010 | 字段是否final            |
| ACC_VOLATILE  | 0x0040 | 字段是否volatile         |
| ACC_TRANSIENT | 0x0080 | 字段是否transient        |
| ACC_SYNTHETIC | 0x1000 | 字段是否由编译器自动产生 |
| ACC_ENUM      | 0x4000 | 字段是否enum             |

##### name_index

field的自定义名称，字段名

##### descriptor_index

字段类型的描述

| 类型                 | 说明                     |
| -------------------- | ------------------------ |
| B                    | 基本类型byte             |
| C                    | CHAR                     |
| D                    | double                   |
| F                    | float                    |
| I                    | int                      |
| J                    | long                     |
| S                    | short                    |
| Z                    | boolean                  |
| V                    | void                     |
| L + 类全限定名 + “;” | 比如：Ljava/lang/String; |

数组类型的描述，每一维度需要在前面加一个“[”;如：String[][],将被记录为“[[java/lang/String;”,int[]将被记录为“[I”



#### methods_count u2，method_info:methods[methods_count]

method_info结构与field_info是一样的。



#### attributes_count，attribute_info:attributes[attributes_count]

attribute_name_index u2|attribute_length u4|info attribute_length

Class文件、字段表、方法表都可以携带自己的属性表集合，以描述某些场景专有的信息。attributes也可以嵌套attributes。

| 类型                                 | 值                       | 说明                                                         |
| ------------------------------------ | ------------------------ | ------------------------------------------------------------ |
| Code                                 | 方法表中                 | Java代码编译成的字节码指令                                   |
| ConstantValue                        | 字段表中                 | final关键字定义的常量值                                      |
| Deprecated                           | 类、方法表、字段表       | 被声明为deprecated的方法和字段                               |
| Exceptions                           | 方法表中                 | 方法声明的异常                                               |
| LocalVariableTable                   | Code属性中               | 方法的局部变量描述                                           |
| LocalVariableTypeTable               | 类中                     | DK1.5中新增的属性，它使用特征签名代替描述符，是为了引入泛型语法之后能描述泛型参数化类型而添加 |
| innerClasses                         | 类中                     | 内部类列表                                                   |
| EnclosingMethod                      | 类中                     | 仅当一个类为局部类或者匿名类时，才能拥有这个属性，这个属性用于表示这个类所在的外围方法 |
| LineNumberTable                      | Code属性中               | Java源码的行号与字节码指令的对应关系                         |
| StackMapTable                        | Code属性中               | JDK1.6中新增的属性，供新的类型检查验证器(Type Checker)检查和处理目标方法的局部变量和操作数栈所需要的类型是否匹配 |
| Signature                            | 类中、方法表中、字段表中 | DK1.5新增的属性，这个属性用于支持泛型情况下的方法签名，在Java语言中，任何类、接口、初始化方法或成员的泛型签名如果包含了类型变量(Type Variables)或参数类型(Parameterized Types),则Signature属性会为它记录泛型签名信息。由于Java的泛型采用擦除法实现，在为了避免类型信息被擦除后导致签名混乱，需要这个属性记录泛型中的相关信息 |
| SourceFile                           | 类中                     | 记录源文件名称                                               |
| SourceDebugExtension                 | 类中                     | JDK1.6中新增的属性，SourceDebugExtension用于存储额外的调试信息。如在进行JSP文件调试时，无法通过Java堆栈来定位到JSP文件的行号，JSR-45规范为这些非Java语言编写，却需要编译成字节码运行在Java虚拟机汇中的程序提供了一个进行调试的标准机制，使用SourceDebugExtension就可以存储这些调试信息。 |
| Synthetic                            | 类中、方法表中、字段表中 | 标识方法或字段为编译器自动产生的                             |
| RuntimeVisibleAnnotations            | 类中、方法表中、字段表中 | JDK1.5中新增的属性，为动态注解提供支持。RuntimeVisibleAnnotations属性，用于指明哪些注解是运行时(实际上运行时就是进行反射调用)可见的。 |
| RuntimeInvisibleAnnotations          | 类中、方法表中、字段表中 | JDK1.5中新增的属性，作用与RuntimeVisibleAnnotations相反用于指明哪些注解是运行时不可见的。 |
| RuntimeVisibleParameterAnnotations   | 方法表中                 | JDK1.5中新增的属性，作用与RuntimeVisibleAnnotations类似，只不过作用对象为方法的参数。 |
| RuntimeInvisibleParameterAnnotations | 方法表中                 | JDK1.5中新增的属性，作用与RuntimeInvisibleAnnotations类似，只不过作用对象为方法的参数。 |
| AnnotationDefault                    | 方法表中                 | JDK1.5中新增的属性，用于记录注解类元素的默认值               |
| BootstrapMethods                     | 类中                     | JDK1.7新增的属性，用于保存invokedynamic指令引用的引导方法限定符 |

### 总结
java文件编译后生成class文件，class文件按照固定的规则依次表示不同的功能数据供JVM识别。
JVM通过解析各项表示不同功能的信息进行对象的生成，方法的调用等等。

以上，看完后相信对类加载会有更深层次的理解。