---
layout: post
title: "Using Locks for More Granular Concurrent Access"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2009-07-02T17:05:30-07:00
comments: true
---

Locks introduced in Java 5 allow a more granular control on a shared resource. When a method is synchronized only the thread that obtains the lock is allowed to enter into that method or any other synchronized method of the class. However some times we may need a more granular control, typically when we have two or more resources we want to share across threads.

e.g.

{% highlight java %}
package org.lock;
 
public class LockExample {
private boolean initialized;
int salary=0;
 
public void increaseSalary() {
if(!initialized) {
initialized=true;
salary=1000;
}
salary=salary+100;
}
 
public void reset() {
initialized=false;
salary=0;
 
}
 
}
{% endhighlight %}

In this example we need to synchronize access on the initialization and the reset operations. Also we want to make sure that threads have a serialized access when they increase salary.

This is where we can leverage locks. We define two locks, one for initialization and one for the salary operation.

{% highlight java %}
package org.lock;
 
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
 
public class LockExample {
        Lock initializationLock=new ReentrantLock();
        Lock salaryLock=new ReentrantLock();
        private boolean initialized;
        int salary=0;
       
        public void increaseSalary() {
                if(!initialized) {
                        initializationLock.lock();
                        try {
                                initialized=true;
                                salary=1000;
                        } finally {
                                initializationLock.unlock();
                        }
                }
                salaryLock.lock();
                try {
                        salary=salary+100;
                } finally {
                        salaryLock.unlock();
                }
        }
       
        public void reset() {
                initializationLock.lock();
                try {
                        initialized=false;
                        salary=0;
                } finally {
                        initializationLock.unlock();
                }
        }
}
{% endhighlight %}

If you are using Java 1.4 you can utilize temporary synchronization objects to achieve the same result

{% highlight java %}
package org.lock;
 
public class Lock14Example {
        private Object initializationLock=new Object();
        private Object salaryLock=new Object();
        private boolean initialized;
        int salary=0;
       
        public void increaseSalary() {
                if(!initialized) {
                        synchronized(initializationLock) {
                                initialized=true;
                                salary=1000;
                        }
                }
                synchronized(salaryLock) {
                        salary=salary+100;
                }
        }
       
        public void reset() {
                synchronized(initializationLock) {
                        initialized=false;
                        salary=0;
                }
        }
}
{% endhighlight %}
