











# 手写Handler-Looper



> 思考:
>
> 为什么Android单独实现这一套逻辑

### Handler拥有的核心能力

* 线程间通信
* 延时任务执行



### Handler

```java
// 伪代码
public class Handler{
    private MessageQueue mQueue;
    
    public void sendMessage(Message msg){
        ...
    }
    
    public void sendMessageDelayed(Message msg, long delayMills){
        msg.target = this;
        delay = Math.max(delay, 0L);
        msg.when = System.currentTimeMills() + delay;
        messageQueue.enqueueMessage(msg);
    }
    
    public void post(@NonNull Runnable r){
        
    }
    
    public void postDelayed(@NonNull Runable r, long delayMills){
        Message msg = new Message();
        msg.callback = r;
        sendMessageDelayed(msg, delay);
    }
}
```



### Looper

```java
public final class Looper{
    private static final ThreadLocal<Looper> THREAD_LOCAL = new ThreadLocal<Looper>();
    private MessageQueue messageQueue;
    
    public static void prepare(){
        if(THREAD_LOCAL.get() != null){
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        THREAD_LOCAL.set(new Looper());
    }
    
    public static Looper myLooper(){
        return THREAD_LOCAL.get();
    }
    
    public MessageQueue getQueue(){ return messageQueue;}
    
    public static void loop(){
        Looper me = myLooper();
        if(me == null){ thread new RuntimeException("No Looper")};
        while(true){
            MessageQueue msg = me.messageQueue.next();
            if(msg == null) continue;
            msg.target.dispatchMessage(msg);
        }
    }
    
    public void quit(){ messageQueue.quit();}
}
```



### MessageQueue

* 持有消息
* 消息按时间排序(优先级)
* 队列为空时阻塞读取
* 头节点有延时可以定时阻塞

```java
public class MessageQueue{
    private DelayQueue<Message> queue = new DelayQueue<>();
    public void enqueueMessage(Message msg){
        queue.add(msg);
    }
    
    public Message next(){
        try{
            return queue.take();
        }catch(InterruptedException e){
            ...
        }
    }
    
    public void quit(){ queue.clear();}
}
```



### Android为什么不用DelayQueue

* DelayQueue实现中很多地方都用到了锁, 效率比较低.

* Android的MQ可以针对单线程读取的场景进行优化

  > MQ 的读取都是通过looper是单线程行为, 不需要考虑多线程的场景, 但是DelayQueue是针对多线程的, 所以MQ的读取效率比DelayQueue高



