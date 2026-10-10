# 143. 重排链表

- LeetCode：[143. 重排链表](https://leetcode.cn/problems/reorder-list/)
- 难度：中等

## 先理解题意

原链表顺序是：

```text
L0 -> L1 -> L2 -> ... -> Ln-1 -> Ln
```

重排后需要变成：

```text
L0 -> Ln -> L1 -> Ln-1 -> L2 -> Ln-2 -> ...
```

也就是依次选择：

```text
第一个、最后一个、第二个、倒数第二个……
```

例如：

```text
1 -> 2 -> 3 -> 4 -> 5
```

重排后：

```text
1 -> 5 -> 2 -> 4 -> 3
```

题目要求直接修改节点之间的 `next` 指针，不能只修改节点中的值。

## 解题思路

这道题可以拆成三个步骤：

```text
1. 使用快慢指针找到链表中点
2. 断开链表，并反转后半段
3. 将前半段和反转后的后半段交替合并
```

例如：

```text
原链表：1 -> 2 -> 3 -> 4 -> 5
```

找到中点并断开：

```text
前半段：1 -> 2 -> 3
后半段：4 -> 5
```

反转后半段：

```text
5 -> 4
```

交替合并：

```text
1 -> 5 -> 2 -> 4 -> 3
```

## 用到的数据结构及原因

### 链表节点指针

整个过程只需要若干节点指针，不需要把链表节点存入数组。

主要指针包括：

- `slow`、`fast`：寻找中点。
- `prev`、`current`、`next`：反转后半段。
- `first`、`second`：交替合并两段链表。

选择指针操作的原因：

1. 可以直接修改原链表。
2. 不需要额外保存全部节点。
3. 额外空间复杂度为 `O(1)`。

## 第一步：找到中点

让：

- `slow` 每次走一步。
- `fast` 每次走两步。

循环条件是：

```text
fast.next 不为空，并且 fast.next.next 不为空
```

循环结束后，`slow` 位于前半段最后一个节点。

### 奇数长度

```text
1 -> 2 -> 3 -> 4 -> 5
          ^
         slow
```

前半段保留中间节点 `3`：

```text
前半段：1 -> 2 -> 3
后半段：4 -> 5
```

### 偶数长度

```text
1 -> 2 -> 3 -> 4
     ^
    slow
```

分成：

```text
前半段：1 -> 2
后半段：3 -> 4
```

## 第二步：断开并反转后半段

先保存后半段头节点：

```text
second = slow.next
```

然后断开两段：

```text
slow.next = null
```

这一步很重要。如果不断开，原来的连接仍然存在，后续合并时可能形成环。

随后使用第 206 题的三指针方法反转后半段：

```text
4 -> 5 -> null
```

变成：

```text
5 -> 4 -> null
```

## 第三步：交替合并

使用：

```text
first 指向前半段
second 指向反转后的后半段
```

每轮先保存两段各自的下一个节点：

```text
firstNext = first.next
secondNext = second.next
```

然后连接：

```text
first.next = second
second.next = firstNext
```

最后让两个指针分别移动到保存的位置，继续下一轮。

## 过程示例

合并：

```text
前半段：1 -> 2 -> 3
后半段：5 -> 4
```

第一轮先保存：

```text
firstNext = 2
secondNext = 4
```

连接 `1` 和 `5`：

```text
1 -> 5 -> 2 -> 3
```

第二轮处理 `2` 和 `4`：

```text
1 -> 5 -> 2 -> 4 -> 3
```

后半段处理完毕，重排结束。

## 伪代码

```text
如果链表为空或只有一个节点：
    直接结束

slow 和 fast 都指向 head

当 fast 后面至少还有两个节点：
    slow 走一步
    fast 走两步

second = slow.next
slow.next = null

使用三指针反转 second 链表

first = head
second = 反转后的后半段头节点

当 second 不为空：
    保存 first.next
    保存 second.next

    first.next = second
    second.next = 保存的 first.next

    first 和 second 分别向后移动
```

## JavaScript 代码

```js
/**
 * @param {ListNode} head
 * @return {void} Do not return anything, modify head in-place instead.
 */
var reorderList = function (head) {
  if (head === null || head.next === null) {
    return;
  }

  // 第一步：找到前半段最后一个节点
  let slow = head;
  let fast = head;

  while (fast.next !== null && fast.next.next !== null) {
    slow = slow.next;
    fast = fast.next.next;
  }

  // 第二步：断开并反转后半段
  let current = slow.next;
  slow.next = null;

  let prev = null;

  while (current !== null) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }

  // 第三步：交替合并两段链表
  let first = head;
  let second = prev;

  while (second !== null) {
    const firstNext = first.next;
    const secondNext = second.next;

    first.next = second;
    second.next = firstNext;

    first = firstNext;
    second = secondNext;
  }
};
```

## 为什么合并循环只判断 `second !== null`

前半段节点数量总是大于或等于后半段：

- 偶数长度时，两段节点数量相同。
- 奇数长度时，前半段比后半段多一个中间节点。

所以只要后半段全部插入，前半段剩余部分自然已经位于正确位置。

## 复杂度分析

设链表有 `n` 个节点：

- 时间复杂度：`O(n)`。找中点、反转和合并分别都是线性遍历。
- 空间复杂度：`O(1)`。只使用有限个节点指针。

## 容易出错的地方

### 1. 找到中点后没有断开链表

必须执行：

```text
slow.next = null
```

否则反转和重新连接时可能保留旧连接，形成环或错误顺序。

### 2. 反转后半段时丢失后续节点

修改 `current.next` 之前必须保存：

```text
next = current.next
```

### 3. 合并时没有先保存两个 `next`

连接操作会修改 `first.next` 和 `second.next`。必须在修改前分别保存原来的下一个节点。

### 4. 交换节点值代替重排节点

题目要求改变节点连接顺序，不能只重新排列 `val`。

### 5. 忘记空链表或单节点

这两种情况不需要重排，应直接结束。

### 6. 返回一个新链表

本题要求原地修改 `head` 指向的链表，函数不需要返回结果。

## 与其他题目的关系

这道题组合了几个常见链表技巧：

- 第 141 题的快慢指针思想：寻找链表中点。
- 第 206 题的三指针操作：反转后半段链表。
- 第 88 题类似的双路合并思想：交替取两段中的节点。

把三个步骤分别掌握后，这道题就不需要作为一个全新的整体死记。
