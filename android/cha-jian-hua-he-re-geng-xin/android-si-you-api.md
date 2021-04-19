# Android私有API

## 私有API有哪些类型

```java
/**
* @hide
*/
@SystemApi
public boolean convertFromTranslucent(){
    try{
        mTranslucentCallback = null;
        if(ActivityManager.getService().convertFromTranslucent(mToken)){
            WindowManagerGlobal.genInstance().changeCanvasOpacity(mToken, true);
        }
    }catch(RemoteException e){
        // pass
    }
}

// 这种情况有2种解决方案
// 1. 通过提供jar来骗过编译器, 能编译成功, 之后在运行时就能找到
// 2. 通过反射调用的方式
```

## 如何访问私有API

* 自行编译源码, 并导入项目工程\(只对public hide方法有效\)
* 使用反射访问私有\(private\)的方法

  ```java
  Method initMethod = AssetManager.class.getDeclaredMethod("init");
  initMethod.setAccessible(true);
  ```

{% hint style="info" %}
`setAccessible(true)不仅仅可以`**`绕过访问权限`**`的控制,还可以`**`修改final变量`**
{% endhint %}

## Android P如何做到对私有API访问的限制

## 如何规避限制

