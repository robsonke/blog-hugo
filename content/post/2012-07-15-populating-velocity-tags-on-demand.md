---
author: admin
comments: true
date: 2012-07-15 07:36:06+00:00
layout: post
slug: populating-velocity-tags-on-demand
title: Populating Velocity tags on demand
wordpress_id: 272
categories:
- Software
tags:
- java
- velocity
---

Some months ago I was looking for a solution to my performance issue in [Apache Velocity](http://velocity.apache.org/). 5% Of the tags used in a Velocity context was causing a slow down which where only used in a very small amount of processed templates. I did some googling on words like dynamic, template, real time, etcetera but without a good result. The solution wasn't that rocket science but it won't hurt sharing. The trick is registering an eventhandler, more specifically implementing your own ReferenceInsertionEventHandler. That allows you to hook in once a tag is being parsed in a template.

Implementation of a ReferenceInsertionEventHandler:

``` java
package nl.tigrou.velocity;

import java.util.List;
import java.util.Map;

import org.apache.velocity.app.event.ReferenceInsertionEventHandler;
import org.apache.velocity.context.Context;
import org.apache.velocity.util.ContextAware;

/**
 *
 * Eventhandlers which is executing and populating the context on the fly
 * for specific (slow) tags
 *
 */
public class OnDemandTagReferenceHandler implements ReferenceInsertionEventHandler, ContextAware
{
  private Context context;

  @Override
  public void setContext(Context context)
  {
    this.context = context;
  }

  @Override
  public Object referenceInsert(String reference, Object value)
  {
    if(value == null && reference.equals("myDynamicTag"))
    {
      // get your value
      String myValue = myServiceClass.getDynamicValue();
      return myValue;
    }
    return value;
  }
}
```


All what's left is registering this event handler to your context. For example programmatically:

``` java
Velocity.init();   
ctx = new VelocityContext();   
EventCartridge ec = new EventCartridge(); 
ec.addEventHandler(new OnDemandTagReferenceHandler()); 
ec.attachToContext(ctx);
```

Next topic will probably be about documenting webservices. I'm currently stucked in a good solution.
