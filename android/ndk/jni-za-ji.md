---
description: 记录一些琐碎的知识
---

# JNI----杂记

## Q: JNI的native库只能用c, c++编写吗?

A:不是的, 其实只要是符合对JNI函数调用的规则就可以

规则有哪些呢?

分为静态绑定和动态绑定

**静态绑定**

* 符号表可见
* 命名符合Java native的`包名_类名_方法名`
* 符号名按照C语言的规范修饰

**动态绑定**

* 函数本身的定义没啥要求
* JNI可识别函数入口, JNI\_OnLoad\(\)进行注册即可

可以编写native的语言有`GoLang`,`Rust`, `Kotlin Native`, `Scala Native`

