# 记账软件项目

## 一、技术栈

### **后端**：

SpringBoot（2.3.x）、SpringSecurity（5.3.x）、MybatisPlus（3.3.x）、JWT、Redis（5.0.x，分布式缓存中间件）、rdm（redis可视化）、JDK（21.0.2）、MySQL（8.0）、MariaDB（防止MySQL闭源无法使用，数据特性、性能都超越了MySQL）

### **前端**：

Axure（原型图设计）、Vue

### **小程序端**：

Uniapp、UView

### **开发工具**：

idea、Navicat Premium（数据库可视化）、VMware（17，虚拟机中安装Linux环境）

### **开发工具问答**

1、为什么使用SpringBoot？框架简单易用

2、为什么使用SpringSecurity+JWT？

引入SpringSecurity+JWT目的是为了做无状态基于Token的签发和鉴

3、为何要引入redis

为了实现扫码授权登录，进而引入了Redis。因授权需要的uuid是一个有效期内有效的数据，本身不需要存入数据，而存入Redis的话编码相对简单，性能也很好
 后续也会将一些用户不经常改变的数据存入Redis

4、docker

更快速的交付和部署（打包镜像发布测试一键运行）；更便捷的升级和扩缩容（项目打包成一个镜像，拓展服务器）；更简单的系统运维（开发和测试环境实现高度一致）；更高效的计算资源利用（内核级别的虚拟化，可以在一个物理机上运行很多的容器实例，最大化服务器性能）

## 二、主要功能

### 1、用户/管理员登陆

### 2、个人账户设置（用户模式）

从其他界面回到用户信息的时候，需要刷新用户信息，比如账目总额、记账天数... 这些，所以需要查询用户信息。

**使用redis完成计数服务**（为什么不用mysql）

### 3、用户管理（管理员模式）

### 4、角色管理（管理员模式）

### 5、权限管理（管理员模式）

### 6、记录账目（收入/支出）

### 7、图表分析

### 8、周期分析

### 9、账本导入导出

### 10、设置预算

### 11、多账本管理



## 三、项目难点

### 1、权限控制 + 前端动态路由

### 2、微信扫码小程序授权登录

![image-20240313131240265](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240313131240265.png)

轮询时可能获取到三种状态：1、已扫描；2、已过期；3、已授权

### 3、手机验证码验证登录



## 四、代码分析

### main

#### common

##### anotation

```
@Target({ElementType.METHOD})
```

定义注解的作用对象，比如作用于类、属性、或方法等

```
@Retention(RetentionPolicy.RUNTIME)
```

定义注解的生命周期：

RetentionPolicy.SOURCE : 仅存在于源代码中，编译阶段会被丢弃，不会包含于class字节码文件中。@Override, @SuppressWarnings都属于这类注解。

RetentionPolicy.CLASS : 默认策略，在class字节码文件中存在，在类加载的时被丢弃，运行时无法获取到。

RetentionPolicy.RUNTIME : 始终不会丢弃，可以使用反射获得该注解的信息。自定义的注解最常用的使用方式。

##### LocalUserId和LocalUser

[ThreadLocal类]: 都调用了ThreadLocal，一个获取用户，一个获取用户Id

##### manager

[AsyncManager类]: 异步任务管理器
[ShutdownManager类]: 停止异步任务（关闭后台任务任务线程池）
[AsyncFactory类]: 异步工厂类，记录操作日志

##### aspect



#### exception

异常类

#### mapper

​	数据存储对象，相当于DAO层，mapper层直接与数据库打交道(执行SQL语句)，接口提供给service层。

​	某个DAO一定是和数据库的某一张表一一对应的，其中封装了增删改查基本操作，建议DAO只做原子操作，增删改查。

​	负责与数据库进行联络的一些任务都封装在此，dao层的设计首先是设计dao层的接口，然后在Spring的配置文件中定义此接口的实现类，然后就可以再模块中调用此接口来进行数据业务的处理，而不用关心此接口的具体实现类是哪个类，显得结构非常清晰，dao层的数据源配置，以及有关数据库连接参数都在Spring配置文件中进行配置。

#### service

​	service层即为业务逻辑层，可以理解为对一个或者多个DAO进行得再次封装，主要是针对具体的问题的操作，把一些数据层的操作进行组合，间接与数据库打交道(提供操作数据库的方法)。要做这一层的话，要先设计接口，再实现类。

​	service层主要负责业务模块的应用逻辑应用设计。同样是首先设计接口，再设计其实现类，接着再Spring的配置文件中配置其实现的关联。这样我们就可以在应用中调用service接口来进行业务处理。service层的业务实，具体要调用已经定义的dao层接口，封装service层业务逻辑有利于通用的业务逻辑的独立性和重复利用性。程序显得非常简洁。

​	Service层应该既调用DAO层的接口，又要提供接口给Controller层的类来进行调用，它刚好处于一个中间层的位置。每个模型都有一个Service接口，每个接口分别封装各自的业务处理方法。

#### controller

负责请求转发，接收页面过来的参数，传给service处理，接到返回值，并再次传给页面。

controller层负责具体的业务模块流程的控制，在此层要调用service层的接口来控制业务流程，控制的配置也同样是在Spring的配置文件里进行，针对具体的业务流程，会有不同的控制器。

![image-20240315144842871](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20240315144842871.png)

springmvc实现页面请求和响应；spring实现业务逻辑处理；mabatis实现程序和数据库进行交互

#### constant

要用到的各种常量

#### BO（Busine Object）

业务对象 BO，可以包括一个或多个其它的对象。

例如：学生的综合情况，需要学生的基本信息、成绩等。

用法：

- 不可以继承自 Entity
- BO 对象不得用于 controller 层

#### DTO（Data Transfer Object）

DTO模式是指将数据封装成普通的JavaBeans，在J2EE多个层次之间传输（信使）。

在业务逻辑层或者交互层用到数据库中不存在的字段时，往DTO类里放这些字段，这些字段的意义就相当于一些经处理过的数据库字段，实质意义就是方便数据交互，提高效率。

能够完整表达一个业务模块的输出。

#### entity（DO，Data Object）

数据对象 XxxxEntity

用法：

- 以 Entity 为结尾（阿里是以 DO 为结尾）
- Xxxx 与数据库表名保持一致
- 类中字段要与数据库字段保持一致，不能缺失或者多余
- 类中的每个字段添加注释，并与数据库注释保持一致
- 不允许有组合

#### VO（View Object）

视图对象 XxxxVO，用于展示层，它的作用是把某个指定页面（或组件）的所有数据封装起来。

用法：

- 不可继承自 Entity
- VO 可以继承、组合其他 DTO，VO，BO 等对象
- VO 只能用于返回前端、rpc 的业务数据封装对象

#### utils

[AddressUtil]: 获取地理位置
[IpUtil]: 
[HttpUtil]: 发送get请求；发送post请求
[ClassUtil]: 获取属性值、bean转为map（属性名-属性值）
[DateUtil]: 获取当前时间、年月份，按周分割时间段，按月份分割时间段
[EncryptUtil]: 加密工具类，使用MessageDigest进行单向加密（无密码）；使用KeyGenerator进行单向/双向加密（可设密码）；使用KeyGenerator双向加密，DES/AES；md5加密算法进行加密（不可逆）；使用SHA1加密算法进行加密（不可逆）；使用DES加密算法进行加密和解密（可逆）；使用AES加密算法经行加密和解密（可逆）；使用异或进行加密和解密
[ThreadUtil]: 线程相关工具，等待、关闭线程、打印线程异常
[StringUtil]: 字符串工具类
[SpringContextUtil]: Spring获取上下文的工具类
[ServletUtil]: 服务器工具类
[RedisUtil]: 
[PasswordUtil]: 密码格式、密码检查、随机密码
[IDUtil]: 生成32位不带“-”的ID

### 集群的session共享问题

多台tomcat不共享session存储空间，当请求切换到不同tomcat服务时导致数据丢失的问题。

引入**redis**，基于redis实现共享session登陆
