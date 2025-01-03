# 全局配置

你可以在安装过程中，通过`-set`来修改以下的客制化参数，例如：

```
helm install vgpu vgpu-charts/vgpu --set devicePlugin.deviceMemoryScaling=5 ...
```

* `devicePlugin.deviceSplitCount:` 
  整数类型，预设值是10。GPU的分割数，每一张GPU都不能分配超过其配置数目的任务。若其配置为N的话，每个GPU上最多可以同时存在N个任务。
* `devicePlugin.deviceMemoryScaling:` 
  浮点数类型，预设值是1。NVIDIA装置显存使用比例，可以大于1（启用虚拟显存，实验功能）。对于有*M*显存大小的NVIDIA GPU，如果我们配置`devicePlugin.deviceMemoryScaling`参数为*S*，在部署了我们装置插件的Kubenetes集群中，这张GPU分出的vGPU将总共包含 `S * M` 显存。
* `devicePlugin.migStrategy:`
  字符串类型，目前支持"none“与“mixed“两种工作方式，前者忽略MIG设备，后者使用专门的资源名称指定MIG设备，使用详情请参考mix_example.yaml，默认为"none"
* `devicePlugin.disablecorelimit:`
  字符串类型，"true"为关闭算力限制，"false"为启动算力限制，默认为"false"
* `scheduler.defaultMem:`
  整数类型，预设值为0，表示不配置显存时使用的默认显存大小，单位为MB。当值为0时，代表使用全部的显存。
* `scheduler.defaultCores:`
  整数类型(0-100)，默认为0，表示默认为每个任务预留的百分比算力。若设置为0，则代表任务可能会被分配到任一满足显存需求的GPU中，若设置为100，代表该任务独享整张显卡
* `scheduler.defaultGPUNum:`
  整数类型，默认为1，如果配置为0，则配置不会生效。当用户在 pod 资源中没有设置 nvidia.com/gpu 这个 key 时，webhook 会检查 nvidia.com/gpumem、resource-mem-percentage、nvidia.com/gpucores 这三个 key 中的任何一个 key 有值，webhook 都会添加 nvidia.com/gpu 键和此默认值到 resources limit中。
* `scheduler.defaultSchedulerPolicy.nodeSchedulerPolicy:` 字符串类型，预设值为"binpack", 表示GPU节点调度策略，"binpack"表示尽量将任务分配到同一个GPU节点上，"spread"表示尽量将任务分配到不同GPU节点上。
* `scheduler.defaultSchedulerPolicy.gpuSchedulerPolicy:` 字符串类型，预设值为"spread", 表示GPU调度策略，"binpack"表示尽量将任务分配到同一个GPU上，"spread"表示尽量将任务分配到不同GPU上。
* `resourceName:`
  字符串类型, 申请vgpu个数的资源名, 默认: "nvidia.com/gpu"
* `resourceMem:`
  字符串类型, 申请vgpu显存大小资源名, 默认: "nvidia.com/gpumem"
* `resourceMemPercentage:`
  字符串类型，申请vgpu显存比例资源名，默认: "nvidia.com/gpumem-percentage"
* `resourceCores:`
  字符串类型, 申请vgpu算力资源名, 默认: "nvidia.com/cores"
* `resourcePriority:`
  字符串类型，表示申请任务的任务优先级，默认: "nvidia.com/priority"

# Pod配置（在注解中指定）

* `nvidia.com/use-gpuuuid:` 
  字符串类型, 如: "GPU-AAA,GPU-BBB"
  如果设置, 该任务申请的设备只能是字符串中定义的设备之一。
* `nvidia.com/nouse-gpuuuid`
  字符串类型, 如: "GPU-AAA,GPU-BBB"
  如果设置, 该任务不能使用字符串中定义的任何设备
* `nvidia.com/nouse-gputype:`
  字符串类型, 如: "Tesla V100-PCIE-32GB, NVIDIA A10"
  如果设置, 该任务不能使用字符串中定义的任何设备型号
* `nvidia.com/use-gputype`
  字符串类型, 如: "Tesla V100-PCIE-32GB, NVIDIA A10"
  如果设置, 该任务申请的设备只能使用字符串中定义的设备型号。
* `hami.io/gpu-scheduler-policy`
  字符串类型, "binpack" 或 "spread"
  spread:, 调度器会尽量将任务均匀地分配在不同GPU中
  binpack: 调度器会尽量将任务分配在已分配的GPU中，从而减少碎片
* `hami.io/node-scheduler-policy`
  字符串类型, "binpack" 或 "spread"
  spread: 调度器会尽量将任务均匀地分配到不同节点上
  binpack: 调度器会尽量将任务分配在已分配任务的节点上，从而减少碎片 
* `nvidia.com/vgpu-mode`
  字符串类型, "hami-core" 或 "mig"
  该任务希望使用的vgpu类型


# 容器配置（在容器的环境变量中指定）

* `GPU_CORE_UTILIZATION_POLICY:`
  字符串类型，"default", "force", "disable"
  默认为"default"
  代表容器算力限制策略， "default"为默认，"force"为强制限制算力，一般用于测试算力限制的功能，"disable"为忽略算力限制
* `CUDA_DISABLE_CONTROL`
  布尔类型，"true", "false"
  默认为false
  若设置为true，则代表屏蔽掉容器层的资源隔离机制，需要注意的是，这个参数只有在容器创建时指定才会生效，一般用于调试