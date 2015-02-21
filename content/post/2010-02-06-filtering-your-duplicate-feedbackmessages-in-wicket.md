---
author: admin
comments: true
date: 2010-02-06 10:11:00+00:00
layout: post
slug: filtering-your-duplicate-feedbackmessages-in-wicket
title: Filtering your duplicate feedbackmessages in Wicket
wordpress_id: 235
categories:
- Software
tags:
- java
- LinkedIn
- wicket
---

Last week I noticed that in some of my webapps written in Wicket, messages appear multiple times in a FeedbackPanel (Wicket component). That happens since multiple parts of the same page do different kind of validation which might occur in the same error/warning messages. Not a big problem but duplicate messages look a bit weird to your visitor. It's easy to fix and probably no rocket science to you but I post it anyway :).

The solution contains two parts, a custom FeedbackPanel and an implementation of the IFeedbackMessageFilter.

Custom FeedbackPanel:

[code language="java"]
private class UniqueMessagesFeedbackPanel extends FeedbackPanel
  {
    private UniqueMessageFilter filter = new UniqueMessageFilter();

    @Override
    protected void onBeforeRender()
    {
      super.onBeforeRender();
      // clear old messages
      filter.clearMessages();
    }

    public UniqueMessagesFeedbackPanel(String id)
    {
      super(id);
      setFilter(filter);
    }
  }
[/code]



And the filter class:


[code language="java"]
public class UniqueMessageFilter implements IFeedbackMessageFilter
  {
    List<FeedbackMessage> messages = new ArrayList<FeedbackMessage>();

    public void clearMessages()
    {
      messages.clear();
    }

    @Override
    public boolean accept(FeedbackMessage currentMessage)
    {
      // too bad that FeedbackMessage doesnt have an equals implementation
      for(FeedbackMessage message: messages)
        if(message.getMessage().toString().equals(currentMessage.getMessage().toString()))
          return false;
      messages.add(currentMessage);
      return true;
    }
  }
[/code]



I'm looking for a a more beautiful way to cleanup the messages but it seems impossible since the FeedbackMessage class does not implement an own equals version. And if it was implemented, it propably checks on too much properties, like the component which throws the message and that still results in duplicate messages.
