---
title: 红黑树
date: 2021/08/05 09:00
math: true
categories: 
- [数据结构, 树]
tags:
- [java]
---

# 红黑树

:::info

红黑树也是一种自平衡的二叉搜索树也叫做平衡二叉B树

:::

## 红黑树性质

红黑树必须满足以下 5 条性质

1. 节点是 [RED]{.red}或者 [BLACK]{.BLACK}
2. 根节点是 [BLACK]{.BLACK}
3. 叶子节点（外部节点，空节点）都是 [BLACK]{.BLACK}
4.  [RED]{.red} 节点的子节点都是 [BLACK]{.BLACK}
   -  [RED]{.red}节点的 parent 都是  [BLACK]{.BLACK}
   -  从根节点到叶子节点的所有路径上不能有 2 个连续的 [RED]{.red} 节点
5. 从任一节点到叶子节点的所有路径都包含相同数目的  [BLACK]{.BLACK}节点

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628125482796-tree_37.png)

## 红黑树和B树关系

1. 红黑树 和 4阶B树（2-3-4树）具有等价性
2. [BLACK]{.BLACK}节点与它的 [RED]{.red}子节点融合在一起，形成1个B树节点
3. 红黑树的 [BLACK]{.BLACK} 节点个数 与 4阶B树的节点总个数 相等

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628129373294-tree_38.png)

### 红黑树与B树转换

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628127028761-tree_39.png)

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628126961139-tree_41.png)

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628127013478-tree_40.png)

## 元素的添加

:::info

B树中，新元素必定是添加到叶子节点中

4阶B树所有节点的元素个数 x 都符合  $1 \le x \le 3$

推荐新添加的结点颜色默认为 [RED]{.red}, 这样可以让红黑树的性质尽快满足

如果添加的是根结点，直接染成 [BLACK]{.BLACK} 即可

:::

### 父节点是黑色

1. 直接满足 4 阶 B 树的性质, 不用做任何处理
2. 加入结点均为 [BLACK]{.BLACK} 结点

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628129443654-tree_42.png)

### 叔父节点不是 RED

#### LL / RR

1. parent 染成  [BLACK]{.BLACK}，grand 染成 [RED]{.red}
2.  grand 进行单旋操作

- LL：右旋转
- RR：左旋转

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628129589222-tree_43.png)

#### LR/RL

1. 自己染成   [BLACK]{.BLACK}，grand 染成 [RED]{.red}
2. 进行双旋操作

- LR：parent 左旋转， grand 右旋转
- RL：parent 右旋转， grand 左旋转

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628130062900-tree_44.png)

### 叔父节点是 RED

#### LL

1. parent、uncle 染成 [BLACK]{.BLACK}
2. grand 向上合并

- 染成  [RED]{.red}，当做是新添加的节点进行处理
- grand 向上合并时，可能继续发生上溢
- 若上溢持续到根节点，只需将根节点染成 BLACK

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628142089873-tree_45.png)

#### RR

1. parent、uncle 染成 BLACK
2. grand 向上合并
3. 染成  [RED]{.red}，当做是新添加的节点进行处理

:::info

与 「 LL 」 情况处理基本一致

:::

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628142568036-tree_46.png)

#### LR

1. parent、uncle 染成 BLACK
2. grand 向上合并
3. 染成  [RED]{.red}，当做是新添加的节点进行处理

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628144914495-tree_47.png)

#### RL

1. parent、uncle 染成 BLACK
2.  grand 向上合并
3. 染成  [RED]{.red}，当做是新添加的节点进行处理

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628145093421-tree_48.png)

### 代码

```java 核心代码
protected void afterAdd(Node<E> node) {
    Node<E> parent = node.parent;
    if (parent == null) {
        // 添加是根结点
        black(node);
        return;
    }
    if (isBlack(parent)) {
        // 如果父结点是黑色, 直接返回
        return;
    }
    // 叔父结点
    Node<E> uncle = parent.sibling();
    // 祖父结点
    Node<E> grand = parent.parent;
    if (isRed(uncle)) {
        // 叔父结点是红色[上溢]
        black(parent);
        black(uncle);
        // 把祖父结点当做新添加的结点
        afterAdd(red(grand));
        return;
    }
    // 祖父结点不是红色
    if (parent.isLeftChild()) {
        // L
        if (node.isLeftChild()) {
            // LL
            black(parent);
            red(grand);
            rotateRight(grand);
        }else {
            // LR
            black(node);
            red(grand);
            rotateLeft(parent);
            rotateRight(grand);
        }
    }else {
        // R
        if (node.isLeftChild()) {
            // RL
            black(node);
            red(grand);
            rotateRight(parent);
            rotateLeft(grand);
        }else {
            // RR
            black(parent);
            red(grand);
            rotateLeft(grand);
        }
    }

}
```



## 元素删除

:::info

B树中，最后真正被删除的元素都在叶子节点中

:::

### 删除红色结点

1. 直接删除，不用作任何调整

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628146143891-tree_49.png)

### 删除拥有1个Red子节点的Black结点

- 判断条件: 用以替代的子节点是  [RED]{.red}
- 将替代的子节点染成 BLACK 即可保持红黑树性质

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628147387276-tree_50.png)

### 删除Black叶子结点—兄弟为BLACK

- BLACK 叶子节点被删除后，会导致B树节点下溢

#### sibling 至少有 1 个  [RED]{.red}子节点

1. 进行旋转操作
2. 旋转之后的中心节点继承 **parent** 的颜色
3. 旋转之后的左右节点染为 BLACK

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628149854637-tree_51.png)

#### sibling 没有 1 个 RED 子节点

1. 将 sibling 染成 RED、parent 染成 BLACK

:::info

如果 parent 是 BLACK会导致 parent 也下溢这时只需要把 parent 当做被删除的节点处理即可

:::

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628150340598-tree_52.png)

### 删除Black叶子结点—sibling为[RED]{.red}

1. sibling 染成 BLACK，parent 染成 RED，进行旋转
2. 于是又回到 sibling 是 BLACK 的情况

![](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1628152453409-tree_53.png)

### 代码

```java 核心代码
protected void afterRemove(Node<E> node, Node<E> replacement) {
    if (isRed(node)) {
        // 删除的结点是红色
        return;
    }
    if (isRed(replacement)) {
        // 用于取代 node 结点是红色
        black(replacement);
        return;
    }
    Node<E> parent = node.parent;
    if (parent == null) {
        // 删除是根结点
        return;
    }
    // 删除黑色叶子结点 [下溢]
    boolean left = parent.left == null || node.isLeftChild();
    Node<E> sibling = left ? parent.right : parent.left;
    if (left) {
        // 被删除的结点在左边,兄弟结点在右边
        if (isRed(sibling)) {
            // 兄弟结点是红色
            // 将兄弟结点转换为黑色
            black(sibling);
            red(parent);
            rotateLeft(parent);
            // 更换兄弟结点
            sibling = parent.right;
        }
        // 兄弟结点是黑色
        if (isBlack(sibling.left) && isBlack(sibling.right)) {
            // 兄弟结点没有一个红色子结点
            // 父结点向下合并
            boolean parentBlack = isBlack(parent);
            black(parent);
            red(sibling);
            if (parentBlack) {
                // 父结点是黑色
                afterRemove(parent, null);
            }
        }else  {
            // 兄弟结点至少有一个红色子结点
            // 向兄弟结点借元素
            if (isBlack(sibling.right)) {
                // 兄弟结点左边是黑色
                rotateRight(sibling);
                sibling = parent.right;
            }
            color(sibling, colorOf(parent));
            black(sibling.right);
            black(parent);
            rotateLeft(parent);
        }
        // -------------------------------
    }else {
        // 被删除的结点在右边,兄弟结点在左边
        if (isRed(sibling)) {
            // 兄弟结点是红色
            // 将兄弟结点转换为黑色
            black(sibling);
            red(parent);
            rotateRight(parent);
            // 更换兄弟结点
            sibling = parent.left;
        }
        // 兄弟结点是黑色
        if (isBlack(sibling.left) && isBlack(sibling.right)) {
            // 兄弟结点没有一个红色子结点
            // 父结点向下合并
            boolean parentBlack = isBlack(parent);
            black(parent);
            red(sibling);
            if (parentBlack) {
                // 父结点是黑色
                afterRemove(parent, null);
            }
        }else  {
            // 兄弟结点至少有一个红色子结点
            // 向兄弟结点借元素
            if (isBlack(sibling.left)) {
                // 兄弟结点左边是黑色
                rotateLeft(sibling);
                sibling = parent.left;
            }
            color(sibling, colorOf(parent));
            black(sibling.left);
            black(parent);
            rotateRight(parent);
        }
    }
}
```

## 一些辅助函数

```java
private static final boolean RED = false;
private static final boolean BLACK = true;
private Node<E> red(Node<E> node) {
    return color(node, RED);
}
private Node<E> black(Node<E> node) {
    return color(node, BLACK);
}
private boolean colorOf(Node<E> node) {
    return node == null ? BLACK : ((RBNode<E>)node).color;
}
private boolean isBlack(Node<E> node) {
    return colorOf(node) == BLACK;
}
private boolean isRed(Node<E> node) {
    return colorOf(node) == RED;
}
private Node<E> color(Node<E> node, boolean color) {
    if (node == null) {
        return null;
    }
    ((RBNode<E>)node).color = color;
    return node;
}
```

