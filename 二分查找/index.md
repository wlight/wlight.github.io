# 二分查找

# 什么是二分查找
二分查找针对的是一个有序的数据集合，查找思想有点类似分治思想。每次都通过跟区间的中间元素对比，讲待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为 0。
# 时间复杂度分析
## 时间复杂度
假设数据大小是 n，每次查找后数据都会缩小为原来的一半，也就是除以 2，最坏情况下，直到查找被缩小为空才停止。
被查找区间的大小变化：

$n, \frac{n}{2} , \frac{n}{4}, \frac{n}{8},....,\frac{n}{2^{k}}, ....$

这是一个等比数列。其中 $\frac{n}{2^{k}}=1$ 时，k 的值就是总共缩小的次数。而每一次缩小操作只涉及两个数据的大小比较，所以，经过了 k 次缩小操作，时间复杂度就是 O(k)。通过 $\frac{n}{2^{k}}=1$，我们可以求得 $k=\log2^{n}$,所以时间复杂度就是 O(logn)。
# 代码实现
最简单的二分查找：就是在有序数组中不存在重复的元素。
实现代码如下：
## 非递归代码实现
```go
func bsearch(arr []int, n int, target int) int {
	low := 0
	height := n - 1
	for low <= height {
		mid := low + (height-low)>>1
		if arr[mid] == target {
			return mid
		} else if arr[mid] < target {
			low = mid + 1
		} else {
			height = mid - 1
		}
	}
	return -1
}
```

## 递归代码实现
```go
// 二分查找递归实现
func bsearch(arr []int, n int, target int) int {
	return bsearchInternal(arr, 0, n-1, target)
}

func bsearchInternal(arr []int, low, high, target int) int {
	if low > high {
		return -1
	}

	mid := low + ((high-low)>>1)
	if arr[mid] == target {
		return mid
	} else if arr[mid] < target {
		return bsearchInternal(arr, mid+1, high, target)
	} else {
		return bsearchInternal(arr, low, mid-1, target)
	}
}
```
## 容易出错的三个地方

1、循环退出条件

注意是 low<=high，而不是 low<high

2、mid 的取值

`mid := (low+high)/2`这种写法很容易出问题，如果low和high都很大，两者之和就可能会溢出。改进的方法可以写成这样：`mid := low+(high-low)/2`，为了优化性能，除以2可以写成位运算，`mid := low+((high-low)>>1)`

3、low和high的更新

low= mid+1, high=mid-1

# 二分查找应用场景的局限性
1.  首先，二分查找依赖的是循序表结构，简单点说就是数组
2. 其次，二分查找针对的是有序数据
3. 再此，数据量太小不适合二分查找
4. 最后，数据量太大也不适合二分查找，因为数组结构是需要连续的空间

