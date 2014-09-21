---
layout: post
title: "InheritableThreadLocal and Tomcat"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2008-05-08T05:36:57-07:00
comments: true
---

InheritableThreadLocal does not work with tomcat as tomcat thread pool does not clear the thread local contacts after a request has been executed. This results in no good way to share information information between the parent and the child threads that are created using thread local variables.

One way to solve this problem is to maintain a reference to the parent thread by writing your own thread that is constructed by a thread factory.

{% highlight java %}
	public class MyThread extends Thread {
        private Thread parent;
        public MyThread(Thread parent) {
                super();
                this.parent = parent;
        }
        public Thread getParent() {
                return parent;
        }
}
{% endhighlight %}


{% highlight java %}
import java.util.concurrent.ThreadFactory;
public class MyThreadFactory implements ThreadFactory {
        @Override
        public Thread newThread(Runnable arg0) {
                return new MyThread(Thread.currentThread());
        }
}

{% endhighlight %}


{% highlight java %}
import java.util.HashMap;
import java.util.Map;
public class ClassNeedingInheritableThreadLocal {
        private Map inheritableThreadLocal=new HashMap();
        public Object getInheritableThreadContext() {
                return inheritableThreadLocal.get(getThread());
        }
        public void setInheritableThreadContext(Object context) {
                inheritableThreadLocal.put(getThread(), context);
        }
        private Thread getThread() {
                Thread currentThread=Thread.currentThread();
                if(currentThread instanceof MyThread) {
                        currentThread=getParent((MyThread) currentThread);
                }
                return currentThread;
        }
        public Thread getParent(MyThread myThread) {
                Thread parent=myThread.getParent();
                if(parent instanceof MyThread) {
                        return getParent((MyThread) parent);
                } else {
                        return parent;
                }
        }
}
{% endhighlight %}

Create a executor service by using our thread factory

{% highlight java %}
ExecutorService service=Executors.newFixedThreadPool(10, new MyThreadFactory());
{% endhighlight %}

Remember that you would need to clean up all the objects that were put in the thread local. This can be done with a filter or by using AOP.