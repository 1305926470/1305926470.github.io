---
layout: single
title:  "链表"
date:   2021-03-17 00:04:46 +0800
permalink: /algo/case/linkedlist
toc: true
toc_sticky: true
---



[TOC]

## 链表是高频题

链表应该是面试时被提及最频繁的数据结构。

链表的结构很简单, 它由指针把若干个节点连接成链状结构。

链表的创建、插入节点、删除节点等操作都只需要20行左右的代码就能实现, **链表的代码量比较适合面试**。

而像哈希表、有向图等复杂数据结构, 实现它们的一个操作需要的代码量都较大, 很难在几十分钟的面试中完成。

另外,由于链表是一种动态的数据结构, 其需要对指针进行操作, 因此应聘者需要有较好的编程功底才能写出完整的操作链表的代码。而且链表这种数据结构很灵活, 面试官可以用链表来设计具有挑战性的面试题。基于上述几个原因, 很多面试官都特别青眯与链表相关的题目。



## 先分析，写测试用例

**先仔细分析, 顺便列出测试用例**

解决与链表相关的问题总是有大量的指针操作, 而指针操作的代码总是*容易出错*的。很多面试官喜欢出与链表相关的问题。来考査应聘者的编码功底。

为了避免出错, 我们最好先进行全面的分析。在实际软件开发周期中,设计的时间通常不会比编码的时间短。

在面试的时候我们不要急于动手写代码, 而是一开始仔细分析和设计,这将会给面试官留下很好的印象。与其很快写出一段漏洞百出的代码, 倒不如仔细分析再写出鲁棒的代码。



*怎么样避免边界错误？*

- 如果机试，用实现准备好的测试用例，跑一下。
- 如果是纸上手写，就只能在心中，把样本带入跑一下。

## 练习

### 从尾到头打印链表

题目:输入一个链表的头节点, 从尾到头反过来打印出每个节点的值链表节点定义如下:

```go
type ListNode struct {
    Val int
    Next *ListNode
}
```

例如输入 {67,0,24,58} 返回 [58,24,0,67]。

**方法一**：先逆序反转链表，再遍历，时间复杂度 O(n), 空间复杂度 O(1); 但这种方式会改变原链表。

```go
// 逆序反转链表，再遍历，但这个会修改原链表
//67->0->24->58
// 方法一：逆序反转链表，再遍历，但这个会修改原链表
//67->0->24->58
func PrintListFromTailToHead(head *ListNode) {
    if head == nil {
        return
    }
    if head.Next == nil { // 只有一个节点时，很容易漏掉这种处理
        fmt.Println(head.Val)
        return
    }
    // 链表逆序
    prev := head
    p := prev.Next
    // prev p next
    // 如果移动到最后一个位置，就判断 p != nil, 且别使用 p.next；
    // 如果使用 p.next，就得for 判断 p.next != nil，p 最后指向的是倒数第二位。
    for p.Next != nil {
        next := p.Next
        p.Next = prev
        prev = p
        p = next
    }
    p.Next = prev
    // 调整 head 指针
    head.Next = nil
    head = p

    for p := head; p != nil; p = p.Next {
        fmt.Print(p.Val, " ")
    }
    fmt.Println()
}
```

**特别强调**

- 链表的头和位部需要小心处理，很多时候都需要单独处理，所以最好，仔细模拟一遍

- 通过 for 循环遍历链表时，需要注意 p.Next 是否为空指针。如果移动到最后一个位置，就判断 p != nil, 且别使用 p.next；如果使用 p.next，就得for 判断 p.next != nil，p 最后指向的是倒数第二位。

    

**方法二**：如果不能改变原链表，先遍历放到栈里，再取出来，即 LIFO。但是时间和空间复杂度 O(n)。

```go
// 方法二：如果不能改变原链表，先遍历放到栈里，再取出来，即 LIFO。但是时间和空间复杂度 O(n)。
// 可以使用数组，来实现栈, 即先放数组，再逆序输出
func PrintListFromTailToHead2(head *ListNode) {
    if head == nil {
        return
    }

    n := 0
    for p := head; p != nil; p = p.Next {
        n++
    }

    arr := make([]int, 0, n)
    for p := head; p != nil; p = p.Next {
        arr = append(arr, p.Val)
    }

    for i := n - 1; i >= 0; i-- {
        fmt.Print(arr[i], " ")
    }
    fmt.Println()
}
```



**测试用例**

- 输入链表为 nil
- 链表只有一个节点
- 链表有多个节点

```go
func TestPrintListFromTailToHead(t *testing.T) {
    var head *ListNode
    head = Build(head, []int{67,0,24,58, 100,101,102,103})
    PrintListFromTailToHead(head)

    head = nil
    head = Build(head, []int{67})
    PrintListFromTailToHead(head)

    head = nil
    head = Build(head, []int{})
    PrintListFromTailToHead(head)
}
```

*测试用例真的太有必要了*，尤其是场景覆盖全面的测试用例，代码写完，感觉好像差不多了，但是一跑用例，往往发现漏了很多情况的处理。

### 试题18.1：在O(1)时间内删除链表节点。

给定单向链表的`头指针`和一个`待删除节点`指针,定义一个函数在O(1)时间内删除该节点。



分析：

- 因为不知道前一个节点的地址，一般的思路是先顺序查找到前一个, 再删除，时间复杂度自然就是O(n)了。
- 根据待删除节点，可以很容易找到下一个节点，可以把下一个节点的数据复制到待删除节点。
- 如果待删除节点位于尾部怎么办呢？那就只能顺序遍历了。
- 此外，如果只有一个节点，还需要把 head 置为 nil。

时间分析：

- 对于n-1个非尾节点而我们可以在O(1)时间内把下一个节点的内存复制覆盖要删除的节点,并删除下一个节点;

- 对于尾节点而言, 由于仍然需要顺序查找,时间复杂度是O(n)。
- 因此, 总的`平均时间复杂度`是[(n-1)*O(1)+O(n))]/n, 结果还是O(1)。



### 试题22:链表中倒数第k个节点

输入一个链表，输出该链表中倒数第k个结点。

解法一：快慢指针

```go
func FindKthToTail( pHead *ListNode ,  k int ) *ListNode {
    if pHead == nil || k < 1 {
        return nil
    }

    fastP := pHead

    mov := 0
    for fastP != nil && mov < k {
        fastP = fastP.Next
        mov++
    }
    if mov < k { // 如果链表中节点的数目小于于k
        return nil
    }

    slowP := pHead
    for fastP != nil {
        fastP = fastP.Next
        slowP = slowP.Next
    }

    return slowP
}

type ListNode struct {
    Val int
    Next *ListNode
}
```

题目不难，但是要注意边界条件

- head 指针为空时
- k 大于节点数时
- k 为 0，甚至是负数时
- 再次强调，全面的测试用例很重要。不要太相信自己的感觉。

### 试题23:链表中环的入口节点

给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。

- 判断是否包含环，使用快慢指针，快指针每次两步，慢指针每次走一步。两者再次相遇时，说明慢指针被快指针套圈，说明有环。
- 确定环的节点数。在上述基础上，继续移动，同时计数，当快慢指针再次相遇，慢指针刚好一圈，快的两圈。得到环的节点数 loopNodes。
- 从头开始，找到倒数第 n 个节点，即为环的入口节点。结合 loopNodes 个节点数，快指针先走 loopNodes 步，然后一起一步一步移动快慢指针，当两个指针相遇时，此时刚好在环的入口节点上。

```go
func EntryNodeOfLoop(pHead *ListNode) *ListNode{
    if pHead == nil {
        return nil
    }

    // 确定是否有环
    fastP := pHead
    slowP := pHead
    for {
        // 没有环
        if fastP == nil || fastP.Next == nil || fastP.Next.Next == nil {
            return nil
        }
        fastP = fastP.Next.Next
        slowP = slowP.Next
        if fastP == slowP { // 快慢指针相遇，说明有环
            break
        }
    }

    // 确定环节点数 loopNodes
    loopNodes := 0
    for {
        fastP = fastP.Next.Next
        slowP = slowP.Next
        loopNodes++
        if fastP == slowP { // 再次相遇刚好满指针绕环一圈
            break
        }
    }

    // 再次从头开始，结合 loopNodes 个节点数，快指针先走 loopNodes 步，
    // 然后一起一步一步移动快慢指针，当两个指针相遇时，此时刚好在环的入口节点上。
    fastP = pHead
    slowP = pHead
    for i := 0; i < loopNodes; i++ {
        fastP = fastP.Next
    }
    for slowP != fastP {
        slowP = slowP.Next
        fastP = fastP.Next
    }
    return slowP
}
```

测试用例：

- 输入 nil；
- 链表有环；
- 链表无环；
- 链表是首尾相连的环；
- 链表不规则，直接是一个图，甚至多个环；

### 试题24:反转链表

输入一个链表，反转链表后，输出新链表的表头。

```go
func ReverseList( pHead *ListNode ) *ListNode {
    if pHead == nil {
        return nil
    }

    p := pHead
    next := p.Next
    if next == nil { // 只有一个节点
        return pHead
    }

    for next.Next != nil {
        nextNext := next.Next
        next.Next = p
        p = next
        next = nextNext
    }
    // 末尾节点
    next.Next = p
    // 头指针处理
    pHead.Next = nil
    pHead = next
    return pHead
}
```

测试用例：

- 输入空指针
- 输入一个节点，这个容易漏判
- 两个节点时，正常样本，

此外，操作多个指针，指针头和尾的临界操作，非常容易出错。



### 试题25:合并两个排序的链表

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。

略

```go
// 测试用例：
//  输入链pHead1表为 nil
//  输入链pHead2表为 nil
//  链表只有一个元素
//  链表数据相等时
```





### 试题52:两个链表的第一个公共节点

输入两单个链表，找出它们的第一个公共结点。类似一个 Y 的形状。

解法一：从最后一个节点往前，到分叉口的节点，即为第一个公共节点。`从后往前，先进后出`，`栈`是个不错的选择。先遍历两个链表放入两个栈，然后同时逐个出栈。当下一个出现不同时，则刚好找到目标。

解法二：利用`快慢指针`。先统计两个链表长度，假设两个链表长度相差 n 个节点。快指针先遍历链表走 n 步。另一个再出发，一起遍历，相遇时，就是目标节点出现的时候。嗯嘛。

```go
// 解法二
//测试用例
// 链表为 nil
// 链表没有共同节点
// 第一个节点就重合
// 最后一个节点重复
// 中间某一节点重合
func FindFirstCommonNode( pHead1 *ListNode ,  pHead2 *ListNode ) *ListNode {
    if pHead1 == nil || pHead2 == nil {
        return nil
    }

    // 先统计链表长度
    l1 := 0
    for p := pHead1; p != nil; p = p.Next {
        l1++
    }
    l2 := 0
    for p := pHead2; p != nil; p = p.Next {
        l2++
    }

    p1 := pHead1
    p2 := pHead2
    // 长链先遍历
    if l1 >= l2 {
        delta := l1 - l2
        for i := 0; i < delta; i++ {
            p1 = p1.Next
        }
    } else {
        delta := l2 - l1
        for i := 0; i < delta; i++ {
            p2 = p2.Next
        }
    }

    // 一起移动指针
    for p1 != nil && p2 != nil {
        if p1.Val == p2.Val {
            return p1
        }
        p1 = p1.Next
        p2 = p2.Next
    }

    return nil
}
```













## reference

剑指offer

牛客网

LeetCode