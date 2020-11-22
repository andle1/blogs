​		Java程序运行时，必须经过编译和运行两个步骤。首先将后缀名为.java的源文件进行编译，最终生成后缀名为.class的字节码文件。然后Java虚拟机将编译好的字节码文件加载到内存（这个过程被称为类加载，是由加载器完成的），然后虚拟机针对加载到内存的java类进行解释执行，显示结果。



## **Java的运行原理**

​			在Java中引入了虚拟机的概念，即在机器和编译程序之间加入了一层抽象的虚拟的机器。这台虚拟的机器在任何平台上都提供给编译程序一个的共同的接口。编译程序只需要面向虚拟机，生成虚拟机能够理解的代码，然后由解释器来将虚拟机代码转换为特定系统的机器码执行。在Java中，这种供虚拟机理解的代码叫做字节码（ByteCode），它不面向任何特定的处理器，只面向虚拟机。每一种平台的解释器是不同的，但是实现的虚拟机是相同的。Java源程序经过编译器编译后变成字节码，字节码由虚拟机解释执行，虚拟机将每一条要执行的字节码送给解释器，解释器将其翻译成特定机器上的机器码，然后在特定的机器上运行。



## **Java代码编译和执行的整个过程**

Java代码编译是由Java源码编译器来完成，流程图如下所示：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B0/pIYBAFrhQ-WAN_-bAAA9GHdpZic508.jpg)



Java字节码的执行是由JVM执行引擎来完成，流程图如下所示：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/AF/o4YBAFrhQ-WAQXyzAABRBYZXIHo024.jpg)



Java代码编译和执行的整个过程包含了以下三个重要的机制：

> ​	Java源码编译机制
>
> ​    类加载机制
>
> ​	类执行机制

####  	1、Java源码编译机制

 	Java 源码编译由以下三个过程组成：（javac –verbose 输出有关编译器正在执行的操作的消息）

 	分析和输入到符号表

 	注解处理

 	语义分析和生成class文件

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B0/pIYBAFrhQ_2AdRgZAADfmz-yPeM816.jpg)

最后生成的class文件由以下部分组成：

 	1、结构信息。包括class文件格式版本号及各部分的数量与大小的信息

 	2、元数据。对应于Java源码中声明与常量的信息。包含类/继承的超类/实现的接口的声明信息、域与方法声明信息和常量池

 	3、方法信息。对应Java源码中语句和表达式对应的信息。包含字节码、异常处理器表、求值栈与局部变量区大小、求值栈的类型记录、调试符号信息



#### 2、类加载机制

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B0/pIYBAFrhRAqAWLo9AACUq8dYUb4981.jpg)

 	1）Bootstrap ClassLoader /启动类加载器

 	$JAVA_HOME中jre/lib/rt.jar里所有的class，由C++实现，不是ClassLoader子类

 	2）Extension ClassLoader/扩展类加载器

 	负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包

 	3）App ClassLoader/ 系统类加载器

 	负责记载classpath中指定的jar包及目录中class

 	4）Custom ClassLoader/用户自定义类加载器（java.lang.ClassLoader的子类）

 	属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader

 	加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap  ClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。

 	类加载双亲委派机制介绍和分析

 	 在这里，需要着重说明的是，JVM在加载类时默认采用的是双亲委派机制。通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。



#### 3、类执行机制

​            JVM是基于栈的体系结构来执行class字节码的。线程创建后，都会产生程序计数器（PC）和栈（Stack），程序计数器存放下一条要执行的指令在方法内的偏移量，栈中存放一个个栈帧，每个栈帧对应着每个方法的每次调用，而栈帧又是有局部变量区和操作数栈两部分组成，局部变量区用于存放方法中的局部变量和参数，操作数栈中用于存放方法执行过程中产生的中间结果。



##### **执行过程如下**

 	1、为main方法创建栈帧：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiJOAVBmzAAAxNN9bTJc820.jpg)

 	

 	局部变量表长度为2，slot0存放参数args，slot1存放局部变量Student s，操作数栈最大深度为5。

 	2、new#7指令，在java堆中创建一个Student对象，并将其引用值放入栈顶。

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiJeAPiWOAABIX8pHwX4028.jpg)

 	3、初始化一个对象（通过实例构造的方式）

 	up指令：复制栈顶的值，然后将复制的结果入栈。

 	bipush 23：将单字节常量值23入栈。

 	ldc #8：将#8这个常量池中的常量即”dqrcsc”取出，并入栈。

 	ldc #9：将#9这个常量池中的常量即”20150723”取出，并入栈。

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiKmAIszDAABTjOyyIWU935.jpg)

 	4、invokespecial #10：调用#10这个常量所代表的方法，即Student.（）这个方法，这步是为了初始化对象s的各项值

 	 《init》（）方法，是编译器将调用父类的《init》（）的语句、构造代码块、实例字段赋值语句，以及自己编写的构造方法中的语句整合在一起生成的一个方法。保证调用父类的《init》（）方法在最开头，自己编写的构造方法语句在最后，而构造代码块及实例字段赋值语句按出现的顺序按序整合到《init》（）方法中。

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiKmAAzW1AAAjUDP86GU315.jpg)

 	注意到Student.《init》（）方法的最大操作数栈深度为3，局部变量表大小为4。

 	此时需注意：从dup到ldc #9这四条指令向栈中添加了4个数据，而Student.（）方法刚好也需要4个参数：

 	public Student（int age， String name， String sid）{

 	super（age，name）;

 	this.sid = sid;

 	}1234567

 	虽然定义中只显式地定义了传入3个参数，而实际上会隐含传入一个当前对象的引用作为第一个参数，所以四个参数依次为this，age，name，sid。

 	上面的4条指令刚好把这四个参数的值依次入栈，进行参数传递，然后调用了Student.《init》（）方法，会创建该方法的栈帧，并入栈。栈帧中的局部变量表的第0到4个slot分别保存着入栈的那四个参数值。

 	创建Studet.《init》（）方法的栈帧：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiLGAYtiHAABS3lRmiU8741.jpg)

Student.《init》（）方法中的字节码指令：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiMSAMpWdAAAyS6hpwr8632.jpg)

 	aload_0：将局部变量表slot0处的引用值入栈

 	aload_1：将局部变量表slot1处的int值入栈

 	aload_2：将局部变量表slot2处的引用值入栈

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiMyAYf4kAABYgjsQXDA328.jpg)



 	invokespecial #1：调用Person.（）方法，同调用Student.过程类似，创建栈帧，将三个参数的值存放到局部变量表等，这里就不画图了……

 	从Person.（）返回之后，用于传参的栈顶的3个值被回收了。

 	aload_0：将slot0处的引用值入栈。

 	aload_3：将slot3处的引用值入栈。

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiNSADm1aAABWt506pmQ354.jpg)

 	putfield #2：将当前栈顶的值”20150723”赋值给0x2222所引用对象的sid字段，然后栈中的两个值出栈。

 	return：返回调用方即main（）方法，当前方法栈帧出栈。

 	重新回到main（）方法中，继续执行下面的字节码指令：

 	astore_1：将当前栈顶引用类型的值赋值给slot1处的局部变量，然后出栈。

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiNyAVoDSAABIq4wQ2JU551.jpg)

 	5、到这儿为止，第一行代码执行完毕，将s返回给局部变量表，执行下边的

 	public static void main（String［］ args）{

 	Student s = new Student（23，“dqrcsc”，“20150723”）;//执行完毕

 	s.study（5，6）;

 	Student.getCnt（）;

 	s.run（）;

 	}1234567891011

 	aload_1：slot1处的引用类型的值入栈

 	iconst_5：将常数5入栈，int型常数只有0-5有对应的iconst_x指令

 	bipush 6：将常数6入栈

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiO2AblSjAABPDDn3-Qo333.jpg)

 	6、开始执行第二行代码，也就是strudy方法

 	invokevirtual #11：调用虚方法study（），这个方法是重写的接口中的方法，需要动态分派，所以使用了invokevirtual指令。

 	创建study（）方法的栈帧：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiOuAVbc2AAAmMzjkfIg237.jpg)

最大栈深度3，局部变量表5

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhigqAVyHzAAB3SCDdLSI949.jpg)

 	方法的java源码：

 	这里写代码片public int study（int a， int b）{

 	int c = 10;

 	int d = 20;

 	return a+b*c-d;

 	}123456789

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiQ-AGVAJAABDgVewG78245.jpg)

 	bipush 10：将10入栈

 	istore_3：将栈顶的10赋值给slot3处的int局部变量，即c，出栈。

 	bipush 20：将20入栈

 	istore 4：将栈顶的20付给slot4处的int局部变量，即d，出栈。

 	上面4条指令，完成对c和d的赋值工作。

 	iload_1、iload_2、iload_3这三条指令将slot1、slot2、slot3这三个局部变量入栈：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiRCAYyE7AABUr_wAovc043.jpg)

imul：将栈顶的两个值出栈，相乘的结果入栈：

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B5/o4YBAFrhiReANihSAABT-PN-Yb4790.jpg)

 	iadd：将当前栈顶的两个值出栈，相加的结果入栈

 	iload 4：将slot4处的int型的局部变量入

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiSmACl8OAABQSp01REc116.jpg)

 	isub：将栈顶两个值出栈，相减结果入栈：

 	ireturn：将当前栈顶的值返回到调用方。

![java程序的执行过程详解](http://file.elecfans.com/web1/M00/4F/B7/pIYBAFrhiTCAC7ggAABSSLFjmgU524.jpg)

 	7、到这儿为止，第二行代码执行完毕，返回值返回给s，执行下边的

 	public static void main（String［］ args）{

 	Student s = new Student（23，“dqrcsc”，“20150723”）;//执行完毕

 	s.study（5，6）;

 	Student.getCnt（）;

 	s.run（）;

 	}1234567891011

 	invokestatic #12 调用静态方法getCnt（）不需要传任何参数

 	pop：getCnt（）方法有返回值，将其出栈

 	aload_1：将slot1处的引用值入栈

 	invokevirtual #13：调用0x2222对象的run（）方法，重写自父类的方法，需要动态分派，所以使用invokevirtual指令

 	return：main（）返回，程序运行结束。