# test_opt_flags_nvidia.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_matmul_details/test_opt_flags_nvidia.py`
- **Purpose / 用途:** Pytest coverage for test opt flags nvidia; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test opt flags nvidia 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-15 (module)
```python
 1| import pytest
 2| import torch
 3| from triton._internal_testing import is_cuda
 4| 
 5| from triton_kernels.matmul import matmul, matmul_torch, PrecisionConfig
 6| from triton_kernels.matmul_details.opt_flags import InapplicableConstraint, scoped_opt_flags_constraints
 7| from triton_kernels.matmul_details.opt_flags_details import opt_flags_nvidia
 8| from triton_kernels.numerics_details.mxfp import MXFP_BLOCK_SIZE, downcast_to_mxfp
 9| from triton_kernels.tensor import FP4, UINT8, Storage, Tensor, convert_layout, wrap_torch_tensor
10| from triton_kernels.tensor_details import layout
11| from triton_kernels.tensor_details.layout import BlackwellMX4ValueShuffledLayout
12| from triton_kernels.tensor_details.layout_details.blackwell_scale import BlackwellMXScaleLayout
13| from triton_kernels.testing import assert_close
14| 
15| 
```
**EN:** This block imports `pytest`, `torch`, `triton._internal_testing (is_cuda)`, `triton_kernels.matmul (matmul, matmul_torch, PrecisionConfig)`, `triton_kernels.matmul_details.opt_flags (InapplicableConstraint, scoped_opt_flags_constraints)`, `triton_kernels.matmul_details.opt_flags_details (opt_flags_nvidia)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (FP4, UINT8, Storage, Tensor)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pytest`, `torch`, `triton._internal_testing (is_cuda)`, `triton_kernels.matmul (matmul, matmul_torch, PrecisionConfig)`, `triton_kernels.matmul_details.opt_flags (InapplicableConstraint, scoped_opt_flags_constraints)`, `triton_kernels.matmul_details.opt_flags_details (opt_flags_nvidia)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (FP4, UINT8, Storage, Tensor)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 16-16 (_make_blackwell_scale_tensor)
```python
16| def _make_blackwell_scale_tensor():
```
**EN:** Defines function `_make_blackwell_scale_tensor()` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `Storage`, `Tensor`, `torch.empty`, `BlackwellMXScaleLayout` to implement its workflow.

**CN:** 定义函数 `_make_blackwell_scale_tensor()`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `Storage`, `Tensor`, `torch.empty`, `BlackwellMXScaleLayout` 来实现其工作流程.

### Block 3 — Lines 17-17 (_make_blackwell_scale_tensor)
```python
17|     scale_storage = Storage(torch.empty((1, 128), dtype=torch.uint8), BlackwellMXScaleLayout())
```
**EN:** Assigns `scale_storage` and calls `Storage`.

**CN:** 将 `scale_storage`，并调用 `Storage`.

### Block 4 — Lines 18-20 (_make_blackwell_scale_tensor)
```python
18|     return Tensor(scale_storage, dtype=UINT8)
19| 
20| 
```
**EN:** Returns `Tensor(scale_storage, dtype=UINT8)`.

**CN:** 返回 `Tensor(scale_storage, dtype=UINT8)`.

### Block 5 — Lines 21-21 (_make_blackwell_mxfp4_weight)
```python
21| def _make_blackwell_mxfp4_weight(device, k, n):
```
**EN:** Defines function `_make_blackwell_mxfp4_weight(device, k, n)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `downcast_to_mxfp`, `wrap_torch_tensor`, `layout.make_default_matmul_mxfp4_w_sc...`, `convert_layout`, `torch.randn` to implement its workflow.

**CN:** 定义函数 `_make_blackwell_mxfp4_weight(device, k, n)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `downcast_to_mxfp`, `wrap_torch_tensor`, `layout.make_default_matmul_mxfp4_w_sc...`, `convert_layout`, `torch.randn` 来实现其工作流程.

### Block 6 — Lines 22-22 (_make_blackwell_mxfp4_weight)
```python
22|     weight_fp = torch.randn((n, k), device=device, dtype=torch.bfloat16).T
```
**EN:** Assigns `weight_fp` and references `torch.randn((n, k), device=device, dtype=torch.bfloat16).T`.

**CN:** 将 `weight_fp` and 引用 `torch.randn((n, k), device=device, dtype=torch.bfloat16).T`.

### Block 7 — Lines 23-23 (_make_blackwell_mxfp4_weight)
```python
23|     weight_val, weight_scale = downcast_to_mxfp(weight_fp, torch.uint8, axis=-2)
```
**EN:** Assigns `weight_val`, `weight_scale` and calls `downcast_to_mxfp`.

**CN:** 将 `weight_val`, `weight_scale`，并调用 `downcast_to_mxfp`.

### Block 8 — Lines 24-24 (_make_blackwell_mxfp4_weight)
```python
24|     weight_val = wrap_torch_tensor(weight_val, dtype=FP4)
```
**EN:** Assigns `weight_val` and calls `wrap_torch_tensor`.

**CN:** 将 `weight_val`，并调用 `wrap_torch_tensor`.

### Block 9 — Lines 25-25 (_make_blackwell_mxfp4_weight)
```python
25|     weight_scale = wrap_torch_tensor(weight_scale, dtype=UINT8)
```
**EN:** Assigns `weight_scale` and calls `wrap_torch_tensor`.

**CN:** 将 `weight_scale`，并调用 `wrap_torch_tensor`.

### Block 10 — Lines 26-26 (_make_blackwell_mxfp4_weight)
```python
26|     weight_scale_layout = layout.make_default_matmul_mxfp4_w_scale_layout(mx_axis=-2)
```
**EN:** Assigns `weight_scale_layout` and calls `layout.make_default_matmul_mxfp4_w_scale_layout`.

**CN:** 将 `weight_scale_layout`，并调用 `layout.make_default_matmul_mxfp4_w_scale_layout`.

### Block 11 — Lines 27-27 (_make_blackwell_mxfp4_weight)
```python
27|     weight_scale = convert_layout(weight_scale, weight_scale_layout)
```
**EN:** Assigns `weight_scale` and calls `convert_layout`.

**CN:** 将 `weight_scale`，并调用 `convert_layout`.

### Block 12 — Lines 28-30 (_make_blackwell_mxfp4_weight)
```python
28|     return weight_val, weight_scale
29| 
30| 
```
**EN:** Returns `(weight_val, weight_scale)`.

**CN:** 返回 `(weight_val, weight_scale)`.

### Block 13 — Lines 31-31 (_make_batched_blackwell_mxfp4_weight)
```python
31| def _make_batched_blackwell_mxfp4_weight(device, batch_size, k, n):
```
**EN:** Defines function `_make_batched_blackwell_mxfp4_weight(device, batch_size, k, n)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.randn((batch_size, n, k), devic...`, `downcast_to_mxfp`, `wrap_torch_tensor`, `layout.make_default_matmul_mxfp4_w_sc...`, `convert_layout` to implement its workflow.

**CN:** 定义函数 `_make_batched_blackwell_mxfp4_weight(device, batch_size, k, n)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `torch.randn((batch_size, n, k), devic...`, `downcast_to_mxfp`, `wrap_torch_tensor`, `layout.make_default_matmul_mxfp4_w_sc...`, `convert_layout` 来实现其工作流程.

### Block 14 — Lines 32-32 (_make_batched_blackwell_mxfp4_weight)
```python
32|     weight_fp = torch.randn((batch_size, n, k), device=device, dtype=torch.bfloat16).transpose(-2, -1)
```
**EN:** Assigns `weight_fp` and calls `torch.randn((batch_size, n, k), device=device, ...`.

**CN:** 将 `weight_fp`，并调用 `torch.randn((batch_size, n, k), device=device, ...`.

### Block 15 — Lines 33-33 (_make_batched_blackwell_mxfp4_weight)
```python
33|     weight_val, weight_scale = downcast_to_mxfp(weight_fp, torch.uint8, axis=-2)
```
**EN:** Assigns `weight_val`, `weight_scale` and calls `downcast_to_mxfp`.

**CN:** 将 `weight_val`, `weight_scale`，并调用 `downcast_to_mxfp`.

### Block 16 — Lines 34-34 (_make_batched_blackwell_mxfp4_weight)
```python
34|     weight_val = wrap_torch_tensor(weight_val, dtype=FP4)
```
**EN:** Assigns `weight_val` and calls `wrap_torch_tensor`.

**CN:** 将 `weight_val`，并调用 `wrap_torch_tensor`.

### Block 17 — Lines 35-35 (_make_batched_blackwell_mxfp4_weight)
```python
35|     weight_scale = wrap_torch_tensor(weight_scale, dtype=UINT8)
```
**EN:** Assigns `weight_scale` and calls `wrap_torch_tensor`.

**CN:** 将 `weight_scale`，并调用 `wrap_torch_tensor`.

### Block 18 — Lines 36-36 (_make_batched_blackwell_mxfp4_weight)
```python
36|     weight_scale_layout = layout.make_default_matmul_mxfp4_w_scale_layout(mx_axis=-2)
```
**EN:** Assigns `weight_scale_layout` and calls `layout.make_default_matmul_mxfp4_w_scale_layout`.

**CN:** 将 `weight_scale_layout`，并调用 `layout.make_default_matmul_mxfp4_w_scale_layout`.

### Block 19 — Lines 37-37 (_make_batched_blackwell_mxfp4_weight)
```python
37|     weight_scale = convert_layout(weight_scale, weight_scale_layout)
```
**EN:** Assigns `weight_scale` and calls `convert_layout`.

**CN:** 将 `weight_scale`，并调用 `convert_layout`.

### Block 20 — Lines 38-40 (_make_batched_blackwell_mxfp4_weight)
```python
38|     return weight_val, weight_scale
39| 
40| 
```
**EN:** Returns `(weight_val, weight_scale)`.

**CN:** 返回 `(weight_val, weight_scale)`.

### Block 21 — Lines 41-41 (_shuffle_blackwell_mxfp4_weight)
```python
41| def _shuffle_blackwell_mxfp4_weight(weight):
```
**EN:** Defines function `_shuffle_blackwell_mxfp4_weight(weight)` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `BlackwellMX4ValueShuffledLayout`, `convert_layout` to implement its workflow.

**CN:** 定义函数 `_shuffle_blackwell_mxfp4_weight(weight)`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `BlackwellMX4ValueShuffledLayout`, `convert_layout` 来实现其工作流程.

### Block 22 — Lines 42-42 (_shuffle_blackwell_mxfp4_weight)
```python
42|     shuffled_layout = BlackwellMX4ValueShuffledLayout()
```
**EN:** Assigns `shuffled_layout` and calls `BlackwellMX4ValueShuffledLayout`.

**CN:** 将 `shuffled_layout`，并调用 `BlackwellMX4ValueShuffledLayout`.

### Block 23 — Lines 43-45 (_shuffle_blackwell_mxfp4_weight)
```python
43|     return convert_layout(weight, shuffled_layout)
44| 
45| 
```
**EN:** Returns `convert_layout(weight, shuffled_layout)`.

**CN:** 返回 `convert_layout(weight, shuffled_layout)`.

### Block 24 — Lines 46-53 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
46| @pytest.mark.parametrize(
47|     "constraints",
48|     [
49|         pytest.param({"is_persistent": False}, id="regular"),
50|         pytest.param({"is_persistent": True, "block_m": 128}, id="persistent"),
51|     ],
52| )
53| def test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked(device, constraints):
```
**EN:** Defines function `test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked(device, constraints)` with decorators `pytest.mark.parametrize('constraints'...` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; invokes `torch.manual_seed`. It uses calls such as `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randn`, `downcast_to_mxfp`, `layout.make_default_matmul_mxfp4_w_la...` to implement its workflow.

**CN:** 定义函数 `test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked(device, constraints)`，带有装饰器 `pytest.mark.parametrize('constraints'...`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; invokes `torch.manual_seed`. 其中会调用 `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randn`, `downcast_to_mxfp`, `layout.make_default_matmul_mxfp4_w_la...` 来实现其工作流程.

### Block 25 — Lines 54-55 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
54|     if device != "cuda" or not torch.cuda.is_available() or not is_cuda():
55|         pytest.skip("requires CUDA")
```
**EN:** Checks `device != 'cuda' or not torch.cuda.is_available() or (not is_cuda())`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `device != 'cuda' or not torch.cuda.is_available() or (not is_cuda())`. 真分支主要invokes `pytest.skip`.

### Block 26 — Lines 56-58 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
56|     if torch.cuda.get_device_capability()[0] != 9:
57|         pytest.skip("requires Hopper")
58| 
```
**EN:** Checks `torch.cuda.get_device_capability()[0] != 9`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `torch.cuda.get_device_capability()[0] != 9`. 真分支主要invokes `pytest.skip`.

### Block 27 — Lines 59-62 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
59|     torch.manual_seed(0)
60|     # k=1504 gives 47 MXFP scale columns along K. Hopper scale swizzling pads
61|     # that to 48 columns, so dirtying swizzled zero bytes targets the K-tail
62|     # scale padding. n=256 is one full Hopper N tile, avoiding unrelated N padding.
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 28 — Lines 63-63 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
63|     m, k, n = 64, 1504, 256
```
**EN:** Assigns `m`, `k`, `n` and builds a tuple.

**CN:** 将 `m`, `k`, `n` and 构造一个元组.

### Block 29 — Lines 64-64 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
64|     a = torch.randn((m, k), device=device, dtype=torch.bfloat16)
```
**EN:** Assigns `a` and calls `torch.randn`.

**CN:** 将 `a`，并调用 `torch.randn`.

### Block 30 — Lines 65-65 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
65|     weight_fp = torch.randn((n, k), device=device, dtype=torch.bfloat16).T
```
**EN:** Assigns `weight_fp` and references `torch.randn((n, k), device=device, dtype=torch.bfloat16).T`.

**CN:** 将 `weight_fp` and 引用 `torch.randn((n, k), device=device, dtype=torch.bfloat16).T`.

### Block 31 — Lines 66-67 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
66|     weight_val, weight_scale = downcast_to_mxfp(weight_fp, torch.uint8, axis=-2)
67| 
```
**EN:** Assigns `weight_val`, `weight_scale` and calls `downcast_to_mxfp`.

**CN:** 将 `weight_val`, `weight_scale`，并调用 `downcast_to_mxfp`.

### Block 32 — Lines 68-68 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
68|     value_layout = layout.make_default_matmul_mxfp4_w_layout(mx_axis=-2)
```
**EN:** Assigns `value_layout` and calls `layout.make_default_matmul_mxfp4_w_layout`.

**CN:** 将 `value_layout`，并调用 `layout.make_default_matmul_mxfp4_w_layout`.

### Block 33 — Lines 69-69 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
69|     scale_layout = layout.make_default_matmul_mxfp4_w_scale_layout(mx_axis=-2, num_warps=8)
```
**EN:** Assigns `scale_layout` and calls `layout.make_default_matmul_mxfp4_w_scale_layout`.

**CN:** 将 `scale_layout`，并调用 `layout.make_default_matmul_mxfp4_w_scale_layout`.

### Block 34 — Lines 70-70 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
70|     b = convert_layout(wrap_torch_tensor(weight_val, dtype=FP4), value_layout)
```
**EN:** Assigns `b` and calls `convert_layout`.

**CN:** 将 `b`，并调用 `convert_layout`.

### Block 35 — Lines 71-73 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
71|     b_scale = convert_layout(wrap_torch_tensor(weight_scale, dtype=UINT8), scale_layout)
72| 
73|     # Ones remain ones through the scale swizzle; zeros identify padded bytes.
```
**EN:** Assigns `b_scale` and calls `convert_layout`.

**CN:** 将 `b_scale`，并调用 `convert_layout`.

### Block 36 — Lines 74-77 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
74|     scale_padding = convert_layout(
75|         wrap_torch_tensor(torch.ones_like(weight_scale), dtype=UINT8),
76|         scale_layout,
77|     ).storage.data == 0
```
**EN:** Assigns `scale_padding` and evaluates `convert_layout(wrap_torch_tensor(torch.ones_like(weight_scale), dty...`.

**CN:** 将 `scale_padding` and 计算 `convert_layout(wrap_torch_tensor(torch.ones_like(weight_scale), dty...`.

### Block 37 — Lines 78-79 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
78|     assert bool(scale_padding.any().item())
79| 
```
**EN:** Asserts `bool(scale_padding.any().item())` to enforce invariants.

**CN:** 断言 `bool(scale_padding.any().item())` 以确保不变量成立。

### Block 38 — Lines 80-80 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
80|     b_scale_dirty_padding = convert_layout(wrap_torch_tensor(weight_scale.clone(), dtype=UINT8), scale_layout)
```
**EN:** Assigns `b_scale_dirty_padding` and calls `convert_layout`.

**CN:** 将 `b_scale_dirty_padding`，并调用 `convert_layout`.

### Block 39 — Lines 81-82 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
81|     b_scale_dirty_padding.storage.data[scale_padding] = 0xFF
82| 
```
**EN:** Assigns `b_scale_dirty_padding.storage.data[sc...` and stores constant `255`.

**CN:** 将 `b_scale_dirty_padding.storage.data[sc...` and 保存常量 `255`.

### Block 40 — Lines 83-86 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
83|     precision_kwargs = {
84|         "b_microblock_size": MXFP_BLOCK_SIZE.value,
85|         "out_dtype": a.dtype,
86|     }
```
**EN:** Assigns `precision_kwargs` and builds a dictionary.

**CN:** 将 `precision_kwargs` and 构造一个字典.

### Block 41 — Lines 87-103 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
 87|     try:
 88|         with scoped_opt_flags_constraints(constraints):
 89|             expected = matmul(
 90|                 a,
 91|                 b,
 92|                 None,
 93|                 precision_config=PrecisionConfig(b_mx_scale=b_scale, **precision_kwargs),
 94|             )
 95|             actual = matmul(
 96|                 a,
 97|                 b,
 98|                 None,
 99|                 precision_config=PrecisionConfig(b_mx_scale=b_scale_dirty_padding, **precision_kwargs),
100|             )
101|     except (InapplicableConstraint, NotImplementedError) as e:
102|         pytest.skip(f"inapplicable opt_flags constraint {e}")
103| 
```
**EN:** Wraps code that mainly uses contextual resources with exception/finally handling.

**CN:** 包装代码 that mainly 使用上下文资源 以及异常/收尾处理.

### Block 42 — Lines 104-106 (test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked)
```python
104|     torch.testing.assert_close(actual, expected, rtol=0, atol=0)
105| 
106| 
```
**EN:** Calls `torch.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `torch.testing.assert_close` ，用于副作用、注册或校验。

### Block 43 — Lines 107-108 (test_compute_block_n_blackwell_scale_aligns_to_128)
```python
107| @pytest.mark.parametrize("n, expected", [(64, 128), (200, 256)])
108| def test_compute_block_n_blackwell_scale_aligns_to_128(n, expected):
```
**EN:** Defines function `test_compute_block_n_blackwell_scale_aligns_to_128(n, expected)` with decorators `pytest.mark.parametrize('n, expected'...` for this module. The body mainly prepares intermediate values; prepares intermediate values; checks invariants. It uses calls such as `pytest.mark.parametrize`, `PrecisionConfig`, `opt_flags_nvidia.compute_block_n`, `_make_blackwell_scale_tensor` to implement its workflow.

**CN:** 定义函数 `test_compute_block_n_blackwell_scale_aligns_to_128(n, expected)`，带有装饰器 `pytest.mark.parametrize('n, expected'...`，供本模块使用. 主体主要准备中间值; 准备中间值; 检查不变量. 其中会调用 `pytest.mark.parametrize`, `PrecisionConfig`, `opt_flags_nvidia.compute_block_n`, `_make_blackwell_scale_tensor` 来实现其工作流程.

### Block 44 — Lines 109-112 (test_compute_block_n_blackwell_scale_aligns_to_128)
```python
109|     precision_config = PrecisionConfig(
110|         b_mx_scale=_make_blackwell_scale_tensor(),
111|         b_microblock_size=MXFP_BLOCK_SIZE.value,
112|     )
```
**EN:** Assigns `precision_config` and calls `PrecisionConfig`.

**CN:** 将 `precision_config`，并调用 `PrecisionConfig`.

### Block 45 — Lines 113-113 (test_compute_block_n_blackwell_scale_aligns_to_128)
```python
113|     block_n, block_n_tma = opt_flags_nvidia.compute_block_n(n, None, precision_config)
```
**EN:** Assigns `block_n`, `block_n_tma` and calls `opt_flags_nvidia.compute_block_n`.

**CN:** 将 `block_n`, `block_n_tma`，并调用 `opt_flags_nvidia.compute_block_n`.

### Block 46 — Lines 114-116 (test_compute_block_n_blackwell_scale_aligns_to_128)
```python
114|     assert block_n == block_n_tma == expected
115| 
116| 
```
**EN:** Asserts `block_n == block_n_tma == expected` to enforce invariants.

**CN:** 断言 `block_n == block_n_tma == expected` 以确保不变量成立。

### Block 47 — Lines 117-117 (test_matmul_blackwell_scale_small_n)
```python
117| def test_matmul_blackwell_scale_small_n(device):
```
**EN:** Defines function `test_matmul_blackwell_scale_small_n(device)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; invokes `torch.manual_seed`. It uses calls such as `torch.manual_seed`, `torch.randn`, `_make_blackwell_mxfp4_weight`, `PrecisionConfig`, `matmul` to implement its workflow.

**CN:** 定义函数 `test_matmul_blackwell_scale_small_n(device)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; invokes `torch.manual_seed`. 其中会调用 `torch.manual_seed`, `torch.randn`, `_make_blackwell_mxfp4_weight`, `PrecisionConfig`, `matmul` 来实现其工作流程.

### Block 48 — Lines 118-119 (test_matmul_blackwell_scale_small_n)
```python
118|     if device != "cuda" or not torch.cuda.is_available():
119|         pytest.skip("requires CUDA")
```
**EN:** Checks `device != 'cuda' or not torch.cuda.is_available()`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `device != 'cuda' or not torch.cuda.is_available()`. 真分支主要invokes `pytest.skip`.

### Block 49 — Lines 120-122 (test_matmul_blackwell_scale_small_n)
```python
120|     if torch.cuda.get_device_capability()[0] < 10:
121|         pytest.skip("requires Blackwell or newer")
122| 
```
**EN:** Checks `torch.cuda.get_device_capability()[0] < 10`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `torch.cuda.get_device_capability()[0] < 10`. 真分支主要invokes `pytest.skip`.

### Block 50 — Lines 123-123 (test_matmul_blackwell_scale_small_n)
```python
123|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 51 — Lines 124-124 (test_matmul_blackwell_scale_small_n)
```python
124|     m, n, k = 128, 64, 128
```
**EN:** Assigns `m`, `n`, `k` and builds a tuple.

**CN:** 将 `m`, `n`, `k` and 构造一个元组.

### Block 52 — Lines 125-125 (test_matmul_blackwell_scale_small_n)
```python
125|     a = torch.randn((m, k), device=device, dtype=torch.bfloat16)
```
**EN:** Assigns `a` and calls `torch.randn`.

**CN:** 将 `a`，并调用 `torch.randn`.

### Block 53 — Lines 126-126 (test_matmul_blackwell_scale_small_n)
```python
126|     b, b_scale = _make_blackwell_mxfp4_weight(device, k, n)
```
**EN:** Assigns `b`, `b_scale` and calls `_make_blackwell_mxfp4_weight`.

**CN:** 将 `b`, `b_scale`，并调用 `_make_blackwell_mxfp4_weight`.

### Block 54 — Lines 127-131 (test_matmul_blackwell_scale_small_n)
```python
127|     precision_config = PrecisionConfig(
128|         b_mx_scale=b_scale,
129|         b_microblock_size=MXFP_BLOCK_SIZE.value,
130|         out_dtype=a.dtype,
131|     )
```
**EN:** Assigns `precision_config` and calls `PrecisionConfig`.

**CN:** 将 `precision_config`，并调用 `PrecisionConfig`.

### Block 55 — Lines 132-132 (test_matmul_blackwell_scale_small_n)
```python
132|     tri_y = matmul(a, b, None, precision_config=precision_config)
```
**EN:** Assigns `tri_y` and calls `matmul`.

**CN:** 将 `tri_y`，并调用 `matmul`.

### Block 56 — Lines 133-133 (test_matmul_blackwell_scale_small_n)
```python
133|     ref_y = matmul_torch(a.to(torch.bfloat16), b, None, precision_config=precision_config)
```
**EN:** Assigns `ref_y` and calls `matmul_torch`.

**CN:** 将 `ref_y`，并调用 `matmul_torch`.

### Block 57 — Lines 134-136 (test_matmul_blackwell_scale_small_n)
```python
134|     assert_close(ref_y, tri_y, maxtol=3e-2, rmstol=None)
135| 
136| 
```
**EN:** Calls `assert_close` for side effects, registration, or validation.

**CN:** 调用 `assert_close` ，用于副作用、注册或校验。

### Block 58 — Lines 137-137 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
137| def test_matmul_blackwell_shuffled_mxfp4_weight(device):
```
**EN:** Defines function `test_matmul_blackwell_shuffled_mxfp4_weight(device)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; invokes `torch.manual_seed`. It uses calls such as `torch.manual_seed`, `torch.randn((batch_size, m, k), devic...`, `_make_batched_blackwell_mxfp4_weight`, `_shuffle_blackwell_mxfp4_weight`, `torch.equal` to implement its workflow.

**CN:** 定义函数 `test_matmul_blackwell_shuffled_mxfp4_weight(device)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; invokes `torch.manual_seed`. 其中会调用 `torch.manual_seed`, `torch.randn((batch_size, m, k), devic...`, `_make_batched_blackwell_mxfp4_weight`, `_shuffle_blackwell_mxfp4_weight`, `torch.equal` 来实现其工作流程.

### Block 59 — Lines 138-139 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
138|     if device != "cuda" or not torch.cuda.is_available():
139|         pytest.skip("requires CUDA")
```
**EN:** Checks `device != 'cuda' or not torch.cuda.is_available()`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `device != 'cuda' or not torch.cuda.is_available()`. 真分支主要invokes `pytest.skip`.

### Block 60 — Lines 140-142 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
140|     if torch.cuda.get_device_capability()[0] < 10:
141|         pytest.skip("requires Blackwell or newer")
142| 
```
**EN:** Checks `torch.cuda.get_device_capability()[0] < 10`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `torch.cuda.get_device_capability()[0] < 10`. 真分支主要invokes `pytest.skip`.

### Block 61 — Lines 143-143 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
143|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 62 — Lines 144-144 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
144|     batch_size, m, n, k = 2, 128, 128, 128
```
**EN:** Assigns `batch_size`, `m`, `n`, `k` and builds a tuple.

**CN:** 将 `batch_size`, `m`, `n`, `k` and 构造一个元组.

### Block 63 — Lines 145-145 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
145|     a = torch.randn((batch_size, m, k), device=device, dtype=torch.bfloat16).to(torch.float8_e5m2)
```
**EN:** Assigns `a` and calls `torch.randn((batch_size, m, k), device=device, ...`.

**CN:** 将 `a`，并调用 `torch.randn((batch_size, m, k), device=device, ...`.

### Block 64 — Lines 146-146 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
146|     b, b_scale = _make_batched_blackwell_mxfp4_weight(device, batch_size, k, n)
```
**EN:** Assigns `b`, `b_scale` and calls `_make_batched_blackwell_mxfp4_weight`.

**CN:** 将 `b`, `b_scale`，并调用 `_make_batched_blackwell_mxfp4_weight`.

### Block 65 — Lines 147-150 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
147|     b_shuffled = _shuffle_blackwell_mxfp4_weight(b)
148| 
149|     # Sanity-check the host-side packing; this is the layout consumed by the
150|     # W_SHUFFLED TMA load path in _p_matmul.
```
**EN:** Assigns `b_shuffled` and calls `_shuffle_blackwell_mxfp4_weight`.

**CN:** 将 `b_shuffled`，并调用 `_shuffle_blackwell_mxfp4_weight`.

### Block 66 — Lines 151-152 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
151|     assert torch.equal(b.storage.data, convert_layout(b_shuffled, b.storage.layout).storage.data)
152| 
```
**EN:** Asserts `torch.equal(b.storage.data, convert_layout(b_shuffled, b.storage.layout).storage.data)` to enforce invariants.

**CN:** 断言 `torch.equal(b.storage.data, convert_layout(b_shuffled, b.storage.layout).storage.data)` 以确保不变量成立。

### Block 67 — Lines 153-157 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
153|     precision_config = PrecisionConfig(
154|         b_mx_scale=b_scale,
155|         b_microblock_size=MXFP_BLOCK_SIZE.value,
156|         out_dtype=torch.bfloat16,
157|     )
```
**EN:** Assigns `precision_config` and calls `PrecisionConfig`.

**CN:** 将 `precision_config`，并调用 `PrecisionConfig`.

### Block 68 — Lines 158-161 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
158|     constraints = {
159|         "is_persistent": True,
160|         "block_m": 128,
161|     }
```
**EN:** Assigns `constraints` and builds a dictionary.

**CN:** 将 `constraints` and 构造一个字典.

### Block 69 — Lines 162-164 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
162|     with scoped_opt_flags_constraints(constraints):
163|         tri_y = matmul(a, b_shuffled, None, precision_config=precision_config)
164| 
```
**EN:** Uses context manager(s) `scoped_opt_flags_constraints(constraints)` around code that mainly prepares intermediate values.

**CN:** 使用上下文管理器 `scoped_opt_flags_constraints(constraints)` ，其中代码主要准备中间值.

### Block 70 — Lines 165-165 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
165|     ref_y = matmul_torch(a.to(torch.bfloat16), b, None, precision_config=precision_config)
```
**EN:** Assigns `ref_y` and calls `matmul_torch`.

**CN:** 将 `ref_y`，并调用 `matmul_torch`.

### Block 71 — Lines 166-166 (test_matmul_blackwell_shuffled_mxfp4_weight)
```python
166|     assert_close(ref_y, tri_y, maxtol=3e-2, rmstol=None)
```
**EN:** Calls `assert_close` for side effects, registration, or validation.

**CN:** 调用 `assert_close` ，用于副作用、注册或校验。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `_make_blackwell_scale_tensor`, `_make_blackwell_mxfp4_weight`, `_make_batched_blackwell_mxfp4_weight`, `_shuffle_blackwell_mxfp4_weight`, `test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked`, `test_compute_block_n_blackwell_scale_aligns_to_128`, `test_matmul_blackwell_scale_small_n`, `test_matmul_blackwell_shuffled_mxfp4_weight`.
  **CN:** 主要符号：`_make_blackwell_scale_tensor`, `_make_blackwell_mxfp4_weight`, `_make_batched_blackwell_mxfp4_weight`, `_shuffle_blackwell_mxfp4_weight`, `test_matmul_hopper_mxfp4_rhs_scale_padding_is_masked`, `test_compute_block_n_blackwell_scale_aligns_to_128`, `test_matmul_blackwell_scale_small_n`, `test_matmul_blackwell_shuffled_mxfp4_weight`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `pytest`, `torch`, `triton._internal_testing (is_cuda)`.
  **CN:** 外部模块：`pytest`, `torch`, `triton._internal_testing (is_cuda)`。
- **EN:** Internal modules: `triton_kernels.matmul (matmul, matmul_torch, PrecisionConfig)`, `triton_kernels.matmul_details.opt_flags (InapplicableConstraint, scoped_opt_flags_constraints)`, `triton_kernels.matmul_details.opt_flags_details (opt_flags_nvidia)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (FP4, UINT8, Storage, Tensor, convert_layout, wrap_torch_tensor)`, `triton_kernels.tensor_details (layout)`, `triton_kernels.tensor_details.layout (BlackwellMX4ValueShuffledLayout)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (BlackwellMXScaleLayout)`, `triton_kernels.testing (assert_close)`.
  **CN:** 内部模块：`triton_kernels.matmul (matmul, matmul_torch, PrecisionConfig)`, `triton_kernels.matmul_details.opt_flags (InapplicableConstraint, scoped_opt_flags_constraints)`, `triton_kernels.matmul_details.opt_flags_details (opt_flags_nvidia)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (FP4, UINT8, Storage, Tensor, convert_layout, wrap_torch_tensor)`, `triton_kernels.tensor_details (layout)`, `triton_kernels.tensor_details.layout (BlackwellMX4ValueShuffledLayout)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (BlackwellMXScaleLayout)`, `triton_kernels.testing (assert_close)`。
