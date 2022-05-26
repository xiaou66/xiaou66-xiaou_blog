---
title: 二叉搜索树
date: 2021/08/03 08:00
math: true
categories: 
- [数据结构, 树]
tags:
- [java]
---

# 二叉搜索树

:::info

二叉搜索树是二叉树的一种，是应用非常广泛的一种二叉树，英文简称为 BST，又称二叉查找树、二叉排序树

:::

## 特点

1. 任意一个节点的值都大于其左子树所有节点的值
2. 任意一个节点的值都小于其右子树所有节点的值
3. 它的左右子树也是一棵二叉搜索树

## 操作

### 添加结点

#### 思路

1. 先按照二叉搜索树性质找到父结点 parent
2. 创建新结点 node
3. 判断是插入在左边还是右边
   - parent.left = node 或者 parent.right = node

:::info

如果发现值一样时, 推荐进行将原来的值覆盖

:::

#### 图示

1. 在下列的二叉树搜索树总插入值为 `12` 的结点 

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627905868990-1627905868986-tree_03.png)

#### 代码

```java
public void add(E element) {
    elementNotNullCheck(element);
    //  首个结点
    if (this.root == null) {
        this.root = createNode(element, null);
        size++;
        afterAdd(root);
        return;
    }
    Node<E> node = root;
    int cmp = 0;
    Node<E> parent = root;
    while (node != null) {
        // 比较
        cmp = compare(element, node.element);
        parent = node;
        if (cmp > 0) {
            // 当前插入值大于当前结点值
            // 向右边探索
            node = node.right;
        }else if (cmp < 0) {
            // 当前插入值小于当前结点值
            // 向左边探索
            node = node.left;
        } else {
            // 结点数据一致
            node.element = element;
            return;
        }
    }
    // 创建加入结点
    Node<E> insertNode = createNode(element, parent);
    if (cmp > 0) {
        // 加入右边
        parent.right = insertNode;
    } else {
        // 插入左边
        parent.left = insertNode;
    }
    size++;
}
```

### 删除结点

#### 思路

1. 删除叶子结点——直接删除

   - 向取出结点 `node == node.parent.left`
   - 删除 `node.parent.left = null`

2. 删除度为 1 的结点——用子结点替代原结点的位置

   - 删除结点是左子结点

     `child.parent = node.parent;`
     `node.parent.left = child;`

   - 删除结点是右子结点

     `child.parent = node.parent`

     `node.parent.right = child`

   - 删除结点是根结点

     `root = child`

     `child.parent = null`

        3. 删除结点——度为 2 的结点
           - 向用前驱或者后继结点的值覆盖原结点的值
           - 然后删除相应的前驱或者后继结点

#### 图示

1. 删除叶子结点直接删除

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627907535316-1627907535313-tree_04.png)

1. 删除度为 1 的结点——用子结点替代原结点的位置

   ![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627907545669-1627907545665-tree_05.png)

2. 删除结点——度为 2 的结点

   - 绿色为前驱结点

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627910787604-1627910787602-tree_08.png)
#### 代码
```java
private void remove(Node<E> node) {
    if (node == null) {
        return;
    }
    size--;
    if (node.hasTwoChildren()) {
        // 度为 2 结点
        // 后继结点
        Node<E> s = suffix(node);
        node.element = s.element;
        node = s;
    }
    // 删除 node 结点(node 度必然是 1 和 0)
    Node<E> replacement = node.left != null ? node.left : node.right;
    if (replacement != null) {
        // node 度为 1 的结点
        // 更改 parent
        replacement.parent = node.parent;
        if (node.parent == null) {
            root = replacement;
        } else if (node == node.parent.left) {
            // 父结点左边
            node.parent.left = replacement;
        } else {
            // 父结点右边
            node.parent.right = replacement;
        }
        // 删除的结点之后的处理
        afterRemove(node, replacement);
    }else if (node.parent == null){
        // 根结点
        root = null;
        afterRemove(node, null);
    } else {
        // 叶子结点
        if (node.parent.left == node) {
            node.parent.left = null;
        }else {
            node.parent.right = null;
        }
        afterRemove(node, null);
    }
}
```

