---
layout: single
title:  "查找"
date:   2021-03-19 13:42:46 +0800
permalink: /algo/case/search
toc: true
toc_sticky: true
---

[TOC]



査找和排序都是在程序设计中经常用到的算法。

查找相对简单, 不外乎`顺序査找`、`二分査找`、`哈希表査找`和`二叉排序树査找`。

在面试的时候,不管是用循环还是用递归, 面试官都期待应聘者能够信手拈来写出完整正确的二分査找代码, 否则可能连继续面试的兴趣都没有。



常见的二分查找：

- 一组有序的数字
- 9个球，只有一个重量与众不同，其他都一样，找出来，这个可以`三分法`

`哈希表`和`二又排序树`査找的*重点在于考查对应的数据结构而不是算法*。

哈希表最主要的优点是我们利用它能够在O(1)时间内查找某一元素, 是效率最高的査找方式; 但其缺点是需要额外的空间来实现哈希表。





## 练习

### 试题11:旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。

输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。例如, 数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转, 该数组的最小值为 1 。

（假设所有元素都大于0，若数组大小为0，请返回0）

```go
// 查找最常用的就是二分法，但是常常是二分法的变形。
// 假设所有元素都大于0，若数组大小为0，请返回0。
// 查找最常用的就是二分法，但是常常是二分法的变形。
func MinNumberInRotateArray(rotateArray []int) int {
    if len(rotateArray) == 0 {
        return 0
    }

    idx1 := 0
    idx2 := len(rotateArray) - 1

    for idx1 < idx2 - 1 {
        midIdx := idx1 + (idx2 - idx1) >> 1

        if rotateArray[idx1] == rotateArray[midIdx] && rotateArray[midIdx] == rotateArray[idx2] {
            // 此时只能依靠循序查找
            for i := idx1; i < idx2 - 1; i++ {
                if rotateArray[i] > rotateArray[i+1] {
                    return rotateArray[i + 1]
                }
            }
        }

        if rotateArray[idx1] <= rotateArray[midIdx] {
           idx1 = midIdx
        } else {
           idx2 = midIdx
        }
    }
   return rotateArray[idx2]
}
```

算法思想，不是很复杂，但是边界条件真不少，要想一次写出没 bug 的代码，还真不容易。测试用例：

```go
func TestMinNumberInRotateArray(t *testing.T) {
    var arr []int
    arr = []int{3, 4, 5, 1, 2}
    fmt.Println(MinNumberInRotateArray(arr))

    arr = []int{3, 4, 1, 2}
    fmt.Println(MinNumberInRotateArray(arr))

    arr = []int{1, 0, 1, 1, 1, 1, 1}
    fmt.Println(MinNumberInRotateArray(arr))

    arr = []int{1, 1, 1, 0, 1}
    fmt.Println(MinNumberInRotateArray(arr))

    arr = []int{2, 2, 1}
    fmt.Println(MinNumberInRotateArray(arr))

    arr = []int{2, 1, 2}
    fmt.Println(MinNumberInRotateArray(arr))

    arr = []int{3, 2, 2, 3}
    fmt.Println(MinNumberInRotateArray(arr))
}
```

考查点：

- 考査应聘者对二分查找的理解。本题变换了二分査找的条件,输入的数组不是排序的,而是排序数组的一个旋转。这要求我们对二分查找的过程有深刻的理解。
- 考査沟通能力和学习能力，对新概念如旋转数组的理解和学习，通过提问来弄清楚。
- 考查思维的全面性，有些特殊情况，能否考虑到。

