---
layout: post
title: "Spring Configuration in Java"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2015-01-30T05:19:12-08:00
comments: true
---

Since version 3.0 spring supports java based configuration. The main quibble i have with spring xml configuration is the readability. Once your code base grows the spring xml files become less readable. With java based congfiguration you get good readbility. 

To use the java based configuration one has to annoate the class with @Configuration and the beans with @Bean

{% highlight java linenos%}
@Configuration
public class SpringConfig {
	@Bean
	public MyBean myBean() {
		return new MyBeanImpl();
	}
}
{% endhighlight %}

You can now configure the application context in the following way

{% highlight java linenos%}
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
    MyBean myBean = ctx.getBean(MyBean.class);
}
{% endhighlight %}

The very interesting thing is that dependencies are stasified using method invokcation, which is very intutive.

{% highlight java linenos%}
@Configuration
public class SpringConfig {
	@Bean
	public MyBean myBean() {
		return new MyBeanImpl(myDepBean());
	}
	@Bean
	public MyDepBean myDepBean() {
		return new MyDepBeanImpl();
	}
}
{% endhighlight %}

The most useful feature is that it is possible to configure two beans of the same type very easily. This overcomes the problems faced with spring configuration by annotation.

{% highlight java linenos%}
@Configuration
public class SpringConfig {
	@Bean
	@Qualifier(value="retryProcessor")
	public MyProcessor myRetryableProcessor() {
		return new MyProcessorImpl(retryFailureStrategy());
	}
	@Bean
	@Qualifier(value="skipProcessor")
	public MyProcessor mySkipProcessor() {
		return new MyProcessorImpl(skipFailureStrategy());
	}
	@Bean
	public FailureStrategy retryFailureStategy() {
		return new RetryFailureStrategyImpl();
	}

	@Bean
	public FailureStrategy skipFailureStategy() {
		return new skipFailureStrategyImpl();
	}
}
{% endhighlight %}

The beans now can be accessed in the following way

{% highlight java linenos%}
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
    MyProcessor retry=ctx.getBean("retryProcessor");
    MyProcessor skip=ctx.getBean("skipProcessor");
}
{% endhighlight %}

By default the scope of the beans are singletons. You can define the scope using the @Scope. 

More information is available at [Spring Documentation](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-java)