# Looper相关

* ANR是怎么产生的
* Looper的工作机制是什么
* Looper不会导致ANR的本质原因
* Looper为什么不会导致CPU占用率高

### ANR是怎么产生的

* Service Timeout:
  * 前台服务20s
  * 后台服务200s
* BroadcaseQueue Timeout:

  * 前台广播10s

  * 后台广播60s
* ContentProvider Timeout: 10s
* InputDispatching Timeout: 5s

比如在ActiveServices中有一个 `scheduleServiceForegroundTransitionTimeoutLocked` 这里会开始计算需要的时间

ANR是对开发者写的程序对占用主线程的耗时过多提醒(起监控作用).

### Looper的工作机制是什么





### Looper不会导致ANR的本质原因

Looper是对进程的概念, 而ANR是作用在某个阶段的监控作用,Looper范围很大, ANR是其子集.



### Looper为什么不会导致CPU占用率高

底层epoll_wait会阻塞, 让出CPU的控制权, 等到了有消息的时候, 被系统中断重新唤醒.