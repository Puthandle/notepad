# Java线程总结

## 前置了解

什么是进程？

> 狭义定义：进程是正在运行的程序的实例。 比如我们 **ps -ef** 出来的pid，代表的就是进程ID。
>
> 广义定义：进程是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。它是操作系统]动态执行的基本单元，在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元。

什么是线程？

> 一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。在Unix System V及SunOS中也被称为轻量进程（lightweight processes），但轻量进程更多指内核线程（kernel thread），而把用户线程（user thread）称为线程。

什么是并行？

> 在操作系统中是指，一组程序按独立异步的速度执行，无论从微观还是宏观，程序都是一起执行的 

什么是并发？

> 并发，在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机上运行，但任一个时刻点上只有一个程序在处理机上运行

## 线程的概念。

### 定义

> ### **线程**（英语：thread）是操作系统能够进行**运算调度的最小单位**。它被包含在进程之中，是**进程**中的实际运作单位。

### 线程的状态机

```java
    public enum State {
        // 新建状态
        NEW,
        
        // 运行状态，表示当前线程正在执行
        RUNNABLE,

        // 阻塞状态
        BLOCKED,

        // 等待状态
        WAITING,

        // 等待状态，有时间限制
        TIMED_WAITING,

  		// 终止状态，表示前线程执行完毕
        TERMINATED;
    }
```



## 创建线程的几种方式

### 继承Thread类

```java
public class ThreadImplement extends Thread {
    CountDownLatch countDownLatch;
    public ThreadImplement(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }
  
    @Override
    public void run() {
        try {
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + ":my thread ");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            countDownLatch.countDown();
        }
    }
    public static void main(String[] args) {
        // 第一种：使用extends Thread方式
        CountDownLatch countDownLatch = new CountDownLatch(2);
        for (int i = 0; i < 2; i++) {
            ThreadImplement threadImplement = new ThreadImplement(countDownLatch);
            threadImplement.start();
        }
        try {
            countDownLatch.await();
            System.out.println("thread complete...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



### 继承Runnable接口

```java
public class RunnableImplement  implements Runnable{
    CountDownLatch countDownLatch;
  
    public RunnableImplement(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }
    @Override
    public void run() {
        try {
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + ":my runnable ");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            countDownLatch.countDown();
        }
    }
  
    public static void main(String[] args) {
        // 第二种：使用implements Runnable方式
        CountDownLatch countDownLatch = new CountDownLatch(2);
        RunnableImplement runnableImplement = new RunnableImplement(countDownLatch);
        for (int i = 0; i < 2; i++) {
            new Thread(runnableImplement).start();
        }
        try {
            countDownLatch.await();
            System.out.println("runnable complete...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



### 继承Callable接口

```java
public class CallableImplement implements Callable<Integer> {
  
    public static void main(String[] args) {
        CallableImplement callableImplement = new CallableImplement();
        //1、用futureTask接收结果
        FutureTask<Integer> futureTask = new FutureTask<>(callableImplement);
        new Thread(futureTask).start();
        //2、接收线程运算后的结果
        try {
            //futureTask.get();这个是堵塞性的等待
            Integer sum = futureTask.get();
            System.out.println("sum="+sum);
            System.out.println("-------------------");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
  
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i <101 ; i++) {
            sum+=i;
        }
        return sum;
    }
}
```

### 使用CompletableFuture

```java
public class CompletableFutureImplement {
  
    /**
     * A任务B任务完成后，才执行C任务
     * 返回值的处理
     * @param
     *@return void
     **/
    @Test
    public void completableFuture(){
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("future1 finished!");
            return "future1 finished!";
        });
  
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            System.out.println("future2 finished!");
            return "future2 finished!";
        });
  
        CompletableFuture<Void> future3 = CompletableFuture.allOf(future1, future2);
        try {
            future3.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println("future1: " + future1.isDone() + " future2: " + future2.isDone());
  
    }
  
    /**
     * 在Java8中，CompletableFuture提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，
     * 并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合 CompletableFuture 的方法
     *
     *  注意: 方法中有Async一般表示另起一个线程,没有表示用当前线程
     */
    @Test
    public void test01() throws Exception {
        ExecutorService service = Executors.newFixedThreadPool(5);
        /**
         *  supplyAsync用于有返回值的任务，
         *  runAsync则用于没有返回值的任务
         *  Executor参数可以手动指定线程池，否则默认ForkJoinPool.commonPool()系统级公共线程池
         */
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "xiaoxuzhu";
        }, service);
        CompletableFuture<Void> data = CompletableFuture.runAsync(() -> System.out.println("xiaoxuzhu"));
        /**
         * 计算结果完成回调
         */
        future.whenComplete((x,y)-> System.out.println("有延迟3秒：执行当前任务的线程继续执行:"+x+","+y)); //执行当前任务的线程继续执行
        data.whenCompleteAsync((x,y)-> System.out.println("交给线程池另起线程执行:"+x+","+y)); // 交给线程池另起线程执行
        future.exceptionally(Throwable::toString);
        //System.out.println(future.get());
        /**
         * thenApply,一个线程依赖另一个线程可以使用,出现异常不执行
         */
        //第二个线程依赖第一个的结果
        CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 5).thenApply(x -> x);
  
        /**
         * handle 是执行任务完成时对结果的处理,第一个出现异常继续执行
         */
        CompletableFuture<Integer> future2 = future1.handleAsync((x, y) -> x + 2);
        System.out.println(future2.get());//7
        /**
         * thenAccept 消费处理结果,不返回
         */
        future2.thenAccept(System.out::println);
        /**
         * thenRun  不关心任务的处理结果。只要上面的任务执行完成，就开始执行
         */
        future2.thenRunAsync(()-> System.out.println("继续下一个任务"));
        /**
         * thenCombine 会把 两个 CompletionStage 的任务都执行完成后,两个任务的结果交给 thenCombine 来处理
         */
        CompletableFuture<Integer> future3 = future1.thenCombine(future2, Integer::sum);
        System.out.println(future3.get()); // 5+7=12
        /**
         * thenAcceptBoth : 当两个CompletionStage都执行完成后，把结果一块交给thenAcceptBoth来进行消耗
         */
        future1.thenAcceptBothAsync(future2,(x,y)-> System.out.println(x+","+y)); //5,7
        /**
         * applyToEither
         * 两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的转化操作
         */
        CompletableFuture<Integer> future4 = future1.applyToEither(future2, x -> x);
        System.out.println(future4.get()); //5
        /**
         * acceptEither
         * 两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的消耗操作
         */
        future1.acceptEither(future2, System.out::println);
        /**
         * runAfterEither
         * 两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable
         */
        future1.runAfterEither(future,()-> System.out.println("有一个完成了,我继续"));
        /**
         * runAfterBoth
         * 两个CompletionStage，都完成了计算才会执行下一步的操作（Runnable）
         */
        future1.runAfterBoth(future,()-> System.out.println("都完成了,我继续"));
        /**
         * thenCompose 方法
         * thenCompose 方法允许你对多个 CompletionStage 进行流水线操作，第一个操作完成时，将其结果作为参数传递给第二个操作
         * thenApply是接受一个函数,thenCompose是接受一个future实例,更适合处理流操作
         */
        future1.thenComposeAsync(x->CompletableFuture.supplyAsync(()->x+1))
                .thenComposeAsync(x->CompletableFuture.supplyAsync(()->x+2))
                .thenCompose(x->CompletableFuture.runAsync(()-> System.out.println("流操作结果:"+x)));
        TimeUnit.SECONDS.sleep(5);//主线程sleep,等待其他线程执行
    }
}
```



### 使用线程池

```java
public class ExecutorServiceImplement {
  
    static  class CallableImplement implements Callable<Integer> {
        private CountDownLatch countDownLatch;
  
        public CallableImplement(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }
  
        public Integer call() {
            int sum = 0;
            try {
                for (int i = 0; i <= 100; i++) {
                    sum += i;
                }
                System.out.println("线程执行结果："+sum);
                 
            } finally {
                countDownLatch.countDown();
            }
            return sum;
        }
    }
  
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 第五种：使用使用线程池方式
        // 接受返回参数
        List<Future> futures = new ArrayList<Future>();
        // 給线程池初始化5個线程
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        CountDownLatch countDownLatch = new CountDownLatch(10);
  
        for (int i = 0; i < 10; i++) {
            CallableImplement callableImplement = new CallableImplement(countDownLatch4);
            Future result = executorService.submit(callableImplement);
            futures.add(result);
        }
        // 等待线程池中分配的任务完成后才关闭(关闭之后不允许有新的线程加入，但是它并不会等待线程结束)，
        // 而executorService.shutdownNow();是立即关闭不管是否线程池中是否有其他未完成的线程。
        executorService.shutdown();
        try {
            countDownLatch.await();
            Iterator<Future> iterator = resultItems2.iterator();
            System.out.println("----------------------");
            while (iterator.hasNext()) {
                try {
                    System.out.println("线程返回结果："+iterator.next().get());
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("callable complete...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



## 聊一聊线程池

### 概念

>   线程池就是首先创建一些线程，它们的集合称为线程池。使用线程池可以很好地提高性能，线程池在系统启动时即创建大量空闲的线程，程序将一个任务传给线程池，线程池就会启动一条线程来执行这个任务，执行结束以后，该线程并不会死亡，而是再次返回线程池中成为空闲状态，等待执行下一个任务

### 线程池的继承结构

#### 根接口 

```java
public interface Executor {
    // 可执行的Runnable对象
    void execute(Runnable command);
}
```

### 对外暴露的服务接口

```java
public interface ExecutorService extends Executor {

	// 调用此方法，会等到所有任务都提交了, 才会关闭线程池
    void shutdown();

    // 立即关闭关闭线程池
    List<Runnable> shutdownNow();

    // 是否已经关闭
    boolean isShutdown();

	// 是否已经终止
    boolean isTerminated();

    // 等待终止，调用此方法 主线程会处于一种等待的状态，等待线程池中所有的线程都运行完毕后才继续运行。
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

	// 向线程池中提交任务
    <T> Future<T> submit(Callable<T> task);

	// 向线程池中提交任务
    <T> Future<T> submit(Runnable task, T result);

	// 向线程池中提交任务
    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;


    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;


    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

### 抽象的实现 AbstractExecutorService 

> 代码有点多.这里就不贴了。

### 执行流程

> 从 ExecutorService 的 submit 说起。

```java
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    // 简单包装一下，调用 execute 方法。
    execute(ftask);
    return ftask;
}
```

> execute 的 具体实现位于 ThreadPoolExecutor 中

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    // 看上面的注释，分3步处理
    // 1. 小于核心线程数，加入队列处理
    // 2. 任务可以被加入队列, 加入队列处理
    // 3. 队列数量已满，会尝试新建线程，不成功就拒绝
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

> addWorker 是怎么做的！

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                // 调用线程 start 方法，开始处理，业务逻辑。
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

