# 开发技巧

当我们打入`stful`的时候，回联想快捷生成模板代码

```text
class  extends StatefulWidget {
  @override
  _State createState() => _State();
}


class _State extends State<> {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### flutter 跟原生交互:关于传递Drawable的问题

解决方案

```text
// native 
//添加图标
Drawable appIcon = getAppIcon(context, bean.getKey());
byte[] bytes = ConvertUtils.drawable2Bytes(appIcon);
String encode2String = EncodeUtils.base64Encode2String(bytes);
bean.setIconBase64(encode2String);

// flutter
import 'dart:convert' show base64;
List<Uint8List> _images = [];
setState(() {
  _images.add(base64.decode(info['iconBase64']));
});
```

\*\*\*\*[**flutter 跟原生通信方式**](https://medium.com/flutter/flutter-platform-channels-ce7f540a104e)\*\*\*\*

#### const vs final

> const必须在编译期就确定， final可以后期在runtime赋值

