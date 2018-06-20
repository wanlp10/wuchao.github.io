---
layout: post
title: Liquibase 的学习和使用
category : [JavaEE, Liquibase]
tagline: "Supporting tagline"
tags : [Liquibase]
---
{% include JB/setup %}
# Liquibase 的学习和使用（Maven + SpringBoot + Liquibase）
---

> [官网](http://www.liquibase.org/documentation/index.html) 
>
> [使用 LiquiBase 管理数据库的迁移 - 推酷](http://www.tuicool.com/articles/B7ziIrv)
> 
> [在 Web 项目中使用 LiquiBase 实现数据库自动更新](http://blog.csdn.net/jianyi7659/article/details/7804144)
> 
> [让开发自动化: 实现自动化数据库迁移](https://www.ibm.com/developerworks/cn/java/j-ap08058/index.html)


## 引入依赖（其他依赖根据需要引入）

```
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

<!--break-->

## 配置 SpringBoot 的 Liquibase 属性

```
liquibase.change-log=classpath:db/changelog/db.changelog-master.yaml
liquibase.contexts= # runtime contexts to use
liquibase.default-schema= # default database schema to use
liquibase.drop-first=false
liquibase.enabled=true
```

以上属性全部默认即可，但是如果 "liquibase.change-log" 属性也是默认的话， changelog 文件就要放在默认的位置上（classpath:db/changelog/db.changelog-master.yaml），并且文件名称为 "db.changelog-master.yaml" 。

配置 datasource 指定  DATABASECHANGELOG 表和 DATABASECHANGELOGLOCK 表的存储数据库：

```
spring.datasource.url=jdbc:mysql://localhost:3306/xxx
spring.datasource.username=root
spring.datasource.password=
```

在 SpringBoot 中也可以不指定 datasource，默认使用内存数据库，这时需要引入内存数据库相关的依赖。


## 创建 changelog 文件

Liquibase 日志文件支持 XML, YAML, JSON 等多种格式，这里选择 YAML 格式，官网示例如下：

```
databaseChangeLog:
  - preConditions:
    - runningAs:
        username: liquibase

  - changeSet:
      id: 1
      author: nvoxland
      changes:
        - createTable:
            tableName: person
            columns:
              - column:
                  name: id
                  type: int
                  autoIncrement: true
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: firstname
                  type: varchar(50)
              - column:
                  name: lastname
                  type: varchar(50)
                  constraints:
                    nullable: false
              - column:
                  name: state
                  type: char(2)

  - changeSet:
      id: 2
      author: nvoxland
      changes:
        - addColumn:
            tableName: person
            columns:
              - column:
                  name: username
                  type: varchar(8)
```

上面 changeSet 的 "changes" 属性中使用了 "createTable" 和 "addColumn" 表明该 changeSet 的操作，更多操作可以在 [http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd](http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd) 中查找。

我创建了 changelog 文件后，按照规范写了changeSet，但是在启动后报了以下错误：

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'liquibase' defined in class path resource [org/springframework/boot/autoconfigure/liquibase/LiquibaseAutoConfiguration$LiquibaseConfiguration.class]: Invocation of init method failed; nested exception is liquibase.exception.DatabaseException: NULL not allowed for column "ID"; SQL statement:
INSERT INTO PUBLIC.DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, DESCRIPTION, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES (NULL, NULL, 'classpath:db/development/changelog/db.changelog-master.yaml', NOW(), 1, '7:d41d8cd98f00b204e9800998ecf8427e', 'empty', '', 'EXECUTED', NULL, NULL, '3.5.1', '6904122279') 

Caused by: liquibase.exception.DatabaseException: NULL not allowed for column "ID"; SQL statement:
INSERT INTO PUBLIC.DATABASECHANGELOG (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, DESCRIPTION, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES (NULL, NULL, 'classpath:db/development/changelog/db.changelog-master.yaml', NOW(), 1, '7:d41d8cd98f00b204e9800998ecf8427e', 'empty', '', 'EXECUTED', NULL, NULL, '3.5.1', '6904122279')
```

## 启动项目

顺利启动项目过后，Liquibase 会在数据库中生成 DATABASECHANGELOG 表和 DATABASECHANGELOGLOCK 表。

如果修改了原来的 changeSet ，则在下次启动时会去检查每一个 changeSet，即使 id 和 author 没有改变，但是由于 MD5 校验值改变了，启动项目时会报以下错误，changeSet 标签内部修改的内容不会更新到数据库中。
```
Caused by: liquibase.exception.ValidationFailedException: Validation Failed:
     1 change sets check sum
          classpath:db/develop/db.changelog-master.yaml::47::xxx was: 7:f2ea6ec961a20a0c7af2c99ada7a75d2 but is now: 7:1fd94acb9eb54da3a9f5695189a99871
```
（其中 47 表示 changeSet 的 id 属性值，xxx 表示 changeSet 的 author 属性值。）

如果修改了原来的 include 标签引用的 sql 文件，该标签在数据库中的记录的 MD5 校验值也会改变，项目启动时会报以下错误，sql 文件中修改的内容不会更新到数据库中。
```
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'liquibase' defined in class path resource [org/springframework/boot/autoconfigure/liquibase/LiquibaseAutoConfiguration$LiquibaseConfiguration.class]: Invocation of init method failed; nested exception is liquibase.exception.ValidationFailedException: Validation Failed:
     1 change sets check sum
          classpath:db/production/import.sql::raw::includeAll was: 7:146a339a9d60827a9b0aa64c6337b0ef but is now: 7:6d2747840d5c1195eef9e709011567b8
```
（这时可以到配置的数据库的 DATABASECHANGELOG 表中找到出错的 MD5 校验值所在位置。）

具体参考 [使用 LiquiBase 管理数据库的迁移](http://www.tuicool.com/articles/B7ziIrv)。

如果在配置中使用了 `spring.jpa.properties.hibernate.hbm2ddl.import_files` 指定 sql 脚本文件，可以设置：

```
spring.jpa.hibernate.ddl-auto=validate
```

这样项目启动时就不会执行指定的 sql 脚本了，这时如果要引入 sql 脚本，可以在 changelog 文件中使用 include 标签引入。
将该属性设置为 validate 在使用像 mysql 这样的可持久化数据库是非常必要的，因为使用了可持久化数据库，下次启动项目时会保留
以前持久化到数据库中的所有数据，如果不设置该属性将使用了 `spring.jpa.properties.hibernate.hbm2ddl.import_files`
指定的 sql 脚本文件过滤掉会污染数据库。

> [hibernate.hbm2ddl.auto 配置详解](http://www.cnblogs.com/feilong3540717/archive/2011/12/19/2293038.html)
  [Hibernate 配置详解 (12) 其实我也不想用这么土的名字](http://blog.csdn.net/stefwu/article/details/10584161)
