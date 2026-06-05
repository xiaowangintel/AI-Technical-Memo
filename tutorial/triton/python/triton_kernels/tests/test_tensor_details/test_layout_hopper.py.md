# test_layout_hopper.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_tensor_details/test_layout_hopper.py`
- **Purpose / 用途:** Pytest coverage for test layout hopper; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test layout hopper 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-17 (module)
```python
 1| import pytest
 2| from triton._internal_testing import is_cuda
 3| from triton_kernels.tensor import wrap_torch_tensor, convert_layout, FP4
 4| from triton_kernels.tensor_details.layout import HopperMXScaleLayout, HopperMXValueLayout, StridedLayout
 5| from triton_kernels.numerics_details.mxfp import downcast_to_mxfp, upcast_from_mxfp
 6| from triton_kernels.tensor_details.layout_details.hopper_value import mxfp4_to_bf16_triton
 7| from triton_kernels.tensor_details.layout_details.hopper_scale import unswizzle_mxfp4_scale_hopper
 8| from triton_kernels.target_info import cuda_capability_geq
 9| import triton.language as tl
10| import triton
11| import torch
12| 
13| # ------------------------------------------------------------
14| # Torch tests
15| # ------------------------------------------------------------
16| 
17| 
```
**EN:** This block imports `pytest`, `triton._internal_testing (is_cuda)`, `triton_kernels.tensor (wrap_torch_tensor, convert_layout, FP4)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout, HopperMXValueLayout, StridedLayout)`, `triton_kernels.numerics_details.mxfp (downcast_to_mxfp, upcast_from_mxfp)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.target_info (cuda_capability_geq)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pytest`, `triton._internal_testing (is_cuda)`, `triton_kernels.tensor (wrap_torch_tensor, convert_layout, FP4)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout, HopperMXValueLayout, StridedLayout)`, `triton_kernels.numerics_details.mxfp (downcast_to_mxfp, upcast_from_mxfp)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.target_info (cuda_capability_geq)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 18-22 (test_mxfp4_value_roundtrip)
```python
18| @pytest.mark.parametrize("shape", [(16, 32), (16, 64), (32, 32), (32, 64), (64, 128), (128, 128)])
19| @pytest.mark.parametrize("trans", [False, True])
20| @pytest.mark.parametrize("mx_axis", [0, 1])
21| @pytest.mark.parametrize("mma_version", [2, 3])
22| def test_mxfp4_value_roundtrip(shape, trans, mx_axis, mma_version):
```
**EN:** Defines function `test_mxfp4_value_roundtrip(shape, trans, mx_axis, mma_version)` with decorators `pytest.mark.parametrize('shape', [(16...`, `pytest.mark.parametrize('trans', [Fal...`, `pytest.mark.parametrize('mx_axis', [0...`, `pytest.mark.parametrize('mma_version'...` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `pytest.mark.parametrize`, `torch.randint`, `HopperMXValueLayout`, `list`, `layout.make_transformation` to implement its workflow.

**CN:** 定义函数 `test_mxfp4_value_roundtrip(shape, trans, mx_axis, mma_version)`，带有装饰器 `pytest.mark.parametrize('shape', [(16...`, `pytest.mark.parametrize('trans', [Fal...`, `pytest.mark.parametrize('mx_axis', [0...`, `pytest.mark.parametrize('mma_version'...`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `pytest.mark.parametrize`, `torch.randint`, `HopperMXValueLayout`, `list`, `layout.make_transformation` 来实现其工作流程.

### Block 3 — Lines 23-23 (test_mxfp4_value_roundtrip)
```python
23|     x = torch.randint(0, 256, shape, dtype=torch.uint8, device="cuda")
```
**EN:** Assigns `x` and calls `torch.randint`.

**CN:** 将 `x`，并调用 `torch.randint`.

### Block 4 — Lines 24-25 (test_mxfp4_value_roundtrip)
```python
24|     if trans:
25|         x = x.mT
```
**EN:** Checks `trans`. The true branch mainly prepares intermediate values.

**CN:** 检查 `trans`. 真分支主要准备中间值.

### Block 5 — Lines 26-27 (test_mxfp4_value_roundtrip)
```python
26|     if x.shape[1 - mx_axis] < 32:
27|         pytest.skip("Not enough elements along non-mx axis")
```
**EN:** Checks `x.shape[1 - mx_axis] < 32`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `x.shape[1 - mx_axis] < 32`. 真分支主要invokes `pytest.skip`.

### Block 6 — Lines 28-28 (test_mxfp4_value_roundtrip)
```python
28|     layout = HopperMXValueLayout(mx_axis - 2, mma_version)
```
**EN:** Assigns `layout` and calls `HopperMXValueLayout`.

**CN:** 将 `layout`，并调用 `HopperMXValueLayout`.

### Block 7 — Lines 29-29 (test_mxfp4_value_roundtrip)
```python
29|     shape = list(x.shape)
```
**EN:** Assigns `shape` and calls `list`.

**CN:** 将 `shape`，并调用 `list`.

### Block 8 — Lines 30-30 (test_mxfp4_value_roundtrip)
```python
30|     shape[-1] *= 2
```
**EN:** Updates `shape[-1]` with operator `Mult` using `2`.

**CN:** 更新 `shape[-1]`，使用运算符 `Mult`，并使用 `2`.

### Block 9 — Lines 31-31 (test_mxfp4_value_roundtrip)
```python
31|     transformation = layout.make_transformation(shape, is_fp4=False)
```
**EN:** Assigns `transformation` and calls `layout.make_transformation`.

**CN:** 将 `transformation`，并调用 `layout.make_transformation`.

### Block 10 — Lines 32-32 (test_mxfp4_value_roundtrip)
```python
32|     res = transformation.unswizzle_data(transformation.swizzle_data(x))
```
**EN:** Assigns `res` and calls `transformation.unswizzle_data`.

**CN:** 将 `res`，并调用 `transformation.unswizzle_data`.

### Block 11 — Lines 33-35 (test_mxfp4_value_roundtrip)
```python
33|     assert (res == x).all()
34| 
35| 
```
**EN:** Asserts `(res == x).all()` to enforce invariants.

**CN:** 断言 `(res == x).all()` 以确保不变量成立。

### Block 12 — Lines 36-39 (test_mxfp4_scale_roundtrip)
```python
36| @pytest.mark.parametrize("mx_axis", [0, 1])
37| @pytest.mark.parametrize("num_warps", [4, 8])
38| @pytest.mark.parametrize("shape", [(256, 64), (256, 128), (256, 256)])
39| def test_mxfp4_scale_roundtrip(shape, mx_axis, num_warps):
```
**EN:** Defines function `test_mxfp4_scale_roundtrip(shape, mx_axis, num_warps)` with decorators `pytest.mark.parametrize('mx_axis', [0...`, `pytest.mark.parametrize('num_warps', ...`, `pytest.mark.parametrize('shape', [(25...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.randint`, `HopperMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` to implement its workflow.

**CN:** 定义函数 `test_mxfp4_scale_roundtrip(shape, mx_axis, num_warps)`，带有装饰器 `pytest.mark.parametrize('mx_axis', [0...`, `pytest.mark.parametrize('num_warps', ...`, `pytest.mark.parametrize('shape', [(25...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.randint`, `HopperMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` 来实现其工作流程.

### Block 13 — Lines 40-40 (test_mxfp4_scale_roundtrip)
```python
40|     x = torch.randint(0, 256, shape, dtype=torch.uint8, device="cuda")
```
**EN:** Assigns `x` and calls `torch.randint`.

**CN:** 将 `x`，并调用 `torch.randint`.

### Block 14 — Lines 41-41 (test_mxfp4_scale_roundtrip)
```python
41|     layout = HopperMXScaleLayout(mx_axis=mx_axis - 2, num_warps=num_warps)
```
**EN:** Assigns `layout` and calls `HopperMXScaleLayout`.

**CN:** 将 `layout`，并调用 `HopperMXScaleLayout`.

### Block 15 — Lines 42-42 (test_mxfp4_scale_roundtrip)
```python
42|     transformation = layout.make_transformation(x.shape, is_fp4=False)
```
**EN:** Assigns `transformation` and calls `layout.make_transformation`.

**CN:** 将 `transformation`，并调用 `layout.make_transformation`.

### Block 16 — Lines 43-43 (test_mxfp4_scale_roundtrip)
```python
43|     res = transformation.unswizzle_data(transformation.swizzle_data(x))
```
**EN:** Assigns `res` and calls `transformation.unswizzle_data`.

**CN:** 将 `res`，并调用 `transformation.unswizzle_data`.

### Block 17 — Lines 44-46 (test_mxfp4_scale_roundtrip)
```python
44|     assert (res[:shape[0], :shape[1]] == x).all()
45| 
46| 
```
**EN:** Asserts `(res[:shape[0], :shape[1]] == x).all()` to enforce invariants.

**CN:** 断言 `(res[:shape[0], :shape[1]] == x).all()` 以确保不变量成立。

### Block 18 — Lines 47-57 (test_mxfp4_scale_zero_sized_roundtrip)
```python
47| @pytest.mark.parametrize(
48|     ("shape", "mx_axis"),
49|     [
50|         ((0, 64), -2),
51|         ((64, 0), -1),
52|         ((2, 0), -2),
53|         ((0, 2), -1),
54|     ],
55| )
56| @pytest.mark.parametrize("num_warps", [4, 8])
57| def test_mxfp4_scale_zero_sized_roundtrip(shape, mx_axis, num_warps):
```
**EN:** Defines function `test_mxfp4_scale_zero_sized_roundtrip(shape, mx_axis, num_warps)` with decorators `pytest.mark.parametrize(('shape', 'mx...`, `pytest.mark.parametrize('num_warps', ...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.empty`, `wrap_torch_tensor`, `HopperMXScaleLayout`, `convert_layout` to implement its workflow.

**CN:** 定义函数 `test_mxfp4_scale_zero_sized_roundtrip(shape, mx_axis, num_warps)`，带有装饰器 `pytest.mark.parametrize(('shape', 'mx...`, `pytest.mark.parametrize('num_warps', ...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.empty`, `wrap_torch_tensor`, `HopperMXScaleLayout`, `convert_layout` 来实现其工作流程.

### Block 19 — Lines 58-58 (test_mxfp4_scale_zero_sized_roundtrip)
```python
58|     x = torch.empty(shape, dtype=torch.uint8)
```
**EN:** Assigns `x` and calls `torch.empty`.

**CN:** 将 `x`，并调用 `torch.empty`.

### Block 20 — Lines 59-59 (test_mxfp4_scale_zero_sized_roundtrip)
```python
59|     src = wrap_torch_tensor(x)
```
**EN:** Assigns `src` and calls `wrap_torch_tensor`.

**CN:** 将 `src`，并调用 `wrap_torch_tensor`.

### Block 21 — Lines 60-61 (test_mxfp4_scale_zero_sized_roundtrip)
```python
60|     layout = HopperMXScaleLayout(mx_axis=mx_axis, num_warps=num_warps)
61| 
```
**EN:** Assigns `layout` and calls `HopperMXScaleLayout`.

**CN:** 将 `layout`，并调用 `HopperMXScaleLayout`.

### Block 22 — Lines 62-62 (test_mxfp4_scale_zero_sized_roundtrip)
```python
62|     swizzled = convert_layout(src, layout)
```
**EN:** Assigns `swizzled` and calls `convert_layout`.

**CN:** 将 `swizzled`，并调用 `convert_layout`.

### Block 23 — Lines 63-64 (test_mxfp4_scale_zero_sized_roundtrip)
```python
63|     roundtrip = convert_layout(swizzled, StridedLayout(mx_axis))
64| 
```
**EN:** Assigns `roundtrip` and calls `convert_layout`.

**CN:** 将 `roundtrip`，并调用 `convert_layout`.

### Block 24 — Lines 65-74 (test_mxfp4_scale_zero_sized_roundtrip)
```python
65|     assert roundtrip.storage.data.shape == x.shape
66| 
67| 
68| # ------------------------------------------------------------
69| # Triton tests
70| # ------------------------------------------------------------
71| 
72| # ------------------ upcast mxfp4 to bf16 --------------------
73| 
74| 
```
**EN:** Asserts `roundtrip.storage.data.shape == x.shape` to enforce invariants.

**CN:** 断言 `roundtrip.storage.data.shape == x.shape` 以确保不变量成立。

### Block 25 — Lines 75-79 (_upcast_mxfp4_to_bf16)
```python
75| @triton.jit
76| def _upcast_mxfp4_to_bf16(Y, X, XScale, x_stride_m, x_stride_n, x_scale_stride_m, x_scale_stride_n, y_stride_m,
77|                           y_stride_n, X_BLOCK_M: tl.constexpr, X_BLOCK_N: tl.constexpr, Y_BLOCK_M: tl.constexpr,
78|                           Y_BLOCK_N: tl.constexpr, SCALE_BLOCK_M: tl.constexpr, SCALE_BLOCK_N: tl.constexpr,
79|                           mx_axis: tl.constexpr):
```
**EN:** Defines function `_upcast_mxfp4_to_bf16(Y, X, XScale, x_stride_m, x_stride_n, x_scale_stride_m, x_scale_stride_n, y_stride_m, y_stride_n, X_BLOCK_M, X_BLOCK_N, Y_BLOCK_M, Y_BLOCK_N, SCALE_BLOCK_M, SCALE_BLOCK_N, mx_axis)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `tl.arange`, `tl.load`, `unswizzle_mxfp4_scale_hopper`, `mxfp4_to_bf16_triton`, `tl.store` to implement its workflow.

**CN:** 定义函数 `_upcast_mxfp4_to_bf16(Y, X, XScale, x_stride_m, x_stride_n, x_scale_stride_m, x_scale_stride_n, y_stride_m, y_stride_n, X_BLOCK_M, X_BLOCK_N, Y_BLOCK_M, Y_BLOCK_N, SCALE_BLOCK_M, SCALE_BLOCK_N, mx_axis)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `tl.arange`, `tl.load`, `unswizzle_mxfp4_scale_hopper`, `mxfp4_to_bf16_triton`, `tl.store` 来实现其工作流程.

### Block 26 — Lines 80-80 (_upcast_mxfp4_to_bf16)
```python
80|     offs_m_val = tl.arange(0, X_BLOCK_M)
```
**EN:** Assigns `offs_m_val` and calls `tl.arange`.

**CN:** 将 `offs_m_val`，并调用 `tl.arange`.

### Block 27 — Lines 81-81 (_upcast_mxfp4_to_bf16)
```python
81|     offs_n_val = tl.arange(0, X_BLOCK_N)
```
**EN:** Assigns `offs_n_val` and calls `tl.arange`.

**CN:** 将 `offs_n_val`，并调用 `tl.arange`.

### Block 28 — Lines 82-82 (_upcast_mxfp4_to_bf16)
```python
82|     offs_m_scale = tl.arange(0, SCALE_BLOCK_M)
```
**EN:** Assigns `offs_m_scale` and calls `tl.arange`.

**CN:** 将 `offs_m_scale`，并调用 `tl.arange`.

### Block 29 — Lines 83-84 (_upcast_mxfp4_to_bf16)
```python
83|     offs_n_scale = tl.arange(0, SCALE_BLOCK_N)
84|     # load values
```
**EN:** Assigns `offs_n_scale` and calls `tl.arange`.

**CN:** 将 `offs_n_scale`，并调用 `tl.arange`.

### Block 30 — Lines 85-85 (_upcast_mxfp4_to_bf16)
```python
85|     offs_x = offs_m_val[:, None] * x_stride_m + offs_n_val[None, :] * x_stride_n
```
**EN:** Assigns `offs_x` and evaluates `offs_m_val[:, None] * x_stride_m + offs_n_val[None, :] * x_stride_n`.

**CN:** 将 `offs_x` and 计算 `offs_m_val[:, None] * x_stride_m + offs_n_val[None, :] * x_stride_n`.

### Block 31 — Lines 86-87 (_upcast_mxfp4_to_bf16)
```python
86|     x = tl.load(X + offs_x)
87|     # load scales
```
**EN:** Assigns `x` and calls `tl.load`.

**CN:** 将 `x`，并调用 `tl.load`.

### Block 32 — Lines 88-88 (_upcast_mxfp4_to_bf16)
```python
88|     offs_x_scale = offs_m_scale[:, None] * x_scale_stride_m + offs_n_scale[None, :] * x_scale_stride_n
```
**EN:** Assigns `offs_x_scale` and evaluates `offs_m_scale[:, None] * x_scale_stride_m + offs_n_scale[None, :] * ...`.

**CN:** 将 `offs_x_scale` and 计算 `offs_m_scale[:, None] * x_scale_stride_m + offs_n_scale[None, :] * ...`.

### Block 33 — Lines 89-89 (_upcast_mxfp4_to_bf16)
```python
89|     x_scale = tl.load(XScale + offs_x_scale)
```
**EN:** Assigns `x_scale` and calls `tl.load`.

**CN:** 将 `x_scale`，并调用 `tl.load`.

### Block 34 — Lines 90-90 (_upcast_mxfp4_to_bf16)
```python
90|     x_scale = unswizzle_mxfp4_scale_hopper(x_scale, mx_axis=mx_axis, num_warps=tl.extra.cuda.num_warps())
```
**EN:** Assigns `x_scale` and calls `unswizzle_mxfp4_scale_hopper`.

**CN:** 将 `x_scale`，并调用 `unswizzle_mxfp4_scale_hopper`.

### Block 35 — Lines 91-92 (_upcast_mxfp4_to_bf16)
```python
91|     y = mxfp4_to_bf16_triton(x, x_scale, mx_axis=mx_axis)
92|     # write back output
```
**EN:** Assigns `y` and calls `mxfp4_to_bf16_triton`.

**CN:** 将 `y`，并调用 `mxfp4_to_bf16_triton`.

### Block 36 — Lines 93-93 (_upcast_mxfp4_to_bf16)
```python
93|     offs_m_val = tl.arange(0, Y_BLOCK_M)
```
**EN:** Assigns `offs_m_val` and calls `tl.arange`.

**CN:** 将 `offs_m_val`，并调用 `tl.arange`.

### Block 37 — Lines 94-94 (_upcast_mxfp4_to_bf16)
```python
94|     offs_n_val = tl.arange(0, Y_BLOCK_N)
```
**EN:** Assigns `offs_n_val` and calls `tl.arange`.

**CN:** 将 `offs_n_val`，并调用 `tl.arange`.

### Block 38 — Lines 95-95 (_upcast_mxfp4_to_bf16)
```python
95|     offs_y = offs_m_val[:, None] * y_stride_m + offs_n_val[None, :] * y_stride_n
```
**EN:** Assigns `offs_y` and evaluates `offs_m_val[:, None] * y_stride_m + offs_n_val[None, :] * y_stride_n`.

**CN:** 将 `offs_y` and 计算 `offs_m_val[:, None] * y_stride_m + offs_n_val[None, :] * y_stride_n`.

### Block 39 — Lines 96-98 (_upcast_mxfp4_to_bf16)
```python
96|     tl.store(Y + offs_y, y)
97| 
98| 
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 40 — Lines 99-103 (test_upcast_mxfp4_to_bf16)
```python
 99| @pytest.mark.skipif(not is_cuda(), reason="Only supported on cuda")
100| @pytest.mark.skipif(not cuda_capability_geq(9), reason="Only supported for capability >= 9")
101| @pytest.mark.parametrize("num_warps", [4, 8])
102| @pytest.mark.parametrize("mx_axis", [0, 1])
103| def test_upcast_mxfp4_to_bf16(num_warps, mx_axis):
```
**EN:** Defines function `test_upcast_mxfp4_to_bf16(num_warps, mx_axis)` with decorators `pytest.mark.skipif(not is_cuda(), rea...`, `pytest.mark.skipif(not cuda_capabilit...`, `pytest.mark.parametrize('num_warps', ...`, `pytest.mark.parametrize('mx_axis', [0...` for this module. The body mainly invokes `torch.manual_seed`; invokes `torch.cuda.manual_seed`; prepares intermediate values. It uses calls such as `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `torch.cuda.manual_seed`, `torch.randn` to implement its workflow.

**CN:** 定义函数 `test_upcast_mxfp4_to_bf16(num_warps, mx_axis)`，带有装饰器 `pytest.mark.skipif(not is_cuda(), rea...`, `pytest.mark.skipif(not cuda_capabilit...`, `pytest.mark.parametrize('num_warps', ...`, `pytest.mark.parametrize('mx_axis', [0...`，供本模块使用. 主体主要invokes `torch.manual_seed`; invokes `torch.cuda.manual_seed`; 准备中间值. 其中会调用 `pytest.mark.skipif`, `pytest.mark.parametrize`, `torch.manual_seed`, `torch.cuda.manual_seed`, `torch.randn` 来实现其工作流程.

### Block 41 — Lines 104-104 (test_upcast_mxfp4_to_bf16)
```python
104|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 42 — Lines 105-105 (test_upcast_mxfp4_to_bf16)
```python
105|     torch.cuda.manual_seed(0)
```
**EN:** Calls `torch.cuda.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.manual_seed` ，用于副作用、注册或校验。

### Block 43 — Lines 106-106 (test_upcast_mxfp4_to_bf16)
```python
106|     shape = [64, 64]
```
**EN:** Assigns `shape` and builds a list.

**CN:** 将 `shape` and 构造一个列表.

### Block 44 — Lines 107-107 (test_upcast_mxfp4_to_bf16)
```python
107|     shape[1 - mx_axis] = 32 * num_warps
```
**EN:** Assigns `shape[1 - mx_axis]` and evaluates `32 * num_warps`.

**CN:** 将 `shape[1 - mx_axis]` and 计算 `32 * num_warps`.

### Block 45 — Lines 108-108 (test_upcast_mxfp4_to_bf16)
```python
108|     x = torch.randn(shape, dtype=torch.bfloat16, device="cuda")
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 46 — Lines 109-109 (test_upcast_mxfp4_to_bf16)
```python
109|     x_fp4_val, x_fp4_scale = downcast_to_mxfp(x, torch.uint8, axis=mx_axis)
```
**EN:** Assigns `x_fp4_val`, `x_fp4_scale` and calls `downcast_to_mxfp`.

**CN:** 将 `x_fp4_val`, `x_fp4_scale`，并调用 `downcast_to_mxfp`.

### Block 47 — Lines 110-110 (test_upcast_mxfp4_to_bf16)
```python
110|     x_bf16 = upcast_from_mxfp(x_fp4_val, x_fp4_scale, x.dtype, axis=mx_axis)
```
**EN:** Assigns `x_bf16` and calls `upcast_from_mxfp`.

**CN:** 将 `x_bf16`，并调用 `upcast_from_mxfp`.

### Block 48 — Lines 111-111 (test_upcast_mxfp4_to_bf16)
```python
111|     x_fp4_val = wrap_torch_tensor(x_fp4_val, dtype=FP4)
```
**EN:** Assigns `x_fp4_val` and calls `wrap_torch_tensor`.

**CN:** 将 `x_fp4_val`，并调用 `wrap_torch_tensor`.

### Block 49 — Lines 112-112 (test_upcast_mxfp4_to_bf16)
```python
112|     x_fp4_scale = wrap_torch_tensor(x_fp4_scale)
```
**EN:** Assigns `x_fp4_scale` and calls `wrap_torch_tensor`.

**CN:** 将 `x_fp4_scale`，并调用 `wrap_torch_tensor`.

### Block 50 — Lines 113-113 (test_upcast_mxfp4_to_bf16)
```python
113|     x_fp4_val = convert_layout(x_fp4_val, HopperMXValueLayout(mx_axis=mx_axis - 2, mma_version=3))
```
**EN:** Assigns `x_fp4_val` and calls `convert_layout`.

**CN:** 将 `x_fp4_val`，并调用 `convert_layout`.

### Block 51 — Lines 114-114 (test_upcast_mxfp4_to_bf16)
```python
114|     x_fp4_scale = convert_layout(x_fp4_scale, HopperMXScaleLayout(mx_axis=mx_axis - 2, num_warps=num_warps))
```
**EN:** Assigns `x_fp4_scale` and calls `convert_layout`.

**CN:** 将 `x_fp4_scale`，并调用 `convert_layout`.

### Block 52 — Lines 115-115 (test_upcast_mxfp4_to_bf16)
```python
115|     y = torch.empty_like(x_bf16)
```
**EN:** Assigns `y` and calls `torch.empty_like`.

**CN:** 将 `y`，并调用 `torch.empty_like`.

### Block 53 — Lines 116-116 (test_upcast_mxfp4_to_bf16)
```python
116|     scale_block = [s // 32 if i == mx_axis else s for i, s in enumerate(shape)]
```
**EN:** Assigns `scale_block` and evaluates `[s // 32 if i == mx_axis else s for i, s in enumerate(shape)]`.

**CN:** 将 `scale_block` and 计算 `[s // 32 if i == mx_axis else s for i, s in enumerate(shape)]`.

### Block 54 — Lines 117-117 (test_upcast_mxfp4_to_bf16)
```python
117|     scale_block = x_fp4_scale.storage.layout.swizzle_block_shape(scale_block)
```
**EN:** Assigns `scale_block` and calls `x_fp4_scale.storage.layout.swizzle_block_shape`.

**CN:** 将 `scale_block`，并调用 `x_fp4_scale.storage.layout.swizzle_block_shape`.

### Block 55 — Lines 118-118 (test_upcast_mxfp4_to_bf16)
```python
118|     value_block = [s // 2 if i == mx_axis else s for i, s in enumerate(shape)]
```
**EN:** Assigns `value_block` and evaluates `[s // 2 if i == mx_axis else s for i, s in enumerate(shape)]`.

**CN:** 将 `value_block` and 计算 `[s // 2 if i == mx_axis else s for i, s in enumerate(shape)]`.

### Block 56 — Lines 119-119 (test_upcast_mxfp4_to_bf16)
```python
119|     value_block = x_fp4_val.storage.layout.swizzle_block_shape(value_block)
```
**EN:** Assigns `value_block` and calls `x_fp4_val.storage.layout.swizzle_block_shape`.

**CN:** 将 `value_block`，并调用 `x_fp4_val.storage.layout.swizzle_block_shape`.

### Block 57 — Lines 120-126 (test_upcast_mxfp4_to_bf16)
```python
120|     _upcast_mxfp4_to_bf16[(1, )](
121|         y, x_fp4_val.storage.data, x_fp4_scale.storage.data,  #
122|         x_fp4_val.storage.data.stride(0), x_fp4_val.storage.data.stride(1),  #
123|         x_fp4_scale.storage.data.stride(0), x_fp4_scale.storage.data.stride(1),  #
124|         y.stride(0), y.stride(1),  #
125|         *value_block, *shape,  #
126|         *scale_block, mx_axis=mx_axis, num_warps=num_warps)
```
**EN:** Calls `_upcast_mxfp4_to_bf16[1,]` for side effects, registration, or validation.

**CN:** 调用 `_upcast_mxfp4_to_bf16[1,]` ，用于副作用、注册或校验。

### Block 58 — Lines 127-127 (test_upcast_mxfp4_to_bf16)
```python
127|     assert (y == x_bf16).all()
```
**EN:** Asserts `(y == x_bf16).all()` to enforce invariants.

**CN:** 断言 `(y == x_bf16).all()` 以确保不变量成立。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `test_mxfp4_value_roundtrip`, `test_mxfp4_scale_roundtrip`, `test_mxfp4_scale_zero_sized_roundtrip`, `_upcast_mxfp4_to_bf16`, `test_upcast_mxfp4_to_bf16`.
  **CN:** 主要符号：`test_mxfp4_value_roundtrip`, `test_mxfp4_scale_roundtrip`, `test_mxfp4_scale_zero_sized_roundtrip`, `_upcast_mxfp4_to_bf16`, `test_upcast_mxfp4_to_bf16`。
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
- **EN:** External modules: `pytest`, `triton._internal_testing (is_cuda)`, `triton.language`, `triton`, `torch`.
  **CN:** 外部模块：`pytest`, `triton._internal_testing (is_cuda)`, `triton.language`, `triton`, `torch`。
- **EN:** Internal modules: `triton_kernels.tensor (wrap_torch_tensor, convert_layout, FP4)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout, HopperMXValueLayout, StridedLayout)`, `triton_kernels.numerics_details.mxfp (downcast_to_mxfp, upcast_from_mxfp)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.target_info (cuda_capability_geq)`.
  **CN:** 内部模块：`triton_kernels.tensor (wrap_torch_tensor, convert_layout, FP4)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout, HopperMXValueLayout, StridedLayout)`, `triton_kernels.numerics_details.mxfp (downcast_to_mxfp, upcast_from_mxfp)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.target_info (cuda_capability_geq)`。
