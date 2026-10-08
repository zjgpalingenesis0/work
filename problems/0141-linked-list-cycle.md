# 141. 环形链表

- LeetCode：[141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)
- 难度：简单

## 解题思路

链表有环，表示从某个节点开始，沿着 `next` 指针不断向后移动时，会再次到达之前经过的节点。

例如：

```text
A -> B -> C -> D
          ^    |
          |____|
```

从 `C` 到 `D` 后又回到 `C`，因此永远无法走到 `null`。

可以使用两种方式判断：

1. 使用 `Set` 记录访问过的节点。如果再次遇到同一个节点，说明存在环。
2. 使用快慢指针。慢指针每次走一步，快指针每次走两步；如果存在环，快指针最终会追上慢指针。

## 方法一：使用 `Set`

### 用到的数据结构及原因

#### `Set`（哈希集合）

`Set` 保存已经访问过的链表节点。

选择 `Set` 的原因：

1. 可以平均 `O(1)` 地判断一个节点是否访问过。
2. `Set` 存储的是节点对象本身，可以区分值相同但实际不同的两个节点。
3. 思路直观：重复访问同一个节点就代表存在环。

不能只记录节点值，因为不同节点可能拥有相同的 `val`，值相同并不表示出现了环。

### 用到的方法及原因

#### `visited.has(node)`

判断当前节点对象是否已经访问过。

#### `visited.add(node)`

把当前节点对象加入访问记录。

### 伪代码

```text
创建空 Set
current 指向头节点

当 current 不为空：
    如果 current 已经存在于 Set：
        返回 true

    将 current 加入 Set
    current 移动到下一个节点

返回 false
```

### JavaScript 代码

```js
/**
 * @param {ListNode} head
 * @return {boolean}
 */
var hasCycle = function (head) {
  const visited = new Set();
  let current = head;

  while (current !== null) {
    if (visited.has(current)) {
      return true;
    }

    visited.add(current);
    current = current.next;
  }

  return false;
};
```

### 复杂度分析

- 时间复杂度：`O(n)`。每个节点最多访问一次。
- 空间复杂度：`O(n)`。最坏情况下需要记录所有节点。

## 方法二：快慢指针

### 解题思路

设置两个指针：

- `slow` 每次向后移动一个节点。
- `fast` 每次向后移动两个节点。

如果链表没有环，`fast` 会先走到链表末尾，也就是遇到 `null`。

如果链表存在环，两个指针进入环后会一直在环中移动。因为 `fast` 每轮比 `slow` 多走一步，所以它们之间的距离会不断缩小，最终一定会指向同一个节点。

可以把它想成环形跑道：一个人每次走一步，另一个人每次走两步。只要两个人一直在跑道上，速度快的人最终一定会从后面追上速度慢的人。

### 用到的数据结构及原因

#### 两个节点指针

只需要保存：

- `slow`：慢指针。
- `fast`：快指针。

不需要额外数组或哈希集合，因此空间复杂度为 `O(1)`。

### 用到的方法及原因

#### 快慢指针

利用两个指针的速度差判断是否存在环。

选择快慢指针的原因：

1. 时间复杂度仍然是 `O(n)`。
2. 空间复杂度只有 `O(1)`，优于 `Set` 方法。
3. 不会修改原链表结构。

### 为什么快指针不会跳过慢指针

进入环后，`fast` 每轮比 `slow` 多前进一个节点。从环上相对位置来看，它们之间的距离每轮恰好减少一个，因此一定会出现距离为 `0` 的时刻，不会永远错开。

### 伪代码

```text
slow 指向头节点
fast 指向头节点

当 fast 不为空，并且 fast.next 不为空：
    slow 向后移动一步
    fast 向后移动两步

    如果 slow 和 fast 指向同一个节点：
        返回 true

返回 false
```

### JavaScript 代码

```js
/**
 * @param {ListNode} head
 * @return {boolean}
 */
var hasCycle = function (head) {
  let slow = head;
  let fast = head;

  while (fast !== null && fast.next !== null) {
    slow = slow.next;
    fast = fast.next.next;

    if (slow === fast) {
      return true;
    }
  }

  return false;
};
```

### 复杂度分析

- 时间复杂度：`O(n)`。无环时快指针走到末尾；有环时快慢指针会在有限步内相遇。
- 空间复杂度：`O(1)`。只使用两个节点指针。

## 两种方法如何选择

| 方法 | 时间复杂度 | 空间复杂度 | 特点 |
| --- | --- | --- | --- |
| `Set` | `O(n)` | `O(n)` | 更直观，容易想到 |
| 快慢指针 | `O(n)` | `O(1)` | 不需要额外存储，是本题推荐解法 |

学习时可以先用 `Set` 理解“重复访问节点就是有环”，再掌握快慢指针如何通过速度差检测环。

## 容易出错的地方

### 1. 比较节点值而不是节点对象

以下判断不正确：

```text
slow.val === fast.val
```

两个不同节点可能拥有相同的值。必须判断它们是否指向同一个节点：

```text
slow === fast
```

### 2. 快指针移动前没有判断 `fast.next`

`fast` 每次需要移动两步，所以循环条件必须同时检查：

```text
fast !== null && fast.next !== null
```

否则访问 `fast.next.next` 时可能出现错误。

### 3. 在移动指针前直接比较

如果 `slow` 和 `fast` 都从 `head` 开始，在第一次移动前它们必然相等，但这不能说明链表存在环。

因此应该先移动两个指针，再判断是否相遇。

### 4. 在 `Set` 中存储节点值

应该存储节点对象：

```text
visited.add(current)
```

而不是存储：

```text
visited.add(current.val)
```

### 5. 将“存在环”和“寻找环入口”混在一起

本题只要求判断是否存在环，快慢指针相遇后直接返回 `true` 即可，不需要继续寻找环的入口节点。
