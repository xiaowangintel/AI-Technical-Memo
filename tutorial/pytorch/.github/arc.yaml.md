# arc.yaml — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/arc.yaml`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Stores repository-level GitHub metadata, automation policies, and contribution-facing configuration.
- **用途 (CN)**: 存放仓库级 GitHub 元数据、自动化策略以及面向贡献者的配置。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````yaml
# ARC (Actions Runner Controller) Runner Label Mapping
#
# Maps current GitHub Actions runner labels to new ARC runner labels.
# Reference: https://github.com/pytorch/ci-infra/issues/396
#
# New label format:
#   {os}-[b]{arch}{vendor}{features}-{vcpu}-{memory}[-{gpu_type}[-{gpu_count}]]
#
# Fields:
#   os       - l=Linux, w=Windows, m=MacOS
#   b        - (optional) bare-metal instance
#   arch     - x86=x86_64, arm64=AArch64
#   vendor   - i=Intel, a=AMD, g2/g3/g4=Graviton gen
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 14-25 / 第 14-25 行

````yaml
#   features - (x86 only) avx2, avx512, amx
#   vcpu     - vCPU count
#   memory   - RAM in GiB
#   gpu_type - (optional) t4, a10g, l4
#   gpu_count- (optional, omitted when 1)
#
# Entries marked "# upgraded" had no exact ARC equivalent and were mapped to
# the next larger available runner.

runner_mapping:

  # ---- x86 CPU — Intel AVX-512 (c5, c7i families) ----
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 27-39 / 第 27-39 行

````yaml
  linux.large: l-x86iavx512-2-4                        # c5.large
  linux.2xlarge: l-x86iavx512-8-64                     # c5.2xlarge
  linux.c7i.2xlarge: l-x86iavx512-8-64                 # c7i.2xlarge
  linux.4xlarge: l-x86iavx512-16-128                   # c5.4xlarge
  linux.c7i.4xlarge: l-x86iavx512-16-128               # c7i.4xlarge
  linux.12xlarge: l-x86iavx512-48-384                  # c5.12xlarge
  linux.c7i.12xlarge: l-x86iavx512-48-384              # c7i.12xlarge
  linux.24xl.spr-metal: l-bx86iamx-92-167              # c7i.metal-24xl

  # ---- x86 CPU — Intel AMX (m7i-flex family) ----

  linux.2xlarge.amx: l-x86iamx-8-64                    # m7i-flex.2xlarge
  linux.8xlarge.amx: l-x86iamx-32-128                  # m7i-flex.8xlarge
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 41-53 / 第 41-53 行

````yaml
  # ---- x86 CPU — Intel AVX2 (m4 family) ----

  linux.2xlarge.avx2: l-x86iavx2-8-32                  # m4.2xlarge
  linux.10xlarge.avx2: l-x86iavx2-40-160               # m4.10xlarge

  # ---- x86 CPU — Memory-optimized (r5, r7i families) ----

  linux.r7i.2xlarge: l-x86iavx512-8-64                 # r7i.2xlarge
  linux.r7i.4xlarge: l-x86iavx512-16-128               # r7i.4xlarge
  linux.4xlarge.memory: l-x86iavx512-16-128            # r5.4xlarge
  linux.8xlarge.memory: l-x86iavx512-32-256            # r5.8xlarge
  linux.12xlarge.memory: l-x86iavx512-48-384           # r5.12xlarge
  linux.24xlarge.memory: l-x86iavx512-94-768           # r5.24xlarge
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 55-65 / 第 55-65 行

````yaml
  # ---- x86 CPU — AMD (m6a, m7a families) ----

  linux.24xlarge.amd: l-x86aavx512-125-463             # m7a.24xlarge

  # ---- x86 GPU — T4 (g4dn family) ----

  linux.g4dn.4xlarge.nvidia.gpu: l-x86iavx512-29-115-t4          # g4dn.4xlarge
  linux.g4dn.12xlarge.nvidia.gpu: l-x86iavx512-45-172-t4-4       # g4dn.12xlarge
  linux.g4dn.metal.nvidia.gpu: l-bx86iavx512-94-344-t4-8         # g4dn.metal

  # ---- x86 GPU — A10G (g5 family) ----
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 67-76 / 第 67-76 行

````yaml
  linux.g5.4xlarge.nvidia.gpu: l-x86aavx2-29-113-a10g            # g5.4xlarge
  linux.g5.12xlarge.nvidia.gpu: l-x86aavx2-45-167-a10g-4         # g5.12xlarge
  linux.g5.48xlarge.nvidia.gpu: l-x86aavx2-189-704-a10g-8        # g5.48xlarge

  # ---- x86 GPU — L4 (g6 family) ----

  linux.g6.4xlarge.experimental.nvidia.gpu: l-x86aavx2-29-113-l4 # g6.4xlarge
  linux.g6.12xlarge.nvidia.gpu: l-x86aavx2-45-172-l4-4           # g6.12xlarge

  # ---- ARM64 — Graviton ----
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 78-90 / 第 78-90 行

````yaml
  linux.arm64.2xlarge: l-arm64g2-6-32                            # t4g.2xlarge
  linux.arm64.2xlarge.ephemeral: l-arm64g2-6-32                  # t4g.2xlarge
  linux.arm64.m7g.4xlarge: l-arm64g3-16-62                       # m7g.4xlarge
  linux.arm64.m8g.4xlarge: l-arm64g4-16-62                       # m8g.4xlarge
  linux.arm64.r7g.12xlarge.memory: l-arm64g3-61-463              # r7g.12xlarge
  linux.arm64.m7g.metal: l-barm64g4-62-226                       # m7g.metal


  # ---- x86 GPU — B200 (p6 family) ----

  linux.dgx.b200: l-x86iamx-22-225-b200                             # p6-b200.48xlarge (1 GPU)

  # ---- Partner hardwares ----
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。

### Lines 92-95 / 第 92-95 行

````yaml
  linux.idc.xpu: linux.idc.xpu
  linux.rocm.gpu.2: linux.rocm.gpu.2
  linux.rocm.gpu.gfx950.1: linux.rocm.gpu.gfx950.1
  linux.rocm.gpu.gfx950.2: linux.rocm.gpu.gfx950.2
````

- EN: This section describes repository automation behavior for `.github/arc.yaml`.
- CN: 该部分描述 `.github/arc.yaml` 的仓库自动化行为。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。

## Dependencies / 依赖关系
- Reusable actions / 复用 Action: none
- Inline commands / 内联命令: none
- Environment variables / 环境变量: `ARC`, `AMD`, `RAM`, `CPU`, `AVX`, `AMX`, `AVX2`, `GPU`, `A10G`, `ARM64`, ...
- Named jobs or sections / 命名作业或章节: `runner_mapping`
