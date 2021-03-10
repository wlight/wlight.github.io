# Java笔记_程序基本结构


<!--more-->

### Java程序基本结构

``` java
/**
* 可以用来自动创建文档的注释
*/
public class Hello {
    public static void main(String[] args) {
        // 向屏幕输出文本
        System.out.println("Hello, world!");
        /* 多行注释
        注释
        注释*/
    }
} // class 定义结束
```

 类名要求：

- 类名必须以英文字母开头，后接字母，数字和下划线
- 习惯以大写字母开头

> **Java入口程序规定的方法必须是静态方法，方法名必须为main，括号内的参数必须是String数组。**

方法命名规则：

- 方法名必须以英文字母开头，后接字母，数字和下划线
- 首字母必须小写

> Java的每一行语句必须以分号结束
