---
layout: post
title: Java Serializable
category : [问题记录]
tagline: "Supporting tagline"
tags : [Java, Serializable]
---
{% include JB/setup %}
# Java Serializable
--- 

> [serialVersionUID的作用](http://blog.csdn.net/zzjjiandan/article/details/32336079)   
> 
> [Java基础篇 - Serializable与serialVersionUID的简单说明](http://blog.csdn.net/zhengliebin/article/details/60869629)
> 
> [关于Java Serial Version UID的一些说明](http://blog.csdn.net/u012364372/article/details/51210693) 
> 
> [Java序列化与static](http://blog.csdn.net/yangxiangyuibm/article/details/43227457) 

在网络传输Java对象、将Java对象存储到文件、将Java对象以BLOB形式存储到数据库中时，需要对Java对象进行序列化及反序列化，标准模式是实现Serializable接口。 
实现上述接口时，需要提供一个Serial Version UID，该UID用于标识类的版本。一个对象被序列化后，只要其版本不变，都可以进行反序列化，一旦 
改变造成版本不一致，会抛出InvalidClassException异常。 
建议显示定义UID，如果不显示定义，JVM会自动产生一个值，这个值和编译器的实现有关，不稳定，可能在不同JVM环境下出现反序列化抛出InvalidClassException异常的情况。 
在Eclipse中，提供两种方式显示定义UID，一种是“add default serial version ID”，默认值为1L；另一种是“add generated serial version ID”,默认值是一个很大的数，是根据 
类的具体属性而生成，当类属性有变动时，该值会更改。 
建议采用第一种自动生成方法，当对类进行了不兼容性修改时，需要修改UID。 
采用第二种方法时，如果修改了属性，不重新生成UID时，默认值是不会变的，也可以正常反序列化，但不推荐，毕竟UID的值与实际不符。 
对类进行兼容性和不兼容性修改的情况请参见 [Versioning of Serializable Objects](http://docs.oracle.com/javase/7/docs/platform/serialization/spec/version.html)。 
Hibernate的pojo类建议也采用上述方法，便于扩展。 
对于继承关系，父类实现序列化接口，子类可以继承接口的实现，但需显示定义UID，因为父类UID类型为private static，不可被继承，同时子类作为单独的类需要单独的UID。 

<!--break-->

对于继承关系，父类实现序列化接口，子类可以继承接口的实现，但需显示定义UID，因为父类UID类型为private static，不可被继承，同时子类作为单独的类需要单独的UID。
当一个类有父类有  serialVersionUID   
子类没有重写serialVersionUID，那么jvm会自动生成一个serialVersionUID 

### FAQ 
更新账户时调用了下面的方法：
``` 
public void updateAuthentication(String username, Collection<Authority> authorities) {
        Collection<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        for (Authority authority : authorities) {
            grantedAuthorities.add(new GrantedAuthority() {
                @Override
                public String getAuthority() {
                    return authority.getName();
                }
            });
        }
        org.springframework.security.core.userdetails.User user = new org.springframework.security.core.userdetails.User(username.trim(), "", grantedAuthorities);
        Authentication auth = new UsernamePasswordAuthenticationToken(user, null, grantedAuthorities);
        SecurityContextHolder.getContext().setAuthentication(auth);
    }
```
然后跳转页面时报错：
``` 
org.springframework.data.redis.serializer.SerializationException: Cannot deserialize; nested exception is org.springframework.core.serializer.support.SerializationFailedException: Failed to deserialize payload. Is the byte array a result of corresponding serialization for DefaultDeserializer?; nested exception is java.io.InvalidClassException: com.stone.investment.security.CustomUser; local class incompatible: stream classdesc serialVersionUID = 9141229518910167282, local class serialVersionUID = 1
	at org.springframework.data.redis.serializer.JdkSerializationRedisSerializer.deserialize(JdkSerializationRedisSerializer.java:82)
	at org.springframework.data.redis.core.AbstractOperations.deserializeHashValue(AbstractOperations.java:338)
	at org.springframework.data.redis.core.AbstractOperations.deserializeHashMap(AbstractOperations.java:282)
	at org.springframework.data.redis.core.DefaultHashOperations.entries(DefaultHashOperations.java:227)
	at org.springframework.data.redis.core.DefaultBoundHashOperations.entries(DefaultBoundHashOperations.java:102)
	at org.springframework.session.data.redis.RedisOperationsSessionRepository.getSession(RedisOperationsSessionRepository.java:432)
	at org.springframework.session.data.redis.RedisOperationsSessionRepository.onMessage(RedisOperationsSessionRepository.java:519)
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.executeListener(RedisMessageListenerContainer.java:249)
	at org.springframework.data.redis.listener.RedisMessageListenerContainer.processMessage(RedisMessageListenerContainer.java:239)
	at org.springframework.data.redis.listener.RedisMessageListenerContainer$1.run(RedisMessageListenerContainer.java:967)
	at java.lang.Thread.run(Thread.java:748)
Caused by: org.springframework.core.serializer.support.SerializationFailedException: Failed to deserialize payload. Is the byte array a result of corresponding serialization for DefaultDeserializer?; nested exception is java.io.InvalidClassException: com.stone.investment.security.CustomUser; local class incompatible: stream classdesc serialVersionUID = 9141229518910167282, local class serialVersionUID = 1
	at org.springframework.core.serializer.support.DeserializingConverter.convert(DeserializingConverter.java:78)
	at org.springframework.core.serializer.support.DeserializingConverter.convert(DeserializingConverter.java:36)
	at org.springframework.data.redis.serializer.JdkSerializationRedisSerializer.deserialize(JdkSerializationRedisSerializer.java:80)
	... 10 common frames omitted
Caused by: java.io.InvalidClassException: com.stone.investment.security.CustomUser; local class incompatible: stream classdesc serialVersionUID = 9141229518910167282, local class serialVersionUID = 1
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:687)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1880)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1746)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2037)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2282)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2206)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2064)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
	at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2282)
	at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2206)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2064)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:428)
	at org.springframework.core.serializer.DefaultDeserializer.deserialize(DefaultDeserializer.java:70)
	at org.springframework.core.serializer.support.DeserializingConverter.convert(DeserializingConverter.java:73)
	... 12 common frames omitted

```
Redis 对 ProviderManager 序列化失败，将该方法改为 static 后程序正常运行，因为 Java 不对 static 序列化。
