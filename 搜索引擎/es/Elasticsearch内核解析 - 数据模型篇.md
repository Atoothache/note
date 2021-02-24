# Elasticsearch内核解析 - 数据模型篇

Elasticsearch是一个实时的分布式搜索和分析引擎，它可以帮助我们用很快的速度去处理大规模数据，可以用于全文检索、结构化检索、推荐、分析以及统计聚合等多种场景。

Elasticsearch是一个建立在全文搜索引擎库Apache Lucene 基础上的分布式搜索引擎，Lucene最早的版本是2000年发布的，距今已经18年，是当今最先进，最高效的全功能开源搜索引擎框架，众多搜索领域的系统都基于Lucene开发，比如Nutch，Solr和Elasticsearch等。Elasticsearch第一个版本发布于2010年，发布后就以非常快的速度霸占了开源搜索系统领域，成为目前搜索领域的首选，著名的维基百科，GitHub和Stack Overflow都在使用它。

既然有Lucene娥，为啥还会出现很火的Elasticsearch？回答这个问题之前， 我们先来简单看一下Lucene中的一些数据模型：

## Lucene数据模型

Lucene中包含了四种基本数据类型，分别是：

- Index：索引，由很多的Document组成。
- Document：由很多的Field组成，是Index和Search的最小单位。
- Field：由很多的Term组成，包括Field Name和Field Value。
- Term：由很多的字节组成，可以分词。

上述四种类型在Elasticsearch中同样存在，意思也一样。

Lucene中存储的索引主要分为三种类型：

- Invert Index：倒排索引，或者简称Index，通过Term可以查询到拥有该Term的文档。可以配置为是否分词，如果分词可以配置不同的分词器。索引存储的时候有多种存储类型，分别是：
- DOCS：只存储DocID。
- DOCS_AND_FREQS：存储DocID和词频（Term Freq）。
- DOCS_AND_FREQS_AND_POSITIONS：存储DocID、词频（Term Freq）和位置。
- DOCS_AND_FREQS_AND_POSITIONS_AND_OFFSETS：存储DocID、词频（Term Freq）、位置和偏移。
- DocValues：正排索引，采用列式存储。通过DocID可以快速读取到该Doc的特定字段的值。由于是列式存储，性能会比较好。一般用于sort，agg等需要高频读取Doc字段值的场景。
- Store：字段原始内容存储，同一篇文章的多个Field的Store会存储在一起，适用于一次读取少量且多个字段内存的场景，比如摘要等。

Lucene中提供索引和搜索的最小组织形式是Segment，Segment中按照索引类型不同，分成了Invert Index，Doc Values和Store这三大类（还有一些辅助类，这里省略），每一类里面都是按照Doc为最小单位存储。Invert Index中存储的Key是Term，Value是Doc ID的链表；Doc Value中Key 是Doc ID和Field Name，Value是Field Value；Store的Key是Doc ID，Value是Filed Name和Filed Value。

由于Lucene中没有主键概念和更新逻辑，所有对Lucene的更新都是Append一个新Doc，类似于一个只能Append的队列，所有Doc都被同等对等，同样的处理方式。其中的Doc由众多Field组成，没有特殊Field，每个Field也都被同等对待，同样的处理方式。

从上面介绍来看，Lucene只是提供了一个索引和查询的最基本的功能，距离一个完全可用的完整搜索引擎还有一些距离：

## Lucene的不足

1. Lucene是一个单机的搜索库，如何能以分布式形式支持海量数据?
2. Lucene中没有更新，每次都是Append一个新文档，如何做部分字段的更新？
3. Lucene中没有主键索引，如何处理同一个Doc的多次写入？
4. 在稀疏列数据中，如何判断某些文档是否存在特定字段？
5. Lucene中生成完整Segment后，该Segment就不能再被更改，此时该Segment才能被搜索，这种情况下，如何做实时搜索？

上述几个问题，对于搜索而言都是至关重要的功能诉求，我们接下来看看Elasticsearch中是如何来解这些问题的。

参考：https://zhuanlan.zhihu.com/p/34852646

大佬知乎：https://www.zhihu.com/people/8080/posts