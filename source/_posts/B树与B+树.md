title: B树与B+树
author: Jiahao Wu
tags: []
categories:
  - 数据结构
date: 2021-02-15 14:32:00
---
# B树

B树也称B-树,它是一颗多路平衡查找树。性质如下：  
1. 每个节点最多有m-1个关键字（可以存有的键值对）；  
2. 根节点最少可以只有1个关键字；  
3. 非根节点至少有m/2个关键字；  
4. 每个节点中的关键字都按照从小到大的顺序排列，每个关键字的左子树中的所有关键字都小于它，而右子树中的所有关键字都大于它；  
5. 所有叶子节点都位于同一层，或者说根节点到每个叶子节点的长度都相同；  
6. 每个节点都存有索引和数据，也就是对应的key和value；  
所以，根节点的关键字数量范围：1 <= k <= m-1，非根节点的关键字数量范围：m/2 <= k <= m-1。  

# B+树

B+树其实和B树是非常相似的，我们首先看看相同点：  
1. 根节点至少一个元素；  
2. 非根节点元素范围：m/2 <= k <= m-1。  

不同点：  
1. B+树有两种类型的节点：内部结点（也称索引结点）和叶子结点。内部节点就是非叶子节点，内部节点不存储数据，只存储索引，数据都存储在叶子节点；  
2. 内部结点中的key都按照从小到大的顺序排列，对于内部结点中的一个key，左树中的所有key都小于它，右子树中的key都大于等于它。叶子结点中的记录也按照key的大小排列。
3. 每个叶子结点都存有相邻叶子结点的指针，叶子结点本身依关键字的大小自小而大顺序链接。
4. 父节点存有右孩子的第一个元素的索引。

# B+树和B树相比的优点

**效率高**：B+树非叶子结点不存储数据，所以能够有更多的子结点，降低了树的高度，减少磁盘IO次数，效率更高。此外由于叶结点保存的数据多，对访问的局部性原理利用更好，缓存命中率高。  
**更适合于范围查找**：在B树中进行范围查找时，需要先找到下限制，在进行中序遍历，直到找到上限；而B+树只要找到下限后，通过链表向前遍历即可。  
**更稳定的查询效率**：B+树的每个叶子结点到根结点的距离是一样的，所以磁盘IO次数是稳定的，而B树查询每次磁盘IO次数是不确定的。