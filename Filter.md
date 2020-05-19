## Filter

- 概念：访问服务器资源时，过滤器可以将请求拦截下来，完成一些特殊的功能



- **应用**
  - **登录验证**
  - **统一编码处理**
  - **敏感字符过滤**
  - **但用户登录**



------



### 快速入门

- 注解配置

  ```java
  @WebFilter("拦截的资源目录")
  ```

  

- Filter有三个方法

  - init初始化方法：一般用于加载资源，只在创建过滤器时调用一次

  

  - **doFileter拦截逻辑方法：过滤器主方法，当访问请求拦截的资源目录时，就会调用**
    - **若无使用放行方法，则无法请求资源**
      - **放行方法：chain.doFilter(req, resp);**

  

  - destory销毁方法：一般用于回收资源，只在销毁过滤器时调用一次

```java
@WebFilter("过滤的资源目录")
public class TestFilter implements Filter {
    //销毁方法
    public void destroy() {
    }
	
    //主方法
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        chain.doFilter(req, resp);	//放行
    }

    //初始化方法
    public void init(FilterConfig config) throws ServletException {
    }
}
```



------



### 拦截路径配置（注解配置）

- 拦截一个与拦截多个资源：拦截一个资源value可以省略

```java
@WebFilter("/index.jsp")							//一个资源
@WebFilter(value = {"/index.jsp","/show.jsp"})		//多个资源
```



- 具体资源路径：只会拦截请求index.jsp的请求

```java
@WebFilter("/index.jsp")
```



- 拦截目录：拦截user目录下的所有内容（多用于servlet）
  - **注意：这个user并不是指真实目录，而是指servlet注解配置时的目录**

```java
@WebServlet("/user/stateJudgeServlet")
```

```java
@WebFilter("/user/*")
```



- 拦截所有资源：拦截所有资源

```java
@WebFilter("/*")
```



- 后缀名拦截：拦截所有jsp资源（多用于jsp）

```java
@WebFilter("*.jsp")
```



------



### 拦截方式配置（注解配置）

- 概念：过滤器会按不同的访问方式拦截资源，比如配置了直接访问拦截，假如是转发请求资源，则不会被拦截



- **配置：设置dispatcherTypes属性**

  - REQUEST：服务器直接请求资源，默认值
  - FORWARD：服务器转发请求资源
  - INCLUDE：包含跳转资源
  - ERROR：错误跳转资源
  - ASYNC：异步跳转资源

  

- 同时配置多个方式

```java
@WebFilter("*.jsp" dispatcherTypes = DispatcherType.FORWARD)	//一个
```

```java
@WebFilter("*.jsp" dispatcherTypes = {DispatcherType.REQUEST,DispatcherType.FORWARD})	
```



------



### 执行流程

- 初始化方法
  - 生命周期：服务器启动，创建Filter对象时，自动调用该方法，用于**加载资源**





- 主方法
  - 概念：过滤器先执行放行前的代码，然后放行，然后再执行放行后的代码	

  ```java
  //主方法
  public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) {
       sout("放行前");
       chain.doFilter(req, resp);	//放行
       sout("放行后");
   }
  ```

  

  - **说明**
    - **放行前的代码：表示对请求消息（request对象）的一些处理**
    - **放行后的代码：表示对响应消息（response对象）的一些处理**





- 销毁方法
  - 生命周期：服务器关闭时，Filter对象销毁，调用该方法，**用于回收资源**
    - **注意：只有服务器正常关闭时才调用该方法**



------



### 过滤器链（多个过滤器）

- 执行顺序：如果有两个过滤器1和过滤器2
  1. 过滤器1放行前
  2. 过滤器2放行前
  3. 资源
  4. 过滤器2放行后
  5. 过滤器1放行后



- **使用过滤器顺序（注解配置）**

  - 按过滤器名字排序

  

------



### 登录案例

```java
@WebFilter("/*")
public class TestFilter implements Filter {
    public void destroy() {}

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        //1.强制类型转换
        HttpServletRequest request = (HttpServletRequest) req;
        //2.获取请求资源路径
        String uri = request.getRequestURI();
        //3.判断请求是否包含登录相关资源路径
        //注意：如果登录界面含有一些CSS、JS或者验证码之类的也要允许放行
        if(uri.contains("/login.jsp") || uri.contains("checkcodeServlct")){
             chain.doFilter(req, resp);//包含，放行
        }
        else{
        	if(request.getSession.getAttritube("user") != null){//已经登录
             chain.doFilter(req, resp);//放行
            }
            else{//未登录，跳转到登录界面
             response.sentRedict("/login.jsp");
            }
        }
    }

    public void init(FilterConfig config) throws ServletException { }

}
```



------



### 过滤敏感词

- **思路：在过滤器中，给request对象进行增强。要用代理模式**

```java
@WebFilter("/*")
public class TestFilter implements Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        //1.创建代理对象，增强getParameter方法(实际也要对getParameterMap、getParameterValue增强)
        ServletRequest proxyRequest = (ServletRequest) Proxy.newProxyInstance(req.getClass().getClassLoader(), req.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object o, Method method, Object[] args) throws Throwable {
                //增强getParameter方法
                //判断是否是getParameter方法
                if(method.getName().equals("getParameter")){
                    //增强返回值
                    String value = (String)method.invoke(req,args);
                    if(value != null){
                        for(String str:list){
                            if(value.contains(str)){
                                value = value.replaceAll(str,"***");
                            }
                        }
                    }
                    return  value;
                }
                return method.invoke(req,args);
            }
        });
        //2.放行
        chain.doFilter(req,resp);
    }

    private List<String> list = new ArrayList<>();//敏感词汇文档

    public void init(FilterConfig config) throws ServletException {
        //加载配置文件（敏感词文档）
        try {
            ServletContext servletContext = config.getServletContext();
            String realPath = servletContext.getRealPath("配置文件位置");
            BufferedReader br = new BufferedReader(new FileReader(realPath));
            String line = null;
            while ((line = br.readLine()) != null) {
                list.add(line);
            }
            br.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```







## 代理模式（补充）

- 概念：代理对象代理真实对象，达到增强真实对象功能的目的



- 名词解释

  - 代理对象：用于代理的对象，**比如指定代理商**
  - 真实对象：被代理的对象，**比如生产公司**

  

  - 静态代理：有一个类文件描述代理模式
  - 动态代理：在内存中形成代理类



- **步骤**
  1. **代理对象和真实对象实现相同接口**
  2. **代理对象 = Proxy.newProxyInstance()**
  3. **使用代理对象调用方法**
  4. **增强方法**
     - **增强返回值**
     - **增强参数列表**
     - **增强方法体逻辑**





- 实例
  - 联想公司卖电脑（真实对象）
  - 代理商卖电脑（代理对象）
  - 代理公司帮联想公司卖电脑，每次收15%的费用，购买前专车接送用户，且赠送鼠标垫

```java
//共同接口
public interface SaleComputer(){
    public String sale(double money);
    
    public String show();
}
```

```java
//真实类
public class Lenovo implements SaleComputer{
    public String sale(double money){
        sout("花了"+money+"买了一台电脑");
        return "联想电脑";
    }
    
    public String show(){
        sout("展示电脑...");
    }
}
```

```java
//代理类
public class ProxyLenovo{
    public static void main(String[] args) {
        //获取真实对象
        Lenovo lenovo = new Lenovo();
        
        //动态代理增强Lenovo对象，获取代理对象
        	/*
        		三个参数
        		1.类加载器：真实对象.getClass().getClassLoader()
        		2.接口数据：真实对象.getClass().getInterfaces()
        		3.处理器：new InvocationHandler()
        	*/
        Salecomputer proxyLenovo = 		                                                 		 (Salecomputer)Proxy.newProxyInstance(lenovo.getClass().getClassLoader(),                 lenovo.getClass().getInterfaces(),new InvocationHandler() {//固定写法
            //代理逻辑编写方法，代理对象调用的所有方法都会触发该方法运行
            	/*
            		三个参数
            		1.proxy：代理对象
            		2.method：代理对象调用的方法，被封装成对象
            		3.objects：代理对象调用方法时，使用的参数
            	*/
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          	//判断是否调用sale方法
          if(method.getName().equals("sale")){
            //1.增强参数
              double money = (double) args[0];
              money = 0.85 * money;
            //2.增强方法体逻辑，在这里添加一些东西
              sout("专车接送");
            //使用真实对象调用方法
              String obj = (String)method.invoke(lenovo,args);
            //3.增强返回值
              return obj+"鼠标垫";
          }
          else{
              //使用真实对象调用方法
              Object obj = (String)method.invoke(lenovo,args);
              return obj;
          }
            }
        });
        //调用代理对象调用方法
        proxyLenovo.sale(8000);
    }
}
```





## 监听器

- 概念：功能类似swing监听者，触发时运用设置好的方法，监听servlet中的域对象，一般与监听的域对象的生命周期有关



- 配置

```java
@Listener
```



- **作用**
  - **session监听器可以监控用户在线人数**
  - **session监听器可以与过滤器使用实现单用户登录**