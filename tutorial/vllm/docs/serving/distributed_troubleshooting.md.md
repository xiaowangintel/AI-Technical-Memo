# distributed_troubleshooting.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/distributed_troubleshooting.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This troubleshooting note targets distributed deployments and starts from the most failure-prone layer: inter-node GPU communication.
  **CN:** 这份排障说明面向分布式部署，并从最容易出问题的层面切入：跨节点 GPU 通信。
- **EN:** It also points readers toward Ray scheduling and observability issues, so the page functions as a practical checklist for cluster bring-up.
  **CN:** 它还引导读者关注 Ray 调度与可观测性问题，因此该页面本质上是一份集群启动排查清单。

## Key Concepts / 关键概念
- **EN:** Validate GPU-to-GPU communication after starting the Ray cluster.
  **CN:** 在启动 Ray 集群后先验证 GPU 到 GPU 的通信。
- **EN:** Set NCCL-related environment variables during cluster creation so they propagate to all nodes.
  **CN:** 应在创建集群时设置 NCCL 相关环境变量，以便传播到所有节点。
- **EN:** Use Ray observability when resource requests cannot be fulfilled.
  **CN:** 当资源请求无法满足时，应借助 Ray 的可观测性能力排查。
