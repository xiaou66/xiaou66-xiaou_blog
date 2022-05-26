---
title: AVL树
date: 2021/08/03 10:00
math: true
categories: 
- [数据结构, 树]
tags:
- [java]
---

# AVL树

:::info

AVL树是最早发明的自平衡二叉搜索树之一

:::

## 特点

平衡因子（Balance Factor）：某结点的左右子树的高度差

1. 每个节点的平衡因子只可能是 1、0、-1（绝对值 ≤ 1，如果超过 1，称之为“失衡”）
2. 每个节点的左右子树高度差不超过 1
3. 搜索、添加、删除的时间复杂度是$O(\log{n})$

### 对比

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627953711508-1627953711502-tree_10.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627956495720-1627956495718-tree_09.png)

## 元素添加操作
### 添加会导致失衡

1. 可能会导致所有祖先节点都失衡
2. 父节点、非祖先节点，都不可能失衡

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627960321158-1627960321157-tree_11.png)

### 主要情况

### 右旋转LL(单旋)

1. g.left = p.right
2. p.right = g
3. 让 p 成为这颗子树的根结点
4. 整课树达到平衡

:::warning 

1. T2、p、g 的 parent 值

2. 要先更新 g、再更新 p 的高度

   因为旋转之后 p 是 g 的父结点

:::



![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627960348482-1627960348481-tree_12.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627960306878-1627960306877-tree_13.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627960453571-1627960453570-tree_14.png)

```java 右旋转实现
protected void rotateRight(Node<E> grand) {
    Node<E> parent = grand.left;
    Node<E> child = parent.right;
    grand.left = child;
    parent.left = grand;
    // 让 parent 为子树的根结点
    parent.parent = grand.parent;
    if (grand.isLeftChild()) {
        grand.parent.left = parent;
    }else if (grand.isRightChild()) {
        grand.parent.right = parent;
    }else {
        // grand 是根结点
        root = parent;
    }
    // 更新 child 的 parent
    if (child != null) {
        child.parent = grand;
    }
    // 更新 grand parent
    grand.parent = parent;
    // 更新高度
    updateHeight(grand);
    updateHeight(grand.parent);
}
```

### 左旋转RR(单旋)

1. p.right = p.left
2. p.left = g
3. 让 p 成为这颗子树的根结点
4. 整课树达到平衡

:::warning 

1. T1、p、g 的 parent 值

2. 要先更新 g、再更新 p 的高度

   因为旋转之后 p 是 g 的父结点

:::

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627964462985-1627964462983-tree_15.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627962779509-1627962779508-tree_16.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627964512661-1627964512660-tree_17.png)

```java 左旋转实现
protected void rotateLeft(Node<E> grand) {
    Node<E> parent = grand.right;
    Node<E> child = parent.left;

    grand.right = parent.left;
    parent.left = grand;
    // 让 parent 为子树的根结点
    parent.parent = grand.parent;
    if (grand.isLeftChild()) {
        grand.parent.left = parent;
    }else if (grand.isRightChild()) {
        grand.parent.right = parent;
    }else {
        // grand 是根结点
        root = parent;
    }
    // 更新 child 的 parent
    if (child != null) {
        child.parent = grand;
    }
    // 更新 grand parent
    grand.parent = parent;
    // 更新高度
    updateHeight(grand);
    updateHeight(grand.parent);
}
```

### 左右旋转LR(双旋)

1. 先对父结点进行左旋转
2. 再对祖父结点进行右旋转

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627965230989-1627965230988-tree_18.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627965316387-1627965316386-tree_19.png)



![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627965417413-1627965417412-tree_20.png)

### 右左旋转 RL(双旋)

1. 先对父结点进行右旋转
2. 再对祖父结点进行左旋转

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627965587654-1627965587653-tree_21.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627965620244-1627965620243-tree_23.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627965602394-1627965602392-tree_22.png)

### 代码

```java 核心代码
/**
* 在二叉搜索树插入之后
*/
protected void afterAdd(Node<E> node) {
    while ((node = node.parent) != null) {
        if (isBalanced(node)) {
            // 平衡
            // 更新高度
            updateHeight(node);
        } else {
            // 恢复平衡
            reBalance(node);
            break;
        }
    }
}
/**
 * 恢复平衡
 * @param grand 高度最低的不平衡结点
 */
private void reBalance(Node<E> grand) {
    Node<E> parent = ((AVLNode<E>)grand).tallerChild();
    Node<E> node = ((AVLNode<E>)parent).tallerChild();
    if (parent.isLeftChild()) {
        if (node.isLeftChild()) {
            // LL
            rotateRight(grand);
        }else {
            // LR
            rotateLeft(parent);
            rotateRight(grand);
        }
    } else {
        if (node.isRightChild()) {
            // RR
            rotateLeft(grand);
        }else {
            // RL
            rotateRight(parent);
            rotateLeft(grand);
        }
    }
}
```



## 元素删除操作

### 删除会导致失衡 

1. 删除只可能导致父结点或祖先结点失衡(只有 1 个结点也会失衡)，其他结点都不可能失衡

2. 如果结点被删除，更高层的祖先结点可能也会失衡，需要再一次恢复平衡, 然后又可能导致更高层的祖先结点出现失衡
3. 在极端的情况下，所有的祖先结点都需要进行恢复平衡的操作，共 $O(log{n})$ 次调整

### 主要情况

:::info

主要删除结点的调整和增加结点的调整基本一致，这里只提供不一样的情况

:::

#### LL 右旋转(单旋)

1. 旋转操作与增加一致
2. 绿色结点如果存在则子树高度不变不会失衡,如果绿色结点不存在则会造成子树的高度改变可能会使子树的父结点或者祖先结点失衡

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627974448681-1627974448678-tree_24.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627974496184-1627974496182-tree_26.png)

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1627974473734-1627974473733-tree_25.png)



### 代码

```java 核心代码
protected void afterRemove(Node<E> node, Node<E> replacement) {
    while ((node = node.parent) != null) {
        if (isBalanced(node)) {
            // 平衡
            // 更新高度
            updateHeight(node);
        } else {
            // 恢复平衡
            reBalance(node);
        }
    }
}
/**
 * 恢复平衡
 * @param grand 高度最低的不平衡结点
 */
private void reBalance(Node<E> grand) {
    Node<E> parent = ((AVLNode<E>)grand).tallerChild();
    Node<E> node = ((AVLNode<E>)parent).tallerChild();
    if (parent.isLeftChild()) {
        if (node.isLeftChild()) {
            // LL
            rotateRight(grand);
        }else {
            // LR
            rotateLeft(parent);
            rotateRight(grand);
        }
    } else {
        if (node.isRightChild()) {
            // RR
            rotateLeft(grand);
        }else {
            // RL
            rotateRight(parent);
            rotateLeft(grand);
        }
    }
}
```

