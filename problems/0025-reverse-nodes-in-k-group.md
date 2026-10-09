# 25. K 个一组翻转链表

- LeetCode：[25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)
- 难度：困难

## 先理解题意

链表每 `k` 个节点组成一组，每组内部进行翻转。如果最后剩余节点不足 `k` 个，则保持原来的顺序。

例如：

```text
1 -> 2 -> 3 -> 4 -> 5
k = 2
```

分组是：

```text
[1, 2] [3, 4] [5]
```

前两组分别翻转，最后一组不足两个节点，不翻转：

```text
2 -> 1 -> 4 -> 3 -> 5
```

如果 `k = 3`：

```text
[1, 2, 3] [4, 5]
```

只有第一组翻转：

```text
3 -> 2 -> 1 -> 4 -> 5
```

## 解题思路

处理每一组时，需要完成四件事：

1. 从当前组前一个节点开始，向后寻找第 `k` 个节点。
2. 如果找不到，说明剩余节点不足 `k` 个，直接结束。
3. 翻转当前这 `k` 个节点。
4. 把翻转后的这一组与前后两部分重新连接。

为了统一处理第一组，需要在链表头部添加一个虚拟头节点 `dummy`。

## 用到的数据结构及原因

### 虚拟头节点 `dummy`

在原头节点前添加一个虚拟节点：

```text
dummy -> 1 -> 2 -> 3 -> 4 -> 5
```

选择虚拟头节点的原因：

1. 第一组翻转后，原来的头节点会改变。
2. 有了 `dummy`，每一组前面都一定有一个节点。
3. 所有分组都可以使用相同的连接逻辑。

最终返回：

```text
dummy.next
```

### 指针变量

处理一组时使用以下指针：

- `groupPrev`：当前组前面的一个节点。
- `kth`：当前组的第 `k` 个节点，也就是当前组最后一个节点。
- `groupNext`：下一组的第一个节点，即 `kth.next`。
- `current`：正在翻转的节点。
- `prev`：翻转后 `current` 应该指向的前一个节点。
- `next`：临时保存 `current.next`，防止修改指针后丢失后续链表。

只使用有限个指针，不需要额外数组，因此空间复杂度为 `O(1)`。

## 一组节点的边界

假设当前处理：

```text
groupPrev
    |
    v
前一部分 -> 1 -> 2 -> 3 -> 后一部分
             当前组       ^
                         groupNext
```

当 `k = 3` 时：

```text
kth = 节点 3
groupNext = 节点 3 的下一个节点
```

真正需要翻转的是一个左闭右开的范围：

```text
[groupPrev.next, groupNext)
```

也就是从当前组第一个节点开始，一直处理到 `groupNext` 之前。

## 用到的方法及原因

### 分组寻找第 `k` 个节点

从 `groupPrev` 开始向后移动 `k` 次：

```text
移动后非空：当前组有完整的 k 个节点
移动中遇到 null：剩余节点不足 k 个
```

### 链表原地翻转

翻转节点时执行：

```text
保存 current.next
让 current.next 指向 prev
prev 移动到 current
current 移动到之前保存的 next
```

### 为什么 `prev` 从 `groupNext` 开始

普通链表翻转通常让 `prev` 从 `null` 开始，但本题还需要让翻转后的当前组连接到下一组。

因此直接初始化：

```text
prev = groupNext
```

这样当前组原来的第一个节点在翻转后会自动指向下一组：

```text
翻转前：1 -> 2 -> 3 -> groupNext
翻转后：3 -> 2 -> 1 -> groupNext
```

不需要再单独处理当前组尾部与下一组的连接。

## 过程示例

处理：

```text
dummy -> 1 -> 2 -> 3 -> 4 -> 5
k = 2
```

第一组：

```text
groupPrev = dummy
kth = 2
groupNext = 3
```

翻转 `[1, 2]` 后：

```text
dummy -> 2 -> 1 -> 3 -> 4 -> 5
```

此时节点 `1` 是翻转后这一组的最后一个节点。下一轮应让：

```text
groupPrev = 1
```

第二组：

```text
groupPrev = 1
kth = 4
groupNext = 5
```

翻转后：

```text
dummy -> 2 -> 1 -> 4 -> 3 -> 5
```

剩余节点只有 `5`，不足两个节点，所以停止。

## 伪代码

```text
创建 dummy，并让 dummy.next 指向 head
groupPrev = dummy

不断处理下一组：
    从 groupPrev 开始向后寻找第 k 个节点 kth

    如果 kth 不存在：
        结束循环

    groupNext = kth.next
    oldGroupHead = groupPrev.next

    prev = groupNext
    current = oldGroupHead

    当 current 不等于 groupNext：
        next = current.next
        current.next = prev
        prev = current
        current = next

    groupPrev.next = kth
    groupPrev = oldGroupHead

返回 dummy.next
```

## JavaScript 代码

```js
var getKthNode = function (start, k) {
  let current = start;

  while (current !== null && k > 0) {
    current = current.next;
    k--;
  }

  return current;
};

/**
 * @param {ListNode} head
 * @param {number} k
 * @return {ListNode}
 */
var reverseKGroup = function (head, k) {
  const dummy = new ListNode(0);
  dummy.next = head;

  let groupPrev = dummy;

  while (true) {
    const kth = getKthNode(groupPrev, k);

    if (kth === null) {
      break;
    }

    const groupNext = kth.next;
    const oldGroupHead = groupPrev.next;

    let prev = groupNext;
    let current = oldGroupHead;

    while (current !== groupNext) {
      const next = current.next;
      current.next = prev;
      prev = current;
      current = next;
    }

    groupPrev.next = kth;
    groupPrev = oldGroupHead;
  }

  return dummy.next;
};
```

## 为什么连接时使用 `kth`

翻转前：

```text
groupPrev -> oldGroupHead -> ... -> kth -> groupNext
```

翻转后：

```text
groupPrev -> kth -> ... -> oldGroupHead -> groupNext
```

所以：

- `kth` 变成当前组的新头节点。
- `oldGroupHead` 变成当前组的新尾节点。

需要执行：

```text
groupPrev.next = kth
groupPrev = oldGroupHead
```

第二句是为下一轮做准备，因为当前组的新尾节点正好是下一组前面的节点。

## 复杂度分析

设链表节点总数为 `n`：

- 时间复杂度：`O(n)`。每个节点只会参与有限次数的寻找和翻转。
- 空间复杂度：`O(1)`。只使用有限个指针变量，没有使用递归栈或额外数组。

## 容易出错的地方

### 1. 剩余节点不足 `k` 个时仍然翻转

必须先找到当前组的第 `k` 个节点。如果找不到，应保持剩余节点原顺序并结束。

### 2. 翻转时丢失后续链表

修改 `current.next` 之前必须先保存：

```text
next = current.next
```

否则无法继续访问原链表的下一个节点。

### 3. `prev` 初始化为 `null`

如果把 `prev` 初始化为 `null`，当前组翻转后会与下一组断开。初始化为 `groupNext` 可以直接完成尾部连接。

### 4. 翻转后没有连接新的组头

原来的 `kth` 会变成当前组的新头节点，因此需要：

```text
groupPrev.next = kth
```

### 5. 下一轮的 `groupPrev` 选错

当前组原来的头节点在翻转后会变成尾节点，所以应该保存 `oldGroupHead`，并在翻转后执行：

```text
groupPrev = oldGroupHead
```

### 6. 使用节点值进行翻转

题目要求交换节点本身，而不是只交换节点中的 `val`。应该修改节点的 `next` 指针。
