# Pagerank分布式作业实验报告

组员：柴百里、陈杨平、张爽、王宇春、魏国太、叶浩祥、吴振华



### spark基本概念的理解

Spark是一个实现快速而通用的集群计算的平台。它是一个大一统的软件栈，它的核心是一个对由很多计算任务组成的、运行在多个工作机器或者是一个计算集群上的应用进行调度、分发以及监控的计算引擎。

从上层来看，每个Spark应用都由一个驱动器程序来发起集群上的各种并行操作。驱动器程序包含应用的main函数，并且定义了集群上的分布式数据集，还对这些分布式数据集应用了相关操作。

驱动器程序通过一个SparkContext对象来访问Spark。这个对象代表对计算集群的一个连接。shell启动时已经自动创建了一个SparkContext,，是一个叫作sc的变量。

做实验的时候，对Spark印象最深的一个是RDD惰性求值和它的两种转换方式：转化操作、行动操作



### 算法分布式设计思路 

详细的设计思路第一个Project已经详细描述，本次实验中，我们的基本思路没有发生太大改变



### 关键代码描述

代码总体的变化不大，多了读取转换RDD文件的内容，根据空格拆分

将Rdd文件里面的内容转换成int 然后读取至list列表里面，

使得每一个Pair都满足**[节点，目的节点]**的格式

```python
#-------------------------------------pre process the data ----------------------------------
def Predata(line):							
	line = list(line.strip().split(' '))
	line = [int(x) for x in line]
	return [line[0] , line[1:]]
```



将ranks的值进行排序，通过reduce函数来合并那些key相同的value，让他们相加，再使用map函数，将计算得到的结果放在一个列表里面

```python
ranks = ranks.sortBy(ascending=False , keyfunc = lambda x : x[1])	#sortBy value

Sum = ranks.map(lambda x : x[1]).reduce(lambda x,y : x+y)
```



### 实验前的准备

完成实验的前提是安装好Hadoop和Spark的物理分布式，上个实验我们已经完成，这里就不做过多叙述



为了方便统一，我们使用了柴百里同学的手机热点，然后给每个组员分配了新的IP

因为热点每次分配的IP不固定，所以我们令每台主机再次执行以下命令：

```text
vi /etc/hosts
```

用于调整并更新每台主机的IP



（以下IP为最后一次一起做实验的IP）

|       IP       | 主机名  | 角色（序号） |
| :------------: | :-----: | :----------: |
| 192.168.43.172 | chaibli |    master    |
| 192.168.43.115 |   zs    |   worker1    |
| 192.168.43.129 |  chun   |   worker2    |
| 192.168.43.221 |   tai   |   worker3    |
| 192.168.43.59  |   ye    |   worker4    |
| 192.168.43.151 |   wu    |   worker5    |





### 实验结果



#### N=1000000

迭代10次

![ans1000000](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans1000000.png)



迭代15次

![ans1000000_15](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans1000000_15.png)



#### N=2000000

迭代10次

![ans2000000](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans2000000_10.png)

迭代15次

![ans1000000_15](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans2000000.png)



#### N=4000000

![ans4000000](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans4000000.png)



#### N=5000000

![ans5000000](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans5000000.png)

（这里计算的时间偏短，后面会分析其原因）



#### 登录hadoop查看执行情况

![106_100_time](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\106_100_time.png)



![106_core4_num5](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\106_core4_num5.png)



![106_p2](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\106_p2.png)



![dag](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\dag.png)



![ans](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\ans.png)



### 实验分析

![compare](C:\Users\Chester\Desktop\张爽的study\专必\机器学习\project3\ans\compare.png)

​																		**（单机版和分布式的时间比较）**



### spark性能调优、创新优化



### 遇到的问题



#### 主机IP信息不对

这个在上一次安装的时候，已经遇到过了，就是在重新更换新的热点的时候，没有及时更新host里面的ip的信息，所以导致在个别主机在连接Spark的时候，会出现无法连接的情况。但是我们一开始在启动Hadoop的时候，DataNode是能够正常连接的，我们怀疑是使用的本地地址（127.0.0.1），使得能够正常连接。



#### Worker运行时配置信息的不对称

在用Spark 我们发现在配置文件中的/usr/local/spark/conf/ spark-env.sh中有一个运行内存的不匹配的问题，我们在运行的时候，Master用4G的内存运行，而个别Worker运行的内存是3G或者2G。所以，我们之后每次的运行，就会统一规定好变量，这样就不会报错了。



#### 增加运行内存运行速度反而更加慢

一开始我们为了保证运行，规定每个worker工作的时候运行内存为2G，跑了200w的数据，时间估计为10min，后来提高了运行内存，时间居然到了30min。然后我们登陆上Hadoop的Web端，发现有一个Worker的运行速度特别慢（它只连接上了Spark，Hadoop无法正常连接），我们推测是其他Worker会等它运行完之后再去处理。之后将该工作节点剔除之后，运行的速度明显上升了。



#### 运行时间计算不对

我们在运行实验的时候，发现明明运行了很久，但是实际测试的时间很短。后来得知是RDD惰性求值的问题，我们测试时间的放置节点的位置不对，因为RDD分为行动操作和转换操作，我们之前使用的一直是转换操作，直到最后使用take函数的时候，才会进行行动操作，计算里面的数据内容，这部分消耗的时间是最长的。所以计时应该从take的前面开始，在take的后面结束计时，而我们经过讨论，觉得之前在实验中计算得到的计时时间应该是进行转换操作的时间。





### 实验心得

​       有了上一次的物理分布式安装体验，再加上PageRank的代码写的比较成熟，这一次的实验总体上还是比较顺利的。

​	  我们小组耗费的时间主要花在调参的时间上,以及一些细节问题的处理上（**前面有着重叙述**）。

​      最意外的还是因为我（ZS)大一买的书《Learning Spark》，当时买来想学一下大数据分析说明的，但看了几章直接就丢在书架上了，没想到现在派上了大用处！

​      书里面的内容很全，也并不是很难，关于Rdd的例子很多，真的学到了挺多东西！

​      希望下一个LSH分布式的作业能够更加顺利吧！ 

​	  





### 参考文件以及资料

https://iowiki.com/pyspark/pyspark_rdd.html

[《Learning Spark》](https://book.douban.com/subject/26424281/)

