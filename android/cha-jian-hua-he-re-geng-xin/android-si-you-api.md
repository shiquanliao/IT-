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

Android P 增加了一个API名单

| 白名单 | SDK, 所有APP均能访问 |
| :--- | :--- |
| 浅灰名单 | 仍可以访问的非SDK函数/字段 |
| 深灰名单 | 1. 对目标SDK低于API级别28的应用,允许使用深灰名单接口  2. 对目标SDK低于API级别28的应用,允许使用深灰名单接口 |
| 黑名单 | 受限, 无论目标SDK如何, 平台将表现为似乎接口不存在 |

**Android实现这些api的限制就是通过修改java 反射的逻辑来实现的**

```java
public Method getDeclaredMethod(String name, Class<?>...parameterTypes)
```

```java
private native Method getDeclaredMethodInternal(String name, Class<?>[] args);
```

```cpp
static jobject Class_getDeclaredMethodInternal(JNIEnv *env, jobject javaThis, 
                                              jstring name, jobject args){
    ...
    if(reslut == nullptr || ShouldBlockAccessToMember(result->GetArtMethod(), soa.Self())){
        return nullptr;
    }
    return soa.AddLocalReference<jobject>(result.Get());
}
```

```cpp
ALWAYS_INLINE
static bool ShouldBlockAccessToMember(T *member, Thread *self){
    hiddenapi::Action action = hiddenapi:GetMemberAction(
    member, self, IsCallerTrusted, hiddenapi::Reflection);
    ...
    return action == hidenapi::kDeny;
}
```

## 如何规避限制

