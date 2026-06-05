# test_layout_blackwell.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_tensor_details/test_layout_blackwell.py`
- **Purpose / 用途:** Pytest coverage for test layout blackwell; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test layout blackwell 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-10 (module)
```python
 1| import pytest
 2| import torch
 3| from triton_kernels.tensor_details.layout import BlackwellMXScaleLayout, BlackwellActMXScaleLayout, StridedLayout
 4| from triton_kernels.tensor import make_ragged_tensor_metadata, wrap_torch_tensor, convert_layout
 5| 
 6| # ------------------------------------------------------------
 7| # Torch tests
 8| # ------------------------------------------------------------
 9| 
10| 
```
**EN:** This block imports `pytest`, `torch`, `triton_kernels.tensor_details.layout (BlackwellMXScaleLayout, BlackwellActMXScaleLayout, StridedLayout)`, `triton_kernels.tensor (make_ragged_tensor_metadata, wrap_torch_tensor, convert_layout)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pytest`, `torch`, `triton_kernels.tensor_details.layout (BlackwellMXScaleLayout, BlackwellActMXScaleLayout, StridedLayout)`, `triton_kernels.tensor (make_ragged_tensor_metadata, wrap_torch_tensor, convert_layout)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 11-21 (test_mxfp4_scale_roundtrip)
```python
11| @pytest.mark.parametrize(
12|     "shape",
13|     [
14|         (3, 4096, 1024),
15|         (10, 254, 60),
16|         (1, 320, 160),
17|         (2, 16, 512),
18|         (3, 2, 36),
19|     ],
20| )
21| def test_mxfp4_scale_roundtrip(shape):
```
**EN:** Defines function `test_mxfp4_scale_roundtrip(shape)` with decorators `pytest.mark.parametrize('shape', [(3,...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.randint`, `BlackwellMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` to implement its workflow.

**CN:** 定义函数 `test_mxfp4_scale_roundtrip(shape)`，带有装饰器 `pytest.mark.parametrize('shape', [(3,...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.randint`, `BlackwellMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` 来实现其工作流程.

### Block 3 — Lines 22-22 (test_mxfp4_scale_roundtrip)
```python
22|     x = torch.randint(0, 256, shape, dtype=torch.uint8, device="cuda")
```
**EN:** Assigns `x` and calls `torch.randint`.

**CN:** 将 `x`，并调用 `torch.randint`.

### Block 4 — Lines 23-23 (test_mxfp4_scale_roundtrip)
```python
23|     layout = BlackwellMXScaleLayout()
```
**EN:** Assigns `layout` and calls `BlackwellMXScaleLayout`.

**CN:** 将 `layout`，并调用 `BlackwellMXScaleLayout`.

### Block 5 — Lines 24-24 (test_mxfp4_scale_roundtrip)
```python
24|     transformation = layout.make_transformation(x.shape, is_fp4=False)
```
**EN:** Assigns `transformation` and calls `layout.make_transformation`.

**CN:** 将 `transformation`，并调用 `layout.make_transformation`.

### Block 6 — Lines 25-25 (test_mxfp4_scale_roundtrip)
```python
25|     res = transformation.unswizzle_data(transformation.swizzle_data(x))
```
**EN:** Assigns `res` and calls `transformation.unswizzle_data`.

**CN:** 将 `res`，并调用 `transformation.unswizzle_data`.

### Block 7 — Lines 26-28 (test_mxfp4_scale_roundtrip)
```python
26|     assert (res == x).all()
27| 
28| 
```
**EN:** Asserts `(res == x).all()` to enforce invariants.

**CN:** 断言 `(res == x).all()` 以确保不变量成立。

### Block 8 — Lines 29-30 (test_act_scale_roundtrip_batched)
```python
29| @pytest.mark.parametrize("shape", [(2, 256, 192), (1, 128, 64)])
30| def test_act_scale_roundtrip_batched(shape):
```
**EN:** Defines function `test_act_scale_roundtrip_batched(shape)` with decorators `pytest.mark.parametrize('shape', [(2,...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.randn`, `BlackwellActMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` to implement its workflow.

**CN:** 定义函数 `test_act_scale_roundtrip_batched(shape)`，带有装饰器 `pytest.mark.parametrize('shape', [(2,...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.randn`, `BlackwellActMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` 来实现其工作流程.

### Block 9 — Lines 31-31 (test_act_scale_roundtrip_batched)
```python
31|     x = torch.randn(shape, device="cuda", dtype=torch.float32)
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 10 — Lines 32-32 (test_act_scale_roundtrip_batched)
```python
32|     layout = BlackwellActMXScaleLayout(ragged_metadata=None)
```
**EN:** Assigns `layout` and calls `BlackwellActMXScaleLayout`.

**CN:** 将 `layout`，并调用 `BlackwellActMXScaleLayout`.

### Block 11 — Lines 33-33 (test_act_scale_roundtrip_batched)
```python
33|     transformation = layout.make_transformation(x.shape, is_fp4=False)
```
**EN:** Assigns `transformation` and calls `layout.make_transformation`.

**CN:** 将 `transformation`，并调用 `layout.make_transformation`.

### Block 12 — Lines 34-34 (test_act_scale_roundtrip_batched)
```python
34|     res = transformation.unswizzle_data(transformation.swizzle_data(x))
```
**EN:** Assigns `res` and calls `transformation.unswizzle_data`.

**CN:** 将 `res`，并调用 `transformation.unswizzle_data`.

### Block 13 — Lines 35-37 (test_act_scale_roundtrip_batched)
```python
35|     torch.testing.assert_close(res, x)
36| 
37| 
```
**EN:** Calls `torch.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `torch.testing.assert_close` ，用于副作用、注册或校验。

### Block 14 — Lines 38-39 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
38| @pytest.mark.parametrize("shape", [(256, 192), (128, 64), (130, 65)])
39| def test_act_scale_roundtrip_2d_without_ragged_metadata(shape):
```
**EN:** Defines function `test_act_scale_roundtrip_2d_without_ragged_metadata(shape)` with decorators `pytest.mark.parametrize('shape', [(25...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.randn`, `BlackwellActMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` to implement its workflow.

**CN:** 定义函数 `test_act_scale_roundtrip_2d_without_ragged_metadata(shape)`，带有装饰器 `pytest.mark.parametrize('shape', [(25...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.randn`, `BlackwellActMXScaleLayout`, `layout.make_transformation`, `transformation.unswizzle_data` 来实现其工作流程.

### Block 15 — Lines 40-40 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
40|     x = torch.randn(shape, device="cuda", dtype=torch.float32)
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 16 — Lines 41-41 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
41|     layout = BlackwellActMXScaleLayout(ragged_metadata=None)
```
**EN:** Assigns `layout` and calls `BlackwellActMXScaleLayout`.

**CN:** 将 `layout`，并调用 `BlackwellActMXScaleLayout`.

### Block 17 — Lines 42-42 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
42|     transformation = layout.make_transformation(x.shape, is_fp4=False)
```
**EN:** Assigns `transformation` and calls `layout.make_transformation`.

**CN:** 将 `transformation`，并调用 `layout.make_transformation`.

### Block 18 — Lines 43-43 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
43|     assert transformation.mode == "batched"
```
**EN:** Asserts `transformation.mode == 'batched'` to enforce invariants.

**CN:** 断言 `transformation.mode == 'batched'` 以确保不变量成立。

### Block 19 — Lines 44-44 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
44|     res = transformation.unswizzle_data(transformation.swizzle_data(x))
```
**EN:** Assigns `res` and calls `transformation.unswizzle_data`.

**CN:** 将 `res`，并调用 `transformation.unswizzle_data`.

### Block 20 — Lines 45-45 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
45|     assert res.shape == shape
```
**EN:** Asserts `res.shape == shape` to enforce invariants.

**CN:** 断言 `res.shape == shape` 以确保不变量成立。

### Block 21 — Lines 46-48 (test_act_scale_roundtrip_2d_without_ragged_metadata)
```python
46|     assert torch.equal(res, x)
47| 
48| 
```
**EN:** Asserts `torch.equal(res, x)` to enforce invariants.

**CN:** 断言 `torch.equal(res, x)` 以确保不变量成立。

### Block 22 — Lines 49-50 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
49| @pytest.mark.parametrize("shape", [(256, 192), (128, 64), (130, 65)])
50| def test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata(shape):
```
**EN:** Defines function `test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata(shape)` with decorators `pytest.mark.parametrize('shape', [(25...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.randn`, `wrap_torch_tensor`, `BlackwellActMXScaleLayout`, `convert_layout` to implement its workflow.

**CN:** 定义函数 `test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata(shape)`，带有装饰器 `pytest.mark.parametrize('shape', [(25...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.randn`, `wrap_torch_tensor`, `BlackwellActMXScaleLayout`, `convert_layout` 来实现其工作流程.

### Block 23 — Lines 51-51 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
51|     x = torch.randn(shape, device="cuda", dtype=torch.float32)
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 24 — Lines 52-52 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
52|     x_tri = wrap_torch_tensor(x)
```
**EN:** Assigns `x_tri` and calls `wrap_torch_tensor`.

**CN:** 将 `x_tri`，并调用 `wrap_torch_tensor`.

### Block 25 — Lines 53-53 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
53|     scale_layout = BlackwellActMXScaleLayout(ragged_metadata=None)
```
**EN:** Assigns `scale_layout` and calls `BlackwellActMXScaleLayout`.

**CN:** 将 `scale_layout`，并调用 `BlackwellActMXScaleLayout`.

### Block 26 — Lines 54-54 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
54|     x_tri_scale = convert_layout(x_tri, scale_layout)
```
**EN:** Assigns `x_tri_scale` and calls `convert_layout`.

**CN:** 将 `x_tri_scale`，并调用 `convert_layout`.

### Block 27 — Lines 55-55 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
55|     x_tri_roundtrip = convert_layout(x_tri_scale, StridedLayout(-1))
```
**EN:** Assigns `x_tri_roundtrip` and calls `convert_layout`.

**CN:** 将 `x_tri_roundtrip`，并调用 `convert_layout`.

### Block 28 — Lines 56-58 (test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata)
```python
56|     assert torch.equal(x_tri_roundtrip.data, x)
57| 
58| 
```
**EN:** Asserts `torch.equal(x_tri_roundtrip.data, x)` to enforce invariants.

**CN:** 断言 `torch.equal(x_tri_roundtrip.data, x)` 以确保不变量成立。

### Block 29 — Lines 59-66 (test_act_scale_roundtrip_ragged)
```python
59| @pytest.mark.parametrize(
60|     "slice_sizes, m, k, align_m",
61|     [
62|         ([17, 0, 33, 5], 100, 94, 8),
63|         ([1, 2, 3, 4, 5], 50, 15, 16),
64|     ],
65| )
66| def test_act_scale_roundtrip_ragged(slice_sizes, m, k, align_m):
```
**EN:** Defines function `test_act_scale_roundtrip_ragged(slice_sizes, m, k, align_m)` with decorators `pytest.mark.parametrize('slice_sizes,...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.tensor`, `max`, `make_ragged_tensor_metadata`, `torch.randn` to implement its workflow.

**CN:** 定义函数 `test_act_scale_roundtrip_ragged(slice_sizes, m, k, align_m)`，带有装饰器 `pytest.mark.parametrize('slice_sizes,...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.tensor`, `max`, `make_ragged_tensor_metadata`, `torch.randn` 来实现其工作流程.

### Block 30 — Lines 67-67 (test_act_scale_roundtrip_ragged)
```python
67|     slice_sizes = torch.tensor(slice_sizes, device="cuda", dtype=torch.int32)
```
**EN:** Assigns `slice_sizes` and calls `torch.tensor`.

**CN:** 将 `slice_sizes`，并调用 `torch.tensor`.

### Block 31 — Lines 68-68 (test_act_scale_roundtrip_ragged)
```python
68|     m = max(m, slice_sizes.sum().item())  # there can be padded tokens in the input
```
**EN:** Assigns `m` and calls `max`.

**CN:** 将 `m`，并调用 `max`.

### Block 32 — Lines 69-69 (test_act_scale_roundtrip_ragged)
```python
69|     ragged_metadata = make_ragged_tensor_metadata(slice_sizes, m)
```
**EN:** Assigns `ragged_metadata` and calls `make_ragged_tensor_metadata`.

**CN:** 将 `ragged_metadata`，并调用 `make_ragged_tensor_metadata`.

### Block 33 — Lines 70-70 (test_act_scale_roundtrip_ragged)
```python
70|     x = torch.randn((m, k), device="cuda", dtype=torch.float32)
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 34 — Lines 71-71 (test_act_scale_roundtrip_ragged)
```python
71|     layout = BlackwellActMXScaleLayout(ragged_metadata=ragged_metadata)
```
**EN:** Assigns `layout` and calls `BlackwellActMXScaleLayout`.

**CN:** 将 `layout`，并调用 `BlackwellActMXScaleLayout`.

### Block 35 — Lines 72-72 (test_act_scale_roundtrip_ragged)
```python
72|     transformation = layout.make_transformation(x.shape, is_fp4=False)
```
**EN:** Assigns `transformation` and calls `layout.make_transformation`.

**CN:** 将 `transformation`，并调用 `layout.make_transformation`.

### Block 36 — Lines 73-74 (test_act_scale_roundtrip_ragged)
```python
73|     res = transformation.unswizzle_data(transformation.swizzle_data(x))
74| 
```
**EN:** Assigns `res` and calls `transformation.unswizzle_data`.

**CN:** 将 `res`，并调用 `transformation.unswizzle_data`.

### Block 37 — Lines 75-75 (test_act_scale_roundtrip_ragged)
```python
75|     x_useful_rows = x[ragged_metadata.slice_offs[:-1], :]
```
**EN:** Assigns `x_useful_rows` and evaluates `x[ragged_metadata.slice_offs[:-1], :]`.

**CN:** 将 `x_useful_rows` and 计算 `x[ragged_metadata.slice_offs[:-1], :]`.

### Block 38 — Lines 76-76 (test_act_scale_roundtrip_ragged)
```python
76|     res_useful_rows = res[ragged_metadata.slice_offs[:-1], :]
```
**EN:** Assigns `res_useful_rows` and evaluates `res[ragged_metadata.slice_offs[:-1], :]`.

**CN:** 将 `res_useful_rows` and 计算 `res[ragged_metadata.slice_offs[:-1], :]`.

### Block 39 — Lines 77-77 (test_act_scale_roundtrip_ragged)
```python
77|     torch.testing.assert_close(res_useful_rows, x_useful_rows)
```
**EN:** Calls `torch.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `torch.testing.assert_close` ，用于副作用、注册或校验。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `test_mxfp4_scale_roundtrip`, `test_act_scale_roundtrip_batched`, `test_act_scale_roundtrip_2d_without_ragged_metadata`, `test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata`, `test_act_scale_roundtrip_ragged`.
  **CN:** 主要符号：`test_mxfp4_scale_roundtrip`, `test_act_scale_roundtrip_batched`, `test_act_scale_roundtrip_2d_without_ragged_metadata`, `test_act_scale_convert_layout_roundtrip_2d_without_ragged_metadata`, `test_act_scale_roundtrip_ragged`。
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
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `pytest`, `torch`.
  **CN:** 外部模块：`pytest`, `torch`。
- **EN:** Internal modules: `triton_kernels.tensor_details.layout (BlackwellMXScaleLayout, BlackwellActMXScaleLayout, StridedLayout)`, `triton_kernels.tensor (make_ragged_tensor_metadata, wrap_torch_tensor, convert_layout)`.
  **CN:** 内部模块：`triton_kernels.tensor_details.layout (BlackwellMXScaleLayout, BlackwellActMXScaleLayout, StridedLayout)`, `triton_kernels.tensor (make_ragged_tensor_metadata, wrap_torch_tensor, convert_layout)`。
