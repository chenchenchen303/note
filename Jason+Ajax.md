## Jason

- 概念：**js对象表示法，用于封装对象，类似于java中的封装**。用于存储和交换数据，类似于XML，但比它快



### 语法

- **基本规则**

  1. **数据用键值对存储，键名可用引号或者不用**
  2. **数据用逗号分隔、**
  3. **花括号保存对象**
  4. **方括号保存数组**
  5. **可以互相嵌套**

  ```javascript
  //基本格式
  var a = {"name":"张三","age":23};
  //嵌套格式，{}嵌套[]，对象里面存储了一个数组
  var b = {persons:[{"name":"张三","age":24},{"name":"李四","age":24}]};
  //嵌套格式,[]嵌套{}，对象数组
  var c = [{"name":"张三","age":24},{"name":"李四","age":24}];
  ```

  

- **获取数据**0

  - jason对象.键名
  - jason对象[”键名“] ，这种要加引号
  - 数组对象[索引]，把jason对象看成索引

  ```javascript
  //第一种
  var a = {"name":"张三","age":23};
  var name = a.name;
  //第二种
  var b = {persons:[{"name":"张三","age":24},{"name":"李四","age":24}]};
  var name = b.persons[1].name;
  //第三种
  var c = [{"name":"张三","age":24},{"name":"李四","age":24}];
  var name = c[1].name
  
  
  //遍历一个对象
  var a = {"name":"张三","age":23};
  for(var key in a){
      alert(key+":"+a[key]);		//输出name:张三 age:23
  }
  
  //遍历所有值
  var c = [{"name":"张三","age":24},{"name":"李四","age":24}];
  for(var i = 0; i< c.lenth; i++){
      var p = c[i];
      for(var key in p){
          alter(key+":"+p[key]);
      }
  }
  ```

  

------



### java对象与jason的互转

- 利用jackson解析器



- **Java转Jason**

  1. **导入jar包**
  2. **创建jackson核心对象 ObjectMapper**
  3. **调用ObjectMapper的相关方法进行转换**

  ```java
  Person p = new Person("张三",18);
  /*
  	writeValue(参数1,obj)
  		参数1：
  			File：将obj对象转换为JSON字符串，保存到指定文件中
  			Writer：将obj对象转换为JSON字符串，并将jason数据填充到字符输出流中
  			OutputStream：将obj对象转换为JSON字符串，并将json数据填充到字节输出流中
  			
  	writeValueAsString(参数)：将参数对象转为json字符串
  */
  ObjectMapper mapper = new ObjectMapper();
  String jason = mapper.writeValueAsString(p);
  ```
  - **注意：如果对象是List集合，结果是一个数组；如果对象是Map集合，结果和普通对象一样**





- jason转Java

  1. **导入jar包**
  2. **创建jackson核心对象 ObjectMapper**
  3. **调用ObjectMapper的相关方法进行转换**

  ```java
  String json = "{\"gender\":\"男\",\"name\":\"张三\"}";
  ObjectMapper mapper = new ObjectMapper();
  //readValueAsString(参数1，参数2)：参数1是json对象，参数2是要转化的对象.class
  Person person = mapper.readValueAsString(json,Person.class);
  ```

  



- 注解

  - @JasonIgnore：忽略这个属性，不转化为json对象
  - @JasonFormat：属性格式化

  ```java
  public class Person{
      private String name;
      private int age;
      @JasonIgnore
      private String sex;   	//当将一个person对象转换成json对象后里面没有sex属性
      @JasonFormat(pattern = "yyyy-MM-dd")
      private Date birthday;	//到时候日期格式会被控制成 年-月-日，而不是毫秒值
  }
  ```

  







## Ajax

- 概念：异步的js 与 XML，异步指的是客户端不必一定等服务端作出回应才能做别的事情



- 功能：无需重新加载整个页面情况下，重新加载某些组件，对局部进行刷新



- **原理图**

  ![1587127755(1)](E:\md资料图片\1587127755(1).png)



### 实现方式

- 原生JS实现方式（了解）



- **JQeury实现方式**

  - 注意：使用前先引入JQeury相关jar包

  

  - **$.ajax()，通用方式**

    - **语法：$.ajax({键值对})，在键值对中设置属性，各个键值对用逗号分离**

    ```javascript
    $.ajx({
       url:"loginServlet",					//请求路径
        type:"POST",						//请求方式，不写默认GET
        //data:"username=jack&age=23"		//请求参数
        data:{"username":"jack","age":23},	//用json格式比较好
        success:function(data){				//响应成功后的回调函数
            alert(data);
        },
        error:function(){					//响应错误后的回调函数
        alert("出错啦");
        },
        dataType::"json"					//设置接收到的响应数据的格式
    }
    });
    ```

  

  

  - **$.get()，表示用get方法发送异步请求**

    - **语法：$.get(url,[data],[callback],[type])，后两个参数可写可不写**
      - 参数
        1. url：请求路径
        2. data：请求参数
        3. callback：回调函数
        4. type：响应结果的类型

    ```javascript
    $.get("loginServlet",{username:"rose",age:23},function(data){
        alert(data);
    },"text")
    ```

    

  

  - **$.post()，表示用post方法发送异步请求**

    - **语法：$.get(url,[data],[callback],[type])，后两个参数可写可不写**
      - 参数
        1. url：请求路径
        2. data：请求参数
        3. callback：回调函数
        4. type：响应结果的类型

    ```javascript
    $.post("loginServlet",{username:"rose",age:23},function(data){
        alert(data);
    },"text")
    ```

    







## 案例：检验用户名是否存在