Servlet基础知识和原理
============================

1.什么是Servlet？
----------------------------
在JavaWeb项目中，用来接收请求和响应请求的处理对象就是Servlet，另外Servlet还用来渲染动态的页面。

Servlet就是遵循Servlet开发的一个Java类，它运行在服务端，由服务器调用。

![Servlet请求图示](https://note.youdao.com/yws/api/personal/file/WEB7c4ac52c46ca27f4847c43aa13bb2d54?method=download&shareKey=35ff81b00bdb9e9b54a26863faa500af)
图 1 . Servlet请求图示

2.Servlet运行容器
----------------------------
要理解 Servlet 必须要先把 Servlet 容器说清楚，Servlet 与 Servlet 容器的关系有点像枪和子弹的关系，枪是为子弹而生，而子弹又让枪有了杀伤力。虽然它们是彼此依存的，但是又相互独立发展，这一切都是为了适应工业化生产的结果。从技术角度来说是为了解耦，通过标准化接口来相互协作。

#### Servlet有哪些容器
#### Tomcat、Jetty、Jboss、WebLogic

#### 下面主要以Tomcat来说明：

![Tomcat容器模型](https://note.youdao.com/yws/api/personal/file/WEB36485f8b01428aa7298b3428af60e6c9?method=download&shareKey=bee41c4bbe6c2a6d62423254d461f4df "Tomcat容器模型")
图 2 . Tomcat 容器模型

从上图可以看出 Tomcat 的容器分为四个等级，真正管理 Servlet 的容器是 Context 容器，一个 Context 对应一个 Web 工程，在 Tomcat 的配置文件中可以很容易发现这一点，如下：

#### service.xml的 Context 配置参数

```
  <Context path="/projectOne " docBase="D:\projects\projectOne"
reloadable="true" />
```

3.Servlet 容器的启动过程
----------------------------
Tomcat7 也开始支持嵌入式功能，增加了一个启动类 org.apache.catalina.startup.Tomcat。创建一个实例对象并调用 start 方法就可以很容易启动 Tomcat，我们还可以通过这个对象来增加和修改 Tomcat 的配置参数，如可以动态增加 Context、Servlet 等。下面我们就利用这个 Tomcat 类来管理新增的一个 Context 容器，我们就选择 Tomcat7 自带的 examples Web 工程，并看看它是如何加到这个 Context 容器中的。

清单 2 . 给 Tomcat 增加一个 Web 工程
```
Tomcat tomcat = getTomcatInstance(); //Tomcat tomcat = new Tomcat();
File appDir = new File(getBuildDirectory(), "webapps/examples"); 
tomcat.addWebapp(null, "/examples", appDir.getAbsolutePath()); 
tomcat.start(); 
ByteChunk res = getUrl("http://localhost:" + getPort() + 
              "/examples/servlets/servlet/HelloWorldExample"); 
assertTrue(res.toString().indexOf("<h1>Hello World!</h1>") > 0);
```

以上代码是创建一个 Tomcat 实例并新增一个 Web 应用，然后启动 Tomcat 并调用其中的一个 HelloWorldExample Servlet，看有没有正确返回预期的数据。

#### Tomcat 的 addWebapp 方法的代码如下：
```java
public Context addWebapp(Host host, String url, String path) { 
       silence(url); 
       Context ctx = new StandardContext(); 
       ctx.setPath( url ); 
       ctx.setDocBase(path); 
       if (defaultRealm == null) { 
           initSimpleAuth(); 
       } 
       ctx.setRealm(defaultRealm); 
       ctx.addLifecycleListener(new DefaultWebXmlListener()); 
       ContextConfig ctxCfg = new ContextConfig(); 
       ctx.addLifecycleListener(ctxCfg); 
       ctxCfg.setDefaultWebXml("org/apache/catalin/startup/NO_DEFAULT_XML"); 
       if (host == null) { 
           getHost().addChild(ctx); 
       } else { 
           host.addChild(ctx); 
       } 
       return ctx; 
}
```

