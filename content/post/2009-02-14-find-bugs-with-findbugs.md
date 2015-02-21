---
author: admin
comments: true
date: 2009-02-14 14:54:42+00:00
layout: post
slug: find-bugs-with-findbugs
title: Find bugs with findbugs
wordpress_id: 12
categories:
- Software
tags:
- Eclipse
- java
- LinkedIn
---

This week a colleague of mine mentioned me about a nice tool called [findBugs (open source)](http://findbugs.sourceforge.net/). So I started reading about it and till that point I was never really interested in "code analysis tools", since that's what findBugs is. Since I'm much more fan of some action instead of reading pages of documentation I installed the Eclipse plugin and ran a check on my current project. And I was surprised! It found 153 bugs! So I started exploring them and many of them weren't really bugs but could lead to possible bugs. But about 20 bugs were really serious issues which I hadn't noticed yet (project is still in dev phase) but now have probably saved me a lot of debugging time.

Lets take a look at some of the bugs it mentioned to me. I've made the examples easier to read because we're not interested in all the other code.

**1. Dividing integers and assigning the result to a double**

I'll use the example from findBugs itself:
[sourcecode language='java']
int x = 2;
int y = 5;
// Wrong: yields result 0.0
double value1 =  x / y;
// Right: yields result 0.4
double value2 =  x / (double) y;
[/sourcecode]
**2. Creating instances of String, Long, Integer  etc**

This is not really a bug but just good thing to remember, there are three ways to create an instance:
[sourcecode language='java']
String s = new String("foo"); // which I often used
String s = "foo";
// or
String s = String.valueOf("foo"); // these last two could be up to 3,5 times faster then the one above (based on jvm caching)
[/sourcecode]

**3. Null pointer dereference**

Normally you always catch possible null values. Anyway, in my case I used a variable halfway a method and added also a null check, but there was just one line of code 20 lines higher which could cause problems:
[sourcecode language='java']
private void foo(Object o){

String s = (String) o;
/* 20 lines of code which are not using s */
if(s != null) print(s);
}
[/sourcecode]
Findbugs now told me that I should add a check at the first line too. Pretty impressive in my opinion!

And this is just the top of the iceberg, see the link at the bottom about all kind of things which findBugs will search for. It opened my eyes and I'll use it in every project in the future! Keep up the good work.

Interesting links:



	
  * [Findbugs](http://findbugs.sourceforge.net/)

	
  * [Kind of bugs](http://findbugs.sourceforge.net/bugDescriptions.html)


