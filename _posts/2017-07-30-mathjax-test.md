---
layout: post
title: 'HashMap解读'
date: 2017-07-30
categories: test
tags: HashMap 
---


**前言**：
  JDK 1.8 对 HashMap 进行了比较大的优化，底层实现由之前的 “数组+链表” 改为 “数组+链表+红黑树”，本文就 HashMap 的几个常用的重要方法和 JDK 1.8 之前的死循环问题展开学习讨论。

  JDK 1.8 的 HashMap 的数据结构如下图所示，当链表节点较少时仍然是以链表存在，当链表节点较多时（大于8）会转为红黑树。
**几个点**：
先了解以下几个点，有利于更好的理解 HashMap 的源码和阅读本文。

1、本文中头节点指的是 table 表上索引位置的节点，也就是链表的头节点。

2、根节点（root 节点）指的是红黑树最上面的那个节点，也就是没有父节点的节点。

3、红黑树的根节点不一定是索引位置的头节点（也就是链表的头节点），HashMap 通过 moveRootToFront 方法来维持红黑树的根结点就是索引位置的头结点，但是在 removeTreeNode 方法中，当 movable 为 false 时，不会调用 moveRootToFront 方法，此时红黑树的根节点不一定是索引位置的头节点，该场景发生在 HashIterator 的 remove 方法中。

4、转为红黑树节点后，链表的结构还存在，通过 next 属性维持，红黑树节点在进行操作时都会维护链表的结构，并不是转为红黑树节点，链表结构就不存在了。

5、在红黑树上，叶子节点也可能有 next 节点，因为红黑树的结构跟链表的结构是互不影响的，不会因为是叶子节点就说该节点已经没有 next 节点。

6、源码中一些变量定义：如果定义了一个节点 p，则 pl（p left）为 p 的左节点，pr（p right）为 p 的右节点，pp（p parent）为 p 的父节点，ph（p hash）为 p 的 hash 值，pk（p key）为 p 的 key 值，kc（key class）为 key 的类等等。源码中很喜欢在 if/for 等语句中进行赋值并判断，请注意。
7、源码中进行红黑树的查找时，会反复用到以下两条规则：1）如果目标节点的 hash 值小于 p 节点的 hash 值，则向 p 节点的左边遍历；否则向 p 节点的右边遍历。2）如果目标节点的 key 值小于 p 节点的 key 值，则向 p 节点的左边遍历；否则向 p 节点的右边遍历。这两条规则是利用了红黑树的特性（左节点 < 根节点 < 右节点）。

8、源码中进行红黑树的查找时，会用 dir（direction）来表示向左还是向右查找，dir 存储的值是目标节点的 hash/key 与 p 节点的 hash/key 的比较结果。



**基本属性**：
// 默认容量16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
 
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;    
 
// 默认负载因子0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f; 
 
// 链表节点转换红黑树节点的阈值, 9个节点转
static final int TREEIFY_THRESHOLD = 8; 
 
// 红黑树节点转换链表节点的阈值, 6个节点转
static final int UNTREEIFY_THRESHOLD = 6;   
 
// 转红黑树时, table的最小长度
static final int MIN_TREEIFY_CAPACITY = 64; 
 
// 链表节点, 继承自Entry
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
 
    // ... ...
}
 
// 红黑树节点
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
   
    // ...
}



**定位哈希桶数组索引位置**：
不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。前面说过 HashMap 的数据结构是“数组+链表+红黑树”的结合，所以我们当然希望这个 HashMap 里面的元素位置尽量分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用 hash 算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，不用遍历链表/红黑树，大大优化了查询的效率。HashMap 定位数组索引位置，直接决定了 hash 方法的离散性能。下面是定位哈希桶数组的源码：

// 代码1
static final int hash(Object key) { // 计算key的hash值
    int h;
    // 1.先拿到key的hashCode值; 2.将hashCode的高16位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// 代码2
int n = tab.length;
// 将(tab.length - 1) 与 hash值进行&运算
int index = (n - 1) & hash;


整个过程本质上就是三步：

拿到 key 的 hashCode 值
将 hashCode 的高位参与运算，重新计算 hash 值
将计算出来的 hash 值与 (table.length - 1) 进行 & 运算


**方法解读**：
对于任意给定的对象，只要它的 hashCode() 返回值相同，那么计算得到的 hash 值总是相同的。我们首先想到的就是把 hash 值对 table 长度取模运算，这样一来，元素的分布相对来说是比较均匀的。

但是模运算消耗还是比较大的，我们知道计算机比较快的运算为位运算，因此 JDK 团队对取模运算进行了优化，使用上面代码2的位与运算来代替模运算。这个方法非常巧妙，它通过 “(table.length -1) & h” 来得到该对象的索引位置，这个优化是基于以下公式：x mod 2^n = x & (2^n - 1)。我们知道 HashMap 底层数组的长度总是 2 的 n 次方，并且取模运算为 “h mod table.length”，对应上面的公式，可以得到该运算等同于“h & (table.length - 1)”。这是 HashMap 在速度上的优化，因为 & 比 % 具有更高的效率。

在 JDK1.8 的实现中，还优化了高位运算的算法，将 hashCode 的高 16 位与 hashCode 进行异或运算，主要是为了在 table 的 length 较小的时候，让高位也参与运算，并且不会有太大的开销。

下图是一个简单的例子：

当 table 长度为 16 时，table.length - 1 = 15 ，用二进制来看，此时低 4 位全是 1，高 28 位全是 0，与 0 进行 & 运算必然为 0，因此此时 hashCode 与 “table.length - 1” 的 & 运算结果只取决于 hashCode 的低 4 位，在这种情况下，hashCode 的高 28 位就没有任何作用，并且由于 hash 结果只取决于 hashCode 的低 4 位，hash 冲突的概率也会增加。因此，在 JDK 1.8 中，将高位也参与计算，目的是为了降低 hash 冲突的概率。
