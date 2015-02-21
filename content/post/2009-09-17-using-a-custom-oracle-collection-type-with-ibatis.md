---
author: admin
comments: true
date: 2009-09-17 18:28:24+00:00
layout: post
slug: using-a-custom-oracle-collection-type-with-ibatis
title: Using a custom Oracle collection type with iBatis
wordpress_id: 162
categories:
- Software
tags:
- iBatis
- java
- LinkedIn
---

Some months ago I came across a problem with the more complex custom Oracle types in combination with iBatis. I thought that it would be nice to share it with you. In my case I had to link a list of objects in Java to an array of structs in SQL (Oracle). 

The biggest part of the trick is inside a custom type handler, which is actually a helper class with two important methods, the setParameter and the getResult. setParameter if for mapping parameters before firing the query and the getResult is for processing the result (if you use a custom type as an OUT parameter). For the example I used a Quantity class which has two properties, id and quantity. My Oracle types are a struct (Quantity) and a table of that struct (QuantityList)

The code will show you the rest, because once you know how to deal with this, other cases are quite easy.

[code lang="java"]
package nl.tigrou.test;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;

import java.util.List;
import oracle.jdbc.driver.OracleTypes;
import oracle.sql.ARRAY;
import oracle.sql.ArrayDescriptor;
import oracle.sql.STRUCT;
import oracle.sql.StructDescriptor;

import org.springframework.jdbc.support.nativejdbc.C3P0NativeJdbcExtractor;
import org.springframework.jdbc.support.nativejdbc.CommonsDbcpNativeJdbcExtractor;

import com.ibatis.sqlmap.client.extensions.ParameterSetter;
import com.ibatis.sqlmap.client.extensions.ResultGetter;
import com.ibatis.sqlmap.client.extensions.TypeHandlerCallback;
import com.ibatis.sqlmap.engine.type.JdbcTypeRegistry;
import com.mchange.v2.c3p0.impl.NewProxyConnection;

public class QuantityListTypeHandler implements TypeHandlerCallback
{
  
  private static final String QUANTITY = "QUANTITY";
  private static final String QUANTITYLIST = "QUANTITYLIST";
  
  static
  {
    JdbcTypeRegistry.setType(QUANTITY, OracleTypes.STRUCT);
    JdbcTypeRegistry.setType(QUANTITYLIST, OracleTypes.ARRAY);
  };  

  public void setParameter(ParameterSetter setter, Object parameter) throws SQLException
  {
    try
    {
      List quantities = (List) parameter;
      
      Connection conn = setter. getPreparedStatement(). getConnection();
      if(conn instanceof NewProxyConnection)
      conn = new C3P0NativeJdbcExtractorImpl().getNativeConnection(conn);
      
      StructDescriptor quantityStruct = StructDescriptor.createDescriptor(QUANTITY, conn);
      ArrayDescriptor quantityList = ArrayDescriptor.createDescriptor(QUANTITYLIST, conn);
      
      STRUCT[] elements = new STRUCT[quantities == null ? 0 : quantities.size()];
      
      for (int count = 0; count < elements.length; count++)
      {
        Quantity quantity = quantities.get(count);
        
        elements[count] = new STRUCT(quantityStruct, conn, new Object[] { quantity.getId(), quantity.getQuantity() });
      }
      
      ARRAY array = new ARRAY(quantityList, conn, elements);
      
      setter.setArray(array);
    } catch (SQLException sqle)
    {
      throw sqle;
    }
  }
  
  private class C3P0NativeJdbcExtractorImpl extends C3P0NativeJdbcExtractor
  {
    public Connection getNativeConnection(Connection con) throws SQLException
    {
      return doGetNativeConnection(con);
    }
  }
  
  public Object getResult(ResultGetter getter) throws SQLException
  {
    ARRAY array = (oracle.sql.ARRAY) getter.getArray();
    ResultSet rs = array.getResultSet();
    List quantities = new ArrayList();
    while (rs != null &amp;amp;&amp;amp; rs.next())
    {
     STRUCT struct = (STRUCT) rs.getObject(2);
     Object[] attribs = struct.getAttributes();
     Quantity quantity = new Quantity();
     quantity.setId(((java.math.BigDecimal) attribs[0]).longValue());
     quantity.setQuantity(((java.math.BigDecimal) attribs[1]).intValue());
     quantities.add(quantity);
    }
    return quantities;
  }
  
  public Object valueOf(String value)
  {
    if (value == null)
      return new ArrayList();
    return value;
  }
}
```

And the corresponding query in one of your xml mapping files. The magic is all inside the inline parameter.

[code lang="sql"]
SELECT * FROM sometable x
LEFT JOIN TABLE (
  CAST
  (
    #quantities,handler=quantityListTypeHandler,jdbcType=STRUCT,javaType=Quantity# AS QuantityList
  )
) s ON (s.identifier = x.id)
```

**BTW, if you haven't noticed it yet, [iBatis 3](http://ibatis.apache.org/) is about to release. Check out the release candidates because they did an awesome job!**


