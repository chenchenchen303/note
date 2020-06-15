## JUC包

### 结构图

![img](https://img-blog.csdnimg.cn/20190321160902460.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FWX3dvYWlqYXZh,size_16,color_FFFFFF,t_70)



------



### 知识补充

#### 可见性

- 概念：内存可见性其实就是共享变量在线程间的可见性，**一个线程对共享变量值的修改，能够及时的被其他线程看到**



- 例子

  ```java
  public class Test {
      public static void main(String[] args) {
          ThreadDemo threadDemo = new ThreadDemo();
          new Thread(threadDemo).start();
          
          while (true){
              if(threadDemo.isFlag()){
                  System.out.println("这里输出了。");
                  break;
              }
          }
      }
  }
  
  class ThreadDemo implements Runnable{
      private boolean flag = false;
  
      @Override
      public void run() {
          flag = true;
          System.out.println("flag="+isFlag());
      }
  
      public boolean isFlag() {
          return flag;
      }
  }
  
  预期结果：flag=true
      	这里输出了
  
      结果：flag=true
      
      分析：主线程中的打印没成功且程序没停止，说明它读取到的flag仍然是false，这就叫内存不可见
      原因：线程修改flag的值后，主线程还没来得及重新读取flag就已经进入了循环
  ```



- **原理：将工作内存1中更新过的共享变量刷新到主存中，再将主存中最新的共享变量刷新到工作内存2中**

  - 注意：当前线程只能操作自己的工作内存空间，无法操作其他线程的工作内存，都得通过主存来联系

  ![image-20200521002054002](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200521002054002.png)






- **解决方法**

  - **使用volatile关键字**

  ```java
  上面出错例子解决方法：
      ......
      private volatile boolean flag = false;  //这里采用volatile关键字即可
      ......
  ```
  

  - **使用synchronized关键字**
    1. 线程解锁前，必须把共享变量的最新值刷新到主内存中 
    2. 线程加锁时，将清空工作内存中共享变量的值，故使用共享变量时需要从主内存中重新读取最新的值 

  ```java
  上面出错例子解决方法：
  	......
     @Override
      public synchronized void run() {
          flag = true;
          System.out.println("flag="+isFlag());
      }
  	......
  ```

  

  - **两者差别**

    1. volatile不具备互斥性，synchronize具备互斥性
    2. volatile只能保证可见性，不能保证原子性；synchronized既能保证可见性，又能保证原子性
    3. volatile效率比synchronize高，且花销比synchronize小

    - **尽量使用volatile**

  

------



#### 原子性

- 概念：**概念上有点类似与mysql中的事件，一个或多个操作，要么全部执行且在执行过程中不被任何因素打断，要么全部不执行。**对基本数据类型的变量的读取和赋值操作是原子性操作。



- 例子

  ```java
  public class TestAtomicDemo {
  public static void ain(String[] args) { 
      AtomicDemo ad = new AtomicDemo();
  	for(int i = 0; i < 10; i++) { 
      	new Thread(ad).start();
  		}
  	}
  }
  
  class AtomicDemo implements Runnable{
  	private volatile int serialNumber = 0; //线程共享变量
  	@Override
  	public void run(){ 
          try {
  			Thread.sleep(200);
  			} catch (InterruptedException e) {
  			}
  			System.out.println(getSerialNumber());
  			}
  	public int getSerialNumber(){ 
          return serialNumber++;
  			}
  }
  
  预期结果：10个不同的值
     结果：出现了重复值
     分析：不满足原子性操作
  ```




- **出错分析**
  1. 自增操作是不具备原子性的，它包括读取变量的原始值、进行加1操作、写入工作内存。**这三个子操作可能会分割开执行**
  2. 假如某个时刻变量的值为10，线程1对变量进行自增操作，线程1先读取了变量的原始值，然后线程1被阻塞了
  3. 然后线程2对变量进行自增操作，线程2也去读取变量的原始值，线程2会直接去主存读取的值(volatile就是要直接从内存读取)，发现的值时10，然后进行加1操作，并把11写入工作内存，最后写入主存
  4. 回到线程1接着进行加1操作，由于已经读取了的值，注意此时在线程1的工作内存中的值仍然为10所以线程1对进行加1操作后inc的值为11，然后将11写入工作内存，最后写入主存。



- **解决方法**
1. 使用synchronized关键字或者lock进行同步操作（互斥操作）
  
2. **使用JUC包中的atomic包**
  - **atomic原理是使用了volatile和cas算法，实现了内存的可见性和原子性**
   
  - **这种方法更好，性能高，是一种无锁的同步方法**

 

------

#### CAS算法

- 概念：比较再交换（Compare and Swap），是一种用于实现原子性操作的算法



- **特点**
  1. **失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次发起尝试**
  2. **使用了硬件命令（CPU命令）保证了原子性，效率很高，比使用锁的同步操作效率高**



- **原理**
  1. **CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B**
  2. **当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则返回V**
  3. **以上步骤是不可再分的，具有原子性**

![image-20200522001225839](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200522001225839.png)



1. t1，t2线程是同时更新同一变量56的值，所以他们会把主内存的值完全拷贝一份到自己的工作内存空间，所以t1和t2线程的预期值都为56
2. 假设t1在与t2线程竞争中，线程t1竞争到了，能去更新变量的值，t2线程没争到
3. t1线程去更新变量值改为57，然后写到内存中。
4. 此时对于t2来讲它会检测到，内存值变为了57，与预期值56不一致，就不能把它修改的值写回主内存就会判定就操作失败了（想改的值不再是原来的值，因为很明显，有其它操作先改变了这个值）



- 模仿cas算法的代码

  ```java
  class CAS{
      //共享数据
      private volatile int value;
      
      //获取内存值
      public synchronized int get(){
          return value;
      }
      
      //比较
      public synchronized int compareAndSwap(int expectedValue,int newValue){
          int oldValue = value;
          if(oldValue == expectedValue){
              this.value = newValue;
          }
          return oldValue;
      }
      
      //设置
      public synchronized boolean compareAndSet(int expectedValue,int newValue){
          return expectedValue == compareAndSwap(expectedValue, newValue);
      }
  }
  ```

  

- 应用场景
  1. 乐观锁
  2. 并发容器
  3. 原子类



- **缺点**
  1. **ABA问题**
     - 比如线程1获取数据时是5，这时线程2获取数据把它改为6，线程3获取数据又把它改为5，这是线程1就会以为并没有改变
  2. **自旋时间过长**
     - 由于cas算法中失败的话并不会被挂起，如果这个线程一直连接不上，线程会一直在不断尝试，这个过程会浪费大量的CPU资源



- cas底层原理（了解）

  - **使用了Unsafe操作CPU实现原子操作**

  ![image-20200522231427413](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200522231427413.png)

![image-20200522231509733](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200522231509733.png)

------

### atomic类

- **核心：使用了cas算法**

#### 思维导图

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/2019080418423955.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpeGFicw==,size_16,color_FFFFFF,t_70)

------

#### **原子更新基本类型**

- **概念：用于修饰基本类型，作用类似于包装类，含有特有方法，这些方法含有原子性**



- 类型

  - AtomicBoolean
  - AtomicLong
  - AtomicInteger

  

- 方法
  - `int get()`：获取当前值
  - `int getAndSet(int newValue)` ：以原子方式设置值并放回旧值
  - `int getAndAdd(int delta) `：以原子方式将给定值与当前值相加并返回旧值
  - `int decrementAndGet()` ：以原子方式将当前值减 1，后返回当前值
  - `int incrementAndGet()` ：以原子方式将当前值加 1，后返回当前值
  - `boolean compareAndSet(int expected,int update)`：如果当前数值等于预期值，则以原子方式设置输入值，成功返回true，否则返回false



- 实例

  ```java
  //以AtomicInteger为例
  class Test implements Runnable{
      //原子类型
      private AtomicInteger atomicInteger = new AtomicInteger();
  
      //自增方法
      public void add(){
          atomicInteger.getAndIncrement();
      }
  
      @Override
      public void run() {
          for (int i = 0; i < 1000; i++){
              add();
          }
      }
  
      //测试
      public static void main(String[] args) {
          Test test = new Test();
          new Thread(test).start();
          new Thread(test).start();
          System.out.println("结果是:"+test.atomicInteger);
      }
  }
  ```

  

------



#### 原子更新数组类型

- **概念：类似于数组的对象，含有原子性操作的方法**



- 类型
  - AtomicIntegerArray （整形数组）
  - AtomicLongArray （长整型数组）
  - AtomicReferenceArray （对象数组）



- 方法
  - **许多方法与基本类型中的方法一样，只是多了一个数组索引作为参数**
  - `AtomicIntegerArray(int length)`： 创建给定长度的新 AtomicIntegerArray。
  - `AtomicIntegerArray(int[] array)`： 创建与给定数组具有相同长度的新 AtomicIntegerArray，并从给定数组复制其所有数



- **注意：这不是一个数组，无法用 名字[索引] 获取，要用get(索引值)方法获取**



- 实例

  ```java
  //以AtomicIntegerArray为例
  class Add implements Runnable{
      //原子数组对象
      private AtomicIntegerArray atomicIntegerArray;
  
      //构造方法
      public Add(AtomicIntegerArray atomicIntegerArray) {
          this.atomicIntegerArray = atomicIntegerArray;
      }
  
      @Override
      public void run() {
          for (int i = 0; i<atomicIntegerArray.length(); i++){
              //进行自增
              atomicIntegerArray.getAndIncrement(i);
          }
      }
  }
  
  class Test{
      public static void main(String[] args) {
          AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(10);
          Add add = new Add(atomicIntegerArray);
          //建立线程测试
          Thread[] threads = new Thread[100];
          for (int i  = 0; i<100; i++){
              threads[i] = new Thread(add);
              threads[i].start();
          }
  
          //等待所有线程执行完毕
          try {
              Thread.sleep(4000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
          //观看结果
          for (int i = 0; i <atomicIntegerArray.length();i++){
              System.out.println(atomicIntegerArray.get(i));
          }
      }
  }
  ```



------



#### 原子更新引用类型

- **概念：原子更新基本类型，每次只能更新一个变量，而当需要更新多个变量，比如一个对象的多个属性时候，我们就必须用到原子更新引用类型类**



- 类型
  - AtomicReference
  - AtomicMarkableReference
  - AtomicStampedReference



- 构造方法

  `AtomicReference<指定对象>`：构造指定对象的原子更新引用类型



- **AtomicMarkableReference 与 AtomicStampedReference说明**

  - **概念：这两者是在AtomicReference基础上进行扩展，解决了ABA问题**

    > 详细链接：https://blog.csdn.net/zhaozhirongfree1111/article/details/72781758

  ```java
  举个通俗点的例子，你倒了一杯水放桌子上，干了点别的事，然后同事把你水喝了又给你重新倒了一杯水，你回来看水还在，拿起来就喝，如果你不管水中间被人喝过，只关心水还在，这就是ABA问题。如果你是一个讲卫生讲文明的小伙子，不但关心水在不在，还要在你离开的时候水被人动过没有，因为你是程序员，所以就想起了放了张纸在旁边，写上初始值0，别人喝水前麻烦先做个累加才能喝水。这就是AtomicStampedReference的解决方案。
  ```

  ```java
  AtomicMarkableReference跟AtomicStampedReference差不多， 
  AtomicStampedReference是使用pair的int stamp作为计数器使用，AtomicMarkableReference的pair使用的是boolean mark。 
  还是那个水的例子，AtomicStampedReference可能关心的是动过几次，AtomicMarkableReference关心的是有没有被人动过，方法都比较简单。
  ```

  

- **细节**

  1. **可用get方法获取指定对象，用于调用对象属性或者对象方法**

     ```java
     //假如这个对象里有name和age属性
     System.out.println(reference.get().name);
     System.out.println(reference.get().age);
     ```

  2. **注意只有调用属于引用类型的原子性方法才能保证原子性**

     ```java
     reference.get().setName("李四");		   //这种不能保证原子性
     reference.compareAndSwap(user1,user2);	//这个才可以
     ```

     

- **实例**

  ```java
  public class AtomicTest3 {
  
      public static void main(String[] args) {
          final AtomicReference<Student>  reference  = new AtomicReference<Student>();
          final Student s1 = new Student("张三", 18);
          Student s2 = new Student("李四",20);
          reference.set(s1);
          Thread t1 = new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      reference.compartAndSet(s1,s2);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          });
          Thread t2 = new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println(reference.get().toString());
              }
          });
          t1.start();
          t2.start();
  
      }
  
      //学生类
      static class Student{
          private String name;
          private int  age;
          public Student(String name, int age) {
              super();
              this.name = name;
              this.age = age;
          }
          public String getName() {
              return name;
          }
          public void setName(String name) {
              this.name = name;
          }
          public int getAge() {
              return age;
          }
          public void setAge(int age) {
              this.age = age;
          }
          @Override
          public String toString() {
              return "Student [name=" + name + ", age=" + age + "]";
          }
      }
  }
  
  结果：
      Student [name=李四, age=20]
  ```



------



#### **原子更新字段** 

- **概念：如果我们只需要某个类里的某个字段，那么就需要使用原子更新字段类，讲一个类中的某一个属性变成原子属性，并进行原子操作**



- 类型
  - AtomicIntegerFieldUpdater 原子更新整型的字段的更新器
  - AtomicLongFieldUpdater 原子更新长整型的字段更新器
  - AtomicStampedReference：原子更新带有版本号的引用类型



- 构造方法

  `AtomicReferenceFieldUpdater newUpdater(指定对象.class,属性名)`：构造一个指定对象类型的指定属性为原子属性



- **细节**
  1. **注意变量的可见范围，由于涉及到反射，不能为private**
  2. **必须用volatile修饰**
  3. **不能用static修饰**



- 实例

  ```java
  //以AtomicIntegerFieldUpdater为例
   public static void main(String[] args) {
          Student s1 = new Student("张三", 18);
       	//只升级了age，没有升级name
          AtomicReferenceFieldUpdater  updater = 				       AtomicReferenceFieldUpdater.newUpdater(Student.class,"age");
      }
  
  	//学生类
      static class Student{
          private String name;
        volatile Integer  age;
          public Student(String name, int age) {
              super();
              this.name = name;
              this.age = age;
          }
          public String getName() {
              return name;
          }
          public void setName(String name) {
              this.name = name;
          }
          public int getAge() {
              return age;
          }
          public void setAge(int age) {
              this.age = age;
          }
          @Override
          public String toString() {
              return "Student [name=" + name + ", age=" + age + "]";
          }
      }
  }
  ```



------



#### 累加器

- **概念：LongAdder类似于atomicLong的，效率更加高；而accumulator是在adder基础上的扩展，更加灵活**

- 类型
  - LongAdder
  - DoubleAdder
  - LongAccumulator
  - DoubleAccumulator



##### Adder

- **LongAdder与atomicLong区别**

  - **原理**

    - **atomicLong原理**

      - **atomicLong每次进行操作后，都要将数据刷新到主存以及其他线程的工作空间，保证可见**

      ![image-20200522222331763](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200522222331763.png)

      - **LongAdder原理**

        - **LongAdder每次进行操作，不用进行刷新数据，而是直接在工作空间里进行累加，到最后将所有线程进行数据汇总**

        - 注意

          1. 效率虽然变快，但消耗了更多空间
          2. **存在线程安全问题，最后进行数据汇总如果有并发更新，可能导致统计数据有误差**

          ![image-20200522222925173](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200522222925173.png)

  - **应用**

    - **并发情况**
      - **低并发时，两者效率差不多**
      - **高并发时，LongAdder效率比atomicLong高很多，但是可能数据会出错**
    - **使用场景**
      - **LongAdder适用于统计求和计数，基本只有add方法，而不含有cas方法**



- LongAdder实例

  ```java
  public class LongAdderDemo {
  
      public static void main(String[] args) throws InterruptedException {
          LongAdder counter = new LongAdder();
          ExecutorService service = Executors.newFixedThreadPool(20);
          long start = System.currentTimeMillis();
          for (int i = 0; i < 10000; i++) {
              service.submit(new Task(counter));
          }
          service.shutdown();
          while (!service.isTerminated()) {
  
          }
          long end = System.currentTimeMillis();
          System.out.println(counter.sum());
          System.out.println("LongAdder耗时：" + (end - start));
      }
  
      private static class Task implements Runnable {
  
          private LongAdder counter;
  
          public Task(LongAdder counter) {
              this.counter = counter;
          }
  
          @Override
          public void run() {
              for (int i = 0; i < 10000; i++) {
                  counter.increment();
              }
          }
      }
  }
  ```



------



##### Accumulator

- **方法**
  - **`new LongAccumulator((x, y) -> 表达式, 初始值)`：构造方法**
  - **`void accumulate(long number)`：进行累加**



- **使用步骤**
  1. **利用构造方法新建一个Accumulator，不写初始值默认为0**
  2. **使用`accumulate`方法进行累加，`accumulate`方法的参数是传到x上，而之前累加的值传到y上**



- LongAccumulator实例

  ```java
  public class LongAccumulatorDemo {
  
      public static void main(String[] args) {
          //这里的形式可以改变，比如可以写成(x, y) -> x+y
          LongAccumulator accumulator = new LongAccumulator((x, y) -> 2 + x * y, 1);
          ExecutorService executor = Executors.newFixedThreadPool(8);
          IntStream.range(1, 10).forEach(i -> executor.submit(() -> accumulator.accumulate(i)));
  
          executor.shutdown();
          while (!executor.isTerminated()) {
          }
          System.out.println(accumulator.getThenReset());
      }
  }
  ```



------



### Lock接口

- 概念：锁是一种工具，用于控制对共享资源的访问**Lock与synchronized是最常见的锁，但两者不是相互替代的，Lock功能上更高级，但相应的花销可能更大**



- 为什么需要Lock
  1. synchronized效率低
  2. synchronized不够灵活
  3. synchronized无法确定是否成功获取锁



#### 思维导图

![img](https://img-blog.csdnimg.cn/201906251151037.jpg)

#### 方法

- `lock()`与`unlock()`：获取锁和释放锁

  - **特点：这个不会像synchronized一样，遇到异常自动释放锁，所以必须使用try-finally写法**

    ```java
    lock.lock();
    try{
        .....
    }
    finally{
        lock.unlock();
    }
    ```



- `boolean tryLock()`：用于尝试获取锁，如果当前锁没有被占用，则获取锁并返回true，反之相反

- `boolean tryLock(long time,TimeUnit unit)`：有相同效果，**但如果超过参数指定的超时时间，则放弃**
  - **tryLock也必须使用try-finally写法**

    ```java
    //获取锁成功
    if(lock.tryLock()){
        try{
            ...
        }
        finally{
            lock.unlock();
        }
    }
    //获取锁失败
    else{
        ....
    }
    ```

  - tryLock实例

  ```java
  //两个线程互相争抢两把锁
  public class TryLockDeadlock implements Runnable {
  
  
      int flag = 1;
      static Lock lock1 = new ReentrantLock();
      static Lock lock2 = new ReentrantLock();
  
      public static void main(String[] args) {
          TryLockDeadlock r1 = new TryLockDeadlock();
          TryLockDeadlock r2 = new TryLockDeadlock();
          r1.flag = 1;
          r1.flag = 0;
          new Thread(r1).start();
          new Thread(r2).start();
  
      }
  
      @Override
      public void run() {
          for (int i = 0; i < 100; i++) {
              if (flag == 1) {
                  try {
                      if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程1获取到了锁1");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock2.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程1获取到了锁2");
                                      System.out.println("线程1成功获取到了两把锁");
                                      break;
                                  } finally {
                                      lock2.unlock();
                                  }
                              } else {
                                  System.out.println("线程1获取锁2失败，已重试");
                              }
                          } finally {
                              lock1.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程1获取锁1失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
  
              if (flag == 0) {
                  try {
                      if (lock2.tryLock(3000, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程2获取到了锁2");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程2获取到了锁1");
                                      System.out.println("线程2成功获取到了两把锁");
                                      break;
                                  } finally {
                                      lock1.unlock();
                                  }
                              } else {
                                  System.out.println("线程2获取锁1失败，已重试");
                              }
                          } finally {
                              lock2.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程2获取锁2失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
  ```

  

- `lockInterruptibly()`：作用与tryLock一样，不过超时时间为无限，**且在等待过程是可以被打断的**

  ```java
  public class LockInterruptibly implements Runnable {
  
      private Lock lock = new ReentrantLock();
  
      public static void main(String[] args) {
          LockInterruptibly lockInterruptibly = new LockInterruptibly();
          Thread thread0 = new Thread(lockInterruptibly);
          Thread thread1 = new Thread(lockInterruptibly);
          thread0.start();
          thread1.start();
  
          try {
              Thread.sleep(2000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          //线程打断方法
          thread1.interrupt();
  }
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + "尝试获取锁");
          try {
              lock.lockInterruptibly();
              try {
                  System.out.println(Thread.currentThread().getName() + "获取到了锁");
                  Thread.sleep(5000);
              } catch (InterruptedException e) {
                  System.out.println(Thread.currentThread().getName() + "睡眠期间被中断了");
              } finally {
                  lock.unlock();
                  System.out.println(Thread.currentThread().getName() + "释放了锁");
              }
          } catch (InterruptedException e) {
              System.out.println(Thread.currentThread().getName() + "获得锁期间被中断了");
          }
      }
  }
  ```

  

------



#### 锁的分类

- **概念：锁有很多种分类，这个分类是从不同角度来看待，它们并非是独立的，就是说一个锁可能即是这种锁，也可能是那种锁**



- 分类图

  ![image-20200523232921461](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200523232921461.png)



------



##### 乐观锁和悲观锁

- **悲观锁（互斥同步锁）**

  - 概念：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞，直到它拿到锁

  

  - **举例说明**

    1. 有两个线程，线程1、线程2竞争相同资源
    2. 线程1先抢到锁，这时线程2拿不到锁会被阻塞，**此时线程2无法访问共享资源直至拿到锁**

    

  - **实例：synchronized和Lock接口**






- **乐观锁（非互斥同步锁）**

  - 概念：总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，**核心是使用了cas算法**

  

  - 举例说明
    1. 有两个线程，线程1、线程2竞争相同资源
    2. **线程1先抢到锁，这时线程2仍能访问共享资源并作出相应操作，但会判断数据是否改变**

  

  - **实例：并发容器与原子类**

    

    

- 两者对比

  - 悲观锁：悲观锁一开始开销比较大，但特点是一劳永逸
  - 乐观锁：乐观锁一开始开销比较小，但如果自旋时间过长或者不断重试，则花销会变大

  

- 应用

  ![image-20200523234413804](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200523234413804.png)

------



##### 可重入锁和非可重入锁

- **概念：可重入是指已经获取锁的线程能再次获取这个锁，不可重入是指已经获取锁的线程能不次获取这个锁，必须释放后才能再次获得**



- 优点
  1. 避免死锁
  2. 提升封装性，不必每次都要释放锁后再获得锁



- **方法**
  - `int getHoldCount()`：查询当前线程保持此锁定的个数，即调用lock()方法的次数
  - `int getQueueLength()`：返回正等待获取此锁定的预估线程数
  - `boolean isHeldByCurrentThread()`：判断当前线程是否获取了改锁



- **实例：以ReentrantLock为例**

  ```java
  //资源要连续处理五次才能处理完成
  public class RecursionDemo {
  
      private static ReentrantLock lock = new ReentrantLock();
  
      private static void accessResource() {
          lock.lock();
          try {
              System.out.println("已经对资源进行了处理");
              if (lock.getHoldCount()<5) {
                  System.out.println(lock.getHoldCount());
                  //线程递归
                  accessResource();
                  System.out.println(lock.getHoldCount());
              }
          } finally {
              lock.unlock();
          }
      }
      public static void main(String[] args) {
          accessResource();
      }
  }
  ```

  

------



##### 公平锁和非公平锁

- **概念：公平指按照线程请求顺序来分配锁，非公平指不完全按照线程请求顺序分配锁，而是按照合适的情况来分配锁**



- 说明

  - 公平

    ![image-20200524001046300](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200524001046300.png)

    ![image-20200524001109782](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200524001109782.png)

  - 非公平

    ![image-20200524001255250](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200524001255250.png)

    ![image-20200524001237163](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200524001237163.png)



- **实例：以ReentrantLock为例**

  ```java
  /*
  	模拟打印，每一次打印需要连续获取两次锁才可以打印完成
  	公平：10个线程各自先打印第一次，再有第二轮打印第二次
  	非公平：第一个线程直接打印两次，然后轮到下一个线程打印两次
  */
  public class FairLock {
  
      public static void main(String[] args) {
          PrintQueue printQueue = new PrintQueue();
          Thread thread[] = new Thread[10];
          for (int i = 0; i < 10; i++) {
              thread[i] = new Thread(new Job(printQueue));
          }
          for (int i = 0; i < 10; i++) {
              thread[i].start();
              try {
                  Thread.sleep(100);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  
  class Job implements Runnable {
  
      PrintQueue printQueue;
  
      public Job(PrintQueue printQueue) {
          this.printQueue = printQueue;
      }
  
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + "开始打印");
          printQueue.printJob(new Object());
          System.out.println(Thread.currentThread().getName() + "打印完毕");
      }
  }
  
  class PrintQueue {
  
      //公平
      private Lock queueLock = new ReentrantLock(true);
      //非公平
      //private Lock queueLock = new ReentrantLock(false);
  
      public void printJob(Object document) {
          queueLock.lock();
          try {
              int duration = new Random().nextInt(10) + 1;
              System.out.println(Thread.currentThread().getName() + "正在打印，需要" + duration);
              Thread.sleep(duration * 1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              queueLock.unlock();
          }
  
          /*
          	公平：此时会先执行等待队列中的线程，而这个线程会重新排队
          	非公平：会直接执行以下代码
          */
          queueLock.lock();
          try {
              int duration = new Random().nextInt(10) + 1;
              System.out.println(Thread.currentThread().getName() + "正在打印，需要" + duration+"秒");
              Thread.sleep(duration * 1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              queueLock.unlock();
          }
      }
  }
  ```



- 特例
  - tryLock()方法不满足公平性，如果线程释放了锁，调用tryLock方法的线程就会抢到锁，不管等待队列前是否有其他线程



- 对比

  ![image-20200524002436237](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200524002436237.png)

  

------



##### 共享锁和排他锁

- **概念**
  - **共享锁（读锁）：获取共享锁后，无法进行修改数据，只能查看数据，但允许多个线程同时获得共享锁**
  - **排他锁（独占锁）：锁被获取后，其他线程不得对共享资源进行操作**



- **典型：读写锁ReentrantReadWriteLock**
  - **概念：读写锁有两种锁定方式，一种是写锁定属于独占锁，一种是读锁定属于共享锁**
  - **规则**
    1. **可以有多个线程申请读锁**
    2. **只允许一个写锁存在**
    3. **读锁和写锁不能并存**



- 实例

  ```java
  public class CinemaReadWrite {
  
      //获得读写锁
      private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
      //获得读锁
      private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
      //获得写锁
      private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
  
      private static void read() {
          readLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了读锁，正在读取");
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放读锁");
              readLock.unlock();
          }
      }
  
      private static void write() {
          writeLock.lock();
          try {
              System.out.println(Thread.currentThread().getName() + "得到了写锁，正在写入");
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              System.out.println(Thread.currentThread().getName() + "释放写锁");
              writeLock.unlock();
          }
      }
  
      public static void main(String[] args) {
          new Thread(()->read(),"Thread1").start();
          new Thread(()->read(),"Thread2").start();
          new Thread(()->write(),"Thread3").start();
          new Thread(()->write(),"Thread4").start();
      }
  }
  ```



- **读锁和写锁的交互**
  - 公平性
    - 公平：所有读写锁按顺序执行
    - **非公平**
      1. **写锁是可以插队的（写锁插队可以避免饥饿）**
      2. **读锁仅在等待队列头结点不是想获取写锁的线程的时候是可以插队的**
    
    
    
  - **升降级**
  
    - 相比来说，写锁更高级，读锁较低级
    - **规则：只支持降级，不支持升级**
  
    > 细节：https://blog.csdn.net/aitangyong/article/details/38315885?ops_request_misc=&request_id=&biz_id=102&utm_term=%E8%AF%BB%E5%86%99%E9%94%81%E5%8D%87%E9%99%8D%E7%BA%A7&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-38315885
  
    ```java
    public class Upgrading {
    
        private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(
                false);
        private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
        private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();
    
        //升级
        private static void readUpgrading() {
            readLock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "得到了读锁，正在读取");
                Thread.sleep(1000);
                System.out.println("升级会带来阻塞");
                writeLock.lock();
                System.out.println(Thread.currentThread().getName() + "获取到了写锁，升级成功");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(Thread.currentThread().getName() + "释放读锁");
                readLock.unlock();
            }
        }
    
        //降级
        private static void writeDowngrading() {
            writeLock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "得到了写锁，正在写入");
                Thread.sleep(1000);
                readLock.lock();
                System.out.println("在不释放写锁的情况下，直接获取读锁，成功降级");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readLock.unlock();
                System.out.println(Thread.currentThread().getName() + "释放写锁");
                writeLock.unlock();
            }
        }
    
        public static void main(String[] args) throws InterruptedException {
    //        System.out.println("先演示降级是可以的");
    //        Thread thread1 = new Thread(() -> writeDowngrading(), "Thread1");
    //        thread1.start();
    //        thread1.join();
    //        System.out.println("------------------");
    //        System.out.println("演示升级是不行的");
            Thread thread2 = new Thread(() -> readUpgrading(), "Thread2");
            thread2.start();
        }
    }
    
    ```
  
    

------



##### 自旋锁和阻塞锁

- **概念**
  - **自旋锁：运用了cas算法，不停的运行线程，进行尝试获得锁，直到获得锁成功**
  - **阻塞锁：当与其他线程竞争失败后，就会处于阻塞状态，必须由其他线程唤醒才能继续参与竞争**



- 实例

  ```java
  public class SpinLock {
  
      private AtomicReference<Thread> sign = new AtomicReference<>();
  
      public void lock() {
          Thread current = Thread.currentThread();
          while (!sign.compareAndSet(null, current)) {
              System.out.println("自旋获取失败，再次尝试");
          }
      }
  
      public void unlock() {
          Thread current = Thread.currentThread();
          sign.compareAndSet(current, null);
      }
  
      public static void main(String[] args) {
          SpinLock spinLock = new SpinLock();
          Runnable runnable = new Runnable() {
              @Override
              public void run() {
                  System.out.println(Thread.currentThread().getName() + "开始尝试获取自旋锁");
                  spinLock.lock();
                  System.out.println(Thread.currentThread().getName() + "获取到了自旋锁");
                  try {
                      Thread.sleep(300);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      spinLock.unlock();
                      System.out.println(Thread.currentThread().getName() + "释放了自旋锁");
                  }
              }
          };
          Thread thread1 = new Thread(runnable);
          Thread thread2 = new Thread(runnable);
          thread1.start();
          thread2.start();
      }
  }
  ```

  



- 运用

  ![image-20200524085330987](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200524085330987.png)

------



#### 线程八锁

- **概念：八个例子，用于理解锁锁住了什么** ，**区分类锁和对象锁**



- **规则**
  1. **锁住了什么？**
     - 非静态同步方法，锁住了一个对象中所有非静态同步操作，不包括普通方法和静态同步方法
     - 静态同步方法，锁住了一个类中所有静态同步操作，不包括普通方法和非静态同步方法
  2. **对象锁和类锁**
     - 对象锁：一个对象里的非静态同步方法互斥，不同对象的非静态同步方法不互斥
     - 类锁：一个类里的静态同步方法互斥，即使有多个对象里的静态同步方法也互斥



```java
//例子1
/*
	条件：
		1.线程A调用邮件方法，线程B调用SMS方法
		2.调用B线程前睡0.1秒
*/
class Phone{
    public synchronized void sendEmail(){
        System.out.println("*****sendEmail");
    }

    public synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendEmail
    ****sendSMS
    
分析：锁锁住的是所有同步的操作，所以邮件方法和SMS方法是锁住的，每次只能调用1个这两个线程本来应该是去争取        CPU使用权的，但是B线程前线程睡了1毫秒，所以A线程比较先拿到CPU使用权，所以先打印邮件方法，再打印SMS      方法
```



```java
//例子2
/*
	条件：
		1.线程A调用邮件方法，线程B调用SMS方法
		2.邮件方法睡4秒
		3.调用B线程前睡0.1秒
*/
class Phone{
    public synchronized void sendEmail(){
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*****sendEmail");
    }

    public synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendEmail
    ****sendSMS
    
分析：与第一个例子原理一样，不管是否有邮件方法是否睡眠了4秒这两个线程本来应该是去争取CPU使用权的，但是B线程      前线程睡了1毫秒，所以A线程比较先拿到CPU使用权，所以先打印邮件方法，再打印SMS方法
```



```java
//例子3
/*
	条件：
		1.新增普通方法
		2.线程A调用邮件方法，线程B调用普通方法
		3.邮件方法睡4秒
		4.调用B线程前睡0.1秒
*/
class Phone{
    public synchronized void sendEmail(){
        System.out.println("*****sendEmail");
    }

    public synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
    
    public void sendHello(){
        System.out.println("*****sendHello");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone.sendHello();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendHello
    ****sendEmail
    
分析：由于锁锁住的是同步的操作，对于普通（非同步）方法是不上锁的，所以线程B调用普通方法与线程A调用同步方法      是不互斥的，两者同时进行，由于邮件方法睡了4秒，而普通方法不用沉睡，所以先打印普通方法，再打印邮件
```



```java
//例子4
/*	
	条件：
		1.两部手机（两个实例对象）
		2.线程A调用对象1邮件方法，线程B调用对象2SMS方法
		3.邮件方法睡4秒
		4.调用B线程前睡0.1秒
*/
class Phone{
    public synchronized void sendEmail(){
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*****sendEmail");
    }

    public synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone2.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendSMS
    ****sendEmail
    
分析：由于非静态同步方法锁的是一个对象里的所有同步操作，这里有两个对象，所以AB线程互不干涉，由于邮件方法睡      眠4秒所以先打应SMS，再打印邮件
```



```java
//例子5
/*
	条件：
		1.邮件和SMS方法都为静态方法
		2.线程A调用邮件方法，线程B调用SMS方法
		3.邮件方法睡4秒
		4.调用B线程前睡0.1秒
*/
class Phone{
    public static synchronized void sendEmail(){
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*****sendEmail");
    }

    public static synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendEmail
    ****sendSMS
    
分析：静态方法锁锁住的是这个类的所有静态同步操作，所以由于线程A先执行，线程B就不能调用了，必须等A执行完，所		以先打印邮件，再打印SMS
```



```java
//例子6
/*
	条件：
		1.邮件和SMS方法都为静态方法
		2.两部手机（两个实例对象）
		3.线程A调用对象1邮件方法，线程B调用对象2SMS方法
		4.邮件方法睡4秒
		5.调用B线程前睡0.1秒
*/
class Phone{
    public static synchronized void sendEmail(){
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*****sendEmail");
    }

    public static synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone2.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendEmail
    ****sendSMS
    
分析：静态方法锁的是类，所以即使是两个对象，也要受到影响，所以先打印邮件，再打印SMS
```



```java
//例子7
/*
	条件：
		1.邮件为静态方法，SMS方法为普通同步方法
		2.线程A调用邮件方法，线程B调用SMS方法
		3.邮件方法睡4秒
		4.调用B线程前睡0.1秒
*/
class Phone{
    public static synchronized void sendEmail(){
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*****sendEmail");
    }

    public synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendSMS
    ****sendEmail
    
分析：静态方法锁住的是静态同步方法，非静态同步方法是属于对象的，不受影响，有点类似于非静态同步方法和普通方法	   关系，所以AB线程互不干扰，由于邮件有沉睡4秒，所以先打印SMS，在打印邮件
```



```java
//例子8
/*
	条件：
		1.邮件为静态方法，SMS方法为普通同步方法
		2.两部手机（两个实例对象）
		3.线程A调用对象1邮件方法，线程B调用对象2SMS方法
		4.邮件方法睡4秒
		5.调用B线程前睡0.1秒
*/
class Phone{
    public static synchronized void sendEmail(){
        TimeUnit.SECONDS.sleep(4);
        System.out.println("*****sendEmail");
    }

    public synchronized void sendSMS(){
        System.out.println("*****sendSMS");
    }
}

class Demo {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            try{
                phone.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"A").start();

        Thread.sleep(100);

        new Thread(()->{
            try{
                phone2.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"B").start();
    }
}

结果
    ****sendSMS
    ****sendEmail
    
分析：由上一例可知，静态同步和非静态同步是互不干涉的，这里即使是两个对象也是同理
```



------



#### Condition

- 概念：条件对象，用于控制线程流程，安排线程顺序，**类似于唤醒机制，使用Condition使某个线程等待，再用另一个线程唤醒**



- **方法**
  - **`void await()`：使线程进入等待状态**
  - **`void signal()`：唤醒等待队列中等待时间最长的线程**
  - **`void signalAll()`：唤醒所有等待的线程**



- 使用步骤

  1. 创建Condition对象（由Lock对象调用方法生成）
  2. 在需要等待的线程中调用`await`方法
  3. 用其他线程去唤醒

  ![image-20200529093023552](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529093023552.png)



- 实例

  ```java
  /**
   * 描述：     演示Condition的基本用法
   */
  class ConditionDemo1 {
      private ReentrantLock lock = new ReentrantLock();
      private Condition condition = lock.newCondition();
  
      void method1() throws InterruptedException {
          lock.lock();
          try{
              System.out.println("条件不满足，开始await");
              condition.await();
              System.out.println("条件满足了，开始执行后续的任务");
          }finally {
              lock.unlock();
          }
      }
  
      void method2() {
          lock.lock();
          try{
              System.out.println("准备工作完成，唤醒其他的线程");
              condition.signal();
          }finally {
              lock.unlock();
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
          ConditionDemo1 conditionDemo1 = new ConditionDemo1();
          //注意，要先等待再唤醒
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Thread.sleep(1000);
                      conditionDemo1.method1();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }).start();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      Thread.sleep(1000);
                      conditionDemo1.method2();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }).start();
      }
  }
  ```

  ```java
  /**
   * 描述：     演示用Condition实现生产者消费者模式
   */
  public class ConditionDemo2 {
  
      private int queueSize = 10;
      private PriorityQueue<Integer> queue = new PriorityQueue<Integer>(queueSize);
      private Lock lock = new ReentrantLock();
      private Condition notFull = lock.newCondition();
      private Condition notEmpty = lock.newCondition();
  
      public static void main(String[] args) {
          ConditionDemo2 conditionDemo2 = new ConditionDemo2();
          Producer producer = conditionDemo2.new Producer();
          Consumer consumer = conditionDemo2.new Consumer();
          producer.start();
          consumer.start();
      }
  
      class Consumer extends Thread {
  
          @Override
          public void run() {
              consume();
          }
  
          private void consume() {
              while (true) {
                  lock.lock();
                  try {
                      while (queue.size() == 0) {
                          System.out.println("队列空，等待数据");
                          try {
                              notEmpty.await();
                          } catch (InterruptedException e) {
                              e.printStackTrace();
                          }
                      }
                      queue.poll();
                      notFull.signalAll();
                      System.out.println("从队列里取走了一个数据，队列剩余" + queue.size() + "个元素");
                  } finally {
                      lock.unlock();
                  }
              }
          }
      }
  
      class Producer extends Thread {
  
          @Override
          public void run() {
              produce();
          }
  
          private void produce() {
              while (true) {
                  lock.lock();
                  try {
                      while (queue.size() == queueSize) {
                          System.out.println("队列满，等待有空余");
                          try {
                              notFull.await();
                          } catch (InterruptedException e) {
                              e.printStackTrace();
                          }
                      }
                      queue.offer(1);
                      notEmpty.signalAll();
                      System.out.println("向队列插入了一个元素，队列剩余空间" + (queueSize - queue.size()));
                  } finally {
                      lock.unlock();
                  }
              }
          }
      }
  
  }
  ```

  

------



### 并发容器

#### 思维导图

![img](https://gitee.com/helloword7/Images/raw/master/thread/17.png)



------

#### 知识补充

##### Vector与Hashtable

- 概念：两个过时的容器，性能差，已被取代，**但他们是线程安全的**



- **特点：这两个容器的底层源码方法都使用了synchronized修饰**

  - **线程安全**
  - **由于synchronized的限制，并发性能差**
  - **并发操作时，可能产生异常**

  ![image-20200527225546520](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200527225546520.png)



------



##### 同步的Map与List

- 概念：使用`Collections.synchronizedMap()`或者`Collections.synchronizedList()`对Map与List进行升级，**使他们变成线程安全**



- **缺点：与Vector与Hashtable线程安全的原理一样，升级后在底层源码方法使用了synchronized同步代码块，性能低下**



- 用法

  ```java
  public class SynList {
      public static void main(String[] args) {
          List<Integer> list = Collections.synchronizedList(new ArrayList<Integer>());
          Map<Object, Object> objectObjectMap = Collections.synchronizedMap(new HashMap<>());
      }
  }
  ```



- **同步容器的弊端**
  - **容器迭代时不允许修改内容**
  - **并发时修改数据会出现线程安全问题（只读的并发是安全的）**



- **HashMap死机问题（存在于jdk1.7版本及以下）**

  - **现象：CPU使用率100%导致死机**

  - **原因：在并发操作时，对HashMap的扩容造成链表死循环，你指向我，我指向你**
  
  - > 详情：https://coolshell.cn/articles/9606.html



------



##### HashMap的数据结构

- 概念：HashMap的数据结构在jdk1.8时发生了较大变化，**从之前的拉链法到现在的使用红黑树**



- **结构**

  - **java7：使用了拉链法，当数据的哈希值一样时，便会生成链表**

    ![image-20200528213500635](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200528213500635.png)

    

  - **java8：使用了红黑树，当数据的哈希值一样且个数超过一定数量时，不再使用链表而是使用红黑树**

    ![image-20200528213902580](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200528213902580.png)





------



#### Concurrent

- **概念：一个线程安全的集合容器，是Map集合和Set集合的线程安全升级版，Map集合的升级版都是Concurrent，Set集合升级版还有一个是CopyOnWrite，下面笔记主要以ConcurrentMap为例**



- **种类**

  | 集合类型 | 对应的并发容器         | 介绍                         |
  | -------- | ---------------------- | ---------------------------- |
  | Map      | ConcurrentMap          | 接口                         |
  | Map      | ConcurrentHashMap      | ConcurrentMap实现类          |
  | Map      | ConcurrentNavigableMap | ConcurrentMap子接口          |
  | Map      | ConcurrentSkipListMap  | ConcurrentNavigableMap实现类 |
  | Set      | ConcurrentSkipListSet  | 类                           |

  

##### ConcurrentHashMap

###### 结构

- 概念：ConcurrentHashMap在1.8时结构发生了较大变化

  

- **Java7：使用分段锁，每一个Segment里面有许多个类似于HashMap的数据结构，且每一个Segment都实现了ReentrantLock，每一个Segment都是一个独立的锁，所以实现了并发操作**

![image-20200528220643522](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200528220643522.png)


- **Java8：取消了分段锁，数据结构与Java8的HashMap结构一样，但使用了cas算法和用synchronized锁住**

  ![image-20200528220735803](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200528220735803.png)

- 两者对比

  1. 提高了并发度，从1.7的16变成了每个Node都并发
  2. 提高了性能，使用了红黑树



------



###### 使用

- **概念：ConcurrentHashMap虽然是线程安全的，但并不是所有操作都是线程安全的，它只能保证一次的操作在并发时是安全的，进行复合操作仍然是不安全的，<u>必须使用它提供的进行组合操作方法来保证线程安全</u>**



- 实例

```java
//出现线程安全问题的ConcurrentHashMap
public class OptionsNotSafe implements Runnable {

    private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<String, Integer>();

    public static void main(String[] args) throws InterruptedException {
        scores.put("小明", 0);
        Thread t1 = new Thread(new OptionsNotSafe());
        Thread t2 = new Thread(new OptionsNotSafe());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(scores);
    }


    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            	//这样进行多次的get和put是不能保证线程安全的
                Integer score = scores.get("小明");
                Integer newScore = score + 1;
                scores.put("小明",newScore);
        }
    }
}
```

```java
//使用提供的方法保证线程安全
public class OptionsNotSafe implements Runnable {

    private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<String, Integer>();

    public static void main(String[] args) throws InterruptedException {
        scores.put("小明", 0);
        Thread t1 = new Thread(new OptionsNotSafe());
        Thread t2 = new Thread(new OptionsNotSafe());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(scores);
    }


    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            while (true) {
                Integer score = scores.get("小明");
                Integer newScore = score + 1;
                //使用其特有的replace方法，将复合操作化为1步，就不会出现线程安全问题
                boolean b = scores.replace("小明", score, newScore);
                if (b) {
                    break;
                }
            }
        }

    }
}
```



------



### 并发工具类

- **概念：一些用于并发操作的工具类，用于线程协作，控制并发流程，用于满足我们的业务逻辑**



#### CountDownLatch

- 概念：倒数门闩，顾名思义，**类似于一个计数器，只有倒计时的数到了0才能执行以后的代码，否则线程会被阻塞住**



- **方法**
  - **`CountDownLatch(int count)`：构造函数，构造一个倒数门闩，参数是倒数的数**
  - **`void await()`：等待方法，调用此方法的线程会被挂起，会等到倒数完毕才会执行下面的代码**
  - **`void countDown()`：将倒数的值减1**



- 使用步骤
  1. 使用构造函数构造一个倒数门闩
  2. 进行倒数，这段期间线程处于等待状态
  3. 倒数完成，执行线程接下来的任务



- **应用**

  - **一等多：多个线程等待一个线程执行结束后才开始进行**
  - **多等一：一个线程等待多个线程执行结束后才开始进行**
  - 一等一：一个线程等待一个线程执行结束后才才开始进行
  - 多等多：多个线程等待多个线程执行结束后才才开始进行

  

- **实例：一对多，多对一**

  ```java
  /*
   描述：模拟100米跑步，5名选手都准备好了，只等裁判员一声令下，所有人同时开始跑步。当所有人都到终点后，	   比赛结束。
   */
  public class CountDownLatchDemo1And2 {
  
      public static void main(String[] args) throws InterruptedException {
          CountDownLatch begin = new CountDownLatch(1);
          CountDownLatch end = new CountDownLatch(5);
          ExecutorService service = Executors.newFixedThreadPool(5);
          
          for (int i = 0; i < 5; i++) {
              final int no = i + 1;
              Runnable runnable = new Runnable() {
                  @Override
                  public void run() {
                      System.out.println("No." + no + "准备完毕，等待发令枪");
                      try {
                          begin.await();
                          System.out.println("No." + no + "开始跑步了");
                          Thread.sleep((long) (Math.random() * 10000));
                          System.out.println("No." + no + "跑到终点了");
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      } finally {
                          end.countDown();
                      }
                  }
              };
              service.submit(runnable);
          }
          
          //裁判员检查发令枪...
          Thread.sleep(5000);
          System.out.println("发令枪响，比赛开始！");
          begin.countDown();
  
          end.await();
          System.out.println("所有人到达终点，比赛结束");
      }
  }
  ```

  

------



#### Semaphore

- 概念：信号量，可以用来限制或管理数量有限的资源的使用情况，**也可以理解为许可证，只有获得许可证线程才能访问或者进行某些操作，而没有获取许可证的线程会被挂起，直到获取许可证才进行**

  ![image-20200529002030598](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529002030598.png)



- **方法**

  - **`Semaphore(int permits,boolean fair)`：构造方法，创建一个信号量，第一个参数是许可证的数量，第二个参数是指定是否公平，默认为true**
    - **公平：等待获取许可证的队列是公平的，不准插队，等待最久的线程首先获取许可证**
    - **不公平：等待获取许可证的队列是不公平的，可插队**
  - **`void acquire(int permits)`：获取许可证，参数是指一次性拿走多少张许可证，默认为1**

  - **`void acquireUninterruptibly`：作用与acquire一样，但获取过程是可以被打断的**

  - **`boolean tryacquire(int permits,long time,TimeUnit unit)`：与tryLock方法一样，这里也是进行尝试获取许可证**
    - **注意：这个方法是不公平的**
  - **`void release()`：归还许可证**



- 使用步骤
  1. 初始化Semaphore并指定许可证数量
  2. 在需要获取许可证的线程中使用`acquire()`或者`acquireUninterruptibly()`获取许可证
  3. 线程执行结束后，调用`release()`归还许可证



- **信号量特殊操作**

  1. **一次性获取或释放多个许可证**

     - 比如有两个线程AB，线程A比较消耗资源，那么我们就规定线程A要拿到多个许可证才可以运行，线程B拿到比较少的许可证就可以运行
     - **注意：一般拿到多少许可证就要释放多少许可证**

     

  2. **信号量的获取和释放并不一定是要同一个线程**

     - 可以是线程A获取，线程B释放，只要合乎逻辑即可

     

  3. **由于信号量可以实现条件等待，所以可以相当于一个轻量级的countDownLatch**

     - **线程1完成准备工作线程2才开始：线程2调用acquire()，线程1调用release()**
     
     ```java
     class SemaphoreDemo {
     
         static Semaphore semaphore = new Semaphore(0, true);
     
         public static void main(String[] args) {
             new Thread(new Runnable() {
                 @Override
                 public void run() {
                     try {
                         semaphore.acquire();
                     } catch (InterruptedException e) {
                         e.printStackTrace();
                     }
                     System.out.println(Thread.currentThread().getName() + "拿到了许可证");
                     try {
                         Thread.sleep(2000);
                     } catch (InterruptedException e) {
                         e.printStackTrace();
                     }
                     System.out.println(Thread.currentThread().getName() + "释放了许可证");
                     semaphore.release();
                 }
             }, "A").start();
     
             new Thread(new Runnable() {
                 @Override
                 public void run () {
                     try {
                         Thread.sleep(2000);
                     } catch (InterruptedException e) {
                         e.printStackTrace();
                     }
                     System.out.println(Thread.currentThread().getName() + "释放了许可证");
                     semaphore.release();
                 }
             },"B").start();
     	}
     }
     ```
     
     



- **实例**

  ```java
  public class SemaphoreDemo {
  
      static Semaphore semaphore = new Semaphore(5, true);
  
      public static void main(String[] args) {
          ExecutorService service = Executors.newFixedThreadPool(50);
          for (int i = 0; i < 100; i++) {
              service.submit(new Task());
          }
          service.shutdown();
      }
  
      static class Task implements Runnable {
  
          @Override
          public void run() {
              try {
                  semaphore.acquire();
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "拿到了许可证");
              try {
                  Thread.sleep(2000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "释放了许可证");
              semaphore.release();
          }
      }
  }
  ```



------



#### CyclicBarrier

- 概念：循环栅栏，**与 CountDownLatch有点类似，都能阻塞一组线程，不过不同的是倒计时的不是专门的数，而是线程数量**





- **方法**
  - **`CyclicBarrier(int number)`：创建一个CyclicBarrier对象，参数是要等待的线程数**
  - **`void await()`：等待方法，只有线程数量到达指定数量，才能执行下面的代码**





- **与CountDownLatch区别**

  1. **用于倒计时的数不一样，CountDownLatch是专门的计时数，而CylicBarrier是线程数量**
  2. **CountDownLatch不可重用，CylicBarrier可以重用**
     - **CountDownLatch倒计数为0时，就不能再起作用了**
     - **CylicBarrier即使为0后，还能继续使用**

  ![image-20200529113717395](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529113717395.png)

- **实例**

```java
/**
 * 描述：    演示CyclicBarrier
 */
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("所有人都到场了， 大家统一出发！");
            }
        });
        for (int i = 0; i < 10; i++) {
            new Thread(new Task(i, cyclicBarrier)).start();
        }

    }

    static class Task implements Runnable{
        private int id;
        private CyclicBarrier cyclicBarrier;

        public Task(int id, CyclicBarrier cyclicBarrier) {
            this.id = id;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程" + id + "现在前往集合地点");
            try {
                Thread.sleep((long) (Math.random()*10000));
                System.out.println("线程"+id+"到了集合地点，开始等待其他人到达");
                cyclicBarrier.await();
                System.out.println("线程"+id+"出发了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```



------



#### Exchanger

- **概念：交换者，用于进行线程间的数据交换,它提供一个同步点,在这个同步点,两个线程通过`exchange`方法交换(共享)彼此的数据**



- **方法**
  - **`Exchanger(V x)`：构建一个交换者，参数是要交换的数据类型，这里用到了泛型**
  - **`V exchange(V x)`**
    - **对于发送者：发送参数作为共享数据，V是泛型**
    - **对于接收者：返回值是要共享的数据，但如果没有发送者发送改线程会被阻塞，直至有人发送**
  - **`V exchange(V x, long timeout, TimeUnit unit) `：这种对于接收者可以设置等待时间**



- **注意：交换区存储的数据只能有一个，如果再次交换数据，原有的数据会被替代**



- **实例**

  ```java
  /*
  	老板给员工发工资，工资就是要发送的数据
  */
  class demo{
      public static void main(String[] args) {
          Exchanger<Integer> exchanger = new Exchanger<>();
          new Boss(exchanger).start();
          new Employee(exchanger).start();
      }
  }
  
  class Boss extends Thread {
      Exchanger<Integer> exchanger = null;
      public Boss(Exchanger<Integer> exchanger) {
          super();
          this.exchanger = exchanger;
      }
      @Override
      public void run() {
          int money = 10000;
          int msg = 0;
          try {
              exchanger.exchange(money);
              msg = exchanger.exchange(msg);
              if (msg == 1){
                  System.out.println("继续加油");
              }
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  
  class Employee extends Thread {
      Exchanger<Integer> exchanger = null;
      public Employee(Exchanger<Integer> exchanger) {
          super();
          this.exchanger = exchanger;
      }
      @Override
      public void run() {
          int money = 0;
          int msg = 1;
          try {
              money = exchanger.exchange(money);
              System.out.println("收到工资："+money);
              exchanger.exchange(msg);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```

  



-------



### 线程池

- **概念：一个容器，存储了许多可以复用的线程，提高了效率**



- 优点
  1. 避免了反复创造线程，减少了线程创建和销毁的开销
  2. 避免同时运行过多的线程，占用了内存空间
  3. **能统一管理线程**



##### **基本参数**

![image-20200529153050419](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529153050419.png)





- **线程池的扩容**

  - corePoolSize：核心线程数，默认的线程数
  - maxPoolSize：最大线程数，指线程池中最多能拥有的线程数量

  ![image-20200529153715333](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529153715333.png)

  ![image-20200529182441655](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529182441655.png)

  1. **线程池创建初始化后，里面没有任何线程，当有任务需要执行时，才建立线程**
  2. **如果线程数小于corePoolSize，即使其他工作线程处于空闲，也会创建一个新线程来运行新任务**
  3. **如果线程数等于或大于corePoolSize但小于maxPoolSize，则将任务放入等待队列**
  4. **如果等待队列已满，且线程数量小于maxPoolSize，则扩容，创建一个新线程来运行任务**
  5. **如果等待队列已满，且线程数量等于maxPoolSize，则拒绝改任务**





- workQueue：工作队列，当任务超过核心线程数时，剩下的任务会存放再工作队列中
  - **种类**
    1. **交换队列：`SynchronousQueue`长度为0，会直接新建一个线程执行任务，不会有任务等待，直接扩容**
    2. **无界队列：`LinkedBlockingQueue`长度无限，不会扩容，有风险如果任务过多会导致内存溢出**
    3. **有界队列：`ArrayBlockingQueue`长度有限，超过长度才扩容**
    4. **延时队列：`DelayedWorkQueue`长度无限，能设置参数控制延时执行任务**





- KeepAliveTime：线程空闲时存活的时间
  - 对于超过corePoolSize的线程，如果空闲时间超过KeepAliveTime则会被收回





- **threadFactory：线程工厂，用于制造线程，一般使用默认的线程工厂**
  - 如果要制造具有特殊功能的线程，可以设置不同线程工厂的参数，来控制新建线程的优先级、名字等属性



- Handler：拒绝策略
  - 拒绝时机
    - 线程已经超过最大线程数，已经饱和
    - Executor已经关闭，提交新任务会被拒绝
  - 种类
    - AbortPolicy：产生异常，表示提交任务失败
    - DiscardPolicy：抛弃此提交的任务，但是不进行任何提示
    - DiscardOldestPolicy：抛弃等待队列中时间最长的任务，执行新任务
    - CallerRunsPolicy：让传递新任务的线程去执行这个任务





-----



##### 用法

###### **自动创造线程池**

| 线程池名称          | 核心线程数 | 最大线程数 | 线程存活时间 | 工作队列种类 |
| ------------------- | ---------- | ---------- | ------------ | ------------ |
| FixedThreadPool     | 同指定参数 | 同指定参数 | 0            | 无界队列     |
| CachedThreadPool    | 0          | 最大值     | 60秒         | 交换队列     |
| SingleThreadPool    | 1          | 1          | 0            | 无界队列     |
| ScheduledThreadPool | 同指定参数 | 最大值     | 10毫秒       | 延时队列     |
| WorkStealingPool    |            |            |              |              |





- **WorkStealingPool**

  - **概念：抢占线程池，这个是jdk1.8后引入的，是一个并行的线程池**

  - 原理：Fork/Join框架

  - 构造函数：`WorkStealingPool(int number)`：参数是指同时并行的线程数，默认电脑CPU线程数

  - 特点

    1. 抢占式工作，谁抢占到CPU谁就工作，所以执行任务的顺序是无序的
    2. 任务窃取，当一个线程执行完自己的任务后，会帮助还在执行任务的线程完成任务

    ```
    假如我们需要做一个比较大的任务，可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。比如A线程负责处理A队列里的任务。但是，有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是他就去其他线程的队列里窃取一个任务来执行。而在这时他们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。
    ```

    ![image-20200529181829522](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200529181829522.png)





- **特点**

  - **FixedThreadPool：固定线程线程池，永远不会扩容，且任务数是无限的**
  - **CachedThreadPool：可缓存线程池，可以回收空闲的线程**
  - **SingleThreadPool：单线程线程池，只有一个线程**
  - **ScheduledThreadPool：定时线程池，支持定期或者周期执行任务**

  ```java
  public class ScheduledThreadPoolTest {
      public static void main(String[] args) {
          ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(10);
          //延迟5秒执行这个任务
        	threadPool.schedule(new Task(), 5, TimeUnit.SECONDS);
          //第一次执行任务延迟1秒，接下来每3秒执行一次
        	threadPool.scheduleAtFixedRate(new Task(), 1, 3, TimeUnit.SECONDS);
      }
  }
  ```



-----



###### **手动创造线程池**

- 优点：自动创造的线程池属性已经写死，不一定能满足我们的业务需求，我们可以自己创造一个线程池，灵活的满足业务需要



- **线程数量的选择**

  - CPU密集型：运用CPU较多的任务，比如计算Hash、加密等

    - 数量：CPU线程的1~2倍

    

  - 耗时IO型：多进行文件的操作，比如读写数据库、网络读写等

    - 数量：一般是CPU线程的许多倍
    - 公式：线程数 = CPU线程数 * （1 + 平均等待时间 / 平均工作时间）



- 步骤
  1. 创建一个类，继承ThreadPoolExecutor，重写方法
  2. 自己自定义新的功能



-----



###### Executor家族辨析

> - 详情：https://blog.csdn.net/weixin_40304387/article/details/80508236?ops_request_misc=&request_id=&biz_id=102&utm_term=executor%E4%B8%8Eexecutors&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-80508236

![img](https://img-blog.csdn.net/20180530120605822)



- **说明：线程池是最下面的ThreadPoolExecutor，它的上面那些父类父接口实现了线程池的一些操作**
  - **Executor：定义了执行方法`execute`**
  - **ExecutorService：定义了线程池停止方法和一些关于任务的方法**
  - **AbstractExecutorService：重写了他父接口的方法**
  - **ScheduledExecutorService：涉及到周期定期线程池**
  - **ThreadPoolExecutor：线程池**



- **Executors：一个工具类，里面主要用于提供线程池相关的操作，用于创建线程池（返回ThreadPoolExecutor）**



-----



###### 使用方法

- **构造方法：使用Executors工具类创建，用ExecutorService接收**

  ```java
  //以FixedThreadPool为例
  ExecutorService executorService = Executors.newSingleThreadExecutor();
  ```



- **执行方法：在ExecutorService类中**

  - `void execute(Runnable var)`：执行任务

  - `Future<?> submit(Runnable task)`：执行任务，有返回值

    > - 两者区别详情：https://blog.csdn.net/cpf2016/article/details/50150205?utm_source=copy

  - 定时线程池的执行方法比较特殊，见上



- **停止方法：在ExecutorService类中**
  - **`void shutdown()`：关闭了线程池，不再接收新的任务，但仍会执行正在运行和在等待队列中的任务**
  - **`List<Runnable> shutdownNow()`：暴力方法，正在进行的线程会被中断，等待队列的任务会被移除并封装成一个List返回**
  - `boolean isShutdown()`：判断线程池是否进入shutdown状态
  - `boolean isTerminated()`：判断线程池释放完全停止
  - `boolean awaitTermination(long var1, TimeUnit unit)`：判断在指定时间里线程池是否完全停止



- **钩子方法：有点类似于监听器，在ThreadPoolExecutor类中**

  - **`void beforeExecute(Thread t, Runnable r)` ：线程池执行任务前**
  - **`void afterExecute(Runnable r, Throwable t)`：线程池执行任务后**
  - **`void terminated()`：线程池停止后**

  ```java
  public class PauseableThreadPool extends ThreadPoolExecutor {
  
      private final ReentrantLock lock = new ReentrantLock();
      private Condition unpaused = lock.newCondition();
      private boolean isPaused;
  
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit,
              BlockingQueue<Runnable> workQueue) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
      }
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit, BlockingQueue<Runnable> workQueue,
              ThreadFactory threadFactory) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
      }
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit, BlockingQueue<Runnable> workQueue,
              RejectedExecutionHandler handler) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, handler);
      }
  
      public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime,
              TimeUnit unit, BlockingQueue<Runnable> workQueue,
              ThreadFactory threadFactory, RejectedExecutionHandler handler) {
          super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory,
                  handler);
      }
  
      //钩子方法
      @Override
      protected void beforeExecute(Thread t, Runnable r) {
          super.beforeExecute(t, r);
          lock.lock();
          try {
              while (isPaused) {
                  unpaused.await();
              }
          } catch (InterruptedException e) {
              e.printStackTrace();
          } finally {
              lock.unlock();
          }
      }
  
      private void pause() {
          lock.lock();
          try {
              isPaused = true;
          } finally {
              lock.unlock();
          }
      }
  
      public void resume() {
          lock.lock();
          try {
              isPaused = false;
              unpaused.signalAll();
          } finally {
              lock.unlock();
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
          PauseableThreadPool pauseableThreadPool = new PauseableThreadPool(10, 20, 10l,
                  TimeUnit.SECONDS, new LinkedBlockingQueue<>());
          Runnable runnable = new Runnable() {
              @Override
              public void run() {
                  System.out.println("我被执行");
                  try {
                      Thread.sleep(10);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          };
          for (int i = 0; i < 10000; i++) {
              pauseableThreadPool.execute(runnable);
          }
          Thread.sleep(1500);
          pauseableThreadPool.pause();
          System.out.println("线程池被暂停了");
          Thread.sleep(1500);
          pauseableThreadPool.resume();
          System.out.println("线程池被恢复了");
      }
  }
  ```

  

------



##### 线程池状态

| 状态名     | 状态简介                                                     |
| ---------- | ------------------------------------------------------------ |
| RUNNING    | 运行状态，接受任务并排队处理任务                             |
| SHUTDOWN   | 不接受新任务，但处理排队任务                                 |
| STOP       | 不接受新任务，不处理排队任务，中断正在进行的任务             |
| TIDYING    | 整洁状态，所有任务都终止，且排队队列为空，会运行terminated()钩子方法 |
| TERMINATED | terminated()钩子方法运行结束                                 |



------



##### 使用注意

- 避免任务堆积
  - 如果使用无界队列，任务堆积会导致OOM问题



- 避免线程数过多
  - 线程数过多，会占用过的的内存



- 避免线程泄露
  - 线程使用完后没有被回收，会占用大量内存





------



### Threadlocal

- **概念：线程本地变量，这个变量是线程本地独有的，与其他的线程互不干扰**



#### 原理

- **Thread、Threadlocal、ThreadloaclMap关系**

  - **每个Thread里面有一个ThreadMap，ThreadMap是由许多个Threadlocal与它的值形成键值对组成的Map**

  ![image-20200530200045571](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200530200045571.png)

  



- **方法**
  - **`T initialValue()`：初始化Threadlocal值，默认返回NULL，使用Threadlocal时要重写此方法**
    1. **延时方法，只有当我们在线程中调用`get()`方法才会触发**
    2. **如果第一次调用`get()`方法前已经使用过了`set()`方法，则不会触发这个方法**
    3. **只需初始化一次就够了，除非这个Threadlocal值已经被`remove()`了**
  - `T set(T t)`：设置新值
  - `T get()`：获取Threadlocal的值
  - `void remove()`：移除当前Threadlocal的值



- 方法原理

  - get方法

    - 先获取当前线程的ThreadlocalMap，使用`map.getEntry`方法，把本Threadlocal作为参数传入，取出map中Threadlocal的值
    - 如果这个Threadlocal值为空，则调用setInitialValue方法进行初始化

    ```java
     public T get() {
            Thread t = Thread.currentThread();
         	//获取map
            ThreadLocal.ThreadLocalMap map = this.getMap(t);
         	//map不为空
            if (map != null) {
                ThreadLocal.ThreadLocalMap.Entry e = map.getEntry(this);
                //如果Threadlocal存在则返回这个值
                if (e != null) {
                    T result = e.value;
                    return result;
                }
            }
    		//调用setInitialValue方法
            return this.setInitialValue();
        }
    ```

  - **setInitialValue方法**

    - **调用initialValue方法获取它的值，把它保存到ThreadlocalMap里面去，所以我们要重写这个方法**

    ```java
    private T setInitialValue() {
            T value = this.initialValue();
            Thread t = Thread.currentThread();
            ThreadLocal.ThreadLocalMap map = this.getMap(t);
            if (map != null) {
                map.set(this, value);
            } else {
                this.createMap(t, value);
            }
    
            if (this instanceof TerminatingThreadLocal) {
                TerminatingThreadLocal.register((TerminatingThreadLocal)this);
            }
    
            return value;
        }
    ```

  - set方法

    - 与setInitialValue类似，不过没有调用setInitialValue方法，而是直接设置到ThreadlocalMap里面去

    ```java
    public void set(T value) {
            Thread t = Thread.currentThread();
            ThreadLocal.ThreadLocalMap map = this.getMap(t);
            if (map != null) {
                map.set(this, value);
            } else {
                this.createMap(t, value);
            }
    
        }
    ```

  - initialValue方法

    - 默认返回空值，所以我们要重写

    ```java
    protected T initialValue() {
            return null;
        }
    ```

    



-----



#### 应用

##### 第一种

- **概念：每一个线程需要一个独享的对象，各个线程的对象是不共享的，一般用于各种工具类**

  

- **实例：让每一个线程利用时间类打印时间，总共有1000个线程，每一次的时间都不同**

  

  ![image-20200530112710224](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200530112710224.png)
  

  1. 利用for循环创造1000个线程，且每一个线程都新建一个工具类对象

     - 缺点
       - 1000个线程不停地创造销毁，浪费资源
       - 1000个工具类对象不停地创造销毁，浪费资源

     ```java
     public class ThreadLocalNormalUsage01 {
     
         public static void main(String[] args) throws InterruptedException {
             for (int i = 0; i < 30; i++) {
                 int finalI = i;
                 new Thread(new Runnable() {
                     @Override
                     public void run() {
                         String date = new ThreadLocalNormalUsage01().date(finalI);
                         System.out.println(date);
                     }
                 }).start();
                 Thread.sleep(100);
             }
     
         }
     
         public String date(int seconds) {
             //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
             Date date = new Date(1000 * seconds);
             SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
             return dateFormat.format(date);
         }
     }
     ```

  2. 利用线程池，和静态的工具类

     - 缺点：出现线程安全问题

     ```java
     public class ThreadLocalNormalUsage03 {
     
         public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
         static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
     
         public static void main(String[] args) throws InterruptedException {
             for (int i = 0; i < 1000; i++) {
                 int finalI = i;
                 threadPool.submit(new Runnable() {
                     @Override
                     public void run() {
                         String date = new ThreadLocalNormalUsage03().date(finalI);
                         System.out.println(date);
                     }
                 });
             }
             threadPool.shutdown();
         }
     
         public String date(int seconds) {
             //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
             Date date = new Date(1000 * seconds);
             return dateFormat.format(date);
         }
     }
     ```

  3. 利用线程池，和静态的工具类，工具类加锁

     - 缺点：影响了性能

     ```java
     public class ThreadLocalNormalUsage04 {
     
         public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
         static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
     
         public static void main(String[] args) throws InterruptedException {
             for (int i = 0; i < 1000; i++) {
                 int finalI = i;
                 threadPool.submit(new Runnable() {
                     @Override
                     public void run() {
                         String date = new ThreadLocalNormalUsage04().date(finalI);
                         System.out.println(date);
                     }
                 });
             }
             threadPool.shutdown();
         }
     
         public String date(int seconds) {
             //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
             Date date = new Date(1000 * seconds);
             String s = null;
             synchronized (ThreadLocalNormalUsage04.class) {
                 s = dateFormat.format(date);
             }
             return s;
         }
     }
     ```



- **完美解决方法：利用Threadlocal**

  ```java
  public class ThreadLocalNormalUsage05 {
  
      public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
  
      public static void main(String[] args) throws InterruptedException {
          for (int i = 0; i < 1000; i++) {
              int finalI = i;
              threadPool.submit(new Runnable() {
                  @Override
                  public void run() {
                      String date = new ThreadLocalNormalUsage05().date(finalI);
                      System.out.println(date);
                  }
              });
          }
          threadPool.shutdown();
      }
  
      public String date(int seconds) {
          //参数的单位是毫秒，从1970.1.1 00:00:00 GMT计时
          Date date = new Date(1000 * seconds);
          SimpleDateFormat dateFormat = ThreadSafeFormatter.dateFormatThreadLocal.get();
          return dateFormat.format(date);
      }
  }
  
  
  class ThreadSafeFormatter {
  	//使用了Threadlocal
      public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
          @Override
          protected SimpleDateFormat initialValue() {
              return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          }
      };
  }
  ```



- **总结**
  1. **必须注意，这个独享的对象不是线程间共享的数据，这对象在每个线程之间是没有逻辑联系的，是独立的**
  2. **Threadlocal与线程池可以形成很好的搭配，以上面例子为例，线程池中有10个线程，这10个线程创建了10个属于他们自己的独享对象**



-----



##### 第二种

- **概念：每个线程需要保持全局变量，每个线程里的方法可以直接调用，避免了不断传参的麻烦**



- **实例：模仿web项目，每一个请求相当于一个线程，线程里有多个service，都要从一个全局变量里获得一个参数，但这个参数在不同线程的值是不同的**

  1. 使用一个全局变量

     - 缺点：由于不同线程之间这个值是不同的，所以这样无法实现需求

     ![image-20200530124812198](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200530124812198.png)

     

  2. 使用usermap

     - 缺点：是多线程操作，存在线程安全问题

     ![image-20200530120745514](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200530120745514.png)

     

  3. 使用currentMap或者加锁

     - 缺点：性能较差

  

- **完美解决方法：使用Threadlocal**

  ```java
  public class ThreadLocalNormalUsage06 {
      public static void main(String[] args) {
          new Service1().process("");
      }
  }
  
  class Service1 {
      public void process(String name) {
          User user = new User("超哥");
          UserContextHolder.holder.set(user);
          new Service2().process();
      }
  }
  
  class Service2 {
      public void process() {
          User user = UserContextHolder.holder.get();
          ThreadSafeFormatter.dateFormatThreadLocal.get();
          System.out.println("Service2拿到用户名：" + user.name);
          new Service3().process();
      }
  }
  
  class Service3 {
      public void process() {
          User user = UserContextHolder.holder.get();
          System.out.println("Service3拿到用户名：" + user.name);
          UserContextHolder.holder.remove();
      }
  }
  
  class UserContextHolder {
      public static ThreadLocal<User> holder = new ThreadLocal<>();
  }
  
  class User {
      String name;
      public User(String name) {
          this.name = name;
      }
  }
  ```

  

- 总结
  
  - 使用Threadlocal的set和get方法，存储一个全局变量，这个变量是每一个线程独有的，彼此没有联系，避免了不断地传参



-----



#### 使用注意

- 内存泄漏

  - 概念：某个对象不再有用，但却没有被回收，仍然占着内存，可能导致OOM
  - **解决方法：每次使用Threadlocal结束后，调用remove方法**

  ![image-20200530211607498](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200530211607498.png)

![image-20200530211858359](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200530211858359.png)

-----



### Future 和 Callable

- **概念：Callable是一个任务类，是Runnable的扩展；而Future是用于操作Callable的类**



#### Callable

- Runnable的缺陷
  1. 没有返回值
  2. 不能声明抛出异常，只能处理



- 使用：类似于Runnable

  - 创建一个类实现Callable接口
  - 重写call方法

  ```java
  class demo implements Callable<V>{
      @Override
      public V call() throws Exception {
          .......
          return v;
      }
  }   
  ```

  



-----



#### Future

- **概念：Future可以理解成是一个辅助类，里面有许多方法，可以用于操作Callable，同时也是一个储存器，存储了Callable任务的结果**



- **方法**

  - **`V get()`：获取任务的结果**

    - **任务正常完成：返回任务的结果**
    - **任务尚未完成：阻塞当前线程直至任务完成并返回结果**
    - **任务执行过程出现异常：会抛出`ExecutionException`，不论call方法产生的异常是什么**
    - **任务被取消：会抛出`CancellationException`**

    

  - **`V get(long var1, TimeUnit unit)`：获取任务结果，参数是设置阻塞时间**

    - **任务超时：会抛出`TimeoutException`**

    

  - **`boolean cancel(boolean flag)`：取消任务的执行**

    - **任务尚未开始，取消该任务，返回true**
  - **任务已完成或者已取消，返回false**
    - **任务正在进行**
      - **参数为true，会中断这个任务**
      - **参数为false，不会中断这个任务**
  
  
  
  - **`boolean isCancelled()`：判断任务是否被取消**
  
  
  
  - **`boolean isDone()`：判断任务是否执行完毕**
    - **注意：执行完毕不代表任务成功执行，任务失败或者中断或者抛出异常也会返回true**





- 实例

  ```java
  /**
   * 描述：     演示批量提交任务时，用List来批量接收结果
   */
  public class MultiFutures {
  
      public static void main(String[] args) throws InterruptedException {
          ExecutorService service = Executors.newFixedThreadPool(20);
          ArrayList<Future> futures = new ArrayList<>();
          for (int i = 0; i < 20; i++) {
              Future<Integer> future = service.submit(new CallableTask());
              futures.add(future);
          }
          Thread.sleep(5000);
          for (int i = 0; i < 20; i++) {
              Future<Integer> future = futures.get(i);
              try {
                  Integer integer = future.get();
                  System.out.println(integer);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } catch (ExecutionException e) {
                  e.printStackTrace();
              }
          }
      }
  
      static class CallableTask implements Callable<Integer> {
  
          @Override
          public Integer call() throws Exception {
              Thread.sleep(3000);
              return new Random().nextInt();
          }
      }
  }
  ```

  





- 细节

  1. 当`call()`方法里面有异常时，不会立即抛出，而是等到`get()`调用时才会抛出

     ```java
     /**
     描述：演示get方法过程中抛出异常，for循环为了演示抛出Exception的时机：并不是说一产生异常就抛出，		 直到我们get执行时，才会抛出。
      */
     public class GetException {
     
         public static void main(String[] args) {
             ExecutorService service = Executors.newFixedThreadPool(20);
             Future<Integer> future = service.submit(new CallableTask());
     
     
             try {
                 for (int i = 0; i < 5; i++) {
                     System.out.println(i);
                     Thread.sleep(500);
                 }
                 future.get();
             } catch (InterruptedException e) {
                 e.printStackTrace();
                 System.out.println("InterruptedException异常");
             } catch (ExecutionException e) {
                 e.printStackTrace();
                 System.out.println("ExecutionException异常");
             }
         }
     
     
         static class CallableTask implements Callable<Integer> {
     
             @Override
             public Integer call() throws Exception {
                 throw new IllegalArgumentException("Callable抛出异常");
             }
         }
     }
     ```

  2. 使用Future时，要对各种异常进行处理

     ```java
     public class Timeout {
     
         private static final Ad DEFAULT_AD = new Ad("无网络时候的默认广告");
         private static final ExecutorService exec = Executors.newFixedThreadPool(10);
     
         static class Ad {
     
             String name;
     
             public Ad(String name) {
                 this.name = name;
             }
     
             @Override
             public String toString() {
                 return "Ad{" +
                         "name='" + name + '\'' +
                         '}';
             }
         }
     
     
         static class FetchAdTask implements Callable<Ad> {
     
             @Override
             public Ad call() throws Exception {
                 try {
                     Thread.sleep(3000);
                 //处理中断异常
                 } catch (InterruptedException e) {
                     System.out.println("sleep期间被中断了");
                     return new Ad("被中断时候的默认广告");
                 }
                 return new Ad("旅游订票哪家强？找某程");
             }
         }
     
     
         public void printAd() {
             Future<Ad> f = exec.submit(new FetchAdTask());
             Ad ad;
             try {
                 ad = f.get(2000, TimeUnit.MILLISECONDS);
             //处理中断异常
             } catch (InterruptedException e) {
                 ad = new Ad("被中断时候的默认广告");
             //处理错误异常    
             } catch (ExecutionException e) {
                 ad = new Ad("异常时候的默认广告");
             //处理超时异常
             } catch (TimeoutException e) {
                 ad = new Ad("超时时候的默认广告");
             }
             exec.shutdown();
             System.out.println(ad);
         }
     
         public static void main(String[] args) {
             Timeout timeout = new Timeout();
             timeout.printAd();
         }
     }
     ```

  3. **Future被cancel后，无法在继续调用get获得结果，它的生命周期是无法回退的**

     ```java
     class Timeout {
     
         private static final Ad DEFAULT_AD = new Ad("无网络时候的默认广告");
         private static final ExecutorService exec = Executors.newFixedThreadPool(10);
     
         static class Ad {
     
             String name;
     
             public Ad(String name) {
                 this.name = name;
             }
     
             @Override
             public String toString() {
                 return "Ad{" +
                         "name='" + name + '\'' +
                         '}';
             }
         }
     
     
         static class FetchAdTask implements Callable<Ad> {
     
             @Override
             public Ad call() throws Exception {
                 try {
                     Thread.sleep(3000);
                 } catch (InterruptedException e) {
                     System.out.println("sleep期间被中断了");
                 }
                 return new Ad("旅游订票哪家强？找某程");
             }
         }
     
     
         public void printAd() {
             Future<Ad> f = exec.submit(new FetchAdTask());
             Ad ad;
             try {
                 ad = f.get(2000, TimeUnit.MILLISECONDS);
             } catch (InterruptedException e) {
                 ad = new Ad("被中断时候的默认广告");
             } catch (ExecutionException e) {
                 ad = new Ad("异常时候的默认广告");
             } catch (TimeoutException e) {
                 System.out.println("超时，未获取到广告");
                 boolean cancel = f.cancel(false);
                 try {
                     Thread.sleep(2000);
                     //再次重新调用get会抛出异常
                     ad = f.get();
                 } catch (InterruptedException ex) {
                     ex.printStackTrace();
                     ad=null;
                 } catch (ExecutionException ex) {
                     ex.printStackTrace();
                     ad=null;
                 }
                 System.out.println("cancel的结果：" + cancel);
             }
             exec.shutdown();
             System.out.println(ad);
         }
     
         public static void main(String[] args) {
             Timeout timeout = new Timeout();
             timeout.printAd();
         }
     }
     ```

     

-----



#### FutureTask

- 概念：FutureTask是一个包装器，他同时实现了Future和Runnable，可以实现Callable转换为Runnable，**所以既可以作为Runnable被线程执行，也能作为Future获取任务结果**

  ![image-20200531093657193](C:\Users\86183\AppData\Roaming\Typora\typora-user-images\image-20200531093657193.png)



- 使用
  1. 把一个Callable对象作为参数，生成FutureTask对象
  2. 把这个FutureTask对象作为Runnable，使用线程去执行
  3. 用FutureTask获取任务结果





- 实例

  ```java
  /**
   * 描述：     演示FutureTask的用法
   */
  public class FutureTaskDemo {
  
      public static void main(String[] args) {
          Task task = new Task();
          FutureTask<Integer> integerFutureTask = new FutureTask<>(task);
          ExecutorService service = Executors.newCachedThreadPool();
          service.submit(integerFutureTask);
  
          try {
              System.out.println("task运行结果："+integerFutureTask.get());
  
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (ExecutionException e) {
              e.printStackTrace();
          }
      }
  }
  
  class Task implements Callable<Integer> {
  
      @Override
      public Integer call() throws Exception {
          System.out.println("子线程正在计算");
          Thread.sleep(3000);
          int sum = 0;
          for (int i = 0; i < 100; i++) {
              sum += i;
          }
          return sum;
      }
  }
  ```

  