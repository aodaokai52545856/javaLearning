# [Spring-boot BeanFactory运行时动态注册bean](https://blog.csdn.net/Frame_M/article/details/88841299)

### 1、定义一个没有被Spring管理的`Controller`

```java
public class UserController implements InitializingBean{

    private UserService userService;

    public UserService getUserService() {

        return userService;

    }

    public void setUserService(UserService userService) {

        this.userService = userService;

    }

    @Override

    public void afterPropertiesSet() throws Exception {

        System.out.println("我是动态注册的你,不是容器启动的时候注册的你");

    }

    public String toAction(String content){
        return "-->" +  userService.doService(content);
    }
}
```

需要注意的是,如果要注入UserService,需要提供它的getter和setter方法

现在启动springboot工程,显而易见这个类是不会被Spring管理的，接下来我们定义一个获取Spring上下文的工具类，如下

工具类

```java
public class SpringContextUtil {
    private static ApplicationContext applicationContext;
    //获取上下文
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
    //设置上下文
    public static void setApplicationContext(ApplicationContext applicationContext) {
        SpringContextUtil.applicationContext = applicationContext;
    }
}
```

### 二、在Springboot的启动类中,保存当前Spring上下文,代码如下：

```java
@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        ApplicationContext app = SpringApplication.run(HelloApplication.class, args);
        SpringContextUtil.setApplicationContext(app);
    }
}
```

### 三、在另一个被Spring管理的容器中，写如下方法，进行bean的动态注册

```java
@Controller
public class ApplicationConfig{
@GetMapping("/bean")
public String registerBean() {

    //将applicationContext转换为ConfigurableApplicationContext
    ConfigurableApplicationContext configurableApplicationContext = (ConfigurableApplicationContext) SpringContextUtil.getApplicationContext();
    
    // 获取bean工厂并转换为DefaultListableBeanFactory
    DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) configurableApplicationContext.getBeanFactory();

    // 通过BeanDefinitionBuilder创建bean定义
    BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(UserController.class);

    // 设置属性userService,此属性引用已经定义的bean:userService,这里userService已经被spring容器管理了.
    beanDefinitionBuilder.addPropertyReference("userService", "userService");

    // 注册bean
    defaultListableBeanFactory.registerBeanDefinition("userController", beanDefinitionBuilder.getRawBeanDefinition());

    UserController userController = (UserController) defaultListableBeanFactory .getBean("userController");
    return userController.toAction("动态注册生成调用");
}}
```

 

