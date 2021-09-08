#### Interceptor拦截器的使用

##### 拦截器的三个阶段

- preHandle

  该方法将在Controller处理**之前进行调用。**返回true才能继续执行

  SpringMVC中的Interceptor拦截器是链式的，可以同时存在多个Interceptor，然后SpringMVC会根据声明的前后顺序一个接一个的执行，而且所有的Interceptor中的preHandle方法都会在Controller方法调用之前调用。

- postHandle

  在Controller的方法调用**之后执行**，但是它会在DispatcherServlet进行视图的渲染**之前执行**，也就是说在这个方法中你可以对ModelAndView进行操作。

- afterCompletion

  **该方法将在整个请求完成之后，也就是DispatcherServlet渲染了视图执行，不能再修改数据。**

  （这个方法的主要作用是用于清理资源的）

##### 编写拦截器

```
@Component
@Slf4j
public class AuthInterceptor extends HandlerInterceptorAdapter {

    @Resource
    private IThirdApiService ssoSystemService;

    public static final int RESULT_NUM = 22000;
    public static final int NOT_LOGIN = 22002;


    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return auth(request, response);
    }


    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        // ignore
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        ResponseT.threadLocal.remove();
    }

    private boolean auth(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String requestUri = request.getRequestURI();
        if (request.getMethod().equals(RequestMethod.OPTIONS.name())) {
            return true;
        }
        String ttlValue = request.getHeader("ttl");

        if(StringUtils.isBlank(ttlValue)){
            ttlValue = request.getHeader("ttl");
        }

        if (StringUtils.isEmpty(ttlValue)) {
            return handleNotAuthorized(response, true);
        }

        try {
            ResponseT<LoginUserInfo> responseT = ssoSystemService.getLoginUser(ttlValue);
            log.info("权限拦截器拦截到path:{} 校验结果:{}", requestUri, responseT);
            if (responseT.getCode() != RESULT_NUM) {
                return handleNotAuthorized(response, false);
            }
            ResponseT.threadLocal.set(responseT);
        } catch (Exception ex) {
            log.error("ttl error", ex);
            return handleNotAuthorized(response, false);
        }
        return true;
    }

    private boolean handleNotAuthorized(HttpServletResponse response, boolean isTtlEmpty) throws IOException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json; charset=utf-8");
        ResponseT responseT = new ResponseT();
        responseT.setCode(NOT_LOGIN);
        if (isTtlEmpty) {
            responseT.setMessage("登录校验失败:非法请求");
        } else {
            responseT.setMessage("登录校验失败");
        }
        log.info("权限拦截器拦截校验失败:{}", responseT);
        response.getWriter().write(JSON.toJSONString(responseT));
        response.getWriter().flush();
        return false;
    }

}
```



##### 最后将拦截器配置到项目里

```
@Configuration
public class InterceptorConfiguration implements WebMvcConfigurer {

    @Resource
    AuthInterceptor authInterceptor;

    @Autowired
    RedisUtil redisUtil;

    /**
     * addInterceptors.
     * @param registry registry.
     */
    @Override
    public void addInterceptors(final InterceptorRegistry registry) {
        registry.addInterceptor(new WebLogInterceptor())
                .addPathPatterns("/**")
                .excludePathPatterns("/51shebao/**", "/loginPage.htm");
        //保险大厅登录校验
        registry.addInterceptor(authInterceptor).addPathPatterns("/bloc/**");
    }
}
```