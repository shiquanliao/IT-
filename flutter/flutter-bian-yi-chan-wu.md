# Flutter包瘦身

## Flutter 编译之后的产物

* IOS端

  | 产物           | 大小  | 占比  | 介绍                                                         |
  | -------------- | ----- | ----- | ------------------------------------------------------------ |
  | Flutter        | 7.2MB | 50.6% | 即为Flutter Engine, C++代码编译而成                          |
  | icudtl.dart    | 883kb | 5.7%  | 国际化支持相关数据文件,大小固定为883kb                       |
  | App            | 5.2MB | 36.7% | dart业务代码AOT编译产物,包含:<br />_kDartIsolateSnapshotData,<br />_kDartVmSnapshotData<br />_kDartIsolateSnapshotInstructions,<br />_kDartVmSnapshotInstructions |
  | Flutter_assets | 1MB   | 7%    | 图片,字体,LICENSE等静态资源                                  |

  

* Android端

  | 产物           | 大小  | 占比 | 介绍                                          |
  | -------------- | ----- | ---- | --------------------------------------------- |
  | libflutter.so  | 3.2MB | 62%  | Flutter Engine, c++代码编译产物               |
  | libapp.so      | 1.5MB | 29%  | Flutter业务与框架的Dart代码编译产物,4部分组成 |
  | flutter.jar    | 310KB | 5.9% | Flutter引擎的java代码编译产物                 |
  | flutter_assets | 170KB | 3.1% | 图片,字体,LICENSE等静态资源                   |

  