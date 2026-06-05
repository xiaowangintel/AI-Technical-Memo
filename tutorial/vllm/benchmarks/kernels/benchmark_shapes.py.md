# benchmark_shapes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_shapes.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, shape enumeration utilities, centered around `WEIGHT_SHAPES`, `WEIGHT_SHAPES_MOE`. / 实现与基准测试编排、形状枚举工具相关的逻辑，核心符号包括 `WEIGHT_SHAPES`, `WEIGHT_SHAPES_MOE`。

## Line-by-Line Analysis / 逐行分析
### Top-level setup (lines 4-94)
```python
WEIGHT_SHAPES = {
    "ideal": [[4 * 256 * 32, 256 * 32]],
    "mistralai/Mistral-7B-v0.1/TP1": [
        [4096, 6144],
        [4096, 4096],
        [4096, 28672],
        [14336, 4096],
    ],
    "mistralai/Mistral-7B-v0.1/TP2": [
        [4096, 3072],
        [2048, 4096],
        [4096, 14336],
        [7168, 4096],
    ],
    "mistralai/Mistral-7B-v0.1/TP4": [
        [4096, 1536],
        [1024, 4096],
        [4096, 7168],
        [3584, 4096],
    ],
    "meta-llama/Llama-2-7b-hf/TP1": [
        [4096, 12288],
        [4096, 4096],
        [4096, 22016],
        [11008, 4096],
    ],
    "meta-llama/Llama-2-7b-hf/TP2": [
        [4096, 6144],
        [2048, 4096],
        [4096, 11008],
        [5504, 4096],
    ],
    "meta-llama/Llama-2-7b-hf/TP4": [
        [4096, 3072],
    # ... omitted for brevity ...
    ],
    "ibm-granite/granite-3.0-1b-a400m": [
        [32, 8, 1024, 1024],
    ],
    "ibm-granite/granite-3.0-3b-a800m": [
        [40, 8, 1024, 1536],
    ],
}
```
**EN:** This top-level block prepares shared state such as `WEIGHT_SHAPES`, `WEIGHT_SHAPES_MOE`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `WEIGHT_SHAPES`, `WEIGHT_SHAPES_MOE`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

## Key Concepts / 关键概念
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。

## Dependencies / 依赖关系
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: none.
- **CN:** 第三方依赖：无。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
