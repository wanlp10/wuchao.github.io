---
layout: post
title: Difference between path and classpath in Java
category : [JavaEE]
tagline: "Supporting tagline"
tags : [Java]
---
{% include JB/setup %}
# Difference between path and classpath in Java
---

> [Difference between path and classpath in Java](https://www.sitesbay.com/java/difference-between-path-and-classpath)

### path   
`path` variable is set for providing path for all Java tools like java, javac, javap, javah, jar, appletviewer. In Java to run any program we use java tool and for compile Java code use javac tool. All these tools are available in bin folder so we set path upto bin folder.

### classpath  
`classpath` variable is set for providing path of all Java classes which is used in our application. All classes are available in lib/rt.jar so we set classpath upto lib/rt.jar.

### Difference between path and classPath
path	classpath
path variable is set for providing path for all java tools like java, javac, javap, javah, jar, appletviewer	classpath variable is set for provide path of all java classes which is used in our application.

### JDK Folder Hierarchy  

![](/images/2017-10-26-jdk-folder-hierarchy.png)


Path variable is set for use all the tools like java, javac, javap, javah, jar, appletviewer etc. 

Example: 
``` 
"C:\Program Files\Java\jdk1.6.0\bin"
```

![](/images/2017-10-26-path-configuration.png)

All the tools are present in bin folder so we set path upto bin folder.

Classpath variable is used to set the path for all classes which is used in our program so we set classpath upto rj.jar. in rt.jar file all the .class files are present. When we decompressed rt.jar file we get all .class files. 

Example: 
``` 
"C:\Program Files\Java\jre1.6.0\jre\lib\rt.jar"
```
In above rt.jar is a jar file where all the .class files are present so we set the classpath upto rt.jar. 

![](/images/2017-10-26-classpath-configuration.png) 
 

