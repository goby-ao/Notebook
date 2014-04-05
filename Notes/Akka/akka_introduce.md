# Akka 介绍
----------
##1.简介
官方文档是是这么说的：

    We believe that writing correct concurrent, fault-tolerant and scalable applications is too hard. Most of the time it's because we are using the wrong tools and the wrong level of abstraction. Akka is here to change that. Using the Actor Model we raise the abstraction level and provide a better platform to build scalable, resilient and responsive applications—see the Reactive Manifesto for more details. For fault-tolerance we adopt the "let it crash" model which the telecom industry has used with great success to build applications that self-heal and systems that never stop. Actors also provide the abstraction for transparent distribution and the basis for truly scalable and fault-tolerant applications.
    

翻译一下


    我们都知道，要想写一个能正确的可并发、容错性好的可扩展应用是非常困难的。最主要的原因是我们用错了工具和在错误的级别上进行抽象（actor 模式的抽象级别要远高于共享内存模式）。akka 正式为了应对这种情况而孕育而生。actor 模型提高了抽象级别并且提供了一个更好的平台去构建正确的、并发的可扩展应用。akka 采用的是一种称为 “let it crash”（让它崩溃）的模式去获得良好的容错性。这种模式在电信领域获得了极大的成功,erlang就是采用这种模式。有了上面这些，akka 就可以很容易构建自我恢复和永不停息的应用。actor 模型也提供了这样一种抽象，使得分布式更加透明化。

我自己提炼一下：

akka 是一个用 Scala 写的可以在 JVM         上简单构建并发和分布式程序开源软件框架，它将开发人员从显式地处理锁和线程管理的工作中解脱出来，使编写并发和分布式系统更加容易。它基于 actor 模型实现并发 ， Actor模型为编写并发和分布式系统提供了一种更高的抽象级别。（灵感来自于 Erlang，天生为并发而生的函数式编程语言）。


+ 关键词：`分布式程序`，`并发`，`扩展性`，`容错`，`actor模型`，`Scala`
+ keyword： `distributed application`，`scalable`，`fault-tolerant`，`actor model`，`Scala`，`Concurrency`


----------

##2.相关知识
下面简单介绍一下分布式、并发以及程序语言相关的知识。

###2.1并发与分布式
为什么要这么麻烦写并发程序，因为性能越来越重要，并且随着多核的普及，并发能够最大化利用计算机计算资源。另外很多程序确实是需要并行处理的。例如在分布式系统中并发是不可或缺的，分布式系统里很多操作是并发执行的。

这里再谈谈分布式，我认为分布式系统理论是云计算技术的奠基，云计算多半只是一个概念，商业模式，技术上，底层还是分布式。分布式涉及的事务是相当复杂，要实现数据的一致性，就在今年实现分布式一致性 paxos 算法的作者拿下图灵奖。`容错性`,`可扩展性`是分布式最大的特点。

###2.2函数式语言与 Scala
####2.2.1函数式语言
摘抄wiki上的概念：
>函数式编程是一种编程模型，他将计算机运算看做是数学中函数的计算，并且避免了状态以及变量的概念

函数式语言是一种编程范式，编程思想，顾名思义：纯函数式。近些年，函数式语言的得到快速发展，并且流行起来。实际上更严格一点的来分，函数式语言可分为`纯函数式语言`和`非纯函数式`。

**纯函数式语言**
<ul>
<li>强静态类型
<ul>
<li>oncurrent Clean</li>
<li>Haskel</li>
<li>Miranda</li>
</ul></li><li>弱类型<uL><li>Lazy K</li>
</ul>
</li>
</ul>

**非纯函数式编程语言**
<ul>
<li>强静态类型
<ul>
<li>F#</li>
<li>ML</li>
<li>OCaml</li>
<li>Scala</li>
</ul></li><li>强动态类型<uL>
<li>Erlang</li>
<li>LISP</li>
<li>Scheme</li>
<li>Clojure</li>
<li>Mathematica</li>
<li>R</li>
</ul>
</li>
</ul>
下面谈谈为什么函数式语言可以写高并发程序，函数式语言中没有变量，没有状态，函数计算结果也不依赖状态，同一个输入只有一个输出，因此不会出现多个资源竞争同一个变量的情况，也不需要所谓的进程锁。例如

> un1(x) + fun2(y) + fun3(z) 这样的表达式，如果你有三个核的话，可以每个核并行计算一个fun..

>但是命令式程序中是不可以让他们同时进行计算的，原因是命令式程序依赖某个状态，因此相同输入函数未必对应相同输出，还要考虑当时的机器状态，具体到这个例子当中就是

>fun1(x) + fun2(y) 不一定等于 fun2(y) + fun1(x) >它们彼此之间必须保持严格的运算次序，这样才能保证程序被正确执行，因此也就无法并行化了

把非函数式编程里的循环写成递归，这函数式语言经常出现。函数式语言还优化了递归，转为尾递归，避免递归栈过大撑爆内存。

虽然函数式语言很强大，不过它的适用范围有限，最适合地还是解决局部性的数学上的问题，要让函数式编程来做 CRUD，来做我们传统的逻辑性很强的 Web 编程，就有些免为其难了。就像如果要用 Scala 完全取代今天的 Java 的工作，我想恐怕效果会很糟糕。而让 Scala 来负责底层服务的编写，恐怕再合适不过了。

###2.2.2 Scala

Scala 是一种多范式的编程语言，无缝集成面向对象编程和函数式编程的各种特性，运行在 JVM（java 虚拟机） 上，并兼容现有的Java程序。用来以简明、优雅、类型安全的方式表示常见的编程模式，可以大大提高 Java 员的编程效率。Scala 可以与 Java 互操作。它用 scalac 这个编译器把源文件编译成Java的class文件（即在JVM上运行的字节码）。你可以从 Scala 中调用所有的 Java 类库，也同样可以从 Java 应用程序中调用 Scala 的代码。所以也可以跨平台。本文介绍的 akka 就是用 Scala 写的。

###2.2.3 JVM
JVM 也就是java虚拟机，java 底层的东西，架构图如下：

![JVM 构成][1]

另外，对 JVM 感兴趣的可以看看[这篇文章][2]，这里不深入介绍。

###2.3  actor模型
大多数流行的语言并发是基于多线程之间的共享内存，使用同步方法防止写争夺。而 Erlang 和 Scala 都是基于 actor 模型来实现轻松的并发。actor 可以实现高并发事务。
按 akka 官方的说法:
> actor 模型提高了抽象级别并且提供了一个更好的平台去构建正确的、并发的可扩展应用。
> actor 提供了：
> 
 - 对并发/并行程序的简单的、高级别的抽象
 - 异步、非阻塞、高性能的事件驱动编程模型
 - 非常轻量的事件驱动处理（1G内存可容纳约270万个actors

可以看看 actor 的结构图，actor 家族就是一个树结构。parent actor 创建和监督 child actor ，可以一直这么延伸下去，跟人类繁衍是一样的道理。
![actor结构][3]

Actor能回应接收到消息，能够自我决策，创建更多的 actor，发送更多的消息，决定如何回应下一个接收到的消息。
Actor是计算实体，它回复接收到的消息，能够并行的：
 
 1. 发生有限的消息给其他Actor  
 2. 创建有限数目的新Actor  
 3. 指定小一个消息到达时的行为

actor 不同享状态，所以不会有同步的问题，同样所有的 actor 都是异步的。( akka 也提供同步的 API 例如`Future接口`)。

actor系统的精华就是任务的分解。任务要被分割的要足够小，每个分片都会被委托给子 actor。这里用到的思想很明显就是“分而治之”，万变不离其宗啊！要想做好这一切，不仅仅要把任务组织的非常清晰（利于分解），也要对处理任务的最终 actor 在以下方面进行考虑：哪些消息需要处理、如何正常的回应、如何处理失败等等。如果一个处理特定任务的 actor 失败了，它会向它的监督者发出响应错误的消息去寻求错误处理的方法。这种递归式（监督者也有自己的监督者）的结构可以保证错误在合适的级别上进行处理。

尽量避免在 actor 间传递可变对象，这样又会陷入共享内存的并发危险中。

Actor模型的一些不足：
 
1. Actor提供了模块和封装，缺少继承和分层。 
2. 由于Actor能够动态创建其他Actor，这种行为使得系统的行为动态变化，很难控制。 
3. 行为置换（behaviour replacement）。由于行为是动态的，很难用静态语言实现。静态分享不能支持反射，运行系统的重新配置。优化困难。为了保证消息的可靠传递，需要无限制的邮箱，需要的无线堆栈在某些架构下并不能满足。 
4. 异步消息对于某些范式和算法并不适合。比如对消息顺序有严格要求的系统，虽然可以通过等待实现，但会严重降低Actor模型的效率。在OOP中，Actor会增加Actor的数量，增加系统开销。


----------


##3 Akka 
很多介绍来自与官方的文档 [akka doc][4],另外一部分来自自己的理解。
###3.1 总体介绍
Akka的主要特性如下：

**容错性**
使用“let-it-crash”语义和监管者树形结构来实现容错。非常适合编写永不停机、自愈合的高容错系统。监管者树形结构可以跨多个JVM来提供真正的高容错系统。

**位置透明性**
Akka的所有元素都为分布式环境而设计：所有actor都仅通过发送消息进行互操作，所有操作都是异步的。

**事务性actors**
事务性Actor是actor与STM(Software Transactional Memory)的组合。它使你能够使用自动重试和回滚来组合出原子消息流。

**Scala 和 Java APIs**
Akka同时提供 Scala API 和 Java API.

**两种不同的使用方式**
以库的形式：在web应用中使用，放到 WEB-INF/lib 中或者作为一个普通的Jar包放进classpath。
以微内核的形式：你可以将应用放进一个独立的内核。

**Akka平台提供哪些有竞争力的功能？**
Akka提供可扩展的实时事务处理，是一个运行时与编程模型一致的系统，为以下目标设计：

 1. 垂直扩展（并发） 
 2. 水平扩展（远程调用） 
 3. 高容错

以网友画的一张图作为 Akka 特性的总结
![Akka][5]

##3.2 Akka 关键模块之 Actors
###3.2.1 actor 的构成
actor 的构成

 - parent/children actor： 其中父actor是其创建者和监督者，子actor是从其派生
 - Actor Reference： 简单的说就是actor的地址，具体分为逻辑地址和物理地址，跟IP地址是一样的概念。逻辑地址跟HTTP URL地址的构成类似，后面会讲到.
 - Behavior(行为)：  actor 在不同的时间收到消息的处理行为可以是不同的。行为可以随着时间的变化而变化。`become`，`unbecome`中有相关的部分
 - State（状态）：actor 的状态可能来自 actor 对象中的变量。
 - Mailbox：actor 的目的就是处理消息，一个 actor 可能并发的收到很多其他 actor 发来的消息，所以 Mailbox 就是存这些消息的地方，会把很多 Sender 的消息入队 。Mailbox 默认设置是 FIFO。
 - Supervisor Strategy（监督策略）：可以定制一套对 children 错误的处理策略
 - 当 actor 被 terminated 掉时：释放所有的资源，所有的消息被发到 Sytem Dead Letters ，也会产生 event stream。同时 Mailbox 的 referce 会被 Dead Letter 的地址替代。


###3.2.2 
Supervision and Monitoring
Actor References, Paths and Addresses
Location Transparency
Message Delivery Reliability
Typed Actors
Fault Tolerance
Dispatchers
Mailboxes
Routing

###3.2.3 Building Finite State Machine Actors
Persistence
Testing Actor Systems

###3.2.4 Futures and Agents
Futures
Agents

###3.2.5 Networking
Cluster Specification
Cluster Usage
Remoting
Serialization
I/O
Using TCP
Using UDP
ZeroMQ
Utilities
Event Bus
Logging
Scheduler
Duration



  [1]: http://image18.360doc.com/DownloadImg/2010/11/2519/7102304_2.png
  [2]: http://www.360doc.com/content/10/1125/19/4154133_72389630.shtml
  [3]: http://www.kankanews.com/ICkengine/wp-content/plugins/wp-o-matic/cache/5ba7e5571f_180741-lmQE-578710.png
  [4]: http://doc.akka.io/docs/akka/snapshot/java.html
  [5]: http://static.oschina.net/uploads/space/2013/0702/161058_pgPY_578710.png