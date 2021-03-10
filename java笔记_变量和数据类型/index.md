# Java笔记_变量和数据类型


<!--more-->

### 变量和数据类型

变量必须先定义后再使用，类型在前，命名在后

```java
int x = 1;
```

> **不写初始值，就相当于指定了默认值。**
>
> int默认值为0

#### 基本数据类型

基本数据类型是CPU可以直接进行运算的类型。

- 整数类型：byte，short， int，long
- 浮点数类型：float，double
- 字符类型：char
- 布尔类型：boolean



计算机内存的最小存储单元是字节（byte），一个字节就是一个8位二进制数，即8个bit，它的二进制表示范围从`00000000` ~ `11111111`，换算成十进制是0~255，换算成十六进制是`00`~`ff`。

![image-20210310093912553](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/image-20210310093912553.png)

> `byte`恰好就是一个字节，而`long`和`double`需要8个字节

#### 整型

各种整型能表示的最大范围：

- byte：-128 ~ 127
- short: -32768 ~ 32767
- int: -2147483648 ~ 2147483647
- long: -9223372036854775808 ~ 9223372036854775807

Java只定义了带符号的整型

```java
public class Main {
    public static void main(String[] args) {
        int i = 2147483647;
        int i2 = -2147483648;
        int i3 = 2_000_000_000; // 加下划线更容易识别
        int i4 = 0xff0000; // 十六进制表示的16711680
        int i5 = 0b1000000000; // 二进制表示的512
        long l = 9000000000000000000L; // long型的结尾需要加L
    }
}
```

#### 浮点型

浮点类型的数就是小数，因为小数用科学计数法表示的时候，小数点是可以“浮动”的，如1234.5可以表示成

12.345x10^2或者1.2345x10^3，所以称为浮点数。

```java
float f1 = 3.14f;
float f2 = 3.12e38f; // 科学计数法表示对的3.14x10^38
double d = 1.79e308;
double d2 = -1.79e308;
double d3 = 4.9e-324;
```

float 类型，需要加上`f`后缀

float最大3.4x10^38，double最大1.79x10^308

#### 布尔类型

只有两个值：true和false，boolean 表示为4个字节整数

#### 字符类型

字符类型char 表示一个字符，char类型除了可以表示标准的ASCII外，还可以表示Unicode字符

```java
char a = 'A';
char zh = '中'
```

使用单引号`'`，且仅有一个字符，双引号是字符串`"`

#### 引用类型

#### 常量

定义变量的时候，如果加上`final`修饰符，这个变量就变成了常量

```java
final double PI = 3.14;
double r = 5.0;
double area = PI * r * r;
PI = 300; // compile error！
```

常量在定义时进行初始化后就不可再此赋值，再此复制会导致编译错误。

常量名通常全部大写

#### var关键字

使用`var`定义变量，编译器会根据复制语句自动推断变量的类型

```java
var sb = new StringBuilder();
// 自动变成
StringBuilder sb = new StringBuilder();
```

#### 变量的作用范围
