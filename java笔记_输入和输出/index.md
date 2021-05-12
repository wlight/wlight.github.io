# Java笔记_输入和输出


<!--more-->

#### 输出

`System.out.println`：输出并换行

`System.out.print`：输出不换行

#### 格式话输出

`System.out.printf`：通过占位符`%?`，可以把后面得参数格式化成指定格式

```java
double d = 3.1516;
System.out.printf("%.2f\n", d); // 显示两位小数3,14
```

占位符，把各种数据类型格式化成指定得字符串

| 占位符 | 说明                             |
| ------ | -------------------------------- |
| %d     | 格式化输出整数                   |
| %x     | 格式化输出十六进制整数           |
| %f     | 格式化输出浮点数                 |
| %e     | 格式化输出科学计数法表示的浮点数 |
| %s     | 格式化字符串                     |

> 由于%表示占位符，因此两个%% 表示一个%字符本身

详细格式化参数参考JDK文档[java.util.Formatter]([Formatter (Java SE 11 & JDK 11 ) (oracle.com)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Formatter.html#syntax))

#### 输入

从控制台读取
