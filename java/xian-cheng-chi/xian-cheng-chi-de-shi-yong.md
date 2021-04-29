# 线程池的使用

### 线程的使用

{% tabs %}
{% tab title="single" %}
```text
public class Main1 {
    public static void main(String[] args) {
        Thread thread = new Thread(new Task());
        thread.start();
        System.out.println("Thread name  1: " + Thread.currentThread().getName());
    }

    private static class Task implements Runnable {
        @Override
        public void run() {
            System.out.println("thread name  2: " + Thread.currentThread().getName());
        }
    }
}
```
{% endtab %}

{% tab title="1..10" %}
```text
public class Main2 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new Task());
            thread.start();
        }
        System.out.println("Main Thread name: " + Thread.currentThread().getName());
    }

    private static class Task implements Runnable {
        @Override
        public void run() {
            System.out.println("Task Thread name: " + Thread.currentThread().getName());
        }
    }
}
```
{% endtab %}
{% endtabs %}

### newFixedThreadPool

{% tabs %}
{% tab title="普通使用" %}
```text
public class Main3 {
    public static void main(String[] args) {
        // creat the pool
        ExecutorService service = Executors.newFixedThreadPool(10);

        // submit the tasks for execution
        for (int i = 0; i < 100; i++) {
            service.execute(new Task());
        }
        System.out.println("main Thread Name: " + Thread.currentThread().getName());
    }

    private static class Task implements Runnable {
        @Override
        public void run() {
            System.out.println("Task Thread Name: " + Thread.currentThread().getName());
        }
    }
}
```
{% endtab %}

{% tab title="计算密集型" %}
```text
public class Main4 {
    public static void main(String[] args) {

        // get count of available cores
        int coreCount = Runtime.getRuntime().availableProcessors();
        ExecutorService service = Executors.newFixedThreadPool(coreCount);

        // submit the tasks for execution
        for (int i = 0; i < 100; i++) {
            service.execute(new CpuIntensiveTask());
        }
    }

    private static class CpuIntensiveTask implements Runnable {

        @Override
        public void run() {
            // some CPU intensive operations
            System.out.println("Task Thread Name: " + Thread.currentThread().getName());
        }
    }
}
```
{% endtab %}

{% tab title="IO密集型" %}
```
public class Main5 {
    public static void main(String[] args) {
        
        // must higher count for IO Tasks
        ExecutorService service = Executors.newFixedThreadPool(100);
        
        // submit the tasks for execution
        for (int i = 0; i < 100; i++) {
            service.execute(new IOTask());
        }
    }

    private static class IOTask implements Runnable {

        @Override
        public void run() {
            // some IO operations which will cause thread to block/wait
        }
    }
}

```
{% endtab %}
{% endtabs %}

### CachedThreadPool

```text
public class Main6 {
    public static void main(String[] args) {

        // for lot of short lived tasks
        ExecutorService service = Executors.newCachedThreadPool();

        // submit the tasks for execution
        for (int i = 0; i < 100; i++) {
            service.execute(new Task());
        }
    }

    private static class Task implements Runnable {
        @Override
        public void run() {
            System.out.println("Task Thread Name: " + Thread.currentThread().getName());
        }
    }
}

```

### ScheduledThreadPool

```text
public class Main7 {
    public static void main(String[] args) {

        // for scheduling of tasks
        ScheduledExecutorService service = Executors.newScheduledThreadPool(10);

        // task to run after 10 second delay
        service.schedule(new Task(), 10, TimeUnit.SECONDS);

        // task to run repeatedly every 10 seconds
        service.scheduleAtFixedRate(new Task(), 15, 10, TimeUnit.SECONDS);

        // task to run repeatedly 10 seconds after previous task completes
        service.scheduleWithFixedDelay(new Task(), 15, 10, TimeUnit.SECONDS);
    }

    private static class Task implements Runnable {
        @Override
        public void run() {
            // task that needs to run
            // based on schedule
            System.out.println("Task Thread Name: " + Thread.currentThread().getName());
        }
    }
}

// scheduleWithFixedDelay 最大的区别就是 ，scheduleAtFixedRate  不受任务执行时间的影响。


```

{% hint style="warning" %}
对于ScheduleAtFixedRate

当**task**的执行时间\(比如:4s\)大于**period\(2s\)**的时候, 2个task的**时间间隔**将不是period\(2s\)的值,而是任务时间执行的时间\(4s\).

如果task的执行时间\(1s\)小于**period\(2s\)**,那个2个task的时间间隔将为period的时间\(2s\).

对于ScheduleWithFixedDelay: **执行时间 + period**
{% endhint %}

### SingleThreadedExecutor

{% hint style="warning" %}
**Pool size depends on the task type**
{% endhint %}

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Task Type</b>
      </th>
      <th style="text-align:left">Ideal pool size</th>
      <th style="text-align:left">Considerations</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">CPU Intensive</td>
      <td style="text-align:left">CPU Core count</td>
      <td style="text-align:left">How many other applications(or other executors/threads)</td>
    </tr>
    <tr>
      <td style="text-align:left">IO Intensive</td>
      <td style="text-align:left">High</td>
      <td style="text-align:left">
        <p>Exact number will depend on rate of task submissions and</p>
        <p>average task wait time.</p>
        <p></p>
        <p>Too many threads will increase memory consumption too.</p>
      </td>
    </tr>
  </tbody>
</table>

\*\*\*\*



