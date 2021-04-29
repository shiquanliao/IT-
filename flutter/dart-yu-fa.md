# Dart语法

## 定义变量

* 明确声明\(Explicit\): **变量类型 变量名称 = 赋值;**

  ```dart
  String name = "stone";
  int  age = 18;
  double height = 1.88;
  print('${name}, $age, ${height}');
  ```

  > 注意事项: 定义的变量可以修改值, 但是不能赋值其他类型

  ```dart
  String content = 'Hello Dart';
  content = 'Hello World'; // 正确的
  content = 111; // 错误的, 将一个int值赋值给一个String变量
  ```

* 类型推导\(Type Inference\): **var/dynamic/const/final 变量名称 = 赋值;**

  ```dart
  // var的使用示例:
  var name = "stone";
  name = "liu";
  print(name.runtimeType);  // String

  //var的错误用法:
  var age = 18;
  age = 'why'; // 不可以将String赋值给一个int类型

  // dynamic的使用, 确实希望这样做,可以使用dynamic来声明变量:
  // 但是在开发中, 通常情况下不使用dynamic, 因为类型的变量会带来潜在的危险
  dynamic name = 'stone';
  print(name.runtimeType); // String
  name = 18;
  print(name.runtimeType); // int

  // final & const 的使用
  // final 和 const 都是用来定义常量的, 也就是定义之后赋值都不可以修改的
  final name = "stone";
  name = "liu"; // 错误做法

  const age = 19;
  age = 20; // 错误做法
  ```

  > final 和 const 区别:
  >
  > * const 在编译期就确定了
  > * final在赋值时,可以动态获取,运行时赋值一次

  ```dart
  String getName(){
      return "stone";
  }

  main(List<String> args){
      const name = getName(); // 错误的做法, 因为要执行函数才能获取到值
      final name = getName(); // 正确的做法
  }
  ```

_**const放在赋值语句的右边，可以共享对象，提高性能**_

```dart
class Person{
    const Person();
}

main(List<String> args){
    final a = const Person();
    final b = const Person();
    print(identical(a, b)); // true

    final m = Person();
    final n = Person();
    print(identical(m, n)); // false
}
```

## 数据类型

* 数字类型

  ```text
  // 字符串和数字之间的转化:
  var one = int.parse('111');
  var two = double.parse('12.22');
  print('$one ${one.runtimeType}'); // 111 int
  print('${two} ${two.runtimeType}'); // 12.22 double

  var num1 = 123;
  var num2 = 123.456;
  var num1Str = num1.toString();
  var num2StrD = num2.toStringAsFixed(2); // 保留两位小数
  ```

* 布尔类型

  ```text

  ```

  **注意: Dart中不能判断非0即真, 或者非空即真**

  Dart的类型安全性意味着您不能使用if\(非booleanvalue\)或assert\(非booleanvalue\)之类的代码。

  ```dart
  var message = 'Hello Dart';
  // 错误的写法
  if (message) {
    print(message)
  }
  ```

* 字符串类型

  ```dart
  var name = 'coderwhy';
  var age = 18;
  var height = 1.88;
  print('my name is ${name}, age is $age, height is $height');
  ```

* 集合类型: **List / set / map**

  ```text
  // List定义
  // 1.使用类型推导定义
  var letters = ['a', 'b', 'c', 'd'];
  print('$letters ${letters.runtimeType}');

  // 2.明确指定类型
  List<int> numbers = [1, 2, 3, 4];
  print("$numbers ${numbers.runtimeType}");
  ```

  ```dart
  // Set的定义
  // 1.使用类型推导定义
  var letterSet = {'a', 'b', 'c', 'd'};
  print('$lettersSet ${lettersSet.runtimeType}');

  // 2.明确指定类型
  Set<int> numbersSet = {1, 2, 3, 4};
  print('$numbersSet ${numbersSet.runtimeType}');
  ```

  ```dart
  // Map的定义
  // 1.使用类型推导定义
  var infoMap1 = {'name': 'stone', 'age': 18};
  print('$infoMap1 ${infoMap1.runtimeType}');

  // 2.明确指定类型
  Map<String, Object> infoMap2 = {'height': 1.88, 'address': '北京市'};
  print('$infoMap2 ${infoMap2.runtimeType}');
  ```

  **集合的常见操作**

  `length`

  ```dart
  // 获取集合的长度
  print(letters.length);
  print(lettersSet.length);
  print(infoMap1.length);
  ```

  `add` , `remove`

  ```dart
  // 添加/删除/包含元素
  numbers.add(5);
  numbersSet.add(5);
  print('$numbers $numbersSet');

  numbers.remove(1);
  numbersSet.remove(1);
  print('$numbers $numbersSet');

  print(numbers.contains(2));
  print(numbersSet.contains(2));

  // List根据index删除元素
  numbers.removeAt(3);
  print('$numbers');
  ```

  Map

  ```dart
  // Map的操作
  // 1.根据key获取value
  print(infoMap1['name']); // why

  // 2.获取所有的entries
  print('${infoMap1.entries} ${infoMap1.entries.runtimeType}'); // (MapEntry(name: why), MapEntry(age: 18)) MappedIterable<String, MapEntry<String, Object>>

  // 3.获取所有的keys
  print('${infoMap1.keys} ${infoMap1.keys.runtimeType}'); // (name, age) _CompactIterable<String>

  // 4.获取所有的values
  print('${infoMap1.values} ${infoMap1.values.runtimeType}'); // (why, 18) _CompactIterable<Object>

  // 5.判断是否包含某个key或者value
  print('${infoMap1.containsKey('age')} ${infoMap1.containsValue(18)}'); // true true

  // 6.根据key删除元素
  infoMap1.remove('age');
  print('${infoMap1}'); // {name: }
  ```

## 函数

* 参数问题

  \`\`\` 1. 命名可选参数: {param1, param2, ...} 2. 位置可选参数: \[param1, param2, ...\]

  // 命名可选参数 printInfo1\(String name, {int age, double height}\) { print\('name=$name age=$age height=$height'\); }

  // 调用printInfo1函数 printInfo1\('why'\); // name=why age=null height=null printInfo1\('why', age: 18\); // name=why age=18 height=null printInfo1\('why', age: 18, height: 1.88\); // name=why age=18 height=1.88 printInfo1\('why', height: 1.88\); // name=why age=null height=1.88

  // 定义位置可选参数 printInfo2\(String name, \[int age, double height\]\) { print\('name=$name age=$age height=$height'\); }

  // 调用printInfo2函数 printInfo2\('why'\); // name=why age=null height=null printInfo2\('why', 18\); // name=why age=18 height=null printInfo2\('why', 18, 1.88\); // name=why age=18 height=1.88

// 参数的默认值 printInfo4\(String name, {int age = 18, double height=1.88}\) { print\('name=$name age=$age height=$height'\); }

```text
   **返回值问题: 所有函数都返回一个值。如果没有指定返回值，则语句返回null;隐式附加到函数体。**

  ```dart
  main(List<String> args) {
    print(foo()); // null
  }

  foo() {
    print('foo function');
  }
```

### 赋值

* ??=赋值操作

  ```dart
  var name2 = null;
  name2 ??= "james";
  ```

* 条件运算符: **expr1 ?? expr2**

  ```dart
  var temp = "stone";
  var name = temp ?? "liu";
  ```

* 级联语法: `..`

  ```dart
  class Person {
    String name;

    void run() {
      print("${name} is running");
    }

    void eat() {
      print("${name} is eating");
    }

    void swim() {
      print("${name} is swimming");
    }
  }

  main(List<String> args) {
    final p1 = Person();
    p1.name = 'why';
    p1.run();
    p1.eat();
    p1.swim();

    final p2 = Person()
                ..name = "why"
                ..run()
                ..eat()
                ..swim();
  }
  ```

## 类

* 构造函数

  ```dart
  // 新的构造方法
  Person.fromMap(Map<String, Object> map) {
    this.name = map['name'];
    this.age = map['age'];
  }

  // 通过上面的构造方法创建对象
  var p3 = new Person.fromMap({'name': 'kobe', 'age': 30});
  print(p3);
  ```

* 初始化列表

  ```dart
  class Point {
    final num x;
    final num y;
    final num distance;

    // 错误写法
    // Point(this.x, this.y) {
    //   distance = sqrt(x * x + y * y);
    // }

    // 正确的写法
    Point(this.x, this.y) : distance = sqrt(x * x + y * y);
  }
  ```

* 重定向构造方法

  ```dart
  class Person {
    String name;
    int age;

    Person(this.name, this.age);
    Person.fromName(String name) : this(name, 0);
  }
  ```

* 常量构造函数

  ```dart
  main(List<String> args) {
    var p1 = const Person('why');
    var p2 = const Person('why');
    print(identical(p1, p2)); // true
  }

  class Person {
    final String name;

    const Person(this.name);
  }
  ```

  > 注意点:
  >
  > 注意一：拥有常量构造方法的类中，所有的\*_成员变量必须是final修饰\*_的.
  >
  > 注意二: 为了可以通过常量构造方法，创建出相同的对象，不再使用 **new**关键字，而是使用const关键字. 如果是将结果赋值给const修饰的标识符时，const可以省略.

* 隐式接口

  ```dart
  abstract class Runner {
    run();
  }

  abstract class Flyer {
    fly();
  }

  class SuperMan implements Runner, Flyer {
    @override
    run() {
      print('超人在奔跑');
    }

    @override
    fly() {
      print('超人在飞');
    }
  }
  ```

* Mixin混入

  ```dart
  main(List<String> args) {
    var superMan = SuperMain();
    superMan.run();
    superMan.fly();
  }

  mixin Runner {
    run() {
      print('在奔跑');
    }
  }

  mixin Flyer {
    fly() {
      print('在飞翔');
    }
  }

  // implements的方式要求必须对其中的方法进行重新实现
  // class SuperMan implements Runner, Flyer {}

  class SuperMain with Runner, Flyer {

  }
  ```

## 泛型

* 泛型方法

  ```dart
  main(List<String> args) {
    var names = ['why', 'kobe'];
    var first = getFirst(names);
    print('$first ${first.runtimeType}'); // why String
  }

  T getFirst<T>(List<T> ts) {
    return ts[0];
  }
  ```

* 泛型类

库文件中内容的显示和隐藏

```dart
import 'lib/student/student.dart' show Student, Person;

import 'lib/student/student.dart' hide Person;
```

**库中内容和当前文件中的名字冲突**

当各个库有命名冲突的时候，可以使用`as关键字`来使用命名空间

```dart
import 'lib/student/student.dart' as Stu;

Stu.Student s = new Stu.Student();
```

在开发自己的库太大的时候怎么拆分

`mathUtils.dart`

```dart
int sum(int num1, int num2) {
  return num1 + num2;
}
```

`dateUtils.dart`文件

```dart
String dateFormat(DateTime date) {
  return "2020-12-12";
}
```

`utils.dart`文件

```dart
library utils;

export "mathUtils.dart";
export "dateUtils.dart";
```

`test_libary.dart`文件

```dart
import "lib/utils.dart";

main(List<String> args) {
  print(sum(10, 20));
  print(dateFormat(DateTime.now()));
}
```

### 多核CPU的利用

**如何创建Isolate呢？**

```dart
import "dart:isolate";

main(List<String> args){
    Isolate.spawn(foo, "hello Islate");
}

void foo(info){
    print("new isolate: $info");
}
```

#### Isolate通信机制

```dart
import "dart:isolate";

main(List<String> args) async{
    // 1.创建管道
    ReceivePort receivePort = ReceivePort();

    // 2.创建新的Isolate
    Isolate isolate = await Isolate.spawn<SendPort>(foo, receivePort.sendPort);

    // 3.监听管道消息
    receivePort.listen((data){
        print('Data: $data');
        // 不再使用时，我们会关闭管道
        receivePort.close();
        // 需要将isolate杀死
        isolate?.kill(priority: Isolate.immediate);
    });
}

void foo(SendPort sendPort){
    sendPort.send("Hello world");
}
```

但是我们上面的通信变成了单向通信，如果需要双向通信呢？

* 事实上双向通信的代码会比较麻烦；
* Flutter提供了支持并发计算的`compute`函数，它内部封装了Isolate的创建和双向通信；
* 利用它我们可以充分利用多核心CPU，并且使用起来也非常简单；

注意：下面的代码不是dart的API，而是Flutter的API，所以只有在Flutter项目中才能运行

```dart
main(List<String> args) async {  
    int result = await compute(powerNum, 5); 
    print(result);
}
int powerNum(int num) {  return num * num;}
```

