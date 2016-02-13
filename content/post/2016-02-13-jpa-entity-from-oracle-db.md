---
author: Rob
date: 2016-02-13 13:00:00+00:00
layout: post
slug: jpa-entity-from-oracle-db
title: Generate some helper code for a JPA Entity based on an Oracle database
categories:
- java
- jpa
- oracle
---

Being a lazy developer, I don't like writing repetitive code and I think none of the developers enjoy it. So as being one, you start looking into options to avoid those tasks. This time, I looked into a reverse engineer tool for JPA entities. There are tools around and also IDE's include fancy wizards to do so but so far I didn't like them. They mostly were either verbose, slow or just not doing what I needed. And what I needed was pretty simple, generating a list of fields based on a database table. As I'm using Oracle, I don't have to think about any other vendors. So I wrote a simple though effective select query that gives me all I need.

I'd thought sharing would be good, but I have to mention a few assumptions I made:

* it's based on Oracle metadata tables
* it's assuming that unique constraints have the name: u_ + column name
* it's assuming that number(1,0) are booleans
* it uses the primitive type boolean to generate getter like isBoolean() instead of getIsBoolean() and to avoid null values

## The query
You probably just need to change the table_name where clause at the bottom.

```sql
SELECT
  '@Column(' || z.col_def || z.col_name ||
     z.col_precision || z.col_scale || z.col_length || z.col_unique || ') private ' ||
     z.java_type || ' ' || z.java_name || ';' AS java_code
FROM
(
  SELECT
    y.java_name,
    y.column_name,
    y.col_def,
    y.col_name,
    y.data_type,
    y.java_type,
    y.data_length,
    y.data_precision,
    y.data_scale,
    y.col_length,
    y.col_scale,
    y.col_precision,
    nvl2(y.data_unique, ', unique = true', '') AS col_unique
  FROM
  (
    SELECT
      x.java_name,
      x.column_name,
      'columnDefinition = "' || x.column_name || '"' AS col_def,
      ', name = "' || x.column_name || '"' AS col_name,
      x.data_type,
      x.java_type,
      x.data_length,
      x.data_precision,
      x.data_scale,
      nvl2(x.string_length, ', length = ' || x.string_length, '') AS col_length,
      nvl2(x.data_scale, ', scale = ' || x.data_scale, '') AS col_scale,
      nvl2(x.data_precision, ', precision = ' || x.data_precision, '') AS col_precision,
      CASE WHEN x.java_type = 'boolean' THEN x.data_unique ELSE null END AS data_unique
    FROM
    (
      SELECT
        lower(utc.column_name) AS column_name, utc.data_type, utc.data_length, utc.data_precision, utc.data_scale,
        decode(utc.data_type, 'VARCHAR2', 'String', 'NUMBER', (CASE WHEN utc.data_precision = 1 AND utc.data_scale = 0  THEN 'boolean' WHEN utc.data_precision < 10 THEN 'Integer' ELSE 'Long' END), 'DATE', 'Date', 'Object') AS java_type,
        regexp_replace(replace(initcap(utc.column_name),'_'),'(.{0}).(.+)','\1' || lower(substr(replace(initcap(utc.column_name),'_'),1,1)) || '\2') AS java_name,
        decode(utc.data_type, 'VARCHAR2', utc.data_length, null) AS string_length,
        (
          SELECT decode(count(1), 1, 'true', null)
          FROM user_ind_columns uic
          JOIN user_constraints uc ON (uc.table_name = uic.table_name)
          WHERE uic.table_name='CUSTOMER'
          AND uc.constraint_name = 'U_' || uic.column_name
          AND uc.constraint_type = 'U'
          AND uic.column_name = utc.column_name
        ) AS data_unique
      FROM user_tab_columns utc
      WHERE utc.table_name = 'CUSTOMER'
      ORDER BY utc.column_name
    ) x
  ) y
) z;
```

## The result
And this gives you the below result. Just generate some getters and setters with your IDE and you're good to go.

```java
@Column(columnDefinition = "customer_id", name = "customer_id", unique = true)
private Long customerId;

@Column(columnDefinition = "account_nr", name = "account_nr", precision = 10, scale = 0)
private Integer accountNr;

@Column(columnDefinition = "department", name = "department", length = 100)
private String department;

@Column(columnDefinition = "firstname", name = "firstname", length = 50)
private String firstname;

@Column(columnDefinition = "lastname", name = "lastname", length = 100)
private String lastname;

@Column(columnDefinition = "happy", name = "happy")
private boolean happy;
```



