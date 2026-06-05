# parallelism_scaling.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/parallelism_scaling.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This overview helps readers choose a scaling strategy for single-model replicas and MoE serving, ranging from one GPU to multi-node distributed inference.
  **CN:** 该概览帮助读者为单模型副本和 MoE 服务选择扩展策略，覆盖从单 GPU 到多节点分布式推理的多种场景。
- **EN:** Its value lies in deployment heuristics: the page tells users when to stay single-GPU, when to use tensor parallelism, when to add pipeline parallelism, and which runtime details matter in containers.
  **CN:** 它的价值在于部署启发式规则：告诉用户何时维持单 GPU，何时使用 tensor parallelism，何时引入 pipeline parallelism，以及容器环境中哪些运行时细节最关键。

## Key Concepts / 关键概念
- **EN:** Model size relative to device capacity drives the parallelism choice.
  **CN:** 模型大小相对于设备容量的关系决定并行策略选择。
- **EN:** Tensor parallelism, pipeline parallelism, and MoE-related strategies are treated as distinct scaling tools.
  **CN:** tensor parallelism、pipeline parallelism 与 MoE 相关策略被视为不同的扩展工具。
- **EN:** Operational details such as Ray backend selection and `IPC_LOCK` can affect real deployments.
  **CN:** Ray 后端选择和 `IPC_LOCK` 等运维细节会影响真实部署效果。
