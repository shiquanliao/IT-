---
description: 探究java跟native之间的数据传递方式跟需要注意的地方
---

# JNI----数据传递

## 字符串操作

* GetStringUTFChars/ReleaseStringUTFChars

  ```c++
  const char* (*GetStringUTFChars)(JNIEnv*, jstring, jboolean*);
  // 返回const char*
  // 编码格式为Modified-UTF-8
  // \0编码成0xC080, 不会影响C字符串结尾
  ```

  

* GetStringChars/ReleaseStringChars

  ```c++
  const jchar* (*GetStringChars)(JNIEnv*, jstring, jboolean*);
  // 返回const jchar*
  // JNI函数自动处理字节序转换
  // java字节序是大段，c是小段，网络序是大段
  ```

* GetStringUTFRegion/GetStringRegion

  ```c++
  void   (*GetStringUTFRegion)(JNIEnv*, jstring, jsize, jsize, char*);
  // 先在c层创建足够容量的空间
  // 将字符串的莫一部分复制到开辟好的空间
  // 针对性复制， 少量读取时效率更优
  ```

  

* GetStringCritical/ReleaseStringCritical

  ```
  const jchar* (*GetStringCritical)(JNIEnv*, jstring, jboolean*);
  // 调用对中间会停掉Jvm GC --- 这个时间段不允许GC
  // 调用对之间不能有其他的JNI操作
  // 调用对可以嵌套
  ```

  

  