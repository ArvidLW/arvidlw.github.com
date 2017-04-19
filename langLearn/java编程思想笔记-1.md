# java编程思想笔记-1



[TOC]





# Java classpath与jar

***

javac是在当前目录下查找类文件。
CLASSPATH是用来指定.class路径的，不是用来指定.java文件的路径的。
设置classpath跟编译无关.(java程序编译,是设置path).classpath是和程序运行有关。

 反正JVM如果在当前路径下找不到的话会到classpath下面找。
没有使用 -classpath / -cp 指定CLASSPATH时；运行时默认把当前路径设为CLASSPATH

现在安装JDK，根本不需要设置CLASSPATH环境变量。因为最初的JDK自己找不到tools.jar等几个文件，才需要设置。现在已经没有这个问题了。
java -cp .;c:\classes\myClass.jar;d:\classes\*.jar packname.mainclassname 
jar包文件的运行
java -cp HelloWorld.jar org.test.HelloWorld（正确）
java -jar HelloWorld.jar org.test.HelloWorld（错误）

classpath指定程序编译时可以用的包，jar文件等，编译时会到相应的路径中找由import导入的包和类，检查是否存在这些包。
Jar包可以独立运行，但要指定main-class，也可以由manifest指定
Eclispse导入jar,那么在编译的时候就会用到将这些写入编译运行的命令中。
Package：每个java文件下只能有一个package声明

Java核心技术第八版第4.8类路径，由于运行库文件（rt.jar和在jre/lib/jre/lib/ext目录下的一些其他的jar文件）会被自动地搜索，所以不必将它们显式地列在类路径中。

Javac编译器总是在当前的目录中查找文件，但java虚拟机公在类路径中有”.”目录时个才查看当前目录。如果没有设置类路径，那也并不会产生什么问题，默认的类路径包含“.”目录。然而如果设置了类路径却忘记了包含“.”目录，则程序仍然可以通过编译，但不能运行。

Import仅仅可以导入其他包中的公有类。

Linux:

Java -classpath /home/user/classdir:.:/home/user/archives/archive.jar myprog.java

或者用分号代替冒号
是否是要搜索整个目录下归档jar文件，应该要搜索吧，试一下就知道了
JAR 文件仅用于 .class 文件和类路径，不用于 .java 文件和源路径。
需要向类路径添加的只是 JAR 文件本身，而不是包含 JAR 文件的目录（从实质上讲，JAR 文件可以充当包含编译后的 .class 文件的一种目录）
那意思是查不查到jar呢，还是不查找

# 包名与类名

***

包名小写：一般由互联网域名反写。

类名：单词首字母大写。

方法名：首字母小写，其它单词开头大写。

常量：大写并用下划线分割单词。

## main

​	Main()是Java应用程序的入口方法，该方法必须为public static void类型，方法必须接收一个字符串数组的参数，根据习惯字符串数组名可以自己设定，不过一般为args，不能有返回值。

​	JVM在运行这个Java应用程序时，首先会调用这main方法，调用时不实例化这个类的对象，而是通过类名直接调用因此需要限制为public static.

**Eg:******

public class HelloWorld { 

    public static void main(String args[]) { 

        System.out.println("Hello World!"); 

    } 

}

# 初始化与清理

***

### **初始化过程**

1. 任何其他事物发生之前，将分配给对象的存储空间初始化为二进制的零。

2. 调用基类构造器。

3. 按照声明顺序调用成员的初始化方法。

4. 调用导出类的构造器主体。

   ​

   > **（如果基类的构造方法中调用了子类会覆盖的方法，那么在子类进行调用时，会出现未初始化完成但基类调用子类中覆盖的方法从而导致输出不正确的情况）**




### **构造器与重载方法**

1. 基本类型重载的类型自动提升：byte->short->int->long->float->double，还有char->int。


2. 重载方法的区分：参数个数，参数类型，以及参数顺序。

3. 如果你已经定义了一个构造器，编译器就不会帮你自动创建默认构造器。

4. 构造器中可以用this最多调用一个构造器，且必须将构造器放在最起始处，否则编译器会报错。同时编译器禁止在其他方法中调用构造器。

5. 可变参数列表function(Obeject... args)：参数传递可以为0到n个

6. 在子类中如果没有默认基类（父类）构造器（即父类不带参数的构造器），则需要自己写构造函数，并用super()显示调用基类的构造函数。

7. 构造器实际上是static方法，只不过该static声明是隐式的。

   ​

### **垃圾回收**

1. finalize()不能用作通用的清理方法，因为垃圾回收不一定被执行，只有等到相应存储空间用完时才会释放对象占用空间。

2. java并未提供析构函数或相似的概念，要做类似的清理工作必须自己手动创建一个执行清理工作的普通方法，并明确的调用它们。

3. java虚拟机采用一种自适应的垃圾回收技术：自适应的，分代的，停止-复制，标记-清扫。其中自适应是指停止-复制与标记-清扫自动切换，分代的是指内存分配以块为单位，其引用代数表明其存活，对象较大会占用单独的块，它不会被复制，而是增加引用代数。

   ​

### **静态块与初始化**

1. static方法的内部不能调用非静态方法。

2. java类成员变量会自动初始化，基本类型都会赋给相应初值，如果是对象引用则赋为null。

3. java可以在定类成员变量时直接为其赋值，而C++不可以。

4. 初始化顺序，变量定义的先后顺序决定了初始化顺序，但先初始化变量，再初始化方法。另外，**先初始化静态对象，再初始化静态方法。**

5. 静态块：static {}，非静态块（实例初始化）：{}。

   ​

### **数组初始化**

1. 可以用new创建基本类型数组，eg: int[] a=new int[4]，但是不能用new创建基本类型数据。数组中元素会自动初始化为空值（对于数字和字符，就是0，对于布尔型，是false）。
2. 数组是一个非基本类型对象，所以如果in[] a; 那么此时a为null；非基本类型数组（即数组元素为非基本类型），那么其中的元素就为一个引用，初始化为null。如果为基本类型数组（即数组元素为基本类型），那么初始化为0或者false。
3. 数组初始化列表的最后一个逗号是可选的。eg: int[] a={1,2,3,}与{1,2,3}均可。




### **名称屏蔽**

1. 如果Java的基类拥有某个已被多次重载的方法名称，那么在导出类中重新定义该方法名称并不会屏蔽其在基类中的任何版本（这一点与C++不同）。可以用@override注解，来约束避免把重写写成了重载。

2. 重写：方法名称、参数类型、参数个数、参数顺序、返回值都相同才行。否则在C++中就是一个新方法，而屏蔽基类中方法名称相同的其它方法（隐藏方法）；但在Java中有特征不一样就会为重载方法，不会屏蔽基类中原有方法。两种语言都可以用override表明所写的为重写方法，而防止出错。

   ​

# 访问权限控制

***

### **类访问权限**

1. public：公开访问权限。

2. 无修饰（相当于c++中friendly）：包访问权限（即同一个包中的类可以访问，像protected）。

   ​

### **类中成员访问权限**

1. public：公开访问权限。

2. protected：包访问权限；继承访问权限。

3. 无修饰：包访问权限；继承访问权限。同protected。

4. private：不能被外部访问；子类可以继承，但是不能访问。（由1->4开放程度逐渐降低）

   ​

### **类构造方法访问权限**

1. public类的默认构造方法访问权限为public，无修饰（friendly）默认构造方法为无修饰（即protected）。

   ​



### **代理（委托）**

1. Java并没有提供对它的直接支持，但是很多开发工具却支持。代理就是将一个成员对象置于所要构造的新类中（像组合），但同时我们在新类中暴露成员对象的所有方法（像继承）。代理使得代码分离，逻辑清晰。

   ```java
   public class SpaceshipControls{
   	void up(int velocity){}
     	void down(int velocity){}
     	void left(int velocity){}
     	void right(int velocity){}
     	//……
   }
   /**代理，很多开发工具可以生成下面例子*/
   public class SpaceShipDelegation{
     private String name;
     private SpaceShipControls controls=new SpaceShipControls();
     public SpaceShipDelegation(String name){
       this.name=name;
     }
     //代理方法
     public void up(int velocity){
       controls.up(velocity);
     }
     public void down(int velocity){
       controls.down(velocity);
     }
     public void left(int velocity){
       controls.left(velocity);
     }
     public void right(int velocity){
       controls.right(velocity);
     }
     //……
   ```




# 关键字

***

### **final**

1. 空白final，java允许先申明非静态空白final，但在使用前必须对它赋值，而且必须是在构造方法中，或者**初始化块**中{}，否则会报错。但对于static final 则不能这样。

2. final参数：无法在方法中更改参数引用所指向的对象，即其引用地址。

3. final方法：将方法锁定，防止任何继承类修改含义，并且不会被覆盖，如果在子类试图**覆盖final方法会报错**。

4. 所有private方法都隐式地指定为final的，但是private方法是隐藏在类中的，**不属于接口的一部分**，所以不会被覆盖，如果子类试图覆盖该方法的话，它只会在子类中创建一个新的同名的方法，而且不会报错或出错。

5. final类静止继承，final类中所有方法**隐式指定为final** 。

6. final修饰类，表示这个类不能被继承。

7. final修饰变量，不能是类成员变量还是方法中的局部变量，表示该变量值不能改变。

8. final修饰方法，表示方法不能被子类重写。

   ​

### **public**

1. main方法必须是public而且是静态的。

2. 一个.java文件中，可以有多个类，但至多有一个public类，并且以该类名为文件名，如果无public类，则可以用任意名字命名。

   ​

### **static**

1. 见绑定方式 5。

2. 见调用关系 2。

3. 见静态块与初始化 1。

4. 可以修饰类中的成员变量，但不能修饰方法中定义的变量。

   ​



# 继承与多态

***

**如果一个类没有显示的继承自哪个类，那么它自动继承自Object。**

### **绑定方式**

1. 前期绑定：程序执行前进行绑定，由编译器和连接程序实现。

2. 后期绑定：即动态绑定或运行时绑定.

3. Java中除了static方法和final方法（private方法属于final方法）之外，其他所有的方法都是后期绑定

4. 只有普通方法（除了了static和final）的调用是多态，对于域（定义的变量）这些的访问不是多态,但在实际应用中，通常将这些域设为private或者本来就是可以通过普通方法访问和设置的。

5. 向上转型：对象会丢失没有在基类的新的方法。向下转型：对象本来就是那个类型或现有类型与原有类型之间的类型。即被向上转型后的对象，可以向下转型为原来的类型或者向下转型为与原来类型和现有类型之间的类型。

6. 静态方法不能重写，静态方法是与类关联的，不能实现多态。

   ​

   > **如果子类中试图重写静态方法，那么其实只是在该类中新建了一个与基类一模一样的静态方法，而基类中的相应静态方法便被隐藏了。**




### **调用关系**

1. 子类实例化时，会按照继承层次向上链接找到相应构造器，再从从基类到子类依次调用，首先调用基类构造器。

2. 子类调用其静态方法时，不会对基类构造器进行相应的执行，静态方法可以继承但不能被覆盖。

   ​

### **协变返回类型**

1. 在导出类的被覆盖方法可以返回基类方法的返回类型的某种导出类型。

   ```java
   class Grain{
     public String toString(){ return "Grain";}
   }
   class Wheat extends Grain{
     public String toString(){ return "Wheat";}
   }
   class Mill{
     //
     Grain process(){ return new Grain();}
   }
   class WheatMill extends Mill{
     //重写方法（覆写方法）
     //返回类型为Wheat, Wheat为Grain的导出类。即覆盖方法本来要求方法特征和返回类型一样才行，但这里说明返回类型为基类的导出类型也可以。
     Wheat process(){ return new Wheat();}
   }
   ```




# 接口

***

### **一般接口**

1. 抽象类（普通类与接口之间的一种类）：abstract class, 抽象类可以不包含抽象方法，抽象类不能实例化，包含抽象方法的类必须为抽象类。

2. 接口：interface 纯抽象类，不提供任何具体实体。接口中被定义的方法默认为public且必须为public. 接口中可以包含域，也就是声明定义变量，**但这些域隐式的为static和final的**。对于接口的访问修饰，可以用public和无修饰，相应权限与其它类一样。

3. 一个类至多仅可以继承一个类，但是可以实现多个接口。如果基类与接口，或者接口与接口之间有完全相同（返回类型，签名）的方法，那么不会有什么问题，而且如果基类中如果有接口中方法的实现，那么也可以不用在子类再实现那个接口就能创建对象（创建对象的前提是所有定义都是存在且实现了的）；**如果有签名或者返回类型不同那么，就会报错**。

4. 类实现接口，必须实现里面全部的方法。并且在接口中被 定义的方法必须在实现类中被定义为public，否则会出错（由于接口中所有方法都public，**在实现类中不加修饰词的话默认为protected**，这样访问权限就降低了，所以出错。）

5. 接口，**可以让方法接受接口类型，这样让任何类都可以对该方法进行适配，只需要实现接口就可以**。

6. 接口中定义的域不能是空final，因为这里面的域都是静态的（static），**域是static的，那么在类第一次加载时就要被一次性的初始化，所以不能是空final**，因为空final必须需要在构造函数中初始化，即在实例化的时候进行。而调用staitic域或函数时不须要执行构造函数。**接口中域是public static final的所以可以通过接口+成员变量名访问**。

7. 接口和继承不一样，继承的子类向上转型会丢掉新方法和新域；而接口的类似向上转型的形式，其实是统一标准，实则调用的是接口不用实现类中的方法，相当于c++的虚函数。

   ​

### **接口嵌套**

1. 接口可以嵌套在类或其他接口中使用。嵌套在类中的接口可以是public、protected、private的，**嵌套在接口中的接口默认是public而且只能是public.**

2. public、protected嵌套接口，就像非嵌套接口一样，拥有public和包访问权限，比较常见。private接口，可以被实现为public、private、protected的内部类，但是它只能被自身所在的类中的方法调用，不能被在外部单独调用。

   ​

# 设计模式

***

1. 工厂方法设计模式：在工厂对象上调用的是创建方法，生成接口的某个实现的对象。通过这种方式，代码将完全与接口实现分离，这就可以透明的将某个实现替换成另一个实现。

2. 适配器设计模式：

3. 迭代器设计模式（内部类）：

4. 命令设计模式：

5. 策略设计模式（Java编程思想p443）：

   ​



# 内部类

***

**每个内部类都可以独立的实现一个接口，无论外部类是否实现了某个接口。**

**除了静态内部类，其它内部类不能有静态成员。**

内部类可以向上转型为它的基类和接口，从而隐藏细节。

**内部类（除了匿名内部类）和普通类一样，可以实现多个接口，可以继承类。**

普通内部类**对象**隐式地保存了一个指向它外围类**对象**的引用。

内部类的继承：除了嵌套类，其它内部类对象中都包含对外围类对象的引用，所以在继承时要传递外国类的引用，要记得在构造方法中用ObjectName.super()调用外围类构造方法。

**内部类不能被覆盖，但是可以被明确的继承。见p214《JAVA编程思想》**



### **一般内部类**

1. **内部类对象是外部类对象的一个成员，可以用public、……进行修饰，在拥有外部类对象之前是不可能创建内部类对象**

2. 内部类，可以访问其外围对象的所有成员。此外，**内部类拥有其外围类的所有元素的访问权**（与C++嵌套类不同，C++中只是单纯的名字隐藏机制，与外围对象没有联系，也没有隐含的访问权）。

3. 访问外部实例对象用OuterClassName.this，访问自己：用this。

4. 从其他对象去创建某个内部类的对象，需要用.new语法

5. 指明类对象：OuterClassName.InnerClassName

   ```java
   public class DotNew{
       public class Inner{}
       public static void main(String[] args){
         DotNew dn=new DotNew();
         DotNew.Inner dni=dn.new Inner();//必须使用外部类对象去创建内部类对象
       }
     }
   ```



### **静态内部类（嵌套类）**

1. 由static修饰，可以定义static成员，包含static方法，和实例方法，只能访问外部类的静态成员，**且不能从静态内部类的对象中访问外部类对象的非静态成员**，不能通过OuterClassName.this链到其外围对象。

2. 嵌套类中可以继续嵌套类

3. 嵌套类，可以通过外部类名+点+内部类名+点+静态成员来访问。

4. 可以单独创建**嵌套类对象**，而不需要其**外围类对象**。

5. 一个内部类被嵌套多少层并不重要，它能透明访问它所嵌入的**所有外围类**的所有成员。

   ​



### **局部内部类**

类定义于方法内部

1. 局部类访问，直接在方法中实例化，注意，先定义后访问。
2. 局部内部类不能访问定义它的方法的局部变量，**除非局部变量为final**。
3. 当if中有局部类时，在编译时就已经和其它类一起编译了。
4. 作用域：c/c++不支持if/while等分支循环结构内部作用域对外可见。？？？？
5. **局部内部类的名字在方法外是不可见的。**




### **匿名内部类**

1. 在匿名内部类中使用参数引用为final的外部对象。**（如果未在匿名内部类内使用，而只是用于基类构造方法的重载，则可以不是final，见p198《JAVA编程思想》）**

   ```java
   abstract class Base{
     public Base(int i){
       System.out.println("Base constructor");
     }
     public abstract void f();
   }
   public class AnonymousConstructor{
     public static Base getBase(int i){
       //返回匿名对象，传入参数i,用于基类构造方法
       return new Base(i){
         {System.out.println("Instance initializer");}
         public void f(){
           System.out.println("In anonymous f()");
         }
       }
     }
     public static void main(String[] args){
       Base base=getBase(47);
       base.f();
     }
   }
   /*output:
   Base constructor
   Instance initializer
   In anonymous f()
   */
   ```

2. 匿名内部类用实例初始化块当作它的构造器，匿名无名所以不能重载，但能通过super调用父类方法。

3. **可以扩展类，也可以实现接口，但是不能两者兼备。如果实现接口也只能实现一个接口。**

   ```java
   interface Person{
     public void eat();
   }
   public class Demo{
     public static void main(String[] args){
       //实现接口
       Person p=new Person(){
         //实现方法
         public void eat(){
           System.out.println("eat something");
         }
       }
     }
   }
   ```



# 接口内部的类

***

1. 放到接口中的任何类都是自动为public和static的。
2. 可以在接口内部类中实现外围接口。

# 集合与容器

***

Java容器类类库的用途是“保存对象”，并将其划分为两个不同的概念。

容器不能持有基本类型。

1. collection：一个独立的元素序列，符合某种规则。可以转换为数组
   - List---必须按照插入顺序保存元素（可以有重复元素）
     - ArrayList：插入移除慢
     - LinkedList：顺机访问慢，添加了栈、队列、双端队列的方法。
   - set---不能有重复元素，与collection完全一样的接口（除了TreeSet）
     - HashSet：无序，用了散列
     - TreeSet：比较结果升序排列，用了红-黑树
     - LinkedHashSet：被增加顺序保存，有了链表和散列
   - queue----队列保持先进先出的顺序（可以有重复元素）
2. Map：键值对对象
   - HashMap：无序，Map中查找最快
   - TreeMap：比较结果升序
   - LinkedHashMap：按插入顺序保存，速度与HashMap比较接近。

# 异常

***

如果没有处理异常，那么会当程序退出前，调用printStackTrace()方法，打印异常栈中的信息。

当RuntimeException，没有被捕获而直达main()时，在程序退出前也会调用printStackTrace()方法，打印异常栈中的信息。

try-catch-finally是一种用于运行时检查处理异常的机制

### **异常链**

1.  常常会在捕获一个异常后抛出另外一个异常，并希望把异常原始信息保存下来，这被称为异常链。

2.  在JDK1.4以前，程序员必须自己编写代码来保存原始异常信息

3.  现在所有Throwable子类子构造器中都可以接受一个cause对象作为参数。这个cause就异常原由，代表着原始异常，即使在当前位置创建并抛出行的异常，也可以通过这个cause追踪到异常最初发生的位置。

4.  **只有**三种基本的异常类Error,Exception,RunimeException提供了**带cause参数的构造器**，如果要把**其他的异常**链接起来只能initCause()来设置cause，而不是构造器。

    ```java
                 /**
    * 异常链
    *
    */
    public class ExceptionCause {
       public static void main(String[] args) throws Exception {
    	test1();
       }

       private static void test1() throws Exception{
    	try{
    	    test2();
    	}catch(NullPointerException ex){
    //1	    Exception bussinessEx = new Exception("packag exception");
    //	    bussinessEx.initCause(ex);
    //	    throw bussinessEx;
    	    throw new Exception("packag exception", ex);// 2
    //3	    throw (Exception)ex.fillInStackTrace().initCause(ex);
    /**
       1和2分别通过initCause()和构造器设置cause。
       3的出发点和1 2 一样，当能否运行通过？答案是不能，参考
       http://zy19982004.iteye.com/admin/blogs/1974796 throwable 不能是它自己的 cause。
    */
    	}
       }
       
       private static void test2(){
    	test3();
       }
       
       private static void test3(){
    	throw new  NullPointerException("str is null");
       }
    }
    /*output:
       Exception in thread "main" java.lang.Exception: packag exception  
       at com.jyz.study.jdk.exception.ExceptionCause.test1(ExceptionCause.java:18)  
       at com.jyz.study.jdk.exception.ExceptionCause.main(ExceptionCause.java:11)  
       Caused by: java.lang.NullPointerException: str is null  
       at com.jyz.study.jdk.exception.ExceptionCause.test3(ExceptionCause.java:31)  
       at com.jyz.study.jdk.exception.ExceptionCause.test2(ExceptionCause.java:27)  
       at com.jyz.study.jdk.exception.ExceptionCause.test1(ExceptionCause.java:16)  
       ... 1 more
    */
    ```



### **运行时异常**

1. RuntimeException也叫不受检查异常，这种异常属于错误，将自动捕获，但它可以被手动抛出。在编译时无法判断，在运行时可能会发生的异常，是一种错误，发生后会中止程序运行。
2. 如果RuntimeException没有被捕获而直达main()，那么在程序退出前将调用异常的printStackTrace()方法，打印异常调用栈。
3. 运行时异常也可以被捕获。

### finally

1. 无论try块中的异常是否抛出都可以执行的代码块，常用来做清理和收尾工作。
2. 与return结合使用，使得方法可以从多个点返回，同时可以在保证清理进行。

### **缺陷**

```java
//case1:前一个异常还没处理就抛出下一个异常，导致异常丢失
class VeryImportantException extends Exception{
  public String toString(){
    return "A very important exception!";
  }
}
class HoHumException extends Exception{
  public String toString(){
    return "A trivial exception!";
  }
}
public class LostMessage{
  void f() throws VeryImportantException{
    throw new VeryImportantException();
  }
  void dispose() throws HoHumException{
    throw new HoHumException();
  }
  public static void main(String[] args){
    try{
      LostMessage lm=new LostMessage();
      try{
        lm.f();//抛出重要异常
      }finally{
        lm.dispose();//重要异常还没被处理就抛出另一个不重要的异常了。
      }
    }catch(Exception e){
      System.out.println(e);
    }
  }
}
/*output:
A trivial exception!
*/
```

```java
//case2:从finally中返回也会导致异常丢失。
public class ExceptionSilencer{
  public static void main(String[] args){
    try{
      throw new RuntimeException();//运行时异常，不受检查异常，编译时无法判断，而运行时遇到则会抛出的异常，中止程序运行，是一种错误。
      //or throw new Exception();
    }finally{
      return;
    }
  }
}
/*
可以看到即使抛出了异常也不会有任何输出，一般来说只要抛出了异常，那么就会在程序退出前打印异常栈。
*/
```


### **异常限制**

1. 当覆盖方法时，只能抛出在基类方法的异常说明里列出的那些异常或异常的导出类。
2. 当接口和父类中有相同的方法但却抛出不同的异常时，以父类方法抛出的异常为准。
3. 派生对象可以匹配其基类的处理程序。

### **用途**

1. 嵌套try-catch-finally语句可以用来清理（即使发生异常）。（见Java编程思想p274）

# String

***

1. 对String进行操作(增删改查等编辑)，编译器会自动构建StringBuilder对象，然后对String进行操作。（每次操作都会构建一个StringBuilder对象）。
2. 在循环体内进行操作，为了提高效率，避免每次都新建StringBuilder对象，最好自己创建一个StringBuilder对象，来操作String，进行增删改查。
3. 当需要改变字符串的内容时，String类的方法都会返回一个新的String对象。如果内容没有发生变化，String的方法只是返回指向原对角的引用而已。
4. **StringName.format()，整型不能格式化为浮点，浮点也不能格式化为整型。这里只是对其值进行格式化的输出，而不是输出为不同类型。**
5. 对于format("%b",x)，当x是boolean则转换为对应的true or false，否则，只要x不为null，则永远为true。String.format()为静态方法（其实内部也是创建了个Formatter对象），也可以创建Formatter对象来操作。

# 类型

***

1. 在Java中，所有的类型转换都是在运行时进行正确检查的。*RTTI*（Run-Time Type Identification)

2. 所有类都是在对其第一次使用时，动态加载到JVM中，static初始化是在类加载时进行的。

3. 构造器是类的静态方法，使用new操作符创建类的新对象也会被当作对类的静态成员引用。

4. 编译时已经知道了所有的类型，所以可以直接进行强制转换，和其它使用，如Class.forName(className)查找某个类的Class类对象。

5. 每编译一个新类就会产生一个Class对象（被保存在一个同名的.class文件中）。Class对象保存class的基本信息，用来创建类的所有的**“常规”**的对象。Java使用Class对象来执行其RTTI.

6. 类对象可以用.getClass方法来获取Class对象引用。

7. 类字面常量来生成对Class对象的引用。className.class。当使用.class来创建对Class对象的引用时，不会自动地初始化该Class对象，初始化被延迟到了对静态方法或者非常数静态域进行首次引用时执行。（初始化有效地实现了尽可能的惰性）

8. static final值是编译期常量，这个值不需要对它所在类进行初始化就可以被读取。只是static或者final则不可以，因为这样的话要对它再进行链接和初始化。

9. 泛化的Class引用：向Class引用添加泛型语法的原因仅仅是为了提供编译期类型检查。

   ```java
   class ClassReferences{
     public static void main(String[] args){
       Class<Integer> intClass=int.class;
       Class<?> intClass2=in.class;
       Class<? extends Number> bounded=int.class;
     }
   }
   ```

# 数组Arrays

***

1. 对象数组保存的是引用，基本类型数组直接保存基本类型值。对象数组被初始化为null，基本类型数组被初始化为0或false。

2. 创建多维数组用花括号与逗号分开，最后一个逗号可要可不要。

3. Arrays.deepToString()，转换多维数组成字符串。

4. 粗糙数组，构成矩阵的每个向量都可以具有任意长度。

5. Arrays.fill()，可以填充数组。

6. 基本计数生成器Generator类，CountingGenerator，利用反射的机制生成数。（java编程思想p447）

7. 复制数组System.arraycopy()，复制数组比用for循环复制要快很多，但不会执行自动包装与自动拆包，对对象的复制为浅复制，只是复制对象的引用，而不是对对象本身拷贝。

8. 随机数的用法：

   ```java
   Random r=new Random(47);
   r.nextBoolean();
   ```

9. 数组与泛型不能很好的结合，不能实例化具有参数化类型的数组。eg:

   ```java
   Peel<Banana>[] peels=new Peel<Banana>[10];//Illegal
   ```

# 列表List

***

1. Arrays.asList(arr)由数组作为的List，无法增加删除元素。

# 泛型

***

**类型擦除使得泛型不适用于类或方法内部。**

### **伪泛型**

java的泛型是**伪泛型**，因为在编译期间所有的**泛型信息**都会**被擦除掉**。

正确理解泛型概念的首要前提是理解**类型擦出**（type erasure）。Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为**类型擦除**。

如在代码中定义的List<object>和List<String>等类型，在编译后都会编程List。JVM看到的只是List，而由泛型附加的类型信息对JVM来说是不可见的。Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法避免在运行时刻出现类型转换异常的情况。**类型擦除也是Java的泛型实现方法与C++模版机制实现方式之间的重要区别**。

例1：证明java泛型的类型擦除

```java
public class Test4 {  
    public static void main(String[] args) {  
        ArrayList<String> arrayList1=new ArrayList<String>();  
        arrayList1.add("abc");  
        ArrayList<Integer> arrayList2=new ArrayList<Integer>();  
        arrayList2.add(123);  
        System.out.println(arrayList1.getClass()==arrayList2.getClass());  
    }  
} 
/**output:
true
*/
//说明String与Integer信息被擦除掉了，只留下原始类型。
```

例2：反射调用

```java
public class Test4 {  
    public static void main(String[] args) throws IllegalArgumentException, SecurityException, IllegalAccessException, InvocationTargetException, NoSuchMethodException {  
        ArrayList<Integer> arrayList3=new ArrayList<Integer>();  
        arrayList3.add(1);//这样调用add方法只能存储整形，因为泛型类型的实例为Integer  
        arrayList3.getClass().getMethod("add", Object.class).invoke(arrayList3, "asd");  
        for (int i=0;i<arrayList3.size();i++) {  
            System.out.println(arrayList3.get(i));  
        }  
    }  
  //在程序中定义了一个ArrayList泛型类型实例化为Integer的对象，如果直接调用add方法，那么只能存储整形的数据。不过当我们利用反射调用add方法的时候，却可以存储字符串。这说明了Integer泛型实例在编译之后被擦除了，只保留了原始类型。
```



### **类型擦除后保留原始类型**

在上面，两次提到了原始类型，什么是原始类型？原始类型（raw type）就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型。无论何时定义一个泛型类型，相应的原始类型都会被自动地提供。类型变量被擦除（crased），并使用其**限定类型（无限定的变量用Object）替换**。

例3：

```java
class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T  value) {  
        this.value = value;  
    }  
}  
```

Pair<T>的原始类型为：

```java
class Pair {  
    private Object value;  
    public Object getValue() {  
        return value;  
    }  
    public void setValue(Object  value) {  
        this.value = value;  
    }  
}  
```

因为在Pair<T>中，T是一个无限定的类型变量，所以用Object替换。其结果就是一个普通的类，如同泛型加入java变成语言之前已经实现的那样。在程序中可以包含不同类型的Pair，如Pair<String>或Pair<Integer>，但是，擦除类型后它们就成为**原始的Pair类型**了，**原始类型都是Object**。从上面的那个例2中，我们也可以明白ArrayList<Integer>被擦除类型后，原始类型也变成了Object，所以通过反射我们就可以存储字符串了。

如果类型变量有限定，那么原始类型就用第一个边界的类型变量来替换。

比如Pair这样声明：

```java
public class Pair<T extends Comparable& Serializable> {  }
```

那么原始类型就是Comparable
注意：如果Pair这样声明public class Pair<T extends Serializable&Comparable>(注：这里T指实现了Serializable和Comparable接口或类的子类) ，那么原始类型就用Serializable替换，而编译器在必要的时要向Comparable插入强制类型转换。为了提高效率，**应该将标签（tagging）接口（即没有方法的接口）放在边界限定列表的末尾**。



### **原始类型和泛型变量的类型**

在调用泛型方法的时候，可以指定泛型，也可以不指定泛型。

在不指定泛型的情况下，泛型变量的类型为 该方法中的几种类型的同一个父类的最小级，直到Object。

在指定泛型的时候，该方法中的几种类型必须是该泛型实例类型或者其子类。

```java
public class Test2{  
    public static void main(String[] args) {  
        /**不指定泛型的时候*/  
        int i=Test2.add(1, 2); //这两个参数都是Integer，所以T为Integer类型  
        Number f=Test2.add(1, 1.2);//这两个参数一个是Integer，以风格是Float，所以取同一父类的最小级，为Number  
        Object o=Test2.add(1, "asd");//这两个参数一个是Integer，以风格是Float，所以取同一父类的最小级，为Object  
  
                /**指定泛型的时候*/  
        int a=Test2.<Integer>add(1, 2);//指定了Integer，所以只能为Integer类型或者其子类  
        int b=Test2.<Integer>add(1, 2.2);//编译错误，指定了Integer，不能为Float  
        Number c=Test2.<Number>add(1, 2.2); //指定为Number，所以可以为Integer和Float  
    }  
      
    //这是一个简单的泛型方法  
    public static <T> T add(T x,T y){  
        return y;  
    }  
}  
```

其实在泛型类中，不指定泛型的时候，也差不多，只不过这个时候的泛型类型为Object，就比如ArrayList中，如果不指定泛型，那么这个ArrayList中可以放任意类型的对象。例：

```java
public static void main(String[] args) {  
        ArrayList arrayList=new ArrayList();  
        arrayList.add(1);  
        arrayList.add("121");  
        arrayList.add(new Date());  
    }  
```
# 记号

***

@since 1.7 指的是从jdk1.7开始的新特性，以前的就不支持了，在intellij中会有提醒报错，这时你可以在project structure中，设置 language level为相应的版本就可以了。

language level相当于兼容的最低jdk版本。