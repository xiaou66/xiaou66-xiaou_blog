---
title: 树&二叉树
date: 2021/08/02 14:45
math: true
categories: 
- [数据结构, 树]
tags:
- [java]
---

# 树

## 树的基本概念

### 树结点

- **结点**: 使用树结构存储的每一个数据元素都被称为“结点”
- **父亲结点**: 在当前结点下还有结点并且相连，这个结点是被相连的结点的父结点
- **孩子结点**：父亲结点相连的结点
- **祖父结点**：父结点的父结点
- **兄弟结点**：拥有同一个父亲结点的结点
- **叶子结点**：没有孩子结点
- **非叶子结点**：至少有一个孩子结点
- **树根结点**：每一个非空树都有且只有一个被称为根的结点
- **前驱结点**：中序遍历时的前一个结点
- **后继结点**：中序遍历时的后一个结点

### 树中的度

- **结点的度**：一个父节点的孩子结点的个数

- **树的度**: 所有结点度中的最大值

### 层次

- **层次**： 根节点在第 1 层，根节点的子节点在第 2 层，以此类推

- **结点的深度**: 从根节点到当前节点的唯一路径上的节点总数

- **结点的高度**：从当前节点到最远叶子节点的路径上的节点总数

- **树的深度**: 所有节点深度中的最大值

- **树的高度**:  所有节点高度中的最大值

  :::info

   树的深度 等于 树的高度

  :::

### 子树和空树

- **空树**：如果集合本身为空，那么构成的树就被称为空树。空树中没有结点。
- **子树**：把所有结点都看成根结点都可以形成一颗子树
  ::: info
  单个结点也是一棵树，只不过根结点就是它本身
  :::

### 有序树和无序树

:::info

如果树中结点的子树从左到右看，谁在左边，谁在右边，是有规定的，这棵树称为有序树；反之称为无序树。在有序树中，一个结点最左边的子树称为"第一个孩子"，最右边的称为"最后一个孩子"。

:::

## 二叉树 

### 二叉树的特点

1. 每个节点的度最大为 2
2. 左子树和右子树是有顺序的(有序树)
3. 即使某节点只有一棵子树，也要区分左右子树

### 二叉树的性质

1. 非空二叉树的第 i 层，最多有 $2^{i-1}$ 个节点（ i ≥ 1 )
2. 在高度为 h 的二叉树上最多有 $2^h-1$ 个结点（ h ≥ 1 ）
3. n0 = n2 + 1

## 满二叉树

### 满二叉树的特点

1. 最后一层节点的度都为 0，其他节点的度都为 2
2. 在同样高度的二叉树中，满二叉树的叶子节点数量最多、总节点数量最多
3. 满二叉树一定是真二叉树，真二叉树不一定是满二叉树

### 满二叉树的性质

假设满二叉树的高度为 h（ h ≥ 1 ）

1. 第 i 层的节点数量: $2^{i-1}$个
2. 叶子结点数量: $2^{h-1}$ 个
3. 总结点数量: $2^h-1$个
4. $h = \log_{2}{(n+1\\})$

## 完全二叉树

### 完全二叉树的特点

1. 对节点从上至下、左至右开始编号，其所有编号都能与相同高度的满二叉树中的编号对应
2. 叶子节点只会出现最后 2 层，最后 1 层的叶子结点都靠左对齐
3. 完全二叉树从根结点至倒数第 2 层是一棵满二叉树
4. 满二叉树一定是完全二叉树，完全二叉树不一定是满二叉树

### 完全二叉树的性质

1. 度为 1 的节点只有左子树
2. 度为 1 的节点要么是 1 个，要么是 0 个
3. 同样节点数量的二叉树，完全二叉树的高度最小

假设完全二叉树的高度为 h（ h ≥ 1 ）

1. 至少有$2^{h-1}$个结点
2. 最多有$2^h-1$个结点

总结点数量为 n
$$
\begin{aligned}
&2^{h-1}\le n < 2^h \\
&h-1 \le log_{2}{n} < h \\
&h = floor(log_{2}{n}) + 1
\end{aligned}
$$

#### 结点情况

##### 从 1 开始标号

一棵有 n 个节点的完全二叉树（n > 0），从上到下、从左到右对节点从 1 开始进行编号，对任意第 i 个节点

1. 如果 i = 1 ，它是根节点

2. 如果 i > 1 ，它的父节点编号为 floor( i / 2)
3. 如果 2i ≤ n ，它的左子节点编号为 2i
4. 如果 2i > n ，它无左子节点
5. 如果 2i + 1 ≤ n ，它的右子节点编号为 2i + 1
6. 如果 2i + 1 > n ，它无右子节点

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627895955704-1627895955698-tree_01.png)

##### 从 0 开始标号

一棵有 n 个节点的完全二叉树（n > 0），从上到下、从左到右对节点从 0 开始进行编号，对任意第 i 个节点

1. 如果 i = 0 ，它是根节点
2. 如果 i > 0 ，它的父节点编号为 floor( (i – 1) / 2 )
3. 如果 2i + 1 ≤ n – 1 ，它的左子节点编号为 2i + 1
4. 如果 2i + 1 > n – 1 ，它无左子节点
5. 如果 2i + 2 ≤ n – 1 ，它的右子节点编号为 2i + 2
6. 如果 2i + 2 > n – 1 ，它无右子节点

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627896124778-1627896124773-tree_02.png)

## 二叉树结点类型判断

### 叶子结点

```java
public boolean isLeaf() {
    return left == null && right == null;
}
```

### 有两个孩子结点

```java
public boolean hasTwoChildren() {
    return left != null && right != null;
}
```

### 左孩子结点

```java
public boolean isLeftChild() {
    return parent != null && parent.left == this;
}
```

### 右孩子结点

```java
public boolean isRightChild() {
	return parent != null && parent.right == this;
}
```

### 兄弟结点

```java
public Node<E> sibling() {
    if (isLeftChild()) {
        return parent.right;
    }
    if (isRightChild()) {
        return parent.left;
    }
    return null;
}
```

### 前驱结点

:::info

中序遍历时的前一个结点

:::

1. `node.left != null` 代表前驱结点在左子树中最右边的

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627910049292-1627910049291-tree_06.png)

1. `node.left == null && node.parent != null` 
   - 到父结点中找直到发现结点是父结点右子树为止

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627910007276-1627910007275-tree_07.png)

```java
private Node<E> precursor(Node<E> node) {
    if (node == null) {
        return null;
    }
    Node<E> p = node.left;
    // 前驱结点在左子树中(left, right, right, right....)
    if (node.left != null) {
        while (p.right != null) {
            p = p.right;
        }
        return p;
    }
    // 父结点、祖父结点中寻找前驱结点
    while (node.parent != null && node == node.parent.left) {
        node = node.parent;
    }
    // node.parent == null
    // node == node.parent.right
    return node.parent;
}
```

### 后继结点

:::info

中序遍历时的后一个结点

:::

```java
private Node<E> suffix(Node<E> node) {
    if (node == null) {
        return null;
    }
    Node<E> p = node.right;
    if (p != null) {
        while (p.left != null) {
            p = p.left;
        }
        return p;
    }
    while (node.parent != null && node == node.parent.right) {
        node = node.parent;
    }
    return node.parent;
}
```

