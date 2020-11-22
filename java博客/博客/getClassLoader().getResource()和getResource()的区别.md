在Java中需要加载一个文件时，使用getResource()方法进行加载，会报错

>  [Caused by: java.lang.NullPointerException: Location is required.

这是对 a.class.getClass().getClassLoader().getResource()和 a.class.getClass().getResource()的理解不够深入的原因。(a代表某个类)



##### 最简单的理解：

> .getClass().getClassLoader() 理解为根目录



##### 区别主要如下：

> - .getClass().getResource(fileName) ：表示**只会在当前调用类所在的同一路径下**查找该fileName文件；
> - .getClass().getClassLoader().getResource(fileName)：表示**只会在根目录下（/）**查找该文件；
> - fileName如果是前面加“/”，如"/fileName"，则表示绝对路径，取/目录下的该文件；
>      如果是前面没有加“/”，如"fileName"，则表示相对路径，取与调用类同一路径下的该文件。
> - 如果路径中包含包名 ，getClass().getResource("com/xxx/1.xml");
>      包名的层级使用"/"隔开（正斜杠），而非“.”（半角句号）。



举例：
包com.aaa下有调用类A，需要引用配置文件1.xml：

###### 1. 配置文件在包com.aaa下

> getClass().getResource("1.fxml") **——成功**
> getClass().getResource("/1.fxml")——失败
> getClass().getClassLoader().getResource("1.fxml")——失败
> getClass().getClassLoader().getResource("/1.fxml")——失败

第2条失败，原因是使用了绝对路径，路径不正确（/目录下没有该文件）。应为：

> getClass().getResource("/com/aaa/1.fxml") （com前有"/"，表示绝对目录，从/目录开始）

第3条失败是因为相对路径不正确，应为：

> getClass().getClassLoader().getResource("com/aaa/1.fxml")
> （此处注意com前面没有“/”，因为getClassLoader()已经表示/目录）

第4条失败是绝对路径不正确，因为当前已在/目录下，再使用/1.fxml就出错。可以改为如下：

> getClass().getClassLoader().getResource("./1.fxml")



###### 2.配置文件在根目录下，

> getClass().getResource("1.fxml") ——失败
> getClass().getResource("/1.fxml")**——成功**
> getClass().getClassLoader().getResource("1.fxml")**——成功**
> getClass().getClassLoader().getResource("/1.fxml")——失败

第1条是使用相对路径，路径不正确所以失败，应为：

> getClass().getResource("../../1.fxml")

第4条失败是因为当前路径已经为/。可以使用：

> getClass().getClassLoader().getResource("./1.fxml")



补充：

.\   表示项目文件所在目录
..\ 表示项目文件所在目录向上一级目录
..\..\表示项目文件所在目录向上二级目录



为什么是这样读取文件的?

> ​		原因是在编译以后，和开发的时候的文件目录是不一样的，主目录就是 / ，而配置文件实际上是直接在根目录下（如果配置文件在开发的时候直接放在resource下的话）。

![1565076474080](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1565076474080.png)

这里配置文件直接在根目录下的情况：

![1565076852646](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1565076852646.png)