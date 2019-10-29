#### 伸缩性角度去看 HashMap

* ###### HashMap

  * Hash

    散列：将一个任意的长度通过Hash 算法转换成一个固定值

  * Map

    键值对

  * 总结：通过Hash算出来的值，然后通过值定位到这个map，然后value存储到这map中

* 源码分析



* 手写实现

* ##### Hashmap的底层是什么

  * **底层使用哈希表（数组+链表），当链表长度超过 8 时（其实是9），会将链表转成红黑树，以实现 O(log n) 时间复杂度内查找**
  * 为什么使用数组? 
    * 使用一段连续存储单元存储数据。对于指定下标的查找，时间复杂度为 O(1)，对于一般的插入删除操作，涉及到数组元素的移动，时间复杂度为 O(n) , 因此引入链表
  * 链表的特点
    * 对链表的增加删除操作在查找到操作位置之后，只需要处理节点间的引用即可，时间复杂度为 O(1) ，查找操作需要遍历链表，所有节点进行逐一对比，时间复杂度为 O(n) , 因此引入了红黑树
  * 红黑树的特点
    * 支持查找、插入、删除等操作，其时间复杂度最坏为 O(logn)

* ##### 红黑树是什么？

  * 一种接近平衡二叉搜索树，在每个结点上增加一个存储位表示结点的颜色，可以是 Red 或 Black ，通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出两倍，因而是接近平衡的
  * 红黑树的性质
    * 每个结点要么是红要么是黑
    * 根结点是黑的
    * 每个叶子结点（指树尾端 NIL指针或 NULL结点）都是黑的
    * 如果一个结点是红的，那么它的两个子结点都是黑的
    * 对于任意结点而言，其到叶子结点树尾端NIL指针的每条路径都包含相同数目的黑结点

* ##### 进行位干扰时要使用 (n - 1) & hash？
  
  ```java
  /*
  (n - 1) & hash == hash % (n-1) 相当于取模运算
  例如   15 	0000 0000 0000 0000 0000 0000 0000 1111
        hash    0110 1000 1101 1001 1001 1111 0001 1101
        结果	0000 0000 0000 0000 0000 0000 0000 1101
  分析：任意二进制数 x 和15(01111)进行与运算，得到的结果就是 x 的低四位，也就将值限制在 0000-1111，也就是0-15之间，避免了数组越界以及大量的hash碰撞   位运算与取模运算实质相同，但是位运算的时间效率比取模运算好一个数量级，另一方面，这也是数组容量必须是2的次方的原因，因为2的次方减1之后就是01111...
  因为数组扩容，需要rehash  ，一旦数据量很大时，一定会造成性能的降低，也就意味性能的瓶颈，使用位运算的优势就出来了
  */
  ```
* ##### hash 函数怎么实现的？ 还有哪些hash 的实现方式？
  
  * hashCode 值的高 16 bit 和低16 bit 进行异或运算拿到 hash 
  * 然后 ( n - 1 ) & hash ，也就是数组容量 - 1 & hash 得到的结果就是所在的hash桶 下标 
  
* ##### 扩容因子为什么是0.75？
  
  * 设置为1  ，hash碰撞增多，查询效率低
  * 设置为0.5，空间利用率低
  * 根据牛顿二项式推算出来得到 log2 ，约等于0.69 为最均衡的值 
  * 因此取空间利用率与时间复杂度的均值 0.75
  
* ##### HashMap中put方法过程
  
  * 对 key求hash值，然后在计算下标
  * 如果没有碰撞，直接放入桶中
  * 如果碰撞了，以链表的方式连接到后面
  * 如果链表长度超过阈值 ( TREEIFY_THRESHOLD == 8 )，就把链表转成红黑树
  * 如果相同的结点已经存在就替换旧结点
* 如果桶满了（容量 * 扩容因子 0.75），就需要 resize ，数组容量必须是 2 的次方，且是以 2 倍的方式进行扩容
  
* ##### HashMap 怎么解决冲突？ 扩容过程是什么？ 假如一个值在原数组中，现在移动到了新数组，位置肯定发生改变，那是什么定位到在这个值在新数组中的位置？
  
  * 将新结点加到链表后
  * 容量扩充为原来的 2 倍，然后对每个结点重新计算哈希值
  * 这个值可能在两个地方，一个是原下标的位置，另一个是在下标为 <原下标+原容量>的位置
  
* ##### 抛开 Hash Map，Hash冲突有哪些解决方法？
  
  * 开放定址，链地址法
  
* ##### 针对 HashMap 中某个Entry 链太长，查找的时间复杂度可能达到 O(n) ，怎么优化？
  
  * 将链表转为红黑树。JDK8 已经实现

* JDK7 中HashMap在**高并发情况下可能会出现死锁问题**
  
* 死锁(链表形成了环),高并发场景下,进行扩容,可能使链表形成了环
  
* ##### JDK8 中由链表切换为红黑树的阈值真的是 8 吗?

  * 链表转红黑树时链表长度其实不是8  而是 9 
  * 8 是根据泊松分布推导出来的