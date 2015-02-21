---
author: admin
comments: true
date: 2011-06-11 09:50:34+00:00
layout: post
slug: cached-credentials-in-http-basic-authentication
title: Cached credentials in http basic authentication
wordpress_id: 249
categories:
- Software
tags:
- java
- LinkedIn
---

After some radio silence, it's time for a short post. Being busy with [Duchenne Heroes](http://www.duchenneheroes.nl) last year and some weeks of snowboarding I'm posting a new blog about Java's AuthCache to avoid breaking other people's head as it broke mine.

Two weeks ago I was working on a connection with a common webservice which authentication was based on plain http basic authentication. I used [CXF](http://cxf.apache.org/) as a client library to handle the plumbing for me, the basic code looked like this:

[code language="java"]
JaxWsProxyFactoryBean clientFactory = new JaxWsProxyFactoryBean();
clientFactory.setAddress("http://webservice.url.com");
clientFactory.setServiceClass(MyLocalInterface.class);
clientFactory.setUsername("rob");
clientFactory.setPassword("secret");
fundaSoap = clientFactory.create(MyLocalInterface.class);
[/code]

From the start this worked great and simple, till the moment that I set up another connection based on the same url but other credentials...... That's where the problem began. Connection nr 2 was now messed up with the credentials of connection nr 1 and as I never heard of this behavior, I refused to believe it either. But after diving into server logs, I couldn't deny it anymore. My client code was screwed up. After some googling I found the problem, the JDK is caching credentials based on the url and port of your http request. And resetting that cache by calling the Authenticator class doesn't help you due to this bug:

[http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6626700](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6626700)

So there's just one way to avoid this bug and that's fooling the JDK that you've used the credentials before:

[code language="java"]
AuthCacheValue.setAuthCache(new AuthCache()
{
  @Override
  public void remove(String arg0, AuthCacheValue arg1) { }

  @Override
  public void put(String arg0, AuthCacheValue arg1) { }

  @Override
  public AuthCacheValue get(String arg0, String arg1)
  {
    return null;
  }
});
[/code]

I hope my post will help other people with the same kind of issues as this took me some time to figure out what was happening and I don't want that somebody else spoils his or her time on this.

Apart from this post, I'm really happy with the CXF library (based on the old XFire project) and I'll try to publish more on that later.
