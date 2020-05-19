## IO流

- 概念：IO流分输入流（input）和输出流（output），**输入流是从硬盘中写到内存中，输出流是从内存中写到硬盘上**



- **流数据：分为字符流和字节流**
  - **1个字符 = 2个字节 = 16个二进制位**



------

### 文件存储原理

- 概念：所有文件都是二进制的字节数据



- **编码**
  - **二进制的字节数据变成十进制的数**
    - **0~127：按ASCII码转换**
    - **其他值：查询系统默认码表（GBK）**
    - **其他值：查询默认码表（UTF-8）（在IDEA上）**



- **写入：硬盘输入内存**
  1. **read方法将文件中的一个二进制字节数写入内存**
  2. **二进制字节数转换成十进制的整数**
  3. **返回该整数**



- **读出：内存输出硬盘**
  1. **write方法写入一个十进制整数**
  2. **系统转换为二进制，存到文件中**
  3. **打开文件时，转为十进制整数，然后解码**（文件中的数据一直是二进制形式，只是当我们打开看到时才开始解码成十进制然后转码）



------



### 字节输出流（OutputStream）

- 概念：是字节输出流的顶级父类，抽象类



- 用其子类FileOutputStream（文件字节输出流来演示）





- **构造方法**

  - **FileOutputStream(String name) ：创建一个向具有指定名称的文件中写入数据的文件输出流**

  - **FileOutputStream(File file)：创建一个指向File对象表示的文件中写入数据的文件输出流**

  - **参数：写入数据的目的**

    - **String name：目的地是一个文件的路径**
    - **File file：目的地是一个文件**

    

  - 作用

    1. 创建一个流对象
    2. 会根据参数指定的路径，创造一个新的空文件夹
    3. 会把创建好的流对象指向文件夹





- 原理
  - java程序 --> JVM --> 操作系统 --> 操作写数据的方法 --> 写到文件夹中

![1587307996(1)](E:\md资料图片\1587307996(1).png)





- **步骤**
  1. **创建一个FileOutputStream对象，构造方法中传递写入数据的目的地**
  2. **调用FileOutputStream对象的方法write，把数据写入到文件中**
  3. **释放流对象**

```java
/*
相关方法
1.close()：释放资源
2.flush()；刷新此输出流并强制任何缓冲的输出字节被写出
3.write(参数)	
	1.write(int b)：将输入的参数写入此输出流
	2.write(byte[] b)：将这个字节数组写入此输出流
	3.write(byte[] b,int off,int len)：将这个字节数组写入此输出流，从索引off开始，长度为len
*/	

//一次写一个字节
FileOutputStream fos = new FileOutputStream("AA.txt");
fos.write(97);	//会转成二进制存入流中,然后记事本打开时再转换为十进制，然后按ASCII码表转换成a
fos.close();

//一次写多个字节
FileOutputStream fos = new FileOutputStream("AA.txt");
byte[] byte1 = {97,98,99,100};
byte[] byte2 = {-97,-98,-99,100,101};//如果是负数，则会与下一个字节组成一个中文字符，按系统默认码表
fos.write(byte1);//结果是abcd
for.write(byte2);//结果是??e 
fos.close();

//写字符串
FileOutputStream fos = new FileOutputStream("AA.txt");
for.write("你好".getBytes());
fos.close();
```





- 续写和换行
  - 续写：在构造参数时，后面写上true就可以完成续写了
  - 换行，在输出流中加入**\r\n**

```java
FileOutputStream fos = new FileOutputStream("AA.txt",true);	//续写
for.write("你好".getBytes());
for.write("\r\n".getBytes());	//换行
fos.close();
```



------



### 字节输入流（InputStream）

- 概念：是字节输入流的顶级父类，抽象类



- 用其子类FileInputStream（文件字节输入流来演示）





- **构造方法**

  - **FileInputStream(String name) ：创建一个向具有指定名称的文件中写入数据的文件输入流**

  - **FileInputStream(File file)：创建一个指向File对象表示的文件中写入数据的文件输入流**

  - **参数：写入数据的目的**

    - **String name：目的地是一个文件的路径**
    - **File file：目的地是一个文件**

    

  - 作用

    1. 创建一个流对象
    2. 会把创建好的流对象指向指定路径





- 原理
  - java程序 --> JVM --> 操作系统 --> 操作读数据的方法 --> 读取文件数据到内存中

![1587307854(1)](E:\md资料图片\1587307854(1).png)





- **步骤**
  1. **创建输入流对象，绑定读取的数据源**
  2. **使用read方法，读取文件**
  3. **释放资源**

```java
/*
    read方法：读取文件中一个字节并返回字节的值，然后把指针向后移一位，读取到文件末尾返回-1
    	返回值：读取到的字节
    	read():返回当前指针指向的字节，指针后移一位
    	read(byte[] b)：读取b.lenth个字节数存入到b数组中，返回值时有效读取的字节数，指针后移lenth位
    	read(byte[] b,int off,int lenth):off数组开始索引，lenth转换字节个数，一般为1024或者整数倍
*/

//读取一个字节
FileInputStream fis = new FileInputStream("AA.txt");
int len = fis.read();
sout(len);		 //假如文件中内容第一个字母是a，这里打印97
sout((char)len); //打印a
fis.close();

//读取多个字节
FileInputStream fis = new FileInputStream("AA.txt");
int len = 0;
while(len = fis.read() != -1){
    sout(len);			//假如文件中内容是abc，这里打印979899
	sout((char)len);	//假如文件中内容是abc，这里打印abc
}
fis.close();


//读取多个字节（数组形式）
//这种方式可能造成内存浪费
FileInputStream fis = new FileInputStream("AA.txt");
byte[] bytes = new byte[1024];
while(len = fis.read(bytes) != -1){//数组每次都会被重置
    sout(len);			//假如文件内容时abc，这里打印3
    sout(bytes);	    //假如文件中内容是abc，这里打印979899
    sout((char)bytes);  //假如文件中内容是abc，这里打印abc
}
fis.close();
//最佳方式
FileInputStream fis = new FileInputStream("AA.txt");
byte[] bytes = new byte[1024];
while(len = fis.read(bytes,0,len) != -1){	//这里做了改动
    sout(len);			
    sout(bytes);	   
    sout((char)bytes);  
}
fis.close();
```



- **注意：如果含有写入和读出的操作时，释放资源先释放写入，后释放读出**



------



### 