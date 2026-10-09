# 23. 合并 K 个升序链表

- LeetCode：[23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)
- 难度：困难

## 解题思路

每条链表自身已经升序，因此合并后的最小节点一定来自某条链表当前的头节点。

假设有三条链表：

```text
1 -> 4 -> 5
1 -> 3 -> 4
2 -> 6
```

一开始只需要比较三个头节点：

```text
1、1、2
```

取出其中最小的节点后，再把它的下一个节点加入候选范围。重复这个过程，就能按照从小到大的顺序连接所有节点。

关键问题是：如何快速找到 `k` 个候选节点中的最小节点？

可以使用最小堆。最小堆的堆顶始终是当前值最小的节点，插入和删除堆顶的时间复杂度都是 `O(log k)`。

## 方法一：最小堆

### 用到的数据结构及原因

#### 最小堆

堆中最多保存每条链表的一个候选节点，也就是每条链表当前尚未合并的第一个节点。

选择最小堆的原因：

1. 可以在 `O(1)` 时间内取得当前最小节点。
2. 删除最小节点和插入新节点都是 `O(log k)`。
3. 堆中最多只有 `k` 个节点，不需要一次性存储全部节点。

JavaScript 没有适用于本题的通用内置优先队列，因此需要使用数组实现一个最小堆。

#### 虚拟头节点 `dummy`

创建一个不属于最终数据的虚拟节点，让 `tail` 从它开始连接结果。

选择虚拟头节点的原因：

1. 不需要单独处理结果链表的第一个节点。
2. 每次取出节点后都可以统一执行 `tail.next = node`。
3. 最终返回 `dummy.next` 即可。

### 最小堆需要的方法

#### `push(node)`

把新节点放到堆数组末尾，再不断与父节点比较并向上交换，恢复最小堆性质。

#### `pop()`

取出堆顶最小节点。将数组末尾节点移动到堆顶，再不断与较小的子节点交换并向下移动，恢复最小堆性质。

#### 父子下标关系

对于堆数组中下标为 `index` 的节点：

```text
父节点下标：floor((index - 1) / 2)
左子节点下标：index * 2 + 1
右子节点下标：index * 2 + 2
```

### 合并过程

```text
1. 将所有非空链表的头节点放入最小堆。
2. 从堆中取出值最小的节点，连接到结果链表末尾。
3. 如果该节点还有下一个节点，就把下一个节点放入堆中。
4. 重复以上操作，直到堆为空。
```

注意：取出一个节点后，只加入它的下一个节点，而不是把整条链表全部加入堆中。因为每条链表已经有序，下一个节点才是这条链表新的最小候选节点。

### 伪代码

```text
创建最小堆

遍历所有链表：
    如果链表头节点不为空：
        将头节点加入最小堆

创建虚拟头节点 dummy
tail 指向 dummy

当最小堆不为空：
    取出堆顶最小节点 node
    将 node 连接到 tail 后面
    tail 移动到 node

    如果 node.next 不为空：
        将 node.next 加入最小堆

返回 dummy.next
```

### JavaScript 代码

```js
class MinHeap {
  constructor() {
    this.heap = [];
  }

  get size() {
    return this.heap.length;
  }

  push(node) {
    this.heap.push(node);
    this.bubbleUp(this.heap.length - 1);
  }

  pop() {
    if (this.heap.length === 1) {
      return this.heap.pop();
    }

    const minNode = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.sinkDown(0);

    return minNode;
  }

  bubbleUp(index) {
    while (index > 0) {
      const parent = Math.floor((index - 1) / 2);

      if (this.heap[parent].val <= this.heap[index].val) {
        break;
      }

      [this.heap[parent], this.heap[index]] =
        [this.heap[index], this.heap[parent]];
      index = parent;
    }
  }

  sinkDown(index) {
    const length = this.heap.length;

    while (true) {
      const leftChild = index * 2 + 1;
      const rightChild = index * 2 + 2;
      let smallest = index;

      if (
        leftChild < length &&
        this.heap[leftChild].val < this.heap[smallest].val
      ) {
        smallest = leftChild;
      }

      if (
        rightChild < length &&
        this.heap[rightChild].val < this.heap[smallest].val
      ) {
        smallest = rightChild;
      }

      if (smallest === index) {
        break;
      }

      [this.heap[index], this.heap[smallest]] =
        [this.heap[smallest], this.heap[index]];
      index = smallest;
    }
  }
}

/**
 * @param {ListNode[]} lists
 * @return {ListNode}
 */
var mergeKLists = function (lists) {
  const minHeap = new MinHeap();

  for (const head of lists) {
    if (head !== null) {
      minHeap.push(head);
    }
  }

  const dummy = new ListNode(0);
  let tail = dummy;

  while (minHeap.size > 0) {
    const node = minHeap.pop();
    tail.next = node;
    tail = node;

    if (node.next !== null) {
      minHeap.push(node.next);
    }
  }

  return dummy.next;
};
```

### 复杂度分析

设 `N` 是所有链表的节点总数，`k` 是链表数量：

- 时间复杂度：`O(N log k)`。每个节点都会进入和离开堆一次，堆中最多有 `k` 个节点。
- 空间复杂度：`O(k)`。最小堆最多保存每条链表的一个节点。

## 方法二：分治合并

### 解题思路

合并两个升序链表可以在线性时间内完成，因此也可以把 `k` 条链表两两合并：

```text
第一轮：每两条链表合并，k 条变成约 k / 2 条
第二轮：继续两两合并，变成约 k / 4 条
……
最后：只剩一条链表
```

这与归并排序的合并过程类似。

### 用到的数据结构及原因

#### 输入数组 `lists`

直接在 `lists` 中保存每轮两两合并后的链表头节点，不需要额外创建同等大小的辅助数组。

#### 虚拟头节点 `dummy`

在合并两条链表时统一处理结果链表的连接操作，避免单独判断第一个节点。

### 用到的方法及原因

#### 两两合并

每次比较两条链表的当前节点，将值更小的节点连接到结果链表。

#### 分治

每轮将链表数量减半，使每个节点只在大约 `log k` 轮合并中被处理。

与从左到右逐条合并相比，分治能够避免前面已经形成的长链表被重复扫描过多次。

### 伪代码

```text
定义 mergeTwoLists(list1, list2)：
    使用双指针合并两条升序链表
    返回合并后的头节点

interval = 1

当 interval 小于链表数量：
    每隔 interval * 2 选择一组链表：
        合并 lists[i] 和 lists[i + interval]
        将结果保存回 lists[i]

    interval 扩大两倍

返回 lists[0]
```

### JavaScript 代码

```js
var mergeTwoLists = function (list1, list2) {
  const dummy = new ListNode(0);
  let tail = dummy;

  while (list1 !== null && list2 !== null) {
    if (list1.val <= list2.val) {
      tail.next = list1;
      list1 = list1.next;
    } else {
      tail.next = list2;
      list2 = list2.next;
    }

    tail = tail.next;
  }

  tail.next = list1 !== null ? list1 : list2;
  return dummy.next;
};

/**
 * @param {ListNode[]} lists
 * @return {ListNode}
 */
var mergeKLists = function (lists) {
  if (lists.length === 0) {
    return null;
  }

  for (let interval = 1; interval < lists.length; interval *= 2) {
    for (
      let i = 0;
      i + interval < lists.length;
      i += interval * 2
    ) {
      lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
    }
  }

  return lists[0];
};
```

### 复杂度分析

- 时间复杂度：`O(N log k)`。一共有约 `log k` 轮，每轮所有节点总共被处理一次。
- 空间复杂度：`O(1)`。迭代写法复用输入数组和原链表节点，不计算返回链表所占空间。

## 两种方法如何选择

| 方法 | 时间复杂度 | 额外空间 | 特点 |
| --- | --- | --- | --- |
| 最小堆 | `O(N log k)` | `O(k)` | 思路直接，随时取当前最小节点 |
| 分治合并 | `O(N log k)` | `O(1)` | 复用合并两个链表，JavaScript 实现更短 |

如果重点是理解“多个有序来源中不断选择最小值”，学习最小堆；如果已经熟悉合并两个升序链表，分治写法通常更容易在 JavaScript 中实现。

## 容易出错的地方

### 1. 把所有节点一次性放入堆中

这样虽然也能得到正确结果，但额外空间会变成 `O(N)`。每条链表只需要保留当前的一个候选节点。

### 2. 取出节点后忘记加入它的下一个节点

堆顶节点被使用后，它所在链表的下一个节点会成为新的候选节点，必须加入堆中。

### 3. 忘记过滤空链表

初始化堆时只能加入非空头节点：

```text
head !== null
```

### 4. 分治合并时步长写错

当当前合并间隔是 `interval` 时，每一组的起点需要前进：

```text
interval * 2
```

每轮结束后，`interval` 也要扩大两倍。

### 5. 创建了全新的所有链表节点

本题可以直接修改原节点的 `next` 指针并复用节点，不需要为每个值创建新节点。
