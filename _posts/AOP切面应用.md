### AOP切面应用

代码解析：

##### `@Aspect  `
把当前类作为一个切面来供容器来进行读取

##### `@Pointcut`
- 是植入Advice的触发条件。每个Pointcut的定义包括2部分，一是表达式，二是方法签名。方法签名必须是 public及void型。**Pointcut中的方法只需要方法签名，而不需要在方法体内编写实际代码。**

  - 有多种方式来进行各种匹配
    args()
    @args()
    **execution()**
    this()
    target()
    @target()
    within()
    @within()
    **@annotation**
  - **@within(匹配被注解的类)和@target(匹配调用被注解的类)针对类的注解,@annotation是针对方法的注解，都是用来匹配自定义注解**
  - 用的最多的是**execution**来实现匹配各种范围

##### `@Before`

标识一个前置增强方法，相当于BeforeAdvice的功能，相似功能的还有

##### `@Around`

环绕增强，相当于MethodInterceptor

##### `@AfterReturning`

后置增强，相当于AfterReturningAdvice，方法正常退出时执行

##### `@AfterThrowing`

异常抛出增强，相当于ThrowsAdvice

##### `@After`

 final增强，不管是抛出异常或者正常退出都会执行

```
@Slf4j
@Aspect  //把当前类作为一个切面来供容器来进行读取
@Component
public class ClogAspect {

    @Pointcut("@within(com.shebao.growth.config.Clog)")
    public void pointCut1(){

    }

    @Before("pointCut1()")
    public void before(JoinPoint joinPoint) throws Throwable {
        try{
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Method method = signature.getMethod();
            String methodName = method.getName();
            Class<?>[] parameterTypes = method.getParameterTypes();
            Class<?> aClass = joinPoint.getTarget().getClass();
            Method method1 = aClass.getMethod(methodName, parameterTypes);
            ApiOperation annotation = method1.getAnnotation(ApiOperation.class);
            String value = "";
            if(null != annotation){
                value = annotation.value();
            }
            //获取参数名称
            DefaultParameterNameDiscoverer discover = new DefaultParameterNameDiscoverer();
            String[] parameterNames = discover.getParameterNames(method1);
            //入参
            Object[] args = joinPoint.getArgs();
            log.info("============业务方法:{}【开始】!",value);

            if(null != parameterNames && null != args && parameterNames.length > 0 && args.length > 0){
                StringBuffer stringBuffer = new StringBuffer();
                for (int i = 0; i < args.length; i++) {
                    try{
                        if(!"response".equals(parameterNames[i]) && !"request".equals(parameterNames[i])){
                            stringBuffer.append(parameterNames[i]).append(":").append(null == args[i] ? "" : String.valueOf(args[i])).append("  ");
                        }
                    }catch (Exception e){
                        //吃掉异常，不影响接口的正常请求
                    }
                }
                log.info("【入参】:" + stringBuffer.toString());
            }
        }catch (Exception e){
            //吃掉异常，不影响接口的正常请求
        }
    }

    @After("pointCut1()")
    public void after(JoinPoint joinPoint) throws NoSuchMethodException {
        try{
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Method method = signature.getMethod();
            String methodName = method.getName();
            Class<?>[] parameterTypes = method.getParameterTypes();
            Class<?> aClass = joinPoint.getTarget().getClass();
            Method method1 = aClass.getMethod(methodName, parameterTypes);
            ApiOperation annotation = method1.getAnnotation(ApiOperation.class);
            String value = "";
            if(null != annotation){
                value = annotation.value();
            }
            log.info("============业务方法:{}【结束】!",value);
        }catch (Exception e){
            //吃掉异常，不影响接口的正常请求
        }
    }
}
```