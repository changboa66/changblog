---
layout: mypost
title: Spring-MVC
categories: [Spring]
---

#### Spring MVC执行流程
```
1. 发起请求request到前端控制器DispatcherServlet
2. 前端控制器根据URL请求处理映射器HandlerMapping查找Handler。
3. 处理映射器HandlerMapping向前端映射器DispatcherServlet返回HandlerExecutionChain
4. 前端控制器调用处理适配器HandlerAdapter执行Handler。
5. Handler执行完成后给处理映射器返回ModelAndView。
6. 处理映射器向前端控制器返回ModelAndView。
7. 前端控制器请求试图解析器解析视图。
8. 前端控制器渲染视图，响应给用户response。
```
