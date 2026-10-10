# 19. 删除链表的倒数第 N 个结点

- LeetCode：[19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
- 难度：中等

## 先理解题意

给定链表：

```text
1 -> 2 -> 3 -> 4 -> 5
```

如果 `n = 2`，需要删除倒数第二个节点，也就是节点 `4`：

```text
1 -> 2 -> 3 -> 5
```

倒数位置为：

```text
倒数第 1 个：5
倒数第 2 个：4
倒数第 3 个：3
倒数第 4 个：2
倒数第 5 个：1
```

## 解题思路

使用快慢指针，让 `fast` 先向前移动 `n` 步，使 `fast` 和 `slow` 保持固定距离。

然后同时移动两个指针：

- 当 `fast` 到达最后一个节点时，`slow` 正好位于待删除节点的前一个节点。
- 执行 `slow.next = slow.next.next`，跳过待删除节点。

为了统一处理删除头节点的情况，两个指针都从虚拟头节点 `dummy` 开始。

## 用到的数据结构及原因

### 虚拟头节点 `dummy`

在原头节点前创建一个虚拟节点：

```text
dummy -> 1 -> 2 -> 3 -> 4 -> 5
```

选择虚拟头节点的原因：

1. 删除节点需要找到它的前一个节点。
2. 如果删除的是原头节点，它在原链表中没有前一个节点。
3. 添加 `dummy` 后，原头节点的前一个节点就是 `dummy`，所有位置都能使用同一套删除逻辑。

最终返回 `dummy.next`，因为链表头可能已经改变。

### 快慢指针

- `fast`：先向前移动 `n` 步，然后与慢指针一起移动。
- `slow`：最终停在待删除节点的前一个节点。

使用两个指针可以在一次遍历中找到倒数第 `n` 个节点，而不需要先计算链表长度。

## 用到的方法及原因

### 固定距离的快慢指针

开始时：

```text
fast = dummy
slow = dummy
```

先让 `fast` 向前移动 `n` 步。之后同时移动两个指针，直到 `fast` 到达最后一个节点。

由于它们之间始终保持固定距离，此时 `slow.next` 就是倒数第 `n` 个节点。

### 修改 `next` 指针删除节点

假设当前结构是：

```text
slow -> 待删除节点 -> 下一个节点
```

执行：

```text
slow.next = slow.next.next
```

就会让 `slow` 直接连接到后面的节点，从链表中跳过待删除节点。

## 过程示例

对于：

```text
1 -> 2 -> 3 -> 4 -> 5
n = 2
```

添加虚拟头节点：

```text
dummy -> 1 -> 2 -> 3 -> 4 -> 5
```

`fast` 先移动两步：

```text
slow = dummy
fast = 2
```

然后同时移动，直到 `fast` 到达最后一个节点：

```text
第一次：slow = 1，fast = 3
第二次：slow = 2，fast = 4
第三次：slow = 3，fast = 5
```

此时：

```text
slow.next = 4
```

执行删除后得到：

```text
1 -> 2 -> 3 -> 5
```

## 伪代码

```text
创建 dummy，并让 dummy.next 指向 head
fast = dummy
slow = dummy

让 fast 向前移动 n 步

当 fast.next 不为空：
    fast 向前移动一步
    slow 向前移动一步

此时 slow.next 是待删除节点
让 slow.next 指向 slow.next.next

返回 dummy.next
```

## JavaScript 代码

```js
/**
 * @param {ListNode} head
 * @param {number} n
 * @return {ListNode}
 */
var removeNthFromEnd = function (head, n) {
  const dummy = new ListNode(0);
  dummy.next = head;

  let fast = dummy;
  let slow = dummy;

  for (let i = 0; i < n; i++) {
    fast = fast.next;
  }

  while (fast.next !== null) {
    fast = fast.next;
    slow = slow.next;
  }

  slow.next = slow.next.next;

  return dummy.next;
};
```

## 为什么是先走 `n` 步，再判断 `fast.next`

目标是让 `slow` 停在待删除节点的前一个节点，而不是待删除节点本身。

因此采用：

```text
fast 先移动 n 步
随后移动到 fast.next 为空为止
```

当 `fast` 位于最后一个节点时，`slow` 正好位于待删除节点前面。

也可以让 `fast` 先移动 `n + 1` 步，再循环判断 `fast !== null`。两种写法本质相同，但移动步数和循环条件不能混用。

## 删除头节点的情况

例如：

```text
1 -> 2 -> 3
n = 3
```

`fast` 移动三步后位于节点 `3`，`fast.next` 是 `null`，所以两个指针不再一起移动。`slow` 仍位于 `dummy`，执行删除后得到：

```text
dummy -> 2 -> 3
```

返回 `dummy.next`，结果就是 `2 -> 3`。

## 复杂度分析

设链表有 `L` 个节点：

- 时间复杂度：`O(L)`。快慢指针都只向前移动，不会回退。
- 空间复杂度：`O(1)`。只使用虚拟节点和两个指针。

## 容易出错的地方

### 1. 没有使用虚拟头节点

如果删除的是原头节点，就没有前一个节点可以修改。使用 `dummy` 可以统一处理所有位置。

### 2. 返回原来的 `head`

删除的可能正是原头节点，因此应该返回 `dummy.next`。

### 3. 快指针移动步数与循环条件混用

下面两种方案都可以，但只能选择其中一种：

```text
方案一：先走 n 步，循环检查 fast.next
方案二：先走 n + 1 步，循环检查 fast
```

搭配错误会让慢指针停错位置。

### 4. 删除时只移动了 `slow`

移动局部变量 `slow` 不会删除节点，必须修改前一个节点的连接：

```text
slow.next = slow.next.next
```

### 5. 把倒数第 `n` 个理解成下标

`n` 从 `1` 开始：倒数第 `1` 个是尾节点。
