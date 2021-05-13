# HashMap, HashSet

## HashMap

面试问题:

* 你看过HashMap的源码吗, 知道原理吗

  ```java
  原理: 数组+链表(红黑树)的方法
  实体为Node<K,V>[] table
  static class Node<K,V> implements Map.Entry<K,V>{
  	final int hash;
      final K key;
      V value;
      Node<K,V> next;
  }
  
  ```

  

* 为什么用数组+链表(红黑树)

  ```
  数组是为了查询快(O(1)), 插入在没有碰撞的条件下为(O(1))
  hash时间复杂度为O(1)
  ```

  

* hash冲突的有几种解决方案

  ```
  1. 开放定址法
  2. 链地址法
  3. 再hash
  4. 公共溢出区域
  ```

  

* 可以用LinkedList代替数组吗

  ```
  可以, 但是效率不行.
  ```

  

* 既然可以,为什么HashMap不用LinkedList, 而使用数组

  ```
  数组查询为O(1), 链表为O(n)
  ```

### 什么时候扩容

如果bucket超过了`load factor * curretn capacity`, load factor默认为0.75

### 为什么hashMap扩容为2的n次幂

HashMap为了存取高效，要尽量较少碰撞，就是要尽量把数据分配均匀，每个链表长度大致相同，这个实现就在把数据存到哪个链表中的算法(对hash值取模).

`hash % length`, 但是这种运算不如位移运算快。

`hash % length == hash & (length-1)`

那为什么是2的n次方呢？

因为2的n次方实际就是1后面n个0，2的n次方-1，实际就是n个1。 例如长度为8时候，3&(8-1)=3 2&(8-1)=2 ，不同位置上，不碰撞。 而长度为5的时候，3&(5-1)=0 2&(5-1)=0，都在0上，出现碰撞了。 所以，保证容积是2的n次方，是为了保证在做(length-1)的时候，每一位都能&1 ，也就是和1111……1111111进行与运算。



### 知道hashmap中put元素的过程是什么样么?

对key的hashCode()做hash运算，计算index; 如果没碰撞直接放到bucket里； 如果碰撞了，以链表的形式存在buckets后； 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树(JDK1.8中的改动)； 如果节点已经存在就替换old value(保证key的唯一性) 如果bucket满了(超过load factor*current capacity)，就要resize。



### 知道hashmap中get元素的过程是什么样么?

对key的hashCode()做hash运算，计算index; 如果在bucket里的第一个节点里直接命中，则直接返回； 如果有冲突，则通过key.equals(k)去查找对应的Entry;

- 若为树，则在树中通过key.equals(k)查找，O(logn)；
- 若为链表，则在链表中通过key.equals(k)查找，O(n)。

*说说String中hashcode的实现?(此题频率很高)*

```java
public int hashCode(){
    int h = hash;
    if(h == 0 && value.length > 0){
        char val[] = value;
        
        for(int i = 0; i < value.length; i++){
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

哈希计算公式可以计为`s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`

那为什么以31为质数呢?

主要是因为31是一个奇质数，所以`31*i=32*i-i=(i<<5)-i`，这种位移与减法结合的计算相比一般的运算快很多。

*我不用红黑树，用二叉查找树可以么?* 可以。但是二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。

*HashMap在并发编程环境下有什么问题啊?*

- (1)多线程扩容，引起的死循环问题
- (2)多线程put的时候可能导致元素丢失
- (3)put非null元素后get出来的却是null

*你一般用什么作为HashMap的key?*

一般用Integer、String这种不可变类当HashMap当key，而且String最为常用。

[HashMap原文](https://zhuanlan.zhihu.com/p/76735726)

TreeMap底层是用红黑树实现的, 可以保证顺序



**ArrayMap和SparseArray比使用HashMap有更高的内存效率。**

[深度解读ArrayMap优势与缺陷](http://gityuan.com/2019/01/13/arraymap/)