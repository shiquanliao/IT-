# OOM相关

OOM发送的JVM内存区域

* OOM几乎覆盖了所有的内存区域, 但是我们通过指的是堆内存
* Native Headp在物理内存不够也会抛OOM

优化方法

* 使用合适的数据结构
* 避免使用枚举\(枚举占用:24Bytes, int占用4Bytes\), 枚举类型安全

  > A single enum can add about **1.0 to 1.4 KB** of size to your app's classes.dex file

* Bitmap的使用
  * 尽量根据实际需求选择合适的**分辨率**
  * 注意原始文件**分辨率**与**内存缩放**的结果
  * 不要使用帧动画, 使用代码实现动效
  * 考虑对bitmap的重采样和复用配置



Q: 为什么Android会自己设计`ArrayMap`, `SparseArray`

A: 在移动设备端内存资源很珍贵，`HashMap`为实现快速查询带来了很大内存的浪费. 所以开发了`ArrayMap`, 后面又优化了key为int的情况, 出现了`SparseArray`.

> SparseArray 主要是为了解决装箱拆箱, 提高效率, 传统的MAP都是泛型的, 所以基本类型都会有一个装箱拆箱过程.

[详细比较这三者的一篇文章](http://gityuan.com/2019/01/13/arraymap/)

## 内存优化5R法则

* Reduce 缩减: 降低图片分辨率/重采样/抽稀策略
* Reuse 复用: 池化策略/避免频繁创建对象, 减少GC压力
* Recycle 回收: 主动销毁, 结束, 避免内存泄漏/生命周期闭环
* Refactor 重构: 更适合的数据结构/更合理的程序框架
* Revalue重审: 谨慎使用large heap/第三方框架

