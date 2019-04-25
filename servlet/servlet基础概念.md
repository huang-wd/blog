# Servlet
## Servlet Api
### Servlet Api 4个java包
 * javax.servlet
    包含定义Servlet和Servlet容器之 间契约的类和接口。
    
 * javax.servlet.http
   包含定义HTTP Servlet和 Servlet容器之间契约的类和接口。
   
 * javax.servlet.annotation
   包含标注Servlet、 Filter、Listener的标注。它还为被标注元件定义元数据。
   
 * javax.servlet.description
   包含提供程序化登录 web应用程序的配置信息的类型。
   
-------

Servlet技术的核心是Servlet，它是所有Servlet类必 须直接或间接实现的一个接口。在编写实现Servlet的 Servlet类时，直接实现它。在扩展实现这个接口的类 时，间接实现它。
    
servlet接口定义额servlet与servlet容器之间的契约。归结起来就是，servlet容器将servlet类载入内存，并在servlet实例上调用具体的方法。在一个应用程序中，每种servlet类型只能有一个实例。
    
用户请求致使servlet容器调用servlet的service方法，并传入一个ServletRequest实例和一个ServletResponse实例。ServletRequest中封装了当前的HTTP请求，因此，开发人员不需要解析和操作原始的HTTP数据。ServletResponse表示当前用户的HTTP相应，使得将响应发回给用户变得十分容易。
    
对于每个应用程序，Servlet容器还会创建一个ServletContext实例。这个对象中封装了上下文（应用程序）的环境详情。每个上下文只有一个ServletContext。每个Servlet实例也都有一个封装Servlet配置的ServletConfig。
    
### Servlet接口定义了五个方法

```
public void init(ServletConfig config) throws ServletException;
public ServletConfig getServletConfig();
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
public String getServletInfo();
public void destroy();
```
init、service、destroy是生命周期方法。Servlet容器根据一下规则调用这3个方法：
* init，当该Servlet第一次被请求时，Servlet容器会调用这个方法。这个方法在后续的请求中不再会被调用。我们可以利用这个方法执行相应的初始化工作。这个方法传入ServletConfig,
    
    
    
