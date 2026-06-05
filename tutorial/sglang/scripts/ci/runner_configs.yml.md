# runner_configs.yml — Configuration Analysis / 配置分析

## Source / 来源
- **File**: `scripts/ci/runner_configs.yml`
- **Repository**: sgl-project/sglang
- **Purpose**: Per-runner-config CUDA setup details. This configuration file is part of SGLang's `ci` automation flow. / 该配置文件用于支撑 SGLang 中的 `runner_configs` 流程，主要负责CI 编排、测试执行。它属于 `ci` 自动化路径的一部分。

## Content Analysis / 内容分析
### Lines 1-16: Document comments and metadata / 文档注释与元数据
```yaml
# Per-runner-config CUDA setup details. Single source of truth for the
# `runner_config` field on `register_cuda_ci(...)` calls. Consumed by
# scripts/ci/runner_configs.py (CLI wrapper), which is in turn called by
# .github/workflows/_pr-test-stage.yml.
#
# Each runner_config carries:
#   - install: install script path
#   - artifact_version: actions/download-artifact major version
#   - install_timeout: install-step wall-clock cap (minutes), enforced via
#     `timeout-minutes:` on the install step in _pr-test-stage.yml
#   - runs_on: GHA runner label for the stage's `runs-on:`. The literal
#     `$b200_runner` is substituted at workflow-load time with the dynamic
#     b200 runner tag from check-changes (see runner_configs.py --map).
#   - rdma_devices (optional): exported as SGLANG_CI_RDMA_ALL_DEVICES env
#     to the stage job; absent means unset.
```
**EN:** These lines provide document-level comments or metadata before the first configuration key.
**CN:** 这些行位于第一个配置键之前，用于提供文档级注释或元数据。

### Lines 17-21: Defines top-level key `_anchors` / 定义顶层键 `_anchors`
```yaml
_anchors:
  default_install: &default scripts/ci/cuda/ci_install_dependency.sh
  deepep_install:  &deepep  scripts/ci/cuda/ci_install_deepep.sh
  dsv4_install:    &dsv4    scripts/ci/cuda/ci_install_dsv4_dep.sh
```
**EN:** This section defines the `_anchors` mapping. Its nested keys (default_install, deepep_install, dsv4_install) refine how downstream automation should behave.
**CN:** 该部分定义了 `_anchors` 这个映射。其内部键（default_install, deepep_install, dsv4_install）进一步约束下游自动化的行为。

### Lines 22-33: Defines top-level key `runner_configs` / 定义顶层键 `runner_configs`
```yaml
runner_configs:
  1-gpu-small:       { install: *default, artifact_version: v4, install_timeout: "20", runs_on: 1-gpu-5090 }
  1-gpu-large:       { install: *default, artifact_version: v4, install_timeout: "20", runs_on: 1-gpu-h100 }
  2-gpu-large:       { install: *default, artifact_version: v4, install_timeout: "20", runs_on: 2-gpu-h100 }
  4-gpu-b200:        { install: *default, artifact_version: v6, install_timeout: "20", runs_on: $b200_runner }
  4-gpu-h100:        { install: *default, artifact_version: v4, install_timeout: "20", runs_on: 4-gpu-h100 }
  8-gpu-h200:        { install: *default, artifact_version: v4, install_timeout: "20", runs_on: 8-gpu-h200 }
  8-gpu-h20:         { install: *deepep,  artifact_version: v4, install_timeout: "20", runs_on: 8-gpu-h20, rdma_devices: "mlx5_1,mlx5_2,mlx5_3,mlx5_4" }
  deepep-4-gpu-h100: { install: *deepep,  artifact_version: v4, install_timeout: "20", runs_on: 4-gpu-h100 }
  deepep-8-gpu-h200: { install: *deepep,  artifact_version: v4, install_timeout: "20", runs_on: 8-gpu-h200-deepep }
  dsv4-4-gpu-b200:   { install: *dsv4,    artifact_version: v6, install_timeout: "30", runs_on: $b200_runner }
  dsv4-8-gpu-h200:   { install: *dsv4,    artifact_version: v4, install_timeout: "30", runs_on: 8-gpu-h200 }
```
**EN:** This section defines the `runner_configs` mapping. Its nested keys (1-gpu-small, 1-gpu-large, 2-gpu-large, 4-gpu-b200, 4-gpu-h100, 8-gpu-h200) refine how downstream automation should behave.
**CN:** 该部分定义了 `runner_configs` 这个映射。其内部键（1-gpu-small, 1-gpu-large, 2-gpu-large, 4-gpu-b200, 4-gpu-h100, 8-gpu-h200）进一步约束下游自动化的行为。

## Key Concepts / 关键概念
- **Git state inspection** / Git 状态检查
- **Runner provisioning** / 运行器准备
- **Top-level key `_anchors`** / 顶层键 `_anchors`
- **Top-level key `runner_configs`** / 顶层键 `runner_configs`

## Dependencies / 依赖关系
- **Referenced files / 引用文件**: `.github/workflows/_pr-test-stage.yml`, `_pr-test-stage.yml`, `runner_configs.py`, `scripts/ci/cuda/ci_install_deepep.sh`, `scripts/ci/cuda/ci_install_dependency.sh`, `scripts/ci/cuda/ci_install_dsv4_dep.sh`, `scripts/ci/runner_configs.py`
- **Runner or job keys / 运行器或作业键**: `runner_configs`
