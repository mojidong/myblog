---
title: "java排序违返规约错误"
date: 2019-02-23T20:33:39+08:00
lastmod: 2019-02-23T20:33:39+08:00
description: "java排序违返规约错误"
categories: ["java"]
tag: ["java"]
---

## 问题

最近发现线上一段大半年都没有变动的代码报错。
错误栈如下：

```java
Exception in thread "main" java.lang.IllegalArgumentException: Comparison method violates its general contract!
	at java.util.TimSort.mergeHi(TimSort.java:899)
	at java.util.TimSort.mergeAt(TimSort.java:516)
	at java.util.TimSort.mergeCollapse(TimSort.java:441)
	at java.util.TimSort.sort(TimSort.java:245)
	at java.util.Arrays.sort(Arrays.java:1438)
	at java.util.Arrays$ArrayList.sort(Arrays.java:3895)
```

通过日志定位代码发现为排序报错

```java
list.sort(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1 > o2 ? 1 : -1;
    }
});
```

很奇怪这代码为什么会报错，之前的排序好像都是这样写的并没有问题。

## 原因

根据日志错误信息`Comparison method violates its general
contract!`通过google查询发现是违反了排序规约，原来从java7开始针对排序进行了更严格的限制。
![](/images/java_sort_contract.png)

1. sgn(compare(x, y)) == -sgn(compare(y, x))
2. ((compare(x, y)>0) && (compare(y, z)>0)) implies compare(x, z)>0
3. compare(x, y)==0 implies that sgn(compare(x, z))==sgn(compare(y, z))

**简单来说就是违反了自反性和传递性**。

到这里问题的原因基本定位了，前不久将应用jdk6升级到了8,
那为什么大半年了才暴露出来。

**扒了一下源码发现只有在排序元素大于32才会有机率出现此错误。**

下面是一段测试代码:

```java
public static void main(String[] args) throws Exception {
    while (true) {
        List<Integer> list = new ArrayList<>();
        while (list.size() < 32) {
            list.add(new Random().nextInt(100));
        }

        System.out.println("list:" + list);

        list.sort(new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o1 > o2 ? 1 : -1;
            }
        });
    }
}
```

## 解决方案

### 1. 实现规约的限制
拿开的代码头来说，`o1 == o2`和`o1 = null, o2 = null`的case没考虑，所以导致违反了自反性和传递性。正确的代码如下：

```java
list.sort(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        
        if(o1 == null && o2 == null) {
            return 0;
        }
        
        if(o1 == null) {
            return -1;
        }
        
        if(o2 == null) {
            return 1;
        }
        
        if(o1 == o2) {
            return 0;
        }
        
        return o1 > o2 ? 1 : -1;
    }
});
```

### 2. 降级排序算法为java7之前的版本
1. 代码里设置系统变量降级
```java
System.setProperty("java.util.Arrays.useLegacyMergeSort", "true"); 
```

2. jvm启动参数增加降级参数
```bash
-Djava.util.Arrays.useLegacyMergeSort=true
```

> 可根据自身的实际情况来选则降级方案，这只是临时修复方案，应尽量改为更为严谨的排序算法。
