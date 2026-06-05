# troubleshooting.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/usage/troubleshooting.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This is a broad troubleshooting catalog covering slow downloads, model loading delays, OOMs, quality regressions, logging, distributed networking, debugging techniques, and environment-level failures. It serves more as an operator playbook than a single-issue guide.
- **CN:** 这是一份覆盖面很广的排障手册，涉及下载慢、模型加载慢、OOM、生成质量回退、日志、分布式网络、调试技巧以及环境层故障等问题。它更像是面向运维与开发者的操作手册，而非单一问题说明。

## Key Concepts / 关键概念
- **EN:** For download or load hangs, the document recommends isolating network vs. disk bottlenecks and optionally using `--load-format dummy`.
  **CN:** 遇到下载或加载卡顿时，文档建议区分网络瓶颈和磁盘瓶颈，并可借助 `--load-format dummy` 跳过权重加载做隔离测试。
- **EN:** OOM and quality changes are treated as configuration issues, with links to memory-saving options and `generation_config` rollback advice.
  **CN:** 对于 OOM 与生成质量变化，文档更多从配置角度处理，分别给出节省内存方案和回退 `generation_config` 默认值的建议。
- **EN:** Debugging guidance includes extra logging env vars, disabling multiprocessing for breakpoints, and using `--enforce-eager` to localize CUDA graph errors.
  **CN:** 调试建议包括启用更多日志环境变量、关闭多进程以使用断点，以及用 `--enforce-eager` 定位 CUDA graph 错误。
- **EN:** It also includes NCCL/GLOO sanity checks and Python multiprocessing safeguards such as `if __name__ == "__main__":`.
  **CN:** 文档还包含 NCCL/GLOO 自检方法，以及 `if __name__ == "__main__":` 等 Python 多进程防护写法。
