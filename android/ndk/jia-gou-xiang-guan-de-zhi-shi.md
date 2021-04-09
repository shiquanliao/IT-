# JNI----CPU架构相关的知识

## CPU架构适配

### 兼容性

Android中CPU架构有哪些:

* mips系列：mips, mips64
* x86系列：x86, x86\_64
* arm系列：armeabi, armeabi-v7a, arm64-v8a

**兼容模式运行的Native库无法获得最优性能， 兼容模式容易出现一些难以排查的内存问题，系统优先加载对应架构目录下的SO库** 





### so库体积优化

* 默认隐藏所有符号，只公开必要的
  * -fvisibility=hidden
* 禁用C++ Exception & RTTI
  * -fno-exception -fno-rtti
* 不要使用iostream, 优先使用Android Log
* 使用gc-sections 去除无用代码
  * LOCAL\_CFLAGS += -ffunction-sections -fdata-sections
  * LOCAL\_LDFLAGS += -WI, --gc-sections



### APPso库瘦身

* 为APP提供不同架构的Nativie库，构建时分包

```text
splits{
    abi{
        enable true
        reset()
        include "armeabi-v7a", "arm64-v8a", "x86", "x86-64"
        universalApk true
    }
}
```

* 动态加载SO库（比如：机器学习模型so库）







