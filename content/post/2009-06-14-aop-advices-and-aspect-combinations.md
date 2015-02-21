---
author: admin
comments: true
date: 2009-06-14 18:37:18+00:00
layout: post
slug: aop-advices-and-aspect-combinations
title: AOP Advices and Aspect combinations
wordpress_id: 153
categories:
- Software
tags:
- AOP
- java
- LinkedIn
- spring
---

It's about a month ago that I was completely stucked with a Spring AOP configuration. While working on a better transaction management (see older post), I kept struggling in a circle of either a non functioning transaction system or broken AOP aspects. It really drove me nuts so I decided to avoid that pain for somebody else. No examples and code this time, just a simple pointer to the Spring documentation (which clearly took too much time to find :)).

Let's start with the note in the documentation:

**_Advising aspects_**

**_In Spring AOP, it is not possible to have aspects themselves be the target of advice from other aspects. The @Aspect annotation on a class marks it as an aspect, and hence excludes it from auto-proxying._**

([Source](http://static.springframework.org/spring/docs/2.5.x/reference/aop.html#aop-at-aspectj) - Spring reference docs)

After searching a bit more information, since the text above not really explains why they're excluded, I found the following 2 "rules":



	
  * Classes with annotations _@AspectJ_ and classes that implement or extend any other AOP component are excluded from the autoproxy. This is because they aren't target classes, and they perform tasks in Spring AOP infrastructure.

	
  * Proxy is not applied to beans that implement the interfaces _BeanPostProcessor_ or _BeanFactoryPostProcessor_. The class _AnnotationAwareAspectJAutoProxyCreator_ implements the interface _BeanPostProcessor_, which allows the class to modify the life cycle of beans on which a proxy must be created and applied.


The first point here explains why Aspects are not proxied. The second point names another condition.

So, conclusion:

**Never use Aspects (@Apects) and proxy based transaction management on the same bean! **

It's simple to avoid by creating a separate Aspect which advices other beans. Something like this:

[code lang="java"]
@Aspect
public class TestAspect
{

  @Before("execution(* someThing) ")
  public void doSomethingElse()
  {

  }
}
[/code]

That's it! At the moment I'm working on the website of my local athletics club, with [CodeIgniter](http://www.codeigniter.com) I'm finally enjoying PHP programming again.
