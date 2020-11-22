

##### 对实例方法上锁：

```java
private synchronized void setX(int x){
       this.x = x;
}
```

没加 synchronized 的反编译结果：

```java
 private void setX(int);
    descriptor: (I)V
    flags: ACC_PRIVATE
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: iload_1
         2: putfield      #4                  // Field x:I
         5: return
      LineNumberTable:
        line 26: 0
        line 27: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/jvm/bytecode/MyTest2;
            0       6     1     x   I

```



加 synchronized 的反编译结果：

```java
private synchronized void setX(int);
    descriptor: (I)V
    flags: ACC_PRIVATE, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: iload_1
         2: putfield      #4                  // Field x:I
         5: return
      LineNumberTable:
        line 26: 0
        line 27: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/jvm/bytecode/MyTest2;
            0       6     1     x   I

```

唯一的区别就是在 访问标志这里多了 ACC_SYNCHRONIZED。

synchronized 修饰的实例方法的话，表示的是给当前对象加锁。它并不会管方法的描述符，故默认已经在code里面执行过了，moniterenter 和moniterexit 方法也会执行（表示对于对象的上锁和解锁）。

可重入锁：一个线程拿到对象后调用 synchronized 修饰的方法，此方法里又调用此类的另外的 synchronized 修饰的方法，这里拿到了几个锁，entry count 就加几次（看官网moniterenter解释），那么就必须退出几次，即 moniterexit 几次，直到 entry count 变为0。

能不用 synchronized  就不用，因为阻塞状态会浪费 cpu 资源。



##### 对实例方法的某部分加锁：

```java
  private void test(String str){
        synchronized (str){
            System.out.println("hello word");
        }
    }
```

这里方法的执行过程中就会出现 monitorexit，正常执行结束会规划锁并且跳转22行，即return。但是第二次出现的原因是，如果操作出现了问题，在退出程序之前，还得把锁退还，再抛出异常。

```java
private void test(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: ACC_PRIVATE
    Code:
      stack=2, locals=4, args_size=2
         0: aload_1
         1: dup
         2: astore_2
         3: monitorenter
         4: getstatic     #12                 // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #13                 // String hello word
         9: invokevirtual #14                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_2
        13: monitorexit
        14: goto          22
        17: astore_3
        18: aload_2
        19: monitorexit
        20: aload_3
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 32: 0
        line 33: 4
        line 34: 12
        line 35: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  this   Lcom/jvm/bytecode/MyTest2;
            0      23     1   str   Ljava/lang/String;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class com/jvm/bytecode/MyTest2, class java/lang/String, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4

```



##### 对静态方法上锁：

当给 static 方法添加 synchronize 时，表示的是给当前类的Class对象上锁。这里就表示给 test 对象的Class 对象上锁。

```java
public class test{
    private synchronized static void test2(){
        
    }
}
```



这里其实就是类锁和对象锁的区别，一个是实例对象本身加锁，一个是 Class 对象。