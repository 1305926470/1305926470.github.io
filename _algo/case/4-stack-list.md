---
layout: single
title:  "栈和队列"
date:   2021-03-17 17:59:46 +0800
permalink: /algo/case/stack-list
toc: true
toc_sticky: true
---

[TOC]





栈的特点是后进先出。栈是一个非常常见的数据结构。

通常栈是一个不考虑排序的数据结构, 需要On)时间才能找到栈中最大或者最小的元素。如果想要在O1)时间内得到枝的最大值或者最小值, 则需要对栈做特殊的设计, 试题30“包含min函数的栈”。

队列特点是先进先出。

栈和队列虽然是特点针锋相对的两个数据结构, 但有意思的是它们却相互联系。如用两个栈实现队列。



## 练习

### 试题9:用两个栈实现队列

题目:用两个栈实现一个队列。队列的声明如下, 请实现它的两个函数 appendtail和 deletehead, 分别完成在队列尾部插入节点和在队列头部删除节点的功能。



从上述队列的声明中可以看出,一个队列包含了两个栈 stack1和stack2, 因此这道题的意图是要求我们操作这两个“先进后出”的栈实现一个“先进先出”的队列 Queue。

- 当 stack2 为空时, 我们把 stack1 中的`所有`元素逐个弹出并压入 stack2。

- 当 stack2不为空时, 在 stack2 中的栈顶元素是最先进入队列的元素, 可以弹出。

由于先进入队列的元素被压到 stack1的底端, 经过弹出和压入操作之后就处于 stack2的顶端, 又可以直接弹出。





### 试题30:包含min函数的栈

题目:定义栈的数据结构,请在该类型中实现一个能够得到栈的最小元素的min函数。在该栈中,调用min、push及pop的时间复杂度都是O(1)。

思路：使用两个栈，一个主栈存放元素；一个辅助栈用于存放主栈元push素时，对应的最小元素。

| 步骤 | 操作    | 数据栈  | 辅助栈     | 最小值 |
| ---- | ------- | ------- | ---------- | ------ |
| 1    | push(3) | 3       | 3          | 3      |
| 2    | push(4) | 3,4     | 3，3       | 3      |
| 3    | push(2) | 3,4,2   | 3，3，2    | 2      |
| 4    | push(1) | 3,4,2,1 | 3，3，2，1 | 1      |
| 5    | pop()   | 3,4,2   | 3，3，2    | 2      |
| 6    | pop()   | 3,4     | 3, 3       | 3      |
| 7    | push(0) | 3,4,0   | 3, 3, 0    | 0      |

```go
type Stack struct {
    Data    []int // 主栈元素
    MinList []int // 每个主栈元素对应的最小值
    nextIdx int   // 下个存放元素位置
}

var St *Stack

func Push(node int) {
    if St == nil || St.nextIdx == 0 {
        St = &Stack{
            Data:    []int{node},
            MinList: []int{node},
            MaxList: []int{node},
            nextIdx: 1,
        }
        return
    }
    // 扩容，填充
    if St.nextIdx >= len(St.Data) {
        St.Data = append(St.Data, 0)
        St.MinList = append(St.MinList, 0)
    }
    // 辅助栈
    if node < St.MinList[St.nextIdx-1] {
        St.MinList[St.nextIdx] = node
    } else {
        St.MinList[St.nextIdx] = St.MinList[St.nextIdx-1]
    }

    St.Data[St.nextIdx] = node
    St.nextIdx++
    return
}

func Pop() {
    if St == nil {
        return
    }
    if St.nextIdx == 0 {
        return
    }
    St.nextIdx--
}

func Min() int {
    if St == nil {
        return 0
    }
    if St.nextIdx > 0 {
        return St.MinList[St.nextIdx-1]
    }
    return 0
}
```

测试用例：

```
// "PSH3","MIN","PSH4","MIN","PSH2","MIN","PSH3", "MIN",
// "POP","MIN",
// "POP","MIN",
// "POP","MIN",
// "PSH0","MIN"
```

### 试题31:栈的压入、弹出序列——复杂问题分析

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）



思路：

- 从前往后遍历弹出序列，如果对应的数字不在栈顶，需要把输入序列对应的数字及之前的数压入栈中，也就是1，2，3，4要压入栈中。

- 如果弹出数字已经在栈顶了，直接弹出，比较下一个数字，也就是 3 != 5所以，继续压入栈，直到遇见 5。类似直到最后。
- 对于第二个弹出序列：第一个是 4 ，先把 1，2，3，4要压入栈中。然后先弹出 4 和 3，后面把 5压入栈，弹出。
- 剩下的栈顶是2 ，但对应的弹出数字是 1，不等。此时所有数都已经压入栈中。所以 第二个不是弹出序列。

```go
// 输入 nil
// 输入只有一个元素
// 输入的两个数组不一样长
// 输入是弹出序列
// 输入不是弹出序列的
func IsPopOrder(pushV []int, popV []int) bool {
    if len(pushV) != len(popV) {
        return false
    }
    if len(pushV) == 0 {
        return false
    }

    stack := make([]int, len(pushV), len(pushV))
    stackTop := 0
    length := len(pushV)
    jMin := 0
    var i int
    for i = 0; i < length; {
        if jMin >= length {
            break
        }
        for j := jMin; j < length; j++ {
            if pushV[j] != popV[i] { // 不等则压入栈
                stack[stackTop] = pushV[j]
                stackTop++
                jMin++ // 记录待入栈位置
            } else {
                i++
                jMin++
                break
            }
        }
        // 判断栈顶元素是否能弹出
        k := stackTop - 1
        for k >= 0 {
            if stack[k] != popV[i] { // 不相等继续把输入序列压入栈
                break
            }
            // 相等
            i++
            k--
            stackTop--
        }
    }

    return i == length
}
```









