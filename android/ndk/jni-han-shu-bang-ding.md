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

[**JNI 方法解析 ( JNIEXPORT 与 JNICALL 宏定义作用)**](https://blog.csdn.net/shulianghan/article/details/104072587)

`extern "C"`的作用：**避免C++编绎器按照C++的方式去编绎C函数 **

JNIEXPROT的作用：**强制公开这个类，让其可见**

```c++
#define JNIEXPORT __attribute__((visibity("defalut")))
```

JNICALL的作用：函数入栈规则

```c++
# 在Android中为空
#define JNICALL
```



#### 动态绑定

```
int registerMethods(JNIEnv *env, const char *className, const JNINativeMethod *method,
			int methodsLength){
	// step1: get class
    jclass clazz = env->FindClass(className);
    if(class == NULL) return JNI_ERROR;
    
    // step2: register methods
    if((env->RegisterNatives(class, methods, methodsLength)) < 0)
    	return JNI_ERROR;
    return JNI_OK;
}
```

特点：

* 动态绑定可以在任何时刻触发
* 动态绑定之前根据静态规则查找Native函数
* 动态绑定可以在绑定之后任何时刻取消绑定

动态跟静态绑定比较

|                  | 动态绑定     | 静态绑定                      |
| ---------------- | ------------ | ----------------------------- |
| Native函数名     | 无要求       | 固定规则且采用C的名称修饰规则 |
| Native函数可见性 | 无要求       | 可见                          |
| 动态替换         | 可以         | 不可以                        |
| 调用性能         | 无需查找     | 有额外的查找开销              |
| 开发体验         | 几乎无副作用 | 重构代码时比较繁琐            |
| Android Studio   | 不能关联跳转 | 可以关联跳转                  |

**理解动态绑定跟静态绑定的互补关系跟调用时机（可以作为技巧的Hook点，运行时替换so库）**