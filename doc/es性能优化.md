# ES性能优化

保证es的内存大小等于50% 的RAM 大小。你也应该把ElasticSearch 最小和最大的内存池设置为**一样的值**



在`elasticsearch.yml` 文件中有一些调优的事情我们可以做，可以极大的改进重写的节点的性能。首先是设置`bootstrap.mlockall` 为 true。这会强制JVM 立即分配所有的`ES_MIN_MEM`。这意味着Java 在启动时就有了所有所需的内存



ElasticSearch 是假定你将大量使用搜索，所以分配了大部分的内存用于保障搜索。这不是我的集群的使用情况，我需要大量写入，所以调整`indices.memory.index_buffer_size`到50%，从而恢复在我们使用方式下内存分配的平衡。在我的设置里，我还提高了刷新间隔和日志刷新的传输总数。否则，ElasticSearch 将需要接近每分钟都刷新传输日志。



另外一个需要我们调整以避免灾难性的失败的地方是线程池的设置。

多节点高速写入时会导致bulk线程池塞满，就写入失败了。

查看bulk线程池使用情况

```
GET _cat/thread_pool?v
```

![image-20190324014850574](/Users/huzekang/Library/Application Support/typora-user-images/image-20190324014850574.png)

可以看出我的slave1节点在最新的这次任务中新增了8个拒绝的。是因为多数文档都被写入到slave1了。我的节点分片情况如图

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190324015131.png)

查看目前es的线程池设置情况

```
GET /_nodes
```



核心线程池

- indexing普通的索引请求的线程池

- bulk批量请求，和单条的索引请求不同的线程池

- get  Get-by-ID 操作

- search所有的搜索和查询请求

- merging专用于管理 Lucene 合并的线程池



线程池类型

1、cache

无限制的线程池，为每个请求创建一个线程

2、fixed

有着固定大小的线程池，大小由size属性指定，允许你指定一个队列（使用queue_size属性指定）用来保存请求，直到有一个空闲的线程来执行请求。如果Elasticsearch无法把请求放到队列中（队列满了），该请求将被拒绝

其中bulk线程池设置如下：

![](https://raw.githubusercontent.com/peter1040080742/picbed/master/20190325100835.png)





ElasticSearch 会做它认为能获得最好性能的事情。我们发现，在生产环境中，这意味着生成成千上万的线程来处理传入的请求。在高负载下，你的集群很快会被压垮。为了避免这种情况，我们分别设置了search,index 和bulk 线程池的最大值。我们主要的操作是bulk，所以给bulk 设置了60个线程，其他的操作设置了20个线程。

```json
##################################################################
# /etc/elasticsearch/elasticsearch.yml


## Threadpool Settings ##

# Search pool

thread_pool.search.size: 13
thread_pool.search.queue_size: 300

# Bulk pool
thread_pool.bulk.size: 9
thread_pool.bulk.queue_size: 2000

# Index pool
thread_pool.index.size: 8
thread_pool.index.queue_size: 300

# Indices settings

indices.memory.index_buffer_size: 30%


# Indexing Settings 
#Since elasticsearch 5.x index level settings can NOT be set on the nodes 
#configuration like the elasticsearch.yaml, in system properties or command line 
#arguments.In order to upgrade all indices the settings must be updated via the 
#/${index}/_settings API. Unless all settings are dynamic all indices must be closed 
#in order to apply the upgradeIndices created in the future should use index templates 
#to set default values. 

#Please ensure all required values are updated on all indices by executing: 

#curl -XPUT 'http://localhost:9200/_all/_settings?preserve_existing=true' -d '{
#  "index.refresh_interval" : "30s",
#  "index.translog.flush_threshold_ops" : "50000"
#}'

#index.refresh_interval: 30s
#index.translog.flush_threshold_ops: 50000

# Minimum nodes alive to constitute an operational cluster
# 尽量避免脑裂，需要添加最小数量的主节点配置
discovery.zen.minimum_master_nodes: 2

# Unicast Discovery (disable multicast)
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: [ "logsearch-01", "logsearch-02", "logsearch-03" ]
```



### 线程池官方说明

****

许多人 *喜欢* 调整线程池。 无论什么原因，人们都对增加线程数无法抵抗。索引太多了？增加线程！搜索太多了？增加线程！节点空闲率低于 95％？增加线程！

Elasticsearch 默认的线程设置已经是很合理的了。对于所有的线程池（除了 `搜索` ），线程个数是根据 CPU 核心数设置的。 如果你有 8 个核，你可以同时运行的只有 8 个线程，只分配 8 个线程给任何特定的线程池是有道理的。

搜索线程池设置的大一点，配置为 `int（（ 核心数 ＊ 3 ）／ 2 ）＋ 1` 。

你可能会认为某些线程可能会阻塞（如磁盘上的 I／O 操作），所以你才想加大线程的。对于 Elasticsearch 来说这并不是一个问题：因为大多数 I／O 的操作是由 Lucene 线程管理的，而不是 Elasticsearch。

此外，线程池通过传递彼此之间的工作配合。你不必再因为它正在等待磁盘写操作而担心网络线程阻塞， 因为网络线程早已把这个工作交给另外的线程池，并且网络进行了响应。

最后，你的处理器的计算能力是有限的，拥有更多的线程会导致你的处理器频繁切换线程上下文。 一个处理器同时只能运行一个线程。所以当它需要切换到其它不同的线程的时候，它会存储当前的状态（寄存器等等），然后加载另外一个线程。 如果幸运的话，这个切换发生在同一个核心，如果不幸的话，这个切换可能发生在不同的核心，这就需要在内核间总线上进行传输。

这个上下文的切换，会给 CPU 时钟周期带来管理调度的开销；在现代的 CPUs 上，开销估计高达 30 μs。也就是说线程会被堵塞超过 30 μs，如果这个时间用于线程的运行，极有可能早就结束了。

人们经常稀里糊涂的设置线程池的值。8 个核的 CPU，我们遇到过有人配了 60、100 甚至 1000 个线程。 这些设置只会让 CPU 实际工作效率更低。

所以，下次请不要调整线程池的线程数。如果你真 *想调整* ， 一定要关注你的 CPU 核心数，最多设置成核心数的两倍，再多了都是浪费。



**批量操作的被拒绝数**

如果你碰到了队列被拒，一般来说都是批量索引请求导致的。 通过并发导入程序发送大量批量请求非常简单。越多越好嘛，对不？

事实上，每个集群都有它能处理的请求上限。一旦这个阈值被超过，队列会很快塞满，然后新的批量请求就被拒绝了。

这是一件 _好事情_ 。队列的拒绝在回压方面是有用的。它们让你知道你的集群已经在最大容量了。这比把数据塞进内存队列要来得好。增加队列大小并不能增加性能，它只是隐藏了问题。当你的集群只能每秒钟处理 10000 个文档的时候，无论队列是 100 还是 10000000 都没关系——你的集群还是只能每秒处理 10000 个文档。

队列只是隐藏了性能问题，而且带来的是真实的数据丢失的风险。在队列里的数据都是还没处理的，如果节点挂掉，这些请求都会永久的丢失。此外，队列还要消耗大量内存，这也是不理想的。

在你的应用中，优雅的处理来自满载队列的回压，才是更好的选择。当你收到拒绝响应的时候，你应该采取如下几步：

1. 暂停导入线程 3–5 秒。
2. 从批量操作的响应里提取出来被拒绝的操作。因为可能很多操作还是成功的。响应会告诉你哪些成功，哪些被拒绝了。
3. 发送一个新的批量请求，只包含这些被拒绝过的操作。
4. 如果依然碰到拒绝，再次从步骤 1 开始。

通过这个流程，你的代码可以很自然的适应你集群的负载，做到自动回压。

拒绝不是错误：它们只是意味着你要稍后重试。



# ES官方调优指南翻译

# Table of Contents

1. [第一部分：调优索引速度](http://wangnan.tech/post/elasticsearch-how-to/#%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86%EF%BC%9A%E8%B0%83%E4%BC%98%E7%B4%A2%E5%BC%95%E9%80%9F%E5%BA%A6)
2. [第二部分-调优搜索速度](http://wangnan.tech/post/elasticsearch-how-to/#%E7%AC%AC%E4%BA%8C%E9%83%A8%E5%88%86-%E8%B0%83%E4%BC%98%E6%90%9C%E7%B4%A2%E9%80%9F%E5%BA%A6)
3. [第三部分：通用的一些建议](http://wangnan.tech/post/elasticsearch-how-to/#%E7%AC%AC%E4%B8%89%E9%83%A8%E5%88%86%EF%BC%9A%E9%80%9A%E7%94%A8%E7%9A%84%E4%B8%80%E4%BA%9B%E5%BB%BA%E8%AE%AE)



ES发布时带有的默认值，可为es的开箱即用带来很好的体验。全文搜索、高亮、聚合、索引文档 等功能无需用户修改即可使用,当你更清楚的知道你想如何使用es后，你可以作很多的优化以提高你的用例的性能,下面的内容告诉你 你应该/不应该 修改哪些配置

# 第一部分：调优索引速度

（[https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html）](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html%EF%BC%89)

1. 使用批量请求批量请求将产生比单文档索引请求好得多的性能。

为了知道批量请求的最佳大小，您应该在具有单个分片的单个节点上运行基准测试。 首先尝试索引100个文件，然后是200，然后是400，等等。 当索引速度开始稳定时，您知道您达到了数据批量请求的最佳大小。 在配合的情况下，最好在太少而不是太多文件的方向上犯错。 请注意，如果群集请求太大，可能会使群集受到内存压力，因此建议避免超出每个请求几十兆字节，即使较大的请求看起来效果更好。

1. 发送端使用多worker/多线程向es发送数据
   发送批量请求的单个线程不太可能将Elasticsearch群集的索引容量最大化。 为了使用集群的所有资源，您应该从多个线程或进程发送数据。 除了更好地利用集群的资源，这应该有助于降低每个fsync的成本。

请确保注意TOO_MANY_REQUESTS（429）响应代码（Java客户端的EsRejectedExecutionException），这是Elasticsearch告诉您无法跟上当前索引速率的方式。 发生这种情况时，应该再次尝试暂停索引，理想情况下使用随机指数回退。

与批量调整大小请求类似，只有测试才能确定最佳的worker数量。 这可以通过逐渐增加工作者数量来测试，直到集群上的I / O或CPU饱和。

1. 调大 refresh interval
   默认的index.refresh_interval是1s，这迫使Elasticsearch每秒创建一个新的分段。 增加这个价值（比如说30s）将允许更大的部分flush并减少未来的合并压力。
2. 加载大量数据时禁用refresh和replicas
   如果您需要一次加载大量数据，则应该将index.refresh_interval设置为-1并将index.number_of_replicas设置为0来禁用刷新。这会暂时使您的索引处于危险之中，因为任何分片的丢失都将导致数据 丢失，但是同时索引将会更快，因为文档只被索引一次。 初始加载完成后，您可以将index.refresh_interval和index.number_of_replicas设置回其原始值。
3. 设置参数，禁止OS将es进程swap出去
   您应该确保操作系统不会swapping out the java进程，通过禁止swap
   （[https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html）](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html%EF%BC%89)
4. 为filesystem cache分配一半的物理内存
   文件系统缓存将用于缓冲I / O操作。 您应该确保将运行Elasticsearch的计算机的内存至少减少到文件系统缓存的一半。
5. 使用自动生成的id（auto-generated ids）
   索引具有显式id的文档时，Elasticsearch需要检查具有相同id的文档是否已经存在于相同的分片中，这是昂贵的操作，并且随着索引增长而变得更加昂贵。 通过使用自动生成的ID，Elasticsearch可以跳过这个检查，这使索引更快。
6. 买更好的硬件
   搜索一般是I/O 密集的，此时，你需要
   a.为filesystem cache分配更多的内存
   b.使用SSD硬盘
   c.使用local storage（不要使用NFS、SMB 等remote filesystem）
   d.亚马逊的 弹性块存储（Elastic Block Storage）也是极好的，当然，和local storage比起来，它还是要慢点
   如果你的搜索是 CPU-密集的，买好的CPU吧
7. 加大 indexing buffer size
   如果你的节点只做大量的索引，确保index.memory.index_buffer_size足够大，每个分区最多可以提供512 MB的索引缓冲区，而且索引的性能通常不会提高。 Elasticsearch采用该设置（java堆的一个百分比或绝对字节大小），并将其用作所有活动分片的共享缓冲区。 非常活跃的碎片自然会使用这个缓冲区，而不是执行轻量级索引的碎片。

默认值是10％，通常很多：例如，如果你给JVM 10GB的内存，它会给索引缓冲区1GB，这足以承载两个索引很重的分片。

1. 禁用_field_names字段
   _field_names字段引入了一些索引时间开销，所以如果您不需要运行存在查询，您可能需要禁用它。
   （_field_names：[https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-field-names-field.html）](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-field-names-field.html%EF%BC%89)
2. 剩下的，再去看看 “调优 磁盘使用”吧
   （[https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-disk-usage.html）中有许多磁盘使用策略也提高了索引速度。](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-disk-usage.html%EF%BC%89%E4%B8%AD%E6%9C%89%E8%AE%B8%E5%A4%9A%E7%A3%81%E7%9B%98%E4%BD%BF%E7%94%A8%E7%AD%96%E7%95%A5%E4%B9%9F%E6%8F%90%E9%AB%98%E4%BA%86%E7%B4%A2%E5%BC%95%E9%80%9F%E5%BA%A6%E3%80%82)

# 第二部分-调优搜索速度

1. filesystem cache越大越好
   为了使得搜索速度更快， es严重依赖filesystem cache
   一般来说，需要至少一半的 可用内存 作为filesystem cache，这样es可以在物理内存中 保有 索引的热点区域（hot regions of the index）
2. 用更好的硬件
   搜索一般是I/O bound的，此时，你需要
   a.为filesystem cache分配更多的内存
   b.使用SSD硬盘
   c.使用local storage（不要使用NFS、SMB 等remote filesystem）
   d.亚马逊的 弹性块存储（Elastic Block Storage）也是极好的，当然，和local storage比起来，它还是要慢点
   如果你的搜索是 CPU-bound，买好的CPU吧
3. 文档模型（document modeling）
   文档需要使用合适的类型，从而使得 search-time operations 消耗更少的资源。咋作呢？
   答：避免 join操作。具体是指
   a.nested 会使得查询慢 好几倍
   b.parent-child关系 更是使得查询慢几百倍
   如果 无需join 能解决问题，则查询速度会快很多
4. 预索引 数据
   根据“搜索数据最常用的方式”来最优化索引数据的方式
   举个例子：
   所有文档都有price字段，大部分query 在 fixed ranges 上运行 range aggregation。你可以把给定范围的数据 预先索引下。然后，使用 terms aggregation
5. Mappings（能用 keyword 最好了）
   数字类型的数据，并不意味着一定非得使用numeric类型的字段。
   一般来说，存储标识符的 字段（书号ISBN、或来自数据库的 标识一条记录的 数字），使用keyword更好（integer，long 不好哦，亲）
   6.避免运行脚本
   一般来说，脚本应该避免。 如果他们是绝对需要的，你应该使用painless和expressions引擎。
6. 搜索rounded 日期
   日期字段上使用now，一般来说不会被缓存。但，rounded date则可以利用上query cache
   rounded到分钟等
7. 强制merge只读的index
   只读的index可以从“merge成 一个单独的 大segment”中收益
8. 预热 全局序数（global ordinals）
   全局序数 用于 在 keyword字段上 运行 terms aggregations
   es不知道 哪些fields 将 用于/不用于 term aggregation，因此 全局序数 在需要时才加载进内存
   但，可以在mapping type上，定义 eager_global_ordinals==true，这样，refresh时就会加载 全局序数
9. 预热 filesystem cache
   机器重启时，filesystem cache就被清空。OS将index的热点区域（hot regions of the index）加载进filesystem cache是需要花费一段时间的。
   设置 index.store.preload 可以告知OS 这些文件需要提早加载进入内存

11使用索引排序来加速连接
索引排序对于以较慢的索引为代价来加快连接速度非常有用。在索引分类文档中阅读更多关于它的信息。

12.使用preference来优化高速缓存利用率
有多个缓存可以帮助提高搜索性能，例如文件系统缓存，请求缓存或查询缓存。然而，所有这些缓存都维护在节点级别，这意味着如果连续运行两次相同的请求，则有一个或多个副本，并使用循环（默认路由算法），那么这两个请求将转到不同的分片副本，阻止节点级别的缓存帮助。

由于搜索应用程序的用户一个接一个地运行类似的请求是常见的，例如为了分析索引的较窄的子集，使用标识当前用户或会话的优选值可以帮助优化高速缓存的使用。

13.副本可能有助于吞吐量，但不会一直存在
除了提高弹性外，副本可以帮助提高吞吐量。例如，如果您有单个分片索引和三个节点，则需要将副本数设置为2，以便共有3个分片副本，以便使用所有节点。

现在假设你有一个2-shards索引和两个节点。在一种情况下，副本的数量是0，这意味着每个节点拥有一个分片。在第二种情况下，副本的数量是1，这意味着每个节点都有两个碎片。哪个设置在搜索性能方面表现最好？通常情况下，每个节点的碎片数少的设置将会更好。原因在于它将可用文件系统缓存的份额提高到了每个碎片，而文件系统缓存可能是Elasticsearch的1号性能因子。同时，要注意，没有副本的设置在发生单个节点故障的情况下会出现故障，因此在吞吐量和可用性之间进行权衡。

那么复制品的数量是多少？如果您有一个具有num_nodes节点的群集，那么num_primaries总共是主分片，如果您希望能够一次处理max_failures节点故障，那么正确的副本数是max（max_failures，ceil（num_nodes / num_primaries） - 1）。

14.打开自适应副本选择
当存在多个数据副本时，elasticsearch可以使用一组称为自适应副本选择的标准，根据包含分片的每个副本的节点的响应时间，服务时间和队列大小来选择数据的最佳副本。这可以提高查询吞吐量并减少搜索量大的应用程序的延迟。

# 第三部分：通用的一些建议

1、不要 返回大的结果集
es设计来作为搜索引擎，它非常擅长返回匹配query的top n文档。但，如“返回满足某个query的 所有文档”等数据库领域的工作，并不是es最擅长的领域。如果你确实需要返回所有文档，你可以使用Scroll API

2、避免 大的doc。即，单个doc 小了 会更好
given that(考虑到) http.max_context_length默认==100MB，es拒绝索引操作100MB的文档。当然你可以提高这个限制，但，Lucene本身也有限制的，其为2GB
即使不考虑上面的限制，大的doc 会给 network/memory/disk带来更大的压力；
a.任何搜索请求，都需要获取 _id 字段，由于filesystem cache工作方式。即使它不请求 _source字段，获取大doc _id 字段消耗更大
b.索引大doc时消耗内存会是 doc本身大小 的好几倍
c.大doc的 proximity search, highlighting 也更加昂贵。它们的消耗直接取决于doc本身的大小

3、避免 稀疏
a.不相关数据 不要 放入同一个索引
b.一般化文档结构（Normalize document structures）
c.避免类型
d.在 稀疏 字段上，禁用 norms & doc_values 属性

稀疏为什么不好？
Lucene背后的数据结构 更擅长处理 紧凑的数据
text类型的字段，norms默认开启；numerics, date, ip, keyword，doc_values默认开启
Lucene内部使用 integer的doc_id来标识文档 和 内部API交互。
举个例子：
使用match查询时生成doc_id的迭代器，这些doc_id被用于获取它们的norm，以便计算score。当前的实现是每个doc中保留一个byte用于存储norm值。获取norm值其实就是读取doc_id位置处的一个字节
这非常高效，Lucene通过此值可以快速访问任何一个doc的norm值；但，给定一个doc，即使某个field没有值，仍需要为此doc的此field保留一个字节
doc_values也有同样的问题。2.0之前的fielddata被现在的doc_values所替代了。
稀疏性 最明显的影响是 对存储的需求（任何doc的每个field，都需要一个byte）；但是呢，稀疏性 对 索引速度和查询速度 也是有影响的，因为：即使doc并没有某些字段值，但，索引时，依然需要写这些字段，查询时，需要skip这些字段的值
某个索引中拥有少量稀疏字段，这完全没有问题。但，这不应该成为常态
稀疏性影响最大的是 norms&doc_values ，但，倒排索引（用于索引 text以及keyword字段），二维点（用于索引geo_point字段）也会受到较小的影响

如何避免稀疏呢？
1、不相关数据 不要 放入同一个索引
给个tip：索引小（即：doc的个数较少），则，primary shard也要少
2、一般化文档结构（Normalize document structures）
3、避免类型（Avoid mapping type）
同一个index，最好就一个mapping type
在同一个index下面，使用不同的mapping type来存储数据，听起来不错，但，其实不好。given that(考虑到)每一个mapping type会把数据存入 同一个index，因此，多个不同mapping type，各个的field又互不相同，这同样带来了稀疏性 问题
4、在 稀疏 字段上，禁用 norms & doc_values 属性
a.norms用于计算score，无需score，则可以禁用它（所有filtering字段，都可以禁用norms）
b.doc_vlaues用于sort&aggregations，无需这两个，则可以禁用它
但是，不要轻率的做出决定，因为 norms&doc_values无法修改。只能reindex

秘诀1：混合 精确查询和提取词干（mixing exact search with stemming）
对于搜索应用，提取词干（stemming）都是必须的。例如：查询 skiing时，ski和skis都是期望的结果
但，如果用户就是要查询skiing呢？
解决方法是：使用multi-field。同一份内容，以两种不同的方式来索引存储
query.simple_query_string.quote_field_suffix，竟然是 查询完全匹配的

秘诀2：获取一致性的打分
score不能重现
同一个请求，连续运行2次，但，两次返回的文档顺序不一致。这是相当坏的用户体验

如果存在 replica，则就可能发生这种事，这是因为：
search时，replication group中的shard是按round-robin方式来选择的，因此两次运行同样的请求，请求如果打到 replication group中的不同shard，则两次得分就可能不一致

那问题来了，“你不是整天说 primary和replica是in-sync的，是完全一致的”嘛，为啥打到“in-sync的，完全一致的shard”却算出不同的得分？

原因就是标注为“已删除”的文档。如你所知，doc更新或删除时，旧doc并不删除，而是标注为“已删除”，只有等到 旧doc所在的segment被merge时，“已删除”的doc才会从磁盘删除掉

索引统计（index statistic）是打分时非常重要的一部分，但，由于 deleted doc 的存在，在同一个shard的不同copy（即：各个replica）上 计算出的 索引统计 并不一致

个人理解：
a. 所谓 索引统计 应该就是df，即 doc_freq
b. 索引统计 是基于shard来计算的

1. 搜索时，“已删除”的doc 当然是 永远不会 出现在 结果集中的
2. 索引统计时，for practical reasons，“已删除”doc 依然是统计在内的

假设，shard A0 刚刚完成了一次较大的segment merge，然后移除了很多“已删除”doc，shard A1 尚未执行 segment merge，因此 A1 依然存在那些“已删除”doc

于是：两次请求打到 A0 和 A1 时，两者的 索引统计 是显著不同的

如何规避 score不能重现 的问题？使用 preference 查询参数
发出搜索请求时候，用 标识字符串 来标识用户，将 标识字符串 作为查询请求的preference参数。这确保多次执行同一个请求时候，给定用户的请求总是达到同一个shard，因此得分会更为一致（当然，即使同一个shard，两次请求 跨了 segment merge，则依然会得分不一致）
这个方式还有另外一个优点，当两个doc得分一致时，则默认按着doc的 内部Lucene doc id 来排序（注意：这并不是es中的 _id 或 _uid）。但是呢，shard的不同copy间，同一个doc的 内部Lucene doc id 可能并不相同。因此，如果总是达到同一个shard，则，具有相同得分的两个doc，其顺序是一致的

score错了
score错了（Relevancy looks wrong）
如果你发现

1. 具有相同内容的文档，其得分不同
2. 完全匹配 的查询 并没有排在第一位
   这可能都是由 sharding 引起的
3. 默认情况下，搜索文档时，每个shard自己计算出自己的得分。
4. 索引统计 又是打分时一个非常重要的因素。

如果每个shard的 索引统计相似，则 搜索工作的很好
文档是平分到每个primary shard的，因此 索引统计 会非常相似，打分也会按着预期工作。但，万事都有个但是：

1. 索引时使用了 routing（文档不能平分到每个primary shard 啦）
2. 查询多个索引
3. 索引中文档的个数 非常少
   这会导致：参与查询的各个shard，各自的 索引统计 并不相似（而，索引统计对 最终的得分 又影响巨大），于是 打分出错了（relevancy looks wrong）

那，如何绕过 score错了（Relevancy looks wrong）？

如果数据集较小，则，只使用一个primary shard（es默认是5个），这样两次查询 索引统计 不会变化，因而得分也就一致啦
另一种方式是，将search_type设置为：dfs_query_then_fetech（默认是query_then_fetch）
dfs_query_then_fetch的作用是

1. 向 所有相关shard 发出请求，要求 所有相关shard 返回针对当前查询的 索引统计
2. 然后，coordinating node 将 merge这些 索引统计，从而得到 merged statistics
3. coordinating node 要求 所有相关shard 执行 query phase，于是 发出请求，这时，也带上 merged statistics。这样，执行query的shard 将使用 全局的索引统计
   大部分情况下，要求 所有相关shard 返回针对当前查询的 索引统计，这是非常cheap的。但，如果查询中 包含 非常大量的 字段/term查询，或者有 fuzzy查询，此时，获取 索引统计 可能并不cheap，因为 为了得到 索引统计 可能 term dictionary 中 所有的term都需要被查询一遍