---
author: admin
comments: true
date: 2009-04-07 20:38:27+00:00
layout: post
slug: keep-an-eye-on-your-ibatis-cachemodels
title: Keep an eye on your iBatis cachemodels
wordpress_id: 82
categories:
- Software
tags:
- iBatis
- java
- LinkedIn
---

As you might have read already, I'm using iBatis for more then a year now and so far I really like it. Ok, it has some drawbacks and not comparable to something like Hibernate. IBatis is just an OR Mapper. The current 2.3.x release is a bit outdated in my opinion but they're working hard on the new and shiny 3.0 version ([read here](http://opensource.atlassian.com/confluence/oss/display/IBATIS/iBATIS+3.0+Whiteboard) about the plans) but that's still far away, a total rewrite takes time.

One of the things clear things in iBatis is the caching mechanism, it's plain and simple and easy customizable. By default, it's using a LRU algorithm but you can change that easily to something different, or maybe even cooler, you're own algorithm. But that's for another post in the future. Last weekend I was thinking about how to monitor the succes of your iBatis query caches. You could do that (probably, never tried) with JMX but I was looking for a simple and clean solution. I already did the same in a [Wicket](http://wicket.apache.org) application but I didn't want too many extra dependencies. So I wrote a simple Servlet which accesses the Spring context and prints the cachemodels with their hitratio. Actually pretty simple, it took a lot more time to create a db schema with data and writing some code which uses a cache :).

Simple screenshot:

![picture-2](http://tigrou.nl/wp-content/uploads/2009/04/picture-2.png)



## **The actual code**



My apologees for the horrible code layout, I'm searching for another code preview thingy for Wordpress....



### web.xml



[sourcecode language='xml']


  ibatiscachemodel
  
  
    org.springframework.web.context.ContextLoaderListener
  
  
    contextConfigLocation
    /WEB-INF/applicationContext.xml
  
  
    myhttprequesthandler
    org.springframework.web.context.support.HttpRequestHandlerServlet
  
  
    myhttprequesthandler
    /CacheServlet
  

[/sourcecode]



### applicationContext.xml



[sourcecode language='xml']



	
		
		
		
		
	

	
		
		
	

	
		
		
	


	
		
	


	
		
	

	
		
	

[/sourcecode]



### HelperClassDaoImpl (exposing the cachemodels from the SqlMapClient)


[sourcecode language='java']
package nl.tigrou.test.persistence;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

import org.springframework.orm.ibatis.support.SqlMapClientDaoSupport;

import com.ibatis.sqlmap.engine.impl.ExtendedSqlMapClient;
import com.ibatis.sqlmap.engine.impl.SqlMapExecutorDelegate;

public class HelperDaoImpl extends SqlMapClientDaoSupport implements HelperDao {

	public List getCacheModels() {
		final ExtendedSqlMapClient ext = (ExtendedSqlMapClient) getSqlMapClient();

		SqlMapExecutorDelegate dlgt = ext.getDelegate();
		Iterator iter = dlgt.getCacheModelNames();
		List models = new ArrayList();

		while (iter.hasNext())
			models.add(iter.next());
		return models;
	}

	 public SqlMapExecutorDelegate getSqlMapExecutorDelegate()
	  {
	    final ExtendedSqlMapClient ext = (ExtendedSqlMapClient) getSqlMapClient();
	    return ext.getDelegate();
	  }

}
[/sourcecode]



### MyHttpRequestHandler (inheriting from Spring's HttpRequestHandler to access the Spring context)



[sourcecode language='java']
package nl.tigrou.test;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import nl.tigrou.test.persistence.HelperDao;
import nl.tigrou.test.service.CountryService;

import org.springframework.web.HttpRequestHandler;

public class MyHttpRequestHandler implements HttpRequestHandler {

	private CountryService countryService;
	private HelperDao helper;
		
	public void handleRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		
		response.setContentType("text/html");
		PrintWriter pw = new PrintWriter (response.getOutputStream());

		pw.println("");
	    pw.println("");
	    pw.println("

# Cachemodels

");
	    
	    countryService.getCountries();
	    
	    pw.println("<table >");
	    
	    Double ratio;
	    for (String name: helper.getCacheModels()) {
	    	ratio = helper.getSqlMapExecutorDelegate().getCacheModel(name).getHitRatio();
			pw.println("<tr >
<td >" + name + "
</td>
<td >" + ((ratio.isNaN())? 0 : ratio) + "
</td></tr>");
		}
	    pw.println("</table>");
	    
	    pw.println("");
	    pw.flush();
	}

	public void setCountryService(CountryService countryService) {
		this.countryService = countryService;
	}

	public CountryService getCountryService() {
		return countryService;
	}

	public void setHelper(HelperDao helper) {
		this.helper = helper;
	}

	public HelperDao getHelper() {
		return helper;
	}
}
[/sourcecode]

Of course there are several other files (classes and iBatis files) but those are not really interesting. I've zipped my eclipse project and uploaded it here. [Download](http://tigrou.nl/wp-content/uploads/2009/04/ibatiscachemodel.zip)
