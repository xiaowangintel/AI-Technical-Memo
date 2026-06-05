# tolerances.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/ir/tolerances.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Default numeric tolerances used to compare different IR implementations. / 用于比较不同 IR 实现的默认数值容差。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-4)
```python
import torch
```
**EN:** This import block brings in PyTorch operator-registration utilities, inspection helpers, and vLLM IR support modules used to define or dispatch custom IR ops inside vLLM's custom IR registry and provider system.
**CN:** 这一组导入语句引入了 PyTorch 算子注册工具、代码检查辅助模块以及 vLLM IR 支撑组件，用于在vLLM 自定义 IR 注册与 provider 系统中定义或派发自定义 IR 算子。

### Constants / assignments (lines 6-36)
```python
ToleranceSpec = dict[torch.dtype, dict[str, float]]
DEFAULT_TOLERANCES: ToleranceSpec = {
    # 52-bit mantissa; machine epsilon ~1.1e-16
    torch.float64: {"atol": 1e-8, "rtol": 1e-8},
    # 23-bit mantissa; machine epsilon ~1.2e-7.
    # Values from PyTorch test_transformers.py reference defaults.
    torch.float32: {"atol": 1e-5, "rtol": 1.3e-6},
    # 10-bit mantissa; machine epsilon ~9.8e-4.
    # Standard tolerance used across vLLM kernel tests.
    torch.float16: {"atol": 1e-3, "rtol": 1e-3},
    # 7-bit mantissa; machine epsilon ~7.8e-3.
    # Wider rtol than float16 to account for the coarser mantissa.
    torch.bfloat16: {"atol": 1e-3, "rtol": 1.6e-2},
    # 3-bit mantissa; machine epsilon ~6.25e-2.
    # Derived from vLLM fp8 kernel tests (merge_attn_states, silu_mul_fp8).
    torch.float8_e4m3fn: {"atol": 1e-1, "rtol": 1e-1},
    # 2-bit mantissa; machine epsilon ~1.25e-1.
    # Wider than e4m3fn due to the smaller mantissa.
    torch.float8_e5m2: {"atol": 2e-1, "rtol": 2e-1},
    # 1-bit mantissa; machine epsilon ~2.5e-1. Packed pair format (x2).
    # Derived from vLLM fp4 tests (test_silu_mul_nvfp4_quant: atol=3e-1).
    torch.float4_e2m1fn_x2: {"atol": 3e-1, "rtol": 3e-1},
    # Integer quantized; off-by-one from rounding is expected.
    # rtol=0 because relative error is meaningless for small integers.
...
```
**EN:** These module-level assignments declare constants or lightweight configuration objects that shape later control flow inside vLLM's custom IR registry and provider system.
**CN:** 这些模块级赋值语句声明了常量或轻量配置对象，用于影响vLLM 自定义 IR 注册与 provider 系统中的后续控制流程。

## Key Concepts / 关键概念
- **Custom IR ops / 自定义 IR 算子**
  - **EN:** The code models vLLM-specific IR operations or lowers them to concrete providers.
  - **CN:** 这些代码建模了 vLLM 特有的 IR 算子，或将其降级到具体 provider。
- **Numerical tolerance specs / 数值容差规格**
  - **EN:** Tolerance tables define how strictly different provider implementations are compared.
  - **CN:** 容差表定义了比较不同 provider 实现时应采用的严格程度。

## Dependencies / 依赖关系
- **PyTorch / Torch 栈**: `import torch`
