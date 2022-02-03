# [最全面的Java多线程学习概述](https://youzhixueyuan.com/java-concurrent-programming.html)

> 优知学院- Java多线程与并发系列

****

#### Java并发编程的技能的5个方面

* 多线程
* 线程池
* 线程锁
* 并发工具类
* 并发容器

****

#### 多线程的4种创建方式

<img src="https://cdn.jsdelivr.net/gh/Mark-Jackson-Github/images@master/uPic/image-20220202231911222.png" alt="image-20220202231911222" style="zoom:50%;" />

1. 继承Thread类
2. 实现Runnable接口
3. 实现Callable接口
4. 利用线程池创建接口

****

#### Java线程的生命周期

<img src="https://cdn.jsdelivr.net/gh/Mark-Jackson-Github/images@master/uPic/image-20220202232110136.png" alt="image-20220202232110136" style="zoom:50%;" />

****

#### Java线程的通信与协作

<img src="https://cdn.jsdelivr.net/gh/Mark-Jackson-Github/images@master/uPic/image-20220202232246614.png" alt="image-20220202232246614" style="zoom:50%;" />

****

#### 线程的生命周期

<img src="https://cdn.jsdelivr.net/gh/Mark-Jackson-Github/images@master/uPic/image-20220203021729064.png" alt="image-20220203021729064" style="zoom:50%;" />

1. **初识状态new：**指的是线程已被创建，但是还是不允许CPU执行。