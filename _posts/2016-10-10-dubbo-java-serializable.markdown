---
layout: post
title:  "dubbo 2.8.4 反序列化异常类失败"
date:   2016-10-21
categories: dubbo  java serializable kyro bug
---

在dubbo 2.8.4 版本下，provider丢出一个runtime异常，consumer反序列化该异常失败丢出：

~~~
Serialization trace:
stackTrace (cn.xxx.user.exception.UserBusinessException)
java.io.IOException: com.esotericsoftware.kryo.KryoException: Error during Java deserialization.
Serialization trace:
stackTrace (cn.xxx.user.exception.UserBusinessException)
	at com.alibaba.dubbo.common.serialize.support.kryo.KryoObjectInput.readObject(KryoObjectInput.java:127)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DecodeableRpcResult.decode(DecodeableRpcResult.java:94)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DecodeableRpcResult.decode(DecodeableRpcResult.java:117)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboCodec.decodeBody(DubboCodec.java:98)
	at com.alibaba.dubbo.remoting.exchange.codec.ExchangeCodec.decode(ExchangeCodec.java:134)
	at com.alibaba.dubbo.remoting.exchange.codec.ExchangeCodec.decode(ExchangeCodec.java:95)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboCountCodec.decode(DubboCountCodec.java:46)
	at com.alibaba.dubbo.remoting.transport.netty.NettyCodecAdapter$InternalDecoder.messageReceived(NettyCodecAdapter.java:134)
	at org.jboss.netty.channel.SimpleChannelUpstreamHandler.handleUpstream(SimpleChannelUpstreamHandler.java:70)
	at org.jboss.netty.channel.DefaultChannelPipeline.sendUpstream(DefaultChannelPipeline.java:564)
	at org.jboss.netty.channel.DefaultChannelPipeline.sendUpstream(DefaultChannelPipeline.java:559)
	at org.jboss.netty.channel.Channels.fireMessageReceived(Channels.java:268)
	at org.jboss.netty.channel.Channels.fireMessageReceived(Channels.java:255)
	at org.jboss.netty.channel.socket.nio.NioWorker.read(NioWorker.java:88)
	at org.jboss.netty.channel.socket.nio.AbstractNioWorker.process(AbstractNioWorker.java:109)
	at org.jboss.netty.channel.socket.nio.AbstractNioSelector.run(AbstractNioSelector.java:312)
	at org.jboss.netty.channel.socket.nio.AbstractNioWorker.run(AbstractNioWorker.java:90)
	at org.jboss.netty.channel.socket.nio.NioWorker.run(NioWorker.java:178)
	at org.jboss.netty.util.ThreadRenamingRunnable.run(ThreadRenamingRunnable.java:108)
	at org.jboss.netty.util.internal.DeadLockProofWorker$1.run(DeadLockProofWorker.java:42)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: com.esotericsoftware.kryo.KryoException: Error during Java deserialization.
Serialization trace:
stackTrace (cn.xxx.user.exception.UserBusinessException)
	at com.esotericsoftware.kryo.serializers.JavaSerializer.read(JavaSerializer.java:42)
	at com.esotericsoftware.kryo.Kryo.readObjectOrNull(Kryo.java:745)
	at com.esotericsoftware.kryo.serializers.DefaultArraySerializers$ObjectArraySerializer.read(DefaultArraySerializers.java:357)
	at com.esotericsoftware.kryo.serializers.DefaultArraySerializers$ObjectArraySerializer.read(DefaultArraySerializers.java:276)
	at com.esotericsoftware.kryo.Kryo.readObjectOrNull(Kryo.java:745)
	at com.esotericsoftware.kryo.serializers.ObjectField.read(ObjectField.java:113)
	at com.esotericsoftware.kryo.serializers.FieldSerializer.read(FieldSerializer.java:507)
	at com.esotericsoftware.kryo.Kryo.readClassAndObject(Kryo.java:776)
	at com.alibaba.dubbo.common.serialize.support.kryo.KryoObjectInput.readObject(KryoObjectInput.java:125)
	... 22 more
Caused by: java.io.StreamCorruptedException: invalid stream header: 79737200
	at java.io.ObjectInputStream.readStreamHeader(ObjectInputStream.java:806)
	at java.io.ObjectInputStream.<init>(ObjectInputStream.java:299)
	at com.esotericsoftware.kryo.serializers.JavaSerializer.read(JavaSerializer.java:40)
	... 30 more
~~~

debuger该问题发现是kyro2.22的序列号bug，查找其最新版本2.24.0，替换依赖,测试通过


