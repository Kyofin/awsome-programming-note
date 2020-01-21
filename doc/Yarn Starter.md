# Yarn Starter

## 介绍

YARN（Yet Another Resource Negotiator）是Hadoop的资源管理系统。



YARN把资源管理和任务的调度/监控拆分到了独立的进程，即**ResourceManager**（RM）和**每个程序的ApplicationMaster**，一个程序要么是一个单独的job或者是由DAG表示的多个job。

**RM和NodeManager（NM）构成了数据计算的框架**。**RM拥有最大的权利**来决断系统中每个程序所需的资源。**NM是每个机器上的一个代理，负责container的管理**，监控它们资源使用（cpu、内存、磁盘、网络），并汇报给RM。

**每个程序的ApplicationMaster是框架的特定库，负责与RM协商资源，并与NMs一起工作来执行和监控任务。**



**RM有两个组成部分：Scheduler和ApplicationsManager**（AM）。

Scheduler，在容量和队列的限制下，负责为各种程序分配资源。Scheduler只负责调度，不负责监控和跟踪程序的状态；也不为由于程序错误或硬件错误导致的任务失败提供重启保证。Scheduler基于程序的资源需求来执行调度功能；进一步说，是基于对资源的抽象，即Container（cpu、内存、磁盘、网络）来进行调度的。

Scheduler有一个可插拔策略，负责在各种队列和程序之间对集群资源进行划分。例如当前的scheduler有CapacityScheduler和FairScheduler。



**AM负责接收job的提交、协商第一个container来运行程序的ApplicationMaster**，以及为出错的ApplicationMaster container提供重启服务。每个程序的ApplicationMaster负责与Scheduler协商资源适当的的container，并追踪它们的状态和监控进度。

通过ReservationSystem，YARN还支持资源预定。



## YARN应用的运行

[![img](https://chaomai.github.io/images/2019/15585937581379.jpg)](https://chaomai.github.io/images/2019/15585937581379.jpg)



## 资源请求

YARN的资源请求模型会考虑，

- 每个容器需要的资源。
- 局部性（主要指数据的局部性）。
  例如如果使用了HDFS的数据，会优先使用存放副本的结点，其次是存有这些副本的机架，最后才是集群的任意结点。

YARN应用可以在任意时刻提出资源的申请，

- 在一开始就申请所有的资源。
- 以动态的方式，在需要更多资源的时候提出。



## 应用生命周期

按照应用的类型，应用的生命周期会有较大差异，主要分为以下3个模型，

1. 一个应用对应一个用户的job，例如MR任务。
2. 一个应用对应一个工作流或用户jobs的session，container可以在job之间复用，并cache数据，例如Spark。
3. 一个长期运行的应用被多个用户共享。这样的应用一般作为协调者的角色存在。



## YARN优势

- 可扩展性（Scalability）
  每个应用都有一个专门的application master，分离了资源调度和task管理。就MR任务而言，这模型与Google MapReduce论文中所述的模型更加接近，即，一个master协调worker上的map和reduce任务。
- 可用性（Availability）
  拆分RM和application master简化了高可用的实现。先为RM提供高可用，再为YARN应用提供高可用。
- 利用率（Utilization）
  相比MapReduce 1，精细化了资源的管理，应用可以按需请求资源。
- 多租户（Multitenancy）
  YARN支持除MapReduce外的其他分布式计算框架。



## 调度

YARN有3中调度器：FIFO调度器、容量调度器和公平调度器。

### 关于container

vcore是一个host的cpu核心占用比例。

container是，

- cpu（vcore）、内存、磁盘、网络的抽象。
- 在有task或ApplicationMaster运行的时候，表示一个已分配的资源。
- *不同*于docker中的container概念。

```
public abstract class ContainerLaunchContext {
  // ...
  /**
   * Add the list of <em>commands</em> for launching the container. All
   * pre-existing List entries are cleared before adding the new List
   * @param commands the list of <em>commands</em> for launching the container
   */
  @Public
  @Stable
  public abstract void setCommands(List<String> commands);
  // ...
}
```

### FIFO调度器（FIFO Scheduler）

按提交的顺序运行应用，首先为第一个应用分配资源，如果可以满足，再依次为其他应用服务。

### 容量调度器（Capacity Scheduler）

为每个组织分配一个专门的队列，每个队列可配置为使用一定量的集群资源，队列可以再进行划分。同一个队列内使用FIFO策略进行调度。

关于资源的使用，

- 队列中单个任务使用的资源不会超过队列的容量。
- 如果队列满，且集群有空闲的资源，调度器可以把资源分配给此队列（可配置），弹性队列。
- 正常情况下，容量调度器不会抢占容器，因此如果一个队列随着使用，资源不够时，只能等待其他队列释放资源。
  容量调度器也可以执行work-preserving preemption，RM会请求应用返回容器。

### 公平调度器（Fair Scheduler）

- 每个队列有权重元素，用于fair share的计算。
- 默认队列和动态创建的队列，权重为1（默认队列的可配置）。
- 调度器会使用最小资源数量来进行资源分配进行优先排序。如果两个队列的资源都低于fair share额度，那么远低于最小资源数量的队列，会被有限分配资源。

#### 队列放置

公平调度器使用一个规则的系统来判断应用所属队列。

#### 饥饿和抢占

FairShare的计算会被用于判断饥饿以及是否进行抢占。在计算FairShare时，有两种：

- Steady FairShare，按照配置文件中所有queue的weight，计算出的。
- Instantaneous FairShare，，按照配置文件中所有queue的weight，仅对包含活动应用程序的queue计算出的。

在配置`yarn.scheduler.fair.preemption`和`yarn.scheduler.fair.preemption.cluster-utilization-threshold`后，抢占会启用。

**饥饿**有两种：

- FairShare Starvation
  判定条件为：

  1. 未获得所要求的资源。

  2. 应用程序资源使用低于Instantaneous FairShare。

  3. 应用程序的资源使用低于fairSharePreemptionThreshold，并持续fairSharePreemptionTimeout。

     要注意的是，在同一个队列里面，如果存在多个应用程序，它们会平均的分摊Instantaneous FairShare。因此可能存在队列整体不是饥饿状态，但是每个应用程序是。

- MinShare Starvation
  判定条件为：

  1. 未获得所要求的资源。
  2. 应用程序资源使用低于MinShare。
  3. 应用程序的资源使用低于MinShare，并持续MinSharePreemptionTimeout。

决定需要进行抢占的时候，可能在多个队列中都有可抢占的container，决定container是否可以被抢占，需要满足：

- 所在队列是可抢占的。
- 杀死container以后不会导致应用程序的资源低于Instantaneous FairShare。

启用抢占**并不能**保证队列或应用程序能够获得所有的Instantaneous FairShare。只能最终保证脱离饥饿的状态，即获得fairSharePreemptionThreshold份额的资源。

FairShare Starvation、MinShare Starvation以及抢占的关系如下：

[![img](https://chaomai.github.io/images/2019/15598134204968.jpg)](https://chaomai.github.io/images/2019/15598134204968.jpg)

#### Best Practice

- 一般**不建议**配置MinShare Starvation或minimum resources。
  增加复杂性的同时，并不能带来多少好处。
- 如果配置minimum resources，所有minimum resources的加和不能超出总的资源数。



## 延迟调度

局部性是YARN调度时优先考虑的，但如果发现所请求的节点资源不够，那么任务可能就会被调度到其他节点上了。此时如果等待几秒，能够增加在所请求节点上分配到container的机会。





## Yarn资源队列配置和使用



### 前言

试想一下，你现在所在的公司有一个hadoop的集群。但是A项目组经常做一些定时的BI报表，B项目组则经常使用一些软件做一些临时需求。那么他们肯定会遇到同时提交任务的场景，这个时候到底如何分配资源满足这两个任务呢？是先执行A的任务，再执行B的任务，还是同时跑两个？

如果你存在上述的困惑，可以多了解一些yarn的资源调度器。

在Yarn框架中，调度器是一块很重要的内容。有了合适的调度规则，就可以保证多个应用可以在同一时间有条不紊的工作。最原始的调度规则就是FIFO，即按照用户提交任务的时间来决定哪个任务先执行，但是这样很可能一个大任务独占资源，其他的资源需要不断的等待。也可能一堆小任务占用资源，大任务一直无法得到适当的资源，造成饥饿。所以FIFO虽然很简单，但是并不能满足我们的需求。

### Web查看

http://localhost:8088/cluster/nodes
![在这里插入图片描述](https://img-blog.csdn.net/20181018103320238?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMzgzNDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二. 调度器的选择

在Yarn中有三种调度器可以选择：FIFO Scheduler ，Capacity Scheduler，FairS cheduler。

### FIFO Scheduler

FIFO Scheduler把应用按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。

FIFO Scheduler是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。大的应用可能会占用所有集群资源，这就导致其它应用被阻塞。在共享集群中，更适合采用Capacity Scheduler或Fair Scheduler，这两个调度器都允许大任务和小任务在提交的同时获得一定的系统资源。

下面“Yarn调度器对比图”展示了这几个调度器的区别，从图中可以看出，在FIFO 调度器中，小任务会被大任务阻塞。

### Capacity Scheduler

而对于Capacity调度器，有一个专门的队列用来运行小任务，但是为小任务专门设置一个队列会预先占用一定的集群资源，这就导致大任务的执行时间会落后于使用FIFO调度器时的时间。

### FairS cheduler

在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。如下图所示，当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。

需要注意的是，在下图Fair调度器中，从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终的效果就是Fair调度器即得到了高的资源利用率又能保证小任务及时完成。
![在这里插入图片描述](https://img-blog.csdn.net/20180912140209122?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxMzgzNDM1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 三. capacity调度器

### 3.1 什么是capacity调度器

Capacity 调度器允许多个组织共享整个集群，每个组织可以获得集群的一部分计算能力。通过为每个组织分配专门的队列，然后再为每个队列分配一定的集群资源，这样整个集群就可以通过设置多个队列的方式给多个组织提供服务了。除此之外，队列内部又可以垂直划分，这样一个组织内部的多个成员就可以共享这个队列资源了，在一个队列内部，资源的调度是采用的是先进先出(FIFO)策略。

通过上面那幅图，我们已经知道一个job可能使用不了整个队列的资源。然而如果这个队列中运行多个job，如果这个队列的资源够用，那么就分配给这些job，如果这个队列的资源不够用了呢？其实Capacity调度器仍可能分配额外的资源给这个队列，这就是“弹性队列”(queue elasticity)的概念。

在正常的操作中，Capacity调度器不会强制释放Container，当一个队列资源不够用时，这个队列只能获得其它队列释放后的Container资源。当然，我们可以为队列设置一个最大资源使用量，以免这个队列过多的占用空闲资源，导致其它队列无法使用这些空闲资源，这就是”弹性队列”需要权衡的地方。

Capacity调度器说的通俗点，可以理解成一个个的资源队列。这个资源队列是用户自己去分配的。比如我大体上把整个集群分成了AB两个队列，A队列给A项目组的人来使用。B队列给B项目组来使用。但是A项目组下面又有两个方向，那么还可以继续分，比如专门做BI的和做实时分析的。那么队列的分配就可以参考下面的树形结构

```
root
------a[60%]
      |---a.bi[40%]
      |---a.realtime[60%]
------b[40%]
12345
```

a队列占用整个资源的60%，b队列占用整个资源的40%。a队列里面又分了两个子队列，一样也是2:3分配。

虽然有了这样的资源分配，但是并不是说a提交了任务，它就只能使用60%的资源，那40%就空闲着。只要资源实在空闲状态，那么a就可以使用100%的资源。但是一旦b提交了任务，a就需要在释放资源后，把资源还给b队列，直到ab平衡在3:2的比例。

粗粒度上资源是按照上面的方式进行，在每个队列的内部，还是按照FIFO的原则来分配资源的。



## 常用命令

杀掉应用。

```
 yarn application -kill jobId
```

