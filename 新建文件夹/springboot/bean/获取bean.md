#### 普通代码中获取bean的几种方式

最近在项目中，因代码模式要求，需要在普通类中去主动调用bean实例，经过参考分析，做如下的整理。

> ```
> 在初始化时保存ApplicationContext对象 
> 通过Spring提供的utils类获取ApplicationContext对象 
> 继承自抽象类ApplicationObjectSupport 
> 继承自抽象类WebApplicationObjectSupport 
> 实现接口ApplicationContextAware 
> 通过Spring提供的ContextLoader
> 注意：通过继承类或者实现接口都需要交给spring去托管，否则无法使用ApplicationContext对象
> ```

------

###### 在初始化时保存ApplicationContext对象

```
ApplicationContext ac = new FileSystemXmlApplicationContext("applicationContext.xml"); 
ac.getBean("userService");12
```

------

###### 通过Spring提供的utils类获取ApplicationContext对象

```
ApplicationContext ac1 = WebApplicationContextUtils.getRequiredWebApplicationContext(ServletContext sc); 
ApplicationContext ac2 = WebApplicationContextUtils.getWebApplicationContext(ServletContext sc); 
ac1.getBean("beanId"); 
ac2.getBean("beanId");1234
```

实例

```
public class MyServlet extends HttpServlet {
 public void init() throws ServletException {         
        super.init();
        ServletContext servletContext = getServletContext(); 
        WebApplicationContext ctx =WebApplicationContextUtils.getWebApplicationContext(servletContext);
        (Bean)ctx.getBean("bean");
  }  
}12345678
```

------

###### 继承自抽象类ApplicationObjectSupport

> 说明：抽象类ApplicationObjectSupport提供getApplicationContext()方法。能够方便的获取ApplicationContext。
> Spring初始化时。会通过该抽象类的setApplicationContext(ApplicationContext context)方法将ApplicationContext 对象注入。

------

###### 继承自抽象类WebApplicationObjectSupport

> 说明：类似上面方法。调用getWebApplicationContext()获取WebApplicationContext

------

###### 实现接口ApplicationContextAware

```
@Component
public class SpringContextUtil implements ApplicationContextAware {
    /**
     * Spring应用上下文环境
     */
    private static ApplicationContext applicationContext;

    /**
     * 实现ApplicationContextAware接口的回调方法。设置上下文环境
     *
     * @param applicationContext
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        SpringContextUtil.applicationContext = applicationContext;
    }

    /**
     * @return ApplicationContext
     */
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    /**
     * 获取对象
     *
     * @param name
     * @return Object
     * @throws BeansException
     */
    public static Object getBean(String name) throws BeansException {
        return applicationContext.getBean(name);
    }

    /**
     * 获取对象通过Class
     *
     * @param cls
     * @return Object
     * @throws BeansException
     */
    public static <C> Object getBean(Class<C> cls) throws BeansException {
        return applicationContext.getBean(cls);
    }123456789101112131415161718192021222324252627282930313233343536373839404142434445
```

------

###### 通过Spring提供的ContextLoader

```
WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
wac.getBean(beanID);
```