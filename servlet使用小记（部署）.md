写好了[Anti-Spam](https://github.com/LangleyChang/Anti-Spam)的核心代码，今天打算放在tomcat上运行起来。没有想到学习servlet的进度这么慢，又是折腾了一天tomcat，现在把折腾的结果记录下来。

以最简单的Hello,world为例，webapps中目录结构

	Helloworld
		|- index.html
		|_ WEB_INF
				|- class - me - zchang - Helloworld.class
				|- lib
				|_ web.xml
这个目录结构在配置路径的时候显得非常重要，现在我要实现的功能是在点击主页的按钮后显示"Hello, world.".

具体的代码不多说了，直接贴（直接手写的，没调试）

> index.html

	<html>
	<head>
	<title>Hello, world<title>
	</head>
		<form method="POST" action="hello">注意这个路径
		<input type="submit" value="click here">
		</form>
	</html>

> web.xml

	<?xml version="1.0" encoding="ISO-8859-1">
	<web-app xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee                       http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" version="3.1" metadata-complete="true">
	<servlet>
		<servlet-name>helloworld<servlet-name>
		<servlet-class>me.zchang.Helloworld<servlet-class>注意这个路径
	</servlet>

	<servlet-mapping>
		<servlet-name>helloworld</servlet-name>和上面的name匹配
		<url-pattern>/hello</url-pattern>重要
	</servlet-mapping>
	</web-app>
> Helloworld.java

	package me.zchang
	
	import javax.servlet.ServletException;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;

	public class HelloWorld extends HttpServlet{
		@Override
		protected void doPost(HttpServletRequest req,HttpServletResponse resp)
		throws ServletException,IOException{
			resp.setContentType("text/html");
			PrintWriter out = resp.getWriter();
			out.println("<html>");
			out.println("<body>");
			out.println("<p>Hello, world.</p>")
			out.println("</body>")
			out.println("</html>")
		}
	}

这样，打开http://localhost:8080/Helloworld大概就能看到效果了。

现在我说明一下这几个路径的含义。


首先，在`web.xml`注册一个servlet，需要用`<servlet>`标签，里面描述了servlet的名字（随便取）和对应“主函数”（就是程序入口）所在的类，接着，在下面的`<servlet-mapping>`中定义html/jsp调用servlet时需要用到的路径，我这里将齐定义为`/hello`，`/`表示这个webapp的根目录。这样，相当于在`/hello`处创建了一个虚拟的目录，html/jsp在调用时，可以认为这个路径下是存在文件的。
所以，配置好的目录相当与如下

	Helloworld
		|- index.html
		|- WEB_INF
		:		|- class - me - zchang - Helloworld.class
		:		|- lib
		:		|_ web.xml
		:_ hello
	
所以，在`index.html`中，提交的`action`是`hello`，它可以认为调用的是一个与自己同目录下的叫做'hello'的servlet程序。

别看这个问题这么简单，可是纠结了我大半天的…… 以防忘记。
	

> Written with [StackEdit](https://stackedit.io/).