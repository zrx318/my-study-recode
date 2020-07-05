### SpringMVC 初始化及请求转发实现原理

　　简而言之，在 DispatcherServlet 的 init 方法中完成请求转发的准备工作，在 DispatcherServlet的 service 方法中完成真正的请求转发过程。

1. Spring MVC 的初始化过程是从 DispatcherServlet 的 init 方法执行开 始的，首先读取配置文件(或者按照注解的配置)进行初始化，以通过 ContextLoaderListener 建立起来的 WebApplicatioContext 作为父上 下文，这个过程中比较重要的一步就是初始化我们所有的 Controller 层的 Bean放在 IOC 容器中以备使用。

2. DispatcherServlet 的 init 方法第二个重要的步骤就是初始化 HandlerMapping 类型的 Bean，以 RequestMappingHandler 为例，在 通过 getBean 方法实例化一个 RequestMappingHandler 类型的对象 的时候，先扫描我们所有自定义的 Intercepetor 类型的对象，注册到 RequestMappingHandler 中以备使用。之后会扫描所有使用 @Controller 注解标注的 Controller 类，获取类中所有使用 RequestMapping注解标注的方法，之后在RequestMappingHandler 中建立起一个 Map，Map 的 key 值为 RequestMapping 注解中具体 的 Url，Map 的 value 值为 Controller 所在的具体路径(包名+类名)+ 对应的方法名。

3. 请求分发到具体的 Controller 是在 DispatcherServlet 的 service 方法 中实现的，简要来说就是通过用户请求的 url 在 RequestMappingHandler 中进行匹配，之后反射调用对应 Controller 层中的相应方法来实现的。在调用具体业务方法之前，还会检测是否 有 interceptor 需要调用。


   