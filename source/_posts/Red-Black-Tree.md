---
title: 红黑树详解
date: 2017-06-13 15:12:22
categories: Algorithms
tags: 
- Algorithms
-  R-B Tree
---
#### 红黑树定义
红黑树一种特殊的二叉查找树，在多种语言的多种数据结构中都有实现，查询效率较高。其特性是：
- 每个节点或者黑色，或者是红色。
- 根节点是黑色。
- 每个叶子结点（NIL）是黑色。
- 如果一个结点是红色，它的子节点必须是黑色。
- 从一个结点到该节点的子孙叶子结点的所有路径上包含相同数目的黑色结点。（确保没有一条路径会比其他路径长出两倍，接近于平衡二叉树。）

![Alt text](/images/Red-Black Tree/1490578618842.png)

#### 红黑树的时间复杂度
**定理：一棵含有n个结点的红黑树的高度之多为$2log(n+1)$**
可以通过证明：高度为h的红黑树，他的包含的内节点的个数至少为$2^{h/2}-1$个。

#### 红黑树的基本操作
**左旋**
对X进行左旋，意味着，将“X的右孩子”设为“X的父亲节点”；即，将 X变成了一个左节点。 因此，左旋中的“左”，意味着“被旋转的节点将变成一个左节点”。
![Alt text](/images/Red-Black Tree/1491482439906.png)

《算法导论》伪代码：
```java
LEFT-ROTATE(T, x)  
 y ← right[x]            // 前提：这里假设x的右孩子为y。下面开始正式操作
 right[x] ← left[y]      // 将 “y的左孩子” 设为 “x的右孩子”，即 将β设为x的右孩子
 p[left[y]] ← x          // 将 “x” 设为 “y的左孩子的父亲”，即 将β的父亲设为x
 p[y] ← p[x]             // 将 “x的父亲” 设为 “y的父亲”
 if p[x] = nil[T]       
	 root[T] ← y         // 情况1：如果 “x的父亲” 是空节点，则将y设为根节点
 else 
	 if x = left[p[x]]  
		// 情况2：如果 x是它父节点的左孩子，则将y设为“x的父节点的左孩子”
        left[p[x]] ← y 
     else 
	    // 情况3：(x是它父节点的右孩子) 将y设为“x的父节点的右孩子”
	    right[p[x]] ← y   
 left[y] ← x             // 将 “x” 设为 “y的左孩子”
 p[x] ← y                // 将 “x的父节点” 设为 “y”
```

**右旋**
对X进行右旋，意味着，将“X的左孩子”设为“X的父亲节点”；即，将X变成了一个右节点。因此，右旋中的“右”，意味着“被旋转的节点将变成一个右节点”。
![Alt text](/images/Red-Black Tree/1491485291292.png)
《算法导论》伪代码：
````java
RIGHT-ROTATE(T, y)  
 x ← left[y]             // 前提：这里假设y的左孩子为x。下面开始正式操作
 left[y] ← right[x]      // 将 “x的右孩子” 设为 “y的左孩子”，即 将β设为y的左孩子
 p[right[x]] ← y         // 将 “y” 设为 “x的右孩子的父亲”，即 将β的父亲设为y
 p[x] ← p[y]             // 将 “y的父亲” 设为 “x的父亲”
 if p[y] = nil[T]       
   root[T] ← x                 // 情况1：如果 “y的父亲” 是空节点，则将x设为根节点
 else 
	 if y = right[p[y]]  
        right[p[y]] ← x   // 情况2：如果 y是它父节点的右孩子，则将x设为“y的父节点的左孩子”
     else 
	     left[p[y]] ← x    // 情况3：(y是它父节点的左孩子) 将x设为“y的父节点的左孩子”
 right[x] ← y            // 将 “y” 设为 “x的右孩子”
 p[y] ← x                // 将 “y的父节点” 设为 “x”
```


#### 红黑树的添加
**执行步骤：**
1. 将红黑树作为二叉查找树，将结点插入。
2. 将插入结点的颜色置为"红色"。
3. 通过旋转或者着色等操作，将二叉树重新变为一个红黑树。
	- 如果被插入节点是根节点，则直接将该结点涂为黑色。
	- 如果被插入节点的父节点是黑色，则不需要处理，认为红黑树。
	- 如果被插入节点的父节点是红色，则需要根据叔叔结点的情况进行进一步处理。

**对红色结点的调整**

| Case|     现象说明	|   处理策略|
| -------- | --------| :------: |
| Case 1   |当前节点的父节点是红色，叔叔节点也是红色。  | (01) 将“父节点”设为黑色。(02) 将“叔叔节点”设为黑色。(03) 将“祖父节点”设为“红色”。(04) 将“祖父节点”设为“当前节点”(红色节点)|
| Case 2|当前节点的父节点是红色，叔叔节点是黑色，且当前节点是其父节点的右孩子 | 	(01) 将“父节点”作为“新的当前节点”。(02) 以“新的当前节点”为支点进行左旋。|
| Case 3|当前节点的父节点是红色，叔叔节点是黑色，且当前节点是其父节点的左孩子|(01) 将“父节点”设为“黑色”。(02) 将“祖父节点”设为“红色”。(03) 以“祖父节点”为支点进行右旋。|
![Alt text](/images/Red-Black Tree/1491531856578.png)

```java
//红黑树调整函数fixAfterInsertion()
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;
    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {//如果y为null，则视为BLACK
                setColor(parentOf(x), BLACK);              // 情况1
                setColor(y, BLACK);                        // 情况1
                setColor(parentOf(parentOf(x)), RED);      // 情况1
                x = parentOf(parentOf(x));                 // 情况1
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);                       // 情况2
                    rotateLeft(x);                         // 情况2
                }
                setColor(parentOf(x), BLACK);              // 情况3
                setColor(parentOf(parentOf(x)), RED);      // 情况3
                rotateRight(parentOf(parentOf(x)));        // 情况3
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);              // 情况4
                setColor(y, BLACK);                        // 情况4
                setColor(parentOf(parentOf(x)), RED);      // 情况4
                x = parentOf(parentOf(x));                 // 情况4
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);                       // 情况5
                    rotateRight(x);                        // 情况5
                }
                setColor(parentOf(x), BLACK);              // 情况6
                setColor(parentOf(parentOf(x)), RED);      // 情况6
                rotateLeft(parentOf(parentOf(x)));         // 情况6
            }
        }
    }
    root.color = BLACK;
}
```

#### 红黑树的删除操作
**执行步骤：**
1. 将红黑树作为二叉查找树，将节点删除。
	- 如果是叶节点，直接将叶节点删除。
	- 如果删除结点只有一个儿子，将其子节点顶替其位置。
	- 如果有两个非空子节点，则需要找出其后继节点。然后把后继节点替换到当前位置，并递归地删除后继节点。

2. 通过“旋转和着色”来修正该树，使之重新成为一颗红黑树。

**基本思想：**
删除操作的总体思想是从兄弟节点借调黑色节点使树保持局部的平衡，如果局部的平衡达到了，就看整体的树是否是平衡的，如果不平衡就接着向上追溯调整。

删除修复操作分为四种情况(删除黑节点后)：
- 待删除的节点的兄弟节点是红色的节点。
- 待删除的节点的兄弟节点是黑色的节点，且兄弟节点的子节点都是黑色的。
- 待调整的节点的兄弟节点是黑色的节点，且兄弟节点的左子节点是红色的，右节点是黑色的(兄弟节点在右边)，如果兄弟节点在左边的话，就是兄弟节点的右子节点是红色的，左节点是黑色的。
- 待调整的节点的兄弟节点是黑色的节点，且右子节点是是红色的(兄弟节点在右边)，如果兄弟节点在左边，则就是对应的就是左节点是红色的。

**对黑色结点的调整**

| Case | 现象说明|   处理方案|
| -------- | --------| ------ |
| Case 1|  X是黑色结点，X的兄弟是红色结点，并且X的兄弟的子节点都是黑色节点|(01) 将x的兄弟节点设为“黑色”。(02) 将x的父节点设为“红色”。(03) 对x的父节点进行左旋。（无法从兄弟结点借调黑色结点，将兄弟结点上升，从兄弟结点的子节点借调，将其转换为Case 2,3,4）|
| Case 2| X是黑色结点，X的兄弟是黑色结点，并且X的兄弟的子节点都是黑色节点。|	(01) 将x的兄弟节点设为“红色”。(02) 设当前结点为X的父节点。（把兄弟结点变红后，有可能导致祖父结点失去平衡，因此需要回溯到父节点进行调整。）|
| Case 3| X是黑色结点，X的兄弟是黑色结点，兄弟结点的左孩子是红色结点。|(01) 将x兄弟节点的左孩子设为“黑色”。(02) 将x兄弟节点设为“红色”。(03) 对x的兄弟节点进行右旋。（中间状态，借用侄子结点的红色，变成黑色来平衡查找树。）|
| Case 4| X是黑色结点，X的兄弟是黑色结点，兄弟结点的右孩子是红色结点。|	(01) 将x父节点颜色 赋值给 x的兄弟节点。(02) 将x父节点设为“黑色”。(03) 将x兄弟节点的右子节设为“黑色”。(04) 对x的父节点进行左旋。(05) 设置“x”为“根节点”。（将兄弟结点的左黑色结点借调过来。）|
![Alt text](/images/Red-Black Tree/1491572932688.png)

```java
private void fixAfterDeletion(Entry<K,V> x) {
    while (x != root && colorOf(x) == BLACK) {
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);                   // 情况1
                setColor(parentOf(x), RED);             // 情况1
                rotateLeft(parentOf(x));                // 情况1
                sib = rightOf(parentOf(x));             // 情况1
            }
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);                     // 情况2
                x = parentOf(x);                        // 情况2
            } else {
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);       // 情况3
                    setColor(sib, RED);                 // 情况3
                    rotateRight(sib);                   // 情况3
                    sib = rightOf(parentOf(x));         // 情况3
                }
                setColor(sib, colorOf(parentOf(x)));    // 情况4
                setColor(parentOf(x), BLACK);           // 情况4
                setColor(rightOf(sib), BLACK);          // 情况4
                rotateLeft(parentOf(x));                // 情况4
                x = root;                               // 情况4
            }
        } else { // 跟前四种情况对称
            Entry<K,V> sib = leftOf(parentOf(x));
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);                   // 情况5
                setColor(parentOf(x), RED);             // 情况5
                rotateRight(parentOf(x));               // 情况5
                sib = leftOf(parentOf(x));              // 情况5
            }
            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);                     // 情况6
                x = parentOf(x);                        // 情况6
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);      // 情况7
                    setColor(sib, RED);                 // 情况7
                    rotateLeft(sib);                    // 情况7
                    sib = leftOf(parentOf(x));          // 情况7
                }
                setColor(sib, colorOf(parentOf(x)));    // 情况8
                setColor(parentOf(x), BLACK);           // 情况8
                setColor(leftOf(sib), BLACK);           // 情况8
                rotateRight(parentOf(x));               // 情况8
                x = root;                               // 情况8
            }
        }
    }
    setColor(x, BLACK);
}
```

**删除操作总结：**
红黑树的删除操作是最复杂的操作，复杂的地方就在于当删除了黑色节点的时候，如何从兄弟节点去借调节点，以保证树的颜色符合定义。由于红色的兄弟节点是没法借调出黑节点的，这样只能通过选择操作让他上升到父节点，而由于它是红节点，所以它的子节点就是黑的，可以借调。

对于兄弟节点是黑色节点的可以分成3种情况来处理，当所以的兄弟节点的子节点都是黑色节点时，可以直接将兄弟节点变红，这样局部的红黑树颜色是符合定义的。但是整颗树不一定是符合红黑树定义的，需要往上追溯继续调整。

对于兄弟节点的子节点为左红右黑或者 (全部为红，右红左黑)这两种情况，可以先将前面的情况通过选择转换为后一种情况，在后一种情况下，因为兄弟节点为黑，兄弟节点的右节点为红，可以借调出两个节点出来做黑节点，这样就可以保证删除了黑节点，整棵树还是符合红黑树的定义的，因为黑色节点的个数没有改变。

红黑树的删除操作是遇到删除的节点为红色，或者追溯调整到了root节点，这时删除的修复操作完毕。

#### 红黑树Java实现
```java
package edu.tsinghua.zhangfei.datastructure;

/**
 * Created by Fei Zhang on 2017/4/7.
 * Email:zhangfei614@126.com
 * - 每个节点或者黑色，或者是红色。
 * - 根节点是黑色。
 * - 每个叶子结点（NIL）是黑色。
 * - 如果一个结点是红色，它的子节点必须是黑色。
 * - 从一个结点到该节点的子孙叶子结点的所有路径上包含相同数目的黑色结点。（确保没有一条路径会比其他路径长出两倍，接近于平衡二叉树。）
 */


public class RBTree<T extends Comparable<T>> {
    static final boolean RED = true;
    static final boolean BLACK = false;


    private RBNode<T> root;

    static final class RBNode<T extends Comparable<T>> {
        T value;
        RBNode<T> left;
        RBNode<T> right;
        RBNode<T> parent;
        boolean color;

        RBNode(T value) {
            this.value = value;
        }

        RBNode(T value, boolean color, RBNode<T> parent, RBNode<T> left, RBNode<T> right) {
            this.value = value;
            this.color = color;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

        boolean isRed() {
            return color;
        }
    }

    public RBTree() {
        this.root = null;
    }

    /**
     * 对X进行左旋，意味着，将“X的右孩子”设为“X的父亲节点”；
     * 即，将 X变成了一个左节点。 因此，左旋中的“左”，意味着“被旋转的节点将变成一个左节点”。
     *
     * @param node
     */
    private void leftRotate(RBNode<T> node) {
        if (node != null) {
            RBNode r = node.right;
            node.right = r.left;
            if (r.left != null)
                r.left.parent = node;
            r.parent = node.parent;
            if (node.parent == null)
                this.root = r;
            else if (node == node.parent.left)
                node.parent.left = r;
            else
                node.parent.right = r;
            r.left = node;
            node.parent = r;
        }
    }

    /**
     * 对X进行右旋，意味着，将“X的左孩子”设为“X的父亲节点”；
     * 即，将X变成了一个右节点。因此，右旋中的“右”，意味着“被旋转的节点将变成一个右节点”。
     *
     * @param node
     */
    private void rightRotate(RBNode<T> node) {
        if (node != null) {
            RBNode l = node.left;
            node.left = l.right;
            if (l.right != null)
                l.right.parent = node;
            l.parent = node.parent;
            if (node.parent == null)
                this.root = l;
            else if (node.parent.left == node)
                node.parent.left = l;
            else
                node.parent.right = l;
            l.right = node;
            node.parent = l;
        }
    }


    private RBNode find(T value){
        int cmp;
        RBNode t = this.root;
        if (t == null) return null;
        do {
            cmp = value.compareTo((T) t.value);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t;
        } while (t != null);
        return null;
    }

    /**
     * 插入一个值：
     * 1. 将红黑树作为二叉查找树，将结点插入。
     * 2. 将插入结点的颜色置为"红色"。
     * 3. 通过旋转或者着色等操作，将二叉树重新变为一个红黑树。
     * 如果有值，则返回引用；没有值则返回null
     *
     * @param value
     * @return
     */
    public RBNode insert(T value) {
        int cmp;
        RBNode parent, t = this.root;
        //如果根为空，则直接插入。
        if (t == null) {
            RBNode node = new RBNode(value);
            this.root = node;
            return null;
        }
        do {
            parent = t;
            cmp = value.compareTo((T) t.value);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t;
        } while (t != null);
        RBNode node = new RBNode(value, RED, parent, null, null);
        if (cmp < 0)
            parent.left = node;
        else
            parent.right = node;
        fixAfterInsertion(node);
        return null;
    }

    /**
     * 插入值后的调整：
     * - 如果被插入节点是根节点，则直接将该结点涂为黑色。
     * - 如果被插入节点的父节点是黑色，则不需要处理，认为红黑树。
     * - 如果被插入节点的父节点是红色，则需要根据叔叔结点的情况进行进一步处理。
     * | Case 1|当前节点的父节点是红色，叔叔节点也是红色。| (01) 将“父节点”设为黑色。(02) 将“叔叔节点”设为黑色。(03) 将“祖父节点”设为“红色”。(04) 将“祖父节点”设为“当前节点”(红色节点)|
     * | Case 2|当前节点的父节点是红色，叔叔节点是黑色，且当前节点是其父节点的右孩子 | 	(01) 将“父节点”作为“新的当前节点”。(02) 以“新的当前节点”为支点进行左旋。|
     * | Case 3|当前节点的父节点是红色，叔叔节点是黑色，且当前节点是其父节点的左孩子|(01) 将“父节点”设为“黑色”。(02) 将“祖父节点”设为“红色”。(03) 以“祖父节点”为支点进行右旋。|
     *
     * @param node
     */
    private void fixAfterInsertion(RBNode node) {
        node.color = RED;
        while (node != null && node != root && colorOf(parentOf(node))== RED) {
            if (parentOf(node) == leftOf(parentOf(parentOf(node)))) {
                RBNode uncle = rightOf(parentOf(parentOf(node)));
                //case 1
                if (colorOf(uncle) == RED) {
                    setColor(parentOf(node),BLACK);
                    setColor(uncle,BLACK);
                    setColor(parentOf(parentOf(node)),RED);
                    node = parentOf(parentOf(node));
                } else {
                    //case 2
                    if (node == rightOf(parentOf(node))) {
                        node = parentOf(node);
                        leftRotate(node);
                    }
                    //case 3
                    setColor(parentOf(node),BLACK);
                    setColor(parentOf(parentOf(node)),RED);
                    rightRotate(parentOf(parentOf(node)));
                }
            } else {
                RBNode uncle = leftOf(parentOf(parentOf(node)));
                if (colorOf(uncle) == RED) {
                    setColor(parentOf(node),BLACK);
                    setColor(uncle,BLACK);
                    setColor(parentOf(parentOf(node)),RED);
                    node = parentOf(parentOf(node));
                } else {
                    if (node == leftOf(parentOf(node))) {
                        node = parentOf(node);
                        rightRotate(node);
                    }
                    setColor(parentOf(node),BLACK);
                    setColor(parentOf(parentOf(node)),RED);
                    leftRotate(parentOf(parentOf(node)));
                }
            }
        }
        root.color = BLACK;
    }

    /**
     * 删除一个值：
     * 1. 将红黑树作为二叉查找树，将节点删除。
     * - 如果是叶节点，直接将叶节点删除。
     * - 如果删除结点只有一个儿子，将其子节点顶替其位置。
     * - 如果有两个非空子节点，则需要找出其后继节点。然后把后继节点替换到当前位置，并递归地删除后继节点。
     * 2. 通过“旋转和着色”来修正该树，使之重新成为一颗红黑树。
     *
     * @param value
     * @return
     */
    public void delete(T value) {
        RBNode t = find(value);
        if (t == null) return;
        //如果是双非空子节点，找到后继，并替换
        if (t.left != null && t.right != null) {
            RBNode successor = successor(t);
            t.value = successor.value;
            t = successor;
        }
        RBNode replacement = t.left != null ? t.left : t.right;
        //如果是单非空子节点，则直接删除替换
        if (replacement != null) {
            replacement.parent = t.parent;
            if (t.parent == null) {
                root = replacement;
            } else if (t == t.parent.left) {
                t.parent.left = replacement;
            } else {
                t.parent.right = replacement;
            }
            t.parent = t.left = t.right = null;
            //如果是双空结点
        } else if (t.parent == null) {
            this.root = null;
        } else {
            if (t.color == BLACK)
                fixAfterDeletion(t);
            if (t.parent != null) {
                if (t == t.parent.left)
                    t.parent.left = null;
                else if (t == t.parent.right)
                    t.parent.right = null;
                t.parent = null;
            }
        }

    }

    /**
     * 寻找后继结点
     * //在查找过程中，如果节点x右子树不为空，那么返回右子树的最小节点即可
     * //如果节点x的右子树为空，那么后继节点为x的某一个祖先节点的父节点，而且该祖先节点是作为其父节点的左儿子
     *
     * @param node
     * @return
     */
    private RBNode successor(RBNode node) {
        if (node == null) return null;
        if (node.right != null) {
            RBNode p = node.right;
            while (p.left != null) p = p.left;
            return p;
        } else {
            RBNode p = node.parent;
            while (p != null && node == p.right) {
                node = p;
                p = p.parent;
            }
            return p;
        }
    }

    /**
     * 基本思路：
     * 删除操作的总体思想是从兄弟节点借调黑色节点使树保持局部的平衡，如果局部的平衡达到了，就看整体的树是否是平衡的，如果不平衡就接着向上追溯调整。
     * | Case 1| X是黑色结点，X的兄弟是红色结点，并且X的兄弟的子节点都是黑色节点|(01) 将x的兄弟节点设为“黑色”。(02) 将x的父节点设为“红色”。(03) 对x的父节点进行左旋。（无法从兄弟结点借调黑色结点，将兄弟结点上升，从兄弟结点的子节点借调，将其转换为Case 2,3,4）|
     * | Case 2| X是黑色结点，X的兄弟是黑色结点，并且X的兄弟的子节点都是黑色节点。|	(01) 将x的兄弟节点设为“红色”。(02) 设当前结点为X的父节点。（把兄弟结点变红后，有可能导致祖父结点失去平衡，因此需要回溯到父节点进行调整。）|
     * | Case 3| X是黑色结点，X的兄弟是黑色结点，兄弟结点的左孩子是红色结点。|(01) 将x兄弟节点的左孩子设为“黑色”。(02) 将x兄弟节点设为“红色”。(03) 对x的兄弟节点进行右旋。（中间状态，借用侄子结点的红色，变成黑色来平衡查找树。）|
     * | Case 4| X是黑色结点，X的兄弟是黑色结点，兄弟结点的右孩子是红色结点。|	(01) 将x父节点颜色 赋值给 x的兄弟节点。(02) 将x父节点设为“黑色”。(03) 将x兄弟节点的右子节设为“黑色”。(04) 对x的父节点进行左旋。(05) 设置“x”为“根节点”。（将兄弟结点的左黑色结点借调过来。）|
     *
     * @param node
     */
    private void fixAfterDeletion(RBNode node) {
        while (node != root && colorOf(node) == BLACK) {
            if (node == leftOf(parentOf(node))) {
                RBNode sib = rightOf(parentOf(node));
                if(colorOf(sib) == RED){
                    setColor(sib,BLACK);
                    setColor(parentOf(node),RED);
                    leftRotate(parentOf(node));
                    sib = rightOf(parentOf(node));
                }
                if(colorOf(leftOf(sib)) == BLACK && colorOf(rightOf(sib)) == BLACK){
                    setColor(sib,RED);
                    node = parentOf(node);
                }else{
                    if(colorOf(rightOf(sib)) == BLACK){
                        setColor(leftOf(sib),BLACK);
                        setColor(sib,RED);
                        rightRotate(sib);
                        sib = rightOf(parentOf(node));
                    }
                    setColor(sib,colorOf(parentOf(node)));
                    setColor(parentOf(node),BLACK);
                    setColor(rightOf(sib),BLACK);
                    leftRotate(parentOf(node));
                    node = root;
                }
            } else {
                RBNode sib = leftOf(parentOf(node));
                if(colorOf(sib) == RED){
                    setColor(sib,BLACK);
                    setColor(parentOf(node),RED);
                    rightRotate(parentOf(node));
                    sib = leftOf(parentOf(node));
                }
                if(colorOf(leftOf(sib)) == BLACK && colorOf(rightOf(sib)) == BLACK){
                    setColor(sib,RED);
                    node = parentOf(node);
                }else{
                    if(colorOf(leftOf(sib)) == BLACK){
                        setColor(rightOf(sib),BLACK);
                        setColor(sib,RED);
                        leftRotate(sib);
                        sib = leftOf(parentOf(node));
                    }
                    setColor(sib,colorOf(parentOf(node)));
                    setColor(parentOf(node),BLACK);
                    setColor(leftOf(sib),BLACK);
                    rightRotate(parentOf(node));
                    node = root;
                }
            }
        }
    }


    public void preOrder(){
        preOrderHelper(this.root);
        System.out.println();
    }
    private void preOrderHelper(RBNode node){
        if(node == null) return;
        System.out.print("(" + node.value.toString() + "," + colorName(colorOf(node)) + ")");
        preOrderHelper(node.left);
        preOrderHelper(node.right);
    }

    public void midOrder(){
        midOrderHelper(this.root);
        System.out.println();
    }

    private void midOrderHelper(RBNode node){
        if(node == null) return;
        midOrderHelper(node.left);
        System.out.print("(" + node.value.toString() + "," + colorName(colorOf(node)) + ")");
        midOrderHelper(node.right);
    }


    private static boolean colorOf(RBNode p) {
        return (p == null ? BLACK : p.color);
    }

    private static RBNode parentOf(RBNode p) {
        return (p == null ? null : p.parent);
    }

    private static void setColor(RBNode p, boolean c) {
        if (p != null)
            p.color = c;
    }

    private static RBNode leftOf(RBNode p) {
        return (p == null) ? null : p.left;
    }

    private static RBNode rightOf(RBNode p) {
        return (p == null) ? null : p.right;
    }

    private static String colorName(boolean color){
        return color == RED ? "RED" : "BLACK";
    }


    public static void main(String[] args){
        Integer[] test = new Integer[]{1,2,3,4,5,6,7,8};
        RBTree<Integer> rbTree = new RBTree<Integer>();
        for(Integer integer: test) rbTree.insert(integer);
        rbTree.delete(4);
        rbTree.preOrder();
        rbTree.midOrder();
    }
}

```


***参考链接：***
*[红黑树深入剖析及Java实现](http://tech.meituan.com/redblack-tree.html)*
*[史上最清晰的红黑树讲解（上）](http://www.importnew.com/21818.html)*
*[史上最清晰的红黑树讲解（下）](http://www.importnew.com/21822.html)*
*[红黑树(一)之 原理和算法详细介绍](http://www.cnblogs.com/skywang12345/p/3245399.html)*
