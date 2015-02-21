---
author: admin
comments: true
date: 2010-01-02 15:47:07+00:00
layout: post
slug: context-based-urls-with-apache-wicket
title: Context based urls with Apache Wicket
wordpress_id: 225
categories:
- Software
tags:
- java
- LinkedIn
- wicket
---

It's been a while that I wrote something technical on my blog. The last ones were mainly about biking which is taking a lot of my spare time these days. I haven't blogged about Wicket before but that doesn't mean that I use it a lot. For the last 2 years, I'm using it at my work at [Maxxton](http://www.maxxton.com) for developing rich content websites. In one of the usecases a month ago, we needed something for storing parameters in the urls which would never get lost after clicking around. After a little googling on the web, I found an example on the [Wicket wiki ](http://cwiki.apache.org/WICKET/wicket-and-localized-urls.html)which allowed you to add the locale in the url. I've re-used that example and made it a bit more generic by allowing key-value pairs instead of one hardcoded string.

The used keys are defined by a simple enum:



``` java

private enum Param {
  LANGUAGE("lan"), THEME("theme");

  private final String value;

  Param(String value) {
    this.value = value;
  }

  public String getValue() {
    return value;
  }
};

```


In this example I used two different params which can be used in combination with eachother or separated. I won't add all the Java code in long listings over here because of the amount of code but you can download my Eclipse project at the end of this blogpost.


There are actually two important spots, a custom RequestCodingStrategy and the registration of that class in the WebApplication. But before I show that code, let's give some examples of the urls which will be created/working:


  * http://localhost:8080/urlexample**/theme/red/lan/nl**/home
  * http://localhost:8080/urlexample**/lan/nl**/home
  * http://localhost:8080/urlexample**/theme/red/**home
  * http://localhost:8080/urlexample**/**home

All of these urls are pointing to the homepage mounted at "home". The bold parts are the key-value pairs which are read in our RequestCodingStrategy and stored in the session for later use. The beauty of this solution is that every generated url, will still contain these values. So in my example, I created a link to another page called "end" and that url will be something like http://localhost:8080/urlexample**/theme/red/lan/nl**/end.

So how does that look like, lets start with the 2 overridden methods in my WebApplication:


``` java

@Override
protected IRequestCycleProcessor newRequestCycleProcessor() {
  return new WebRequestCycleProcessor() {

    @Override
    protected IRequestCodingStrategy newRequestCodingStrategy() {
      return 
        new ParamUrlCodingStrategy(super.newRequestCodingStrategy());
    }
  };
}

@Override
public Session newSession(Request request, Response response) {
  return new CustomSession(request);
}

```

The newRequestCycleProcessor() returns the custom RequestCodingStrategy instead of the default one. and the newSession() returns our custom session for storing the values. Nothing special.

Lets take a look now to our custom RequestCodingStrategy where most of the magics happens. I only show a stripped version of the one in the example project:



``` java

public class ParamUrlCodingStrategy extends RequestCodingStrategy {

  public ParamUrlCodingStrategy(final IRequestCodingStrategy defaultStrategy) {
    super(defaultStrategy);
  }

  @Override
  public RequestParameters decode(final Request request) {
    final String requestLanguage = getParamValue(request.getPath(),Param.LANGUAGE);
    final String requestTheme = getParamValue(request.getPath(),Param.THEME);

    // for example, use a custom session object to store your values
    if (requestLanguage != null) {
      ((CustomSession) Session.get()).setLanguage(requestLanguage);
    }
    if (requestTheme != null) {
      ((CustomSession) Session.get()).setTheme(requestTheme);
    }

    String url = request.decodeURL(request.getURL());

    // remove params from request
    for (Param param : Param.values()) {
      url = stripParamFromPath(url, param);
    }
    final String urlWithoutParams = url;

    // use decorator for decoding
    return getDecoratedStrategy().decode(new RequestDecorator(request) {
      @Override
      public String getURL() {
        return urlWithoutParams;
      }

      @Override
      public String getPath() {
        return urlWithoutParams;
      }
    });
  }

  @Override
  public CharSequence encode(final RequestCycle requestCycle, final IRequestTarget requestTarget) {
    String url = getDecoratedStrategy().encode(requestCycle, requestTarget).toString();

    // rewrite only requests for pages &amp;amp; links
    if (requestTarget instanceof IBookmarkablePageRequestTarget
    || requestTarget instanceof IPageRequestTarget) {

      String theme = ((CustomSession) Session.get()).getTheme();
      String language = ((CustomSession) Session.get()).getLanguage();

      // language
      if (language != null &amp;amp;&amp;amp; !language.isEmpty()) {
        if (url.startsWith("../")) {
          final int lastIndex = url.lastIndexOf("../") + 3;
          final String remainingUrl = url.substring(lastIndex);
          if (!remainingUrl.isEmpty()) {
            url = url.substring(0, lastIndex)
            + Param.LANGUAGE.getValue() + "/" + language
            + "/" + remainingUrl;
          }
        }
        // if starts with . -&gt; skip
        else if (!url.startsWith(".")) {
        url = Param.LANGUAGE.getValue() + "/" + language + "/"
        + url;
        }
      }
      // and theme
      if (theme != null &amp;amp;&amp;amp; !theme.isEmpty()) {
      if (url.startsWith("../")) {
      final int lastIndex = url.lastIndexOf("../") + 3;
      final String remainingUrl = url.substring(lastIndex);
      if (!remainingUrl.isEmpty()) {
      url = url.substring(0, lastIndex)
      + Param.THEME.getValue() + "/" + theme + "/"
      + remainingUrl;
      }
      }
      // if starts with . -&gt; skip
      else if (!url.startsWith(".")) {
      url = Param.THEME.getValue() + "/" + theme + "/" + url;
      }
      }
      return url;
    }
    return url;
  }

  private String getParamValue(final String path, Param param) {
    int index = path.indexOf(param.getValue());
    if (index &gt;= 0) {
      int indexOfNextSlash = path.indexOf("/", index+ param.getValue().length() + 1);
      String paramValue = path.substring(index + param.getValue().length() + 1, indexOfNextSlash);
      return paramValue;
    }
    return null;
  }

  private String stripParamFromPath(final String path, Param param) {
    String paramValue = getParamValue(path, param);

    return path.replace(param.getValue() + "/" + paramValue + "/", "");
  }

  @Override
  public IRequestTarget targetForRequest(final RequestParameters requestParameters) {
    // delegate call to decorated codingStrategy, but with params removed
    String newPath = requestParameters.getPath();
    for (Param param : Param.values()) {
      newPath = stripParamFromPath(newPath, param);
    }

    requestParameters.setPath(newPath);
    return super.targetForRequest(requestParameters);
  }

  @Override
  public IRequestTargetUrlCodingStrategy urlCodingStrategyForPath(final String path) {
    String newPath = path;
    for (Param param : Param.values()) {
      newPath = stripParamFromPath(newPath, param);
    }

    if (newPath != null &amp;amp;&amp;amp; !newPath.isEmpty()) {
      // treat this kind of situation by returning some not null codingStrategy,
      // to let this kind of request treated by wicket
      return new PassThroughUrlCodingStrategy(newPath);
    }
    return super.urlCodingStrategyForPath(newPath);
  }

}

```

As you can see, the encode() and decode() methods do most of the processing. It's actually nothing more then adding and removing/reading the parameters. The reason that I liked it, is that it's always executed, no matter what code is used inside the pages. That's a secure way of adding parameters and not loosing them and as a little dessert, Google likes them as well! Urls are indexed including state in Google which allows you to create language based urls for example.

So that's it for now. The post only shows the keyfactors of my example. If you're more interested, you should download my example project and dive into the code.

