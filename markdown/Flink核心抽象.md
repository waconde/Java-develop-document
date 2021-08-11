# Flink核心抽象
##  环境对象

|  核心接口   | 运行级别 |  包含信息 | 
|  ----  | ----  | ----  | ----  |
| StreamExecutionEnvironment  | 开发时 | Job名称、并行度、容错、监控等配置信息 |
|  Environment  | 运行时 | 由StreamExecutionEnvironment 衍生而来，包含作业需要的Job名称、并行度、容错、监控等配置信息 |
| RuntimeContext | 运行时 | 同时拥有Environment配置信息，以及算子ID、算子本身等  | 

### 执行环境
1.  LocalStreamEmvironment  
  本地执行环境，在的那个JVM中使用多线程模拟flink集群  
  基本的工作流程如下：   
  1）执行Flink作业的Main函数生成Streamgraph，转化为JobGraph  
  2) 设置任务运行的配置信息  
  3) 依据配置信息启动对应的LocalFlinkMiniCluster  
  4) 依据配置信息和miniCluster生成对应的MiniClusterClient  
  5) 通过MiniClusterClient提交JobGraph到Minicluster  
   
2. RemoteStreamEnvironment   
   在大规模数据中心中部署的Flink生成集群的执行环境  
   工作流程如下：   
   1）执行Flink作业的Main函数生成Streamgraph，转化为JobGraph   
  2) 设置任务运行的配置信息  
  3) 提交JobGraph到远程的Flink集群。 
3. StreamContextEnvironment  
   在Cli命令行或者单元测试时候会被使用，执行步骤同上。   
   
4. StreamPlanEnvironment  
  在Flink Web UI 管理界面中可视化展现Job的时候，专门用来生成执行计划（实际上就是StreamGraph） 
  
5. ScalaShellStreamEnvironment  
Scala Shell 执行环境， 可以在命令行中交互式开发Flink作业
工作流程如下：   
1） 校验部署模式，目前Scala Shell 仅支持attached模式   
2） 上传每个作业需要的jar文件。
其余步骤与RemoteStreamEnvironment类似。 

### 运行时环境
运行时环境子在Flink中叫做Environment，该接口定义了在运行时刻Task所需要的所有配置信息，包括在静态配置和调度器调度之后生成的动态配置信息。 

主要有以下两个实现类：   

1. RuntimeEnvironment: Task执行时的环境信息。
2. SavepointEnvironment： 是Environment的最小实现。

### 运行时上下文
RuntimeContext是Function运行时的上下文，封装了Function运行时可能需要的所有信息，让Funciton在运行时能够获取到作业级别的信息，如并行度相关信息Task名称、执行配置信息（ExecutionConfig）、State等。   

不同的场景下的RuntimeContext:   

1. StreamRuntimeContext: 在流计算UDF中使用的上下文，用来访问作业信息、状态等。 
2. DistributeRuntimeUDFContext： 由运行时UDF所在的批处理算子创建，在DataSet批处理中使用
3. RuntimeUDFContext ： 在批处理应用的UDF中使用
4. SavepointRuntimeContext： 支持对检查点和保存点进行操作，包括读取】、变更、写入等。 
5. CepRuntimeContext： CEP复杂事件处理中使用的上下文。 
 
 ## 数据流元素
 数据流元素在Flink中叫做StreamElement，有数据记录StreamRecord、延迟标记LatencyMarker、Watermark、流状态标记StreamStatus。   
 
  1.  StreamRecord  
  StreamRecord表示数据流中的一条记录，也叫做数据记录。 
  包含以下内容：  
   1）数据的值本身  
   2)  时间戳
  2. LatencyMarker
  LatencyMarker用来近似评估延迟，LatencyMarker在Source中创建，并向下游发送， 
  3.

  
 ## 数据转换
 ## 算子
 ## 函数体系
 ## 数据分区
 ## 连接器
 ## 分布式ID 
