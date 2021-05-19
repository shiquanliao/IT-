# 动态化

## 方案对比

* 替换Flutter编译产物
* AOT 搭载 JIT(需要抽离一份DartVM独立编译, 再以动态库的方式引入. 会增加包体积)
*  动态生产 DSL: 类似React Native框架(js转dart)
* 纯Dart的动态化(美团的Flap)

