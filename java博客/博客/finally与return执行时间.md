### Java finally语句到底是在return之前还是之后执行？



​		Java中异常捕获机制try...catch...finally块中的finally语句是不是一定会被执行？很多人都说不是，当然他们的回答是正确的，经过我试验，**至少有两种情况下finally语句是不会被执行的：**

> **（1）try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执行，这也说明了finally语句被执行的必要而非充分条件是：相应的try语句一定被执行到。**
>
> **（2）在try块中有System.exit(0);这样的语句，System.exit(0);是终止Java虚拟机JVM的，连JVM都停止了，所有都结束了，当然finally语句也不会被执行到**



Finally语句的执行与return的关系：

> 一、 finally语句是在try的return语句执行之后，return返回之前执行。

```java
package com.it.dao;

public class Test{
    public static void main(String[] args) {
        System.out.println(test1());
    }
    private static int test1() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80;
        }
        catch (Exception e) {
            System.out.println("catch block");
        }
        finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }
        }
        return b;
    }
}
```

结果：

```JAVA
try block
finally block
b>25, b = 100
100
```

​		结果说明return语句已经执行了再去执行finally语句，不过并没有直接返回，而是等finally语句执行完了再返回结果。

如果觉得这个例子还不足以说明这个情况的话，下面再加个例子加强证明结论：

```JAVA
public class FinallyTest1 {

    public static void main(String[] args) {
        System.out.println(test11());
    }
    
    public static String test11() {
        try {
            System.out.println("try block");
           return test12();
      } finally {
           System.out.println("finally block");
       }
  }

  public static String test12() {
       System.out.println("return statement");

       return "after return";
   } 
}
```

结果：

```JAVA
try block
return statement
finally block
after return
```



> 二、**finally块中的return语句会覆盖try块中的return返回。**

```java
public class FinallyTest2 {
    public static void main(String[] args) {
        System.out.println(test2());
    }

    public static int test2() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80;
            
        } catch (Exception e) {
            System.out.println("catch block");
            
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }

            return 200;
        }

        // return b;
    }

}
```

结果：

```java
try block
finally block
b>25, b = 100
200
```

​		这说明finally里的return直接返回了，就不管try中是否还有返回语句，这里还有个小细节需要注意，finally里加上return过后，finally外面的return b就变成不可到达语句了，也就是永远不能被执行到，所以需要注释掉否则编译器报错。



> ##### **三、 如果finally语句中没有return语句覆盖返回值，那么原来的返回值可能因为finally里的修改而改变也可能不变。**

```java
public class FinallyTest3 {

    public static void main(String[] args) {
        System.out.println(test3());
    }

    public static int test3() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80;
            
        } catch (Exception e) {
            System.out.println("catch block");
            
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }

            b = 150;    //这行代码未执行
        }

        return 2000;
    }

}
```

结果：

```JAVA
try block
finally block
b>25, b = 100
100
```

```java
import java.util.*;

public class FinallyTest6
{
    public static void main(String[] args) {
        System.out.println(getMap().get("KEY").toString());
    }
     
    public static Map<String, String> getMap() {
        Map<String, String> map = new HashMap<String, String>();
        map.put("KEY", "INIT");
         
        try {
            map.put("KEY", "TRY");
            return map;
        }
        catch (Exception e) {
            map.put("KEY", "CATCH");
        }
        finally {
            map.put("KEY", "FINALLY");
            map = null;
        }
         
        return map;
    }
}
```

结果：

```java
FINALLY
```

​		为什么测试用例1中finally里的b = 150并没有起到作用而测试用例2中finally的map.put("KEY", "FINALLY")起了作用而map = null;却没起作用呢？

首先明确：看类的class字节码文件就知道了。 return的时候是`复制`了一个变量然后返回，所以之后finally操作的变量如果是基本类型的话不会影响返回值。 但是如果返回值是引用类型的话，因为指向同一个对象所以还是有影响的。

在map = null 的代码块中，
上面的 return 已经确定的要返回对象的地址(第一个地址)
后面将map 置为null，修改了 map 的地址（第二个地址），
但是并不能影响 要 return  的那个地址（返回的还是第一个地址），
返回的还是第一个map引用 指向的那个 hashMap 对象。
所以打印出来的是 finally.

##### 复制的含义：

在return 操作之前，暂且认定局部变量表中有一个reference0指向堆内存中map对象。                     return map的时候：首先会将reference0指向的map对象压栈，然后弹出栈到局部变量表中reference1中，即现在两个引用指向堆中的map对象，而 return 会返回是经过操作数栈得到的 referebce1， 也就是所谓的"复制"。                                        

接着finally代码块会继续执行代码，但是这时候的操作都是针对reference0引用，（注意map.put()操作不属于本方法，它应该是在map 的put方法栈帧中去改变），对于map=null，它会把reference0的引用入栈，然后修改了它的地址，但是并不影响返回的 referebce1

（这里注意了，所有的方法内部的操作，一定是经过操作数栈的，即使是返回一个值这么简单的操作，都会先从变量表拿出引用，经操作以后再返回）



为了验证，这里对之前的代码做修改：

```JAVA
package com.it.dao;

import java.util.HashMap;
import java.util.Map;

public class Test
{
    public static void main(String[] args) {
        System.out.println(getMap().get("KEY").toString());
    }

    public static Map<String, String> getMap() {
        Map<String, String> map = new HashMap<String, String>();
        map.put("KEY", "INIT");

        try {
            map.put("KEY", "TRY");
        }
        catch (Exception e) {
            map.put("KEY", "CATCH");
        }
        finally {
            map.put("KEY", "FINALLY");
            map = null;
        }
        return map;
    }
}
```

结果：

```java
Exception in thread "main" java.lang.NullPointerException
	at com.it.dao.Test.main(Test.java:9)
```

原因就是它这里的map在最后给变为空，因此不管怎么做都是null。



> ##### 四、 **try块里的return语句在异常的情况下不会被执行，这样具体返回哪个看情况。**

```java
package com.it.dao;

public class Test {
    public static void main(String[] args) {

        System.out.println(test4());
    }

    public static int test4() {
        int b = 20;

        try {
            System.out.println("try block");
            b = b / 0;
            return b += 80;
            
        } catch (Exception e) {
            b += 15;
            System.out.println("catch block");
            
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }
            b += 50;
        }

        return b;
    }
}
```

结果：

```JAVA
catch block
finally block
b>25, b = 35
85
```



> ##### 五、 当发生异常后，catch中的return执行情况与未发生异常时try中return的执行情况完全一样。