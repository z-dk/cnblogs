

大部分工具都是基于或者要用到`JMX`,所以程序启动时需要添加参数`-Dcom.sun.management.jmxremote`开启JMX管理功能,JDK6及以上版本默认开启

## jps

jps:JVM Process Status Tool,功能与ps类似,可以列出正在运行的虚拟机进程,并显示虚拟机执行的主类(Main class,main函数所在的类)名称以及这些进程的本地虚拟机唯一ID(`LVMID`, Local Virtual Machine Identifier),对于本地虚拟机来说,LVMID与操作系统的进程ID是一致的

**jps命令格式:**

jps [options] [hostid]

**jps执行示例:**

```sh
jps -l
17040 jdk.jcmd/sun.tools.jps.Jps
12508
```

**jps工具主要选项**

| 选项 | 作用                                         |
| ---- | -------------------------------------------- |
| -q   | 只输出`LVMID`,省略主类名称                   |
| -m   | 输出虚拟机进程启动时传递给主类main函数的参数 |
| -l   | 输出主类全名,如果是jar包则输出jar包路径      |
| -v   | 输出虚拟机进程启动的`JVM`参数                |

## jstat

jstat:JVM Statistics Monitoring Tool,是用于监视虚拟机各种运行状态信息的命令行工具,可以显示本地或者远程虚拟机进程中的类加载,内存,垃圾收集,即时编译等运行时数据

**jstat命令格式:**

jstat [option vmid [interval] [s|ms] [count]]

如果是本地虚拟机进程VMID与LVMID是一致的,如果是远程格式为:protocol://lvmid[@hostname:port/servername]

参数interval和count代表查询间隔和次数,如果省略则只查询一次,例如:

```sh
# 每250ms查询一次进程垃圾收集状况,一共查询20次
jstat -gc 12508 250 20
```

options选项代表希望查询的虚拟机信息,主要分三类:类加载,垃圾收集,运行期编译状况

| 选项              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| -class            | 监视类加载,卸载数量,总空间以及类加载耗时                     |
| -gc               | 监视Java堆状况,包括eden区,2个survivor区,老年代,永久代等容量,已用空间,垃圾收集时间合计等信息 |
| -gccapacity       | 监视内容与-gc基本相同,但输出主要关注Java堆各个区域使用到的最大最小空间 |
| -gcutil           | 监视内容与-gc基本相同,主要输出已使用空间占总空间的百分比     |
| -gccause          | 与-gcutil功能一样,额外输出导致上一次垃圾收集产生的原因       |
| -gcnew            | 监视新生代垃圾收集状况                                       |
| -gcnewcapacity    | 监视内容与-gcnew一样,主要关注使用的最大,最小空间             |
| -gcold            | 监视老年代垃圾收集状况                                       |
| -gcoldcapacity    | 与-gcold一样,主要关注使用的最大,最小空间                     |
| -gcpermcapacity   | 监视永久代的最大,最小空间                                    |
| -compiler         | 输出即时编译器编译过的方法,耗时等信息                        |
| -printcompilation | 输出已经被即时编译过的方法                                   |

## jinfo

jinfo:Configuration Info for Java 作用是实时查看和调整虚拟机的各项参数,使用jps -v可查看虚拟机启动时显示指定的参数列表,如果想知道未被显示指定的系统默认值,可以使用jinfo -flag,在JDK6之后jinfo加入了在运行期修改部分参数值的能力

**jinfo命令格式:**

jinfo [option] pid

```sh
jinfo -flag CMSInitiatingOccupancyFraction 1444
-XX:CMSInitiatingOccupancyFraction=85
```

## jmap

jmap:Memory Map for Java 用于生成堆转储快照,一般称为heapdump或dump文件;生成dump文件也可以使用:

- 启动参数-XX:+HeapDumpOnOutOfMemoryError,可以让虚拟机在内存溢出异常出现之后自动生成堆转储快照
- 通过-XX:+HeapDumpOnCtrlBreak参数可以使用Ctrl+Break键让虚拟机生成dump文件
- 在Linux系统下通过Kill -3命令发送进程退出信号,"恐吓"一下虚拟机,可能拿到堆转储快照

jmap还可以查询finalize执行队列,Java堆和方法区的详细信息,如空间使用率,当前用的哪种收集器

**jmap命令格式:**

jmap [option] vmid

| 选项           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| -dump          | 生成堆转储快照,格式为-dump:[live,]format=b,file=\<filename>,live子参数说明是否只dump存活的对象 |
| -finalizerinfo | 显示在F-Queue中等待Finalizer线程执行finalize方法的对象,仅在Linux/Solaris平台有效 |
| -heap          | 显示Java堆信息,使用哪种垃圾收集器,参数配置,分代状况等        |
| -histo         | 显示堆中对象统计信息,包括类,实例数量,合计容量                |
| -permstat      | 以classloader为统计口径显示永久代内存状态                    |
| -F             | 当虚拟机进程对-dump选项没有响应时,可使用这个选项强制生成dump快照 |

## jhat

jhat JVM Heap Analysis Tool命令与jmap搭配使用,来分析jmap生成的堆转储快照(内置了Web服务器,生成分析结果后可在浏览器中查看),别用这个了,用VisualVM吧或者其他专业工具吧

## jstack

jstack Stack Trace for Java用于生成虚拟机当前时刻的线程快照,也就是当前虚拟机内每一条线程正在执行的方法堆栈集合,生成线程快照目的通常是定位线程出现长时间停顿的原因,如线程间死锁,死循环,请求外部资源导致长时间挂起等

**jstack命令格式:**

jstack [option] vmid

| 选项 | 作用                                        |
| ---- | ------------------------------------------- |
| -F   | 当正常输出的请求不被响应时,强制输出线程堆栈 |
| -l   | 除堆栈外,显示关于锁的附加信息               |
| -m   | 如果调用本地方法的话可以显示C/C++的堆栈     |

JDK5起Java.lang.Thread类中新增了getAllStackTraces()方法用于获取虚拟机中所有线程的StackTraceElement对象,使用这个方法可以完成jstack的大部分功能

## jdk附带的全部工具及简要用途

### 基础工具

用于支持基本的程序创建和运行

| 名称         | 主要作用                                               |
| ------------ | ------------------------------------------------------ |
| appletviewer | 在不使用web浏览器的情况下运行和调试applet,jdk11中移除  |
| extcheck     | 检查jar冲突的工具,从jdk9中被移除                       |
| jar          | 创建和管理jar文件                                      |
| java         | Java运行工具,用于运行class或jar文件                    |
| javac        | 用于Java编程语言的编译器                               |
| javadoc      | Java的API文档生成器                                    |
| javah        | C语言头文件和Stub函数生成器,用于编写JNI方法            |
| javap        | Java字节码分析工具                                     |
| jlink        | 将module和它的依赖打包成一个运行时镜像文件             |
| jdb          | 基于JPDA协议的调试器,以类似于GDB的方式进行调试Java代码 |
| jdeps        | Java类依赖性分析器                                     |
| jdeprscan    | 用于搜索jar包中使用了deprecated的类,从jdk9开始提供     |

### 安全

用于程序签名,设置安全测试等

| 名称       | 主要作用                                                     |
| ---------- | ------------------------------------------------------------ |
| keytool    | 管理密钥库和证书,主要用户获取或缓存kerberos协议的票据授权票据,允许用户查看本地凭据缓存和密钥表中的条目 |
| jarsigner  | 生成并验证jar签名                                            |
| policytool | 管理策略文件的GUI工具,用于管理用户策略文件(.Java.policy)在jdk10被移除 |

### 国际化	

用于创建本地语言文件

| 名称         | 主要作用                                                     |
| ------------ | ------------------------------------------------------------ |
| native2ascii | 本地编码到ASCII编码的转换器(Native-to-ASCII Converter)用于任意受支持的字符编码和对应的ASCII编码和Unicode转义之间的相互转换 |

### 远程方法调用

用于跨web或网络的服务交互

| 名称        | 主要作用                                                     |
| ----------- | ------------------------------------------------------------ |
| rmic        | JavaRMI编译器,为使用JRMP或IIOP协议的远程对象生成Stub,Skeleton和Tie类,也用于生成OMG IDL |
| rmiregistry | 远程对象注册表服务,用于在当前主机的指定端口上创建并启动一个远程对象注册表 |
| rmid        | 启动激活系统守护进程,允许在虚拟机中注册或激活对象            |
| serialver   | 生成并返回指定的序列化版本id                                 |

### Java IDL和RMI-IIOP

在jdk11中结束了十余年的CORBA支持,不再提供了

| 名称       | 主要作用                                                     |
| ---------- | ------------------------------------------------------------ |
| tnameserv  | 提供对命名服务的访问                                         |
| idlj       | IDL转Java编译器(IDL-to-Java Compiler)生成映射OMG IDL接口的Java源文件,并用以Java编程语言编写的使用CORBA功能的应用程序的Java源文件,IDL意为接口定义语言(Interface Definition Language) |
| orbd       | 对象请求代理守护进程(Object Request Broker Daemon)提供客户端查找和调用CORBA环境服务端上的持久化对象的功能,使用ORBD代替瞬态命名服务tnameserv,ORBD包括瞬态命名服务和持久命名服务,ORBD工具集成了服务器管理器,互操作命名服务和引导名称服务器的功能,当客户端想进行服务器定位,注册和激活功能时,可以于servertool一起使用 |
| servertool | 为应用程序注册,注销,启动和关闭服务器提供易用的接口           |

### 部署工具

用于程序打包,发布和部署

| 名称         | 主要作用                                                     |
| ------------ | ------------------------------------------------------------ |
| javapackager | 打包,签名Java和JavaFX应用程序,在jdk11中被移除                |
| pack200      | 使用Java GZIP压缩器将jar文件转换为压缩的pack200文件,压缩的压缩文件是高度压缩的jar,可以直接部署,省去带宽并减少下载时间 |
| unpack200    | 将pack200生成的打包文件解压提取为jar文件                     |

### Java web start

| 名称   | 主要作用                                           |
| ------ | -------------------------------------------------- |
| javaws | 启动Java web start并设置各种选项的工具,jdk11被移除 |

### 性能监控和故障处理

用于监控分析Java虚拟机运行信息,排查问题

| 名称      | 主要作用                                                     |
| --------- | ------------------------------------------------------------ |
| jps       | 显示系统内所有虚拟机进程                                     |
| jstat     | 收集虚拟机各方面的运行数据                                   |
| jstatd    | jstat的守护程序,启动一个RMI服务器应用程序,用于监视测试的HotSpot虚拟机的创建和终止,并提供一个界面允许远程监控工具附加到在本地系统上运行的虚拟机,jdk9中集成到了JHSDB中 |
| jinfo     | 显示虚拟机的配置信息,jdk9中集成到了JHSDB中                   |
| jmap      | 生成堆转储快照heapdump文件,jdk9中集成到了JHSDB中             |
| jhat      | 分析堆转储快照,同样jdk9集成到了JHSDB                         |
| jstack    | 显示虚拟机的线程快照,jdk9集成到了JHSDB中                     |
| jhsdb     | Java HotSpot Debugger,一个基于Serviceability Agent的HotSpot进程调试器,jdk9开始提供 |
| jsadebugd | Java serviceability agent debug daemon适用于Java的可维护性代理调试守护程序,主要用于附加到指定的Java进程,核心文件或充当一个调试服务器 |
| jcmd      | JVM Command虚拟机诊断命令工具,将诊断命令发送到虚拟机,jdk7开始提供 |
| jconsole  | Java console,用于监控Java虚拟机的使用JMX规范的图形工具,可以监控本地和远程虚拟机,还可以监控和管理应用程序 |
| jmc       | Java Mission Control 包含用于监控和管理Java应用程序的工具,而不会引入与这些工具相关联的性能开销,开发者可以使用jmc命令创建JMC工具,从jdk7的update40开始集成进来 |
| jvisualvm | Java VisualVM一种图形化工具,在Java虚拟机中运行时提供有关的Java技术的应用程序的详细信息,其提供内存和CPU分析,堆转储分析,内存泄漏检测,MBean访问和垃圾收集,jdk6开始提供,jdk9不再打入jdk中, 仍在保存更新发展,可独立下载 |

### WebService工具

与CORBA一起在JDK11中被移除

| 名称      | 主要作用                                                     |
| --------- | ------------------------------------------------------------ |
| schemagen | 用于XML绑定的Schema生成器,用于生成XML Schema文件             |
| wsgen     | XML Web Service 2.0的JavaAPI,生成用于JAX-WS Web Service的JAX-WS便携式产物 |
| wsimport  | XML Web Service 2.0的JavaAPI,主要用于根据服务端发布的WSDL文件生成客户端 |
| xjc       | 主要用于根据XML Schema文件生成对应的Java类                   |

### REPL和脚本工具

| 名称       | 主要作用                                                     |
| ---------- | ------------------------------------------------------------ |
| jshell     | 基于Java的Shell REPL(Read-Eval-Print Loop)交互工具           |
| jjs        | 对Nashorn引擎的调用入口,是基于Java实现的一个轻量级高性能JavaScript运行环境 |
| jrunscript | Java命令行脚本外壳工具(Command Line Script Shell)主要用于解释执行JavaScript,Groovy,Ruby等脚本语言 |