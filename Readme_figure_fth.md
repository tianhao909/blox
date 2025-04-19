### Blox
这是逐步复现Blox中各个图表结果的指南

#### 图6，图7
在图6和图7中，我们评估了FiFo、Tiresias和Optimus在开源Philly Trace上的表现。

在一个终端窗口中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

在第二个终端窗口中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_multi_run.py --simulate --load 6 --exp-prefix test
```

一旦仿真结束，它将生成使用FIFO、Tiresias和LAS调度器的响应性和作业完成时间（JCT）。

#### 图12和图13

运行具有不同接收策略的LAS调度器。这将提供图12和图13中的平均JCT。

在一个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator_dual_load.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

```fth
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python ./simulator_runner/simulator_dual_load.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

在第二个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_multi_run.py --simulate --load 6 --exp-prefix test
```

#### 图14和15 a) 和 b)

运行以下代码将生成动态调度器的运行时统计数据。

对于图14和图15 b)：
在一个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator_dual_load.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

在第二个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_dynamic_policy.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

图15 a)：

在一个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator_dual_load_small_large.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

在另一个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_dynamic_policy.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```