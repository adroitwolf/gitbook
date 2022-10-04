# springboot中对AOP技术的了解和使用

我们在项目中如果使用spring的框架的话，aop技术多多少少也接触过，所以打算在这里总结一下AOP的技术核心和常用方法。

## AOP的基本使用方法 --> 全局AOP监听

我们现在要在Controller层做一个监听，每次进入controller的方法的时候，都启动AOP进行方法检查等操作。

接下来我们看一下源代码：

Controller层：

```java

@RestController
public class PageController {  //这个类在com.demo.controller包下
@RequestMapping("/hello")

   public Map<String,String> helloWorld(){
       Map<String,String > map = new HashMap<>();

       System.out.println("代码执行中");
       map.put("hello","world");
       System.out.println("继续执行");
       return map;
   }
}
```

AOP：

```java


/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019 2019/6/2 16:06
 * Description: ://TODO ${END}
 */
@Slf4j
@Aspect
@Component
public class WebLogAspect {

 @Pointcut("execution(public * com.demo.controller.*.*(..))")
    private void weblog(){}

    @Before("weblog()")
    private void doBefore(JoinPoint joinPoint) throws Throwable {
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        log.info("URL: "+request.getRequestURL().toString());
        log.info("Method" + request.getMethod());
        log.info("HTTP_METHOD: "+request.getMethod());

        log.info("IP: "+request.getRemoteAddr());
        Enumeration<String> enu = request.getParameterNames();
        while (enu.hasMoreElements()) {
            String name = enu.nextElement();
            log.info("name:{},value:{}", name, request.getParameter(name));
        }
    }

    @AfterReturning(returning = "ret", pointcut = "weblog()")
    public void doAfterReturning(Object ret) throws Throwable {
        log.info("RESPONSE: " + ret);
    }
}
```

* 下面讲解一下这个AOP的常用技术

**@Aspect** 标记这是一个切面，如果没有这个注解，那么这个方法不启动

**Pointcut** 切点，后面的execution是决定监听哪里的方法

**Befoe** 前置通知

**After** 后置通知

## 下面是我常用的AOP方法

利用自定义注解决定切点

下面我们自定义一个注解

```java
@Target(ElementType.METHOD)
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface Cache {
  boolean ifDelete() default false;
}
```

在我们的Controller层方法上加入这个注解

```java

@RequestMapping("/hello")
   @Cache(ifDelete = true)
   public Map<String,String> helloWorld(){
       Map<String,String > map = new HashMap<>();

       System.out.println("代码执行中");
       map.put("hello","world");
       System.out.println("继续执行");
       return map;
   }
```

AOP层：

```java

@Slf4j
@Aspect
@Component
public class TestAsp {

    @Around("@annotation(com.demo.flag.Cache)")
    public Object testJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {

        System.out.println("进入aop");

        return joinPoint.proceed();
        System.out.println("代码执行aop后");
    }
}
```

我们访问一下这个地址，结果是这个样子的

```bash
>进入aop
>代码执行中
>继续执行
>代码执行aop后
```

这里不需要PointCut，而是利用@annotation将所有有上面这个注解的方法都会进入这个AOP方法。

其实这个@Around是一个环绕通知，你可以在这里通过jionPoint获得到访问者IP，变量参数等等。

这个joinPoint.proceed()方法是让controller方法的必要条件。

joinPoint.proceed()方法后的代码是处理完方法后需要执行的代码，这样就可以想象成这个Around将controller层的某个方法'包裹'起来了。

## JoinPoint的常用方法

* 获取到方法的注解

```java
MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();

        Cache annotation = methodSignature.getMethod().getAnnotation(Cache.class);
```
