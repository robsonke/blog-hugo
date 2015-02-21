---
author: admin
comments: true
date: 2009-05-09 08:50:07+00:00
layout: post
slug: springs-threadlocaltargetsource
title: Spring's ThreadLocalTargetSource
wordpress_id: 119
categories:
- Software
tags:
- java
- LinkedIn
- spring
---

[ThreadLocalTargetSource](http://static.springframework.org/spring/docs/2.5.x/api/org/springframework/aop/target/ThreadLocalTargetSource.html), as the name already explains, is a proxied ThreadLocal helper class. Spring provides this class together with other pooling mechanisms as a better version of the Sun's ThreadLocal class. One of the benefits is the already created wrapper class around the threadlocal and much more important, it destroys automatically the old thread references when the BeanFactory is destructed.

But actually these kind of features always sound to good to be true to me. I really wanted to figure out how safe it is and how far you can go. A simple testapplication showed me that it's absolutely powerful!

So far I didn't have much experience with ThreadLocal's, they sound a bit evil to but on the other hand very useful. And I was afraid for the memory usage, keeping objects in memory per thread must have its cost. But Spring insures me that the objects will be automatically removed as soon as possible, although that still means that objects can live quite some time. Especially the automatically destroy functionality was for me a good reason to use the ThreadLocalTargetSource and not a custom wrapper for a ThreadLocal. If you really would like to use the ThreadLocal, be sure that you don't forget to unset the object for every requestcycle. Otherwise it's a just a matter of time before your application will start throwing Java heap space errors :).

Now it's time for my example, to start with the bean config:

``` xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.0.xsd">

<bean id="testBean" class="org.springframework.aop.framework.ProxyFactoryBean">
<property name="targetSource" ref="testThreadLocalTargetSource" />
</bean>
<bean id="testBeanUtil" class="nl.tigrou.threadlocal.TestBeanUtil">
<property name="testBean" ref="testBean" />
</bean>
<bean id="testThreadLocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource"
destroy-method="destroy">
<property name="targetBeanName" value="testTarget" />
</bean>
<bean id="testTarget" class="nl.tigrou.threadlocal.TestBeanImpl" scope="prototype">
<property name="myValue" value="boe!" />
</bean>
</beans>

```

A simple Pojo:

``` java

public class TestBeanImpl implements TestBean {

  private String myValue;

  public void setMyValue(String myValue) {
    this.myValue = myValue;
  }

  public String getMyValue() {
    return myValue;
  }

}

```

A util class to test the proxy'ing even better:

``` java

public class TestBeanUtil {

  private static TestBean testBean;

  public void setTestBean(TestBean testBean) {
    TestBeanUtil.testBean = testBean;
  }

  public static TestBean getTestBean() {
    return testBean;
  }
}

```

And a simple testclient with 2 testcases:

``` java

public class ThreadLocalTest {

  private ClassPathXmlApplicationContext context;

  public static void main(String[] args) {
    new ThreadLocalTest();
  }

  public ThreadLocalTest() {

    context = new ClassPathXmlApplicationContext("applicationContext.xml");

    for(int i=0; i < 10; i++)
    {
      ThreadTest test = new ThreadTest(i);
      test.setName("Thread1-"+i);
      test.start();
    }

    for(int i=0; i < 10; i++)
    {
      Thread2Test test2 = new Thread2Test(i);
      test2.setName("Thread2-"+i);
      test2.start();
    }
  }

  private class ThreadTest extends Thread
  {
    private final int number;

    public ThreadTest(int number) {
      this.number = number;
    }

    public void run() {
      TestBean testBean = (TestBean) context.getBean("testBean");

      testBean.setMyValue(String.valueOf(number));
      System.out.println(Thread.currentThread().getName() + ", bean value: " + testBean.getMyValue());
    }
  }

  private class Thread2Test extends Thread
  {
    private final int number;

    public Thread2Test(int number) {
      this.number = number;
    }

    public void run() {
      TestBeanUtil.getTestBean().setMyValue(String.valueOf(number));
      System.out.println(Thread.currentThread().getName() + ", bean value: " + TestBeanUtil.getTestBean().getMyValue());
    }
  }

}

```

The above testclient starts 10 x 2 threads where both different threadclasses read and set the value in the threadlocal. The result in the console is:

    
    Thread1-0, bean value: 0
    Thread1-2, bean value: 2
    Thread1-3, bean value: 3
    Thread2-4, bean value: 4
    Thread1-4, bean value: 4
    Thread2-5, bean value: 5
    Thread1-9, bean value: 9
    Thread2-6, bean value: 6
    Thread2-2, bean value: 2
    Thread1-7, bean value: 7
    Thread2-0, bean value: 0
    Thread2-1, bean value: 1
    Thread2-8, bean value: 8
    Thread1-5, bean value: 5
    Thread1-8, bean value: 8
    Thread1-6, bean value: 6
    Thread2-3, bean value: 3
    Thread2-7, bean value: 7
    Thread2-9, bean value: 9
    Thread1-1, bean value: 1


Totally random order of execution as you can expect with threads but never a duplicate combination of threadnumber and value! Very simple but very powerful. You can access the beanvalues from everywhere withing the same thread, even with a static reference in a util class.

Next subject, my AOP headache issue with a combination of Advices and Aspects.
