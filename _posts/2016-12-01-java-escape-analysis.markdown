---
layout: post
comments: true
title: Java Escape Analysis
date: '2016-12-01 10:49'
categories:
  - testing-and-analysis
  - performance
---

This post is adapted from  [post1](https://www.burnison.ca/articles/escape-analysis-stack-allocated-objects) and [post 2](https://dzone.com/articles/do-not-let-your-java-objects-escape).

Starting from Java SE 6U23, JVM by default optimizes the performance of Java programs (especially long running ones) via [_escape analysis_](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/performance-enhancements-7.html#escapeAnalysis).

---

#### What is **_escape analysis_** and why it matters?
In compiler optimization, escape analysis (EA) is a method for determining the dynamic scope of pointers - where in the program a pointer can be accessed ([Wikipedia](https://en.wikipedia.org/wiki/Escape_analysis)). For Java programs, EA is a technique that can analyze the scope of a new object's uses and decide whether to allocate it on the Java heap ([research paper](http://dl.acm.org/citation.cfm?id=320386)).

Allocating objects on heap has several draw backs.

- First, objects can be placed on different memory addresses, which can be far away from each other. Such allocation can deplete CPU caches, causing performance degradation.
- Second, when many objects are allocated on heap, GC needs to run frequently. Every time it scans the heap, it slows down your program.

If we do not allocate certain temporary objects on heap, but create a stack local representation, we can get several benefits.

- It improves data locality. Accessing data on stack is far more efficient than accessing data on heap.
- It reduces the burden of garbage collectors. When objects are allocated on stack, there is no need for garbage collection as the block of memory assigned to the call frame on stack will be freed when the method call returns.
- We can freely use local abstractions like Lambdas, Functions, Streams and Iterators without worrying about too much performance penalty.

Now in what cases, we can allocate objects on stack. Let's first understand the lifetime of Java objects.

---

#### Three types of **_object escapement_**
- `NoEscpae`: an object does not escape the method that creates it, and the thread in which the method is invoked. In such cases, the objects are scalar replaceable objects, meaning their allocation could be removed from generated code. For example, when an object is created locally in a method and the method does not create any other thread objects, the object is a 'NoEscape' one and cannot be observed outside the current method or thread.
- `ArgEscape`: an object that is passed as an argument to a method but cannot otherwise be observed outside the method or by other threads (the object escapes that method via being passed as method arguments, but does not escape the thread in which it is created).
- `GlobalEscape`: an object escapes the method and thread. For example, an object stored in a static field or a field of an escaped object, or returned as the result of the current method.

---

#### Demo with an example
From the above discussion, we can see `NoEscape` objects clearly are stack allocatable. Let's demonstrate the benefits of EA by an example.

```java
import org.apache.commons.lang3.builder.EqualsBuilder;
import java.io.*;
import java.util.Random;

public final class EscapeTest {
    private final int a;
    private final int b;

    EscapeTest(final int a, final int b) {
        this.a = a;
        this.b = b;
    }

    @Override
    public boolean equals(final Object obj) {
        final EscapeTest other = (EscapeTest)obj;
        return new EqualsBuilder()
            .append(this.a, other.a)
            .append(this.b, other.b)
            .isEquals();
    }

    public static void main(final String[] args) throws IOException {
        Runtime runtime = Runtime.getRuntime();
        final Random random = new Random();
        for(int i = 0; i < 5_000_000; i++){
            final EscapeTest t1 = new EscapeTest(random.nextInt(), random.nextInt());
            final EscapeTest t2 = new EscapeTest(random.nextInt(), random.nextInt());
            if(t1.equals(t2)){
                System.out.println("Prevent anything from being optimized out.");
            }
        }
        long maxMemory = runtime.maxMemory();
        long allocatedMemory = runtime.totalMemory();
        System.out.println("max memory: " + maxMemory / 1024 / 1024);
        System.out.println("used memory: " + allocatedMemory / 1024 / 1024);
        System.in.read();
    }
}
```

Let's first run the program with EA enabled.

```
java -Xmx1G -cp "./;lang.jar" EscapeTest
```

Here is the output:

```
max memory: 910
used momory: 123
```

Then Let's disable EA and run the program again.

```
java -Xmx1G -XX:-DoEscapeAnalysis -cp "./;lang.jar" EscapeTest
```

Here is the output:

```
max memory: 910
used memory: 216
```

We can clearly see the difference in memory usage. With EA enabled, the `NoEscape` objects such as `t1` and `EqualsBuilder` (in fact if `equals()` method is inlined with compiler optimization, `t2` is also `NoEscape`) do not need to be allocated on heap.

Let's further analyze the object allocation details with the `jps` and `jmap` tool. We can first use `jps` to get the pid of the running Java program and then use `jmap` to see memory details with the following command:

```bash
jmap -histo pid | head
```

Here is the result for the run with EA enabled:

```
num     #instances         #bytes  class name
----------------------------------------------
  1:        336476        8075424  EscapeTest
  2:           555        1232544  [I
  3:          3660         518768  [C
  4:          8732         139712  org.apache.commons.lang3.builder.EqualsBuilder
  5:           176         128032  [B
  6:          2849          68376  java.lang.String
  7:           582          66160  java.lang.Class
```

Here is the result for the run with EA disabled:

```
num     #instances         #bytes  class name
----------------------------------------------
   1:       1532068       36769632  EscapeTest
   2:        766034       12256544  org.apache.commons.lang3.builder.EqualsBuilder
   3:           150        2805064  [I
   4:          2484         321504  [C
   5:           582          66160  java.lang.Class
   6:          2338          56112  java.lang.String
   7:           865          34600  java.util.TreeMap$Entry
```
Look at the number of `EscapeTest` and `EqualsBuilder` objects and you will see how useful EA is to help improve runtime performance of Java programs.


#### **References**
1. Jong-Deok Choi, Manish Gupta, Mauricio Serrano, Vugranam C. Sreedhar, and Sam Midkiff. 1999. Escape analysis for Java. In Proceedings of the 14th ACM SIGPLAN conference on Object-oriented programming, systems, languages, and applications (OOPSLA '99).
2. JVM Documentation: [https://docs.oracle.com/javase/8/docs/technotes/guides/vm/performance-enhancements-7.html#escapeAnalysis](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/performance-enhancements-7.html#escapeAnalysis)
