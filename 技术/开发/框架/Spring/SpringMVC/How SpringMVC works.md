![Image title](How%20SpringMVC%20works/How_MVC_Works-793x397.png)

This is an in-depth look at the powerful features and internal workings of Spring Web MVC, which is a part of the [Spring Framework](https://projects.spring.io/spring-framework/).

The source code for this article is available [over on GitHub](https://github.com/Baeldung/stackify/tree/master/spring-mvc).

## Project Setup

Throughout this article, we’ll use the latest and greatest Spring Framework 5. We’re focusing here on the Spring’s classic web stack, which has been available from the very first versions of the framework and is still the primary way of building web applications with Spring.

For starters, to set up your test project, you’ll use Spring Boot and some of its starter dependencies; you’ll also need to define the parent:



```xml
&lt;parent&gt;

​    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;

​    &lt;artifactId&gt;spring-boot-starter-parent&lt;/artifactId&gt;

​    &lt;version&gt;2.0.0.M5&lt;/version&gt;

​    &lt;relativePath/&gt;

&lt;/parent&gt;

&lt;dependencies&gt;

​    &lt;dependency&gt;

​        &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;

​        &lt;artifactId&gt;spring-boot-starter-web&lt;/artifactId&gt;

​    &lt;/dependency&gt;

​    &lt;dependency&gt;

​        &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;

​        &lt;artifactId&gt;spring-boot-starter-thymeleaf&lt;/artifactId&gt;

   &lt;/dependency&gt;

&lt;/dependencies&gt;
```



Note that, in order to use Spring 5, you need to also use Spring Boot 2.x. At the time of writing, this is a milestone release, available in the [Spring Milestone Repository](https://repo.spring.io/libs-milestone/org/springframework/boot/spring-boot-starter-web/). Let’s add this repository to your Maven project:

```
&lt;repositories&gt;
    &lt;repository&gt;
        &lt;id&gt;spring-milestones&lt;/id&gt;
        &lt;name&gt;Spring Milestones&lt;/name&gt;
        &lt;url&gt;https://repo.spring.io/milestone&lt;/url&gt;
        &lt;snapshots&gt;
            &lt;enabled&gt;false&lt;/enabled&gt;
        &lt;/snapshots&gt;
    &lt;/repository&gt;
&lt;/repositories&gt;
```



You can check out the current version of Spring Boot on [Maven Central](https://search.maven.org/#search|gav|1|g%3A"org.springframework.boot" AND a%3A"spring-boot-starter-web").

## Sample Project

To understand how Spring Web MVC works, you’ll implement a simple application with a login page. To show the login page, create a *@Controller*-annotated class *InternalController* with a GET mapping for the context root.

The *hello()* method is parameterless. It returns a *String* which is interpreted by Spring MVC as a view name (in our case, the *login.html* template):

1

```
import org.springframework.web.bind.annotation.GetMapping;
```

2

```

```

3

```
@GetMapping("/")
```

4

```
public String hello() {
```

5

```
    return "login";
```

6

```
}
```



To process a user login, create another method that handles POST requests with login data. It then redirects the user either to the success or failure page, depending on the result.

Note that the *login()* method receives a domain object as an argument and returns a *ModelAndView* object:

1

```
import org.springframework.web.bind.annotation.PostMapping;
```

2

```
import org.springframework.web.servlet.ModelAndView;
```

3

```

```

4

```
@PostMapping("/login")
```

5

```
public ModelAndView login(LoginData loginData) {
```

6

```
    if (LOGIN.equals(loginData.getLogin()) 
```

7

```
      &amp;&amp; PASSWORD.equals(loginData.getPassword())) {
```

8

```
        return new ModelAndView("success", 
```

9

```
          Collections.singletonMap("login", loginData.getLogin()));
```

10

```
    } else {
```

11

```
        return new ModelAndView("failure", 
```

12

```
          Collections.singletonMap("login", loginData.getLogin()));
```

13

```
    }
```

14

```
}
```



*ModelAndView* is a holder of two distinct objects:

- Model – a key-value map of data used to render the page
- View – a template of the page that is filled with data from the model

These are joined for convenience so that the controller method can return them both at once.

To render your HTML page, use [Thymeleaf](http://www.thymeleaf.org/) as a view template engine, which has solid, out-of-the-box integration with Spring.

## Servlets as the Foundation of a Java Web Application

So, what does actually happen when you type *http://localhost:8080/* in the browser, press Enter, and the request hits the web server? How do you get from this request to seeing a web form in the browser?

Given the project is a simple Spring Boot application, you’ll be able to run it via the *Spring5Application*.

Spring Boot uses [Apache Tomcat](https://stackify.com/tomcat-performance-monitoring/) by default. Hence, running the application, you are likely to see the following information in the log:

1

```
2017-10-16 20:36:11.626  INFO 57414 --- [main] 
```

2

```
  o.s.b.w.embedded.tomcat.TomcatWebServer  : 
```

3

```
  Tomcat initialized with port(s): 8080 (http)
```

4

```

```

5

```
2017-10-16 20:36:11.634  INFO 57414 --- [main] 
```

6

```
  o.apache.catalina.core.StandardService   : 
```

7

```
  Starting service [Tomcat]
```

8

```

```

9

```
2017-10-16 20:36:11.635  INFO 57414 --- [main] 
```

10

```
  org.apache.catalina.core.StandardEngine  : 
```

11

```
  Starting Servlet Engine: Apache Tomcat/8.5.23
```



Since Tomcat is a Servlet container, naturally every HTTP request sent to a Tomcat web server is processed by a Java servlet. So the Spring Web application entry point is, not surprisingly, a servlet.

[A servlet](https://en.wikipedia.org/wiki/Java_servlet) is, simply put, a core component of any Java web application; it’s low-level and does not impose too much in the way of specific programming patterns, such as MVC.

An HTTP servlet can only receive an HTTP request, process it in some way, and send a response back.

And, starting with the Servlet 3.0 API, you can now move beyond XML configuration and start leveraging Java configuration (with minor restrictions).

## DispatcherServlet as the Heart of Spring MVC

What we really want to do as developers of a web application is to abstract away the following tedious and boilerplate tasks and focus on useful business logic:

- mapping an HTTP request to a certain processing method
- parsing of HTTP request data and headers into data transfer objects (DTOs) or domain objects
- model-view-controller interaction
- generation of responses from DTOs, domain objects, etc.

The Spring [*DispatcherServlet*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html) provides exactly that. It is the heart of the Spring Web MVC framework; this core component receives all requests to your application.

As you’ll see, *DispatcherServlet* is very extensible. For example, it allows you to plug in different existing or new adapters for a lot of tasks:

- map a request to a class or method that should handle it (implementations of the *HandlerMapping*interface)
- handle a request using a specific pattern, like a regular servlet, a more complex MVC workflow, or just a method in a POJO bean (implementations of the *HandlerAdapter* interface)
- resolve views by name, allowing you to use different templating engines, XML, XSLT or any other view technology (implementations of the *ViewResolver* interface)
- parse multipart requests by using the default Apache Commons file uploading implementation or writing your own *MultipartResolver*
- resolve locale with any *LocaleResolver* implementation, including cookie, session, *Accept* HTTP header, or any other way of determining the locale expected by the user

## Processing of an HTTP Request

First, let’s trace the processing of simple HTTP requests to a method in your controller layer and back to the browser/client.

The *DispatcherServlet* has a long inheritance hierarchy; it’s worth understanding these individual aspects one by one, top-down. The request processing methods will interest us the most.

### ![https://lh3.googleusercontent.com/4ahYReme6gkGU8NIU--JzvxgCckXNYzCBa_Fi7Xk6DwqNctyNj0pOB_UPU4Euboy66vfURfovJK8VUgJMTz0ms2yQtFnqEhw9iJnWd_pCCpWN5tNXIYhUFtkCxakO-GTyuKlHoqs](How%20SpringMVC%20works/https-lh3-googleusercontent-com-4ahyreme6gkgu8ni-118x300.png)

Understanding the HTTP request, both locally during standard development, [as well as remotely](https://stackify.com/prefix-remote-http-calls/), is a critical part of understanding the MVC architecture.

### GenericServlet

*GenericServlet* is a part of the Servlet specification not directly focused on HTTP. It defines the *service()*method that receives incoming requests and produces responses.

Note how *ServletRequest* and *ServletResponse* method arguments are not tied to the HTTP protocol:

1

```
public abstract void service(ServletRequest req, ServletResponse res) 
```

2

```
  throws ServletException, IOException;
```



This is the method that is eventually called on any request to the server, including a simple GET request.

### HttpServlet

*HttpServlet* class is, as the name suggests, the HTTP-focused Servlet implementation, also defined by the specification.

In more practical terms, *HttpServlet* is an abstract class with a *service()* method implementation that splits the requests by the HTTP method type and looks roughly like this:

1

```
protected void service(HttpServletRequest req, HttpServletResponse resp)
```

2

```
    throws ServletException, IOException {
```

3

```

```

4

```
    String method = req.getMethod();
```

5

```
    if (method.equals(METHOD_GET)) {
```

6

```
        // ...
```

7

```
        doGet(req, resp);
```

8

```
    } else if (method.equals(METHOD_HEAD)) {
```

9

```
        // ...
```

10

```
        doHead(req, resp);
```

11

```
    } else if (method.equals(METHOD_POST)) {
```

12

```
        doPost(req, resp);
```

13

```
        // ...
```

14

```
    }
```



### HttpServletBean

Next, *HttpServletBean* is the first Spring-aware class in the hierarchy. It injects the bean’s properties using the servlet *init-param* values received from the *web.xml* or from *WebApplicationInitializer*.

In case of the requests to your application, the *doGet()*, *doPost()*, etc methods are called for those specific HTTP requests.

### FrameworkServlet

*FrameworkServlet* integrates the Servlet functionality with a web application context, implementing the *ApplicationContextAware* interface. But it is also able to create a web application context on its own.

As you already saw, the *HttpServletBean* superclass injects init-params as bean properties. So, if a context class name is provided in the *contextClass* init-param of the servlet, then an instance of this class will be created as an application context. Otherwise, a default *XmlWebApplicationContext* class will be used.

As XML configuration is out of style nowadays, Spring Boot configures *DispatcherServlet* with *AnnotationConfigWebApplicationContext* by default. But you could change that easily.

For example, if you need to configure your Spring Web MVC application with a Groovy-based application context, you could use the following configuration of *DispatcherServlet* in the *web.xml* file:

1

```
    dispatcherServlet
```

2

```
        org.springframework.web.servlet.DispatcherServlet
```

3

```

```

4

```
        contextClass
```

5

```
        org.springframework.web.context.support.GroovyWebApplicationContext
```



The same configuration may be done in a more modern Java-based way using the WebApplicationInitializer class.

### DispatcherServlet: Unifying the Request Processing

The *HttpServlet.service()* implementation, which routes requests by the type of HTTP verb, makes perfect sense in the context of low-level servlets. However, at the Spring MVC level of abstraction, method type is just one of the parameters that can be used to map the request to its handler.

And so, the other main function of the *FrameworkServlet* class is to join the handling logic back into a single *processRequest()* method, which in turn calls the *doService()* method:

1

```
@Override
```

2

```
protected final void doGet(HttpServletRequest request, 
```

3

```
  HttpServletResponse response) throws ServletException, IOException {
```

4

```
    processRequest(request, response);
```

5

```
}
```

6

```

```

7

```
@Override
```

8

```
protected final void doPost(HttpServletRequest request, 
```

9

```
  HttpServletResponse response) throws ServletException, IOException {
```

10

```
    processRequest(request, response);
```

11

```
}
```

12

```

```

13

```
// …
```



### DispatcherServlet: Enriching the Request

Finally, the *DispatcherServlet* implements the *doService()* method. Here, it adds to the request some useful objects that may come in handy down the processing pipeline: web application context, locale resolver, theme resolver, theme source etc.:

1

```
request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, 
```

2

```
  getWebApplicationContext());
```

3

```
request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
```

4

```
request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
```

5

```
request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
```



Also, the *doService()* method prepares input and output flash maps. Flash map is basically a pattern to pass parameters from one request to another request that immediately follows. This may be very useful during redirects (like showing the user a one-shot information message after the redirect):

1

```
FlashMap inputFlashMap = this.flashMapManager
```

2

```
  .retrieveAndUpdate(request, response);
```

3

```
if (inputFlashMap != null) {
```

4

```
    request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, 
```

5

```
      Collections.unmodifiableMap(inputFlashMap));
```

6

```
}
```

7

```
request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
```



Then, the *doService()* method calls the *doDispatch()* method that is responsible for request dispatching.

### DispatcherServlet: Dispatching the Request

The main purpose of the *dispatch()* method is to find an appropriate handler for the request and feed it the request/response parameters. The handler is basically any kind of *Object* and is not limited to a specific interface. This also means that Spring needs to find an adapter for this handler that knows how to “talk” to the handler.

To find the handler that matches the request, Spring goes through the registered implementations of the [*HandlerMapping*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerMapping.html) interface. There are many different implementations that could suit your needs.

*SimpleUrlHandlerMapping* allows mapping a request by its URL to a certain processing bean. For example, it can be configured by injecting its *mappings* property with a *java.util.Properties* instance similar to this:

1

```
/welcome.html=ticketController
```

2

```
/show.html=ticketController
```



Probably the most widely-used class for handler mapping is *RequestMappingHandlerMapping*, which maps a request to a *@RequestMapping*-annotated method of a *@Controller* class. This is exactly the mapping that connects the dispatcher with the *hello()* and *login()* methods of your controller.

Note that your Spring-aware methods are annotated with *@GetMapping* and *@PostMapping*correspondingly. These annotations, in turn, are marked with the *@RequestMapping* meta-annotation.

The *dispatch()* method also takes care of some other HTTP-specific tasks:

- short-circuiting processing of the GET request in case the resource was not modified
- applying the multipart resolver for corresponding requests
- short-circuiting processing of the request if the handler chose to handle it asynchronously

### Handling the Request

Now that Spring determined the handler for the request and the adapter for the handler, it’s time to finally handle the request. Here’s the signature of the *HandlerAdapter.handle()* method. It’s important to note that the handler has a choice in how to handle the request:

- Write the data to the response object by itself and return null
- Return a *ModelAndView* object to be rendered by the *DispatcherServlet*

1

```
@Nullable
```

2

```
ModelAndView handle(HttpServletRequest request, 
```

3

```
                    HttpServletResponse response, 
```

4

```
                    Object handler) throws Exception;
```



There are several provided types of handlers. Here’s how the *SimpleControllerHandlerAdapter* processes a Spring MVC controller instance (do not confuse it with a *@Controller*-annotated POJO).

Notice how the controller handler returns *ModelAndView* object and does not render the view by itself:

1

```
public ModelAndView handle(HttpServletRequest request, 
```

2

```
  HttpServletResponse response, Object handler) throws Exception {
```

3

```
    return ((Controller) handler).handleRequest(request, response);
```

4

```
}
```



The second is *SimpleServletHandlerAdapter,* which adapts a regular *Servlet* as a request handler.

A *Servlet* does not know anything about *ModelAndView* and simply handles the request by itself, rendering the result into the response object. So this adapter simply returns *null* instead of *ModelAndView*:

1

```
public ModelAndView handle(HttpServletRequest request, 
```

2

```
  HttpServletResponse response, Object handler) throws Exception {
```

3

```
    ((Servlet) handler).service(request, response);
```

4

```
    return null;
```

5

```
}
```



In your case, a controller is a POJO with several *@RequestMapping* annotations, so any handler is basically a method of this class wrapped in a *HandlerMethod* instance. To adapt to this handler type, Spring uses the *RequestMappingHandlerAdapter* class.

### Processing Arguments and Return Values of Handler Methods

Note that the controller methods do not usually take *HttpServletRequest* and *HttpServletResponse*arguments, but instead receive and return many different types of data, such as domain objects, path parameters etc.

Also, note that you are not required to return a *ModelAndView* instance from a controller method. You may return a view name, or a *ResponseEntity* or a POJO that will be converted to a JSON response etc.

The *RequestMappingHandlerAdapter* makes sure the arguments of the method are resolved from the *HttpServletRequest.* Also, it creates the *ModelAndView* object from the method’s return value.

There is an important piece of code in the *RequestMappingHandlerAdapter* that makes sure all this conversion magic takes place:

1

```
ServletInvocableHandlerMethod invocableMethod 
```

2

```
  = createInvocableHandlerMethod(handlerMethod);
```

3

```
if (this.argumentResolvers != null) {
```

4

```
    invocableMethod.setHandlerMethodArgumentResolvers(
```

5

```
      this.argumentResolvers);
```

6

```
}
```

7

```
if (this.returnValueHandlers != null) {
```

8

```
    invocableMethod.setHandlerMethodReturnValueHandlers(
```

9

```
      this.returnValueHandlers);
```

10

```
}
```



The *argumentResolvers* object is a composite of different [*HandlerMethodArgumentResolver*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html) instances.

There are over 30 different argument resolver implementations. They allow extracting of any kind of information from the request and providing it as method arguments. This includes URL path variables, request body parameters, request headers, cookies, session data etc.

The *returnValueHandlers* object is a composite of [*HandlerMethodReturnValueHandler*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodReturnValueHandler.html) objects. There are also a lot of different value handlers that can process the result of your method to create *ModelAndView*object expected by the adapter.

For instance, when you return a string from the *hello()* method, the *ViewNameMethodReturnValueHandler*processes the value. But when you return a ready *ModelAndView* from the *login()* method, Spring uses the *ModelAndViewMethodReturnValueHandler*.

### Rendering the View

By now, Spring has processed the HTTP request and received a *ModelAndView* object, so it has to render the HTML page that the user will see in the browser. It does that based on the model and the selected view encapsulated in the *ModelAndView* object.

Also note that you could render a JSON object, or XML, or any other data format that can be transferred via HTTP protocol. We’ll touch more on that in the upcoming REST-focused section here.

Let’s get back to the *DispatcherServlet*. The *render()* method first sets the response locale using the provided [*LocaleResolver*](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/LocaleResolver.html) instance. Let’s assume that your modern browser sets the *Accept* header correctly and that the *AcceptHeaderLocaleResolver* is used by default.

During rendering, the *ModelAndView* object could already contain a reference to a selected view, or just a view name, or nothing at all if the controller was relying on a default view.

Since both *hello()* and *login()* methods specify the desired view as a *String* name, it has to be looked up by this name. So, this is where the *viewResolvers* list comes into play:

1

```
for (ViewResolver viewResolver : this.viewResolvers) {
```

2

```
    View view = viewResolver.resolveViewName(viewName, locale);
```

3

```
    if (view != null) {
```

4

```
        return view;
```

5

```
    }
```

6

```
}
```



This is a list of [*ViewResolver*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/ViewResolver.html) instances, including our *ThymeleafViewResolver* provided by the *thymeleaf-spring5* integration library. This resolver knows where to search for the views, and provides the corresponding view instances.

After calling the view’s *render()* method, Spring finally completes the request processing by sending the HTML page to the user’s browser:

## REST Support

Beyond the typical MVC scenario, we can also use the framework to create REST web services.

Simply put, you can accept a Resource as an input, specify a POJO as a method argument, and annotate it with *@RequestBody*. You can also annotate the method itself with *@ResponseBody* to specify that its result has to be transformed directly to an HTTP response:

1

```
import org.springframework.web.bind.annotation.RequestBody;
```

2

```
import org.springframework.web.bind.annotation.ResponseBody;
```

3

```

```

4

```
@ResponseBody
```

5

```
@PostMapping("/message")
```

6

```
public MyOutputResource sendMessage(
```

7

```
  @RequestBody MyInputResource inputResource) {
```

8

```

```

9

```
    return new MyOutputResource("Received: "
```

10

```
      + inputResource.getRequestMessage());
```

11

```
}
```



This is also possible thanks to the extensibility of Spring MVC.

To marshall the internal DTOs to a REST representation, the framework makes use of the [*HttpMessageConverter*](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html) infrastructure. For example, one of the implementations is *MappingJackson2HttpMessageConverter*, which is able to convert model objects to and from JSON using the Jackson library.

And to further simplify the creation of a REST API, Spring introduces the *@RestController* annotation. This is handy to assume *@ResponseBody* semantics by default and avoid explicitly setting that on each REST controller:

1

```
import org.springframework.web.bind.annotation.RestController;
```

2

```

```

3

```
@RestController
```

4

```
public class RestfulWebServiceController {
```

5

```

```

6

```
    @GetMapping("/message")
```

7

```
    public MyOutputResource getMessage() {
```

8

```
        return new MyOutputResource("Hello!");
```

9

```
    }
```

10

```
}
```



## Conclusion

In this article, you’ve gone through the processing of a request in the Spring MVC framework in detail. You’ve seen how different extensions of the framework work together to provide all the magic and spare you the necessity of handling the tough parts of the HTTP protocol.