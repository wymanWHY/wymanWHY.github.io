---
title: Hadoop (二) —— MapReduce
date: 2018-05-31 20:01:22
categories: 大数据
tags:
    - Hadoop
    - MapReduce
---

![](https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/crop%3D0%2C0%2C820%2C540%3Bc0%3Dbaike92%2C5%2C5%2C92%2C30/sign=c759236df4039245b5fabb4fbaa488f2/d833c895d143ad4bd58493608a025aafa50f06e3.jpg)

<!--more-->

> MapReduce的核心，在于**移动计算，而不是移动数据**



## MapReduce工作流程 ##

以WorderCount为例：将一个文本格式的输入，统计其中每个单词出现的数量

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/6597696.jpg)

MapReduce将这项工作分为了4个步骤完成，分别是`split、Map、shuffle、reduce`。

- split : 将文本以某种规则进行拆分，此处以每行结束的换行符为记号，按行拆分为3项；
- map: 每项继续拆分，并将单词和次数生成映射关系，也就是键值对的形式`<key,value>`;
- shuffle: 将具有相同`key`的键值对放到一块;
- reduce: 把把各个key中的value值聚合（或累积），完成统计，形成新的`<key,value>`;

完成后，将结果写入文件。


### 在代码实现 ###

（依旧以WordCount为例）

**Mapper抽象类：**
```java
protected void setup(Context context)
protected void map(KEYIN key, VALUEIN value,
Context context) throws IOException,
InterruptedException {
context.write((KEYOUT) key, (VALUEOUT) value);
}
protected void cleanup(Context context
) throws IOException, InterruptedException {
// coding
}
public void run(Context context) throws IOException,
InterruptedException {
setup(context);
while (context.nextKeyValue()) {
map(context.getCurrentKey(), context.getCurrentValue(), context);
}
cleanup(context);
}
```

**Reducer抽象类**
```java
protected void setup(Context context
) throws IOException, InterruptedException {
}
protected void reduce(KEYIN key, Iterable<VALUEIN> values, Context context
) throws IOException, InterruptedException {
for(VALUEIN value: values) {
context.write((KEYOUT) key, (VALUEOUT) value);
}
}
protected void cleanup(Context context
) throws IOException, InterruptedException {
}
public void run(Context context) throws IOException, InterruptedException {
setup(context);
while (context.nextKey()) {
reduce(context.getCurrentKey(), context.getValues(), context);
}
cleanup(context);
}
```

**最小的MapReduce驱动:**
```java
public class MinimalMapReduceWithDefaults extends Configured implements
Tool {
public int run(String[] args) throws IOException {
JobConf conf = JobBuilder.parseInputAndOutput(this, getConf(), args);
if (conf == null) {
return -1;}
conf.setInputFormat(TextInputFormat.class);
conf.setMapperClass(IdentityMapper.class);
conf.setMapRunnerClass(MapRunner.class);
conf.setMapOutputKeyClass(LongWritable.class);
conf.setMapOutputValueClass(Text.class);
conf.setPartitionerClass(HashPartitioner.class);
conf.setNumReduceTasks(1);
conf.setReducerClass(IdentityReducer.class);
conf.setOutputKeyClass(LongWritable.class);
conf.setOutputValueClass(Text.class);
conf.setOutputFormat(TextOutputFormat.class);
JobClient.runJob(conf);
return 0;}
public static void main(String[] args) throws Exception {
int exitCode = ToolRunner.run(new MinimalMapReduceWithDefaults(), args);
System.exit(exitCode);
}}
```
## MapReduce剖析 ##

一份作业的完整处理过程，可以分解为两个子任务：map和reduce（如下图）

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1527778356903&di=81cf2cb838d0080255315148f7487e7f&imgtype=jpg&src=http%3A%2F%2Fimg1.imgtn.bdimg.com%2Fit%2Fu%3D3942933336%2C580178229%26fm%3D214%26gp%3D0.jpg)


MapReduce核心组件包括：

1. InputSplits
2. RecordReader
3. Mapper
4. Combiner
5. Partitioner & Shuffle&Sort
6. Reducer

这些组件的上下游联系如下图：
![](http://p7wcdketk.bkt.clouddn.com/18-5-31/55170565.jpg)

这也是MapReduce的工作流结构


### 输入数据分块InputSplits ###


![](http://p7wcdketk.bkt.clouddn.com/18-5-31/91767526.jpg)

InputSplit定义了输入到单个Map任务的输入数据，配置文件`mapred-site.xml`中的`mapred.min.split.size`参数控制这个大小。

我们假设需要将一个较大的文件（比如1T）实现Map操作，可以通过两种方式来提升Map效率：

1. 重新修改Block大小，将Block的大小分得大一些（比如100G），然后将这份文件重新写入HDFS，那么就可以把文件切割为10个，每个子文件占用一份Block，将10份子文件分别执行Map操作，会比直接执行一个1T的作业来的快。

2.修改分区大小，也就是修改配置文件`mapred-site.xml`中的`mapred.min.split.size`参数，分区的大小是block的整数倍，输入文件大小可以分成多少个分区，即需要几个map。

### 数据记录读入RecordReader ###

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/16459732.jpg)

`InputSplit`定义了一个数据分块，但是没有定义如何读取数据记录，`RecordReader`实际上定义了如何将数据
记录转化为一个`<key,value>`对的详细方法，并将数据记录传给`Mapper`类; `TextInputFormat`提供了`LineRecordReader`，可以读入一个文本行数据记录。


### Mapper ###

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/42093935.jpg)

每一个Mapper类的实例生成了一个Java进程，负责处理某一个InputSplit上的数据。

Mapper有两个额外的参数OutputCollector以及Reporter，前者用来收集中间结果，后者用来获得环境参数以及设置当前执行的状态。

### Combiner ###

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/40062552.jpg)

Combiner实现合并相同key的键值对，减少partitioner时候的数据通信开销，它是在本地执行的一个Reducer，只要满足条件才能够执行。

API：
```java
conf.setCombinerClass(Reduce.class)
```

### Partitioner & Shuffle&Sort ###

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/1145835.jpg)

- 分区：在Map工作完成之后，每一个Map函数会将结果传到对应的Reducer所在的节点，此时，用户可以提供一个Partitioner类，用来决定一个给定的`<key,value>`对传给哪个节点；
 
> MR 在分区中默认使用HASH方法，这种方法具有较强的随机性，其造成的结果往往会在`map`过程数据中产生`数据倾斜的可能`，可以通过重写分区算法解决。

- 排序：传输到每一个Reducer节点上的、将被所有的Reduce函数接收到的Key,value对会被Hadoop自动排序（即Map生成的结果传送到某一个节点的时候，会被自动排序）

> 排序： MR 默认按字典顺序排序，也可以重写方法，改为自定义的排序



### Reducer ###

![](http://p7wcdketk.bkt.clouddn.com/18-5-31/83812489.jpg)

---

## Map和Reduce的个数优化 ##


Tasks数的大小能显著的改善Hadoop执行的性能。增加task的个数会增加系统框架的开销，但同时也会增强负载均衡并降低任务失败的开销。一个极 端是1个map、 1个reduce的情况，这样没有任务并行。

另一个极端是1,000,000个map、 1,000,000个reduce的情况，会由于框架的开销过大而使得系统资源耗尽。

**map和reduce的优化分为两块：**



### 1、如何确定整个集群的map/reduce任务最优数 ###

#### Map task数的计算: ####

对于整个集群来说一般只设置`reduce`任务数，而map任务数是由`Splits`个数决定的。一般认为：**一个job的MapTask数量就等于split的个数**  (`mapred.min.split.size`参数)

对于input的每个文件，`split`的个数可以这样计算：
- `文件大小/splitsize` > `1.1`， 创建一个split， **这个split的字节数=splitsize**， 文件剩
余字节数=文件大小-splitsize
- `文件剩余字节数/splitsize` < `1.1`， 剩余的部分作为一个split


#### Reduce个数计算 ####

reduce task的数量由mapred.reduce.tasks这个参数设定，默认值是1

另外，可以通过调用API设置具体的运行过程中的task数量：

```java
setNumReduceTasks(n);
```

通过这条接口设定`ReduceTask`的数量，当设置为1的时候，仅有1个task。

当设置的数值大于1时，会有分成多个reduce task，同样的，会出现多份输出文件。

假设`setNumReduceTasks(N)`，意味着将分了`N`个`reduce task`，`N`个task并行处理各自的reduce任务，因此会独立产生N份结果文件——`多个reduce task的话就会有多个reduce结果， part-r-00000, partr-00001, ...part-r-0000n`，每份文件所保存的即为`task[n]`的执行结果。

Q： 如果现在出现一个需求——统计一个文本中**每个英文字母出现的次数**，这个文本中共有`13`个英文字母，假设现在通过调用接口`setNumReduceTasks`共设置了`20`个task，setNumReduceTasks(20)，那么MR计算之后，会产生多少个结果文件？

A：**20个**！实验后可以看到，绝大部分的结果文件都只有一行：<字母，次数>，但偶尔会有个别结果文件出现了两行，也就是两个字母的出现次数，因为task数量`20`大于要统计的字母数`13`，因此会产生许多的空白的文件，可是并不见得空白文件的数量就是`20-13=7`个！有可能是8个或其他。

**原因在于分区：**分区的方法使用的是HASH算法，会具有一定的概率造成两个或多个字母分到了同一区

如果`setNumReduceTasks(0)`说明没有reduce部分的操作，可以看到计算的结果就是，键值对并没有统计出一个最终的结果，只将map产生的过程结果保存到了最后。


**一般来说reduce task数量是0.95或者0.75*( nodes x mapred.tasktracker.reduce.tasks.maximum)**

> mapred.tasktracker.tasks.reduce.maximum的数量一般设置为各节点cpu core数量，即能同时计算的slot数量。

> 0.95，当map结束时，所有的reduce能够立即启动

> 1.75，较快的节点结束第一轮reduce后，可以开始第二轮的reduce任务，从而提高负载均衡

### 2、单节点map/reduce任务数 ###


#### Map的个数 ####

`mapred.tasktracker.map.tasks.maximum` 默认值为`2`

`优化值： (CPU数量 > 2) ? (CPU数量 * 0.75) : 1` （官方建议）

#### Reduce个数 ####

`mapred.tasktracker.reduce.tasks.maximum` 默认值为`2`

`优化值： (CPU数量 > 2) ? (CPU数量 * 0.50): 1` （官方建议）


### 优化Tips ###

#### Map数过大 ####
- Map only的作业,输出文件太小，产生大量小文件
- 初始化和创建Map的开销比执行还大
- 执行时间最好不低于1分钟

#### Map数太小 ####
- 单个Map时间过长，频繁推测执行
- 数据膨胀，内存不足 

#### Reduce数过大 ####
- 大量的小文件
- 耗费大量不必要的资源

### Reduce数过低 ###
- 数据倾斜——不能通过修改参数改变
- 最佳方法 参数：mapred.reduce.tasks


**注意：Key的分布，某种程度上，决定了Reduce数目**