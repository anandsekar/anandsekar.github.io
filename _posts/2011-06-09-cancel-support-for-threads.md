---
layout: post
title: "Cancel Support for Threads"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2011-06-09T15:45:25-07:00
comments: true
---

A common use case is to use worker threads to execute jobs. While it is easy to spin up threads, one must think about adding cancel support to interrupt the job. One strategy to use is to use Thread.interrupt() and the use of a “Poison Pill” to stop the thread.

Here is a classic way how it is done
{% highlight java%}
public class CancelSupport {
        public static class CommandExecutor implements Runnable {
                private BlockingQueue<String> queue;
                public static final String POISON_PILL  = “stopnow”;
                public CommandExecutor(BlockingQueue<String> queue) {
                        this.queue=queue;
                }
                @Override
                public void run() {
                        boolean stop=false;
                        while(!stop) {
                                try {
                                        String command=queue.take();
                                        if(POISON_PILL.equals(command)) {
                                                stop=true;
                                        } else {
                                                // do command
                                                System.out.println(command);
                                        }
                                } catch (InterruptedException e) {
                                        stop=true;
                                }
                        }
                        System.out.println(“Stopping execution”);
                }
               
        }
}
{% endhighlight %}
One can stop the thread by interrupting it .. and if that fails to send a poison pill
{% highlight java%}
BlockingQueue<String> queue=new LinkedBlockingQueue<String>();
Thread t=new Thread(new CommandExecutor(queue));
queue.put(“hello”);
queue.put(“world”);
t.start();
Thread.sleep(1000);
t.interrupt();
{% endhighlight %}
If interrupting it fails, You can also inject a “poison pill”

{% highlight java%}
BlockingQueue<String> queue=new LinkedBlockingQueue<String>();
Thread t=new Thread(new CommandExecutor(queue));
queue.put(“hello”);
queue.put(“world”);
t.start();
Thread.sleep(1000);
queue.put(“stopnow”);
{% endhighlight %}
