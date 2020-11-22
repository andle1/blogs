* 对象定义在错误的范围（Wrong Scope）
* 异常 （Exception）处理不当
* 集合数据管理不当



##### 对象定义在错误的范围（Wrong Scope）：

如果 Foo 实例对象的生命较长，会导致临时性内存泄漏。（这里的 names 变量其实只有临时作用）

```java
class Foo{
    private String[] names;
    public void doIt(int length){
        if(names == null || names.length < length)
            names = new String[length];
        populate(names);
        print(names);
    }
}
```

JVM 喜欢生命周期短的对象，下面这样做已经足够高效。因为不管 Foo 实例对象存活多长时间，都不会影响局部变量 names 的回收。局部变量使用后是立即被回收的

```java
class Foo{
    public void soIT(int length){
        String[] names = new String[length]
        populate(names);
        print(names);   
    }
}
```



##### 异常 （Exception）处理不当：

```java
Connection conn = Drivermanager.getConnection(url,name,passwd);

try{
    String sql = "do a query sql";
    PrepareStatement stmt = conn.prepareStatement(sql);
    ResultSet rs = stmt.executeQuery();
    while (rs.next()){
        doSomeStuff();
    }
    rs.close();
    conn.close();
} catch (Exception e)
{
    // 如果 doSomeStuff 抛出异常，rs.close();    conn.close(); 不会被调用，会导致内存泄漏和db连接泄漏
}
```

正确的做法：

```java
Connection conn = Drivermanager.getConnection(url,name,passwd);
PrepareStatement stmt = null;
ResultSet rs = null;

try{
    String sql = "do a query sql";
    stmt = conn.prepareStatement(sql);
    rs = stmt.executeQuery();
    while (rs.next()){
        doSomeStuff();
    }
} catch (Exception e){
    // 
} finally {
    if (rs != null){
        rs.close();
    }
    if(stmt != null){
        stmt.close();
    }
    conn.close();
}
```



##### 集合数据管理不当

* 当使用 Array-based 的数据结构（ArrayList，HashMap等）时，尽量减少 resize。
  * 比如 new ArrayList 时，尽量估算 size，在创建的时候把 size 确定
  * 减少 resize 可以避免没有必要的 array coping，gc碎片等问题
* 如果一个 List 只需要顺序访问，不需要随机访问（Random Access），用LinkedList 代替ArrayList
  * LinkedList 本质是链表，不需要 resize，但只适用于顺序访问