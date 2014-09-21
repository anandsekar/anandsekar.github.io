---
layout: post
title: "Multithreaded Producer Consumer in Java"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2011-06-09T15:50:23-07:00
comments: true
---
A classic case for multi-threaded programming is the producer consumer problem. In this case there is a producer that generates stuff to be consumed by the consumer, however the rate of production and the rate of consumption vary. This calls the need for having the producer and consumer run off different threads have co-ordinate them through a shared buffer or queue.

The classic way to implement this in Java is to utilize the object’s monitors and wait and notify.
{% highlight java %}
public class ClassicProducerConsumer {
        public static class Producer implements Runnable {
                private List<Integer> queue;
                private int next = 0;
 
                public Producer(List<Integer> queue) {
                        this.queue = queue;
                }
 
                @Override
                public void run() {
                        while (true) {
                                synchronized (queue) {
                                        queue.add(next);
                                        // The thread must own the monitor on the queue to call notify
                                        queue.notifyAll();
                                }
                                next++;
                        }
                }
        }
 
        public static class Consumer implements Runnable {
                private List<Integer> queue;
 
                public Consumer(List<Integer> queue) {
                        this.queue = queue;
                }
 
                @Override
                public void run() {
                        while (true) {
                                synchronized (queue) {
                                        if (queue.size() > 0) {
                                                Integer number = queue.remove(queue.size() – 1);
                                                System.out.println(number);
                                        } else {
                                                try {
                                                        // The thread must own queue’s monitor to call wait
                                                        queue.wait();
                                                } catch (InterruptedException e) {
                                                }
                                        }
                                }
                        }
                }
        }
 
        public static void main(String args[]) throws Exception {
                List<Integer> queue = new ArrayList<Integer>();
                Thread producer1 = new Thread(new Producer(queue));
                Thread producer2 = new Thread(new Producer(queue));
                Thread consumer1 = new Thread(new Consumer(queue));
                Thread consumer2 = new Thread(new Consumer(queue));
                producer1.start();
                producer2.start();
                consumer1.start();
                consumer2.start();
        }
}
{% endhighlight %}
With the java concurrent package since 1.5 there are much easier ways of implementing it. The best way is to use a blocking queue.
{% highlight java %}
public class ProducerConsumerWithBlockingQueue {
        public static class Producer implements Runnable {
                private BlockingQueue<Integer> queue;
                private int next = 0;
 
                public Producer(BlockingQueue<Integer> queue) {
                        this.queue = queue;
                }
 
                @Override
                public void run() {
                        while (true) {
                                try {
                                        queue.put(next);
                                } catch (InterruptedException e) {
                                }
                                next++;
                        }
                }
        }
        public static class Consumer implements Runnable {
                private BlockingQueue<Integer> queue;
 
                public Consumer(BlockingQueue<Integer> queue) {
                        this.queue = queue;
                }
 
                @Override
                public void run() {
                        while (true) {
                                synchronized (queue) {
                                        Integer next;
                                        try {
                                                next = queue.take();
                                                System.out.println(next);
                                        } catch (InterruptedException e) {
                                        }
                                }
                        }
                }
        }
 
        public static void main(String args[]) throws Exception {
                BlockingQueue<Integer> queue = new LinkedBlockingQueue<Integer>(1);
                Thread producer1 = new Thread(new Producer(queue));
                Thread producer2 = new Thread(new Producer(queue));
                Thread consumer1 = new Thread(new Consumer(queue));
                Thread consumer2 = new Thread(new Consumer(queue));
                producer1.start();
                producer2.start();
                consumer1.start();
                consumer2.start();
        }
}
{% endhighlight %}
One can also use the count down latch for the co-ordination if there is single producer and a single consumer
{% highlight java %}
public class ProducerConsumerWithCountDownLatch {
        public static class Producer implements Runnable {
                private List<Integer> queue;
                private CountDownLatch latch;
                private int next = 0;
 
                public Producer(List<Integer> queue,CountDownLatch latch) {
                        this.queue = queue;
                        this.latch=latch;
                }
 
                @Override
                public void run() {
                        while (true) {
                                synchronized (queue) {
                                        queue.add(next);
                                        latch.countDown();
                                }
                                next++;
                        }
                }
        }
 
        public static class Consumer implements Runnable {
                private List<Integer> queue;
                private CountDownLatch latch;
 
                public Consumer(List<Integer> queue,CountDownLatch latch) {
                        this.queue = queue;
                        this.latch=latch;
                }
 
                @Override
                public void run() {
                        while (true) {
                                Integer number=null;
                                synchronized (queue) {
                                        if (queue.size() > 0) {
                                                number = queue.remove(queue.size() – 1);
                                                System.out.println(number);
                                        }
                                }
                                if(number==null) {
                                        try {
                                                latch.await();
                                        } catch (InterruptedException e) {
                                        }
                                }
                        }
                }
        }
 
        public static void main(String args[]) throws Exception {
                List<Integer> queue = new ArrayList<Integer>();
                CountDownLatch latch=new CountDownLatch(1);
                Thread producer1 = new Thread(new Producer(queue,latch));
                Thread consumer1 = new Thread(new Consumer(queue,latch));
                producer1.start();
                consumer1.start();
        }
}
{% endhighlight %}
Another way is to use the exchanger if there is a single producer and a consumer
{% highlight java %}
public class ProducerConsumerWithExchanger {
        public static class Producer implements Runnable {
                private Exchanger<Integer> exchanger;
                private int next = 0;
 
                public Producer(Exchanger<Integer> exchanger) {
                        this.exchanger=exchanger;
                }
 
                @Override
                public void run() {
                        while (true) {
                                try {
                                        exchanger.exchange(next);
                                } catch (InterruptedException e) {
                                }
                                next++;
                        }
                }
        }
 
        public static class Consumer implements Runnable {
                private Exchanger<Integer> exchanger;
                public Consumer(Exchanger<Integer> exchanger) {
                        this.exchanger=exchanger;
                }
 
                @Override
                public void run() {
                        while (true) {
                                try {
                                        System.out.println(exchanger.exchange(0));
                                } catch (InterruptedException e) {
                                }
                        }
                }
        }
 
        public static void main(String args[]) throws Exception {
                Exchanger<Integer> exchanger=new Exchanger<Integer>();
                Thread producer1 = new Thread(new Producer(exchanger));
                Thread consumer1 = new Thread(new Consumer(exchanger));
                producer1.start();
                consumer1.start();
        }
}
{% endhighlight %}