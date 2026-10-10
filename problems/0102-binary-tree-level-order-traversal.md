# 102. 二叉树的层序遍历

- LeetCode：[102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)
- 难度：中等

## 先理解题意

层序遍历要求从上到下，一层一层地访问二叉树；同一层按照从左到右的顺序访问。

例如：

```text
       3
      / \
     9   20
        /  \
       15   7
```

分层结果是：

```text
第 1 层：[3]
第 2 层：[9, 20]
第 3 层：[15, 7]
```

最终返回：

```text
[[3], [9, 20], [15, 7]]
```

注意结果是二维数组，每一层单独放在一个数组中。

## 解题思路

使用广度优先搜索，也就是 BFS。

队列中保存接下来需要访问的节点：

1. 先把根节点加入队列。
2. 记录当前队列中属于本层的节点数量。
3. 只处理这么多个节点，并把它们的值加入当前层结果。
4. 将这些节点的左右子节点加入队列，供下一层处理。
5. 当前层处理完后，把当前层数组加入最终结果。

最关键的是：

> 开始处理一层之前，先固定这一层的节点数量。

处理本层节点时，新加入的子节点属于下一层，不能在本轮一起处理。

## 用到的数据结构及原因

### 队列 `queue`

队列遵循先进先出：先加入的节点先处理。

选择队列的原因：

1. 根节点最先处理。
2. 当前层的节点会先于它们的子节点处理。
3. 当前层从左到右加入子节点，下一层也会保持从左到右的顺序。

### 读取下标 `front`

JavaScript 数组可以使用 `push()` 在末尾加入节点，但频繁使用 `shift()` 删除第一个元素可能导致后面的元素整体移动。

因此保留数组中的节点，用 `front` 表示下一个要读取的位置：

```text
node = queue[front]
front++
```

这样每次取出队首节点都是常数时间。

### 二维结果数组 `result`

`result` 保存所有层，内部的 `level` 数组保存当前层节点值：

```text
result = [level1, level2, level3, ...]
```

## 用到的方法及原因

### 广度优先搜索 BFS

BFS 会先访问距离根节点较近的节点，再访问更深的节点，天然符合一层一层遍历的要求。

### `queue.push(node)`

把当前节点的左、右子节点加入队尾，等待后续处理。

### 固定 `levelSize`

一层开始前计算：

```text
levelSize = queue.length - front
```

`queue.length` 是队列数组中已经加入的节点总数，`front` 是已经读取的节点数量，它们的差就是当前尚未处理、且属于本层的节点数量。

必须在处理这一层之前保存 `levelSize`，因为处理过程中会继续向 `queue` 添加下一层节点。

## 过程示例

对于：

```text
       3
      / \
     9   20
```

开始时：

```text
queue = [3]
front = 0
levelSize = 1
```

只处理一个节点 `3`，并将它的子节点加入队列：

```text
当前层：[3]
queue = [3, 9, 20]
front = 1
```

下一轮开始：

```text
levelSize = queue.length - front
          = 3 - 1
          = 2
```

所以这一层只处理 `9` 和 `20`。

## 伪代码

```text
如果 root 为空：
    返回空数组

queue = [root]
front = 0
result = []

当 front 小于 queue.length：
    levelSize = queue.length - front
    level = []

    重复 levelSize 次：
        读取 queue[front]
        front 加 1

        将节点值加入 level

        如果左子节点存在：
            加入 queue

        如果右子节点存在：
            加入 queue

    将 level 加入 result

返回 result
```

## JavaScript 代码

```js
/**
 * @param {TreeNode} root
 * @return {number[][]}
 */
var levelOrder = function (root) {
  if (root === null) {
    return [];
  }

  const result = [];
  const queue = [root];
  let front = 0;

  while (front < queue.length) {
    const levelSize = queue.length - front;
    const level = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue[front];
      front++;

      level.push(node.val);

      if (node.left !== null) {
        queue.push(node.left);
      }

      if (node.right !== null) {
        queue.push(node.right);
      }
    }

    result.push(level);
  }

  return result;
};
```

## 复杂度分析

设二叉树有 `n` 个节点：

- 时间复杂度：`O(n)`。每个节点进入队列一次，也被读取一次。
- 空间复杂度：`O(n)`。最坏情况下队列需要保存同一层的大量节点；返回结果也包含所有节点值。

## 为什么不直接一直处理到队列为空

如果没有固定当前层的节点数量，而是在同一个循环中不断处理新加入的节点，就无法知道哪里是一层的结束位置，最终只能得到一维遍历结果：

```text
[3, 9, 20, 15, 7]
```

题目需要的是：

```text
[[3], [9, 20], [15, 7]]
```

所以必须在每层开始时记录 `levelSize`。

## 容易出错的地方

### 1. 空树没有单独处理

如果 `root === null`，应该直接返回：

```text
[]
```

不能把 `null` 加入队列后再访问 `node.val`。

### 2. 在处理本层的过程中重新计算层大小

子节点会不断加入队列，使 `queue.length` 变化。`levelSize` 必须在进入本层循环前固定下来。

### 3. 左右子节点加入顺序写反

题目要求同一层从左到右，所以应该先加入左子节点，再加入右子节点。

### 4. 使用 `shift()` 作为队首操作

`shift()` 写法更直观，但可能移动数组中剩余元素。使用读取下标 `front` 可以避免这一开销。

### 5. 返回了一维数组

每一层都需要创建独立的 `level` 数组，处理完一层后再放进 `result`。

### 6. 把 DFS 和 BFS 的访问顺序混淆

前序、 中序、后序 DFS 会优先深入某一条分支；层序遍历需要优先访问同一深度的节点，因此更适合使用 BFS 队列。
