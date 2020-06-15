

## File类

- 概况：File类是一个用于操作文件的类，但它与操作系统无关，任何操作系统都可以使用这个类
  - 创建文件/文件夹
  - 删除文件/文件夹
  - 获取文件/文件夹
  - 判断文件/文件夹是否存在
  - 遍历文件夹
  - 获取文件大小



------



### File静态成员变量

- `String pathSeparator`：路径分隔符，window默认是`;` Linux默认是`:`

- `char pathSeparatorChar`：同上，不过类型是char



- `String separator`：文件名称分隔符，window默认是`/` Linux默认是`\`
- `char separatorChar`：同上，不过类型是char



- **注意：由于软件可能在不同系统运行，所以不能直接写死**

  ```java
  "C:\develop\a\a.txt"   //windows
  "C:/develop/a/a.txt"   //linux
  "C:"+File.separator+"develop"+File.separator+"a"+File.separator+"a.txt"
  ```

  

------



### 绝对路径和相对路径

- 绝对路径：一个完整的路径，**以盘符开头的路径**
  - 例：`C:\\Users\itcast\\IdeaProjects\\shungyuan\\123.txt`



- 相对路径：一个简化的路径，**是相对于当前项目的根目录**
  - 例：`123.txt`
  - ./ 或者不写相当于同一级
  - ../相当于后退一级，规律可递推下去



- **注意**
  1. **路径是不区分大小写**
  2. **路径中的文件名称分隔符windows使用反斜杠,反斜杠是转义字符,两个反斜杠代表一个普通的反斜杠**
     - `C:\\Users\itcast\\IdeaProjects\\shungyuan\\123.txt`



------



### 构造方法

- `public File(String pathname) ` ：通过将给定的**路径名字符串**转换为抽象路径名来创建新的 File实例

- `public File(String parent, String child) ` ：从**父路径名字符串和子路径名字符串**创建新的 File实例

  - 父路径和子路径都可以改变，比较灵活

- `public File(File parent, String child)` ：从**父抽象路径名和子路径名字符串**创建新的 File实例

  - 父路径是一个File对象，可以对父路径调用File类方法进行一些操作，更加灵活

  ```java
  // 文件路径名
  String pathname = "D:\\aaa.txt";
  File file1 = new File(pathname); 
  
  // 文件路径名
  String pathname2 = "D:\\aaa\\bbb.txt";
  File file2 = new File(pathname2); 
  
  // 通过父路径和子路径字符串
   String parent = "d:\\aaa";
   String child = "bbb.txt";
   File file3 = new File(parent, child);
  
  // 通过父级File对象和子路径字符串
  File parentDir = new File("d:\\aaa");
  String child = "bbb.txt";
  File file4 = new File(parentDir, child);
  ```

  > 说明
  >
  > 1. 一个File对象代表硬盘中实际存在的一个文件或者目录。
  > 2. **无论该路径下是否存在文件或者目录，都不影响File对象的创建**。



------



### 常用方法

#### 获取功能

- `public String getAbsolutePath() ` ：返回此File的绝对路径名字符串

  - 无论路径是绝对的还是相对的，`getAbsolutePath`方法返回的都是绝对路径

  

- ` public String getPath() ` ：将此File转换为路径名字符串。

  - toString方法调用的就是getPath方法

  

- `public String getName()`  ：返回由此File表示的文件或目录的名称

  

- `public long length()`  ：返回由此File表示的文件的长度。 

  - **文件夹是没有大小概念的,不能获取文件夹的大小，获取到的结果是错误的**
  - **如果构造方法中给出的路径不存在，那么length方法返回0**

  ```java
  public class FileGet {
      public static void main(String[] args) {
          File f = new File("d:/aaa/bbb.java");     
          System.out.println("文件绝对路径:"+f.getAbsolutePath());
          System.out.println("文件构造路径:"+f.getPath());
          System.out.println("文件名称:"+f.getName());
          System.out.println("文件长度:"+f.length()+"字节");
  
          File f2 = new File("d:/aaa");     
          System.out.println("目录绝对路径:"+f2.getAbsolutePath());
          System.out.println("目录构造路径:"+f2.getPath());
          System.out.println("目录名称:"+f2.getName());
          System.out.println("目录长度:"+f2.length());
      }
  }
  输出结果：
  文件绝对路径:d:\aaa\bbb.java
  文件构造路径:d:\aaa\bbb.java
  文件名称:bbb.java
  文件长度:636字节
  
  目录绝对路径:d:\aaa
  目录构造路径:d:\aaa
  目录名称:aaa
  目录长度:4096，结果是错误的
  ```



-----



#### 判断功能

- `public boolean exists()` ：此File表示的文件或目录是否实际存在。
- `public boolean isDirectory()` ：此File表示的是否为目录。
- `public boolean isFile()` ：此File表示的是否为文件。
  - **后两个方法使用前提是文件必须存在，否则都放回false**

```java
public class FileIs {
    public static void main(String[] args) {
        File f = new File("d:\\aaa\\bbb.java");
        File f2 = new File("d:\\aaa");
      	// 判断是否存在
        System.out.println("d:\\aaa\\bbb.java 是否存在:"+f.exists());
        System.out.println("d:\\aaa 是否存在:"+f2.exists());
      	// 判断是文件还是目录
        System.out.println("d:\\aaa 文件?:"+f2.isFile());
        System.out.println("d:\\aaa 目录?:"+f2.isDirectory());
    }
}
输出结果：
d:\aaa\bbb.java 是否存在:true
d:\aaa 是否存在:true
d:\aaa 文件?:false
d:\aaa 目录?:true
```



-----



#### 创造删除功能

- `public boolean createNewFile()` ：当且仅当具有该名称的文件尚不存在时，创建一个新的空文件
  - 如果创建成功返回true，如果已有文件存在创建失败返回false
  - **注意**
    - **此方法只能创建文件,不能创建文件夹**
    - **创建文件的路径必须存在,否则会抛出异常**



- `public boolean delete()` ：删除由此File表示的文件或目录 

  - 如果路径是错误的，返回false
  - **如果此File表示目录，则目录必须为空才能删除**
  - **delete方法是直接在硬盘删除文件/文件夹,不走回收站,删除要谨慎**

  

- `public boolean mkdir()` ：创建由此File表示的目录

  - true:文件夹不存在,创建文件夹,返回true
  - false:文件夹存在,不会创建,返回false;构造方法中给出的路径不存在返回false
  - **此方法只能创建文件夹,不能创建文件**



- `public boolean mkdirs()` ：创建由此File表示的目录，包括任何必需但不存在的父目录

```java
public class FileCreateDelete {
    public static void main(String[] args) throws IOException {
        // 文件的创建
        File f = new File("aaa.txt");
        System.out.println("是否存在:"+f.exists()); // false
        System.out.println("是否创建:"+f.createNewFile()); // true
        System.out.println("是否存在:"+f.exists()); // true
		
     	// 目录的创建
      	File f2= new File("newDir");	
        System.out.println("是否存在:"+f2.exists());// false
        System.out.println("是否创建:"+f2.mkdir());	// true
        System.out.println("是否存在:"+f2.exists());// true

		// 创建多级目录
      	File f3= new File("newDira\\newDirb");
        System.out.println(f3.mkdir());// false
        File f4= new File("newDira\\newDirb");
        System.out.println(f4.mkdirs());// true
      
      	// 文件的删除
       	System.out.println(f.delete());// true
      
      	// 目录的删除
        System.out.println(f2.delete());// true
        System.out.println(f4.delete());// false
    }
}
```



-----



#### 遍历功能

* `public String[] list()` ：返回一个String数组，表示该File目录中的所有子文件或目录


* `public File[] listFiles()` ：返回一个File数组，表示该File目录中的所有的子文件或目录

  * **注意**
    1. **list方法和listFiles方法遍历的是构造方法中给出的目录**
    2. **如果构造方法中给出的目录的路径不存在,会抛出空指针异常**
    3. **如果构造方法中给出的路径不是一个目录,也会抛出空指针异常**

```java
public class FileFor {
    public static void main(String[] args) {
        File dir = new File("d:\\java_code");
      
      	//获取当前目录下的文件以及文件夹的名称。
		String[] names = dir.list();
		for(String name : names){
			System.out.println(name);
		}
        
        //获取当前目录下的文件以及文件夹对象，只要拿到了文件对象，那么就可以获取更多信息
        File[] files = dir.listFiles();
        for (File file : files) {
            System.out.println(file);
        }
    }
}
```



------



### File过滤器

- **概念：用于在使用遍历功能时筛选出指定的文件，是遍历方法中重载方法的参数，有两种过滤器，分别是FilenameFilter 和 FileFilter，他们都是接口，所以需要重写里面的`accept`方法（逻辑方法）**

  ![image-20200606154459992](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200606154459992.png)



- **FilenameFilter 和 FileFilter**

  - 概念：这两者都是过滤器，他们也都是接口，都需要重写`accept`方法，不过他们的`accept`方法参数不一样

    

  - FileFileter：参数是一个File对象，意思是遍历的目录（文件夹）

    ```java
    public interface FileFilter {
        boolean accept(File var1);
    }
    ```

    

  - FilenameFilter ：参数是一个File对象+String对象，File对象是遍历的目录，String对象是要找到的文件

    ```java
    public interface FilenameFilter {
        boolean accept(File var1, String var2);
    }
    ```



- 实例

  - FileFileter

  ```java
  /*
      需求:
          遍历c:\\abc文件夹,及abc文件夹的子文件夹
          只要.java结尾的文件
          c:\\abc
          c:\\abc\\abc.txt
          c:\\abc\\abc.java
          c:\\abc\\a
          c:\\abc\\a\\a.jpg
          c:\\abc\\a\\a.java
          c:\\abc\\b
          c:\\abc\\b\\b.java
          c:\\abc\\b\\b.txt
   */
  public class Demo02Filter {
      public static void main(String[] args) {
          File file = new File("c:\\abc");
          getAllFile(file);
      }
   
      public static void getAllFile(File dir){
          File[] files = dir.listFiles(new FileFilter() {
              @Override
              public boolean accept(File pathname) {
                  //过滤规则,pathname是文件夹或者是.java结尾的文件返回true
                  return pathname.isDirectory() || pathname.getName().toLowerCase().endsWith(".java");
              }
          });
   
          for (File f : files) {
              //对遍历得到的File对象f进行判断,判断是否是文件夹
              if(f.isDirectory()){
                  //f是一个文件夹,则继续遍历这个文件夹
                  //我们发现getAllFile方法就是传递文件夹,遍历文件夹的方法
                  //所以直接调用getAllFile方法即可:递归(自己调用自己)
                  getAllFile(f);
              }else{
                  //f是一个文件,直接打印即可
                  System.out.println(f);
              }
          }
      }
  }
  ```

  

  - FilenameFilter 

  ```java
  /*
      需求:
          遍历c:\\abc文件夹,及abc文件夹的子文件夹
          只要.java结尾的文件
          c:\\abc
          c:\\abc\\abc.txt
          c:\\abc\\abc.java
          c:\\abc\\a
          c:\\abc\\a\\a.jpg
          c:\\abc\\a\\a.java
          c:\\abc\\b
          c:\\abc\\b\\b.java
          c:\\abc\\b\\b.txt
   */
  public class Demo02Filter {
      public static void main(String[] args) {
          File file = new File("c:\\abc");
          getAllFile(file);
      }
   
      
      public static void getAllFile(File dir){
          File[] files = dir.listFiles(new FilenameFilter() {
              @Override
              public boolean accept(File dir, String name) {
                  //过滤规则,pathname是文件夹或者是.java结尾的文件返回true
                  return new File(dir,name).isDirectory() || name.toLowerCase().endsWith(".java");
              }
          });
   
          for (File f : files) {
              //对遍历得到的File对象f进行判断,判断是否是文件夹
              if(f.isDirectory()){
                  //f是一个文件夹,则继续遍历这个文件夹
                  //我们发现getAllFile方法就是传递文件夹,遍历文件夹的方法
                  //所以直接调用getAllFile方法即可:递归(自己调用自己)
                  getAllFile(f);
              }else{
                  //f是一个文件,直接打印即可
                  System.out.println(f);
              }
          }
      }
  }
  ```

  

- **原理：以FileFileter，FilenameFileter原理可类推**

  1. **listFiles方法遍历指定目录，获取每一个文件/文件夹封装成File对象**
  2. **调用FileFileter过滤器的accept方法，把封装好的File对象作为参数传递**
  3. **在accept方法中，如果参数File对象满足逻辑，则返回true，否则放回false**
  4. **如果返回true，则将改File对象放入listFiles方法返回的数组中，反之相反**

  ![image-20200606161255655](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200606161255655.png)













## IO流

### 理解

- **概念：I（input）表示输入（读取），O（output）表示输出（写入），流是数据**
  - 输入：把硬盘中的数据读取到内存中
  - 写入：把内存中的数据写入到硬盘中
  - 字符数据：1个字符等于2个字节
  - 字节数据：1个字节等于8个二进制比特位



- **IO分类**
  - 根据数据的流向分为：**输入流**和**输出流**。
    - **输入流** ：把数据从`其他设备`上读取到`内存`中的流。
    - **输出流** ：把数据从`内存` 中写出到`其他设备`上的流。
  - 格局数据的类型分为：**字节流**和**字符流**。
    - **字节流** ：以字节为单位，读写数据的流
    - **字符流** ：以字符为单位，读写数据的流
  - **区别**
    1. **字符流，只能操作文本文件，不能操作图片，视频等非文本文件**
    2. **字节流无法操作含有中文的文本文件，但字符流可以**



- IO的顶级父类

  - **这些顶级父类都是抽象类，所以使用的时候必须使用他们的子类**

  |            |          输入流           |           输出流           |
  | :--------: | :-----------------------: | :------------------------: |
  | **字节流** | 字节输入流**InputStream** | 字节输出流**OutputStream** |
  | **字符流** |   字符输入流**Reader**    |    字符输出流**Writer**    |



-----



### 数据的写入读取原理

- 概念：在计算机中一切都是由二进制的数据构成的，**比如一个记事本里的内容，我们之所以能看到内容而不是二进制的数，是因为在打开的时候，计算机帮我们解码了**



- 解码与编码
  - 编码:字符(能看懂的)-->字节(看不懂的)
  - 解码:字节(看不懂的)-->字符(能看懂的)



- **编码：二进制的字节数据变成十进制的数**
  - **码值0~127：按ASCII码转换**
  - **其他值：查询系统默认码表（GBK）**
    - **注意：在IDEA上，默认码表并不是GBK，而是UTF-8，所以我们的程序才老会出现乱码的错误**
    - **中文在GBK中是2个字节，在UTF-8是3个字节**





- 数据写入（内存 -> 硬盘）
  
- 步骤：java程序 --> JVM --> OS --> OS写入数据的方法 --> 数据写入到硬盘
  
- 数据读取（硬盘 -> 内存）
  - 步骤：java程序 --> JVM --> OS --> OS读取数据的方法 --> 内容读取到内存

  



-----



### 字节输出流（OutputStream）

#### 通用方法

- 概念：OutputStream是字节输出流的顶级抽象父类，里面定义了许多通用的方法供子类使用
  - **`public void close()` ：关闭此输出流并释放与此流相关联的任何系统资源。**
  - **`public void flush()` ：刷新此输出流并强制任何缓冲的输出字节被写出。**
  - **`public void write(byte[] b)`：将 b.length字节从指定的字节数组写入此输出流。**
  - **`public void write(byte[] b, int off, int len)` ：从指定的字节数组写入 len字节，从偏移量 off开始输出到此输出流。**
  - **`public abstract void write(int b)` ：将指定的字节输出流**



------



#### FileOutputStream类

- 概念：文件字节输出流，是OutputStream的子类，以此为例讲解字节输出流的用法



##### 构造方法

- **`public FileOutputStream(File file)`：创建文件输出流以写入由指定的 File对象表示的文件**

- **`public FileOutputStream(String name)`： 创建文件输出流以指定的名称写入文件**

  > 注意：当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有这个文件，会创建该文件。如果有这个文件，会清空这个文件的数据

  ```java
  public class FileOutputStreamConstructor throws IOException {
      public static void main(String[] args) {
     	 	// 使用File对象创建流对象
          File file = new File("a.txt");
          FileOutputStream fos = new FileOutputStream(file);
        
          // 使用文件名称创建流对象
          FileOutputStream fos = new FileOutputStream("b.txt");
      }
  }
  ```





##### 写入字节数据

- 方法
  - `void write(int b)` 方法，每次可以写出一个字节数据
    - **会将参数从十进制按编码转化成二进制写入**
  - `void write(byte[] b)`，每次可以写出数组中的数据
  - `void write(byte[] b, int off, int len)` ,每次写出从off索引开始，len个字节



- **实例**

  ```java
  //每次写入一个字节
  public class FOSWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileOutputStream fos = new FileOutputStream("fos.txt");     
        	// 写出数据
        	fos.write(97); // 写出第1个字节
        	fos.write(98); // 写出第2个字节
        	fos.write(99); // 写出第3个字节
        	// 关闭资源
          fos.close();
      }
  }
  输出结果：
  abc
  ```

  ```java
  //每次写入一个字节数组
  public class FOSWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileOutputStream fos = new FileOutputStream("fos.txt");     
        	// 字符串转换为字节数组
        	byte[] b = "黑马程序员".getBytes();
        	// 写出字节数组数据
        	fos.write(b);
        	// 关闭资源
          fos.close();
      }
  }
  输出结果：
  黑马程序员
  ```

  ```java
  //每次写入指定长度的字节数组
  public class FOSWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileOutputStream fos = new FileOutputStream("fos.txt");     
        	// 字符串转换为字节数组
        	byte[] b = "abcde".getBytes();
  		// 写出从索引2开始，2个字节。索引2是c，两个字节，也就是cd。
          fos.write(b,2,2);
        	// 关闭资源
          fos.close();
      }
  }
  输出结果：
  cd
  ```

  



##### 续写和换行

- 概念：续写指的是是否对指定的流对象的文件进行数据的续写，如果不续写，则会覆盖原来的文件；换行指写入过程中换行



- **续写：对于以下的构造函数，第二个参数传入true表示续写，默认为false**
  - `public FileOutputStream(File file, boolean append)`： 创建文件输出流以写入由指定的 File对象表示的文件
  - `public FileOutputStream(String name, boolean append)`： 创建文件输出流以指定的名称写入文件



- 换行

  - window系统：`\r\n`
  - Unix系统：`\n`
  - Mac系统：`\r`

  ```java
  public class FOSWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileOutputStream fos = new FileOutputStream("fos.txt");  
        	// 定义字节数组
        	byte[] words = {97,98,99,100,101};
        	// 遍历数组
          for (int i = 0; i < words.length; i++) {
            	// 写出一个字节
              fos.write(words[i]);
            	// 写出一个换行, 换行符号转成数组写出
              fos.write("\r\n".getBytes());
          }
        	// 关闭资源
          fos.close();
      }
  }
  
  输出结果：
  a
  b
  c
  d
  e
  ```

  



-----



### 字节输入流（InputStream）

#### 通用方法

- 概念：InPutStream是字节输出流的顶级抽象父类，里面定义了许多通用的方法供子类使用
  - **`public void close()` ：关闭此输入流并释放与此流相关联的任何系统资源。**
  - **`public abstract int read()`： 从输入流读取数据的下一个字节。**
  - **`public int read(byte[] b)`： 从输入流中读取一些字节数，并将它们存储到字节数组 b中**
  - **`public int read(byte[] b,int off,int lenth)`：读取指定长度的字节数组**



#### FileInputStream类

- 概念：文件字节输入流，是InputStream的子类，以此为例讲解字节输入流的用法



##### 构造方法

- `FileInputStream(File file)`： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的 File对象 file命名。
- `FileInputStream(String name)`： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名 name命名

```java
public class FileInputStreamConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileInputStream fos = new FileInputStream(file);
      
        // 使用文件名称创建流对象
        FileInputStream fos = new FileInputStream("b.txt");
    }
}
```



##### 读取字节数据

- 方法
  - **`public abstract int read()`： 从输入流读取数据的下一个字节**
    - **返回值是读取的字节数据然后转为十进制**
  - **`public int read(byte[] b)`： 从输入流中读取一些字节数，并将它们存储到字节数组 b中**
    - **返回值是读取的有效长度**



- 方法原理

  > 硬盘上的数据会指针的概念，指向被读取的字节，默认指向开头。read方法会读取指针指向的字节，并将指针向后移一位，读取到末尾返回-1





- 实例

  ```java
  //每次读取一个字节
  public class FISRead {
      public static void main(String[] args) throws IOException{
        	// 使用文件名称创建流对象
         	FileInputStream fis = new FileInputStream("read.txt");
        	// 定义变量，保存数据
          int b ；
          // 循环读取
          while ((b = fis.read())!=-1) {
              System.out.println((char)b);
          }
  		// 关闭资源
          fis.close();
      }
  }
  输出结果：
  a
  b
  c
  d
  e
  ```

  ```java
  //每次读取一个字节数组
  public class FISRead {
      public static void main(String[] args) throws IOException{
        	// 使用文件名称创建流对象.
         	FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
        	// 定义变量，作为有效个数
          int len ；
          // 定义字节数组，作为装字节数据的容器   
          byte[] b = new byte[2];
          // 循环读取
          while (( len= fis.read(b))!=-1) {
             	// 每次读取后,把数组的有效字节部分，变成字符串打印
              System.out.println(new String(b，0，len));//  len 每次读取的有效字节个数
          }
  		// 关闭资源
          fis.close();
      }
  }
  
  输出结果：
  ab
  cd
  e
  ```

  

-----



### 字节IO流练习例子

- 文件复制

  ```java
  public class Copy {
      public static void main(String[] args) throws IOException {
          // 1.创建流对象
          // 1.1 指定数据源
          FileInputStream fis = new FileInputStream("D:\\test.jpg");
          // 1.2 指定目的地
          FileOutputStream fos = new FileOutputStream("test_copy.jpg");
  
          // 2.读写数据
          // 2.1 定义数组
          byte[] b = new byte[1024];
          // 2.2 定义长度
          int len;
          // 2.3 循环读取
          while ((len = fis.read(b))!=-1) {
              // 2.4 写出数据
              fos.write(b, 0 , len);
          }
  
          // 3.关闭资源
          fos.close();
          fis.close();
      }
  }
  ```

  > **流的关闭原则：先开后关，后开先关。**



-----



### 字符输出流（Writer）

#### 通用方法

- 概念：是字符输出流的顶层父类，定义了子类中通用的方法
  - **`void write(int c)` 写入单个字符。**
  - **`void write(char[] cbuf)`写入字符数组。**
  - **`abstract void write(char[] cbuf, int off, int len)`写入字符数组的某一部分,off数组的开始索引,len写的字符个数。**
  - **`void write(String str)`写入字符串。**
  - **`void write(String str, int off, int len)` 写入字符串的某一部分,off字符串的开始索引,len写的字符个数。**
  - **`void flush()`刷新该流的缓冲。**
  - **`void close()` 关闭此流，但要先刷新它**



------



#### FileWriter类

- 概念：是Writer类的孙子类，Writer -> OutputStreamWriter -> FileWriter，以此类为例说明字符输出流，**构造时使用系统默认的字符编码和默认字节缓冲区**



##### 构造方法

- `FileWriter(File file)`： 创建一个新的 FileWriter，给定要读取的File对象。

- `FileWriter(String fileName)`： 创建一个新的 FileWriter，给定要读取的文件的名称

  ```java
  public class FileWriterConstructor {
      public static void main(String[] args) throws IOException {
     	 	// 使用File对象创建流对象
          File file = new File("a.txt");
          FileWriter fw = new FileWriter(file);
        
          // 使用文件名称创建流对象
          FileWriter fw = new FileWriter("b.txt");
      }
  }
  ```



##### 写入字符数据

- 方法
  - `void write(int c)` 写入单个字符
    - **会把参数从整形变成字符类型写入数据**
  - `void write(char[] cbuf)`写入字符数组。
  - `abstract void write(char[] cbuf, int off, int len)`写入字符数组的某一部分,off数组的开始索引,len写的字符个数。
  - `void write(String str)`写入字符串
    - **也可以直接写入字符串**
  - `void write(String str, int off, int len)` 写入字符串的某一部分,off字符串的开始索引,len写的字符个数。



- 实例

  ```java
  //写入一个字符
  public class FWWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileWriter fw = new FileWriter("fw.txt");     
        	// 写出数据
        	fw.write(97); // 写出第1个字符
        	fw.write('b'); // 写出第2个字符
        	fw.write('C'); // 写出第3个字符
        	fw.write(30000); // 写出第4个字符，中文编码表中30000对应一个汉字。
        
        	/*
          【注意】关闭资源时,与FileOutputStream不同。
        	 如果不关闭,数据只是保存到缓冲区，并未保存到文件。
          */
          // fw.close();
      }
  }
  输出结果：
  abC田
  ```

  ```java
  //写出字符数组
  public class FWWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileWriter fw = new FileWriter("fw.txt");     
        	// 字符串转换为字节数组
        	char[] chars = "黑马程序员".toCharArray();
        
        	// 写出字符数组
        	fw.write(chars); // 黑马程序员
          
  		// 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
          fw.write(b,2,2); // 程序
        
        	// 关闭资源
          fos.close();
      }
  }
  ```

  ```java
  //写出字符串
  public class FWWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileWriter fw = new FileWriter("fw.txt");     
        	// 字符串
        	String msg = "黑马程序员";
        
        	// 写出字符数组
        	fw.write(msg); //黑马程序员
        
  		// 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
          fw.write(msg,2,2);	// 程序
        	
          // 关闭资源
          fos.close();
      }
  }
  ```

  

##### 关闭刷新

- 原理

  > 由于计算机中文件都是由二进制的字节数据构成，所以我们使用字符流的时候，并非直接将字符数据写入硬盘中，而是存在一个转化的过程。这个转化的过程发生在内存的一个区域中，**也叫做字节缓冲区**。**当我们使用write方法时，只是将字符数据写入缓冲区，我们还需要使用刷新方法将数据刷新到硬盘中**



- 方法

  - `void flush()` ：刷新缓冲区，流对象可以继续使用。
  - `void close()`：先刷新缓冲区，然后通知系统释放资源。流对象不可以再被使用了

  ```java
  public class FWWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象
          FileWriter fw = new FileWriter("fw.txt");
          // 写出数据，通过flush
          fw.write('刷'); // 写出第1个字符
          fw.flush();
          fw.write('新'); // 继续写出第2个字符，写出成功
          fw.flush();
        
        	// 写出数据，通过close
          fw.write('关'); // 写出第1个字符
          fw.close();
          fw.write('闭'); // 继续写出第2个字符,【报错】java.io.IOException: Stream closed
          fw.close();
      }
  }
  ```

  

##### 续写换行

- 与OutputStream一样

  ```java
  public class FWWrite {
      public static void main(String[] args) throws IOException {
          // 使用文件名称创建流对象，可以续写数据
          FileWriter fw = new FileWriter("fw.txt"，true);     
        	// 写出字符串
          fw.write("黑马");
        	// 写出换行
        	fw.write("\r\n");
        	// 写出字符串
    		fw.write("程序员");
        	// 关闭资源
          fw.close();
      }
  }
  输出结果:
  黑马
  程序员
  ```

  



-----



### 字符输入流（Reader）

#### 通用方法

- 概念：是字符输入流的顶层父类，定义了子类中通用的方法

  - `public void close()` ：关闭此流并释放与此流相关联的任何系统资源。
  - `public int read()`： 从输入流读取一个字符。
  - `public int read(char[] cbuf)`： 从输入流中读取一些字符，并将它们存储到字符数组 cbuf中 

  



#### FileReader类

- 概念：概念：是Reader类的孙子类，Reader -> OutputStreamReader -> FileReader，以此类为例说明字符输入流，**构造时使用系统默认的字符编码和默认字节缓冲区**



##### 构造方法

- `FileReader(File file)`： 创建一个新的 FileReader ，给定要读取的File对象。

- `FileReader(String fileName)`： 创建一个新的 FileReader ，给定要读取的文件的名称

  ```java
  public class FileReaderConstructor throws IOException{
      public static void main(String[] args) {
     	 	// 使用File对象创建流对象
          File file = new File("a.txt");
          FileReader fr = new FileReader(file);
        
          // 使用文件名称创建流对象
          FileReader fr = new FileReader("b.txt");
      }
  }
  ```

  

##### 读取字符数据

- 方法
  - **`public int read()`： 从输入流读取一个字符**
    - **返回值是读取到的字符转化为整形**
  - **`public int read(char[] cbuf)`： 从输入流中读取一些字符，并将它们存储到字符数组 cbuf中** 
    - **返回值是读取的有效长度，末尾是-1**



- 实例

  ```java
  //每次读取一个字符
  public class FRRead {
      public static void main(String[] args) throws IOException {
        	// 使用文件名称创建流对象
         	FileReader fr = new FileReader("read.txt");
        	// 定义变量，保存数据
          int b ；
          // 循环读取
          while ((b = fr.read())!=-1) {
              System.out.println((char)b);
          }
  		// 关闭资源
          fr.close();
      }
  }
  输出结果：
  黑
  马
  程
  序
  员
  ```

  ```java
  //每次读取一个字符数组
  public class FISRead {
      public static void main(String[] args) throws IOException {
        	// 使用文件名称创建流对象
         	FileReader fr = new FileReader("read.txt");
        	// 定义变量，保存有效字符个数
          int len ；
          // 定义字符数组，作为装字符数据的容器
          char[] cbuf = new char[2];
          // 循环读取
          while ((len = fr.read(cbuf))!=-1) {
              System.out.println(new String(cbuf,0,len));
          }
      	// 关闭资源
          fr.close();
      }
  }
  
  输出结果：
  黑马
  程序
  员
  ```

  

-----







### Properties类

- **概念：一个集合，继承于`Hashtable`，是唯一一个与IO流有关的集合，它使用键值结构存储数据，每个键及其对应值都是一个字符串。由于与IO流相关，所以可以通过IO流将集合的内容写入硬盘，或者从Properties文件中读取数据**



- 普通方法

  - **`public Properties()` :创建一个空的属性列表**

  - **`public Object setProperty(String key, String value)` ： 保存一对属性**
  - **`public String getProperty(String key)` ：使用此属性列表中指定的键搜索属性值**
  - **`public Set<String> stringPropertyNames()` ：所有键的名称的集合**

  > **注意：由于这个集合与IO流有关，所以里面键值的类型都是String类，当然也有普通的put方法可以存放其他类型的值，但关于IO流的时候的应该都是String类型**

  ```java
  public class ProDemo {
      public static void main(String[] args) throws FileNotFoundException {
          // 创建属性集对象
          Properties properties = new Properties();
          // 添加键值对元素
          properties.setProperty("filename", "a.txt");
          properties.setProperty("length", "209385038");
          properties.setProperty("location", "D:\\a.txt");
          // 打印属性集对象
          System.out.println(properties);
          // 通过键,获取属性值
          System.out.println(properties.getProperty("filename"));
          System.out.println(properties.getProperty("length"));
          System.out.println(properties.getProperty("location"));
  
          // 遍历属性集,获取所有键的集合
          Set<String> strings = properties.stringPropertyNames();
          // 打印键值对
          for (String key : strings ) {
            	System.out.println(key+" -- "+properties.getProperty(key));
          }
      }
  }
  输出结果：
  {filename=a.txt, length=209385038, location=D:\a.txt}
  a.txt
  209385038
  D:\a.txt
  filename -- a.txt
  length -- 209385038
  location -- D:\a.txt
  ```

  



- **读取的方法**

  - `void load(InputStream inStream)`

  - ``void load(Reader reader)`

    - 作用：这两个方法都把硬盘中保存的文件(键值对),读取到集合中使用
    - **区别：两个方法参数不同，一个是字符流，一个是字节流，前者能处理含中文的文本文件，后者不能处理含中文的文件**

    > 注意:
    >       1.存储键值对的文件中,键与值默认的连接符号可以使用=,空格(其他符号)
    >       2.存储键值对的文件中,可以使用#进行注释,被注释的键值对不会再被读取
    >       3.存储键值对的文件中,键与值默认都是字符串,不用再加引号

  ```java
  public class ProDemo2 {
      public static void main(String[] args) throws FileNotFoundException {
          // 创建属性集对象
          Properties pro = new Properties();
          // 加载文本中信息到属性集
          pro.load(new FileInputStream("read.txt"));
          // 遍历集合并打印
          Set<String> strings = pro.stringPropertyNames();
          for (String key : strings ) {
            	System.out.println(key+" -- "+pro.getProperty(key));
          }
       }
  }
  输出结果：
  filename -- a.txt
  length -- 209385038
  location -- D:\a.txt
  ```

  



- **写入的方法**

  - `void store(OutputStream out, String comments)`

  - `void store(Writer writer, String comments)`
    - 作用：可以使用Properties集合中的方法store,把集合中的临时数据,持久化写入到硬盘中存储
    - 区别：一个可以写入中文，一个不可以写入中文

    > 参数comment：用来解释说明保存的文件是做什么用的
    >                              不能使用中文,会产生乱码,默认是Unicode编码
    >                               一般使用""空字符串

  ```java
  private static void show02() throws IOException {
          //1.创建Properties集合对象,添加数据
          Properties prop = new Properties();
          prop.setProperty("赵丽颖","168");
          prop.setProperty("迪丽热巴","165");
          prop.setProperty("古力娜扎","160");
   
          //2.创建字节输出流/字符输出流对象,构造方法中绑定要输出的目的地
          FileWriter fw = new FileWriter("09_IOAndProperties\\prop.txt");
   
          //3.使用Properties集合中的方法store,把集合中的临时数据,持久化写入到硬盘中存储
          prop.store(fw,"save data");
   
          //4.释放资源
          fw.close();
       }
  ```





------



### 缓冲流

- 概念：缓冲流也叫高效流，是对以上四种流的增强
  - **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream`
  - **字符缓冲流**：`BufferedReader`，`BufferedWriter`



#### 原理

- 缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率

  ![image-20200610152842681](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200610152842681.png)

> 普通的流：每次是一个字节或一个字符的读取写入
>
> 缓冲流：建一个缓冲区，存储读取写入的数据，等到达一定程度，一次返回多个字节或字符数据



#### 字节缓冲流

- 构造方法

  - `public BufferedInputStream(InputStream in)` ：创建一个 新的缓冲输入流
  - `public BufferedOutputStream(OutputStream out)`： 创建一个新的缓冲输出流

  ```java
  // 创建字节缓冲输入流
  BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
  // 创建字节缓冲输出流
  BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
  ```



- 使用实例：用法与字节流差不多

  ```java
  public class BufferedDemo {
      public static void main(String[] args) throws FileNotFoundException {
        	// 记录开始时间
          long start = System.currentTimeMillis();
  		// 创建流对象
          try (
  			BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
  		 BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
          ){
            	// 读写数据
              int len;
              byte[] bytes = new byte[8*1024];
              while ((len = bis.read(bytes)) != -1) {
                  bos.write(bytes, 0 , len);
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
  		// 记录结束时间
          long end = System.currentTimeMillis();
          System.out.println("缓冲流使用数组复制时间:"+(end - start)+" 毫秒");
          bos.close();
          bis.close();
      }
  }
  缓冲流使用数组复制时间:666 毫秒
  ```

  > 注意：释放资源时，只要释放缓冲流即可，对于参数里的流无需手动释放，系统会自动帮我们释放





#### 字符缓冲流

- 构造方法

  - `public BufferedReader(Reader in)` ：创建一个 新的缓冲输入流
  - `public BufferedWriter(Writer out)`： 创建一个新的缓冲输出流

  ```java
  // 创建字符缓冲输入流
  BufferedReader br = new BufferedReader(new FileReader("br.txt"));
  // 创建字符缓冲输出流
  BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
  ```



- **特有方法**

  - **BufferedReader：`public String readLine()`: 读一行文字**
    - **末尾返回null**
  - **BufferedWriter：`public void newLine()`: 写一行行分隔符,由系统属性定义符号**

  ```java
  //readLine演示
  public class BufferedReaderDemo {
      public static void main(String[] args) throws IOException {
        	 // 创建流对象
          BufferedReader br = new BufferedReader(new FileReader("in.txt"));
  		// 定义字符串,保存读取的一行文字
          String line  = null;
        	// 循环读取,读取到最后返回null
          while ((line = br.readLine())!=null) {
              System.out.print(line);
              System.out.println("------");
          }
  		// 释放资源
          br.close();
      }
  }
  ```

  ```java
  //newLine演示
  public class BufferedWriterDemo throws IOException {
    public static void main(String[] args) throws IOException  {
      	// 创建流对象
    	BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));
      	// 写出数据
        bw.write("黑马");
      	// 写出换行
        bw.newLine();
        bw.write("程序");
        bw.newLine();
        bw.write("员");
        bw.newLine();
    	// 释放资源
        bw.close();
    }
  }
  输出效果:
  黑马
  程序
  员
  ```

  

-----



#### 缓冲流练习

- 对下面文本进行排序

```
3.侍中、侍郎郭攸之、费祎、董允等，此皆良实，志虑忠纯，是以先帝简拔以遗陛下。愚以为宫中之事，事无大小，悉以咨之，然后施行，必得裨补阙漏，有所广益。
8.愿陛下托臣以讨贼兴复之效，不效，则治臣之罪，以告先帝之灵。若无兴德之言，则责攸之、祎、允等之慢，以彰其咎；陛下亦宜自谋，以咨诹善道，察纳雅言，深追先帝遗诏，臣不胜受恩感激。
4.将军向宠，性行淑均，晓畅军事，试用之于昔日，先帝称之曰能，是以众议举宠为督。愚以为营中之事，悉以咨之，必能使行阵和睦，优劣得所。
2.宫中府中，俱为一体，陟罚臧否，不宜异同。若有作奸犯科及为忠善者，宜付有司论其刑赏，以昭陛下平明之理，不宜偏私，使内外异法也。
1.先帝创业未半而中道崩殂，今天下三分，益州疲弊，此诚危急存亡之秋也。然侍卫之臣不懈于内，忠志之士忘身于外者，盖追先帝之殊遇，欲报之于陛下也。诚宜开张圣听，以光先帝遗德，恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也。
9.今当远离，临表涕零，不知所言。
6.臣本布衣，躬耕于南阳，苟全性命于乱世，不求闻达于诸侯。先帝不以臣卑鄙，猥自枉屈，三顾臣于草庐之中，咨臣以当世之事，由是感激，遂许先帝以驱驰。后值倾覆，受任于败军之际，奉命于危难之间，尔来二十有一年矣。
7.先帝知臣谨慎，故临崩寄臣以大事也。受命以来，夙夜忧叹，恐付托不效，以伤先帝之明，故五月渡泸，深入不毛。今南方已定，兵甲已足，当奖率三军，北定中原，庶竭驽钝，攘除奸凶，兴复汉室，还于旧都。此臣所以报先帝而忠陛下之职分也。至于斟酌损益，进尽忠言，则攸之、祎、允之任也。
5.亲贤臣，远小人，此先汉所以兴隆也；亲小人，远贤臣，此后汉所以倾颓也。先帝在时，每与臣论此事，未尝不叹息痛恨于桓、灵也。侍中、尚书、长史、参军，此悉贞良死节之臣，愿陛下亲之信之，则汉室之隆，可计日而待也。
```

```java
public class BufferedTest {
    public static void main(String[] args) throws IOException {
        // 创建map集合,保存文本数据,键为序号,值为文字
        HashMap<String, String> lineMap = new HashMap<>();

        // 创建流对象
        BufferedReader br = new BufferedReader(new FileReader("in.txt"));
        BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));

        // 读取数据
        String line  = null;
        while ((line = br.readLine())!=null) {
            // 解析文本
            String[] split = line.split("\\.");
            // 保存到集合
            lineMap.put(split[0],split[1]);
        }
        // 释放资源
        br.close();

        // 遍历map集合
        for (int i = 1; i <= lineMap.size(); i++) {
            String key = String.valueOf(i);
            // 获取map中文本
            String value = lineMap.get(key);
          	// 写出拼接文本
            bw.write(key+"."+value);
          	// 写出换行
            bw.newLine();
        }
		// 释放资源
        bw.close();
    }
}
```



------



### 转换流

- 概念：一种用于解决编码问题的流，是字节到字符转换的桥梁



#### 流中的编码问题

- 概念：在IDEA中，使用`FileReader` 读取项目中的文本文件。由于IDEA的设置，都是默认的`UTF-8`编码，所以没有任何问题。但是，当读取Windows系统中创建的文本文件时，由于Windows系统的默认是GBK编码，就会出现乱码

  ```java
  public class ReaderDemo {
      public static void main(String[] args) throws IOException {
          FileReader fileReader = new FileReader("E:\\File_GBK.txt");
          int read;
          while ((read = fileReader.read()) != -1) {
              System.out.print((char)read);
          }
          fileReader.close();
      }
  }
  输出结果：
  ���
  ```





#### 原理

- 转换流在进行字节到字符的转换时，查询指定的码表用来解码或编码

  ![image-20200615095938216](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200615095938216.png)





#### InputStreamReader类

- **概念：是Reader的子类，是从字节流到字符流的桥梁。它读取字节，并使用指定的字符集将其解码为字符。它的字符集可以由名称指定，也可以接受平台的默认字符集**



##### 构造方法

- `InputStreamReader(InputStream in)`: 创建一个使用默认字符集的字符流。
- `InputStreamReader(InputStream in, String charsetName)`: 创建一个指定字符集的字符流。

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```



##### 指定编码读取

```java
public class ReaderDemo2 {
    public static void main(String[] args) throws IOException {
      	// 定义文件路径,文件为gbk编码
        String FileName = "E:\\file_gbk.txt";
      	// 创建流对象,默认UTF8编码
        InputStreamReader isr = new InputStreamReader(new FileInputStream(FileName));
      	// 创建流对象,指定GBK编码
        InputStreamReader isr2 = new InputStreamReader(new FileInputStream(FileName) , "GBK");
		// 定义变量,保存字符
        int read;
      	// 使用默认编码字符流读取,乱码
        while ((read = isr.read()) != -1) {
            System.out.print((char)read); // ��Һ�
        }
        isr.close();
      
      	// 使用指定编码字符流读取,正常解析
        while ((read = isr2.read()) != -1) {
            System.out.print((char)read);// 大家好
        }
        isr2.close();
    }
}
```



#### OutputStreamWriter类

概念：是Writer的子类，是从字符流到字节流的桥梁。使用指定的字符集将字符编码为字节。它的字符集可以由名称指定，也可以接受平台的默认字符集。



##### 构造方法

- `OutputStreamWriter(OutputStream in)`: 创建一个使用默认字符集的字符流。
- `OutputStreamWriter(OutputStream in, String charsetName)`: 创建一个指定字符集的字符流。

```java
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("out.txt"));
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```



##### 指定编码写出

```java
public class OutputDemo {
    public static void main(String[] args) throws IOException {
      	// 定义文件路径
        String FileName = "E:\\out.txt";
      	// 创建流对象,默认UTF8编码
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(FileName));
        // 写出数据
      	osw.write("你好"); // 保存为6个字节
        osw.close();
      	
		// 定义文件路径
		String FileName2 = "E:\\out2.txt";
     	// 创建流对象,指定GBK编码
        OutputStreamWriter osw2 = new OutputStreamWriter(new FileOutputStream(FileName2),"GBK");
        // 写出数据
      	osw2.write("你好");// 保存为4个字节
        osw2.close();
    }
}
```





#### 转换流练习

- 案例：将GBK编码的文本文件，转换为UTF-8编码的文本文件

  ```java
  public class TransDemo {
     public static void main(String[] args) {      
      	// 1.定义文件路径
       	String srcFile = "file_gbk.txt";
          String destFile = "file_utf8.txt";
  		// 2.创建流对象
      	// 2.1 转换输入流,指定GBK编码
          InputStreamReader isr = new InputStreamReader(new FileInputStream(srcFile) , "GBK");
      	// 2.2 转换输出流,默认utf8编码
          OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(destFile));
  		// 3.读写数据
      	// 3.1 定义数组
          char[] cbuf = new char[1024];
      	// 3.2 定义长度
          int len;
      	// 3.3 循环读取
          while ((len = isr.read(cbuf))!=-1) {
              // 循环写出
            	osw.write(cbuf,0,len);
          }
      	// 4.释放资源
          osw.close();
          isr.close();
    	}
  }
  ```



------



### 序列化流

- 概念：Java 提供了一种对象**序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该`对象的数据`、`对象的类型`和`对象中存储的属性`等信息。字节序列写出到文件之后，相当于文件中**持久保存**了一个对象的信息
  - 序列化：将一个对象存储到硬盘中
  - 反序列化：将硬盘中存储的对象加载到内存中



#### ObjectOutputStream类

概念：`java.io.ObjectOutputStream` 类，将Java对象的原始数据类型写出到文件,实现对象的持久存储。



##### 构造方法

- `public ObjectOutputStream(OutputStream out)`： 创建一个指定ObjectOutputStream的对象

```java
FileOutputStream fileOut = new FileOutputStream("employee.txt");
ObjectOutputStream out = new ObjectOutputStream(fileOut);
```



##### 序列化操作

1. 一个对象要想序列化，必须满足两个条件:

- **该类必须实现`java.io.Serializable` 接口，`Serializable` 是一个标记接口，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出`NotSerializableException` 。**
- **该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，则该属性必须注明是瞬态的，使用`transient` 关键字修饰**
  - 注意：如果是用static修饰的属性，也不会被序列化

```java
public class Employee implements java.io.Serializable {
    public String name;
    public String address;
    public transient int age; // transient瞬态修饰成员,不会被序列化
    public void addressCheck() {
      	System.out.println("Address  check : " + name + " -- " + address);
    }
}
```

2.写出对象方法

- `public final void writeObject (Object obj)` : 将指定的对象写出。

```java
public class SerializeDemo{
   	public static void main(String [] args)   {
    	Employee e = new Employee();
    	e.name = "zhangsan";
    	e.address = "beiqinglu";
    	e.age = 20; 
    	try {
      		// 创建序列化流对象
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.txt"));
        	// 写出对象
        	out.writeObject(e);
        	// 释放资源
        	out.close();
        	fileOut.close();
        	System.out.println("Serialized data is saved"); // 姓名，地址被序列化，年龄没有被序列化。
        } catch(IOException i)   {
            i.printStackTrace();
        }
   	}
}
输出结果：
Serialized data is saved
```





####  ObjectInputStream类

概念：ObjectInputStream反序列化流，将之前使用ObjectOutputStream序列化的原始数据恢复为对象



##### 构造方法

- `public ObjectInputStream(InputStream in)`： 创建一个指定ObjectInputStream的对象

  

##### 反序列化操作1

如果能找到一个对象的class文件，我们可以进行反序列化操作，调用`ObjectInputStream`读取对象的方法：

- `public final Object readObject ()` : 读取一个对象。

```java
public class DeserializeDemo {
   public static void main(String [] args)   {
        Employee e = null;
        try {		
             // 创建反序列化流
             FileInputStream fileIn = new FileInputStream("employee.txt");
             ObjectInputStream in = new ObjectInputStream(fileIn);
             // 读取一个对象
             e = (Employee) in.readObject();
             // 释放资源
             in.close();
             fileIn.close();
        }catch(IOException i) {
             // 捕获其他异常
             i.printStackTrace();
             return;
        }catch(ClassNotFoundException c)  {
        	// 捕获类找不到异常
             System.out.println("Employee class not found");
             c.printStackTrace();
             return;
        }
        // 无异常,直接打印输出
        System.out.println("Name: " + e.name);	// zhangsan
        System.out.println("Address: " + e.address); // beiqinglu
        System.out.println("age: " + e.age); // 0
    }
}
```

> **对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个 `ClassNotFoundException` 异常。**



##### **反序列化操作2**

**另外，当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个`InvalidClassException`异常。**发生这个异常的原因如下：

- 该类的序列版本号与从流中读取的类描述符的版本号不匹配
- 该类包含未知数据类型
- 该类没有可访问的无参数构造方法

**解决**：**`Serializable` 接口给需要序列化的类，提供了一个序列版本号。`serialVersionUID` 该版本号的目的在于验证序列化的对象和对应类是否版本匹配。**

```java
public class Employee implements java.io.Serializable {
     // 加入序列版本号
     private static final long serialVersionUID = 1L;
     public String name;
     public String address;
     // 添加新的属性 ,重新编译, 可以反序列化,该属性赋为默认值.
     public int eid; 

     public void addressCheck() {
         System.out.println("Address  check : " + name + " -- " + address);
     }
}
```





#### 序列化流练习

- 案例
  1. 把若干学生对象 ，保存到集合中。
  2. 把集合序列化。
  3. 反序列化读取时，只需要读取一次，转换为集合类型。
  4. 遍历集合，可以打印所有的学生信息

```java
public class SerTest {
	public static void main(String[] args) throws Exception {
		// 创建 学生对象
		Student student = new Student("老王", "laow");
		Student student2 = new Student("老张", "laoz");
		Student student3 = new Student("老李", "laol");

		ArrayList<Student> arrayList = new ArrayList<>();
		arrayList.add(student);
		arrayList.add(student2);
		arrayList.add(student3);
		// 序列化操作
		// serializ(arrayList);
		
		// 反序列化  
		ObjectInputStream ois  = new ObjectInputStream(new FileInputStream("list.txt"));
		// 读取对象,强转为ArrayList类型
		ArrayList<Student> list  = (ArrayList<Student>)ois.readObject();
		
      	for (int i = 0; i < list.size(); i++ ){
          	Student s = list.get(i);
        	System.out.println(s.getName()+"--"+ s.getPwd());
      	}
	}

	private static void serializ(ArrayList<Student> arrayList) throws Exception {
		// 创建 序列化流 
		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("list.txt"));
		// 写出对象
		oos.writeObject(arrayList);
		// 释放资源
		oos.close();
	}
}
```