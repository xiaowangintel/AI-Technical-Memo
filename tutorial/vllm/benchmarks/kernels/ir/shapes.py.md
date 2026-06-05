# shapes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/ir/shapes.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements shape enumeration utilities, benchmark orchestration, centered around `NUM_TOKENS`, `COMMON_HIDDEN_SIZES`, `SHAPE_CONFIGS`. / 实现与形状枚举工具、基准测试编排相关的逻辑，核心符号包括 `NUM_TOKENS`, `COMMON_HIDDEN_SIZES`, `SHAPE_CONFIGS`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-5)
```python
"""
Shape configurations for IR op benchmarks.
"""
```
**EN:** The module docstring introduces Shape configurations for IR op benchmarks. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Shape configurations for IR op benchmarks 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 7-7)
```python
import torch
```
**EN:** This block gathers third-party packages such as `torch`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 9-29)
```python
NUM_TOKENS = [1, 2, 4, 16, 64, 256, 1024, 4096, 16384]
COMMON_HIDDEN_SIZES = [
    2048,  # Llama 3.2 1B, Qwen 3 MoE 30B-A3B, Gemma 3n
    3072,  # Gemma 7B/9B
    4096,  # Llama 3 8B, Qwen 3 8B, Mistral 7B
    5120,  # Llama 4 Scout 17B-16E
    7168,  # DeepSeek V3
    8192,  # Llama 3 70B
    16384,  # Llama 3 405B
]

# Each entry maps an op name to a list of kwarg dicts that will be passed
# to that op's registered input generator via op.generate_inputs(**kwargs).
SHAPE_CONFIGS: dict[str, list[dict]] = {
    "rms_norm": [
        {"num_tokens": n, "hidden_size": d, "dtype": dtype}
        for dtype in [torch.float16, torch.bfloat16, torch.float32]
        for d in COMMON_HIDDEN_SIZES
        for n in NUM_TOKENS
    ],
}
```
**EN:** This top-level block prepares shared state such as `NUM_TOKENS`, `COMMON_HIDDEN_SIZES`, `SHAPE_CONFIGS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `NUM_TOKENS`, `COMMON_HIDDEN_SIZES`, `SHAPE_CONFIGS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

## Key Concepts / 关键概念
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。

## Dependencies / 依赖关系
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
