# 二分查找变体

## 四种常见的二分查找变形问题

### 1、查找第一个值等于给定值的元素

```go

func main() {
	arr := []int{1, 3, 4, 5, 6, 8, 8, 8, 11, 18}

	target := 8
	log.Println(bsearch(arr, target))
}

// 查找第一个值等于给定值的元素
func bsearch(arr []int, target int) int {
	low, high := 0, len(arr)-1
	
	// 顺序数组
	// 直接索引取值
	// 终止条件
	for low <= high {
		mid := low + (high-low)>>1
		if arr[mid] > target {
			high = mid - 1
		} else if arr[mid] < target {
			low = mid + 1
		} else {
			if (mid == 0) || (arr[mid-1] != target) {
				return mid
			} else {
				high = mid - 1
			}
		}
	}

	return -1
}
```
### 2、查找最后一个值等于给定值的元素

```go
// 查找最后一个值等于给定值的元素
func bsearch(arr []int, target int) int {
	low, high := 0, len(arr)-1
	
	// 顺序数组
	// 直接索引取值
	// 终止条件
	for low <= high {
		mid := low + (high-low)>>1
		if arr[mid] > target {
			high = mid - 1
		} else if arr[mid] < target {
			low = mid + 1
		} else {
			if (mid == 0) || (arr[mid+1] != target) {
				return mid
			} else {
				high = mid - 1
			}
		}
	}

	return -1
}
```

### 3、查找第一个大于等于给定值的元素
```go
// 查找第一个大于等于给定值的元素
func bsearch(arr []int, target int) int {
	low, high := 0, len(arr)-1
	for low <= high {
		mid := low + ((high - low) >> 1)
		if arr[mid] < target {
			low = mid + 1
		} else {
			if (mid == 0) || (arr[mid - 1] < target) {
				return mid
			} else {
				high = mid - 1
			}
		}
	}

	return -1
}
```
### 4、查找最后一个小于等于给定值的元素

```go
// 查找最后一个小于等于给定值的元素
func bsearch(arr []int, target int) int {
	low, high := 0, len(arr)-1
	for low <= high {
		mid := low + ((high - low) >> 1)
		if arr[mid] > target {
			high = mid - 1
		} else {
			if mid == len(arr)-1 || arr[mid+1] > target {
				return mid
			} else {
				low = mid + 1
			}
		}
	}

	return -1
}
```
