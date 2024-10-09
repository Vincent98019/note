

Restful是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。主要用于客户端和服务器交互类的软件，基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存机制等。

Restful风格的请求是使用“url+请求方式”表示一次请求目的的，HTTP 协议里面四个表示操作方式的动词如下：

* **GET：** 用于获取资源
* **POST：** 用于新建资源
* **PUT：** 用于更新资源
* **DELETE：** 用于删除资源

例如：

* `/user/1 GET`：得到 id = 1 的 user
* `/user/1 DELETE`： 删除 id = 1 的 user
* `/user/1 PUT`：更新 id = 1 的 user
* `/user POST`：新增 user



## HiddenHttpMethodFilter

由于浏览器只支持发送get和post方式的请求，SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

* 当前请求的请求方式必须为post
* 当前请求必须传输请求参数\_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数\_method的值，因此请求参数\_method的值才是最终的请求方式。

在web.xml中注册**HiddenHttpMethodFilter**

```xml
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 目前为止，SpringMVC中提供了两个过滤器：CharacterEncodingFilter和HiddenHttpMethodFilter，在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter，原因：

> * 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字符集的
> * request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作，而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作：`String paramValue = request.getParameter(this.methodParam);`



```java
// 查询所有员工数据
@RequestMapping(value = "/employee", method = RequestMethod.GET)
public String getEmployeeList(Model model){
    Collection<Employee> employeeList = employeeDao.getAll();
    model.addAttribute("employeeList", employeeList);
    return "employee_list";
}

// 删除
@RequestMapping(value = "/employee/{id}", method = RequestMethod.DELETE)
public String deleteEmployee(@PathVariable("id") Integer id){
    employeeDao.delete(id);
    return "redirect:/employee";
}

// 保存
@RequestMapping(value = "/employee", method = RequestMethod.POST)
public String addEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}

// 更新
@RequestMapping(value = "/employee", method = RequestMethod.PUT)
public String updateEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}
```