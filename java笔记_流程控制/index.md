# Java笔记_流程控制


<!--more-->

### if判断

基本语法

```java
if(条件) {
    // 条件满足时执行
} else if {
    // 另一个条件满足时执行
} else {
    // 条件不满足时执行
}
```

判断相等

整数相等可以直接使用`==`

浮点数因为无法精确表示，不能直接使用`==`，利用差值小于某个临界值来判断

引用类型判断相等

`==`是判断是否指向同一个对象

判断内容相同使用`equals()`方法

```java
public class Main {
    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = "HELLO".toLowerCase();
        System.out.println(s1);
        System.out.println(s2);
        if (s1 == s2) { // false
            System.out.println("s1 equals s2");
        } else (s1.equals(s2)) { // true
            System.out.println("s1 equals s2");
        }
    }
}
```

**执行语句`s1.equals(s2)`时，如果变量`s1` 为`null`，会报`NullPointerException`**

要避免`NullPointerException`错误，可以利用短路运算符`&&`

`s1 != null && s1.equals("hello")`

### switch多重选择

`switch`语句根据`switch(表达式)`计算的结果，跳转到匹配的case结果，然后执行后续语句，直到遇到`break`结束执行。

```java
public class Main {
    public static void main(String[] args) {
        int option = 1;
        switch(option) {
            case 1:
                System.out.println("Selected 1");
                break;
            case 2:
                System.out.println("selected 2");
                break;
            defautl:
                System.out.println("Not selected");
                break;
        }
    }
}
```

**switch也可以匹配字符串，字符串匹配时，是比较“内容相等”。**

### while循环

基本用法

```java
while (条件表达式) {
    循环语句
}
// 继续执行后续代码
```

**多站在内存的方向想问题**

### do while 循环

先执行一次循环，然后再判断条件，条件满足继续执行，不满足，退出

```java
do {
    执行循环语句
} while (条件表达式)
```

### for 循环

先初始化计数器，在每次循环前检测循环条件，在每次循环后更新计数器。

```java
for (int i = 1; i <= 100; i++) {
    // 循环语句
}

public class Main {
    public static void main(String[] args) {
        int ns = {1,4,9,16,25};
        int sum = 0;
        for (int i = 0; i < ns.length; i++) {
            sum = sum + ns[i];
        }
        System.out.println(sum);
    }
}


```

#### 灵活使用for循环

缺少初始化语句、循环条件和每次循环更新语句

```java
// 不设置结束条件
for(int i = 0; ; i++) {
    
}
// 不设置结束条件和更新语句
for (int i = 0; ; ) {
    
}
// 都不设置
for ( ; ; ) {
    
}
```

> 通常不推荐

#### for each 循环

用来遍历数组

```java
public class Main {
    public static void main(String[] args) {
        int[] ns = {1,3,5,6,7,12};
        for (int n : ns) {
            System.out.println(n);
        }
    }
}
```

`n`不再是计数器，而是直接对应到数组中的每个元素，但是`for each`无法指定遍历顺序，也无法获取数组的索引。

### break 和 continue

#### break

在循环过程中，可以使用`break`语句跳出当前循环。

#### continue

`continue`提前结束本次循环，直接继续执行下次循环。


