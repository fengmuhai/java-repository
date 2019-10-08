Servlet基础知识和原理
============================

### 1.什么是Servlet？
在JavaWeb项目中，用来接收请求和响应请求的处理对象就是Servlet，另外Servlet还用来渲染动态的页面。

Servlet就是遵循Servlet开发的一个Java类，它运行在服务端，由服务器调用。

![Servlet请求图示]("https://note.youdao.com/ynoteshare1/index.html?id=bee41c4bbe6c2a6d62423254d461f4df&type=note")

### 2.Servlet运行容器
要理解 Servlet 必须要先把 Servlet 容器说清楚，Servlet 与 Servlet 容器的关系有点像枪和子弹的关系，枪是为子弹而生，而子弹又让枪有了杀伤力。虽然它们是彼此依存的，但是又相互独立发展，这一切都是为了适应工业化生产的结果。从技术角度来说是为了解耦，通过标准化接口来相互协作。

#### Servlet有哪些容器
#### Tomcat、Jetty、Jboss、WebLogic

#### 下面主要以Tomcat来说明：
