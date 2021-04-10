---

description: 谈论在NDK开发中，关于出现了crash怎么捕获异常
---

# JNI----native异常捕获

相关的技术点

> * LInux信号
> * native层任意位置获取jclass的方法
> * 底层线程跟JVM的关系

异常捕获我们需要做的工作有: 

1. 捕获异常
2. 清理native层和java层的资源
3. 相关的异常信息上报

**javaVM**

```c++
static JavaVM *javaVM;

extern "C" JNIEXPORT jint JNICALL
JNI_OnLoad(JavaVM *vm, void *) {
    javaVM = vm;
    ...
}
```


**JNIEnvHelper**

```c++
class JNIEnvHelper {
public:
    JNIEnv *env;

    JNIEnvHelper() {
        needDetach = false;
        if (javaVM->GetEnv((void **) &env, JNI_VERSION_1_6) != JNI_OK) {
            if (javaVM->AttachCurrentThread(&env, nullptr) == JNI_OK) {
                needDetach = true;
            }
        }
    }

    virtual ~JNIEnvHelper() {
        if (needDetach) javaVM->DetachCurrentThread();
    }

private:
    bool needDetach;
};
```



**classLoader**

```c++
static jobject classLoader;

jint setUpClassLoader(JNIEnv *env) {
    jclass applicationClass = env->FindClass("com/stone/nativedemo/_App");
    jclass classClass = env->GetObjectClass(applicationClass);
    jmethodID getClassLoaderMethod = env->GetMethodID(classClass, "getClassLoader",
                                                      "()Ljava/lang/ClassLoader;");
    classLoader = env->NewGlobalRef(env->CallObjectMethod(applicationClass, getClassLoaderMethod));
    return classLoader == nullptr ? JNI_ERR : JNI_OK;
}
```



**findClass**

```c++
jclass findClass(JNIEnv *env, const char *name) {
    if (env == nullptr) return nullptr;
    jclass classLoaderClass = env->GetObjectClass(classLoader);
    jmethodID loadClassMethod = env->GetMethodID(classLoaderClass, "loadClass",
                                                 "(Ljava/lang/String;)Ljava/lang/Class;");
    jclass cls = static_cast<jclass>(env->CallObjectMethod(classLoader, loadClassMethod,
                                                           env->NewStringUTF(name)));
    return cls;
}
```



**android_signal_handler**

```c++
static void android_signal_handler(int signum, siginfo_t *info, void *reserved) {
    if (javaVM) {
        JNIEnvHelper jniEnvHelper;
        jclass errorHandleClass = findClass(jniEnvHelper.env,
                                            "com/stone/nativeC/HandleNativeError");
        if (errorHandleClass == nullptr) {
            LOGE("Cannot get error handle class");
        } else {
            jmethodID errorHandleMethod = jniEnvHelper.env->GetStaticMethodID(errorHandleClass,
                                                                              "nativeErrorCallback",
                                                                              "(I)V");
            if (errorHandleMethod == nullptr) {
                LOGE("Cannot get error handle method.");
            } else {
                LOGE("Call java back to notify a native crash!");
                jniEnvHelper.env->CallStaticVoidMethod(errorHandleClass, errorHandleMethod, signum);
            }
        }
    } else {
        LOGE("Jni unloaded.");
    }
    old_signalHandlers[signum].sa_handler(signum);
}
```





```c++
static struct sigaction old_signalHandlers[NSIG]; // old_signalHandlers 为static数组

void setUpGlobalSignalHandler() {
    struct sigaction handler;
    memset(&handler, 0, sizeof(struct sigaction));
    handler.sa_sigaction = android_signal_handler;
    handler.sa_flags = SA_RESETHAND;
#define CATCHSIG(X) sigaction(X, &handler, &old_signalHandlers[X])
    CATCHSIG(SIGQUIT);
    CATCHSIG(SIGILL);
    CATCHSIG(SIGABRT);
    CATCHSIG(SIGBUS);
    CATCHSIG(SIGFPE);
    CATCHSIG(SIGSEGV);
    CATCHSIG(SIGPIPE);
    CATCHSIG(SIGTERM);
#undef CATCHSIG
}
```

------

java层

```java
public class HandleNativeError{
	public static void nativeErrorCallback(int signal){
        Log.e("NativeError","[" + Thread.currentThread().getName() + "] Signal: " + signal);
    }
}
```

