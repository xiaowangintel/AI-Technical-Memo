# test_compaction.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_compaction.py`
- **Purpose / 用途:** Pytest coverage for test compaction; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test compaction 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-5 (module)
```python
1| import pytest
2| import torch
3| from triton_kernels.compaction import compaction, compaction_torch
4| 
5| 
```
**EN:** This block imports `pytest`, `torch`, `triton_kernels.compaction (compaction, compaction_torch)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pytest`, `torch`, `triton_kernels.compaction (compaction, compaction_torch)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 6-12 (test_compaction)
```python
 6| @pytest.mark.parametrize("n_tokens, n_cols, k, p", [
 7|     (8192, 64, 4, 0.5),
 8|     (8192, 64, 4, 1.0),
 9|     (131, 128, 16, 0.6),
10|     (496, 128, 16, 0.),
11| ])
12| def test_compaction(n_tokens, n_cols, k, p, device):
```
**EN:** Defines function `test_compaction(n_tokens, n_cols, k, p, device)` with decorators `pytest.mark.parametrize('n_tokens, n_...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.rand((n_tokens, n_cols), device...`, `yi[:, :k].to`, `torch.randn`, `torch.zeros` to implement its workflow.

**CN:** 定义函数 `test_compaction(n_tokens, n_cols, k, p, device)`，带有装饰器 `pytest.mark.parametrize('n_tokens, n_...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.rand((n_tokens, n_cols), device...`, `yi[:, :k].to`, `torch.randn`, `torch.zeros` 来实现其工作流程.

### Block 3 — Lines 13-13 (test_compaction)
```python
13|     yi = torch.rand((n_tokens, n_cols), device=device).argsort(dim=-1)
```
**EN:** Assigns `yi` and calls `torch.rand((n_tokens, n_cols), device=device).a...`.

**CN:** 将 `yi`，并调用 `torch.rand((n_tokens, n_cols), device=device).a...`.

### Block 4 — Lines 14-14 (test_compaction)
```python
14|     yi = yi[:, :k].to(torch.int32)
```
**EN:** Assigns `yi` and calls `yi[:, :k].to`.

**CN:** 将 `yi`，并调用 `yi[:, :k].to`.

### Block 5 — Lines 15-16 (test_compaction)
```python
15|     yv = torch.randn((n_tokens, k), dtype=torch.bfloat16, device=device)
16|     # "drop" indices from yi with probability `p`
```
**EN:** Assigns `yv` and calls `torch.randn`.

**CN:** 将 `yv`，并调用 `torch.randn`.

### Block 6 — Lines 17-17 (test_compaction)
```python
17|     mask = torch.zeros((n_tokens, n_cols), dtype=torch.int32, device=device)
```
**EN:** Assigns `mask` and calls `torch.zeros`.

**CN:** 将 `mask`，并调用 `torch.zeros`.

### Block 7 — Lines 18-18 (test_compaction)
```python
18|     keep = (torch.rand(yi.shape, device=device) < p)
```
**EN:** Assigns `keep` and evaluates `torch.rand(yi.shape, device=device) < p`.

**CN:** 将 `keep` and 计算 `torch.rand(yi.shape, device=device) < p`.

### Block 8 — Lines 19-21 (test_compaction)
```python
19|     if keep.any():
20|         rows = torch.arange(yi.size(0), device=device).unsqueeze(1).expand_as(yi)
21|         mask[rows[keep], yi[keep]] = 1
```
**EN:** Checks `keep.any()`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `keep.any()`. 真分支主要准备中间值; 准备中间值.

### Block 9 — Lines 22-22 (test_compaction)
```python
22|     chunks = mask.view(*mask.shape[:-1], -1, 32)
```
**EN:** Assigns `chunks` and calls `mask.view`.

**CN:** 将 `chunks`，并调用 `mask.view`.

### Block 10 — Lines 23-23 (test_compaction)
```python
23|     weights = (1 << torch.arange(32, dtype=torch.int32, device=device))
```
**EN:** Assigns `weights` and evaluates `1 << torch.arange(32, dtype=torch.int32, device=device)`.

**CN:** 将 `weights` and 计算 `1 << torch.arange(32, dtype=torch.int32, device=device)`.

### Block 11 — Lines 24-24 (test_compaction)
```python
24|     bitmask = (chunks.int() * weights).sum(dim=-1)
```
**EN:** Assigns `bitmask` and calls `(chunks.int() * weights).sum`.

**CN:** 将 `bitmask`，并调用 `(chunks.int() * weights).sum`.

### Block 12 — Lines 25-25 (test_compaction)
```python
25|     yv_ref, yi_ref = compaction_torch(yv, yi, bitmask)
```
**EN:** Assigns `yv_ref`, `yi_ref` and calls `compaction_torch`.

**CN:** 将 `yv_ref`, `yi_ref`，并调用 `compaction_torch`.

### Block 13 — Lines 26-26 (test_compaction)
```python
26|     yv_tri, yi_tri = compaction(yv, yi, bitmask)
```
**EN:** Assigns `yv_tri`, `yi_tri` and calls `compaction`.

**CN:** 将 `yv_tri`, `yi_tri`，并调用 `compaction`.

### Block 14 — Lines 27-27 (test_compaction)
```python
27|     assert torch.all(yi_ref == yi_tri)
```
**EN:** Asserts `torch.all(yi_ref == yi_tri)` to enforce invariants.

**CN:** 断言 `torch.all(yi_ref == yi_tri)` 以确保不变量成立。

### Block 15 — Lines 28-28 (test_compaction)
```python
28|     assert torch.all(yv_ref == yv_tri)
```
**EN:** Asserts `torch.all(yv_ref == yv_tri)` to enforce invariants.

**CN:** 断言 `torch.all(yv_ref == yv_tri)` 以确保不变量成立。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `test_compaction`.
  **CN:** 主要符号：`test_compaction`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。

## Dependencies / 依赖关系
- **EN:** External modules: `pytest`, `torch`.
  **CN:** 外部模块：`pytest`, `torch`。
- **EN:** Internal modules: `triton_kernels.compaction (compaction, compaction_torch)`.
  **CN:** 内部模块：`triton_kernels.compaction (compaction, compaction_torch)`。
