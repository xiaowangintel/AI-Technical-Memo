# test_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_tensor.py`
- **Purpose / 用途:** Pytest coverage for test tensor; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test tensor 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-15 (module)
```python
 1| import pytest
 2| import torch
 3| from triton_kernels.tensor_details.dtype import BIT
 4| from triton_kernels.tensor import (
 5|     make_ragged_tensor_metadata,
 6|     make_ragged_tensor_metadata_torch,
 7|     remap_ragged_tensor_metadata,
 8|     remap_ragged_tensor_metadata_torch,
 9|     make_bitmatrix_metadata,
10|     make_bitmatrix_metadata_torch,
11|     wrap_torch_tensor,
12| )
13| from triton_kernels.testing import assert_equal
14| 
15| 
```
**EN:** This block imports `pytest`, `torch`, `triton_kernels.tensor_details.dtype (BIT)`, `triton_kernels.tensor (make_ragged_tensor_metadata, make_ragged_tensor_metadata_torch, remap_ragged_tensor_metadata, remap_ragged_tensor_metadata_torch)`, `triton_kernels.testing (assert_equal)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pytest`, `torch`, `triton_kernels.tensor_details.dtype (BIT)`, `triton_kernels.tensor (make_ragged_tensor_metadata, make_ragged_tensor_metadata_torch, remap_ragged_tensor_metadata, remap_ragged_tensor_metadata_torch)`, `triton_kernels.testing (assert_equal)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 16-17 (test_make_ragged_tensor_metadata)
```python
16| @pytest.mark.parametrize("n_slices", [1, 7, 33, 911, 1025])
17| def test_make_ragged_tensor_metadata(n_slices):
```
**EN:** Defines function `test_make_ragged_tensor_metadata(n_slices)` with decorators `pytest.mark.parametrize('n_slices', [...` for this module. The body mainly invokes `torch.manual_seed`; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randint`, `make_ragged_tensor_metadata`, `make_ragged_tensor_metadata_torch` to implement its workflow.

**CN:** 定义函数 `test_make_ragged_tensor_metadata(n_slices)`，带有装饰器 `pytest.mark.parametrize('n_slices', [...`，供本模块使用. 主体主要invokes `torch.manual_seed`; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randint`, `make_ragged_tensor_metadata`, `make_ragged_tensor_metadata_torch` 来实现其工作流程.

### Block 3 — Lines 18-18 (test_make_ragged_tensor_metadata)
```python
18|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 4 — Lines 19-19 (test_make_ragged_tensor_metadata)
```python
19|     device = "cuda"
```
**EN:** Assigns `device` and stores a string literal or docstring.

**CN:** 将 `device` and 保存字符串字面量或文档字符串.

### Block 5 — Lines 20-20 (test_make_ragged_tensor_metadata)
```python
20|     max_slice_size = 200
```
**EN:** Assigns `max_slice_size` and stores constant `200`.

**CN:** 将 `max_slice_size` and 保存常量 `200`.

### Block 6 — Lines 21-21 (test_make_ragged_tensor_metadata)
```python
21|     n_total_rows = max_slice_size * n_slices
```
**EN:** Assigns `n_total_rows` and evaluates `max_slice_size * n_slices`.

**CN:** 将 `n_total_rows` and 计算 `max_slice_size * n_slices`.

### Block 7 — Lines 22-22 (test_make_ragged_tensor_metadata)
```python
22|     slice_sizes = torch.randint(0, max_slice_size, (n_slices, ), dtype=torch.int32, device=device)
```
**EN:** Assigns `slice_sizes` and calls `torch.randint`.

**CN:** 将 `slice_sizes`，并调用 `torch.randint`.

### Block 8 — Lines 23-23 (test_make_ragged_tensor_metadata)
```python
23|     slice_sizes[torch.randint(0, n_slices, (1, ))] = 0
```
**EN:** Assigns `slice_sizes[torch.randint(0, n_slices...` and stores constant `0`.

**CN:** 将 `slice_sizes[torch.randint(0, n_slices...` and 保存常量 `0`.

### Block 9 — Lines 24-24 (test_make_ragged_tensor_metadata)
```python
24|     meta = make_ragged_tensor_metadata(slice_sizes, n_total_rows)
```
**EN:** Assigns `meta` and calls `make_ragged_tensor_metadata`.

**CN:** 将 `meta`，并调用 `make_ragged_tensor_metadata`.

### Block 10 — Lines 25-25 (test_make_ragged_tensor_metadata)
```python
25|     ref = make_ragged_tensor_metadata_torch(slice_sizes, n_total_rows)
```
**EN:** Assigns `ref` and calls `make_ragged_tensor_metadata_torch`.

**CN:** 将 `ref`，并调用 `make_ragged_tensor_metadata_torch`.

### Block 11 — Lines 26-26 (test_make_ragged_tensor_metadata)
```python
26|     assert_equal(meta.slice_sizes, ref.slice_sizes)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 12 — Lines 27-27 (test_make_ragged_tensor_metadata)
```python
27|     assert_equal(meta.slice_offs, ref.slice_offs)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 13 — Lines 28-28 (test_make_ragged_tensor_metadata)
```python
28|     assert_equal(meta.block_offs_data, ref.block_offs_data)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 14 — Lines 29-31 (test_make_ragged_tensor_metadata)
```python
29|     assert_equal(meta.block_schedule_data, ref.block_schedule_data)
30| 
31| 
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 15 — Lines 32-33 (test_remap_ragged_tensor_metadata)
```python
32| @pytest.mark.parametrize("n_slices", [9, 32, 911, 1025])
33| def test_remap_ragged_tensor_metadata(n_slices):
```
**EN:** Defines function `test_remap_ragged_tensor_metadata(n_slices)` with decorators `pytest.mark.parametrize('n_slices', [...` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.randint`, `torch.randperm`, `make_ragged_tensor_metadata`, `make_ragged_tensor_metadata_torch` to implement its workflow.

**CN:** 定义函数 `test_remap_ragged_tensor_metadata(n_slices)`，带有装饰器 `pytest.mark.parametrize('n_slices', [...`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.randint`, `torch.randperm`, `make_ragged_tensor_metadata`, `make_ragged_tensor_metadata_torch` 来实现其工作流程.

### Block 16 — Lines 34-34 (test_remap_ragged_tensor_metadata)
```python
34|     device = "cuda"
```
**EN:** Assigns `device` and stores a string literal or docstring.

**CN:** 将 `device` and 保存字符串字面量或文档字符串.

### Block 17 — Lines 35-35 (test_remap_ragged_tensor_metadata)
```python
35|     max_slice_size = 200
```
**EN:** Assigns `max_slice_size` and stores constant `200`.

**CN:** 将 `max_slice_size` and 保存常量 `200`.

### Block 18 — Lines 36-36 (test_remap_ragged_tensor_metadata)
```python
36|     n_total_rows = max_slice_size * n_slices
```
**EN:** Assigns `n_total_rows` and evaluates `max_slice_size * n_slices`.

**CN:** 将 `n_total_rows` and 计算 `max_slice_size * n_slices`.

### Block 19 — Lines 37-37 (test_remap_ragged_tensor_metadata)
```python
37|     slice_sizes = torch.randint(0, max_slice_size, (n_slices, ), dtype=torch.int32, device=device)
```
**EN:** Assigns `slice_sizes` and calls `torch.randint`.

**CN:** 将 `slice_sizes`，并调用 `torch.randint`.

### Block 20 — Lines 38-39 (test_remap_ragged_tensor_metadata)
```python
38|     slice_sizes[torch.randint(0, n_slices, (1, ))] = 0
39|     # randomly permute slices
```
**EN:** Assigns `slice_sizes[torch.randint(0, n_slices...` and stores constant `0`.

**CN:** 将 `slice_sizes[torch.randint(0, n_slices...` and 保存常量 `0`.

### Block 21 — Lines 40-41 (test_remap_ragged_tensor_metadata)
```python
40|     slice_map = torch.randperm(n_slices, device=device, dtype=torch.int32)
41|     # discard random slices
```
**EN:** Assigns `slice_map` and calls `torch.randperm`.

**CN:** 将 `slice_map`，并调用 `torch.randperm`.

### Block 22 — Lines 42-42 (test_remap_ragged_tensor_metadata)
```python
42|     slice_map[torch.randint(0, len(slice_map), (5, ))] = -1
```
**EN:** Assigns `slice_map[torch.randint(0, len(slice_...` and evaluates `-1`.

**CN:** 将 `slice_map[torch.randint(0, len(slice_...` and 计算 `-1`.

### Block 23 — Lines 43-43 (test_remap_ragged_tensor_metadata)
```python
43|     tri_metadata = make_ragged_tensor_metadata(slice_sizes, n_total_rows)
```
**EN:** Assigns `tri_metadata` and calls `make_ragged_tensor_metadata`.

**CN:** 将 `tri_metadata`，并调用 `make_ragged_tensor_metadata`.

### Block 24 — Lines 44-44 (test_remap_ragged_tensor_metadata)
```python
44|     ref_metadata = make_ragged_tensor_metadata_torch(slice_sizes, n_total_rows)
```
**EN:** Assigns `ref_metadata` and calls `make_ragged_tensor_metadata_torch`.

**CN:** 将 `ref_metadata`，并调用 `make_ragged_tensor_metadata_torch`.

### Block 25 — Lines 45-45 (test_remap_ragged_tensor_metadata)
```python
45|     tri_metadata = remap_ragged_tensor_metadata(tri_metadata, slice_map)
```
**EN:** Assigns `tri_metadata` and calls `remap_ragged_tensor_metadata`.

**CN:** 将 `tri_metadata`，并调用 `remap_ragged_tensor_metadata`.

### Block 26 — Lines 46-46 (test_remap_ragged_tensor_metadata)
```python
46|     ref_metadata = remap_ragged_tensor_metadata_torch(ref_metadata, slice_map)
```
**EN:** Assigns `ref_metadata` and calls `remap_ragged_tensor_metadata_torch`.

**CN:** 将 `ref_metadata`，并调用 `remap_ragged_tensor_metadata_torch`.

### Block 27 — Lines 47-47 (test_remap_ragged_tensor_metadata)
```python
47|     assert_equal(tri_metadata.slice_sizes, ref_metadata.slice_sizes)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 28 — Lines 48-48 (test_remap_ragged_tensor_metadata)
```python
48|     assert_equal(tri_metadata.slice_offs, ref_metadata.slice_offs)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 29 — Lines 49-49 (test_remap_ragged_tensor_metadata)
```python
49|     assert_equal(tri_metadata.block_offs_data, ref_metadata.block_offs_data)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 30 — Lines 50-52 (test_remap_ragged_tensor_metadata)
```python
50|     assert_equal(tri_metadata.block_schedule_data, ref_metadata.block_schedule_data)
51| 
52| 
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 31 — Lines 53-56 (test_make_bitmatrix_metadata)
```python
53| @pytest.mark.parametrize("n_rows", [7, 256, 17111])
54| @pytest.mark.parametrize("n_cols", [13, 32, 128, 811])
55| @pytest.mark.parametrize("k", [1, 4, 8])
56| def test_make_bitmatrix_metadata(n_rows, n_cols, k):
```
**EN:** Defines function `test_make_bitmatrix_metadata(n_rows, n_cols, k)` with decorators `pytest.mark.parametrize('n_rows', [7,...`, `pytest.mark.parametrize('n_cols', [13...`, `pytest.mark.parametrize('k', [1, 4, 8])` for this module. The body mainly branches on runtime conditions; prepares intermediate values; invokes `torch.manual_seed`. It uses calls such as `pytest.mark.parametrize`, `torch.manual_seed`, `torch.arange(n_rows, device=device).u...`, `torch.zeros`, `bitmask_data.index_put_` to implement its workflow.

**CN:** 定义函数 `test_make_bitmatrix_metadata(n_rows, n_cols, k)`，带有装饰器 `pytest.mark.parametrize('n_rows', [7,...`, `pytest.mark.parametrize('n_cols', [13...`, `pytest.mark.parametrize('k', [1, 4, 8])`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; invokes `torch.manual_seed`. 其中会调用 `pytest.mark.parametrize`, `torch.manual_seed`, `torch.arange(n_rows, device=device).u...`, `torch.zeros`, `bitmask_data.index_put_` 来实现其工作流程.

### Block 32 — Lines 57-58 (test_make_bitmatrix_metadata)
```python
57|     if k > n_cols:
58|         pytest.skip("k must be <= n_cols")
```
**EN:** Checks `k > n_cols`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `k > n_cols`. 真分支主要invokes `pytest.skip`.

### Block 33 — Lines 59-59 (test_make_bitmatrix_metadata)
```python
59|     device = "cuda"
```
**EN:** Assigns `device` and stores a string literal or docstring.

**CN:** 将 `device` and 保存字符串字面量或文档字符串.

### Block 34 — Lines 60-62 (test_make_bitmatrix_metadata)
```python
60|     torch.manual_seed(0)
61|     # random permutation of column indices
62|     # NOTE: `indx` *must* be sorted
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 35 — Lines 63-63 (test_make_bitmatrix_metadata)
```python
63|     indx = torch.rand(n_rows, n_cols, device=device).argsort(dim=1).int()[:, :k]
```
**EN:** Assigns `indx` and evaluates `torch.rand(n_rows, n_cols, device=device).argsort(dim=1).int()[:, :k]`.

**CN:** 将 `indx` and 计算 `torch.rand(n_rows, n_cols, device=device).argsort(dim=1).int()[:, :k]`.

### Block 36 — Lines 64-65 (test_make_bitmatrix_metadata)
```python
64|     indx = torch.sort(indx, dim=1)[0]
65|     # create bitmask
```
**EN:** Assigns `indx` and evaluates `torch.sort(indx, dim=1)[0]`.

**CN:** 将 `indx` and 计算 `torch.sort(indx, dim=1)[0]`.

### Block 37 — Lines 66-66 (test_make_bitmatrix_metadata)
```python
66|     rows = torch.arange(n_rows, device=device).unsqueeze(1).expand_as(indx)
```
**EN:** Assigns `rows` and calls `torch.arange(n_rows, device=device).unsqueeze(1...`.

**CN:** 将 `rows`，并调用 `torch.arange(n_rows, device=device).unsqueeze(1...`.

### Block 38 — Lines 67-67 (test_make_bitmatrix_metadata)
```python
67|     bitmask_data = torch.zeros((n_rows, (n_cols + 31) // 32), dtype=torch.int32, device=device)
```
**EN:** Assigns `bitmask_data` and calls `torch.zeros`.

**CN:** 将 `bitmask_data`，并调用 `torch.zeros`.

### Block 39 — Lines 68-68 (test_make_bitmatrix_metadata)
```python
68|     bitmask_data.index_put_((rows, indx // 32), 1 << (indx % 32), accumulate=True)
```
**EN:** Calls `bitmask_data.index_put_` for side effects, registration, or validation.

**CN:** 调用 `bitmask_data.index_put_` ，用于副作用、注册或校验。

### Block 40 — Lines 69-70 (test_make_bitmatrix_metadata)
```python
69|     bitmask = wrap_torch_tensor(bitmask_data.view(torch.uint32), dtype=BIT, shape=(n_rows, n_cols))
70|     # make metadata and compare
```
**EN:** Assigns `bitmask` and calls `wrap_torch_tensor`.

**CN:** 将 `bitmask`，并调用 `wrap_torch_tensor`.

### Block 41 — Lines 71-71 (test_make_bitmatrix_metadata)
```python
71|     metadata_tri = make_bitmatrix_metadata(indx, bitmask)
```
**EN:** Assigns `metadata_tri` and calls `make_bitmatrix_metadata`.

**CN:** 将 `metadata_tri`，并调用 `make_bitmatrix_metadata`.

### Block 42 — Lines 72-72 (test_make_bitmatrix_metadata)
```python
72|     metadata_ref = make_bitmatrix_metadata_torch(indx, bitmask)
```
**EN:** Assigns `metadata_ref` and calls `make_bitmatrix_metadata_torch`.

**CN:** 将 `metadata_ref`，并调用 `make_bitmatrix_metadata_torch`.

### Block 43 — Lines 73-73 (test_make_bitmatrix_metadata)
```python
73|     assert_equal(metadata_tri.col_sum, metadata_ref.col_sum)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 44 — Lines 74-74 (test_make_bitmatrix_metadata)
```python
74|     assert_equal(metadata_tri.row_sorted_indx, metadata_ref.row_sorted_indx)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

### Block 45 — Lines 75-75 (test_make_bitmatrix_metadata)
```python
75|     assert_equal(metadata_tri.col_sorted_indx, metadata_ref.col_sorted_indx)
```
**EN:** Calls `assert_equal` for side effects, registration, or validation.

**CN:** 调用 `assert_equal` ，用于副作用、注册或校验。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `test_make_ragged_tensor_metadata`, `test_remap_ragged_tensor_metadata`, `test_make_bitmatrix_metadata`.
  **CN:** 主要符号：`test_make_ragged_tensor_metadata`, `test_remap_ragged_tensor_metadata`, `test_make_bitmatrix_metadata`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `pytest`, `torch`.
  **CN:** 外部模块：`pytest`, `torch`。
- **EN:** Internal modules: `triton_kernels.tensor_details.dtype (BIT)`, `triton_kernels.tensor (make_ragged_tensor_metadata, make_ragged_tensor_metadata_torch, remap_ragged_tensor_metadata, remap_ragged_tensor_metadata_torch, make_bitmatrix_metadata, make_bitmatrix_metadata_torch)`, `triton_kernels.testing (assert_equal)`.
  **CN:** 内部模块：`triton_kernels.tensor_details.dtype (BIT)`, `triton_kernels.tensor (make_ragged_tensor_metadata, make_ragged_tensor_metadata_torch, remap_ragged_tensor_metadata, remap_ragged_tensor_metadata_torch, make_bitmatrix_metadata, make_bitmatrix_metadata_torch)`, `triton_kernels.testing (assert_equal)`。
