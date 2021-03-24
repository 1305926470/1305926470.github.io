---
layout: single
title:  "排序"
date:   2021-03-19 13:42:46 +0800
permalink: /algo/case/sort
toc: true
toc_sticky: true
---

[TOC]





排序比査找要复杂一些。

面试官会经常要求应聘者比较`插入排序`、`冒泡排序`、`归并排序`、`快速排序`等不同算法的优劣。

强烈建议应聘者在准备面试的时候,一定要对各种排序算法的特点烂熟于胸, 能够从额外空间消耗、平均时间复杂度和最差时间复杂度等方面去比较它们的优缺点。

需要特别强调的是, 很多公司的面试官喜欢在面试环节要求应聘者写出快速排序的代码。应聘者不妨自己写一个快速排序的函数并用各种数据进行测试当测试都通过之后, 再和经典的实现进行比较,看看有什么区別。

## 快排

实现快速排序算法的关键在于先在数组中选择一个数字即为 pivot, 接下来把数组中的数字分为两部分, 比选择的数字小的数字移到数组的左边, 比选择的数字大的数字移到数组的右边。每轮下来，这个 pivot 的索引位置就确定下来了。然后两边的区间数字再重复这个过程。直到结束。

```go
func QuickSort(arr []int, minIdx, maxIdx int) {
    if len(arr) <= 1 {
        return
    }

    if minIdx >= maxIdx {
        return
    }

    pivot := arr[maxIdx] //每次都已最后一个作为临界值，如果数组是逆序的话，时间就变成 O(n^2)了，所以还得随机选一个值。
    ltIdx := minIdx // 记录小于 pivot的个数 

    for i := minIdx; i < maxIdx; i++ {
        if arr[i] < pivot {
            arr[ltIdx], arr[i] = arr[i], arr[ltIdx]
            ltIdx++
        }
    }
    arr[ltIdx], arr[maxIdx] = arr[maxIdx], arr[ltIdx]
    QuickSort(arr, minIdx, ltIdx - 1)
    QuickSort(arr, ltIdx + 1, maxIdx)
}
```

对于 `pivot := arr[maxIdx]` 每次都已最后一个作为临界值，如果数组是逆序的话，时间就变成 O(n^2)了，所以还得随机选一个值。比如下面这样

```go
// pivot := arr[randIndex(minIdx, maxIdx)]
func randIndex(minIdx, maxIdx int) int {
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    return minIdx + r.Int() % (maxIdx - minIdx + 1)
}
```



以用来实现在长度为n的数组中査找第k大的数字。

试题39“数组中出现次数超过一半的数字”

试题40“最小的k个数”都可以用这个函数来解决

## 归并排序

分治法，有点像排块的，但是从低往上。

通常，归并排序需要一个长度为n的辅助数组。

```go
// 归并排序
// 测试用例：
// 空数组
// 数组长度为 1， 2
// 数组逆序
// 数组数都一样大
func MergeSort(arr []int) []int {
    if len(arr) < 2 {
        return arr
    }
    mergeSort(arr, 0, len(arr) - 1)
    return arr
}

func mergeSort(arr []int, idx1, idx2 int) {
    if idx1 >= idx2 {
        return
    }
    mid := idx1 + (idx2 - idx1) >> 1

    // 两个子区间
    mergeSort(arr, idx1, mid)
    mergeSort(arr, mid + 1, idx2)
    // 合并两个子区间
    mergeInternal(arr, idx1, mid, idx2)
}

func mergeInternal(arr []int, idx1, mid, idx2 int) {
    tmp := make([]int, 0, idx2 - idx1 + 1)
    i := idx1
    j := mid + 1
    for i <= mid && j <= idx2 {
        if arr[i] <= arr[j] {
            tmp = append(tmp, arr[i])
            i++
        } else {
            tmp = append(tmp, arr[j])
            j++
        }
    }

    start := i
    end := mid
    if j <= idx2 {
        start = j
        end = idx2
    }
    for k := start; k <= end; k++ {
        tmp = append(tmp, arr[k])
    }
    // 数据复制回原数组
    for k, v := range tmp {
        arr[idx1+k] = v
    }
    return
}
```





## 练习

### 试题40：最小 K 个数

给定一个数组，找出其中最小的K个数。例如数组元素是4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。如果K>数组的长度，那么返回一个空的数组。

例如：输入 [4,5,1,6,2,7,3,8],4 返回 [1,2,3,4]。

解法一：利用堆排序。

```go
// 解法一：利用堆排序，建堆，k 次堆调整
//测试用例
// 空数组
// k 大于数组长度
// 所有数都一样大
func GetLeastKNumbers( input []int ,  k int ) []int {
    if len(input) == 0 {
        return nil
    }

    if k > len(input) {
        return nil
    }
    ret := make([]int, 0, k)
    buildHeap(input)
    maxIdx := len(input) - 1

    for k > 0 {
        ret = append(ret, input[0])
        input[0],input[maxIdx] = input[maxIdx], input[0]
        maxIdx--
        k--
        heapfy(input, 0, maxIdx)
    }
    //return input[maxIdx+1:]
    return ret
}


// 建堆， 小顶堆
// 父节点为 input[i] 两个子树为 input[2*i+1] input[2*i+2]
func buildHeap(input []int) {
    length := len(input)
    for i := (length-1) >> 1; i >= 0; i-- {
        heapfy(input, i, length-1)
    }
}

// 堆调整
func heapify(input []int, i int, maxIdx int) {
    for {
        smallChild := 2 * i + 1
        if smallChild > maxIdx {
            return
        }
        if 2 * i + 2 <= maxIdx {
            if input[2*i+2] < input[smallChild] {
                smallChild = 2*i+2
            }
        }
        // 父子节点交换，使得父节点最小，
        if input[i] > input[smallChild] {
            input[i], input[smallChild] = input[smallChild],input[i]
            i = smallChild // 并继续堆调整
            continue
        }
        return
    }
}
```

































