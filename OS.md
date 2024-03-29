# OS

## 进程和线程

1、进程是系统的基本单位（资源分配和调度），线程是CPU的基本单位（调度和分派）。

2、线程依赖于进程，一个进程至少有一个线程。

3、进程有自己的独立地址空间，线程共享所属进程的地址空间。

4、进程是拥有系统资源的一个独立单位，而线程基本上不拥有系统资源，只拥有一点在运行中必不可少的资源。

5、进程切换开销远大于线程切换，进程切换涉及整个当前进程CPU环境的保存环境的设置以及新被调度运行的CPU环境设置，线程切换只需保存和设置少量的寄存器的内容。

6、线程间通信更方便，同一进程下的线程**共享全局变量等数据**，进程间通信需要以进程间通信（IPC）的方式进行。

7、多进程程序更为健壮，多线程程序中一个线程崩溃会使整个程序崩溃。

## 进程有哪些组成部分

由进程控制块和相应的地址空间组成。

进程控制块主要包括：

1、进程标识符

2、进程的当前状态

3、相应的进程控制信息

地址空间包括：

1、程序段

2、用户数据段：相应的程序处理数据

3、系统数据段：相应的程序运行环境

## 进程的上下文

进程上下文实际上是进程执行活动全过程的静态描述，是可执行的程序代码，是进程的重要组成部分。

用户级上下文：正文、数据、用户堆栈、共享存储区。

寄存器上下文：通用寄存器、程序寄存器（IP）、处理器状态寄存器（EFLAGS）、栈指针寄存器（ESP，Stack Point）

系统级上下文：进程控制块（task_struct）、内存管理信息（mm_struct、vm_area_struct、pgd、pte）、内核栈

## 栈与栈帧

EBP寄存器（又称为帧指针，Frame Pointer）。

栈是从高地址向低地址延伸，一个函数的栈帧用EBP和ESP这两个寄存器来划定范围。EBP指向当前栈帧的底部，ESP始终指向栈帧的顶部。

每一次函数的调用，都会在调用栈（call stack）上维护一个独立的栈帧（stack frame）。每个独立的栈帧一般包括：

1. 函数的返回地址和参数
2. 临时变量：包括函数的非静态局部变量以及编译器自动生成的其他临时变量
3. 函数调用的上下文

```text
;栈帧结构
PUSH EBP            ;函数开始（使用EBP前先把已有值保存到栈中）
MOV EBP, ESP        ;保存当前ESP到EBP中

...                 ;函数体
                    ;无论ESP值如何变化，EBP都保持不变，可以安全访问函数的局部变量、参数

MOV ESP, EBP        ;将函数的起始地址返回到ESP中
POP EBP             ;函数返回前弹出保存在栈中的值
RETN                ;函数终止
```

![v2-3d901f980d242f6c5ce09fb2175cfa1a_720w](C:\Users\Administrator\AppData\Local\Temp\v2-3d901f980d242f6c5ce09fb2175cfa1a_720w.webp)

## 函数调用

1. 被调用函数（callee）的参数按逆序入栈。（不需要参数则没有此步骤）
2. 调用函数（caller）进行调用后的下一条指令（EIP）作为（被调用函数的）返回地址压入栈内。
3. 将当前EBP寄存器的值（也就是调用函数的基地址）压入栈内，并将EBP寄存器的值更新为当前栈顶的地址（就是更新为被调用函数的基地址）
4. 将被调用函数的局部变量等数据压入栈内。

压栈过程ESP寄存器的值不断减小（因为内存逆生长，从高地址向低地址），压入栈内的数据包括调用参数、返回地址、调用函数的基地址、局部变量，其中除调用参数之外的数据组成了被调用函数的状态。在发生调用时，程序还会将被调用函数的指令地址存到EIP寄存器中，这样程序可以执行被调用函数的指令。

## 函数调用结束

1. 弹出被调用函数的局部变量，此时栈顶指向被调用函数的基地址
2. 将存储的调用函数的基地址弹出并存入EBP寄存器，此时栈顶指向返回地址。
3. 将返回地址弹出并存入EIP寄存器，调用函数的指令信息得以恢复。
4. 调用函数的状态已完全恢复，可以继续执行调用函数的指令。

## 资源管理

进程间调度、IO调度、磁盘调度、死锁问题都属于资源管理。

## 同一进程中的线程共享哪些数据

- 进程代码块
- 进程的公有数据（全局变量、静态变量……）
- 进程打开的文件描述符
- 进程的当前目录
- 信号处理器/信号处理函数
- 进程ID与进程组ID

## 线程独占哪些资源

- 线程ID
- 一组寄存器的值
- 线程自身的栈（堆是共享的）
- 错误返回码，一个线程的错误返回码不应该被其他线程修改
- 信号掩码/信号屏蔽字：表示是否屏蔽、阻塞相应的信号

## 进程间通信有哪些方式（不同场合怎样选择合适的通信方式）

1. #### 管道

​	又分为匿名管道**PIPE**和命名管道**FIFO**。管道类似于缓存，一个进程把数据放在缓存中，让另外一个进程取，单个管道是半双工，数据是单向流动的。

​	管道的缺点是效率较低，不适合频繁的交换数据；优点是实现简单，自带同步互斥机制。

##### 	匿名管道

- 匿名管道是半双工的，数据是单向流动的
- 匿名管道使用完会被销毁。在linux任何一个shell中，都可以使用‘|’连接两个命令，shell会将前后两个进程的输入输出用一个管道相连，达到进程间通信。
- 匿名管道只能用于父子进程。同一时刻一个管道只能完成单向的数据传输
- 匿名管道通信**过程**：
  1. 匿名管道的原型是一个int类型的pipe函数，通信成功返回0，失败返回-1。传入参数是一个有两个元素的fd数组，fd[0]和fd[1]，一个是管道读端，一个是管道写端。父进程在产生子进程前，创建一个匿名管道，让两个文件描述符指向管道两端。
  2. 然后父进程fork产生子进程，子进程通过拷贝父进程的进程地址空间获得同一个管道文件的两个描述符，达到使用同一个管道通信的目的。
  3. 父进程关闭fd[0]管道读端，子进程关闭fd[1]管道写端，因为管道是单向的，就能完成进程间通信单向通信。也可以换一下关闭的端口，让数据从子进程流向父进程。

##### 	命名管道

- 命名管道通过mkfifo命令创建，用自己命名的变量或者说文件存储（因为linux下都视为文件），这个文件就是命名管道，有路径名与之关联。
- 命名管道的通信没有亲缘限制，只要能访问命名管道文件路径的进程就能与本进程通信
- 读写数据的方式是先进先出FIFO

2. #### 消息队列

- 消息队列是消息的**链表**，放在**内核**中，有标识符标识。
- 队列中的消息有特定的格式和优先级
- 消息队列和命名管道一样可以支持两个不相关的进程通信，但消息队列能**传递的信息更多**，而且是**独立**的
- 消息队列**独立于发送和接收进程**。进程终止时，消息队列及其内容并不会被删除，**生命周期跟随内核**，如果没有释放消息队列或者没有关闭操作系统，消息队列会一直存在，所以相比于管道，解决了必须等待进程来取数据的问题。
- 消息队列可以实现消息的随机查询，消息不一定要以先进先出的次序读取，也可以按消息的类型读取
- 消息队列的优点就是提到的**数据独立于进程，能双向通信，允许多个进程写入读取数据**。
- 缺点有两个：一个是消息队列**不适合传输比较大的数据**，因为在内核中每个消息体都有一个最大长度的限制，同时所有队列所包含的全部消息体的总长度也是有上限；另一个是消息队列通信中，**存在用户态与内核态之间的数据拷贝开销**，因为进程写入数据到内核中的消息队列时，会发生从用户态拷贝数据到内核态；读数据的时候，会从内核态拷贝数据到用户态。

3. #### 共享内存

​	共享内存解决了消息队列拷贝数据所消耗的时间。

​	系统会给每个进程分配一个独立的虚拟空间，不同的虚拟内存映射到不同的物理内存中，所以即便A与B的虚拟地址是一样，但是他们访问的是不同的物理地址，对于数据的增删改查互不影响。 

​	共享内存的机制，是两个进程各拿出一块虚拟空间，映射到相同的物理内存中。这样一个进程写入东西的时候，另一个进程马上就看到了，不需要来回拷贝，所以共享内存也是**最快的通信方式**。共享内存的优点是通信快，缺点是**需要进行进程间同步**，否则两个进程同时操作一块物理内存会出问题，所以共享内存经常和**信号量**一起使用，或者也可以使用**互斥锁**。

4. #### 信号量

- 信号量是一个整型计数器，不同于其他通信方式，信号量不存储通信数据，而只是进程间同步的方式，用来控制多个进程对共享资源的访问，如果真的传递数据需要结合共享内存。
- 信号量是一种锁机制，用来防止某个进程访问共享资源的时候其他进程也访问这个资源，实现进程间同步。
- 程序对信号量的操作是原子操作，信号量是基于操作系统的PV操作。
- P操作使信号量-1然后再判断信号量是否小于0，如果小于0就先阻塞进程，再让进程进入到信号量的等待队列中，再进行进程调度；否则就让进程继续进行。
- V操作使信号量+1，然后判断信号量是否≤0，如果是，就从信号等待队列释放一个进程，然后再返回原进程或者进行线程调度。
- PV操作成对出现，P操作使进入共享资源之前，V操作是离开共享资源。
- 最简单的信号量就是只能取0和1的变量，叫二值信号量也叫**互斥信号量**，因为这保证了同一时间只有一个进程在访问，保护了共享内存；如果能取多个整数的信号量被称为**通用信号量**，Linux下的信号量函数都是在通用的信号量数组上操作的。

5. #### 信号

- 信号传递的消息特别少，所以用信号进行通信仅用于通知接收进程某个事件发生
- 信号的事件的来源主要是两个:一个是硬件来源，通过键盘输入;另一个是软件来源，比如输入kill命令
- 如果是运行在shell终端的进程，可以通过键盘组合键的方式，给进程发信号：比如ctr+C产生SIGINT信号，表示终止该进程；ctrl + Z产生SIGTSTP信号，表示停止该进程，但未结束。
- 如果进程是在后台运行的，可以通过kill命令来给进程发信号，但需要提前知道进程的PID号
- **信号是进程间通信机制中唯一的异步通信机制，因为可以在任何时候发送信号给某一进程**
- 我们可以设置进程在收到信号后的反应：一个是不进行设置，让进程执行信号的默认操作:另一个是定义一个信号处理函数，进行捕捉信号做相应的处理；另外还可以忽略信号

6. #### 套接字（Socket）

- socket利用**三元组（ip地址、协议、端口）**唯一标识网络中的进程，网络中的进程通信可以利用这个标志与其他进程进行交互。
- 如果是使用本地socket的话，就是本机的两个进程通信
- 如果是网络套接字，可以进行不同主机上的进程间通信，也就是TCP字节流通信和UDP数据报通信，也就是网络编程（实现过多线程客户端聊天室，用本地局域网ip，也是用socket实现）
- socket通信是全双工的，同一时间可以双向通信

消息队列和管道都是4次拷贝，而共享内存（mmap，shmget）是2次。

4次：1-将用户空间的数据拷贝到内核中。2-内核将数据拷贝到内存中。3-从内存取出数据到内核。4-内核到用户空间

2次：1-用户空间到内存。2-内存到用户空间。

## 临界资源

也叫互斥资源、共享变量，一次只能给一个进程使用。

## 临界区

各个进程中对临界资源进行操作的程序片段。

## 同步

多个进程因为合作而使得进程的执行有一定的先后顺序。比如某个进程需要另一个进程提供的消息，获得消息之前进入阻塞态

## 互斥

多个进程在同一时刻只有其中一个能进入临界区

## 并发、并行、异步

**并发：**在一个时间段中同时有多个程序在运行，但其实任一时刻，只有一个程序在CPU上运行，宏观上的并发是通过不断的切换实现的；
**多线程**：并发运行的一段代码。是实现异步的手段
**并行**(和串行相比)：在多CPU系统中，多个程序无论宏观还是微观上都是同时执行**异步**(和同步相比)：同步是顺序执行，异步是在等待某个资源的时候继续做自己的事

## 进程有哪几种状态

**就绪状态**：进程已获得除处理器以外的所有资源，等待分配处理机资源。

**运行状态**：占用处理机资源运行，处于此状态的进程数≤CPU数

**阻塞状态**：进程等待某种条件，在条件满足之前无法执行

**运行态→阻塞态**：往往是由于等待外设，等待主存等资源分配或等待人工干预而引起的。

**阻塞态→就绪态**：等待的条件已满足，只需分配到处理器后就能运行。

**运行态一就绪态**：不是由于自身原因，而是由外界原因使运行状态的进程让出处理器，这时候就变成就绪态。例如时间片用完，或有更高优先级的进程来抢占处理器

**就绪态→运行态**：系统按某种策略选中就绪队列中的一个进程占用处理器，此时就变成了运行态

## **进程调度策略有哪些**

1. #### 批处理系统

- ##### 先来先服务 first-come first-served（FCFS）

​	按照请求的顺序进行调度。非抢占式，开销小，无饥饿问题，响应时间不确定（可能很慢）；对短进程不利，对IO密集型进程不利。

- ##### 最短作业优先 shortest job first（SJF）

​	按估计运行时间最短的顺序进行调度。**非抢占式**，吞吐量高，开销可能较大，可能导致饥饿问题；提供好的响应时间，对长进程不利。

- ##### 最短剩余时间优先 shortest remaining time next（SRTN）

​	按剩余运行时间的顺序进行调度（最短作业优先的抢占式版本）。吞吐量高，开销可能较大，提供好的响应时间。可能导致饥饿问题，对长进程不利。

- ##### 最高响应比优先 Highest Response Ratio Next（HRRN）

​	响应比 = 1 + 等待时间/处理时间。同时考虑了等待时间的长短和估计需要的执行时间长短，很好的平衡了长短进程。非抢占，吞吐量高，开销可能较大，提供好的响应时间，无饥饿问题。

2. #### 交互式系统

​	交互式系统有大量的用户交互操作，在该系统中调度算法的目的是快速地进行响应。

- ##### 时间片轮转 Round Robin

​	将所有就绪进程按FCFS的原则排成一个队列，用完时间片的进程排到队列最后。抢占式（时间片用完时），开销小，无饥饿问题，为短进程提供好的响应时间。

- ##### 优先级调度算法

​	为每个进程分配一个优先级，按优先级进行调度。为了防止低优先级的进程永远等不到调度，可以随着时间的推移增加等待进程的优先级。

- ##### 多级反馈队列调度算法

​	设置多个就绪队列1、2、3，优先级递减，时间片递增。只有等到优先级更高的队列为空时才会调度当前队列中的进程。如果进程用完了当前队列的时间片还未执行完，则会被移到下一队列。
​	抢占式（时间片用完时），开销可能较大，对IO型进程有利，可能会出现饥饿问题

## 什么是优先级反转？怎么解决？

​	高优先级的进程等待被一个低优先级进程占用的资源时，就会出现优先级反转，即优先级较低的进程比优先级较高的进程先执行。

​	**解决办法**

- 优先级天花板（priority ceiling）：当任务申请某资源时，把该任务的优先级提升到可访问这个资源的所有任务中的最高优先级，这个优先级称为该资源的优先级天花板。简单易行。	
- 优先级继承（priority inheritance）：当任务A申请共享资源S时，如果S正在被任务C使用，通过比较任务C与自身的优先级，如发现任务C的优先级小于自身的优先级，则将任务C的优先级提升到自身的优先级，任务C释放资源S后，再恢复任务C的原优先级。

## 僵尸进程

​	子进程**结束后**，它的父进程并没有等待它（调用wait或者waitpid），那么这个子进程将成为一个僵尸进程。

​	僵尸进程是一个已经死亡的进程，但是并没有真正被销毁。它已经放弃了几乎所有内存空间，没有任何可执行代码，也不能被调度，仅仅在进程表中保留一个位置，记载该进程的进程ID、终止状态以及资源利用信息（CPU时间，内存使用量等等）供父进程收集，除此之外，僵尸进程不再占有任何内存空间。这个僵尸进程可能会一直留在系统中直到系统重启。
**危害**：占用进程号，而系统所能使用的进程号是有限的；占用内存。

**以下情况不会产生僵尸进程：**

- 该进程的父进程先结束了。每个进程结束的时候，系统都会扫描是否存在子进程如果有则用Init进程接管，成为该进程的父进程，并且会调用wait等待其结束。
- 父进程调用wait或者waitpid等待子进程结束（需要每隔一段时间查询子进程是否结束）。**wait系统调用会使父进程暂停执行，直到它的一个子进程结束为止**。waitpid则可以加入 WNOHANG（wait-no-hang）选项，如果没有发现结束的子进程，就会立即返回，不会将调用waitpid的进程阻塞。同时，waitpid还可以选择是等待任一子进程（同wait），还是等待指定pid的子进程，还是等待同一进程组下的任一子进程，还是等待组ID等于pid的任一子进程
- 子进程结束时，系统会产生 SIGCHLD（signal-child）信号，可以注册一个信号处理函数，在该函数中调用waitpid，等待所有结束的子进程（注意：一般都需要循环调用waitpid，因为在信号处理函数开始执行之前，可能已经有多个子进程结束了，而信号处理函数只执行一次，所以要循环调用将所有结束的子进程回收）
- 可以用signal（SIGCLD，SIG_IGN）（signal-ignore），表示忽略SIGCHLD信号，那么子进程结束后，内核会进行回收。

## 孤儿进程

一个父进程已经结束了，但是它的子进程还在运行，那么这些子进程将成为孤儿进程。孤儿进程会被Init（进程ID为1）接管，当这些孤儿进程结束时由Init完成状态收集工作。

## 线程同步的作用

​	线程有时候会和其他线程共享一些资源，比如内存、数据库等。当多个线程同时读写同一份共享资源时，可能会发生冲突，因此需要线程同步，多个线程按顺序访问资源。

## 线程同步的方式

​	**互斥量 Mutex**：互斥量是内核对象，只有拥有互斥对象的线程才有访问互斥资源的权限。因为互斥对象只有一个，所以可以保证互斥资源不会被多个线程同时访问；当前拥有互斥对象的线程处理完任务后必须将互斥对象交出，以便其他线程访问该资源。

​	**信号量 Semaphore**：信号量是内核对象，它允许同一时刻多个线程访问同一资源，但是需要控制同一时刻访问此资源的最大线程数量。信号量对象保存了**最大资源计数和当前可用资源计数**，每增加一个线程对共享资源的访问，当前可用资源计数就减1，只要当前可用资源计数大于0，就可以发出信号量信号，如果为0，则将线程放入一个队列中等待。线程处理完共享资源后，应在离开的同时通过Releasesemaphore 函数将当前可用资源数加1。如果信号量的取值**只能为0或1**，那么信号量就成为了互斥量。

​	**事件 Event**：允许一个线程在处理完一个任务后，**主动唤醒**另外一个线程执行任务。事件分为手动重置事件和自动重置事件。手动重置事件被设置为激发状态后，会唤醒所有等待的线程，而且一直保持为激发状态，直到程序重新把它设置为未激发状态。自动重置事件被设置为激发状态后，会唤醒**一个**等待中的线程，然后自动恢复为未激发状态。

​	**临界区 Critical Section**：任意时刻只允许**一个线程**对临界资源进行访问。拥有临界区对象的线程可以访问该临界资源，其它试图访问该资源的线程将被挂起，直到临界区对象被释放。

## 互斥量和临界区

​	互斥量可以**命名**，可以用于**不同进程间的同步**；临界区只能用于**同一进程中线程的同步**。创建互斥量需要的资源更多，因此临界区的优势是**速度快，节省资源**。

## 什么是协程

​	协程是一种**用户态的轻量级线程**，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以**不加锁的访问全局变量**，所以**上下文的切换非常快**。

## 协程和线程进行比较

1. 一个线程可以拥有多个协程，一个进程也可以单独拥有多个协程，这样python能使用多核CPU。
2. 线程进程都是同步机制，而协程是异步。
3. 协程能保留上一次调用时的状态，每次过程重入时，就相当于进入上一次调用的状态。

## 文件描述符



