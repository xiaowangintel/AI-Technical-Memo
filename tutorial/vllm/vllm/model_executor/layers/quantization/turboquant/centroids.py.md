# centroids.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/turboquant/centroids.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides helpers such as `_gaussian_pdf`, `_trapz`, `solve_lloyd_max` for quantization backends, schemes, and utilities. / 提供诸如 `_gaussian_pdf`, `_trapz`, `solve_lloyd_max` 之类的辅助函数，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-10)
```python
"""Lloyd-Max optimal scalar quantizer for TurboQuant.

After rotating a d-dimensional unit vector by a random orthogonal matrix,
each coordinate approximately follows N(0, 1/d) for d >= 64.
We solve the Lloyd-Max conditions to find optimal centroids.

Based on: turboquant-pytorch/lloyd_max.py (Zandieh et al.)
"""
```
**EN:** This docstring gives the module author's high-level intent: Lloyd-Max optimal scalar quantizer for TurboQuant. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Lloyd-Max optimal scalar quantizer for TurboQuant. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 12-15)
```python
import math
from functools import lru_cache

import torch
```
**EN:** This opening block pulls in external dependencies such as `math`, `functools`, `torch` and internal modules such as no internal imports. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `math`, `functools`, `torch`）以及内部模块（如 no internal imports）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `_gaussian_pdf` (lines 18-19)
```python
def _gaussian_pdf(x: float, sigma2: float) -> float:
    return (1.0 / math.sqrt(2 * math.pi * sigma2)) * math.exp(-x * x / (2 * sigma2))
```
**EN:** Defines function `_gaussian_pdf` with signature `_gaussian_pdf(x: float, sigma2: float) -> float`. It mainly works with `x`, `sigma2`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `math.exp`, `math.sqrt`.
**CN:** 定义函数 `_gaussian_pdf`，其签名为 `_gaussian_pdf(x: float, sigma2: float) -> float`。它主要围绕 `x`, `sigma2` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `math.exp`, `math.sqrt`。

### Function `_trapz` (lines 22-28)
```python
def _trapz(f, a: float, b: float, n: int = 200) -> float:
    """Trapezoidal numerical integration (replaces scipy.integrate.quad)."""
    h = (b - a) / n
    result = 0.5 * (f(a) + f(b))
    for i in range(1, n):
        result += f(a + i * h)
    return result * h
```
**EN:** Defines function `_trapz` with signature `_trapz(f, a: float, b: float, n: int=200) -> float`. It mainly works with `f`, `a`, `b`, `n`; implements one step in the quantized-weight execution flow. The body uses iteration. Key calls include `range`, `f`.
**CN:** 定义函数 `_trapz`，其签名为 `_trapz(f, a: float, b: float, n: int=200) -> float`。它主要围绕 `f`, `a`, `b`, `n` 展开；实现量化权重执行流程中的一个步骤。函数体包含循环处理。关键调用包括 `range`, `f`。

### Function `solve_lloyd_max` (lines 31-79)
```python
def solve_lloyd_max(
    d: int,
    bits: int,
    max_iter: int = 200,
    tol: float = 1e-10,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Solve Lloyd-Max optimal quantizer for N(0, 1/d) distribution.

    Args:
        d: Vector dimension (determines variance = 1/d).
        bits: Number of quantization bits.
        max_iter: Maximum Lloyd-Max iterations.
        tol: Convergence tolerance.

    Returns:
        centroids: Sorted tensor of 2^bits optimal centroids.
        boundaries: Sorted tensor of 2^bits - 1 decision boundaries.
    """
    n_levels = 2**bits
    sigma2 = 1.0 / d
    sigma = math.sqrt(sigma2)

    def pdf(x):
        return _gaussian_pdf(x, sigma2)

    lo, hi = -3.5 * sigma, 3.5 * sigma
    centroids = [lo + (hi - lo) * (i + 0.5) / n_levels for i in range(n_levels)]

    for _ in range(max_iter):
        boundaries = [
            (centroids[i] + centroids[i + 1]) / 2.0 for i in range(n_levels - 1)
        ]
        edges = [lo * 3] + boundaries + [hi * 3]
        new_centroids = []
        for i in range(n_levels):
            a, b = edges[i], edges[i + 1]
            num = _trapz(lambda x: x * pdf(x), a, b)
            den = _trapz(pdf, a, b)
            new_centroids.append(num / den if den > 1e-15 else centroids[i])

        if max(abs(new_centroids[i] - centroids[i]) for i in range(n_levels)) < tol:
            break
        centroids = new_centroids

    boundaries = [(centroids[i] + centroids[i + 1]) / 2.0 for i in range(n_levels - 1)]
    return (
        torch.tensor(centroids, dtype=torch.float32),
        torch.tensor(boundaries, dtype=torch.float32),
    )
```
**EN:** Defines function `solve_lloyd_max` with signature `solve_lloyd_max(d: int, bits: int, max_iter: int=200, tol: float=1e-10) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `d`, `bits`, `max_iter`, `tol`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, comprehensions, tensor/kernel operations. Key calls include `math.sqrt`, `range`, `_gaussian_pdf`, `torch.tensor`, `_trapz`, `new_centroids.append`.
**CN:** 定义函数 `solve_lloyd_max`，其签名为 `solve_lloyd_max(d: int, bits: int, max_iter: int=200, tol: float=1e-10) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `d`, `bits`, `max_iter`, `tol` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、推导式、张量或内核操作。关键调用包括 `math.sqrt`, `range`, `_gaussian_pdf`, `torch.tensor`, `_trapz`, `new_centroids.append`。

### Function `get_centroids` (lines 83-86)
```python
def get_centroids(d: int, bits: int) -> torch.Tensor:
    """Get precomputed Lloyd-Max centroids (cached)."""
    centroids, _ = solve_lloyd_max(d, bits)
    return centroids
```
**EN:** Defines function `get_centroids` with signature `get_centroids(d: int, bits: int) -> torch.Tensor`. It mainly works with `d`, `bits`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `lru_cache`, `solve_lloyd_max`.
**CN:** 定义函数 `get_centroids`，其签名为 `get_centroids(d: int, bits: int) -> torch.Tensor`。它主要围绕 `d`, `bits` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `lru_cache`, `solve_lloyd_max`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `_gaussian_pdf`, `_trapz`, `solve_lloyd_max`, `get_centroids` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_gaussian_pdf`, `_trapz`, `solve_lloyd_max`, `get_centroids` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `math`, `functools`, `torch`
- **Internal / 内部**: None / 无
