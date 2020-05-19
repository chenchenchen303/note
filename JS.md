## JS

- 概念：一门客户端脚本语言，可以用来增强用户和html页面的交互，可以控制html元素，让页面有一些动态效果，增强用户体验



- 组成：ECMScript + BOM + DOM



- 与html结合方式

  - 内部js：在html标签中中写js

    - **可定义在html任何地方，但会影响执行顺序**
    - **可以定义多个**

  - 外部js：通过src属性引入外部的js文件

    

------



### ECMScript

#### 注释（与java一样）

- 单行注释：//
- 多行注释：/**/



------



#### 数据类型

- 基本数据类型
  - **number：数字 ，小数/整数/NaN**
  - **String：字符串，字符/字符串，单引号和双引号都可以**
  - boolean：true和false
  - null：一个对象为空的占位符
  - **undefined：未定义。如果一个变量没有给初始化值，默认赋值undefined**



- 数据类型转换

  - **注意：在js中，会按照运算符对数据的要求自动进行类型转换**

    

  - **其他类型转number**

    - String转number，按字面值转换，如果字面值转换不了，则转换成NaN

    - boolean转number，true为1，false为0

    - null 是0，undefined 是NaN

      ```java
      var a = +"123";		 //结果123
      var b = +"a123";	 //结果NaN
      var c = +"123a";	 //结果NaN
      var d = +true;		 //结果为1
      ```

  - 其他类型转boolean

    - number转boolean：0或NaN是false，其他数值是1

    - string转boolean：除了空字符串，其他都是true

    - 对象转boolean：都是true

    - null 与 undefined 转boolean：都是false

      ```javascript
      var a = !!0;  	//false
      var b = !!null;	//false 
      var c = !!"";   //false
      ```

      

------



#### 变量

- 概念：一小块存储数据的内存空间，**js是弱类型的语言，申请变量存储空间时，不用指定存储数据类型**



- 定义格式：var 变量名词 = 值

  ```javascript
  var a = 5;   //var表示所有数据类型
  ```



- 可用运算符 typeof 来确认数据类型

  ```js
  var a = 5;
  alter(typeof(a));
  
  var b = null;
  alter(typeof(b));	//结果是object，不是null。注意，这个是js的一个bug
  ```

  

------



#### 运算符

- 比较运算符
  - 类型相同，直接比较
  - 类型不同，进行数据类型转化后比较
  - === ：全等于，判断数据类型是否一样，不一样则false，一样则再比较数值



------



#### 特殊语法

- switch语句中，可以接受任何数据类型



- js语句可以不加分号



- **定义变量可以使用var关键字，也可以不用**
  - **用：定义的变量是局部变量**
  - **不用：定义的变量是全局变量**



------



#### 基本对象

- **function对象：函数对象**

  - 构造方法

    - function 方法名(参数列表){方法体}

    - var 方法名 = function(参数列表){方法体}

    ```javascript
    function a(){...}
    
    var b = function(){...}
    ```

    

  - 特点

    1. 形参不用写数据类型
    2. 方法是一个对象，方法重名时会覆盖原方法
    3. 调用方法时，参数可以不符合方法的参数列表

    ```javascript
    function a(aa,bb){...}
    
    a(1);			//少了参数
    a(1,1,1);		//多了参数
    a();			//没有参数
    ```

    

  - 属性

    - length：接收的参数个数
    - arguments：接受的参数数组



------



- **Array对象：数组对象，类似于java中的集合**

  - 创建

    - var arr =  new Array(元素列表)
    - var arr = new Array(默认长度)
    - var arr = [元素列表]

    

  - **特点**

    1. **数组中数据类型不是唯一的，可以有多样、**
    2. **数组长度是可变的，没有初始化的值为undefined**

    

  - 方法

    - join(参数)方法：对数组各个元素之间的分隔符重新设置，换成参数作为分隔符
    - push(参数)方法：向数组末尾添加一个元素，并放回数组长度



------



- Date对象：日期对象

  - 创建

    - var date = new Date()

    

  - 方法

    - toLocaleString()：返回当前date对象对应的时间本地字符串格式
    - getTime()：返回当前时间距标准时间的毫秒值





- Math对象：数学对象
  - math对象不用创建，直接调用



- Number、String：对应的包装类



------



- **RegExp对象：正则表达式对象**

  - 概念：定义字符串组成规则的对象

    

  - 创建

    1. var reg = new RegExp("正则表达式");
    2. var reg = /正则表达式/

    

  - **方法**

    - **test(参数)方法：验证参数是否符合正则表达式规定的规则**

    

  - **规则**

    - **单个字符：[]**
      - **\w：单个单词字符**
      - **\d：单个数字字符**
    - **量词符号**
      - **?：表示出现0次或1次**
      - ***：表示出现0次或多次**
      - **+：表示出现1次或多次**
      - **{m,n}：表示m<=数量<=n**
    - 特殊符号
      - ^：开始符号
      - $：结束符号

```javascript
var regexs = /^(?:(?:[0-2][0-3])|(?:[0-1][0-9])):[0-5][0-9]$/;  //时间日期格式
var reg = /^\w{6,12}$/;		//规定字符串长度6~12
var a = "abc";
reg.test(a);				//false
```



------



- **Global对象：全局对象**

  - 概念：不用创建对象，可以直接调用里面的方法

    

  - 方法

    - 编解码方法

      - encodeURI()：url编码
      - decodeURI()：url解码

    - parseInt(参数)方法

      - 把参数逐一转换为数字，知道遇到不是数字结束

      ```javascript
      var a = "a123";
      var b = "12a3";
      
      parseInt(a);  //NaN
      parseInt(b);  //12
      ```

    - isNaN(参数)：判断参数是否为NaN



------



### DOM对象

- **概念：每个载入浏览器的 HTML 文档都会成为 Document 对象，Document 对象是 Window 对象的一部分，可通过 window.document 属性对其进行访问**



- 分类

  - **核心 DOM -- 针对任何结构化文档的标准模型**
    - **Document：文档对象**
    - **Element：元素对象**
    - Attribute：属性对象
    - Text：文本对象
    - Comment：注释对象
    - **Node：节点对象，其他五个的父对象**
  - XML DOM --针对XML文档的标准模型

  - **HTML DOM --针对HTML文档的标准模型**



- **Document对象**

  - 创建

    - window.document（或者直接document）

  - 方法

    - **getElementById(参数)：获取id为参数的标签对象**
      - 获取对象后，就可以设置该标签的一些属性来达到目的

    ```javascript
    <h1 id="aa">
        这是aa
    </h1>
    
    <script>
        var obj = doucment.getElementById("aa")
    	......
    </script>
    ```

    - createElement()：创建一个元素对象



- Element对象

  - 获取/创建：通过document对象

    

  - 方法

    - removeAttribute：删除属性
    - setAttritube：设置属性

    

------



### BOM对象

- 概念：浏览器对象模型，将浏览器各个组成部分封装成各个对象



- 种类
  - Navigator对象：浏览器对象
  - Screen对象：显示器屏幕对象
  - **History对象：历史记录对象**
  - **Location对象：地址栏对象**
  - **Window对象：窗口对象**





- **window对象**

  - 特点

    - **不需要创建对象，可以直接使用。 window.方法名**
    - **window应用甚至可以不用，直接用方法，比如alert方法**

    

  - 方法

    - **与弹出框有关的方法**

      - **alert()：警告框**
      - **confirm()：确认框**
        - **用户点击确认按钮，返回true**
        - **用户点击取消按钮，返回false**
      - prompt()：用户输入框，返回用户输入的值

      

    - **与打开关闭有关的方法**

      - **open()：打开新窗口，若无参数则是空窗口，若有url参数，则打开url对应页面，返回值是新窗口对象**
      - **close()：关闭调用该方法的对象的窗口**

    

    - **与定时器相关的方法（轮播图）**

      - setTimeout()：在指定的毫秒数后执行函数（一次性定时器）
        - 参数1，要执行的代码
        - 参数2，毫秒值
      - clearTimeout()：取消由setTimeout()方法设置的timeout

      ```java
      var id = setTimeout("alter(3)",3000);
      clearTimeout(id);
      ```

      - **setInterval()：按照指定的周期（以毫秒记）来调用函数（循环定时器）**
        - **参数1，要执行的代码**
        - **参数2，毫秒值**
      - **clearInterval()：取消由setInterval()设置的timeout**

      ```java
      var id = setInterval("alter(3)",3000);
      clearInterval(id);
      ```

    

  - 属性

    - 获取其他BOM对象

      - navigator对象：浏览器对象
      - screen对象：显示器屏幕对象
      - history对象：历史记录对象
      - location对象：地址栏对象

      ```javascript
      var a = window.screen;
      var a = screen;		//也可以省略
      ```

    - 获取DOM对象

      - document

      ```javascript
      var a = window.document;
      var a = document;		//也可以省略
      ```

      

------



- Location对象

  - **方法**

    - **reload()：刷新当前页面**

    

  - **属性**

    - **href：当前的url值，若重新赋值则会跳转到对应的url页面**

    ```javascript
    var a = location.href;				//获取当前页面的url，这里实际上是省略了window
    location.href = "www.baidu.com";	//跳转到百度
    ```

    

------



### 事件

- 概念：某些组件执行某些操作后，执行指定代码



- 常见事件

  - 点击事件

    - onclick：单击事件
    - ondblclick：双击事件

    

  - 焦点事件

    - onblur：失去焦点
    - onfocus：获得焦点

    

  - 加载事件

    - onload：一张图片或者一个页面得到加载

    

  - 鼠标事件

    - onmousedown：鼠标按钮按下
    - onmouseup：鼠标按钮被松开
    - onmousemove：鼠标被移动
    - onmouseover：鼠标移到某元素之上
    - onmouseout：鼠标从某元素移开

    

  - 键盘事件

    - onkeydown：某个按键被按下
    - onkeyup：某个按键被松开
    - onkeypress：某个按键被按下并松开

    

  - 选择和改变事件

    - onchange：域的内容被改变
    - onselect：域的内容被选中

    

  - 表单事件

    - onsubmit：确认按钮被点击
      - **返回false，表单被阻止提交**
    - onreset：重置按钮被点击



```javascript
//阻止表单提交

//第一种方法
<script>
    document.getElementById("form").onsumit = function(){
    return false;}
</script>
<form id="form">
 	...
</form>

//第二种方法
<script>
    function checkform(){
    return false;}
</script>
<form id="form" onclick="return checkform()">
 	...
</form>
```



- **注意：假如js代码写在前面，要用入口函数**

  ```javascript
  <script>
      window.onload = function(){		//入口函数，表示该页面所有组件加载完成后调用此函数
      		......
  		}
  </script>
  ```

  



- 补充
  - 全选、全不选、反选、焦点事件：js最后案例视频
  - js前台校验：js最后案例视频