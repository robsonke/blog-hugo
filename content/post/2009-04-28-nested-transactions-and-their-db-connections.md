---
author: admin
comments: true
date: 2009-04-28 20:21:39+00:00
layout: post
slug: nested-transactions-and-their-db-connections
title: Nested transactions and their db connections
wordpress_id: 118
categories:
- Software
tags:
- java
- LinkedIn
- spring
---

Last week I was researching the AOP config from an big application because I wasn't really happy with the way it was implemented right now. The idea was to improve the performance by reducing the number of calls initiated by the AOP proxy. But there was a thing I wasn't really sure about. How does Spring handle transactions when there are nested service class calls from one service to the other?

When I don't how something exactly works, I create a small sample application. Nothing shocking, just a sample app which gives me the possibility to look to the different [transaction propagations.](http://static.springframework.org/spring/docs/2.5.x/reference/transaction.html#tx-propagation) And who knows if it could help other people as well, so I added the idea to my blog.

Let's start with the Spring application context:

``` xml

<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">

<aop:aspectj-autoproxy/>

<tx:advice id="txAdvice" transaction-manager="txManager">

<tx:attributes>
<!-- normally you would exclude read actions from required transactions but I only wrote a get method for testing purpose -->
<!-- tx:method name="get*" propagation="SUPPORTS" read-only="true" /-->
<tx:method name="*" propagation="REQUIRED" />
</tx:attributes>
</tx:advice>

<aop:config>
<aop:pointcut id="serviceOperation" expression="bean(*Service)"/>
<aop:advisor advice-ref="txAdvice" pointcut-ref="serviceOperation"/>
</aop:config>

<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource"/>
</bean>

<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
<property name="driverClass" value="com.mysql.jdbc.Driver" />
<property name="jdbcUrl" value="jdbc:mysql://localhost/world" />
<property name="user" value="" />
<property name="password" value="" />
</bean>

<bean id="sqlMapClient" class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
<property name="configLocation" value="WEB-INF/sqlmapconfig.xml" />
<property name="dataSource" ref="dataSource" />
</bean>

<bean id="countryService" class="nl.tigrou.test.service.CountryServiceImpl">
<property name="countryDao" ref="countryDao" />
<property name="cityService" ref="cityService" />
</bean>

<bean id="countryDao" class="nl.tigrou.test.persistence.CountryDaoImpl">
<property name="sqlMapClient" ref="sqlMapClient" />
</bean>

<bean id="cityService" class="nl.tigrou.test.service.CityServiceImpl">
<property name="cityDao" ref="cityDao" />
</bean>

<bean id="cityDao" class="nl.tigrou.test.persistence.CityDaoImpl">
<property name="sqlMapClient" ref="sqlMapClient" />
</bean>

</beans>
```

The only interesting thing in here is the tx:advice element defining the right propagation settings.

Now the pieces of Java code:

``` java

// grab countries
List countries = countryService.getCountries();

// this is the getCountries method
// which is btw a good example of how you shouldn't do this with iBatis
// always try to avoid N+1 selects!
List countries = countryDao.getCountries();
for(Country country: countries){
    List cities = cityService.getCities(country.getCode());
    country.setCities(cities);
}

```

That's all! Now enable the debug logging on for example the org.springframework.jdbc.datasource package and you'll see something like this:

    2009-04-28 22:07:47,591 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Creating new transaction with name [nl.tigrou.test.service.CountryService.getCountries]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT<br></br>
    2009-04-28 22:07:48,684 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Acquired Connection [com.mchange.v2.c3p0.impl.NewProxyConnection@46d0cc] for JDBC transaction<br></br>
    2009-04-28 22:07:48,821 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,829 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,832 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,834 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,837 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,839 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,840 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:48,842 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Participating in existing transaction<br></br>
    2009-04-28 22:07:49,376 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Initiating transaction commit<br></br>
    2009-04-28 22:07:49,376 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Committing JDBC transaction on Connection [com.mchange.v2.c3p0.impl.NewProxyConnection@46d0cc]<br></br>
    2009-04-28 22:07:49,376 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceTransactionManager - Releasing JDBC Connection [com.mchange.v2.c3p0.impl.NewProxyConnection@46d0cc] after transaction<br></br>
    2009-04-28 22:07:49,378 [http-8080-1] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Returning JDBC Connection to DataSource
  

As you see, Spring initializes a transaction and when another transaction is about to start, Spring runs these other db calls inside the existing transaction. Of course you can heavily influence this by changing the transaction propagation.

**Conclusion: Spring did the job exactly as expected and I'm pretty happy with the solution as well.**

btw, next subject is probably the use of Spring's ThreadLocalTargetSource**.
**
