

## SpringMVC

### SpringMVC请求处理流程

在容器初始化时会建立所有 url 和 controller 的对应关系，保存到Map<url,controller>中，这样就可以根据 request 快速定位到 Controller，然后找到 对应的 method，最后把参数传进方法，执行请求并返回处理结果。

**流程说明（简要版）**：

①：DispatcherServlet 负责接收 request 并将 request 转发给对应的处理组件。

②：DispatcherServlet 接收 request，然后从HandlerMapping查找处理request的controller.

③：Cntroller 处理 request，并返回 ModelAndView 对象

④ ：视图解析器解析ModelAndView对象并返回对应的视图给客户端。

**详细版**：

1、DispatcherServlet 接收到客户端发送的请求。

2、DispatcherServlet 调用HandlerMapping 处理器映射器。

3、HandleMapping 根据请求URL 找到对应的 handler 以及 Interceptor，返回给DispatcherServlet

4、DispatcherServlet 根据handler 调用HanderAdapter 处理器适配器。

5、HandlerAdapter 根据handler 执行处理器，也就是我们controller层写的业务逻辑，并返回一个ModeAndView

6、HandlerAdapter 返回ModeAndView 给DispatcherServlet

7、DispatcherServlet 调用 ViewResolver 视图解析器来 来解析ModeAndView

8、ViewResolve  解析ModeAndView 并返回真正的view 给DispatcherServlet

9、DispatcherServlet 将得到的视图进行渲染，填充到request域中

10、返回给客户端响应结果。

