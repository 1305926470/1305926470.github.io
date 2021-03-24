---
layout: single
title:  "数组"
date:   2021-03-17 15:04:46 +0800
permalink: /algo/case/array
toc: true
toc_sticky: true
---



[TOC]



## 数组

### 试题3.找出数组中重复的数字

在一个长度为n的数组里的所有数字都在0~n-1的范围内。数组中某些数字是重复的, 但不知道有几个数字重复了, 也不知道每个数字重复了,几次。请找出数组中任意一个重复的数字。例如,如果输入长度为7的数组 {2,3,1,0,2,5,3}, 那么对应的输出是重复的数字2或者3。

方法一：对数组排序，时间 O(nlogn)，排序之后找出相同的数就容易了。

方法二：通过一个哈希表，可以O(1) 找到目标，时间为 O(n)，但需要 O(n)的哈希表的额外空间开销。

方法三：思路从前往后把每个数放到与自己值相等的index位置上，有点原地排序的意思。 

```go
// 方法三:循环实现
func FindDuplication(in []int) int {
    n := len(in)
    // 数据判断合法性
    for i := 0; i < n; i++ {
        if in[i] < 0 || in[i] >= n {
            return -1
        }
    }

    for i := 0; i < n; i++ {
        // 如果当前 i 与 in[i] 不相等，就与in[in[i]] 交换，知道满足 i == in[i],然后往后i++
        // 当 in[i] == in[in[i]] 相等时，说明找到重复数据，结束
        for i != in[i] {
            if in[i] == in[in[i]] {
                return in[i]
            }
            in[i], in[in[i]]= in[in[i]], in[i]
        }
    }
    return -1
}


```

代码中尽管有一个两重循环, 但每个数字最多只要交换两次就能找到属于它自己的位置, 因此总的时间复杂度是On)。另外,所有的操作步骤都是在输入数组上进行的, 不需要额外分配内存, 因此空间复杂度为O(1)。

测试用例：

- 包含一个或多个重复数据的输入
- 没有重复数据的输入
- 有超出数组大小的数据输入

```go
func TestFindDuplication(t *testing.T) {
    tests := []struct {
        name string
        in   []int
        want int
    }{
        {"normal input", []int{2,3,1,0,2,5,3}, 2},
        {"not found", []int{2,3,1,0,4,5,6}, -1},
        {"index out of range", []int{2,13,1,0,2,5,3}, -1},
        {"index out of range", []int{-2,3,1,0,2,5,3}, -1},
    }

    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            got := FindDuplication(test.in)
            if got != test.want {
                t.Fatalf(" got: %d\nwant: %d\n", got, test.want)
            }
        })
    }
}
```



### 试题21:调整数组顺序使奇数位于偶数前面

题目:输入一个整数数组,实现一个函数来调整该数组中数字的顺序, 使得所有奇数位于数组的前半部分, 所有偶数位于数组的后半部分。

```go
// 如果不限制，奇数，偶数内部顺序的相对稳定
// 使用两个index 分别从数组首尾往中间移动
func ReOrderArray1( array []int ) []int {
    i := 0
    j := len(array) - 1

    for i < j {
        // i 遇到奇数跳过
        if array[i] % 2 == 1 {
            i++
        }

        // j 遇到偶数跳过
        if array[j] % 2 == 0 {
            j--
        }

        // 如果 i 处于偶数，j处于奇数，则交换，然后往中间移动
        if array[i] % 2 == 0 && array[j] % 2 == 1 && i < j{
            array[i], array[j] = array[j], array[i]
            i++
            j--
        }
    }

    return array
}
```



变形：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相`对位置不变`。

```go
func ReOrderArray( array []int ) []int {
    tmp := make([]int, 0)

    oddNum := 0
    for i := 0; i < len(array); i++ {
        if array[i] % 2 == 1 {
            array[oddNum] = array[i]
            oddNum++
        } else {
            // 偶数
            tmp = append(tmp, array[i])
        }
    }

    for i := 0; i < len(tmp); i++ {
        array[oddNum+i] = tmp[i]
    }

    return array
}
```



### 试题29:顺时针打印矩阵（边界陷阱）

输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，
例如，如果输入如下4 X 4矩阵：则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

```
1  2  3  4
5  6  7  8
9  10 11 12
13 14 15 16
```

```go
// 测试用例：
// 输入 0*0 1*1
// 输入 1*4  4*1
// 输入 4*4
func PrintMatrix( matrix [][]int ) []int {
    h := len(matrix)
    if h  == 0 {
        return nil
    }
    w :=len(matrix[0])
    if  w == 0 {
        return nil
    }
    ret := make([]int, 0, w*h)

    iMin, jMin := 0, 0
    iMax, jMax := w - 1, h - 1
    //var i, j int
    for iMin <= iMax && jMin <= jMax {
        // 右移
        for i := iMin; i <= iMax; i++ {
            ret = append(ret, matrix[jMin][i])
        }
        jMin++
        if jMin > jMax {
            return ret
        }
        // 下移
        for j := jMin;j <= jMax; j++ {
            ret = append(ret, matrix[j][iMax])
        }
        iMax--
        // 左移
        for i:= iMax ;i >= iMin; i-- {
            ret = append(ret, matrix[jMax][i])
        }
        jMax--
        if iMin > iMax {
            return ret
        }
        // 上移
        for j := jMax;j >= jMin; j-- {
            ret = append(ret, matrix[j][iMin])
        }
        iMin++
    }
    return ret
}
```

这道题完全没有涉及复杂的数据结构或者高级的算法, 看起来是一个很简单的问题。

但实际上解决这个问题时会在代码中包含多个循环, 并且需要判断多个边界条件。如果在把问题考虑得很清楚之前就开始写代码, 则不可避免地会越写越混乱。因此解决这个问题的关键在于先形成清晰的思路, 并把复杂的问题分解成若干个简单的问题。

看似简单，但边界条件较多，一次写对其实并不容易，所以测试用例，很重要。



### 试题39:数组中出现次数超过一半的数字

题目：数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

解法一：利用类似快排的思想，随便找一个数，比它小的往左移，比它大的往有移动。一轮循环下来，如果有个数超过一半，那么数组的中位数一定数这个数。然后再循环一次，去人是否过半了。

解法二：如果数组中有一个数字出现的次数超过数组长度的一半, 则它出现的次数比其他所有数字出现次数的和还要多。因此,我们可以在遍历数组的时候保存两个值:一个元素值 a，一个对应的次数 num。如果下一个数与当前保存的a相同，则num++，如果不同则num--，当次数 num 等于0时，当前保存的数 a 要被替换掉，并次数 num重置为1。如果有个数次数过半，最后剩下的一定是它。这里并不能确认就是它，所以要在循环一个次，进行确认。

解法三：用一个哈希表，统计次数，一次循环。就可以。

```go
// 解法二 
// 测试用例
// 数组 []int{2,2,2,3,3,3}
// 数组 []int{1,2,3,2,2,2,5,4,2}
// 数组 []int{1}
// 数组 []int{1,2,3}
func MoreThanHalfNum_Solution( numbers []int ) int {
    if len(numbers) == 0 {
        return 0
    }

    if len(numbers) == 1 {
        return numbers[0]
    }

    cur := numbers[0]
    num := 1
    for i := 1; i < len(numbers); i++ {
        if cur == numbers[i] {
            num++
            continue
        }
        num--
        if num == 0 {
            cur = numbers[i]
            num = 1
        }
    }

    // 再次确认次数是否过半
    num = 0
    for i := 0; i < len(numbers); i++ {
        if cur == numbers[i] {
            num++
        }
    }

    if  2*num > len(numbers) {
        return cur
    }
    return 0
}
```

感觉三种方法都不够好。



### 试题53：数字在升序数组中出现次数

统计一个数字在升序数组中出现的次数。如输入 [1,2,3,3,3,3,4,5],3 返回 4.

二分法

```go
// 测试用例
// 空数组
// 所有数大小一样
// 乱序
// 恰好在开头，结尾
func GetNumberOfK( data []int ,  k int ) int {
    if len(data) == 0 {
        return 0
    }

    count := 0
    idx1 := 0
    idx2 := len(data) - 1
    for idx1 <= idx2 {
        mid := idx1 + (idx2 - idx1) >> 1
        switch {
        case data[mid] > k:
            idx2 = mid - 1
        case data[mid] < k:
            idx1 = mid + 1
        default:
            for i := mid; i <= idx2 ;i++ {
                if data[i] != k {
                    break
                }
                count++
            }
            for i := mid - 1; i >= idx1; i-- {
                if data[i] != k {
                    break
                }
                count++
            }
            return count
        }
    }
    return 0
}
```

### 试题57：和为S的两个数

输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

对应每个测试案例，输出两个数，小的先输出。输入 [1,2,4,7,11,15],15 返回值 [4,11]

```go
// 测试用例
// 空数组
// [1,2,4,7,11,14,16],15
// [1,2,4,7,11,15], 26
// [1,2,4,7,11,15], 3
// [1,2,4,7,11,15], 30
// [7,8] 15
// [8,8] 15
// 乱序
func FindNumbersWithSum( array []int ,  sum int ) []int {
    if len(array) < 2 {
        return nil
    }
    maxIdx := len(array) - 1
    if sum > array[maxIdx] + array[maxIdx-1] {
        return nil
    }
    if sum < array[0] + array[1] {
        return nil
    }
    ret := make([]int, 0, 2)
    i := 0
    j := maxIdx
    for i < j {
        if array[i] + array[j] > sum {
            j--
        }
        if array[i] + array[j] == sum {
            ret = append(ret, array[i], array[j])
            return ret
        }
        if array[i] + array[j] < sum {
            i++
        }
    }
    return ret
}
```





## reference

https://github.com/zhedahht/CodingInterviewChinese2

