# 图片缓存



评价一个缓存算法好坏的维度

* 获取成本
* 缓存成本
* 缓存价值

相关算法`LRU(Least Recently Used)`, `LFU(Least Frequently Used)`

**图片缓存算法一般使用`LRU`**

### LruCache

```java
public class LruCache<K, V>{
    private final LinkedHashMap<K, V> map;
    
    /** Size of this cache in units. Not necessarily the number of elements. */
    private int size;
    private int maxSize;
    private int putCount;
}
```







