# 插件化

## Flutter 与原生通信整体流程

```dart
Future<dynamic> invokeMethod(String method, [dynamic arguments]) async{
    assert(method != null);
    // send message
    final dynamic result = await BinaryMessage.send(
        name,
        codec.encodeMethodCall(MethodCall(method, arguments))
    );
    if(result == null)
        throw MissingPluginException("No implementation found for method ...")
    return codec.decodeEnvelope(result);
}
```

* 这里截取了 send 方法里关键代码， dart 层最终通过调用了 native 方法 Window\_sendPlatformMessage，将序列化后的对象通过 c 层进行发送：

```dart
{
    final _MessageHandler handler = _mockHandlers[channel];
    if(handler != null)
        return handler(message);
    return _sendPlatFormMessage(channel, message);
}

String _sendPlatFormMessage(String name, PlatFormMessageResponseCallback callback,
                           ByteData data) native 'Window_sendPlatformMessage';
```

* 我们在 Flutter engine 的 native 代码中可以找到上述 native 方法的对应实现,这里截取关键部分,可以看到最后是交给了 WindowClient 的 handlePlatformMessage 方法进行实现，我们继续往下跟：

```c
...
dart_state->window()->client()->HandlePlatformMessage(
    fml::MakeRefCounted<PlatformMessage>(name, response)
);
```

* （这里以 Android 举例，iOS 同理）可以看到，在 Android 平台 HandlePlatformMessage 方法中，调用到了 JNI 方法，将 c 层收到的信息向 java 层抛：

```cpp
void PlatformViewAndroid::HandlePlatformMessage(
    fml::RefPtr<blink::PlatformMessage> message) {
  JNIEnv* env = fml::jni::AttachCurrentThread();
  fml::jni::ScopedJavaLocalRef<jobject> view = java_object_.get(env);
  auto java_channel = fml::jni::StringToJavaString(env, message->channel());
  if (message->hasData()) {
    fml::jni::ScopedJavaLocalRef<jbyteArray> message_array(env, env->NewByteArray(message->data().size()));
    env->SetByteArrayRegion(
        message_array.obj(), 0, message->data().size(),
        reinterpret_cast<const jbyte*>(message->data().data()));
    message = nullptr;
    // This call can re-enter in InvokePlatformMessageXxxResponseCallback.
    FlutterViewHandlePlatformMessage(env, view.obj(), java_channel.obj(),
                                     message_array.obj(), response_id);
  } else {
    message = nullptr;
    // This call can re-enter in InvokePlatformMessageXxxResponseCallback.
    FlutterViewHandlePlatformMessage(env, view.obj(), java_channel.obj(),
                                     nullptr, response_id);
  }
}
```

* 看一下 JNI 对应的 java 方法，最终通过 handler.onMessage\(\),完成了本次 dart 信息的传递。方法中的 handler，就是我们前面提到的 MethodHandler，也是我们插件的 Native 模块注册的 MethodHandler

```text
  private void handlePlatformMessage(final String channel, byte[] message, final int replyId) {
        this.assertAttached();
        BinaryMessageHandler handler = (BinaryMessageHandler)this.mMessageHandlers.get(channel);
        if (handler != null) {
            try {
                ByteBuffer buffer = message == null ? null : ByteBuffer.wrap(message);
                handler.onMessage(buffer, new BinaryReply() {
                    // ...
                });
            } catch (Exception var6) {
                // ...
            }
        } else {
            Log.e("FlutterNativeView", "Uncaught exception in binary message listener", var6);
            nativeInvokePlatformMessageEmptyResponseCallback(this.mNativePlatformView, replyId);
        }
    }
```

