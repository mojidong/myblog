---
title: "thread pool中使用thread local的问题"
date: 2019-03-02T20:58:56+08:00
lastmod: 2019-03-02T20:58:56+08:00
categories: ["java"]
tags: ["java", "thread pool"]
---

最近发现生产环境一个十分诡异的问题，这里与大家分享一下。

## 问题

最近发现线上服务报错，报错是偶发的，会自动恢复。主体代码如下:

```java
/**
 * Created by mojidong.
 */
public class Test {

    /**
     * 初始化线程池
     */
    private static final ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 10, 1, TimeUnit.MINUTES,
            new LinkedBlockingQueue<>(128), (r, executor) -> r.run());


    public static void main(String[] args) throws Throwable {

        // 初始上下文
        BizContext.init();
        // 放入信息到上下文
        BizContext.putInfo("mainInfo", "main good");

        // 取出内部上下文用于下面投传到子线程中
        BizContext.InnerContext innerContext = BizContext.getContext();

        List<Callable<Object>>  callables = new ArrayList();
        for (int i = 0; i < 150; i++) {

            int taskCount = i;

            callables.add(() -> {

                // 子线程上下文用主线程内部上下文初始化
                BizContext.setContext(innerContext);

                // 放下信息到上下文中
                BizContext.putInfo("taskCount" + taskCount, taskCount);

                // 清除子线程上下文
                BizContext.clearContext();

                try {
                    TimeUnit.MICROSECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                return null;
            });
        }

        threadPool.invokeAll(callables);

        // 输出主线程上下文信息
        System.out.println("main thread context: " + BizContext.getContext());

        System.out.println(BizContext.getContext().get("mainInfo"));

        // 清除主线程上下文信息
        BizContext.clearContext();

        threadPool.shutdown();

    }


    /**
     * 业务上下文
     */
    static class BizContext {

        private static ThreadLocal<InnerContext> threadLocal = new ThreadLocal();

        public static void init() {
            threadLocal.set(new InnerContext());
        }

        public static void setContext(InnerContext innerContext) {
            threadLocal.set(innerContext);
        }

        public static InnerContext getContext() {
            return threadLocal.get();
        }

        public static void clearContext() {
            threadLocal.remove();
        }

        public static void putInfo(String key, Object object) {
            threadLocal.get().put(key, object);
        }

        public static Object getInfo(String key) {
            return threadLocal.get().get(key);
        }


        /**
         * 内部上下文
         */
        static class InnerContext {

            private Map<String, Object> map = new ConcurrentHashMap<>();

            public void put(String key, Object object) {
                map.put(key, object);
            }

            public Object get(String key) {
                return map.get(key);
            }

            @Override
            public String toString() {
                return map.toString();
            }
        }
    }
}
```

逻辑比较清淅，主要是使用`ThreadLocal`将上下文在主线程和子线程中传递。
通过排查发现回到主线程后上下文被清空了，后续又有操作上下文的代码，所以抛了NPE。

```java
// 这里输出是null
// 输出主线程上下文信息
System.out.println("main thread context: " + BizContext.getContext());
```

通过review代码并没有发现问题所在，主要是问题是偶发，并且之前很长的一段时间都没有问题，
最近也未对这块代码作变更，最诡异的是它会随着时间的推移自动恢复，真是百思不得其解。

## 根源

通过对线上数据分析，发现任务数比较多的情况下大概率会出现该问题，也就是放入线程池中的任务多才会有问题。
到了这里突然灵光乍现，肯定与任务队列溢出有关，立即查看线程池配置代码。

```java
 /**
 * 初始化线程池
 */
private static final ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 10, 1, TimeUnit.MINUTES,
        new LinkedBlockingQueue<>(128), (r, executor) -> r.run());
```

任务队列是`128`，超过之后会触发`RejectedExecutionHandler`，这里的处理方式是：

```java
(r, executor) -> r.run()
```

这个实际是直接在主线程中执行任务代码。到这里问题终于真相大白，由于任务在主线程中执行，
所以任务本应该在子线程中清空上下文的操作实际是在主线程中执行了。

```java
() -> {
    // 子线程上下文用主线程内部上下文初始化
    BizContext.setContext(innerContext);

    // 放下信息到上下文中
    BizContext.putInfo("taskCount" + taskCount, taskCount);

    // 清除子线程上下文
    // 由于在主线程中执行实际是清空了主线程上下文
    BizContext.clearContext();

    try {
        TimeUnit.MICROSECONDS.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
})
```

所以上面代码`BizContext.clearContext();`实际是在主线程中执行的，所以主线程上下文就被清空了。

## 方案
知道了原因就很好处理：

1. 去掉自定义的`RejectedExecutionHandler`，默的`RejectedExecutionHandler`是抛异常。
2. 让任务不超过溢出队列长度。

按照上面的2条代码调整如下：

```java
/**
 * Created by mojidong.
 */
public class Test {

    /**
     * 初始化线程池
     */
    private static final ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 10, 1, TimeUnit.MINUTES,
            new LinkedBlockingQueue<>(128));


    public static void main(String[] args) throws Throwable {

        // 初始上下文
        BizContext.init();
        // 放入信息到上下文
        BizContext.putInfo("mainInfo", "main good");

        // 取出内部上下文用于下面投传到子线程中
        BizContext.InnerContext innerContext = BizContext.getContext();

        List<Callable<Object>>  callables = new ArrayList();
        for (int i = 0; i < 150; i++) {

            int taskCount = i;

            callables.add(() -> {

                // 子线程上下文用主线程内部上下文初始化
                BizContext.setContext(innerContext);

                // 放下信息到上下文中
                BizContext.putInfo("taskCount" + taskCount, taskCount);

                // 清除子线程上下文
                BizContext.clearContext();

                try {
                    TimeUnit.MICROSECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                return null;
            });
        }

        for(List<Callable<Object>> subCallables : Lists.partition(callables, 128)) {
            threadPool.invokeAll(subCallables);
        }


        // 输出主线程上下文信息
        System.out.println("main thread context: " + BizContext.getContext());

        System.out.println(BizContext.getContext().get("mainInfo"));


        BizContext.clearContext();

        threadPool.shutdown();

    }


    /**
     * 业务上下文
     */
    static class BizContext {

        private static ThreadLocal<InnerContext> threadLocal = new ThreadLocal();

        public static void init() {
            threadLocal.set(new InnerContext());
        }

        public static void setContext(InnerContext innerContext) {
            threadLocal.set(innerContext);
        }

        public static InnerContext getContext() {
            return threadLocal.get();
        }

        public static void clearContext() {
            threadLocal.remove();
        }

        public static void putInfo(String key, Object object) {
            threadLocal.get().put(key, object);
        }

        public static Object getInfo(String key) {
            return threadLocal.get().get(key);
        }


        /**
         * 内部上下文
         */
        static class InnerContext {

            private Map<String, Object> map = new ConcurrentHashMap<>();

            public void put(String key, Object object) {
                map.put(key, object);
            }

            public Object get(String key) {
                return map.get(key);
            }

            @Override
            public String toString() {
                return map.toString();
            }
        }
    }
}
```

主要是两处改动点

1.

```java
/**
 * 初始化线程池
 */
private static final ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 10, 1, TimeUnit.MINUTES,
        new LinkedBlockingQueue<>(128));
```

2.
```java
for(List<Callable<Object>> subCallables : Lists.partition(callables, 128)) {
    threadPool.invokeAll(subCallables);
}
```


## 总结
1. 不使建使用在主线程执行的`RejectedExecutionHandler`即`(r, executor) -> r.run()`。
2. 超出溢出队列可以切割任务来避免。




