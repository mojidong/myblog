---
title: "java增强型Thread Local, InheritableThreadLocal"
date: 2019-02-24T10:28:30+08:00
lastmod: 2019-02-24T10:28:30+08:00
description: "java增强型thread local, InheritableThreadLocal"
categories: ["java"]
tag: ["java"]
---

最近使用`InheritableThreadLocal`遇到一些问题，这里总结一下。

## Thread Local

**`ThreadLocal`是绑定到线程上的上下文，他可以让你很方便的在同一个线程里进行上下文数据传递和共享，线程与线程之间的ThreadLocal是相互不可见的。**

下面是一个简单的例子：

```java
public class Test {


    public static void main(String[] args) throws Exception {

        AService aService = new AService();
        BService bService = new BService();

        new Thread(() -> {

            // 初始化线程上下文
            ThreadLocalContext.initContext();

            // 操作上下文
            aService.genNum();

            // 获取上下文信息
            bService.showNum();
        }).start();

        TimeUnit.SECONDS.sleep(5);
    }

    /**
     * 共享上下文
     */
    static class Context {

        private Integer num;

        public Integer getNum() {
            return num;
        }

        public void setNum(Integer num) {
            this.num = num;
        }
    }

    /**
     * 线程上下文
     */
    static class ThreadLocalContext {

        public static ThreadLocal<Context> threadLocal;

        public static void initContext() {
            threadLocal = new ThreadLocal<>();
        }

        public static void putContext(Context context) {
            threadLocal.set(context);
        }

        public static Context getContext() {
            return threadLocal.get();
        }
    }


    static class AService {

        public void genNum() {
            Context context = ThreadLocalContext.getContext();
            if (context == null) {
                context = new Context();
                context.setNum(0);
            }

            context.setNum(context.getNum() + 1);
            ThreadLocalContext.putContext(context);
        }
    }

    static class BService {
        public void showNum() {
            System.out.println(ThreadLocalContext.getContext().getNum());
        }
    }
}
```

从上面可以看到在多个`service`中通过Thread
Local传递共享数据非常便利，不用显示的在现有`service`进行显示传递。
这在一些场景中非常有用，例如对老旧系统的功能修改，可以做到不破坏原有服务接口签名。
还有全链路埋点（分析统计、监控）可以做到解藕。

## InheritableThreadLocal

简单来讲`InheritableThreadLocal`是`ThreadLocal`的增强版，它带了一项新的能力
，**子线程会拷贝父线程中`ThreadLocal`**

之前用`ThreadLocal`我们都是这样做的：

```java
public class Test {


    public static void main(String[] args) throws Exception {

        AService aService = new AService();
        BService bService = new BService();
        CService cService = new CService();

        // 初始化主线程上下文
        ThreadLocalContext.initContext();

        // 主线程中的逻辑，会操作主线程上下文
        cService.setNum();

        // 获取主线程中的上下文
        Context context = ThreadLocalContext.getContext();

        new Thread(() -> {

            // 子线上下文直接用主线程中的上下文创建
            ThreadLocalContext.setContext(context);

            // 操作上下文
            aService.genNum();

            // 获取上下文信息
            bService.showNum();
        }).start();

        TimeUnit.SECONDS.sleep(5);
    }

    /**
     * 共享上下文
     */
    static class Context {

        private Integer num;

        public Integer getNum() {
            return num;
        }

        public void setNum(Integer num) {
            this.num = num;
        }
    }

    /**
     * 线程上下文
     */
    static class ThreadLocalContext {

        public static ThreadLocal<Context> threadLocal;

        public static void initContext() {
            threadLocal = new ThreadLocal<>();
        }

        public static void setContext(Context context) {
            threadLocal = new ThreadLocal<>();
            threadLocal.set(context);
        }

        public static void putContext(Context context) {
            threadLocal.set(context);
        }


        public static Context getContext() {
            return threadLocal.get();
        }
    }


    static class AService {

        public void genNum() {
            Context context = ThreadLocalContext.getContext();
            if (context == null) {
                context = new Context();
                context.setNum(0);
            }

            context.setNum(context.getNum() + 1);
            ThreadLocalContext.putContext(context);
        }
    }

    static class BService {
        public void showNum() {
            System.out.println(ThreadLocalContext.getContext().getNum());
        }
    }

    static class CService {
        public void setNum() {
            Context context = new Context();
            context.setNum(1);
            ThreadLocalContext.putContext(context);
        }
    }

}
```

需要自已手动将主线程中的上下文传递到子线程中的`ThreadLocal`中。

---

下面我们来看看使用`InheritableThreadLocal`
```java
public class Test {


    public static void main(String[] args) throws Exception {

        AService aService = new AService();
        BService bService = new BService();
        CService cService = new CService();

        // 初始化主线程上下文
        ThreadLocalContext.initContext();

        // 主线程中的逻辑，会操作主线程上下文
        InheritableThreadLocal     cService.setNum();

        // 获取主线程中的上下文
        // Context context = ThreadLocalContext.getContext();

        new Thread(() -> {

            // 子线上下文直接用主线程中的上下文创建
            // ThreadLocalContext.setContext(context);

            // 操作上下文
            aService.genNum();

            // 获取上下文信息
            bService.showNum();
        }).start();

        TimeUnit.SECONDS.sleep(5);
    }

    /**
     * 共享上下文
     */
    static class Context {

        private Integer num;

        public Integer getNum() {
            return num;
        }

        public void setNum(Integer num) {
            this.num = num;
        }
    }

    /**
     * 线程上下文
     */
    static class ThreadLocalContext {

        public static InheritableThreadLocal<Context> threadLocal;

        public static void initContext() {
            threadLocal = new InheritableThreadLocal<>();
        }


        public static void setContext(Context context) {
            threadLocal = new InheritableThreadLocal<>();
            threadLocal.set(context);
        }

        public static void putContext(Context context) {
            threadLocal.set(context);
        }


        public static Context getContext() {
            return threadLocal.get();
        }
    }


    static class AService {

        public void genNum() {
            Context context = ThreadLocalContext.getContext();
            if (context == null) {
                context = new Context();
                context.setNum(0);
            }

            context.setNum(context.getNum() + 1);
            ThreadLocalContext.putContext(context);
        }
    }

    static class BService {
        public void showNum() {
            System.out.println(ThreadLocalContext.getContext().getNum());
        }
    }

    static class CService {
        public void setNum() {
            Context context = new Context();
            context.setNum(1);
            ThreadLocalContext.putContext(context);
        }
    }

}
```

可以看到主动转递上下文的过程省去了，`InheritableThreadLocal`会自动搞定。

## 问题
`InheritableThreadLocal`看起来很美，但实际使用下来发现了几个问题：

1. 拷贝`ThreadLocal`是在子线程创建的时候，所以线程池中`ThreadLocal`后续不拷贝主线程的`ThreadLocal`，因为线程池中大多数情况下线程是复用的，重新建的情况很少。
2. 当初创建子线程的主线程如果清除`ThreadLocal`子线程中的`ThreadLocal`也会被清除。

下面是验证代码：

```java
public class Test {

    static ExecutorService executorService = Executors.newFixedThreadPool(1);

    static AService aService = new AService();
    static BService bService = new BService();
    static CService cService = new CService();


    public static void main(String[] args) throws Throwable {

        // 第一次执行
        test();
        TimeUnit.SECONDS.sleep(1);
        
        // 第二次执行
        test();
        TimeUnit.SECONDS.sleep(1);

    }

    private static void test() {
        
        new Thread(() -> {

            // 初始化主线程上下文
            ThreadLocalContext.initContext();

            // 主线程中的逻辑，会操作主线程上下文
            cService.setNum();

            executorService.submit(() -> {


                // 操作上下文
                aService.genNum();

                // 获取上下文信息
                bService.showNum();
            });

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
            }

            ThreadLocalContext.clear();


        }).start();

    }

    /**
     * 共享上下文
     */
    static class Context {

        private Integer num;

        public Integer getNum() {
            return num;
        }

        public void setNum(Integer num) {
            this.num = num;
        }
    }

    /**
     * 线程上下文
     */
    static class ThreadLocalContext {

        public static InheritableThreadLocal<Context> threadLocal;

        public static void initContext() {
            threadLocal = new InheritableThreadLocal<>();
        }


        public static void setContext(Context context) {
            threadLocal = new InheritableThreadLocal<>();
            threadLocal.set(context);
        }

        public static void putContext(Context context) {
            threadLocal.set(context);
        }


        public static Context getContext() {
            return threadLocal.get();
        }

        public static void clear() {
            threadLocal.remove();
        }
    }


    static class AService {

        public void genNum() {
            Context context = ThreadLocalContext.getContext();
            if (context == null) {
                context = new Context();
                context.setNum(0);
            }

            context.setNum(context.getNum() + 1);
            ThreadLocalContext.putContext(context);
        }
    }

    static class BService {
        public void showNum() {
            System.out.println(ThreadLocalContext.getContext().getNum());
        }
    }

    static class CService {
        public void setNum() {
            Context context = new Context();
            context.setNum(1);
            ThreadLocalContext.putContext(context);
        }
    }

}
```

## 总结
1. 总是优先使用`ThreadLocal`。
2. 在使用线程池的情况下应避免使用`InheritableThreadLocal`。
