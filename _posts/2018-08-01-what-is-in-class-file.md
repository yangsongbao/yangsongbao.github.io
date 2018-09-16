

我们都知道java代码在运行时首先要编程成字节码，然后由jvm来执行,那字节码长什么样子？jvm又是如何执行字节码的呢？首先我们来看下字节码长什么样子。有如下的java代码：
``` java
package javalearn;

public class BytecodeTest {

    public static void main(String[] args) {
        int i = 10;
        int j = i + 10;
        System.out.println(j);
    }
}
```
这是一段非常简单的java代码，我们在该java文件所在的目录下执行如下编译命令：
``` java
javac BytecodeTest.java
```
在相同的目录下得到一个BytecodeTest.class文件，这就是字节码文件了。每个 Class 文件都是由8字节为单位的字节流组成，所有的16位、32 位和64位长度的数
据将被构造成2个、4个和8个8 字节单位来表示。多字节数据项总是按照 Big-Endian①的顺序进行存储。当我们用notepad++类似的软件打开该class文件，并以hex形式展示时，可以看到如下的内容：

![image](https://note.youdao.com/yws/public/resource/fa5911e8c2da692662f6577e984562da/xmlnote/A024F7E85A984F86A23C8A2E1345C279/1725)

是不是一脸懵逼，没关系，后面慢慢解释。[按照jvm规范](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.1)，class文件的格式如下
``` java
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
其中u1,u2,u4是jvm规范定义的一组私有数据类型，用来表示Class文件内容，分别代表1、2、4个字节的无符号数。
那么接下来就是要将这个文件结构跟上面的字节码文件内容对应起来，这样硬看是一件不太可能的事，jdk提供了一个工具反编译字节码的内容，以class文件结构的形式展示出来。在class文件所在目录执行以下命令
``` java
 javap -verbose BytecodeTest.class
```
会得到如下结果
``` java
Classfile /D:/project/Learn/javalearn/bytecode/src/main/java/javalearn/BytecodeTest.class
  Last modified 2018-8-8; size 420 bytes
  MD5 checksum 5004b1c89fe3f326100bf32ae6a3da13
  Compiled from "BytecodeTest.java"
public class javalearn.BytecodeTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#14         // java/lang/Object."<init>":()V
   #2 = Fieldref           #15.#16        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #17.#18        // java/io/PrintStream.println:(I)V
   #4 = Class              #19            // javalearn/BytecodeTest
   #5 = Class              #20            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               main
  #11 = Utf8               ([Ljava/lang/String;)V
  #12 = Utf8               SourceFile
  #13 = Utf8               BytecodeTest.java
  #14 = NameAndType        #6:#7          // "<init>":()V
  #15 = Class              #21            // java/lang/System
  #16 = NameAndType        #22:#23        // out:Ljava/io/PrintStream;
  #17 = Class              #24            // java/io/PrintStream
  #18 = NameAndType        #25:#26        // println:(I)V
  #19 = Utf8               javalearn/BytecodeTest
  #20 = Utf8               java/lang/Object
  #21 = Utf8               java/lang/System
  #22 = Utf8               out
  #23 = Utf8               Ljava/io/PrintStream;
  #24 = Utf8               java/io/PrintStream
  #25 = Utf8               println
  #26 = Utf8               (I)V
{
  public javalearn.BytecodeTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: bipush        10
         2: istore_1
         3: iload_1
         4: bipush        10
         6: iadd
         7: istore_2
         8: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: iload_2
        12: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        15: return
      LineNumberTable:
        line 6: 0
        line 7: 3
        line 8: 8
        line 9: 15
}
SourceFile: "BytecodeTest.java"
```
下面我们来对着javap的结果看Class文件的结构。
###### magic：
  魔数，他的唯一作用是确定这个文件是否为一个能被虚拟机所接受的Class文件，固定为0xCAFEBABE，不会改变。根据Class文件结构，magic的类型为u4，占4个字节。

  对应上述的二进制文件中可以看到，文件的前四个字节就是CAFEBABE。
###### minor_version、 major_version：
  副版本号和主版本号， minor_version 和 major_version 的值分别表示 Class 文件的副、主版本。一个Java虚拟机实例只能支持特定范围内的主版本号（Mi 至 Mj）和0至特定范围内（0至m）的副版本号。不同版本的Java虚拟机实现支持的版本号也不同，高版本号的Java虚拟机实现可以支持低版本号的Class文件，反之则不成立。根据Class文件结构，接下来的4个字节为副版本号和主版本号。

  对应上述的二进制文件，0000为副版本号，转换成十进制为0；0034为主版本号，按大端序将十六进制转换为十进制为52；所以版本号为52.0。

###### constant_pool_count
常量池计数器，constant_pool_count的值等于constant_pool表中的成员数加 1。类型为u2，也就是2个字节。

对应上述的二进制文件也就是001b，转换为十进制为27。也就是有27个常量。

###### constant_pool[]
常量池， constant_pool是一种表结构，它包含Class文件结构及其子结构中引用的所有字符串常量、 类或接口名、字段名和其它常量。常量池中的每一项都具备相同的格式特征——第一个字节作为类型标记用于识别该项是哪种类型的常量，称为“tagbyte”。 常量池的索引范围是1至constant_pool_count−1。

对应上述javap的结果，可以看到有一个constant pool的项，下面列举了从#1到#27的常量。

按照虚拟机规范定义，常量有如下类型
![image](https://note.youdao.com/yws/public/resource/fa5911e8c2da692662f6577e984562da/xmlnote/DD4B156B9C5C4A0491F5BD1FED70976B/1780)

我们以常量池中的第一个常量#1为例来说明，根据javap的结果，#1的类型CONSTANT_Methodref_info，根据jvm规范，该类型的结构如下
``` java
CONSTANT_Methodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```
其中用一个字节tag来表示类型。前面对于二进制文件，我们已经分析到了坐标为（00000000，9）的位置，按照规范，（00000000，10）的位置应该就是常量类型了，该位置的值为0a，等于CONSTANT_Methodref_info的值10。紧接着的两个字节为class_index，值为0005，十进制为5，紧接着两个字节为name_and_type_index，值为000e，值为14；5和14均为常量池的位置引用，代表指向常量池#5和#14位置的值。对应javap的结果
``` java
#1 = Methodref          #5.#14
```
可以看到，跟我们的分析结果是对应的。按照这个方法，对应这jvm规范可以对常量池中的常量进行一一的分析。最后一个常量#27结束的坐标为（00000120，0），下面接着分析Class文件结构中的下一个access_flags

###### access_flags
访问标志， access_flags是一种掩码标志，用于表示某个类或者接口的访问权限及基础属性。值和含义如下：

| 标记名 | 值 | 含义|
| :----- | -- | :-: |
| ACC_PUBLIC | 0x0001 | 可以被包的类外访问|
| ACC_FINAL  | 0x0010 | 不允许有子类|
| ACC_SUPER  | 0x0020 | 当用到 invokespecial 指令时，需要特殊处理的父类方法|
| ACC_INTERFACE  | 0x0200 | 标识定义的是接口而不是类|
| ACC_ABSTRACT  | 0x0400 | 不能被实例化|
| ACC_SYNTHETIC  | 0x1000 | 标识并非 Java 源码生成的代码|
按照文件结构为两个字节长度，为坐标（00000120，0-1）的值，为0021，按照上述表格，即为ACC_PUBLIC和ACC_SUPER的组合，与javap结果中的一致。
``` java
flags: ACC_PUBLIC, ACC_SUPER
```

###### this_class super_class
接下来的四个字节，两个字节为this_class，类索引，他的值必须是对 constant_pool 表中项目的一个有效索引值。另外两个字节为super_class，父类索引，对于类来说， super_class的值必须为0或者是对 constant_pool 表中
项目的一个有效索引值。this_class的坐标为（00000120，3-4），值为0004，十进制为4，代表常量池中的#4，从javap的结果看出#4是个Class类型，类名字为javalearn/BytecodeTest，即当前类的名字；super_class的坐标为（00000120，5-6），值为0005，十进制为5，代表常量池中的#5，从javap的结果看出#4是个Class类型，类名字为java/lang/Object，为当前类的默认父类。

###### interfaces_count和interfaces[interfaces_count]
interfaces_count表示当前类或接口的直接父接口数量，两个字节，坐标为（00000120，7-8），值为0000，也就是0，很显然当前类没有父接口。

###### fields_count和fields[fields_count]
fields_count的值表示当前 Class 文件 fields[]数组的成员个数，两个字节，坐标为（00000120，9-a）,值为0000，也就是0，很显然当前类没有成员变量。

###### methods_count
方法计数器， methods_count的值表示当前Class文件methods[]数组的成员个数，两个字节，坐标为（00000120，b-c），值为0002，也就是有2个方法。

###### methods[]
方法表， methods[]数组中的每个成员都必须是一个method_info结构（如下）的数据项，用于表示当前类或接口中某个方法的完整描述。
``` java
method_info {
    u2 access_flags;
    u2 name_index;
    u2 descriptor_index;
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```
access_flags项的值是用于定义当前方法的访问权限和基本属性的掩码标志，取值范围和相应含义如下。
| 标记名 | 值 | 说明|
| :----- | -- | :-: |
|ACC_PUBLIC| 0x0001| public方法可以从包外访问|
|ACC_PRIVATE| 0x0002| private， 方法只能本类中访问|
|ACC_PROTECTED| 0x0004| protected， 方法在自身和子类可以访问|
|ACC_STATIC| 0x0008| static， 静态方法|
|ACC_FINAL| 0x0010| final， 方法不能被重写（覆盖）|
|ACC_SYNCHRONIZED| 0x0020| synchronized， 方法由管程同步|
|ACC_BRIDGE| 0x0040| bridge， 方法由编译器产生|
|ACC_VARARGS| 0x0080| 表示方法带有变长参数|
|ACC_NATIVE| 0x0100| native， 方法引用非 java 语言的本地方法|
|ACC_ABSTRACT| 0x0400| abstract， 方法没有具体实现|
|ACC_STRICT| 0x0800| strictfp， 方法使用 FP-strict 浮点格式|
|ACC_SYNTHETIC| 0x1000| 方法在源文件中不出现，由编译器产生|
两个字节，坐标为（00000120，d-e），值为0001，对应ACC_PUBLIC。

name_index为常量池引用，两个字节，坐标为（00000120，f）和（00000130，0），值为0006，对应#6为<init>，这个方法每个类都有，为类的初始化方法的名字。

descriptor_index为常量池引用，两个字节，坐标为（00000130，1-2），值为0007，对应#7为()V，为jvm定义的一个方法描述符，这里表示该方法无参，返回值为void。

attributes_count的项的值表示这个方法的附加属性的数量。两个字节，坐标为（00000130，3-4），值为0001，也就是只有一个属性。

attribute_info attributes[attributes_count]是一个数组，包含了每个属性。本例只有一个属性Code，其格式定义如下：
``` java
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    { u2 start_pc;
    u2 end_pc;
    u2 handler_pc;
    u2 catch_type;
    } exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```
attribute_name_index为属性名在常量池中的索引，两个字节，坐标（00000130，5-6），值为0x0008，代表#8，值为Code，即证明是Code属性。attribute_length坐标为（00000130，7-a），值为0x0000001d，十进制为29。

max_stack 项的值给出了当前方法的操作数栈在运行执行的任何时间点的最大深度，坐标为（00000130，b-c），值为0x0001，即为1。

max_locals 项的值给出了分配在当前方法引用的局部变量表中的局部变量个数，包括调用此方法时用于传递参数的局部变量，坐标为（00000130，d-e），值为0x0001，即为1。

code_length 项给出了当前方法的code[]数组的字节数，坐标为（00000130，f）--（00000140，2），值为0x0005，十进制为5，即接下来code[]数组有5个字节。

code[]数组给出了实现当前方法的Java虚拟机字节码，根据code_length的值，code[]数组的坐标为（00000140，3-7），值为0x2ab70001b1，至于这些字符代表什么，又有另一套规则，这里先按下不表，只需要记住这些字符代表了方法体的指令。

exception_table_length项的值给出了exception_table[]数组的成员个数量，坐标为（00000140，8-9），值为0x0000，也就是该方法没有异常处理器。

exception_table[]数组的每个成员表示code[]数组中的一个异常处理器，这里没有，先不做介绍。

接下里又是attributes_count，你会奇怪为啥又是他，这里其实表明了附加属性值是可以嵌套的，前面讲方法有附加属性，这里说明附加属性也可以有附加属性，回过头去看Class文件的结构会发现也有附加属性值，该例子中坐标为（00000140，a-b），值为0x0001，也就是Code属性还有一个附加属性，按照分析Code属性的例子，可以分析出该属性为的attribu坐标为（00000140，c-d），值为0x0009，代表#9，即为LineNumberTable，对照jvm规范，该属性的结构如下：
``` java
LineNumberTable_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 line_number_table_length;
    {
        u2 start_pc;
        u2 line_number;
    } line_number_table[line_number_table_length];
}
```
attribute_name_index 已经说过。

attribute_length的坐标为（00000140，e-f）-（00000150，0-1），值为0x00000006，即属性长度为6，为接下来的6个字节。

line_number_table_length坐标（00000150，2-3），值为0x0001。

line_number_table[]数组的每个成员都表明源文件中行号的变化在code[]数组中
都会有对应的标记点，这个数组里每个元素的结构包含start_pc和line_number。

start_pc 项的值必须是 code[]数组的一个索引， code[]数组在该索引处的字符
表示源文件中新的行的起点，坐标为（00000150，4-5），值为0x0000。

line_number 项的值必须与源文件的行数相匹配，值为0x0003。

按照javap结果中的描述
``` java
      LineNumberTable:
        line 3: 0
```
可以一一对应。

按照ClassFile的结构，后面还剩下main方法的字节码和ClassFile本身附加属性的字节码，可以按照上述方法结合jvm规范一一分析，这里不再阐述。
