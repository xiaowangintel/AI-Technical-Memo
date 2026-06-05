# test_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_reduce.py`
- **Purpose / 用途:** Pytest coverage for test reduce; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test reduce 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-11 (module)
```python
 1| import pytest
 2| import torch
 3| from triton.testing import do_bench
 4| from triton_kernels.reduce import reduce, reduce_torch, PostprocessFn, FnSpecs
 5| from triton_kernels.numerics_details.mxfp import upcast_from_mxfp_torch, downcast_to_mxfp_torch
 6| from triton_kernels.numerics import InFlexData, OutFlexData
 7| from triton_kernels.target_info import is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4
 8| import triton
 9| import triton.language as tl
10| 
11| 
```
**EN:** This block imports `pytest`, `torch`, `triton.testing (do_bench)`, `triton_kernels.reduce (reduce, reduce_torch, PostprocessFn, FnSpecs)`, `triton_kernels.numerics_details.mxfp (upcast_from_mxfp_torch, downcast_to_mxfp_torch)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.target_info (is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4)`, `triton` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pytest`, `torch`, `triton.testing (do_bench)`, `triton_kernels.reduce (reduce, reduce_torch, PostprocessFn, FnSpecs)`, `triton_kernels.numerics_details.mxfp (upcast_from_mxfp_torch, downcast_to_mxfp_torch)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.target_info (is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4)`, `triton` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 12-12 (init_mask)
```python
12| def init_mask(mask_mode, B, M, N, device):
```
**EN:** Defines function `init_mask(mask_mode, B, M, N, device)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; branches on runtime conditions. It uses calls such as `mask.expand`, `(torch.rand((B, M, N), device=device)...`, `(torch.rand((1, M, N), device=device)...`, `(torch.rand((B, 1, N), device=device)...`, `(torch.rand((B, M, 1), device=device)...` to implement its workflow.

**CN:** 定义函数 `init_mask(mask_mode, B, M, N, device)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `mask.expand`, `(torch.rand((B, M, N), device=device)...`, `(torch.rand((1, M, N), device=device)...`, `(torch.rand((B, 1, N), device=device)...`, `(torch.rand((B, M, 1), device=device)...` 来实现其工作流程.

### Block 3 — Lines 13-14 (init_mask)
```python
13|     if mask_mode == "none":
14|         return None
```
**EN:** Checks `mask_mode == 'none'`. The true branch mainly returns the computed result.

**CN:** 检查 `mask_mode == 'none'`. 真分支主要返回计算结果.

### Block 4 — Lines 15-16 (init_mask)
```python
15|     if mask_mode == "full":
16|         mask = (torch.rand((B, M, N), device=device) > 0.3).to(torch.int8)
```
**EN:** Checks `mask_mode == 'full'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mask_mode == 'full'`. 真分支主要准备中间值.

### Block 5 — Lines 17-18 (init_mask)
```python
17|     if mask_mode == "broadcast_b":
18|         mask = (torch.rand((1, M, N), device=device) > 0.3).to(torch.int8)
```
**EN:** Checks `mask_mode == 'broadcast_b'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mask_mode == 'broadcast_b'`. 真分支主要准备中间值.

### Block 6 — Lines 19-20 (init_mask)
```python
19|     if mask_mode == "broadcast_m":
20|         mask = (torch.rand((B, 1, N), device=device) > 0.3).to(torch.int8)
```
**EN:** Checks `mask_mode == 'broadcast_m'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mask_mode == 'broadcast_m'`. 真分支主要准备中间值.

### Block 7 — Lines 21-22 (init_mask)
```python
21|     if mask_mode == "broadcast_n":
22|         mask = (torch.rand((B, M, 1), device=device) > 0.3).to(torch.int8)
```
**EN:** Checks `mask_mode == 'broadcast_n'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mask_mode == 'broadcast_n'`. 真分支主要准备中间值.

### Block 8 — Lines 23-25 (init_mask)
```python
23|     return mask.expand(B, M, N)
24| 
25| 
```
**EN:** Returns `mask.expand(B, M, N)`.

**CN:** 返回 `mask.expand(B, M, N)`.

### Block 9 — Lines 26-26 (dtype_str_to_torch)
```python
26| def dtype_str_to_torch(dtype_str: str) -> torch.dtype:
```
**EN:** Defines function `dtype_str_to_torch(dtype_str)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; returns the computed result. It uses calls such as `getattr` to implement its workflow.

**CN:** 定义函数 `dtype_str_to_torch(dtype_str)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 返回计算结果. 其中会调用 `getattr` 来实现其工作流程.

### Block 10 — Lines 27-28 (dtype_str_to_torch)
```python
27|     if dtype_str == "float4_e2m1":
28|         return torch.uint8
```
**EN:** Checks `dtype_str == 'float4_e2m1'`. The true branch mainly returns the computed result.

**CN:** 检查 `dtype_str == 'float4_e2m1'`. 真分支主要返回计算结果.

### Block 11 — Lines 29-30 (dtype_str_to_torch)
```python
29|     if dtype_str == "float8":
30|         return torch.float8_e4m3fn
```
**EN:** Checks `dtype_str == 'float8'`. The true branch mainly returns the computed result.

**CN:** 检查 `dtype_str == 'float8'`. 真分支主要返回计算结果.

### Block 12 — Lines 31-33 (dtype_str_to_torch)
```python
31|     return getattr(torch, dtype_str)
32| 
33| 
```
**EN:** Returns `getattr(torch, dtype_str)`.

**CN:** 返回 `getattr(torch, dtype_str)`.

### Block 13 — Lines 34-35 (plus_a_reduce)
```python
34| @triton.jit
35| def plus_a_reduce(x, a):
```
**EN:** Defines function `plus_a_reduce(x, a)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `tl.sum`, `y.reshape` to implement its workflow.

**CN:** 定义函数 `plus_a_reduce(x, a)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `tl.sum`, `y.reshape` 来实现其工作流程.

### Block 14 — Lines 36-36 (plus_a_reduce)
```python
36|     y = x + a
```
**EN:** Assigns `y` and evaluates `x + a`.

**CN:** 将 `y` and 计算 `x + a`.

### Block 15 — Lines 37-39 (plus_a_reduce)
```python
37|     return tl.sum(y.reshape([x.shape[0], x.shape[1] // 2, 2]), axis=2)
38| 
39| 
```
**EN:** Returns `tl.sum(y.reshape([x.shape[0], x.shape[1] // 2, 2]), axis=2)`.

**CN:** 返回 `tl.sum(y.reshape([x.shape[0], x.shape[1] // 2, 2]), axis=2)`.

### Block 16 — Lines 40-66 (test_op)
```python
40| @pytest.mark.parametrize("B, M, N, postprocess_fn", [
41|     (311, 384, 384, None),
42|     (384, 311, 384, None),
43|     (384, 384, 311, None),
44|     (512, 512, 512, None),
45|     (512, 512, 512, "plus_ten"),
46|     (4, 4, 4, None),
47|     (3, 15, 25, None),
48|     (5, 9999, 2345, None),
49|     (15, 345, 789, None),
50| ])
51| @pytest.mark.parametrize("dtype_str", [
52|     "float16",
53|     "float32",
54|     "mxfloat8",
55|     "flexfloat8",
56| ])
57| @pytest.mark.parametrize("mask_mode", [
58|     "none",  # no mask
59|     "full",  # full-sized mask [B,M,N]
60|     "broadcast_b",  # broadcast over B: [1,M,N]
61|     "broadcast_m",  # broadcast over M: [B,1,N]
62|     "broadcast_n",  # broadcast over N: [B,M,1]
63| ])
64| @pytest.mark.parametrize("dim", [0, 1, 2])
65| def test_op(B, M, N, dtype_str, dim, mask_mode, postprocess_fn):
66|     # Check float8 hardware support
```
**EN:** Defines function `test_op(B, M, N, dtype_str, dim, mask_mode, postprocess_fn)` with decorators `pytest.mark.parametrize('B, M, N, pos...`, `pytest.mark.parametrize('dtype_str', ...`, `pytest.mark.parametrize('mask_mode', ...`, `pytest.mark.parametrize('dim', [0, 1,...` for this module. The body mainly branches on runtime conditions; invokes `torch.manual_seed`; prepares intermediate values. It uses calls such as `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randn`, `init_mask`, `x.clone().detach().requires_grad_` to implement its workflow.

**CN:** 定义函数 `test_op(B, M, N, dtype_str, dim, mask_mode, postprocess_fn)`，带有装饰器 `pytest.mark.parametrize('B, M, N, pos...`, `pytest.mark.parametrize('dtype_str', ...`, `pytest.mark.parametrize('mask_mode', ...`, `pytest.mark.parametrize('dim', [0, 1,...`，供本模块使用. 主体主要根据运行时条件分支; invokes `torch.manual_seed`; 准备中间值. 其中会调用 `pytest.mark.parametrize`, `torch.manual_seed`, `torch.randn`, `init_mask`, `x.clone().detach().requires_grad_` 来实现其工作流程.

### Block 17 — Lines 67-72 (test_op)
```python
67|     if "float8" in dtype_str:
68|         if is_cuda() and torch.cuda.get_device_capability() < (9, 0):
69|             pytest.skip("float8 not supported on CUDA < 9.0")
70|         if is_hip() and not (is_hip_cdna3() or is_hip_cdna4()):
71|             pytest.skip("float8 not supported on AMD GPU < CDNA3")
72| 
```
**EN:** Checks `'float8' in dtype_str`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `'float8' in dtype_str`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 18 — Lines 73-73 (test_op)
```python
73|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 19 — Lines 74-74 (test_op)
```python
74|     device = "cuda"
```
**EN:** Assigns `device` and stores a string literal or docstring.

**CN:** 将 `device` and 保存字符串字面量或文档字符串.

### Block 20 — Lines 75-75 (test_op)
```python
75|     x = torch.randn((B, M, N), device=device, dtype=torch.float32, requires_grad=True)
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 21 — Lines 76-76 (test_op)
```python
76|     x_mscale, x_flex = None, None
```
**EN:** Assigns `x_mscale`, `x_flex` and builds a tuple.

**CN:** 将 `x_mscale`, `x_flex` and 构造一个元组.

### Block 22 — Lines 77-77 (test_op)
```python
77|     y_flex_tri, y_flex_ref = None, None
```
**EN:** Assigns `y_flex_tri`, `y_flex_ref` and builds a tuple.

**CN:** 将 `y_flex_tri`, `y_flex_ref` and 构造一个元组.

### Block 23 — Lines 78-80 (test_op)
```python
78|     if is_mx := dtype_str.startswith("mx"):
79|         dtype = dtype_str_to_torch(dtype_str.removeprefix("mx"))
80|         x, x_mscale = downcast_to_mxfp_torch(x.to(torch.float16), dtype, axis=-1)
```
**EN:** Checks `(is_mx := dtype_str.startswith('mx'))`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `(is_mx := dtype_str.startswith('mx'))`. 真分支主要准备中间值; 准备中间值.

### Block 24 — Lines 81-88 (test_op)
```python
81|     if is_flex := dtype_str.startswith("flex"):
82|         dtype = dtype_str_to_torch(dtype_str.removeprefix("flex"))
83|         expected_scale = torch.tensor([4], device=device, dtype=torch.float32)
84|         x_flex = InFlexData(scale=torch.tensor([2], device=device, dtype=torch.float32))
85|         x = x / x_flex.scale
86|         x = x.to(dtype)
87|         y_flex_tri = OutFlexData(expected_scale=expected_scale, actual_scale=torch.empty_like(expected_scale))
88|         y_flex_ref = OutFlexData(expected_scale=expected_scale, actual_scale=torch.empty_like(expected_scale))
```
**EN:** Checks `(is_flex := dtype_str.startswith('flex'))`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `(is_flex := dtype_str.startswith('flex'))`. 真分支主要准备中间值; 准备中间值.

### Block 25 — Lines 89-89 (test_op)
```python
89|     mask = init_mask(mask_mode, B, M, N, device)
```
**EN:** Assigns `mask` and calls `init_mask`.

**CN:** 将 `mask`，并调用 `init_mask`.

### Block 26 — Lines 90-90 (test_op)
```python
90|     expected_exception = ValueError if dim == 2 and is_mx else None
```
**EN:** Assigns `expected_exception` and uses conditional expression `ValueError if dim == 2 and is_mx else None`.

**CN:** 将 `expected_exception` and 使用条件表达式 `ValueError if dim == 2 and is_mx else None`.

### Block 27 — Lines 91-94 (test_op)
```python
91|     if expected_exception is not None:
92|         with pytest.raises(expected_exception):
93|             reduce(x, dim=dim, mask=mask, x_mxscale=x_mscale)
94|         return
```
**EN:** Checks `expected_exception is not None`. The true branch mainly uses contextual resources; returns the computed result.

**CN:** 检查 `expected_exception is not None`. 真分支主要使用上下文资源; 返回计算结果.

### Block 28 — Lines 95-101 (test_op)
```python
 95|     if postprocess_fn == "plus_ten":
 96|         postprocess_fn_tri = PostprocessFn(specs=FnSpecs("plus_a", plus_a_reduce, ("a", ), reduction_n=2),
 97|                                            fn_args=(10, ))
 98|         postprocess_fn_ref = lambda x: (x + 10).reshape([x.shape[0], x.shape[1] // 2, 2]).sum(dim=2)
 99|     else:
100|         postprocess_fn_tri = postprocess_fn_ref = None
101|     # run forward pass
```
**EN:** Checks `postprocess_fn == 'plus_ten'`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `postprocess_fn == 'plus_ten'`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 29 — Lines 102-102 (test_op)
```python
102|     x_tri = x.clone().detach().requires_grad_(True)
```
**EN:** Assigns `x_tri` and calls `x.clone().detach().requires_grad_`.

**CN:** 将 `x_tri`，并调用 `x.clone().detach().requires_grad_`.

### Block 30 — Lines 103-103 (test_op)
```python
103|     x_ref = x.clone().detach().requires_grad_(True)
```
**EN:** Assigns `x_ref` and calls `x.clone().detach().requires_grad_`.

**CN:** 将 `x_ref`，并调用 `x.clone().detach().requires_grad_`.

### Block 31 — Lines 104-105 (test_op)
```python
104|     y_tri, y_tri_mxscale = reduce(x_tri, dim=dim, mask=mask, x_mxscale=x_mscale, x_flex=x_flex, y_flex=y_flex_tri,
105|                                   postprocess_fn1=postprocess_fn_tri)
```
**EN:** Assigns `y_tri`, `y_tri_mxscale` and calls `reduce`.

**CN:** 将 `y_tri`, `y_tri_mxscale`，并调用 `reduce`.

### Block 32 — Lines 106-107 (test_op)
```python
106|     y_ref, y_ref_mxscale = reduce_torch(x_ref, dim=dim, mask=mask, x_mxscale=x_mscale, x_flex=x_flex, y_flex=y_flex_ref,
107|                                         postprocess_fn1=postprocess_fn_ref)
```
**EN:** Assigns `y_ref`, `y_ref_mxscale` and calls `reduce_torch`.

**CN:** 将 `y_ref`, `y_ref_mxscale`，并调用 `reduce_torch`.

### Block 33 — Lines 108-110 (test_op)
```python
108|     if is_mx:
109|         y_ref = upcast_from_mxfp_torch(y_ref, y_ref_mxscale, torch.float16, axis=-1)
110|         y_tri = upcast_from_mxfp_torch(y_tri, y_tri_mxscale, torch.float16, axis=-1)
```
**EN:** Checks `is_mx`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `is_mx`. 真分支主要准备中间值; 准备中间值.

### Block 34 — Lines 111-111 (test_op)
```python
111|     assert torch.allclose(y_tri.float(), y_ref.float(), atol=1e-3, rtol=1e-3)
```
**EN:** Asserts `torch.allclose(y_tri.float(), y_ref.float(), atol=0.001, rtol=0.001)` to enforce invariants.

**CN:** 断言 `torch.allclose(y_tri.float(), y_ref.float(), atol=0.001, rtol=0.001)` 以确保不变量成立。

### Block 35 — Lines 112-113 (test_op)
```python
112|     if is_flex:
113|         torch.allclose(y_flex_tri.actual_scale, y_flex_ref.actual_scale, atol=1e-3, rtol=1e-3)
```
**EN:** Checks `is_flex`. The true branch mainly invokes `torch.allclose`.

**CN:** 检查 `is_flex`. 真分支主要invokes `torch.allclose`.

### Block 36 — Lines 114-114 (test_op)
```python
114|     run_bwd = postprocess_fn is None and "float8" not in dtype_str
```
**EN:** Assigns `run_bwd` and evaluates `postprocess_fn is None and 'float8' not in dtype_str`.

**CN:** 将 `run_bwd` and 计算 `postprocess_fn is None and 'float8' not in dtype_str`.

### Block 37 — Lines 115-121 (test_op)
```python
115|     if run_bwd:
116|         dy = torch.randn_like(y_tri)
117|         y_tri.backward(dy)
118|         y_ref.backward(dy)
119|         assert torch.allclose(x_tri.grad.float(), x_ref.grad.float(), atol=1e-3, rtol=1e-3)
120| 
121| 
```
**EN:** Checks `run_bwd`. The true branch mainly prepares intermediate values; invokes `y_tri.backward`.

**CN:** 检查 `run_bwd`. 真分支主要准备中间值; invokes `y_tri.backward`.

### Block 38 — Lines 122-123 (bench_reduce)
```python
122| def bench_reduce(B: int = 4, M: int = 4096, N: int = 4096, *, dim: int = 0, dtype: torch.dtype = torch.float16,
123|                  iters: int = 200, mask_mode: str = "none"):
```
**EN:** Defines function `bench_reduce(B, M, N, dim, dtype, iters, mask_mode)` for this module. The body mainly invokes `torch.manual_seed`; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.manual_seed`, `torch.randn((B, M, N), device=device,...`, `init_mask`, `do_bench`, `print` to implement its workflow.

**CN:** 定义函数 `bench_reduce(B, M, N, dim, dtype, iters, mask_mode)`，供本模块使用. 主体主要invokes `torch.manual_seed`; 准备中间值; 准备中间值. 其中会调用 `torch.manual_seed`, `torch.randn((B, M, N), device=device,...`, `init_mask`, `do_bench`, `print` 来实现其工作流程.

### Block 39 — Lines 124-124 (bench_reduce)
```python
124|     torch.manual_seed(0)
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 40 — Lines 125-125 (bench_reduce)
```python
125|     device = "cuda"
```
**EN:** Assigns `device` and stores a string literal or docstring.

**CN:** 将 `device` and 保存字符串字面量或文档字符串.

### Block 41 — Lines 126-126 (bench_reduce)
```python
126|     x = torch.randn((B, M, N), device=device, dtype=torch.float32).to(dtype)
```
**EN:** Assigns `x` and calls `torch.randn((B, M, N), device=device, dtype=tor...`.

**CN:** 将 `x`，并调用 `torch.randn((B, M, N), device=device, dtype=tor...`.

### Block 42 — Lines 127-127 (bench_reduce)
```python
127|     mask = init_mask(mask_mode, B, M, N, device)
```
**EN:** Assigns `mask` and calls `init_mask`.

**CN:** 将 `mask`，并调用 `init_mask`.

### Block 43 — Lines 128-128 (bench_reduce)
```python
128|     ms = do_bench(lambda: reduce(x, dim=dim, mask=mask), rep=iters)
```
**EN:** Assigns `ms` and calls `do_bench`.

**CN:** 将 `ms`，并调用 `do_bench`.

### Block 44 — Lines 129-129 (bench_reduce)
```python
129|     nnz = x.numel() if mask is None else (mask.expand(B, M, N) != 0).sum()
```
**EN:** Assigns `nnz` and uses conditional expression `x.numel() if mask is None else (mask.expand(B, M, N) != 0...`.

**CN:** 将 `nnz` and 使用条件表达式 `x.numel() if mask is None else (mask.expand(B, M, N) != 0...`.

### Block 45 — Lines 130-130 (bench_reduce)
```python
130|     read_bytes = nnz * x.element_size()
```
**EN:** Assigns `read_bytes` and evaluates `nnz * x.element_size()`.

**CN:** 将 `read_bytes` and 计算 `nnz * x.element_size()`.

### Block 46 — Lines 131-131 (bench_reduce)
```python
131|     out_elems = (M * N) if dim == 0 else ((B * N) if dim == 1 else (B * M))
```
**EN:** Assigns `out_elems` and uses conditional expression `M * N if dim == 0 else B * N if dim == 1 else B * M`.

**CN:** 将 `out_elems` and 使用条件表达式 `M * N if dim == 0 else B * N if dim == 1 else B * M`.

### Block 47 — Lines 132-132 (bench_reduce)
```python
132|     write_bytes = out_elems * x.element_size()
```
**EN:** Assigns `write_bytes` and evaluates `out_elems * x.element_size()`.

**CN:** 将 `write_bytes` and 计算 `out_elems * x.element_size()`.

### Block 48 — Lines 133-133 (bench_reduce)
```python
133|     mask_bytes = 0 if mask is None else (mask.numel() * mask.element_size())
```
**EN:** Assigns `mask_bytes` and uses conditional expression `0 if mask is None else mask.numel() * mask.element_size()`.

**CN:** 将 `mask_bytes` and 使用条件表达式 `0 if mask is None else mask.numel() * mask.element_size()`.

### Block 49 — Lines 134-134 (bench_reduce)
```python
134|     bytes_total = read_bytes + write_bytes + mask_bytes
```
**EN:** Assigns `bytes_total` and evaluates `read_bytes + write_bytes + mask_bytes`.

**CN:** 将 `bytes_total` and 计算 `read_bytes + write_bytes + mask_bytes`.

### Block 50 — Lines 135-135 (bench_reduce)
```python
135|     gbps = (bytes_total) / ms / 1e6
```
**EN:** Assigns `gbps` and evaluates `bytes_total / ms / 1000000.0`.

**CN:** 将 `gbps` and 计算 `bytes_total / ms / 1000000.0`.

### Block 51 — Lines 136-136 (bench_reduce)
```python
136|     desc = f"reduce: B={B}, M={M}, N={N}, dim={dim}, dtype={str(dtype).split('.')[-1]}, mask={mask_mode}"
```
**EN:** Assigns `desc` and evaluates `f'reduce: B={B}, M={M}, N={N}, dim={dim}, dtype={str(dtype).split('...`.

**CN:** 将 `desc` and 计算 `f'reduce: B={B}, M={M}, N={N}, dim={dim}, dtype={str(dtype).split('...`.

### Block 52 — Lines 137-145 (bench_reduce)
```python
137|     print(f"{desc} -> {gbps:.2f} GB/s")
138| 
139| 
140| # bench_reduce(B=4, M=8192, N=8192, dim=0, dtype=torch.float16, mask_mode="none")
141| # bench_reduce(B=8192, M=4, N=8192, dim=1, dtype=torch.float16, mask_mode="broadcast_n")
142| # bench_reduce(B=16384, M=4, N=8192, dim=1, dtype=torch.float16, mask_mode="broadcast_n")
143| # bench_reduce(B=65536, M=4, N=8192, dim=1, dtype=torch.float16, mask_mode="broadcast_n")
144| # bench_reduce(B=8192, M=4, N=8192, dim=1, dtype=torch.float16, mask_mode="broadcast_m")
145| # bench_reduce(B=8192, M=4, N=8192, dim=1, dtype=torch.float16, mask_mode="broadcast_b")
```
**EN:** Calls `print` for side effects, registration, or validation.

**CN:** 调用 `print` ，用于副作用、注册或校验。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `init_mask`, `dtype_str_to_torch`, `plus_a_reduce`, `test_op`, `bench_reduce`.
  **CN:** 主要符号：`init_mask`, `dtype_str_to_torch`, `plus_a_reduce`, `test_op`, `bench_reduce`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Flex-point style metadata or scaling is used to control quantized arithmetic.
  **CN:** 使用 Flex-point 风格的元数据或缩放来控制量化计算。
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。
- **EN:** Benchmark-oriented code sweeps parameter spaces and records performance metrics.
  **CN:** 面向基准测试的代码会扫描参数空间并记录性能指标。

## Dependencies / 依赖关系
- **EN:** External modules: `pytest`, `torch`, `triton.testing (do_bench)`, `triton`, `triton.language`.
  **CN:** 外部模块：`pytest`, `torch`, `triton.testing (do_bench)`, `triton`, `triton.language`。
- **EN:** Internal modules: `triton_kernels.reduce (reduce, reduce_torch, PostprocessFn, FnSpecs)`, `triton_kernels.numerics_details.mxfp (upcast_from_mxfp_torch, downcast_to_mxfp_torch)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.target_info (is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4)`.
  **CN:** 内部模块：`triton_kernels.reduce (reduce, reduce_torch, PostprocessFn, FnSpecs)`, `triton_kernels.numerics_details.mxfp (upcast_from_mxfp_torch, downcast_to_mxfp_torch)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.target_info (is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4)`。
