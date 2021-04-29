# 面试总结

[面试题---1](https://www.jianshu.com/p/93821c12a825)

## 1. dart是什么，和flutter有什么关系？

dart是一种面向对象语言，dart是flutter的程序开发语言.

Flutter is Google's UI toolkit.

## 2. main\(\)和runApp\(\)函数在flutter的作用分别是什么？有什么关系吗？

main函数类似于java语言的入口函数, runApp函数是渲染根widget树的函数,

## 3. 什么是widget? 在flutter里有几种类型的widget？分别有什么区别？能分别说一下生命周期吗？

widget是flutter里面的UI组件,2种widget: **statefulWidget, statelessWidget.**

statelessWidget不会自己重新`rebuild`自己，但是statefulWidget会

## 4. 在flutter里streams是什么？有几种streams？有什么场景用到它？

Stream 用来处理连续的异步操作，Stream 是一个抽象类，用于表示一序列异步数据的源。它是一种产生连续事件的方式，可以生成数据事件或者错误事件，以及流结束时的完成事件

Stream 分单订阅流和广播流。

网络状态的监控

使用方式

```dart
Future<int> sumStream(Stream<int> stream) async{
    var sum = 0;
    await for( var value in stream){
        sum += value;
    }
    return sum;
}

Stream<int> countStream(int to) async*{
    for(int i = 1; i <= to; i++){
        yield i;
    }
}

main() async{
    var stream = countStream(10);
    var sum = await sumStream(stream);
    print(sum); // 55
}
```

```dart
void main(){
  StreamController<double> controller = StreamController();
  Stream stream = controller.stream;
  StreamSubscription<double> streamSubscription  = stream.listen((event) {
    print('Value from controller: $event');
  });


  controller.add(12);
  print('---------end');

  streamSubscription.cancel();


  StreamController<double> controller2 = StreamController.broadcast();


  getRandomValue().listen((event) {
    print('1st: $event');
  });

}

Stream<double> getRandomValue() async* {
  var random = Random(2);

  while (true) {
    await Future.delayed(Duration(seconds: 1));
    yield random.nextDouble();
  }
}
```

EventBus也是基于Stream实现的.

## 5. future 和steam有什么不一样？

在 Flutter 中有两种处理异步操作的方式 Future 和 Stream，Future 用于处理单个异步操作，Stream 用来处理连续的异步操作。

## 6. 什么是flutter里的key? 有什么用？

[flutter官方解释文档](https://medium.com/flutter/keys-what-are-they-good-for-13cb51742e7d)

