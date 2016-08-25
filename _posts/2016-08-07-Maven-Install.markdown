---
layout:     post
title:      "Maven安装及简单实用"
date:       2016-08-07
author:     "Sim"
catalog: false
tags:
    - Java
    - Maven
---

# 安装
在安装Maven之前，需要安装Java，由于我的机子已经装过了，这里就不说了。

Maven的下载地址：[http://maven.apache.org/download.html](http://maven.apache.org/download.html)

下载后，解压到`/usr/local/`

然后设置Maven的环境变量，打开命令行工具

```
export M2_HOME=/usr/local/your-folder
export M2=$M2_HOME/bin
export MAVEN_OPTS="-Xms256m -Xmx512m"
```

完成上述命令后，使用`mvn --version`进行测试，成功的话将会输出

```
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /usr/local/apache-maven-3.3.9
Java version: 1.8.0_60, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_60.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.11.6", arch: "x86_64", family: "mac"
```

在这里，我遇到个问题，因为安装Java的时候并没有设置Java的变量，所以导致测试的时候提示

```
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/maven/cli/MavenCli : Unsupported major.minor version 51.0
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClassCond(ClassLoader.java:637)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:621)
    at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:141)
    at java.net.URLClassLoader.defineClass(URLClassLoader.java:283)
    at java.net.URLClassLoader.access$000(URLClassLoader.java:58)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:197)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClassFromSelf(ClassRealm.java:401)
    at org.codehaus.plexus.classworlds.strategy.SelfFirstStrategy.loadClass(SelfFirstStrategy.java:42)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.unsynchronizedLoadClass(ClassRealm.java:271)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClass(ClassRealm.java:254)
    at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClass(ClassRealm.java:239)
    at org.codehaus.plexus.classworlds.launcher.Launcher.getMainClass(Launcher.java:144)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced(Launcher.java:266)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launch(Launcher.java:229)
    at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode(Launcher.java:415)
    at org.codehaus.plexus.classworlds.launcher.Launcher.main(Launcher.java:356)
```

解决的办法是设置`JAVA_HOME`这个变量即可:`export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home`

#

