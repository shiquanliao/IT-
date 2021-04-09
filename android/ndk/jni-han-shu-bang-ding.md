---
description: Java Native方法跟Native函数绑定方式
---

# JNI函数绑定

jni函数绑定有种方式：

* 静态绑定：通过命名规则映射
* 动态绑定：通过jni函数注册

#### 静态绑定

```text
// -----------java
package com.stone.nativeC;

public class NativeCInf{
    public static native void callNativeStatic();
}


// -----------native
extern "C" JNIEXPORT void JNICALL
Java_com_stone_nativeC_NativeCInf_callNativeStatic(JNIEnv *, jclass)

/**
* 当java层为静态函数的时候，为class
* 当java层为成员函数的时候，为jobject
*/
```

#### 动态绑定





