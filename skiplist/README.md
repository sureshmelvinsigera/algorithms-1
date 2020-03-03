## 跳跃列表/跳跃链表 - Skiplist

### 基本原理

对于普通的列表，例如

`[1, 2, 3, 4, 5, 6, 7, 8, 9]`

搜索/插入/删除时最糟复杂度为 O(n)，即从头到尾遍历一遍。而二分查找法可以将速度提升到 O(logn) 的程度。但当数据在普通列表中插入或删除时，插入/删除点之后的元素索引全部需要更新，造成性能上的损耗。因此就有了链表这样的数据结构，保证在插入/删除时，仅仅更新前后两个节点的索引即可。

但对于链表而言，无法通过`index`进行双指针式的二分查找，搜索速度又降回了 O(n) 的程度。如何在有效利用链表插入/删除特性的同时，保证 O(logn) 的搜索速度呢？

跳跃列表的基本思想是，维护一个多层级的数据结构。最底部是最基本的完整的链表，例如`1 -> 2 -> 3 -> 4 -> 5 -> 6`，往上每一层，都是它底部一层的链表元素的节选，例如倒数第二层可能是`1 -> 3 -> 5`，倒数第三层可能是`1 -> 5`，第一层可能是`1`。每一层的每个节点，同时也指向了它在下一层的节点。即，每个更高层都充当下面链表的快速通道

除了最底层是完整的链表以外，每个节点是否出现在上一层，由概率`P`决定。通常`P`是`1/4`或`1/2`。因此，例如对于一个节点 3，它除了一定会在底层以外，出现在倒数第二层的概率是`P`，出现在倒数第三层的概率是`P * P`，依次类推。。最终构造的结果如下图所示

```
1
|
1 ----------------> 5
|                   |
1 ------> 3 ------> 5
|         |         |
1 -> 2 -> 3 -> 4 -> 5 -> 6
```

在构造好这样的数据结构之后，我们保留对第一层的唯一一个节点的引用`head`

### 数据搜索

在数据搜索的时候，我们从第一层开始，层层查找数据可能位于的位置，然后在合适的节点处转入下一层继续搜索。例如在上面的跳跃链表中查找 4：

0. 创建一个节点的引用，指向第一层第一个节点`node = 1`
1. 在第一层的时候，`node.val = 1 < 4`，试图向右搜索，但右侧无节点，因此转入下一层的`node = node.down = 1`
2. 在第二层的时候，`node.val = 1 < 4`，向右搜索`node = node.next = 5`。5 > 1，则转入下一层`node = node.down = 5`
3. 在第三层的时候，`node.val = 5 > 4`，向左搜索`node = node.prev = 3`。3 < 4，则转入下一层`node = node.down = 3`
4. 在第四层的时候，`node.val = 3 < 4`，向右搜索`node = node.next = 4`，找到指定元素

### 数据插入

数据插入需要一直找到最底层的合适的插入位置。例如需要再插入一个 4，则

1. 一直搜索到最底层的`node.val === 4`处，然后创建新的节点`newNode = new Node(4)`
2. 将插入位置的前后节点索引指向新节点，并更新旧节点的前后索引，得到`1 -> 2 -> 3 -> 4 -> 4 -> 5 -> 6`
3. 然后不断计算概率来提升新节点到上一层，直至概率低于`P`，或者已经达到最大层数
4. 根据具体条件来确定是否要更新第一层的引用节点`head`

假设新插入的 4 在插入后提升了两层，则得到

```
1
|
1 ----------------> 4 -> 5
|                   |    |
1 ------> 3 ------> 4 -> 5
|         |         |    |
1 -> 2 -> 3 -> 4 -> 4 -> 5 -> 6
```

### 数据删除

数据删除和插入操作类似，但不是深入到最底层，而是找到最上层的相符元素，然后再深入的进行删除。例如，删除 3 时，

1. 最早搜索到 3 是在倒数第二层
2. 先更新 3 节点前后节点的索引，从`1 -> 3 -> 4 -> 5`更新为`1 -> 4 -> 5`
3. 然后再进入到下一层，将`1 -> 2 -> 3 -> 4 -> 4 -> 5 -> 6`更新为`1 -> 2 -> 4 -> 4 -> 5 -> 6`
4. 根据具体条件来确定是否要更新第一层的引用节点`head`。例如，如果删除的是 1，则也需要更新`head`节点

最终得到

```
1
|
1 -----------> 4 -> 5
|              |    |
1 -----------> 4 -> 5
|              |    |
1 -> 2 -> 4 -> 4 -> 5 -> 6
```

### Reference

- [Wiki - 跳跃列表](https://zh.wikipedia.org/wiki/%E8%B7%B3%E8%B7%83%E5%88%97%E8%A1%A8)
- [跳跃表 - Redis 设计与实现](https://redisbook.readthedocs.io/en/latest/internal-datastruct/skiplist.html)
- [跳跃列表（Skip List）与其在 Redis 中的实现详解](https://www.jianshu.com/p/09c3b0835ba6)
- [Leetcode - No1206. 设计跳表](https://leetcode-cn.com/problems/design-skiplist/)