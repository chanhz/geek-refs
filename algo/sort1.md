# 排序

## 总述
最常用的排序算法：

|    算法       |时间复杂度|是否基于比较|
|:-------------:|:-------:|:--------:|
|冒泡、选择、插入| O(n^2)   |   是     |
|快排、归并     | O(n*logn) | 是       |
|桶、计数、基数 | O(n)      | 否      |


## 如何分析一个排序算法
- 执行效率-时间复杂度
 - 最好情况
 - 最坏情况
 - 平均情况
 - 时间复杂度的系数、常数和低阶
 - 比较次数和交换次数
- 算法的内存消耗-空间复杂度
- 算法的稳定性

稳定排序算法可以保持金额相同的两个对象，在排序之后的前后顺序不变。


## 冒泡排序
- 每次遍历把一个最值归位
- 每次只能比较和交换相邻的两个数
- 相同元素不交换，因此是稳定的
- 最好情况 O(n)
- 最坏情况 O(n^2)

求单个最大值可以使用该算法


## 有序度和逆序度
有序度 = 满有序度 - 逆序度
逆序度代表一个待排序数组转换成一个有序数组需要进行的交换次数
对于冒泡排序来说，逆序度最大值为 n(n-1)/2, 最小值为 0
于是平均交换次数为 n*(n-1)/4


## 插入排序
一个有序数组里插入一个数后，仍然保持有序，这就是插入排序做的事。
通过比较和移动来实现

我们将数组中的数据分为两个区间，已排序区间和未排序区间。初始已排序区间只有一个元素，就是数组的第一个元素。插入算法的核心思想是取未排序区间中的元素，在已排序区间中找到合适的插入位置将其插入，并保证已排序区间数据一直有序。重复这个过程，直到未排序区间中元素为空，算法结束。

### 稳定性
插入相同元素的时候，我们可以后面出现的将元素插在相同元素的位置后面，保持原有前后顺序不变，所以是稳定的排序算法


### 时间复杂度分析
最好：未排序区间的元素是有序的，在有序数组从尾到头查找插入位置时，每次只需要比较一个数字就可以确定插入的位置，因此最好时间复杂度是O(1)
最坏：未排序区间是完全逆序的，O(n^2)

### 插入排序的优化
[希尔排序](https://zh.wikipedia.org/wiki/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F)

```go

import (
	"fmt"
)

func ShellSort(array []int) {
	n := len(array)
	if n < 2 {
		return
	}
	key := n / 2 // 初始化步长为长度的两半
	for key > 0 {
		for i := key; i < n; i++ {
			j := i
			for j >= key && array[j] < array[j-key] {
				array[j], array[j-key] = array[j-key], array[j]
				j = j - key
			}
		}
		key = key / 2
	}
}

func main() {
	array := []int{
		55, 94, 87, 1, 4, 32, 11, 77, 39, 42, 64, 53, 70, 12, 9,
	}
	fmt.Println(array)
	ShellSort(array)
	fmt.Println(array)

}
```

### 为什么插入排序比冒泡排序更受欢迎？
冒泡排序不管怎么优化，元素交换的次数是一个固定值，是原始数据的逆序度。插入排序是同样的，不管怎么优化，元素移动的次数也等于原始数据的逆序度。但是，从代码实现上来看，冒泡排序的数据交换要比插入排序的数据移动要复杂，冒泡排序需要 3 个赋值操作，而插入排序只需要 1 个。我们来看这段操作：
```go
//冒泡排序中数据的交换操作：
if (a[j] > a[j+1]) { 
    // 交换 
    a[j], a[j+1] = a[j+1], a[j]
}
//插入排序中数据的移动操作：
if (a[j] > value) {
     a[j+1] = a[j] // 数据移动
}
```


## 选择排序
选择排序算法的实现思路有点类似插入排序，也分已排序区间和未排序区间。但是选择排序每次会从未排序区间中找到最小的元素，将其放到已排序区间的末尾。

和冒泡排序的区别：可以交换非相邻元素


### 复杂度分析
每次查找最小值，需要完全扫描整个数组，比较次数为 n
n个元素，需要比较 n(n+1)/2次
因此最好、最好和平均时间复杂度为 O(n^2)

### 稳定性分析
每次都需要找到最小的元素,并和前面的元素交换位置，破坏了稳定性


## 