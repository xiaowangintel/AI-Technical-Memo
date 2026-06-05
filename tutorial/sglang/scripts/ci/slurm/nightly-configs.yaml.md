# nightly-configs.yaml — Configuration Analysis / 配置分析

## Source / 来源
- **File**: `scripts/ci/slurm/nightly-configs.yaml`
- **Repository**: sgl-project/sglang
- **Purpose**: Nightly benchmark configurations for srt-slurm powered runners. This configuration file is part of SGLang's `slurm` automation flow. / 该配置文件用于支撑 SGLang 中的 `nightly-configs` 流程，主要负责CI 编排、测试执行。它属于 `slurm` 自动化路径的一部分。

## Content Analysis / 内容分析
### Lines 1-11: Document comments and metadata / 文档注释与元数据
```yaml
# Nightly benchmark configurations for srt-slurm powered runners.
#
# Structure mirrors InferenceX nvidia-master.yaml but only includes fields
# actually needed by the runner — prefill/decode topology details are already
# encoded in each srt-slurm recipe YAML and are not duplicated here.
#
# To add/remove concurrencies: edit conc-list for the relevant search-space entry.
# To add a new runner:         add a new top-level block and create a corresponding
#                              nightly-test-<runner>.yml workflow.
# Never edit workflow YAML files directly for these changes.
```
**EN:** These lines provide document-level comments or metadata before the first configuration key.
**CN:** 这些行位于第一个配置键之前，用于提供文档级注释或元数据。

### Lines 12-31: Defines top-level key `dsr1-fp8-gb200-dynamo-sglang` / 定义顶层键 `dsr1-fp8-gb200-dynamo-sglang`
```yaml
dsr1-fp8-gb200-dynamo-sglang:
  model: deepseek-ai/DeepSeek-R1-0528
  model-prefix: dsr1
  runner: gb200
  precision: fp8
  framework: dynamo-sglang
  multinode: true
  disagg: true
  seq-len-configs:
    - isl: 1024
      osl: 1024
      search-space:
        - conc-list: [1024, 2048, 4096, 6144]
          # https://github.com/NVIDIA/srt-slurm/blob/sglang-nightly-regression/recipes/gb200-fp8/1k1k/max-tpt.yaml
          config_file: recipes/gb200-fp8/1k1k/max-tpt.yaml

        - conc-list: [4096]
          # https://github.com/NVIDIA/srt-slurm/blob/sglang-nightly-regression/recipes/gb200-fp8/1k1k/ultra-tpt.yaml
          config_file: recipes/gb200-fp8/1k1k/ultra-tpt.yaml
```
**EN:** This section defines the `dsr1-fp8-gb200-dynamo-sglang` mapping. Its nested keys (model, model-prefix, runner, precision, framework, multinode) refine how downstream automation should behave.
**CN:** 该部分定义了 `dsr1-fp8-gb200-dynamo-sglang` 这个映射。其内部键（model, model-prefix, runner, precision, framework, multinode）进一步约束下游自动化的行为。

### Lines 32-46: Defines top-level key `dsr1-fp4-gb200-dynamo-sglang` / 定义顶层键 `dsr1-fp4-gb200-dynamo-sglang`
```yaml
dsr1-fp4-gb200-dynamo-sglang:
  model: nvidia/DeepSeek-R1-0528-NVFP4-v2
  model-prefix: dsr1
  runner: gb200
  precision: fp4
  framework: dynamo-sglang
  multinode: true
  disagg: true
  seq-len-configs:
    - isl: 1024
      osl: 1024
      search-space:
        - conc-list: [512, 2048, 4096, 8192]
          # https://github.com/NVIDIA/srt-slurm/blob/sglang-nightly-regression/recipes/gb200-fp4/1k1k/mid-curve.yaml
          config_file: recipes/gb200-fp4/1k1k/mid-curve.yaml
```
**EN:** This section defines the `dsr1-fp4-gb200-dynamo-sglang` mapping. Its nested keys (model, model-prefix, runner, precision, framework, multinode) refine how downstream automation should behave.
**CN:** 该部分定义了 `dsr1-fp4-gb200-dynamo-sglang` 这个映射。其内部键（model, model-prefix, runner, precision, framework, multinode）进一步约束下游自动化的行为。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **YAML configuration** / YAML 配置
- **Git state inspection** / Git 状态检查
- **Cluster scheduling** / 集群调度
- **Runner provisioning** / 运行器准备

## Dependencies / 依赖关系
- **Referenced files / 引用文件**: `//github.com/NVIDIA/srt-slurm/blob/sglang-nightly-regression/recipes/gb200-fp4/1k1k/mid-curve.yaml`, `//github.com/NVIDIA/srt-slurm/blob/sglang-nightly-regression/recipes/gb200-fp8/1k1k/max-tpt.yaml`, `//github.com/NVIDIA/srt-slurm/blob/sglang-nightly-regression/recipes/gb200-fp8/1k1k/ultra-tpt.yaml`, `nvidia-master.yaml`, `recipes/gb200-fp4/1k1k/mid-curve.yaml`, `recipes/gb200-fp8/1k1k/max-tpt.yaml`, `recipes/gb200-fp8/1k1k/ultra-tpt.yaml`
