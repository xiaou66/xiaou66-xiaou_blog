---
title: B树
date: 2020/08/04 09:00
math: true
categories: 
- [数据结构, 树]
tags:
- [java]
---

# B  树

:::info

B树(balanced Tree)是一种平衡的多路搜索树，多用于文件系统、数据库的实现

:::

## 特点

1. 1 个节点可以存储超过 2 个元素、可以拥有超过 2 个子节点
2. 拥有二叉搜索树的一些性质
3. 平衡，每个节点的所有子树高度一致
4. B树中的阶指的是一个结点最多拥有的结点个数

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628038764964-tree_28.png)

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628038751225-tree_27.png)

## M 阶B树的性质

假设一个结点存储的元素个数是 m

1. 根结点 $1 \le x \le m-1$
2. 非根结点 $\left \lceil m/2 \right \rceil -1 \le x \le m-1$

如果有子结点，子结点个数 $y = x + 1$

1. 根结点: $2 \le y \le m$

2. 非根结点  $\left \lceil m/2 \right \rceil  \le x \le m$

比如

- $m = 3, 2 \le y \le 3$ 可以称为 2-3 树
- $m = 4, 2 \le y \le 4$ 可以称为 2-3-4 树

## B树与BST树关系

1. B树 和 二叉搜索树，在逻辑上是等价的
2. 多代结点合并，可以获得一个超级结点
   - 2 代合并的超级节点，最多拥有 4 个子节点（至少是 4 阶B树）
   - 3 代合并的超级节点，最多拥有 8 个子节点（至少是 8 阶B树）
   - n 代合并的超级节点，最多拥有 $2^n$ 个子节点（ 至少是 $2^n$ 阶B树）

:::info

m 阶B树，最多需要 $\log_{2}{m\\}$ 代合并

:::

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628043074010-tree_29.png)

## 搜索

:::info

跟二叉搜索树的搜索类似

:::

1. 先在节点内部从小到大开始搜索元素
2. 如果命中，搜索结束
3. 如果未命中，再去对应的子节点中搜索元素，重复步骤 1

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628044078470-tree_30.png)

## 添加

新添加的元素必定是添加到叶子节点

1. 正常插入
   - 插入 「95」

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628051284735-tree_31.png)

2. 当插入时叶子结点的元素个数超过限制时需要进行「上溢」操作

   - 插入 「101」

   :::default

   如果上溢结点最中间元素的位置为 k

   1. 将 k 位置的元素向上与父结点合并
   2. 将$[0，k - 1]$ 和 $[k + 1, m - 1]$ 位置的元素分裂成 2 个子结点
      - 这两个子结点的元素个数，必然都不会低于最低限制 $\left \lceil m/2 \right \rceil  - 1$

   :::

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628052522553-tree_32.png)

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628052555357-tree_34.png)



![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628052539506-tree_33.png)



## 删除

### 叶子结点

1. 假如需要删除的元素在叶子节点中，那么直接删除即可

### 非叶子结点

1. 先找到前驱或后继元素，覆盖所需删除元素的值

2. 再把前驱或后继元素删除

   :::info

   非叶子节点的前驱或后继元素，必定在叶子节点中

   真正的删除元素都是发生在叶子节点中

   :::

### 删除造成下溢

叶子节点被删掉一个元素后，元素个数可能会低于最低限制($\ge\left \lceil m/2 \right \rceil  - 1$)

#### 解决方案

- 如果下溢节点临近的兄弟节点，有至少 $\left \lceil m/2 \right \rceil$ 个元素，可以向其借一个元素

    1. 将父节点的元素 b 插入到下溢节点的 0 位置（最小位置）
    2. 用兄弟节点的元素 a（最大的元素）替代父节点的元素 b

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628062052845-tree_35.png)

- 如果下溢节点临近的兄弟节点，只有 $\left \lceil m/2 \right \rceil - 1$ 个元素

    1. 将父节点的元素 b 挪下来跟左右子节点进行合并
    2. 合并后的节点元素个数等于$\left \lceil m/2 \right \rceil + \left \lceil m/2 \right \rceil - 2 \le m − 1$
    3. 这个操作可能会导致父节点下溢，依然按照上述方法解决，下溢现象可能会一直往上传播

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628062073390-tree_36.png)

## 4 阶 B 树的性质

1. 所有节点能存储的元素个数 x ：$1 \le x  \le 3$
2. 所有非叶子节点的子节点个数 y ：$2 \le y  \le 4$

