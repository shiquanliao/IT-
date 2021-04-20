# 什么是反射

[**Reflection in Java**](https://www.geeksforgeeks.org/reflection-in-java/)

[关于反射的一篇很好的文章](https://www.slideshare.net/kim.mens/basics-of-reflection-in-java)



反射: 框架设计的灵魂(赋予了java语言更多动态特性)

**反射的定义: 将类的各个组成部分(属性, 构造函数, 方法)封装为其他对象, 这就是反射的机制**



## 获取Class的方法

* **Class.name("全类名")**: 将字节码加载进内存, 返回Class对象
* **类名.class**: 通过类名的属性class获取
* **对象.getclass()**: getClass定义在Object类中

{% hint style="info" %}
**同一个字节码(*.class)文件在一次运行中, 如果类加载器不变, 只会被加载一次, 不论通过上面3种方式的哪一种加载, 他们都是相同的, ==比较为true**
{% endhint %}

**Class.name("全类名"), 类名.class, 对象.getClass()区别**

```java
public class Test {

    public int a;
    protected String b;
    int c;
    private String d;

    public Test() {
        System.out.println("constructor init");
    }

    static {
        System.out.println("static block init");
    }

    {
        System.out.println("default block init");
    }
}
```

```java
// Class.name("全类名")
// 会执行静态代码块

// 类名.class
// 全都不会执行, 仅仅把类装载到内存中

// 对象.getClass()
// 全部会执行, 因为对象都有了, 所有会全部执行
```



Class的常用功能

* 获取成员变量们

  ```java
  public Field getField(String name)
  public Field[] getFields() throws SecurityException 
  
  public native Field[] getDeclaredFields();
  public native Field getDeclaredField(String name) throws NoSuchFieldException;
  ```

* 获取构造函数们

  * 能创建对象, `newInstance`
  * 如果是使用空参的构造函数, 可以简化为通过`Class.newInstance()`直接创建, 不需要反射构造函数

  ```java
  public Constructor<T> getConstructor(Class<?>... parameterTypes);
  public Constructor<?>[] getConstructors() throws SecurityException;
  
  
  ```

  

* 获取成员方法们

* 获取类名
