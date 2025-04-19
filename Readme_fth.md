# Blox
本仓库包含了Eurosys 2024论文《Blox: A Modular Toolkit for Deep Learning Schedulers》的源代码实现。这项工作是微软研究院[Project Fiddle](https://https://aka.ms/msr-fiddle)的一部分。本源代码在[MIT许可证](LICENSE.txt)下提供。

Blox提供了一个模块化的框架，用于实现深度学习研究调度器。Blox提供了可轻松交换/修改的模块化抽象，允许研究人员实现新型的调度和部署策略。

### 提供的抽象
* **作业接收策略** - 允许研究人员实现任何接收策略，并提供一个接口来接收作业。
* **集群管理** - 处理可用节点的添加或删除。
* **作业调度策略** - 实现调度策略，即决定运行哪些作业。
* **作业部署策略** - 实现部署策略，即决定将作业运行在具体的GPU上。
* **作业启动和抢占** - 在集群中的任何节点上启动/抢占特定的作业。
* **指标收集** - 收集任何需要的指标，用于做出合理的部署或启动决策。

### 使用 Blox

Blox的各个组件设计为可以根据不同目标轻松交换。根据经验，现有的大多数深度学习调度器可以通过添加新的调度策略并修改部署策略来实现。



### Blox 仿真器

对于大规模实验，通常的做法是模拟策略在不同负载下的表现。Blox也提供了一个内置的仿真器用于此目的。
Blox的仿真器实现于`_simulator.py`中。研究人员可以配置负载值和负载到达模式。

### Blox 工具

Blox已经具有多个绘图和指标解析工具。根据配置，Blox会自动输出如作业完成时间和作业完成CDF等指标。

### 在 Blox 中编写仿真器

要在Blox中实现新的调度器，用户首先需要确定他们想要修改调度器的哪个部分。

一旦用户确定了贡献的具体位置，他们可以查看以下指南，了解需要修改的代码位置。
- 以下是文件的位置：
  - 调度策略 - /schedulers
  - 部署策略 - /placement
  - 工作负载策略 - /workload
  - 接收策略 - /admission_control

例如，用户可以查看`las_scheduler.py`，该文件实现了最少服务已用（Least Attained Service，LAS）调度器。

### 运行 Blox

Blox有两种运行模式，一种是实集群工作负载模式，另一种是仿真模式。

##### 仿真模式

开始使用Blox的最简单方式是仿真模式。

以下代码将在仿真模式下运行LAS调度策略，使用Philly Trace，并且作业的负载平均值为1.0。

在一个终端中启动：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator_simple.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 1 --exp-prefix test
```

在另一个终端中启动：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python las_scheduler.py --simulate --load 1 --exp-prefix test
```

确保每个进程只在一个机器或Docker容器中运行。我们使用固定的GRPC端口进行通信，如果启动了多个进程，可能会发生意外的后果。

##### 集群模式

在运行调度器的节点上：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python las_scheduler.py --exp-prefix cluster_run
```

在集群中的每个节点启动：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python node_manager.py --ipaddr ip_address_scheduler
```

在某些情况下，如果你想指定节点管理器绑定的接口，可以选择本地IP地址。在AWS上，可能需要选择`eth0`，而在Cloudlab上，优选接口为`enp94s0f0`。
在这些情况下，启动节点管理器的方式如下：

```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python node_manager.py --ipaddr ip_address_scheduler --interface interface_name
```

### 复现实验结果的详细信息
以下是复现Blox实验结果的说明。

#### 安装
Blox使用gRpc和Matplotlib来进行通信和绘制多个收集的指标。我们建议用户创建一个虚拟环境来安装依赖项。
```
pip install grpcio
pip install matplotlib
pip install pandas==1.3.0
pip install grpcio-tools
```



 fth 安装创建一个新的 Conda 环境
conda create -n fth_blox python=3.9 
conda activate fth_blox
```
pip install grpcio
pip install matplotlib
pip install pandas==1.3.0
pip install grpcio-tools

pip install pandas==1.3.0 numpy==1.26.4

pip install --upgrade protobuf tensorflow

cd /home/work/fth/software1/blox/blox/deployment
make
```

pandas 1.3.0 官方支持 Python 3.7.1 及以上版本，包括 Python 3.8 和 3.9。理论上，Python 3.10 也与 pandas 1.3.0 兼容，但安装过程中可能会出现 numpy 版本依赖问题，解决方法是在安装 pandas 之前先行安装 numpy 1.20.0 或以上版本。
需要说明的是，pandas 1.3.0 不支持 Python 3.10 及以上的版本，如果使用 Python 3.10 或更高版本，建议使用 pandas 1.4.0 或更高版本。另外，Python 3.7 已于 2023 年 6 月 27 日停止维护，从 pandas 1.5.0 开始，不再支持 Python 3.7。

conda env remove -n blox_env

###### 运行 Blox 代码
执行仿真时：
在一个终端窗口中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```

```fth
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python ./simulator_runner/simulator.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```
在第二个终端窗口中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_multi_run.py --simulate --load 6 --exp-prefix test
```
上述实验将需要大约8小时来运行，并生成如图6所示的CDF、JCT和FIFO、LAS以及Optimus调度器的运行时间。

要运行不同接收策略的LAS调度器，并提供图12和图13的平均JCT。

复现仅图12

在一个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator_acceptance_policy.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```
在第二个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_multi_run.py --simulate --load 6 --exp-prefix test
```

对于运行具有不同接收策略的LAS调度器，这将提供图12和图13的平均JCT。
在一个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python simulator_dual_load.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 6 --exp-prefix test
```
在第二个终端中：
```
PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python python blox_new_flow_multi_run.py --simulate --load 6 --exp-prefix test
```

#### 并行运行多个解决方案
Blox支持在同一台机器上并行运行多个仿真。用户需要正确指定端口号。
以下是运行多个仿真的示例。请在不同的终端中运行以下命令。

运行第一个仿真：
```
python simulator_simple.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 1 --exp-prefix test --simulator-rpc-port 50511
```
```
python las_scheduler.py --simulate --load 1 --exp-prefix test --simulator-rpc-port 50511
```

``` fth
编译 rm.proto 文件 进入包含 rm.proto 文件的目录，并使用 protoc 命令编译 .proto 文件生成 rm_pb2.py 文件。
假设 rm.proto 文件位于 /home/work/fth/software1/blox/blox/deployment/grpc_proto 目录下，运行以下命令：
cd /home/work/fth/software1/blox/blox/deployment/grpc_proto
protoc --python_out=../grpc_stubs rm.proto
这将生成 rm_pb2.py 文件，并将其放在 ../grpc_stubs 目录下（即 blox/deployment/grpc_stubs 目录）

CC=gcc-11 CXX=g++-11  python simulator_simple.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 1 --exp-prefix test --simulator-rpc-port 50511
```


运行第二个仿真：
```
python simulator_simple.py --cluster-job-log ./cluster_job_log --sim-type trace-synthetic --jobs-per-hour 1 --exp-prefix test --simulator-rpc-port 50501
```
```
python las_scheduler.py --simulate --load 1 --exp-prefix test --simulator-rpc-port 50501
```

#### 在Blox中运行集群实验
Blox允许用户在实际集群上运行实验。然而，在实际集群上运行实验需要额外的设置。

首先，在一个节点上启动你想运行的调度器。例如：
```
python las_scheduler_cluster.py --round-duration 30 --start-id-track 0 --stop-id-track 10
```
以上命令将启动集群调度器。

接下来，在你计划运行作业的每个节点上，启动redis-server并启动作业管理器。
```
redis-server
```
启动redis-server后，你需要启动节点管理器。
```
python node_manager.py --ipaddr {ip_addr_of_scheduler} --interface {network_interface_to_use}
```
启动节点管理器时，我们需要两个必需的参数：调度器的IP地址和你希望使用的网络接口。
要获取网络接口，你可以运行`ip a`命令。

最后，你需要将作业发送到调度器。有关如何提交作业的示例，可以查看[此处](https://github.com/msr-fiddle/blox/blob/main/blox/deployment/job_submit_script.py)。

在集群上启动作业需要两个必需的字段：
- `launch_command`：给出启动命令。
- `launch_params`：你可以通过这个键传递任何命令行参数，如在job_submit_script.py中所做的。

对于每个应用，Blox还会设置以下环境变量。
对于多个作业，Blox会多次启动命令，但每次使用不同的环境变量。用户需要自行配置这些环境变量。以下是可以查询的环境变量及其说明：
```
local_gpu_id : 指定此作业运行的本地GPU ID。
master_ip_address: 如果是分布式作业，这是要使用的主IP地址。
world_size: 运行此命令的总工作进程数。
dist_rank: 当前进程的排名。
job_id: Blox为该作业分配的作业ID。
local_accessible_gpu: 此作业可访问的所有GPU。对于CUDA_VISIBLE_DEVICES尤其有用。
```