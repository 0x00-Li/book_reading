# VictoriaMetrics 如何为TB级别的数据生成快照

> [原文](https://valyala.medium.com/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282)

维多利亚(VictoriaMetrics)是一个高效的时序数据库。支持Prometheus的查询语法PromQL ，并且还有以下核心优势：

- 高性能的数据插入，[详见](https://valyala.medium.com/high-cardinality-tsdb-benchmarks-victoriametrics-vs-timescaledb-vs-influxdb-13e6ee64dd6b)
- 高性能的数据分析查询，[详见](https://valyala.medium.com/when-size-matters-benchmarking-victoriametrics-vs-timescale-and-influxdb-6035811952d4),性能报告，[详见](https://docs.google.com/spreadsheets/d/158AAsLMlGZ72D4MHfSdru_9dt3jpHymqo8Up_vp3LfU/edit?usp=sharing)
- 针对经典的时序数据，高压缩率
- 在不影响数据库性能的条件下，在线即时快照

接下来分析VictoriaMetrics 如何生成即时快照的



## 简单聊下 ClickHouse

VictoriaMetrics 的数据存储结构和 ClickHouse 的[MergeTree table ](https://clickhouse.yandex/docs/en/development/architecture/#merge-tree)非常相似；简单回顾下ClickHouse ,ClickHouse 是在数据分析领域和事件流领域的高性能数据库。在经典的分析查询场景下，ClickHouse 的性能是PostgreSQL和MySQL的10~1000倍。这意味着，单台ClickHouse 可以替代大量的常规数据库。

ClickHouse 的高性能离不开自身的优秀的MergeTree table核心[结构](https://clickhouse.tech/docs/en/development/architecture/)



## 什么是MergeTree

MergeTree是面向列的表引擎，其建立在类似于[日志结构化合并树](https://en.wikipedia.org/wiki/Log-structured_merge-tree)的数据结构上。MergeTree 特征：

- **每列都是分开存储的。** 在进行列扫描的时候，可以不用在读取和跳过其他列的数据上话费资源，减少了列扫描开销。每列的数据相似度也比较高，可以提高每列的数据压缩率。
- **每行是按照“主键”进行排序的，主键可以跨越多列。**主键没有唯一性的约束，多行可能有相同的主键。这样可以通过主键或前缀进行快速查找和范围扫描。另外，由于连续排序通常包含相似的数据，因此可以提高压缩率。
- **行被分割成中等大小的块。**每个块由每个列的块组成。每个数据块，都是独立处理的。这样可以充分利用多CPU系统的完美扩展性，只需要分配不同的CPU处理不同的块即可。数据块的大小是可配置的，但是建议配置为64KB~2MB之间的字块，充分利用CPU缓存提升性能，[CPU缓存和RAM 读的性能对比](https://gist.github.com/jboner/2841832)。另外，当必须从具有多行的块中仅访问几行时，会减少开销。
- **数据块合并成“parts”。**这些“parts”和Log Structured Merge(LSM) tree 中的[SSTables](https://stackoverflow.com/questions/2576012/what-is-an-sstable)很相似。ClickHouse 会在后台进程将较小的“parts”合并到较大的“parts”。与经典的LSM不同，MergeTree 不会按照parts大小划分层级。合并后的parts，在执行查询的时候，检索的part数量变少，会提升查询性能。当然，每个部分都包含与列数成比例的固定数量的文件，因此合并自然会减少文件数量。parts合并还有两外的一个好处，合并操作将有序的行中的相近的列合并到一起，提升了压缩率。

MergeTree 主要有这些特征，现在返回到VictoriaMetrics 如何生成即时快照的？



## VictoriaMetrics 中的即时快照

VictoriaMetrics 将数据存储在类似MergeTree结构中，所以MergeTree 的优点，VictoriaMetrics都有具有。此外，VictoriaMetrics 还提供了倒排索引，以便通过给定的时间序列选择器进行快速查询。倒排索引存储在**合并集**中—一种数据结构，该数据结构建立在MergeTree思想的基础上，并针对倒排索引查找进行了优化。

MergeTree 还有一个上文没提到的重要特征——原子性。意味着：

- 新增的数据要么出现在MergeTree中，要么就无法出现。MergeTree 不可能出现已经串讲的parts 的部分。合并过程也是如此-零件要么完全合并为新零件，要么无法合并。MergeTree中没有部分合并的parts。这是否意味着零件显示为蓝色？ 不是的。parts被组装在临时目录中，然后原子地移动到MergeTree。合并也是如此—在准备好新parts时，旧parts会自动替换为新parts。
- Parts的内容在MergeTree中是不会再边的。永远不会在变的。Parts是不可变的。 只有合并到更大的Parts后才能删除它们。

到这里，想到即时快照怎么生成的吗？继续往下看



许多文件系统都提供诸如硬链接之类的能力。 硬链接文件共享原始文件的内容。 它与原始文件没有区别。 硬链接不会在文件系统上占用额外的空间。 删除原始文件后，硬链接文件将成为“原始文件”，因为它成为唯一指向原始文件内容的文件。 无论文件大小如何，都会立即创建硬链接。 答对了！

**VictoriaMetrics使用硬链接为时间序列数据和反向索引创建即时快照**，因为它们都存储在类似MergeTree的数据结构中。它只是将所有部分中的所有文件原子地硬链接到快照目录中。这是安全的，因为Parts永远不会改变。 之后，可以使用任何合适的工具（例如，用于云存储的rsync或cp）将快照备份/存档到任何存储（例如，Amazon S3或Google Cloud Storage）。存档/备份过程并不着急-可能与所需的时间一样长，因为它与进一步的VictoriaMetrics操作完全脱钩了。 存档之前无需进行快照压缩，因为它已经包含高度压缩的数据-VictoriaMetrics为时间序列数据提供了最佳的压缩率。



新创建的快照不会在文件系统上占用额外的空间，因为其文件指向MergeTree表中的现有文件。 即使快照大小超过10 TB也是如此！ 在原始MergeTree合并/删除从快照链接的部分之后，快照开始占用额外的空间。 因此，不要忘记删除旧快照以释放磁盘空间

