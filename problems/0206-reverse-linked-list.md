# 206. 反转链表

- LeetCode：[206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)
- 难度：简单

## 先理解题意

给定链表：

```text
1 -> 2 -> 3 -> 4 -> null
```

需要修改每个节点的 `next` 指针，让链表方向反过来：

```text
4 -> 3 -> 2 -> 1 -> null
```

题目要求反转的是节点之间的连接方向，不是只把节点中的值倒过来。

## 解题思路

从左向右遍历原链表，每处理一个节点，就把它的 `next` 指针改为指向前一个节点。

处理当前节点时，需要三个指针：

- `prev`：已经完成反转部分的头节点。
- `current`：当前正在处理的节点。
- `next`：提前保存原链表中 `current` 后面的节点。

初始状态：

```text
prev = null
current = 1

null    1 -> 2 -> 3 -> 4 -> null
  ^     ^
 prev current
```

处理节点 `1` 时，想让它指向 `null`：

```text
1 -> null
```

但是修改 `1.next` 之前，必须先保存节点 `2`。否则把 `1.next` 改成 `null` 后，就无法再找到后面的 `2、3、4`。

所以每轮固定执行四步：

```text
1. next = current.next
2. current.next = prev
3. prev = current
4. current = next
```

## 用到的数据结构及原因

### 链表节点

直接修改原链表节点的 `next` 指针，不需要创建新的链表节点，也不需要把节点放入数组。

这样做的原因：

1. 可以原地完成反转。
2. 不需要额外保存所有节点。
3. 空间复杂度为 `O(1)`。

### 三个节点指针

#### `prev`

指向已经反转完成部分的头节点。

随着反转进行，它会依次指向：

```text
null -> 1 -> 2 -> 3 -> 4
```

这里表示 `prev` 每轮更新后所指向的节点，不表示这些节点仍按箭头方向排列。

#### `current`

指向原链表中当前需要处理的节点。

当 `current` 变成 `null` 时，说明所有节点都处理完毕。

#### `next`

临时保存当前节点在原链表中的下一个节点，防止反转指针后丢失剩余链表。

## 用到的方法及原因

### 迭代原地反转

遍历链表并逐个改变 `next` 指针的方向。

选择迭代方法的原因：

1. 每个节点只处理一次，时间复杂度为 `O(n)`。
2. 只使用三个节点指针，空间复杂度为 `O(1)`。
3. 相比递归方法，不需要使用函数调用栈。

## 完整过程示例

原链表：

```text
1 -> 2 -> 3 -> null
```

### 第一轮

```text
prev = null
current = 1
next = 2
```

执行 `current.next = prev`：

```text
1 -> null

剩余待处理：2 -> 3 -> null
```

移动指针后：

```text
prev = 1
current = 2
```

### 第二轮

先保存：

```text
next = 3
```

再反转节点 `2`：

```text
2 -> 1 -> null
```

移动指针后：

```text
prev = 2
current = 3
```

### 第三轮

先保存：

```text
next = null
```

再反转节点 `3`：

```text
3 -> 2 -> 1 -> null
```

移动指针后：

```text
prev = 3
current = null
```

循环结束。此时 `prev` 指向反转后链表的新头节点，所以返回 `prev`。

## 伪代码

```text
prev = null
current = head

当 current 不为空：
    next = current.next
    current.next = prev
    prev = current
    current = next

返回 prev
```

## JavaScript 代码

```js
/**
 * @param {ListNode} head
 * @return {ListNode}
 */
var reverseList = function (head) {
  let prev = null;
  let current = head;

  while (current !== null) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }

  return prev;
};
```

## 为什么最后返回 `prev`

循环结束的条件是：

```text
current === null
```

这说明 `current` 已经越过原链表的最后一个节点。最后一个被处理的节点已经赋值给 `prev`，而它正是反转后链表的新头节点。

例如：

```text
原链表头：1
反转后头：4
```

循环结束时：

```text
prev 指向 4
current 指向 null
```

所以返回 `prev`。

## 复杂度分析

设链表有 `n` 个节点：

- 时间复杂度：`O(n)`。每个节点只访问一次。
- 空间复杂度：`O(1)`。只使用三个节点指针。

## 容易出错的地方

### 1. 修改 `current.next` 前没有保存后续节点

错误顺序：

```text
current.next = prev
next = current.next
```

此时 `current.next` 已经被修改，无法再得到原来的下一个节点。

正确顺序必须是：

```text
next = current.next
current.next = prev
```

### 2. 只交换节点值

本题要求反转链表连接关系，应修改 `next` 指针，而不是交换所有节点的 `val`。

### 3. 指针移动顺序错误

反转完成后，应该先让：

```text
prev = current
```

再让：

```text
current = next
```

### 4. 最后返回 `head` 或 `current`

- 原来的 `head` 在反转后是尾节点。
- 循环结束时 `current` 是 `null`。
- 新的头节点是 `prev`。

### 5. 没有处理空链表

如果 `head` 是 `null`，循环不会执行，直接返回初始值 `prev`，也就是 `null`，因此当前写法可以自然处理空链表。

## 与第 25 题的关系

第 25 题“K 个一组翻转链表”的组内翻转，本质上就是本题的四步操作：

```text
保存下一个节点
反转 current.next
移动 prev
移动 current
```

区别是本题一直反转到 `null`，而第 25 题只反转到当前组结束位置 `groupNext` 之前。
