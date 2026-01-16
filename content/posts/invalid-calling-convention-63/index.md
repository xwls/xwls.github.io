---
title: Invalid calling convention 63
date: 2022-05-06 08:58:21+08:00
categories:
- Java
tags:
- java
- jna
- exception
summary: '在 Java 中使用 jna 调用本地库的方法时，经常会有回调方法，回调方法在 Linux 和 Windows 中是有区别的，使用不正确会抛出异常：java.lang.IllegalArgumentException:
  Invalid calling convention 63'
---

## 异常日志

```text
Caused by: java.lang.IllegalArgumentException: Invalid calling convention 63
 at com.sun.jna.Native.createNativeCallback(Native Method) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.jna.CallbackReference.<init>(CallbackReference.java:263) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.jna.CallbackReference.getFunctionPointer(CallbackReference.java:449) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.jna.CallbackReference.getFunctionPointer(CallbackReference.java:426) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.jna.Function.convertArgument(Function.java:558) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.jna.Function.invoke(Function.java:345) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.jna.Library$Handler.invoke(Library.java:265) ~[jna-5.4.0.jar!/:5.4.0 (b0)]
 at com.sun.proxy.$Proxy105.CLIENT_Init(Unknown Source) ~[na:na]
 at com.xs.dahua.module.LoginModule.init(LoginModule.java:37) ~[classes!/:0.0.1-SNAPSHOT]
 at com.xs.dahua.DahuaJavaServiceApplication.run(DahuaJavaServiceApplication.java:78) [classes!/:0.0.1-SNAPSHOT]
 at org.springframework.boot.SpringApplication.callRunner(SpringApplication.java:812) [spring-boot-2.5.12.jar!/:2.5.12]
 ... 13 common frames omitted
```

## 解决办法

* Windows 下继承自`com.sun.jna.win32.StdCallLibrary.StdCallCallback`。

```java
public interface fDisConnect extends StdCallCallback {
    public void invoke(LLong lLoginID, String pchDVRIP, int nDVRPort, Pointer dwUser);
}
```

* Linux 下继承自`com.sun.jna.Callback`。

```java
public interface fDisConnect extends Callback {
    public void invoke(LLong lLoginID, String pchDVRIP, int nDVRPort, Pointer dwUser);
}