---
layout: post
title: Liquibase "Could not acquire change log lock."
category : [Liquibase]
tagline: "Supporting tagline"
tags : [Liquibase]
---
{% include JB/setup %}
# Liquibase "Could not acquire change log lock."
--- 

Problem: 
``` 
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'liquibase' defined in class path resource [stash-context.xml]: Invocation of init method failed; nested exception is liquibase.exception.LockException: Could not acquire change log lock.  Currently locked by fe80:0:0:0:a00:27ff:fe6a:c787%2 (fe80:0:0:0:a00:27ff:fe6a:c787%2) since 12/6/12 2:08 PM 
```

<!--break-->

Cause
The DATABASECHANGELOGLOCK table has not been updated with the release lock information.
The likely cause of this is that the Stash instance was forced to quit while it was trying to migrate the database schema after an upgrade, with the consequence that the lock was not released. You should always wait for Stash to start up sufficiently for it to provide error messages if there are schema migration problems – never assume that it has hung and kill the process.
Another possible cause of not releasing the lock is that Stash was forced to quit while performing automatic application setup.

Resolution
> Stash needs an exclusive lock on the DATABASECHANGELOGLOCK table in order to start so take care to disconnect / close the tool used to update the database. If you do not do this you may experience Stash hanging on start up with no errors in the logs and no response from the web server. 

 MySQL, SQL Server and Oracle: 
```
UPDATE DATABASECHANGELOGLOCK SET LOCKED=0, LOCKGRANTED=null, LOCKEDBY=null where ID=1;
```

PostgreSQL: 
```
UPDATE DATABASECHANGELOGLOCK SET LOCKED=0, LOCKGRANTED=null, LOCKEDBY=null where ID=1;
```

HSQLDB: 
```
1- Shutdown Stash
2- Open your <STASH_HOME>/data/db.script and look for a string like:
INSERT INTO DATABASECHANGELOGLOCK VALUES(1,TRUE,'2012-06-12 14:08:00.982000','fe80:0:0:0:a00:27ff:fe6a:c787%2 (fe80:0:0:0:a00:27ff:fe6a:c787%2)')
```


> 转发自：[Stash Does Not Start - Could not acquire change log lock](https://confluence.atlassian.com/stashkb/stash-does-not-start-could-not-acquire-change-log-lock-313464945.html)


