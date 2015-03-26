---
layout: post
title: "ActiveMQ High Availablity"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2011-06-10T15:42:46-07:00
comments: true
---

One of the features that looks great about ActiveMQ is the network of brokers. While it looks great in documentation it absolutly does not work. There are two many issues with the alogrithm it uses to hop, which ends up stranding messages in the broker as well as duplicates in a topic. So without a active-active setup the next best case becomes active-passive setup. This is done by configuring ActiveMQ to write its journal’s to a shared disk where the first broker to aquire a lock on the journal becomes the active. This does not work well too when only the shared network journal is unavailable as ActiveMQ holds on the the existing connections. Thus the only way is to write your own locking mechanism to detect unavailability of shared network drive and shutdown ActiveMQ.
![activemqha]({{ site.url }}/assets/images/activemqha/activemqha.png)

Here is a simple program that can detect if the filesystem is available.

{% highlight java %}
public class TestFileSystemAccess implements Runnable{
	private String fileName;
	private static final String CONTENT="content";
	
	public TestFileSystemAccess(String fileName) {
		super();
		this.fileName = fileName;
	}

	private boolean test() {
		try {
			Files.write(CONTENT, new File(fileName), Charsets.UTF_8);
			CharSource source = Files.asCharSource(new File(fileName), Charsets.UTF_8);
			String str=source.read();
			if(!CONTENT.equals(str)) {
				return false;
			} else {
				return true;
			}
		} catch (IOException e) {
			return false;
		}
	}

	@Override
	public void run() {
		while(true) {
			boolean canWrite=test();
			if(!canWrite) {
				// lost access to file system .. exit
				System.exit(1);
			} else {
				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
				}
			}
		}
	}
	
	public void start() {
		Thread t=new Thread(this);
		t.start();
		
	}
}
{% endhighlight java %}
The spring configuration will look something like
{% highlight xml %}
<broker brokerName="broker" persistent="true" useShutdownHook="false">
   <transportConnectors>
     <transportConnector uri="tcp://localhost:61616"/>
   </transportConnectors>
   <persistenceAdapter>
     <kahaPersistenceAdapter dir="shareddir/activemq-data" maxDataFileLength="33554432"/>
   </persistenceAdapter>
 </broker>
 <bean class="TestFileSystemAccess" init-method="start">
 	<constructor-arg>shareddir/activemq-data</constructor-arg>
 </bean>
 {% endhighlight xml %}