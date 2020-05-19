## JQuery

- 概念：一个js框架，封装了js的一些操作，使得进行操作更加简单简洁，**本质就是一些js文件，封装了js的原生代码**



- 使用步骤
  1. 下载JQuery
  2. 导入jar包
  3. 使用



------



### JQuery和JS区别和转换

- 区别
  - **JQuery获取对象，得到是一个对象；JS获取对象，得到的是一个数组对象**
  - JQuery对象在操作时，更加简单
    - 比如一个html有多个div标签，获取对象
      - 用JS获取后得到一个数组，**如果要设置通用属性要用for循环**
      - 用JQuert获取后得到一个对象，但是他也能进行数组操作，**设置通用属性直接设置就行，因为是一个对象**
  - **JQuert对象和JS对象方法是不通用的，需进行转换**



- **转换**
  - **jq 变 js ：jq对象[索引] 或者 jq对象.get(索引)**
  - **js 变 jq ：$(js对象)**



------



### 基础语法

- **事件绑定**

  - 步骤

    1. 获取组件
    2. 调用JQuery方法
       - **注意：JQuert中有自己的方法，不能调用JS中的方法**

    ```javascript
    <input type="button" value="点我" id="a">
        
    <script>
        $("#a").click(function(){		//JQ中的点击事件
        alert("abc")
    })
    </script>
    ```





- **入口函数**

  ```javascript
  //js写法
  window.onload() = function(){
      ......
  }
      
  //JQ写法
  $(function){
      ......
  }
  ```
  - **区别**
    - **JS的入口函数只能写一个，写多个会覆盖**
    - **JQ可以写多个入口函数**





- **样式控制**

  - 使用css方法
  - 格式：$(选择的标签).css(属性值,设置值)

  ```javascript
  $("#a").css("backgroundColor","red");  //设置背景色为红色
  ```

  

------



### 选择器

#### 基本选择器

- 标签选择器（元素选择器）
  - 语法：$("html标签名") ，获得所有匹配标签名称的元素
- id选择器
  - 语法：$("#id值")，获得与指定id属性匹配的元素
- 类选择器
  - 语法：$(”.class值“)，获得与指定的class属性值匹配的元素
- 并集选择器
  - 语法：$("选择器1,选择器2...")，获得多个选择器选中的所有元素





#### 层级选择器

- 后代选择器
  - 语法：$("A B")，获得它的子类元素及子类元素的子类元素（孙子类元素）
- 子选择器
  - 语法：$("A>B")，只获得它的子类元素，不管孙子类元素





#### 属性选择器

- 属性名称选择器

  - 语法：$("A[属性名]")，包含指定属性的选择器

- 属性选择器

  - 语法

    - A指的是要选择的标签
    - $("A[属性名=‘值’]")，包含指定属性等于指定值的选择器
    - $("A[属性名!=‘值’]")，包含指定属性不等于等于指定值**和不含有此属性**的选择器

    - $("A[属性名^=‘值’]")，包含指定属性以指定值开始的选择器
    - $("A[属性名$=‘值’]")，包含指定属性以指定值结束的选择器
    - $("A[属性名*=‘值’]")，包含指定属性包含指定值的选择器

- 复合属性选择器

  - 语法：$("A[属性名=‘值’] [条件]...")，包含多个属性条件选择器





#### 过滤选择器





#### 表单过滤选择器