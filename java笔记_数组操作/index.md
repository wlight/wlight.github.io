# Java笔记_数组操作


<!--more-->

### 遍历

for循环遍历，可以得到数组的索引

```java
for (int i = 0; i < ns.length; i++) {
    
}
```

for each 循环遍历，直接拿到数组元素的值，无法拿到索引

```java
for (int n : ns) {
    
}
```

#### 打印数组内容

直接打印数组变量，得到是数组在JVM中的引用地址

所以使用for each打印

或者使用Java标准库`Arrays.toString()`，快速打印数组的内容；

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] ns = {1,2,3,4,5};
        System.out.println(Arrays.toString(ns)); // [1, 1, 2, 3, 5, 8]
    }
}
```

### 数组排序

#### 冒泡排序

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] ns = {28, 12, 89, 73, 65, 18, 96, 50, 8, 36};
        System.out.println("排序前：" +  Arrays.toString(ns));
        for (int i = 0; i < ns.length; i ++) {
            for (int j = ns.length - 1; j > i; j --) {
                if(ns[j-1] > ns[j]) {
                    int temp = ns[j];
                    ns[j] = ns[j-1];
                    ns[j-1] = temp;
                }
            }
        }
        
        System.out.println("排序后："+ Arrays.toString(ns));
    }
}
```

**Java 标准库内置排序功能，`Arrays.sort()`**

### 多维数组

二维数组就是数组的数组，三维数组就是二维数组的数组；

多维数组的每个元素长度都可以不同；

打印多维数组可以使用`Arrays.deepToString()`；

最常见的多维数组是二维数组，访问二维数组的一个元素使用`array[row][col]`
