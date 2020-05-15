# 一只神奇的猫Tomcat  

## **1.Tomcat基础**  

### 1.1 Web基础  

网络三要素：协议、主机名(IP)、端口号

	比如：http://www.baidu.com
	http为协议；  www.baidu.com为域名(主机名)；  默认端口号为80 
![image-20200512195856136](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512195856136.png)

### 1.2 常见的Web服务器  

WebLogic、WebSphere、JBoss、Tomcat等。前三款均为大型JavaEE项目的服务器使用，并且收费；而Tomcat多用于中小型JavaEE项目中，并且开源免费。

### 1.3 Tomcat目录结构

	bin: 存放Tomcat命令的目录。主要用于启动和关闭Tomcat服务器。
		 .sh结尾的用在Linux系统
		 .bat结尾(批处理文件)的运行在Windows系统
	
	conf: 存放Tomcat配置文件。
		 server.xml文件设置端口号等
		 web.xml设置Tomcat支持的文件类型，包括启动页的信息等
	
	lib: 存放Tomcat运行需要的jar包。
		
	logs:存放日志文件。
	
	temp:存放临时文件。
	
	webapps:存放应用程序。当Tomcat启动后会加载webapps目录下的应用程序。可以以文件夹、war包、jar包
	的形式发布应用。
	
	work:存放Tomcat运行后编译的文件，例如JSP编译后的文件。
		 清空work目录，然后重启Tomcat,可以达到清除缓存的作用。





## 2.Tomcat架构  

### 2.1 HTTP的工作原理  

HTTP协议作为应用层协议，基于TCP/IP协议传递数据的，但是其本身并不涉及数据包的传输。HTTP协议规定了客户端和服务器之间的通信格式。客户端发起HTTP请求，服务器做出响应后返回HTTP格式的数据给客户端。

### 2.2 Tomcat的整体架构  

#### 2.1 Http服务器请求处理

![image-20200512201032235](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512201032235.png)



#### 2.2 Servlet容器工作流程

![image-20200512201742747](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512201742747.png)

#### 2.3 整体架构

Tomcat要实现的两大核心功能：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512202007220.png" alt="image-20200512202007220" style="zoom:150%;" />

Tomcat由两大核心组件组成：连接器（Connector）和容器（Container）。连接器负责对外交流，容器负责内部逻辑处理。

![image-20200512202109239](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512202109239.png)

### 2.3 连接器 — Coyote  

#### 2.3.1  IO模型与协议

![image-20200512203538873](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512203538873.png)

**注意**：<u>Tomcat8.0之前的版本使用的是BIO，之后的使用的是NIO</u>。以上三种IO模型在性能方面均优于以往的BIO。

![image-20200512203658334](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512203658334.png)

**注意**：<u>多采用HTTP/1.1协议。</u>  

	注意：1.一个容器(Catalina)对应着多个连接器(Coyote)。他们单独不能对外提供服务。
		 2.一个service至少包括一个连接器和一个容器。
#### 2.3.2  连接器Coyote组件

![image-20200512204521623](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512204521623.png)

	EndPoint: Coyote通信端点。即通信监听的接口，是具体Socket接收和发送处理器，是对传输层的抽象，因此EndPoint用来实现TCP/IP协议的。  
	Processor: Coyote协议处理接口。接收来自EndPoint的Socket，读取字节流解析成Request和Response对象，并通过Adapter将其提交到容器处理，是对应用层协议的抽象，用来实现HTTP协议的。   
	ProtocalHandler: 封装EndPoint和Processor。  
	Adapter: 作为适配器，将Request对象转化成ServletRequest对象。
### 2.4 容器 — Catalina

Catalina是Tomcat的Servlet容器。

#### 2.4.1  Catalina的地位

Tomcat的模块分层结构图如下：

![image-20200512211819068](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512211819068.png)

Tomcat本质上就是Servlet容器，因此Catalina是Tomcat的核心，其他模块都是为Catalina提供支撑的，比如：Coyote模块提供链接通信，Jasper模块提供JSP引擎，Naming提供JNDI服务，Juli提供日志服务。

#### 2.4.2  Catalina结构

Catalina的主要组件结构如下：

![image-20200512212529924](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512212529924.png)

Catalina负责管理Server,而server表示整个服务器。server下面又有多个服务service，每个服务service对应着多个连接器组件Connector和一个容器组件Container。Tomcat启动的时候就会初始化一个Catalina的实例。  

![image-20200512212950892](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512212950892.png)

#### 2.4.3  Container结构

![image-20200512213503912](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512213503912.png)

![image-20200512213726145](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512213726145.png)

Tomcat使用**<u>组合模式</u>**管理这些容器：

![image-20200512214450886](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512214450886.png)

**<u>连接器Coyote和容器Catalina</u>**之间的关系：

![image-20200512203025014](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512203025014.png)

### 2.5 Tomcat启动流程

![image-20200512214755580](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512214755580.png)

主要通过apache-tomcat-7.0.100-src \ java \ org \ apache \ catalina \ startup \ Bootstrap.java的main()方法启动整个过程。通过父组件调用本身及其子组件从而启动Tomcat。

![image-20200512215549069](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512215549069.png)

**注意**：![image-20200512215858912](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512215858912.png)

### 2.6 Tomcat请求处理流程

Tomcat通过Mapper组件来完成该任务的。Mapper组件将用户请求的URL定位到一个Servlet。（也即键为URL，值为Servlet）当请求到来时，Mapper组件通过解析请求RUL里的域名和路径，再到自己保存的Map里去查找，就能定位到一个Servlet。一个请求URL最后只会定位到一个Wrapper容器，也就是一个Servlet。 

根据请求的URL如何查找到需要执行的Servlet：

![image-20200512222141148](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512222141148.png)     

从Tomcat的设计架构方面来分析Tomcat的请求处理：

![image-20200512222736368](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200512222736368.png)





## 3.Jasper  

### 3.1  Jasper简介

![image-20200513132911363](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513132911363.png)

![image-20200513132958772](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513132958772.png)

上图所示服务器中的红框即为Jasper做的事情。

### 3.2  JSP编译方式

#### 3.2.1  运行时编译

![image-20200513133225628](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513133225628.png)

![image-20200513134211237](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513134211237.png)

#### 3.2.2  预编译

![image-20200513134850700](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513134850700.png)

### 3.3  JSP编译原理

![image-20200513135332978](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513135332978.png)





## 4.Tomcat服务器配置  

### 4.1  server.xml

#### 4.1.1  Server

![image-20200513140049965](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513140049965.png)

#### 4.1.2  Service

![image-20200513140327397](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513140327397.png)

#### 4.1.3  Executor

![image-20200513142253727](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513142253727.png)

#### 4.1.4  Connector

![image-20200513142333650](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513142333650.png)

#### 4.1.5  Engine

![image-20200513142205314](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513142205314.png)

#### 4.1.6  Host

![image-20200513142646220](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513142646220.png)

#### 4.1.7  Context

![image-20200513143828281](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513143828281.png)

### 4.2  tomcat-users.xml

![image-20200513150212094](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513150212094.png)





## 5.Web应用配置  

web.xml是web应用的描述文件。web应用的描述信息主要包括两部分：tomcat / conf / web.xml中默认配置以及web应用WEB-INF / web.xml下的定制配置。

### 5.1  ServletContext初始化参数

在web.xml中配置如下：

```xml
<context-param>
    <param-name>project_param_01</param-name>
    <param-value>itcast</param-value>
</context-param>
```

在servlet文件中获取其输出：

![image-20200513221513473](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513221513473.png)

### 5.2  会话配置

Cookie和Session的关联：

![image-20200513221902789](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513221902789.png)	

会话的配置案例：

![image-20200513222312136](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513222312136.png)

![image-20200513222545344](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513222545344.png)

### 5.3  Servlet配置

Servlet的配置主要是两部分：servlet 和 servlet-mapping。

示例如下：

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513223113159.png)

### 5.4  Listener配置

![image-20200513223443237](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513223443237.png)

### 5.5  Filter配置

![image-20200513223538931](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513223538931.png)

![image-20200513223634577](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513223634577.png)

### 5.6  欢迎页面配置

![image-20200513223707527](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513223707527.png)

### 5.7  错误页面配置

![image-20200513224038281](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200513224038281.png)





## 6.Tomcat管理配置  

Tomcat提供两个管理应用（位于apache-tomcat-7.0.100 \ webapps \ 下）：用于管理Host的host-manager文件夹；用于管理web应用的manager文件夹。

### 6.1  host-manager

![image-20200514104105558](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514104105558.png)

### 6.2  manager

![image-20200514104141046](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514104141046.png)





## 7.JVM配置  

JVM的配置主要是内存的分配。

### 7.1  JVM内存模型图

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514105815449.png" alt="image-20200514105815449" style="zoom:150%;" />

### 7.2  配置选项

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514105944089.png" alt="image-20200514105944089" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514110233896.png" alt="image-20200514110233896" style="zoom:150%;" />





## 8.Tomcat集群  

### 8.1  简介

![image-20200514135004786](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514135004786.png)

### 8.2  环境准备

#### 8.2.1  准备Tomcat

![image-20200514135402791](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514135402791.png)

#### 8.2.2  安装配置Nginx

![image-20200514135620471](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514135620471.png)



### 8.3  负载均衡策略

#### 8.3.1  轮询

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514140622848.png" alt="image-20200514140622848" style="zoom:200%;" />

#### 8.3.2  weight权重

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514141229376.png" alt="image-20200514141229376" style="zoom:200%;" />

#### 8.3.3  ip_hash

![image-20200514142142163](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514142142163.png)

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514142253095.png" alt="image-20200514142253095" style="zoom:200%;" />

### 8.4  Session共享方案

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514213331139.png" alt="image-20200514213331139" style="zoom:200%;" />

#### 8.4.1  ip_hash策略

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514213600313.png" alt="image-20200514213600313" style="zoom:200%;" />

![image-20200514213652529](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514213652529.png)

#### 8.4.2  Session复制

![image-20200514213906682](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514213906682.png)

<u>**注意**</u>：主要用于小型项目，即集群的Tomcat数量较少时。

#### 8.4.3  SSO-单点登录

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514215501348.png" alt="image-20200514215501348" style="zoom:200%;" />

​	



## 9.Tomcat安全  

### 9.1  配置安全

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514220623684.png" alt="image-20200514220623684" style="zoom:200%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514220822920.png" alt="image-20200514220822920" style="zoom:100%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514220851811.png" alt="image-20200514220851811" style="zoom:200%;" />

### 9.2  应用安全

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514220942638.png" alt="image-20200514220942638" style="zoom:150%;" />

### 9.3  传输安全

#### 9.3.1  HTTPS介绍

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514222529721.png" alt="image-20200514222529721" style="zoom:200%;" />

#### 9.3.2  Tomcat支持HTTPS

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514222723873.png" alt="image-20200514222723873" style="zoom:100%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200514223233809.png" alt="image-20200514223233809" style="zoom:150%;" />





## 10.Tomcat性能调优  

#### 10.1  Tomcat性能测试

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515211710482.png" alt="image-20200515211710482" style="zoom:150%;" />

##### 10.1.1  ApacheBench

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515212021842.png" alt="image-20200515212021842" style="zoom:150%;" />

#### 10.2  Tomcat性能优化

主要对JVM的内存和GC垃圾回收两方面进行优化。

##### 10.2.1  JVM参数调优

1）	JVM内存参数

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515212949079.png" alt="image-20200515212949079" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515213512080.png" alt="image-20200515213512080" style="zoom:200%;" />

2） GC策略

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515213623596.png" alt="image-20200515213623596" style="zoom:200%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515213721763.png" alt="image-20200515213721763" style="zoom:200%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515213816435.png" alt="image-20200515213816435" style="zoom:200%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515214139960.png" alt="image-20200515214139960" style="zoom:200%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515214244236.png" alt="image-20200515214244236" style="zoom:200%;" />

##### 10.2.2  Tomcat配置调优

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515214559817.png" alt="image-20200515214559817" style="zoom:200%;" />





## 11.Tomcat附件功能

### 11.1  WebSocket

#### 11.1.1  WebSocket介绍

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515215116334.png" alt="image-20200515215116334" style="zoom:200%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515215323532.png" alt="image-20200515215323532" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515215408347.png" alt="image-20200515215408347" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515215616262.png" alt="image-20200515215616262" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515215741861.png" alt="image-20200515215741861" style="zoom:150%;" />

#### 11.1.2  Tomcat的WebSocket

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515215951227.png" alt="image-20200515215951227" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515220118516.png" alt="image-20200515220118516" style="zoom:150%;" />

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200515220235453.png" alt="image-20200515220235453" style="zoom:150%;" />

