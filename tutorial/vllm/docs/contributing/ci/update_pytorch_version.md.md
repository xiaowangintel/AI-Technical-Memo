# update_pytorch_version.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/ci/update_pytorch_version.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Explains the project policy of tracking the latest PyTorch stable release and encourages testing release candidates early to discover incompatibilities before the final release ships.
- **CN:** 本文解释了项目跟进最新 PyTorch 稳定版的策略，并鼓励尽早测试 RC 版本，以便在正式版发布前发现兼容性问题。
- **EN:** The guide combines package index configuration, CUDA-specific wheel sources, CI file updates, manual Buildkite cache warming, and staged platform rollouts.
- **CN:** 文档把软件包索引配置、CUDA 专用 wheel 源、CI 文件更新、Buildkite 手动预热缓存以及分平台推进策略整合在一起。

## Key Concepts / 关键概念
- **EN:** Release-candidate testing — Testing RC wheels reduces the chance that vLLM is broken only after PyTorch final release.
- **CN:** RC 测试 — 提前测试 RC wheel 可以减少 vLLM 在 PyTorch 正式发布后才暴露问题的风险。
- **EN:** `uv` index strategy — The `unsafe-best-match` strategy is required so `uv` can resolve the intended RC wheels.
- **CN:** `uv` 索引策略 — 需要使用 `unsafe-best-match`，这样 `uv` 才能解析到目标 RC wheel。
- **EN:** CUDA wheel indexes — Different hardware targets rely on explicit PyTorch wheel indexes such as `cu128`, `cpu`, or ROCm URLs.
- **CN:** CUDA wheel 索引 — 不同硬件目标依赖明确指定的 PyTorch wheel 索引，例如 `cu128`、`cpu` 或 ROCm 地址。
- **EN:** Buildkite cache warm-up — Manual builds both validate the new stack and populate sccache so later CI runs are faster.
- **CN:** Buildkite 缓存预热 — 手动构建既能验证新组合，也能填充 sccache，让后续 CI 更快。
