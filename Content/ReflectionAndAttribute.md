
## 一起帮·源栈-ASP.NET培训：反射与特性

### 1. 反射（Reflection）

- **在运行时动态获取/改变.NET程序的 Assembly、Module 和 Type**

- **先看一个最简单的反射：**

  - ```c#
    using System;
    
    namespace ReflectAndAttribute{
        public class Program{
            public static void Main(string[] args){
                // 运行时
                Console.WriteLine("".GetType().Module);
                // System.Private.CoreLib.dll
                // 编译时
                Console.WriteLine(typeof(Int32).Assembly);
                // System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e
            }
        }
    }
    ```

  - 返回的 Type 对象，实现了 IReflect，学习：

    - Is/Has
    - Get/Set
    - 静态/实例
    - 运算符重载



### 2. Assembly、Module 和 Type

- **简单理解：Assembly > Module > Type > Field/Method/Properties**
- **Assembly 是 .NET 中最小的可执行单元，物理上对应着：项目（开发时），.dll 和 .exe（发布时）文件**
- **其中包括：**
  - manifest（清单）
  - Type metadata：F12 查看已编译类库时看到的东西
  - Microsoft Intermediate Language（MSIL）code：Module
  - 其他 resources：如图片/配置/说明文件等
- **其中，manifest 是 Assembly 区分与 Module 的关键标志（Module 没有manifest）**
  - Manifest 描述了 Assembly 中又多少元素，元素之间如何关联，以及版本号等，Assembly 在运行时就按照 manifest 加载相关的元素
- **所以，Assembly 是：**
  - 自描述的（self-describing）
  - 可重用的（reusable）：dll 可以到处被引用使用
  - 可版本控制的（versionable）
- ![image-20200802153216476](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200802153216476.png)
- 理解：
  - 运行时（Runtime）
  - .NET CLR 和 MSIL 是基础：
    - 一个 Assembly 中可以包含多个不同语言编写的 Module
    - .NET core 是跨平台的
  - Module 中就包含：
    - Types：也即 C# 中类、delegate、enum 等更具体的实现





### 3. 反射-调用方法

- **实用中的反射**

  - 在类（及其成员）和字符串之间转换：
    - ORM：生成 SQL 查询字符串，以及根据查询结果返回对象
    - 配置文件制定类名，动态加载

- **插入：条件编译符的讲解**

  - ```c#
    #define word // 只能在当前文件中生效
    
    /// <summary>
    /// 条件编译符(Condition Complier)：#if ... #elif ... #endif：编译时就选择哪一条语句编译
    /// </summary>
    /// <param name="args"></param>
    public static void Main1(string[] args) {
    
    #if word
                Console.WriteLine("This is XML.");
    #else
                Console.WriteLine("This is Memory.");
    #endif
    
    
    #if XML // 条件编译符：使用时可以在 using 前面写 #define XML 来选择编译那条语句（但是这种写法需要修改源码）：这种写法的作用域为当前文件
            // 或者选择编辑项目属性，在生成 -> 常规 -> 条件编译和符号中填写 XML 或者 Memory（这种写法如果需要更换选择则需要重新编译项目）：这种写法的作用域是整个项目
                IRepoistory<Arcticle> repo = new XArticleRepository<Arcticle>();
    #elif Memory
                IRepoistory<Arcticle> repo = new XRepository<Arcticle>();
    #endif
    
    }
    ```

  - 文件中的声明作用域只能是当前文件，还可以在整个项目范围内声明

    - ![image-20200805173009134](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200805173009134.png)
    - 编译符的问题：一旦编译，无法更改！

- **更灵活的方式：使用（配置）文件制定要使用的类**

  - ```c#
    string path = Path.Join(Environment.CurrentDirectory,"config.txt");
    string classType = File.ReadAllText(path);
    ```

  - 其关键：通过字符串获得类

  - ```c#
    // 如果找不到这个类，就返回 null
    Type target = Type.GetType("ReflectAndAttribute.Student");  // 必须是类的全类名，也即 Assembly 中的名称，一般为名称空间名+类名（注意是类名而不是类文件名，在C#中文件名通常就是类名）
    //Type complied = typeof(Student);
    ```

    - ![image-20200805175714692](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200805175714692.png)
  



- **反射：调用方法**

  - ```c#
    /// <summary>
    /// 反射：调用方法
    /// </summary>
    /// <param name="args"></param>
    public static void Main(string[] args) {
        /*string path = Path.Join(Environment.CurrentDirectory,"config.txt");
                string classType = File.ReadAllText(path);*/
    
        // 如果找不到这个类，就返回 null
        Type target = Type.GetType("ReflectAndAttribute.Student");  // 必须是类的全类名，也即 Assembly 中的名称，一般为名称空间名+类名（注意是类名而不是类文件名，在C#中文件名通常就是类名）
        //Type complied = typeof(Student);
    
    
        // GetConstructor(Type[] types) 获取构造函数信息；Invoke(Object []) 调用构造函数并传入参数
        // 这里表示调用 Student 类中的无参构造函数，返回类的实例
        object objStu = target.GetConstructor(new Type[] { }).Invoke(null);
    
        // 这里表示调用 Student 中一个参数为 string 类型的构造函数，并传入参数 "Zlim"
        object objStu2 = target.GetConstructor(new Type[] { typeof(string) }).Invoke(new object[] { "Zlim"});
        // 类似于：Student s = new Student("Zlim");
    
        //objStu.Greet(); --- 不行，因为 objStu 是 object 类型的，而不是 Student 类型
        // 调用类的成员（方法）：Invoke 第一参数是调用哪个实例的方法，第二个参数是传入调用方法的参数值，如果方法不需要传参则写 null
        target.GetMethod("Greet").Invoke(objStu,null);
    }
    ```

- 上述方法还存在一个问题是无法获得指定类型的实例（只能是 object），不利于进一步的开发，所以通常情况下，我们使用接口（或基类）

  - **Student.cs：**

  - ```c#
    using System;
    using System.Collections.Generic;
    using System.Text;
    
    /**
     * @author zlim
     * @create 2020/8/5 17:33:30
     */
    namespace ReflectAndAttribute {
    
        interface IPerson {
            void Greet();
        }
    
        public class Student : IPerson {
    
            public Student() {
                Console.WriteLine("This is Class Student's Constructor without parameter.");
            }
    
            public Student(string Name) {
                Console.WriteLine($"This is Class Student's Constructor with parameter:{Name}.");
            }
    
            public void Greet() {
                Console.WriteLine("Hi there~");
            }
        }
    
        public class Teacher : IPerson {
            public void Greet() {
                Console.WriteLine("Hello,Students.");
            }
    
        }
    
    }
    
    ```

  - **Program.cs**

  - ```c#
    /// <summary>
    /// 反射：调用方法2
    /// </summary>
    /// <param name="args"></param>
    public static void Main(string[] args) {
        string path = Path.Join(Environment.CurrentDirectory, "config.txt");
        string classType = File.ReadAllText(path);
    
        Type target = Type.GetType(classType);
        IPerson person = null;
        try {
            //person = (IPerson)target.GetConstructor(new Type[] { typeof(string) }).Invoke(new object[] { "ZLim"});
            person = target.GetConstructor(new Type[] { typeof(string) }).Invoke(new object[] { "ZLim" }) as IPerson; 
    
        } catch (Exception e) {
            Console.WriteLine("Something wrong happend .... ");
            throw;
        }
    
        person.Greet();
    }
    ```



- **反射“无视”访问修饰符：**

  - ```c#
    public static void Main(string[] args) {
        Type tStu = typeof(Student);
        // 必须加上 BindingFlags 的两个参数
        Console.WriteLine(tStu.GetField("_age",BindingFlags.NonPublic | BindingFlags.Instance).GetValue(new Student()));// 23
    }
    
    public class Student  {
        private int _age = 23;
    }
    ```

- **反射注意事项：**
  - 反射中大量使用了字符串，error prone
  - 反射比较慢，因为没有编译优化，需要在 metadata 中查找





- **插入：枚举与位运算**

  - 区别于 && 和 || ，在进行逻辑位运算时，& 和 | 不会进行短路操作

  - 还可以在两个二进制值之间进行计算

    - ```c#
      Console.WriteLine(1| 2);
      Console.WriteLine(2 | 4);
      Console.WriteLine(1 & 2);
      Console.WriteLine(2 & 4);
      
      Console.WriteLine(((1 | 2) & 1) == 1);// true
      Console.WriteLine(((1 | 2) & 2) == 2);// true
      ```

    - 以上仅对2的整数次方值有效

  - 演示权限管理系统：记录用户的所有权限

    - ```c#
      public class Program{
          public static void Main(string[] args) {
              Student s = new Student();
              s.AddRole(Role.Student);
              Console.WriteLine(s.Roles);
      
              Teacher t = new Teacher();
              t.AddRole(Role.Teacher);
              t.AddRole(Role.TeamLeader);
              Console.WriteLine(t.Roles);// Teacher, TeamLeader
              t.Roles = t.Roles ^ Role.TeamLeader;// 移除某权限：异或该权限即可
              Console.WriteLine(t.Roles);// Teacher
          }
      }
      
      [Flags]
      public enum Role {
          Student = 1,
          Teacher = 2,
          TeacherAssist = 4,
          TeamLeader = 8,
          DormitoryHead = 16
      }
      
      ```

public class Student  {
      
          public Role Roles { get; set; }
      
          public void AddRole(Role role) {
              Roles = Roles | role;
          }
      }
      
      public class Teacher  {
      
          public Role Roles { get; set; }
      
          public void AddRole(Role role) {
              Roles = Roles | role;
          }
      }
      ```






### 4. 特性

- **就好像给类打的一个“标签”**

  - 示例代码：

  - ```C#
    public static void Main5(string[] args) {
        Student s = new Student();
    
        Attribute attribute = MyOwnAttribute.GetCustomAttribute(typeof(Student), typeof(MyOwnAttribute));
        Console.WriteLine((attribute as MyOwnAttribute).Fee);
        // working in BeiJing
        // 5500
    }
    
    [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method,AllowMultiple =true)]// 表示此特性仅能使用与 class 与 method,并且可以标记多次
    public class MyOwnAttribute:Attribute {
        public MyOwnAttribute() {
            Console.WriteLine("MyOwn is init ... ");
        }
    
        // 构造函数的参数可以在 Attribute 标记时赋值
        public MyOwnAttribute(string city) {
            Console.WriteLine($"working in {city}");
        }
    
        // 属性可以在 Attribute 标记时赋值
        public double Fee { get; set; }
    
        // Attribute 中同样可以有方法
        public void SuperVise() {
    
        }
    
    }
    
    [MyOwn]
    [MyOwn("BeiJing",Fee = 5500)]// 相当于 new MyOwn("BeiJing"){ Fee = 5500 }
    public class Student  {
    
        public Role Roles { get; set; }
    
        public void AddRole(Role role) {
            Roles = Roles | role;
        }
    }
    ```

- **特性本质上是一个继承了 Attribute 的类，自定义的特性必须继承自 Attribute**

- **Attribute 的语法特点：**
  - 可使用与任何“目标”元素，包括但不限于：类、类成员、enum、delegate、Assembly .... （通过 AttributeUsage 特性指定）
  - 一个目标元素可以被附着多个 Attribute
  - 可以像方法一样接收参数
  - 构造函数和方法等不会自动执行，直到显式的获取 .... 
- **自定义的 Attribute 类名建议使用 Attribute 后缀，但是实际使用时可以省略该后缀**
- **常用的 Attribute：**
  - Flags：如上枚举示例中
  - Obsolete：已过期
  - Serializable：可序列化





## 编程英语

![image-20200728015707041](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200728015707041.png)



![image-20200805172557948](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200805172557948.png)





## C# 异步编程

### 1. 线程（Thread）：创建线程

#### 1.1 什么是线程 Thread

- **线程是一个可执行路径，它可以独立于其他线程执行。**
- **每个线程都在操作系统的进程（Process）内执行，而操作系统进程提供了程序运行的独立环境**
- **单线程的应用，在进程的独立环境里只跑一个线程，所以该线程拥有独占权。**
- **多线程应用，单个进程中会跑多个线程，它们会共享当前的执行环境（尤其是内存）**
  - 例如，一个线程在后台读取数据，另一个线程在数据到达后进行展示
  - 这个数据就被称作是共享的状态



#### 1.2 示例

- **代码：**

```c#
class Program
{
    static void Main(string[] args)
    {
        // 开启了一个新的线程 Thread
        Thread t = new Thread(WriteY);
        t.Name = "Y Thread ... ";
        // 运行 WriteY()
        t.Start();

        // 同时在主线程也做一些工作
        for (int i = 0; i < 1000; i++)
        {
            System.Console.Write("x");
        }
    }

    static void WriteY()
    {
        for (int i = 0; i < 1000; i++)
        {
            System.Console.Write("y");
        }
    }
}
```



- **在单核计算机上，操作系统必须为每个线程分配“时间片”（在Windows中通常为20毫秒）来模拟并发，从而导致重复的x和y块**
- **在多核或多处理器计算机上，这两个线程可以真正地并行执行（可能受到计算机上其他活动进行的竞争）。**
  - 在本例中，由于控制台处理并发请求机制的微妙性，您仍让会得到重复的x和y块。

- **输出结果：**

![image-20200726191558333](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726191558333.png)



#### 1.3 线程的一些属性

- **线程一旦开始执行，IsAlive就是 true，线程结束就变成了 false。**
- **线程结束的条件就是：线程构造函数传入的委托结束了执行。**
- **线程一旦结束，就无法再重启。**
- **每一个线程都有 Name 属性，通常用于调试。**
  - 线程 Name 只能设置一次，以后更改会抛出异常

- **静态的 Thread.CurrentThread 属性，会返回当前执行的线程**
- **代码：**

```c#
 class Program
 {
     static void Main(string[] args)
     {
         Thread.CurrentThread.Name = "Main Thread ... ";
         // 开启了一个新的线程 Thread
         Thread t = new Thread(WriteY);
         t.Name = "Y Thread ... ";
         // 运行 WriteY()
         t.Start();

         System.Console.WriteLine(Thread.CurrentThread.Name);
         // 同时在主线程也做一些工作
         for (int i = 0; i < 1000; i++)
         {
             System.Console.Write("x");
         }
     }

     static void WriteY()
     {
         System.Console.WriteLine(Thread.CurrentThread.Name);
         for (int i = 0; i < 1000; i++)
         {
             System.Console.Write("y");
         }
     }
 }
```



- **输出结果：**

![image-20200726192213637](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726192213637.png)





### 2. Thread.Join() & Thread.Sleep()

#### 2.1 Join and Sleep

- **调用 Join 方法，就可以等待另一个线程结束。**
- **代码示例：**

```c#
class Program
    {
        static Thread thread1,thread2;
        
        public static void Main()
        {
            thread1 = new Thread(ThreadProc);
            thread1.Name = "Thread1";
            thread1.Start();

            thread2 = new Thread(ThreadProc);
            thread2.Name = "Thread2";
            thread2.Start();
        }

        public static void ThreadProc()
        {
            System.Console.WriteLine("\nCurrent thread:{0}",Thread.CurrentThread.Name);
            if (Thread.CurrentThread.Name == "Thread1" && thread2.ThreadState != ThreadState.Unstarted)
            {
                thread2.Join();
            }
            Thread.Sleep(2000);
            System.Console.WriteLine("\nCurrent thread:{0}",Thread.CurrentThread.Name);
            System.Console.WriteLine("Thread1:{0}",thread1.ThreadState);
            System.Console.WriteLine("Thread2:{0}\n",thread2.ThreadState);
        }
    }
```



- **输出结果：**

![image-20200726204848254](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726204848254.png)



- **添加超时：**
- **调用 Join 的时候，可以设置一个超时，用毫秒或者 TimeSpan 都可以**
  - 如果返回 true，那就表示线程结束了；如果超时了，就会返回 false。
- **示例代码：**

```c#
class Program
{
    static Thread thread1,thread2;

    public static void Main()
    {
        thread1 = new Thread(ThreadProc);
        thread1.Name = "Thread1";
        thread1.Start();

        thread2 = new Thread(ThreadProc);
        thread2.Name = "Thread2";
        thread2.Start();
    }

    public static void ThreadProc()
    {
        System.Console.WriteLine("\nCurrent thread:{0}",Thread.CurrentThread.Name);
        if (Thread.CurrentThread.Name == "Thread1" && thread2.ThreadState != ThreadState.Unstarted)
        {
            if (thread2.Join(2000))
            {
                System.Console.WriteLine("Thread2 has termminated.");
            }else
            {
                System.Console.WriteLine("The timeout has elapsed and Thread1 will resume.");
            }
        }
        Thread.Sleep(2000);
        System.Console.WriteLine("\nCurrent thread:{0}",Thread.CurrentThread.Name);
        System.Console.WriteLine("Thread1:{0}",thread1.ThreadState);
        System.Console.WriteLine("Thread2:{0}\n",thread2.ThreadState);
    }
}
```



- **输出结果：**

![image-20200726220322200](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726220322200.png)



- **示例代码2：**

```c#
class Program2
{
    // TimeSpan.TimeSpan(int hours,int minutes,int seconds);
    static TimeSpan waitTime = new TimeSpan(0,0,1);

    public static void Main()
    {
        Thread newThread = new Thread(Work);
        newThread.Start();

        if (newThread.Join(waitTime + waitTime))
        {
            System.Console.WriteLine("New Thread terminated.");
        }
        else
        {
            System.Console.WriteLine("Join timed out.");
        }
    }

    static void Work()
    {
        Thread.Sleep(waitTime);
    }
}
```



- **示例代码3：**

```c#
static void Main()
{
    // 也可以使用 TimeSpan 实现相同的效果
    TimeSpan interval = new TimeSpan(0,0,2);
    
    for(int i = 0;i < 5;i++)
    {
        Console.WriteLine("Sleep for 2 seconds.");
        // 程序执行结果就是：每隔2秒输出一次 Sleep for 2 seconds. 这句话
        //Thread.Sleep(2000);
        Thread.Sleep(interval);
    }
    
    Console.WriteLine("Main thread exits.");
}
```



- **Thread.Sleep() 方法会暂停当前的线程，并等待制定的时间。**
- **注意：**
  - **Thread.Sleep(0) 这样调用会导致线程立即放弃本身当前的时间片，自动将CPU移交给其他线程**
  - **Thread.Yiels() 做同样的事情，但是它只会把执行时间片交给同一处理器上的其他线程**
  - **当等待 Sleep 或 Join 的时候，线程处于阻塞的状态。**

- **Sleep(0) 或 Yield 有时在高级性能调试的生产代码中很有用。它也是一个很好的诊断工具，有助于发现线程安全问题：**
  - 如果在代码中的任何地方插入 Thread.Yield() 就破坏的程序，那么你的程序几乎肯定就有 bug。





### 3. 阻塞 Blocking

#### 3.1 阻塞

- **如果线程的执行由于某种原因导致暂定，那么就认为该线程被阻塞了。**

  - 例如在 Sleep 或者通过 Join 等待其他线程结束

- **被阻塞的线程会立即将其处理器的时间片生成给其他线程，从此就不再消耗处理器的时间，直到满足其阻塞条件为止。**

  - 直到满足其阻塞条件为止：是指直到满足这个阻塞条件了，然后CPU处理器的时间片就回到了线程，从此线程就可以继续消耗处理器的时间了，也就是说在满足这个阻塞条件之前，当前这个线程就会干等着，(阻塞就是线程的假死状态)，然后直到满足这个阻塞条件了，这个线程才可以继续干活。

- **可以通过 ThreadState 这个属性来判断线程是否处于被阻塞的状态：**

  - ```c#
    bool blocked = (someThread.ThreadState & ThreadState.WaitSleepJoin) != 0;
    ```



#### 3.2 ThreadState

- **ThreadState 是一个 flag enum，通过按位或的形式，可以合并数据的选项。**
- **ThreadState 的选项：其中每一个选项的数字都是2的，并都使用二进制表示**

![image-20200726224250974](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726224250974.png)

- **但是它的大部分的枚举值都没什么用，下面的代码将 ThraedState 剥离为四个最有用的值之一：==UnStarte、Running、WaitSleepJoin 和 Stopped==**

- **ThreadState 属性可用于诊断的目的，但不适用于同步，因为线程状态可能会在测试 ThreadState 和对该信息进行操作之间发生变化。**

  - ```c#
    public static ThreadState SimpleThreadState(ThreadState ts)
    {
        return ts & (ThreadState.Unstarted | 
                     ThreadState.Stopped | 
                     ThreadState.WaitSleepJoin);
    }
    ```



- **示例代码：**

```c#
class Program
{
    static void Main()
    {
        var state = ThreadState.Unstarted | ThreadState.Stopped | ThreadState.WaitSleepJoin;
        System.Console.WriteLine($"{Convert.ToString((int)state,2)}");
        // UnSarted = 8 = 1000  Stopped = 16 = 10000 WaitSleepJoin = 32 = 100000
        /*
         100000|
          10000|
           1000|
        = 111000    
        */ 
          
    }
}
```



- **输出结果：**

![image-20200726225555840](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726225555840.png)



- **线程可能的状态：**

![image-20200726225847712](C:\360极速浏览器X下载\books\C# 异步编程.assets\image-20200726225847712.png)



#### 3.3 接触阻塞 Unblocking

- **当遇到下列四种情况的时候，就会解除阻塞：**
  - **阻塞条件超时**
  - **操作超时（如果设置超时的话）**
  - **通过 Thread.Interrupt() 进行打断**
  - **通过 Thread.Abort() 进行中止**
- **上下文切换：**
  - 当线程阻塞或接触阻塞时，操作系统将执行上下文切换。这会产生少量开销，通常为 1 或 2 微秒。



#### 3.4 I/O-bound vs Compute-bound（或 CPU-Bound）

- **一个花费大部分时间等待某事发生的操作称为 I/O-bound**
  - I/O绑定操作通常涉及输入或输出，但这不是硬性要求：Thread.Sleep() 也被视为 I/O-bound
- **相反，一个花费大部分时间执行 CPU 密集型工作的操作称为 Compute-bound。**



#### 3.5 阻塞 vs 忙等待（自旋）：Blocking vs Spinning

- **IO-bound 操作的工作方式有两种：**

  - 在当前线程上同步等待：
    - Console.ReadLine()，Thread.Sleep(),Thread.Join() ... 
  - 异步的操作，在稍后操作完成时触发一个回调动作。

- **同步等待的 I/O-bound 操作将大部分时间花在阻塞线程上。**

- **它们也可以周期性的在一个循环里进行“打转（自旋）”**

  - ```c#
    while(DateTime.Now < nextStartTime) // 忙等待
        Thread.Sleep(1000);				// I/O-bound
    
    while(DateTime.Now < nextStartTime)
    ```

- **在忙等待和阻塞方面有一些细微差别。**

  - 首先，如果希望条件很快得到满足（可能在几微秒之内），则短暂自旋可能会很有效，因为它避免了上下文切换的开销与延迟
    - .NET Framework 提供了特殊的方法和类来帮助 SpinLock 和 SpinWait
  - 其次，阻塞也不是零成本。这是因为每个线程在生存周期会占用大约 1 MB的内存，并会给 CLR 和操作系统带来持续的管理开销
    - 因此，在需要处理成百上千并发操作的大量 I/O-bound 程序的上下文中，阻塞可能会很麻烦
    - 所以，此类程序需要使用基于回调的方法，在等待完成时完全撤销其线程。



### 4. 本地、共享状态与线程安全（简介）

#### 4.1 本地 vs 共享的状态

- **Local vs Shared State**

1. **Local 本地独立**

   - CLR 为每个线程分配自己的内存栈（Stack），以便使本地变量保持独立

   - ```c#
     static void Main(){
         new Thread(Go).Start();// 在新下线程上调用 Go()
     
         Go();// 在 main 线程上调用 Go()
     }
     
     static void Go()
     {
         // cycles 是本地变量
         // 在每个线程的内存栈上，都会创建 cycles 独立的副本
         for (int cycles = 0; cycles < 5; cycles++)
         {
             Console.WriteLine('?');
         }
     }
     
     // 结果会输出 10 个 ？
     // 因为每个循环是5次
     ```

2. **Shared 共享**

   - 如果多个线程都引用到同一个对象的实例，那么它们就共享了数据

   - ```c#
     class ThreadTest
     {
         bool _done;// 默认值没写就是false
     
         static void Main()
         {
             ThreadTest tt = new ThreadTest(); // 创建了一个共同的实例
             new Thread(tt.Go).Start();
             tt.Go();
             // 主线程main与新线程都使用了tt这个变量
         }
     
         void Go() // 这是一个实例方法
         {
             if (!_done)
             {
                 _done = true;
                 Console.WriteLine("Done");
             }
         }
     
         // 由于两个线程是在同一个 ThreadTest 实例上调用的 Go()，所以它们共享 _done
         // 结果就是只打印一次 Done
     
     }
     ```
```
     
- 被 Lambda 表达式或匿名委托所捕获的本地变量，会被编译器转化为字段（field），所以也会被共享
   
   - ```c#
     class ThreadTest
     {
         static void Main()
         {
             bool done = false;
     
             ThreadStart action = () => 
             {
                 if (!done)
                 {
                     done = true;
                     Console.WriteLine("Done3");
                 }
             };
     
             new Thread(action).Start();
             action();
          // 结果是只打印一次 Done3 
       }
  }
```

   - 静态字段（field）也会在线程间共享数据
   
   - ```c#
     class ThreadTest2
     {
         static bool _done; // 静态字段在同一应用域下的所有线程中被共享
     
         static void Main()
         {
             new Thread(Go).Start();
             Go();
         }
     
         static void Go() // 这是一个实例方法
         {
             if (!_done)
             {
                 _done = true;
                 Console.WriteLine("Done");
             }
         }
     
         // 由于静态字段在同一应用域下的所有线程中被共享
         // 结果就是只打印一次 Done
     
     }
     ```



#### 4.2 线程安全 Thread Safety

- **前面上述例子就引出了==线程安全==这个关键概念（或者说缺乏线程安全）**

- **也即上述例子的输出实际上是无法确定的：**

  - 有可能（理论上）“Done”会被打印两次

  - 如果交换 Go 方法里语句的顺序，那么“Done”被打印两次的几率会大大增加

  - 因为一个线程可能正在评估 if，而另外一个线程在执行 WriteLine 语句，它还没来得及把 done 设为 true。

  - ```c#
    static void Main()
    {
        bool done = false;
    
        ThreadStart action = () => 
        {
            if (!done)
            {
                Console.WriteLine("Done3");
                Thread.Sleep(100);// 100毫秒
                done = true;
            }
        };
    
        new Thread(action).Start();
        action();
    }
    ```

- **我们应该尽可能的避免使用共享状态**



#### 4.3 锁定与线程安全 简介

- **Locking & Thread Safety**

- **在读取和写入共享数据的时候，通过使用一个互斥锁（*exclusive lock*），就可以修复前面例子的问题。**

- **C# 使用 lock 语句来加锁**

- **当两个线程同时竞争一个锁的时候（锁可以基于任何引用类型对象），一个线程会等待或阻塞，直到锁变成可用状态**

- **在多线程上下文中，以这种方式避免不确定性的代码就叫做==线程安全==。**

- **Lock 不是线程安全的银弹，很容易忘记对字段加锁，lock 也会引用一些问题（死锁）**

  - ```c#
    class ThreadSafe
    {
        static bool _done;
    
        static readonly object _locker = new object();
    
        static void Main()
        {
            new Thread(Go).Start();
            Go();
        }
    
        static void Go()
        {
            lock(_locker)// 锁可以基于任何引用类型的对象
            {
                if (!_done)
                {
                    Console.WriteLine("Done with the lock block.");
                    _done = true;
                }
            }
        }
        // 输出结果是打印一遍 Done with the lock block. 
    }
    ```

  

- **思考：i++ 这种表达式是否是线程安全的？（i 是共享数据）**

  - 不是，因为i++不具备原子性



### 5、 向线程传递数据 & 异常处理

