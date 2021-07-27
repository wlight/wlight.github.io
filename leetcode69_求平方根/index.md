# LeetCode69求平方根

### 题目

实现 int sqrt(int x) 函数。

计算并返回 x 的平方根，其中 x 是非负整数。

由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

**示例 1:**

输入: 4
输出: 2

**示例 2:**

输入: 8
输出: 2
说明: 8 的平方根是 2.82842..., 
     由于返回类型是整数，小数部分将被舍去。

### 解题思路

- 要求求出根号 x
- 根号 x 的取值范围一定在[0, x]之间，这个区间内的值是递增有序的，有边界的，可以用下标访问的，二分查找正好满足这三个条件，所以可以使用二分查找

### 实现代码
#### 暴力解法
```go
func mySqrt(x int) int {
	for i := 0; i <= x; i++ {
		res := i * i
		if res == x {
			return i
		} else if res > x {
			return i - 1
		}
	}
	return -1
}
```
#### 二分查找法
**思路**
- 根号 x 的取值范围一定在[0, x]之间，找到最后一个整数平方之后的值小于或等于x
- 查找最后一个小于或等于给定值的元素，所以当 res <= x 时，需要确认一下 这个数的后一位的平方大于 x
- 如果 mid+1的平方小于或等于 x，说明mid肯定不是目的整数，更新low
```go
func mySqrt(x int) int {
	if x == 0 {
		return 0
	}
	low, high := 0, x
	for low <= high {
		mid := low + ((high-low)>>1)
		res := mid * mid
		if res > x {
			high = mid - 1
		} else {
			if (res == x) || (mid + 1)*(mid+1) > x {
				return mid
			}
			low = mid + 1
		}
	}
	return -1
}
```
