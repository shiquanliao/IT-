# 关于Handler Delay消息延时可靠性

**主要的原因是消息太多, 分发不过来, UI主线程处理比较慢,到时延时消息不精准.**

MessageQueue中的enqueueMessage(Message msg, long when)

```java
boolean enqueueMessage(Message msg, long when){
    ...
    synchronized(this){
        Message p = mMessages;
        boolean needWake;
        if(p == null || when ==0 || when < p.when){
            // New head, wake up the event queue if blocked.
        }else{
            // Inserted within the middle of the queue.
        }
        
        if(needWake) nativeWake(mPrt);
    }
}
```

MessageQueue  next()

```java
Message next(){
    
}
```



### Message.obtain() 和Handler.obtainMessage()的区别

> **这两种方式都比直接`new`一个`Message`对象在性能上更优越.**

### Hander中removeMessages方法

1、这个方法使用的前提是之前调用过sendEmptyMessageDelayed(0, time)，意思是延迟time执行handler中msg.what=0的方法；

2、在延迟时间未到的前提下，执行removeMessages(0)，则上面的handler中msg.what=0的方法取消执行；

3、在延迟时间已到，handler中msg.what=0的方法已执行，再执行removeMessages(0)，不起作用。

4、该方法会将handler对应message队列里的消息清空，通过msg.what来找到对应的message。

5、当队列中没有message则handler会不工作，但并不是handler会停止，当队列中有新的message进来后，会继续处理执行。

关于IdleHandler:

> 经过测试发现, idelHandler返回为true的时候,也不一定会一直执行, 当app停止一段时间没操作了, 不会一直重复执行.