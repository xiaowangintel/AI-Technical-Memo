# multi_node.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/multi_node_deployment/multi_node.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Llama 3.1 405B **Run 405B (fp16) on Two Nodes** Note that LLama 405B (fp8) can also be launched on a single node. / 该文档围绕 多节点部署 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Llama 3.1 405B
**EN:** `bash python -m sglang.launch_server --model-path meta-llama/Meta-Llama-3.1-405B-Instruct-FP8 --tp 8 ``
**CN:** 本节围绕 Llama 3.1 405B 展开，概述了 --tp, node, model-path, --model-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek V3/R1
**EN:** Please refer to DeepSeek documents for reference.
**CN:** 本节围绕 DeepSeek V3/R1 展开，概述了 Please, DeepSeek, refer, documents 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-Node Inference on SLURM
**EN:** This example showcases how to serve SGLang server across multiple nodes by SLURM. Submit the following job to the SLURM cluster.
**CN:** 本节围绕 Multi-Node Inference on SLURM 展开，概述了 SBATCH, INFO, echo, HEAD_NODE 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** SBATCH / **CN:** SBATCH
- **EN:** INFO / **CN:** INFO
- **EN:** echo / **CN:** echo
- **EN:** SLURM / **CN:** SLURM
- **EN:** HEAD_NODE / **CN:** HEAD_NODE
- **EN:** NCCL_INIT_ADDR / **CN:** NCCL_INIT_ADDR
- **EN:** node / **CN:** node
- **EN:** --tp / **CN:** --tp

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
