---
title: localthread的使用
date: 2020-07-03 22:58:09
categories: [多线程]
tags: [多线程]
---

​	在学习springCloud项目的时候，购物车模块中拿取用户信息的时候，使用拦截器从tonke中获得了用户的信息，放入到了localthread中，在本次的请求中（也就是在这个线程当中，随时从localthread中拿到的用户对象都是这个用户的信息）。详细的用法参照[廖雪峰老师的文章](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581251653666)。

<!--more-->

``` java
@EnableConfigurationProperties(JwtProperties.class)
public class LoginInterceptor extends HandlerInterceptorAdapter {

    @Autowired
    private JwtProperties props;

    //定义一个线程域，存放登录的对象（通常是这么使用）
    private static final ThreadLocal<UserInfo> t1 = new ThreadLocal<>();

    public LoginInterceptor() {
        super();
    }

    public LoginInterceptor(JwtProperties props) {
        this.props = props;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //查询Token
        String token = CookieUtils.getCookieValue(request, props.getCookieName());
        if (StringUtils.isBlank(token)) {
            //用户未登录,返回401，拦截
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            return false;
        }
        //用户已登录，获取用户信息
        try {
            UserInfo userInfo = JwtUtils.getUserInfo(props.getPublicKey(), token);
            //放入线程域中
            t1.set(userInfo);
            return true;
        } catch (Exception e) {
            //抛出异常，未登录
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            return false;
        }
    }
    // 这个方法在respone信息传回以后执行，需要将当前线程线程域中的东西删除，以备接收下一个线程或他用户
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        //过滤器完成后，从线程域中删除用户信息
        t1.remove();
    }

    /**
     * 获取登陆用户（直接在需要的地方调用）
     * @return
     */
    public static UserInfo getLoginUser() {
        return t1.get();
    }
}
```

目前也就想到这，后面想到了，再补充吧！