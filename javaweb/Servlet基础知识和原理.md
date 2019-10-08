Servlet基础知识和原理
============================

### 1.什么是Servlet？
在JavaWeb项目中，用来接收请求和响应请求的处理对象就是Servlet，另外Servlet还用来渲染动态的页面。

Servlet就是遵循Servlet开发的一个Java类，它运行在服务端，由服务器调用。

![Servlet请求图示](https://note.youdao.com/yws/api/personal/file/WEB7c4ac52c46ca27f4847c43aa13bb2d54?method=getImage&version=2872&cstk=YFzyk_GM)

### 2.Servlet运行容器
要理解 Servlet 必须要先把 Servlet 容器说清楚，Servlet 与 Servlet 容器的关系有点像枪和子弹的关系，枪是为子弹而生，而子弹又让枪有了杀伤力。虽然它们是彼此依存的，但是又相互独立发展，这一切都是为了适应工业化生产的结果。从技术角度来说是为了解耦，通过标准化接口来相互协作。

#### Servlet有哪些容器
#### Tomcat、Jetty、Jboss、WebLogic

#### 下面主要以Tomcat来说明：

![Tomcat容器模型](https://note.youdao.com/yws/api/personal/file/WEB36485f8b01428aa7298b3428af60e6c9?method=download&shareKey=bee41c4bbe6c2a6d62423254d461f4df "Tomcat容器模型")
