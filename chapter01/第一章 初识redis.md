# Redis实战
>我学习的是两本书一本是《Redis设计与实现》这个偏向于Redis用法的底层角度进行讲解，另一本是Redis实战这个偏向于理论加实践角度进行讲解。
## 第一章 初识Redis
**1.1Redis简介**

Redis是一个速度非常快的非关系数据库，他可以存储键(key)与其他不同类型的值(value)之间的映射(mapping)，可以使用客户端分片来扩展写性能。

**1.1.1 redis与其他数据库和软件的对比**

由于是初次学习非关系型数据库的知识我需要先了解什么是关系型数据库什么是非关系型数据库。<br>
****(1) 关系型数据库****
    
***概念***

关系型数据库，是指采用了关系模型来阻止数据的数据库。关系模型指的就是二维表格模型，因此一个关系型数据库就是有二维表及其之间的联系所组成的一个数据组织。<br>
目前十大主流的关系型数据库：MySQL、Microsoft SQL Server、Oracle、SQLite、MariaDB(MySQL的分支)、PostgreSQL、Microsoft Access、Teradata、SAP。

***结构***

* 关系：可以理解为一张二维表，每个关系都具有一个关系名，就是通常说的表名。<br>
* 元组：可以理解为二维表中的一行，在数据库中经常被称为记录。<br>
* 属性：可以理解为二维表中的一列，在数据库中经常被称为字段。<br>
* 域：属性的取值范围，也就是数据库中某一列的取值限制。<br>
* 关键字：一组可以唯一标识元组的属性，数据库中常称为主键，由一个或多个列组成。<br>
* 关系模式：指对关系的描述。其格式为：关系名(属性1，属性2， … … ，属性N)，在数据库中成为表结构。

***优点***

* 容易理解：二维表结构是非常贴近逻辑世界的一个概念，关系模型相对网状、层次等其他模型来说更容易理解。<br>
* 使用方便：通用的SQL语言使得操作关系型数据库非常方便。<br>
* 易于维护：丰富的完整性（实体完整性、参照完整性和用户定义的完整性）大大减低了数据冗余和数据不一致的概率。（表结构易于维护）

***瓶颈***

* 高并发读写需求：网站的用户并发性非常高，往往达到每秒上万次读写请求，对于传统关系型数据库来说，硬盘I/O是一个很大的瓶颈。<br>
* 海量数据的高效读写：网站每天产生的数据量是巨大的，对于关系型数据库来说，在一张包含海量数据的表中查询，效率是非常低的。<br>
* 高扩展性和可用性：在基于web的结构中，数据库是最难进行横向扩展的，当一个应用系统的用户量和访问量与日俱增的时候，数据库却没有办法想web server和app server那样简单的通过添加更多的硬件和服务节点来扩展性能和负载能力。对于很多需要提供24小时不间断服务的网站来说，对数据库系统进行升级和扩展是非常痛苦的事情，往往需要停机维护和数据迁移。

***非必要性功能***

（1）对于网站来说，关系型数据库的许多特性不再需要了：
* 事务一致性：关系型书记库针对事务一致性的维护中有很大的开销，而现在很多web2.0系统对事物的读写一致性不高。<br>
* 读写实时性：对关系数据库来说，插入一条数据之后立即查询，是肯定读出这条数据的，但是对于关系型数据库来说，在一张包含海量数据的表中查询，效率是非常低的。<br>
* 复杂SQL，特别是多表关联查询：任何大数据量的web系统，都非常忌讳多个大表的关联拆线呢，以及复杂的数据分析类型的复杂SQL报表查询，特别是SNS类型网站（SNS，专指社交网络服务，包含了社交软件和社交网站），从需求以及产品阶级角度，就避免了这种情况的产生。往往更多的只是但表的主键查询，以及单表的简单条件分页查询，SQL的功能极大的弱化了。

（2）在关系型数据库中，导致性能欠佳的主要原因是多表的关联查询，以及复杂的数据分析类型的复杂SQL报表查询。为了保证数据库的ACID特性，我们必须尽量按照其要求的范式进行设计，关系型数据库中的表都是存储一个格式化的数据结构。每个元组字段的组成都是一样，即便不是每个元组都需要所有的字段，但数据库会为每个元组分配所有的字段，这样的结构可以便于标语表之间进行连接等操作，但从另一个角度来说他也是关系型数据库性能瓶颈的一个因素。
>注:数据库事务必须具备ACID特性，ACID是Atomic原子性，Consistency一致性，Isolation隔离性，Durability持久性。<br>
>关系型数据库中的一条记录中有若干个属性，若其中某一个属性组(注意是组)能唯一标识一条记录，该属性组就可以成为一个主键<br> 
比如<br> 
学生表(学号，姓名，性别，班级) <br> 
其中每个学生的学号是唯一的，学号就是一个主键 <br> 
对于逐渐与外键的关系详情参考：https://blog.csdn.net/bingqingsuimeng/article/details/51595560?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task
---
****(2) 非关系型数据库****

***概念***

NoSQL最初指的是一个没有SQL功能，轻量级的，开源的关系型数据库。随着时代需求的发展，我们要的不是“no sql”，而是“no relational”，也就是我们现在常说的非关系数据库。2009年明确NoSQL,用于指代那些非关系型的，分布式的，且一般不保证遵循ACID原则的数据存储系统。非关系型数据库的理念，例如，以键值对存储，且结构不固定，每一个元组可以有不一样的字段，每个元组可以根据需要增加一些自己的键值对，这样就不会局限于固定的结构，可以减少一些时间与空间的开销。使用这种方式，用户可以根据需要去添加自己需要的字段，这样，为了获取用户的不同信息，不需要像关系型数据库中，要对多表进行关联查询。仅需要根据key取出相应的value就可以完成查询。但非关系型数据库由于很少的约束，它也不能够提供像SQL所提供的where这种对于字段属性值情况的查询。并且难以体现设计的完整性，它只适合一些较为简单的数据，对于需要进行恢复查询的数据，SQL数据库显得更为合适。<br>
非关系型数据库严格意义上，不是一种数据库，而应该是一种**数据结构化存储方式的集合**，可以是文档或者键值对等。

***分类***

非关系型数据库，都是针对某些特定的应用需求出现的，因此对于特定应用，专门的非关系型数据库会有极高的性能。依据结构化方法以及应用场合的不用，只要分为以下几类：<br>
* 面向高性能并发读写的key-value数据库：以键值对存储数据的一种数据库，类似Java的map，整个数据库就像一个map，每个键都会对应一个唯一的值。该数据库的特点是，就有极高的并发读写性能。经典代表为Redis、Tokyo Cabinet、Flare、Amazon DynamoDB、Memcached、Microsoft Azure Cosmos DB、Hazelcast。
* 面向海量数据访问的面向文档数据库：该类数据库的特点是，可以在海量数据中快速的查询数据。文档存储通常使用内部表示法，可以直接在应用程序中处理，主要是json，而json文档也可以作为纯文本存储或关系数据库系统中。典型代表为MongoDB、Amazon DynamoDB、Couchbase、Microsoft Azure Cosmos DB、CouchDB。
* 面向可扩展性的分布式数据库：普通的关系型数据库都是以行为单位来存储数据的，擅长以行为单位的读入处理，比如特定条件数据的获取。因此，关系型数据可也被称为面向行的数据库。相反，面向列的数据库都是以列为单位来存储数据的，擅长以列为单位读入数据。该类数据库想解决的问题就是传统数据库存在可扩展性上的缺陷，该类数据库可以适应数据量的增加以及数据结构的变化，将数据存储在记录中，能够容纳大量动态类。由于列名和记录键不是固定的，并且由于记录可能有数十亿列，因此可扩展性存储可以看作是二位键值存储。典型代表为Cassandra、HBase、Microsoft Azure Cosmos DB、Datastax Enterprise、Accumulo。
* 面型搜索数据内容的搜索引擎数据库：专门用于搜索内容。主要是用于海量数据进行接近实时的处理和分析处理，可用于机器学习和数据挖掘。典型代表为Elasticsearch、Splunk、Solr、MarkLogic、Sphinx。

![fig.1](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/1721334-20190702155425039-2038506966.png)
![fig.2](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/1721334-20190702155519820-338354367.png)
![fig.3](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/1721334-20190702155535759-490036526.png)
![fig.4](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/1721334-20190702155543217-1929236480.png)

***优点***

* 格式灵活：存储数据的格式可以是key-value形式、文档形式、图片形式等等，使用灵活，应用场景广泛，而关系型数据库则只支持基础类型。
* 速度快：NoSQL可以使用硬盘或者随机存储器作为载体，二关系型数据库只能使用硬盘。
* 高扩展性。
* 成本低：NoSQL数据库部署简单，基本都是开源软件。

***缺陷***

* 不支持SQL语句，学习和使用成本较高。
* 无事务处理。
* 数据结构相对复杂，复杂查询方面稍欠。
---
***CAP理论***

NoSQL的基本需求就是支持分布式存储，严格一致性与可用性需要互相取舍。<br>
CAP理论：一个分布式系统不可能同时满足C(一致性)、A(可用性)、P(分区容错性)三个基本需求，并且最多只能满足其中的两项。对于一个分布式系统来说，分区容错是基本需求，否则不能称之为分布式系统，因此需要在C和A之间寻求平衡。<br>
C(Consistency)一致性：一致性是指更新操作成功并返回客户端完成后，所有节点在同一时间的数据完全一致。与ACID的C完全不同。<br>
C(Consistency)一致性：一致性是指更新操作成功并返回客户端完成后，所有节点在同一时间的数据完全一致。与ACID的C完全不同。<br>
A(Availability)可用性：可用性是指服务一直可用，而且是正常响应时间。<br>
P(Partition tolerance)分区容错性：分区容错性是指分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务。


![一些数据库和缓存数据库的特性与功能](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/%E4%B8%80%E4%BA%9B%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E7%BC%93%E5%AD%98%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E7%89%B9%E6%80%A7%E4%B8%8E%E5%8A%9F%E8%83%BD.png)

**1.1.2 附加特性**

在使用类似Redis这样的内存数据库时，一个首先要考虑的问题就是“当服务器被关闭时服务器存储的数据将何去何从呢？”Redis拥有两种不同形式的持久化方法，它们都可以用小儿紧凑的格式将存储在内存中的数据写入硬盘：第一种持久化方法为时间点转储（point-in-timedump）,转储操作既可以在“指定时间段内指定数据的写操作执行”这一条件被满足时执行，又可以通过调用两条转储到硬盘命令中的任何一条来执行；第二种持久化方法将所有修改了数据库的命令都写入了一个只追加文件里面，用户可以根据数据的重要程度，将追加写入设置为从不同步，每秒同步一次或者每写入一个命令就同步一次。

**1.1.3 使用Redis的理由**

可以使代码更简短、更易懂、更易维护，而且还可以使代码的运行速度更快。<br>
可以避免写入不必要的临时数据，免去了对临时数据进行扫描或者删除的麻烦，并最终改善程序的性能。

**1.2 Redis数据结构简介**

Redis可以存储键与5种不同数据结构类型之间的映射，这5种数据结构类型分别为STRING(字符串)、LIST（列表）、SET（集合）、HASH(散列)和ZSET（有序集合）。有一部分Redis命令对于这5中结构都是通用的，如DEL、TYPE、RENAME等；但也有一部分Redis命令只能对特定的一种或者两种结构使用。

***1.2.1 字符串***

结构如图所示<br>
![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/clipboard.png)

字符串命令<br>
![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/stringcmd.png)

redis字符串命令<br>
![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/rediscmd01.jpg)

***1.2.2 列表***

一个列表结构可以有序地存储多个字符串，和表示字符串时使用的方法一样。<br>
结构示例<br>

![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/list01.jpg)

列表命令<br>

![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/list02.jpg)

代码示例<br>

![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/list03.jpg)

***1.2.3 集合***

Redis的集合和列表都可以存储多个字符串，它们之间的不同在于，列表可以存储多个相同的字符串，而集合则通过使用散列表来保证自己存储的每个字符串都是各不相同的（这些散列表只有键，但没有与键相关联的值）。<br>
结构示例<br>

![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/set01.png)

代码示例<br>

![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/set02.png)

集合命令<br>

![image](https://github.com/makeittrue/Redis-learning-note/blob/master/images/chapter01/set03.png)

>这里还遇到了一些问题，由于对于该数据库的不了解，发生了键值类型冲突的问题。一个键名只能对应一种类型如果命令敲错了的话就要重置这个类型。<br>
代码为：<br>
    del key-name
