# topk.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/topk.py`
- **Purpose / 用途:** Implementation module for topk; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols make_empty, topk_forward, topk_backward, TopK, topk. / 用于 topk 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 make_empty、topk_forward、topk_backward、TopK、topk。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-11 (module)
```python
 1| import torch
 2| import triton
 3| from triton_kernels.topk_details._topk_forward import _topk_forward
 4| from triton_kernels.topk_details._topk_backward import _topk_backward
 5| from triton_kernels.tensor import SparseMatrix, Tensor
 6| from triton_kernels.tensor_details.dtype import BIT
 7| from typing import Optional, Union
 8| from triton_kernels.distributed import SymmetricMemoryPool
 9| from triton_kernels.tensor import wrap_torch_tensor, dtype_to_torch_dtype
10| 
11| 
```
**EN:** This block imports `torch`, `triton`, `triton_kernels.topk_details._topk_forward (_topk_forward)`, `triton_kernels.topk_details._topk_backward (_topk_backward)`, `triton_kernels.tensor (SparseMatrix, Tensor)`, `triton_kernels.tensor_details.dtype (BIT)`, `typing (Optional, Union)`, `triton_kernels.distributed (SymmetricMemoryPool)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch`, `triton`, `triton_kernels.topk_details._topk_forward (_topk_forward)`, `triton_kernels.topk_details._topk_backward (_topk_backward)`, `triton_kernels.tensor (SparseMatrix, Tensor)`, `triton_kernels.tensor_details.dtype (BIT)`, `typing (Optional, Union)`, `triton_kernels.distributed (SymmetricMemoryPool)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 12-12 (make_empty)
```python
12| def make_empty(offset, shape, dtype, device, all_gather, symm_mem_pool):
```
**EN:** Defines function `make_empty(offset, shape, dtype, device, all_gather, symm_mem_pool)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; prepares intermediate values. It uses calls such as `dtype_to_torch_dtype`, `torch.empty`, `symm_mem_pool.make_empty`, `symm_mem_pool.align_up`, `ret.numel` to implement its workflow.

**CN:** 定义函数 `make_empty(offset, shape, dtype, device, all_gather, symm_mem_pool)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 准备中间值. 其中会调用 `dtype_to_torch_dtype`, `torch.empty`, `symm_mem_pool.make_empty`, `symm_mem_pool.align_up`, `ret.numel` 来实现其工作流程.

### Block 3 — Lines 13-13 (make_empty)
```python
13|     dtype = dtype_to_torch_dtype(dtype)
```
**EN:** Assigns `dtype` and calls `dtype_to_torch_dtype`.

**CN:** 将 `dtype`，并调用 `dtype_to_torch_dtype`.

### Block 4 — Lines 14-20 (make_empty)
```python
14|     if all_gather:
15|         rank_id = symm_mem_pool.mesh.local_rank
16|         ret_bufs = symm_mem_pool.make_empty(shape=shape, dtype=dtype, region="topk", region_offset=offset)
17|         ret = ret_bufs[rank_id]
18|         offset = symm_mem_pool.align_up(offset + ret.numel() * ret.element_size(),
19|                                         symm_mem_pool.regions["topk"].alignment)
20|         return ret_bufs, ret, offset
```
**EN:** Checks `all_gather`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `all_gather`. 真分支主要准备中间值; 准备中间值.

### Block 5 — Lines 21-21 (make_empty)
```python
21|     ret = torch.empty(shape, dtype=dtype, device=device)
```
**EN:** Assigns `ret` and calls `torch.empty`.

**CN:** 将 `ret`，并调用 `torch.empty`.

### Block 6 — Lines 22-24 (make_empty)
```python
22|     return (ret, ), ret, 0
23| 
24| 
```
**EN:** Returns `((ret,), ret, 0)`.

**CN:** 返回 `((ret,), ret, 0)`.

### Block 7 — Lines 25-25 (topk_forward)
```python
25| def topk_forward(x, k, apply_softmax=True, dim=1, y_indx=None, n_rows=None, all_gather=False, symm_mem_pool=None):
```
**EN:** Defines function `topk_forward(x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `make_empty`, `cdiv`, `_topk_forward[pids,]`, `wrap_torch_tensor`, `isinstance` to implement its workflow.

**CN:** 定义函数 `topk_forward(x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `make_empty`, `cdiv`, `_topk_forward[pids,]`, `wrap_torch_tensor`, `isinstance` 来实现其工作流程.

### Block 8 — Lines 26-29 (topk_forward)
```python
26|     if not isinstance(x, Tensor):
27|         x_shape = [x.shape[0] if n_rows is None else n_rows, x.shape[1]]
28|         x_shape_max = [x.shape[0], x.shape[1]]
29|         x = wrap_torch_tensor(x, shape=x_shape, shape_max=x_shape_max)
```
**EN:** Checks `not isinstance(x, Tensor)`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `not isinstance(x, Tensor)`. 真分支主要准备中间值; 准备中间值.

### Block 9 — Lines 30-30 (topk_forward)
```python
30|     cdiv = lambda a, b: (a + b - 1) // b
```
**EN:** Assigns `cdiv` and defines a lambda.

**CN:** 将 `cdiv` and 定义一个 lambda.

### Block 10 — Lines 31-31 (topk_forward)
```python
31|     BLOCK_M = 32
```
**EN:** Assigns `BLOCK_M` and stores constant `32`.

**CN:** 将 `BLOCK_M` and 保存常量 `32`.

### Block 11 — Lines 32-32 (topk_forward)
```python
32|     BLOCK_N = 32
```
**EN:** Assigns `BLOCK_N` and stores constant `32`.

**CN:** 将 `BLOCK_N` and 保存常量 `32`.

### Block 12 — Lines 33-33 (topk_forward)
```python
33|     use_provided_indx = y_indx is not None
```
**EN:** Assigns `use_provided_indx` and evaluates `y_indx is not None`.

**CN:** 将 `use_provided_indx` and 计算 `y_indx is not None`.

### Block 13 — Lines 34-34 (topk_forward)
```python
34|     assert symm_mem_pool is not None or not all_gather
```
**EN:** Asserts `symm_mem_pool is not None or not all_gather` to enforce invariants.

**CN:** 断言 `symm_mem_pool is not None or not all_gather` 以确保不变量成立。

### Block 14 — Lines 35-35 (topk_forward)
```python
35|     assert len(x.shape) == 2
```
**EN:** Asserts `len(x.shape) == 2` to enforce invariants.

**CN:** 断言 `len(x.shape) == 2` 以确保不变量成立。

### Block 15 — Lines 36-36 (topk_forward)
```python
36|     assert x.shape_max[-1] < 32768
```
**EN:** Asserts `x.shape_max[-1] < 32768` to enforce invariants.

**CN:** 断言 `x.shape_max[-1] < 32768` 以确保不变量成立。

### Block 16 — Lines 37-37 (topk_forward)
```python
37|     assert dim == 1
```
**EN:** Asserts `dim == 1` to enforce invariants.

**CN:** 断言 `dim == 1` 以确保不变量成立。

### Block 17 — Lines 38-38 (topk_forward)
```python
38|     n_rows, n_cols = x.shape
```
**EN:** Assigns `n_rows`, `n_cols` and references `x.shape`.

**CN:** 将 `n_rows`, `n_cols` and 引用 `x.shape`.

### Block 18 — Lines 39-39 (topk_forward)
```python
39|     n_rows_max, _ = x.shape_max
```
**EN:** Assigns `n_rows_max`, `_` and references `x.shape_max`.

**CN:** 将 `n_rows_max`, `_` and 引用 `x.shape_max`.

### Block 19 — Lines 40-40 (topk_forward)
```python
40|     dev = x.device
```
**EN:** Assigns `dev` and references `x.device`.

**CN:** 将 `dev` and 引用 `x.device`.

### Block 20 — Lines 41-43 (topk_forward)
```python
41|     n_rows_out_max = n_rows_max * symm_mem_pool.mesh.world_size if all_gather else n_rows_max
42|     # scratchpad tensors
43|     # NOTE: these are not returned
```
**EN:** Assigns `n_rows_out_max` and uses conditional expression `n_rows_max * symm_mem_pool.mesh.world_size if all_gather ...`.

**CN:** 将 `n_rows_out_max` and 使用条件表达式 `n_rows_max * symm_mem_pool.mesh.world_size if all_gather ...`.

### Block 21 — Lines 44-45 (topk_forward)
```python
44|     y_vals_bufs, y_vals, offset = make_empty(0, (n_rows_out_max, k), x.dtype, dev, all_gather=all_gather,
45|                                              symm_mem_pool=symm_mem_pool)
```
**EN:** Assigns `y_vals_bufs`, `y_vals`, `offset` and calls `make_empty`.

**CN:** 将 `y_vals_bufs`, `y_vals`, `offset`，并调用 `make_empty`.

### Block 22 — Lines 46-51 (topk_forward)
```python
46|     if y_indx is None:
47|         y_indx_bufs, y_indx, offset = make_empty(offset, (n_rows_out_max, k), torch.int16, dev, all_gather=all_gather,
48|                                                  symm_mem_pool=symm_mem_pool)
49|     else:
50|         y_indx_bufs = (y_indx, )
51|     # create bitmatrix in transposed memory layout:
```
**EN:** Checks `y_indx is None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `y_indx is None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 23 — Lines 52-52 (topk_forward)
```python
52|     n_cols_pad = cdiv(n_cols, BLOCK_N) * BLOCK_N
```
**EN:** Assigns `n_cols_pad` and evaluates `cdiv(n_cols, BLOCK_N) * BLOCK_N`.

**CN:** 将 `n_cols_pad` and 计算 `cdiv(n_cols, BLOCK_N) * BLOCK_N`.

### Block 24 — Lines 53-53 (topk_forward)
```python
53|     n_cols_words = n_cols_pad // 32
```
**EN:** Assigns `n_cols_words` and evaluates `n_cols_pad // 32`.

**CN:** 将 `n_cols_words` and 计算 `n_cols_pad // 32`.

### Block 25 — Lines 54-56 (topk_forward)
```python
54|     bitmatrix_bufs, bitmatrix_data, offset = make_empty(offset, (n_cols_words, cdiv(n_rows_out_max, 32) * 32),
55|                                                         torch.uint32, dev, all_gather=all_gather,
56|                                                         symm_mem_pool=symm_mem_pool)
```
**EN:** Assigns `bitmatrix_bufs`, `bitmatrix_data`, `offset` and calls `make_empty`.

**CN:** 将 `bitmatrix_bufs`, `bitmatrix_data`, `offset`，并调用 `make_empty`.

### Block 26 — Lines 57-57 (topk_forward)
```python
57|     bitmatrix_data = torch.transpose(bitmatrix_data, 0, 1)[:n_rows_max]
```
**EN:** Assigns `bitmatrix_data` and evaluates `torch.transpose(bitmatrix_data, 0, 1)[:n_rows_max]`.

**CN:** 将 `bitmatrix_data` and 计算 `torch.transpose(bitmatrix_data, 0, 1)[:n_rows_max]`.

### Block 27 — Lines 58-58 (topk_forward)
```python
58|     pids = cdiv(n_rows_max, BLOCK_M)
```
**EN:** Assigns `pids` and calls `cdiv`.

**CN:** 将 `pids`，并调用 `cdiv`.

### Block 28 — Lines 59-67 (topk_forward)
```python
59|     _topk_forward[(pids, )](
60|         x.storage.data, x.stride(0),  # inputs
61|         y_vals_bufs, y_indx_bufs, y_vals.stride(0), use_provided_indx,  # output [topk]
62|         bitmatrix_bufs, bitmatrix_data.stride(0), bitmatrix_data.stride(1),  # output [bitmatrix]
63|         n_rows, n_cols,  # shapes
64|         symm_mem_pool.mesh.local_rank * n_rows_max if all_gather else 0, BLOCK_M=BLOCK_M,
65|         BLOCK_N=BLOCK_N,  # tunable parameter
66|         APPLY_SOFTMAX=apply_softmax, N_EXPTS_PAD=n_cols_pad, N_EXPTS_ACT=k,  # constants
67|     )
```
**EN:** Calls `_topk_forward[pids,]` for side effects, registration, or validation.

**CN:** 调用 `_topk_forward[pids,]` ，用于副作用、注册或校验。

### Block 29 — Lines 68-69 (topk_forward)
```python
68|     if all_gather:
69|         symm_mem_pool.hdl.barrier(channel=0)
```
**EN:** Checks `all_gather`. The true branch mainly invokes `symm_mem_pool.hdl.barrier`.

**CN:** 检查 `all_gather`. 真分支主要invokes `symm_mem_pool.hdl.barrier`.

### Block 30 — Lines 70-70 (topk_forward)
```python
70|     bitmatrix_shape = [n_rows * symm_mem_pool.mesh.world_size if all_gather else n_rows, n_cols]
```
**EN:** Assigns `bitmatrix_shape` and builds a list.

**CN:** 将 `bitmatrix_shape` and 构造一个列表.

### Block 31 — Lines 71-71 (topk_forward)
```python
71|     bitmatrix_shape_max = [n_rows_out_max, None]
```
**EN:** Assigns `bitmatrix_shape_max` and builds a list.

**CN:** 将 `bitmatrix_shape_max` and 构造一个列表.

### Block 32 — Lines 72-72 (topk_forward)
```python
72|     bitmatrix = wrap_torch_tensor(bitmatrix_data, dtype=BIT, shape=bitmatrix_shape, shape_max=bitmatrix_shape_max)
```
**EN:** Assigns `bitmatrix` and calls `wrap_torch_tensor`.

**CN:** 将 `bitmatrix`，并调用 `wrap_torch_tensor`.

### Block 33 — Lines 73-75 (topk_forward)
```python
73|     return y_vals, y_indx, bitmatrix
74| 
75| 
```
**EN:** Returns `(y_vals, y_indx, bitmatrix)`.

**CN:** 返回 `(y_vals, y_indx, bitmatrix)`.

### Block 34 — Lines 76-76 (topk_backward)
```python
76| def topk_backward(x, y_indx, dy_vals, k, n_rows, apply_softmax):
```
**EN:** Defines function `topk_backward(x, y_indx, dy_vals, k, n_rows, apply_softmax)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `triton.next_power_of_2`, `torch.empty_like`, `_topk_backward[dy_vals.shape[0],]`, `y_indx.stride`, `dy_vals.stride` to implement its workflow.

**CN:** 定义函数 `topk_backward(x, y_indx, dy_vals, k, n_rows, apply_softmax)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `triton.next_power_of_2`, `torch.empty_like`, `_topk_backward[dy_vals.shape[0],]`, `y_indx.stride`, `dy_vals.stride` 来实现其工作流程.

### Block 35 — Lines 77-77 (topk_backward)
```python
77|     assert dy_vals.shape[-1] == k
```
**EN:** Asserts `dy_vals.shape[-1] == k` to enforce invariants.

**CN:** 断言 `dy_vals.shape[-1] == k` 以确保不变量成立。

### Block 36 — Lines 78-78 (topk_backward)
```python
78|     n_expts_pad = triton.next_power_of_2(x.shape[-1])
```
**EN:** Assigns `n_expts_pad` and calls `triton.next_power_of_2`.

**CN:** 将 `n_expts_pad`，并调用 `triton.next_power_of_2`.

### Block 37 — Lines 79-79 (topk_backward)
```python
79|     dx = torch.empty_like(x)
```
**EN:** Assigns `dx` and calls `torch.empty_like`.

**CN:** 将 `dx`，并调用 `torch.empty_like`.

### Block 38 — Lines 80-84 (topk_backward)
```python
80|     _topk_backward[(dy_vals.shape[0], )](
81|         y_indx, y_indx.stride(0), dy_vals, dy_vals.stride(0), x, x.stride(0),  # inputs
82|         dx,  # outputs
83|         dx.stride(0), x.shape[0], n_rows, x.shape[-1], APPLY_SOFTMAX=apply_softmax, N_EXPTS_ACT=k,
84|         N_EXPTS_PAD=n_expts_pad)
```
**EN:** Calls `_topk_backward[dy_vals.shape[0],]` for side effects, registration, or validation.

**CN:** 调用 `_topk_backward[dy_vals.shape[0],]` ，用于副作用、注册或校验。

### Block 39 — Lines 85-87 (topk_backward)
```python
85|     return dx
86| 
87| 
```
**EN:** Returns `dx`.

**CN:** 返回 `dx`.

### Block 40 — Lines 88-89 (TopK)
```python
88| class TopK(torch.autograd.Function):
89| 
```
**EN:** Defines class `TopK` inheriting from `torch.autograd.Function` to organize related behavior. Main methods are `forward`, `backward`.

**CN:** 定义类 `TopK`，继承自 `torch.autograd.Function`，用于组织相关行为。主要方法有 `forward`, `backward`.

### Block 41 — Lines 90-91 (forward)
```python
90|     @staticmethod
91|     def forward(ctx, x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool):
```
**EN:** Defines function `forward(ctx, x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)` with decorators `staticmethod` for this module. The body mainly prepares intermediate values; invokes `ctx.save_for_backward`; prepares intermediate values. It uses calls such as `topk_forward`, `ctx.save_for_backward` to implement its workflow.

**CN:** 定义函数 `forward(ctx, x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)`，带有装饰器 `staticmethod`，供本模块使用. 主体主要准备中间值; invokes `ctx.save_for_backward`; 准备中间值. 其中会调用 `topk_forward`, `ctx.save_for_backward` 来实现其工作流程.

### Block 42 — Lines 92-92 (forward)
```python
92|         y_vals, y_indx, bitmatrix = topk_forward(x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)
```
**EN:** Assigns `y_vals`, `y_indx`, `bitmatrix` and calls `topk_forward`.

**CN:** 将 `y_vals`, `y_indx`, `bitmatrix`，并调用 `topk_forward`.

### Block 43 — Lines 93-93 (forward)
```python
93|         ctx.save_for_backward(x, y_indx)
```
**EN:** Calls `ctx.save_for_backward` for side effects, registration, or validation.

**CN:** 调用 `ctx.save_for_backward` ，用于副作用、注册或校验。

### Block 44 — Lines 94-94 (forward)
```python
94|         ctx.apply_softmax = apply_softmax
```
**EN:** Assigns `ctx.apply_softmax` and references `apply_softmax`.

**CN:** 将 `ctx.apply_softmax` and 引用 `apply_softmax`.

### Block 45 — Lines 95-95 (forward)
```python
95|         ctx.k = k
```
**EN:** Assigns `ctx.k` and references `k`.

**CN:** 将 `ctx.k` and 引用 `k`.

### Block 46 — Lines 96-96 (forward)
```python
96|         ctx.n_rows = n_rows
```
**EN:** Assigns `ctx.n_rows` and references `n_rows`.

**CN:** 将 `ctx.n_rows` and 引用 `n_rows`.

### Block 47 — Lines 97-98 (forward)
```python
97|         return y_vals, y_indx, bitmatrix
98| 
```
**EN:** Returns `(y_vals, y_indx, bitmatrix)`.

**CN:** 返回 `(y_vals, y_indx, bitmatrix)`.

### Block 48 — Lines 99-100 (backward)
```python
 99|     @staticmethod
100|     def backward(ctx, dy_vals, _0, _1):
```
**EN:** Defines function `backward(ctx, dy_vals, _0, _1)` with decorators `staticmethod` for this module. The body mainly prepares intermediate values; prepares intermediate values; returns the computed result. It uses calls such as `topk_backward` to implement its workflow.

**CN:** 定义函数 `backward(ctx, dy_vals, _0, _1)`，带有装饰器 `staticmethod`，供本模块使用. 主体主要准备中间值; 准备中间值; 返回计算结果. 其中会调用 `topk_backward` 来实现其工作流程.

### Block 49 — Lines 101-101 (backward)
```python
101|         x, y_indx = ctx.saved_tensors
```
**EN:** Assigns `x`, `y_indx` and references `ctx.saved_tensors`.

**CN:** 将 `x`, `y_indx` and 引用 `ctx.saved_tensors`.

### Block 50 — Lines 102-102 (backward)
```python
102|         dx = topk_backward(x, y_indx, dy_vals, ctx.k, ctx.n_rows, ctx.apply_softmax)
```
**EN:** Assigns `dx` and calls `topk_backward`.

**CN:** 将 `dx`，并调用 `topk_backward`.

### Block 51 — Lines 103-105 (backward)
```python
103|         return dx, None, None, None, None, None, None, None
104| 
105| 
```
**EN:** Returns `(dx, None, None, None, None, None, None, None)`.

**CN:** 返回 `(dx, None, None, None, None, None, None, None)`.

### Block 52 — Lines 106-115 (topk)
```python
106| def topk(
107|     x: Union[Tensor, torch.Tensor],
108|     k: int,
109|     apply_softmax: bool = True,
110|     dim: int = 1,
111|     y_indx: Optional[torch.Tensor] = None,
112|     n_rows: Optional[int] = None,
113|     all_gather: bool = False,
114|     symm_mem_pool: SymmetricMemoryPool | None = None,
115| ):
```
**EN:** Defines function `topk(x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `TopK.apply`, `SparseMatrix` to implement its workflow.

**CN:** 定义函数 `topk(x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `TopK.apply`, `SparseMatrix` 来实现其工作流程.

### Block 53 — Lines 116-139 (topk)
```python
116|     """
117|     Computes the top-k values and indices along a specified dimension of a tensor.
118|     Note that the input can be either a `Tensor` or a `torch.Tensor`, but the output will always be a `torch.Tensor`.
119| 
120|     Parameters
121|     ----------
122|     x : Union[triton_kernels.Tensor, torch.Tensor]
123|         Input tensor of shape (n_tokens, n_expts).
124|     k : int
125|         Number of top elements to retrieve.
126|     apply_softmax : bool, default True
127|         Whether to apply softmax to the input tensor before computing top-k.
128|     dim : int, default 1
129|         Dimension along which to compute top-k.
130|     y_indx : torch.Tensor, optional
131|         Pre-allocated tensor for storing indices of top-k elements with shape (n_tokens, k).
132|         If provided, we skip the computation of top-k indices and use this tensor instead.
133|     n_rows : int, optional
134|         Number of rows to apply top-k on. If None, we consider all rows in `x`.
135| 
136|     Returns
137|     -------
138|     SparseMatrix: sparse matrix equal to `x` with non-selected entries set to 0
139|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 54 — Lines 140-140 (topk)
```python
140|     y_vals, y_indx, bitmatrix = TopK.apply(x, k, apply_softmax, dim, y_indx, n_rows, all_gather, symm_mem_pool)
```
**EN:** Assigns `y_vals`, `y_indx`, `bitmatrix` and calls `TopK.apply`.

**CN:** 将 `y_vals`, `y_indx`, `bitmatrix`，并调用 `TopK.apply`.

### Block 55 — Lines 141-143 (topk)
```python
141|     return SparseMatrix(vals=y_vals, indx=y_indx, mask=bitmatrix)
142| 
143| 
```
**EN:** Returns `SparseMatrix(vals=y_vals, indx=y_indx, mask=bitmatrix)`.

**CN:** 返回 `SparseMatrix(vals=y_vals, indx=y_indx, mask=bitmatrix)`.

### Block 56 — Lines 144-151 (topk_torch)
```python
144| def topk_torch(
145|     x,
146|     k,
147|     apply_softmax: bool = True,
148|     dim: int = 1,
149|     y_indx: Optional[torch.Tensor] = None,
150|     n_rows: Optional[int] = None,
151| ) -> SparseMatrix:
```
**EN:** Defines function `topk_torch(x, k, apply_softmax, dim, y_indx, n_rows)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `y_indx.long`, `torch.take_along_dim`, `torch.cat`, `y_indx.int`, `torch.zeros` to implement its workflow.

**CN:** 定义函数 `topk_torch(x, k, apply_softmax, dim, y_indx, n_rows)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `y_indx.long`, `torch.take_along_dim`, `torch.cat`, `y_indx.int`, `torch.zeros` 来实现其工作流程.

### Block 57 — Lines 152-153 (topk_torch)
```python
152|     if n_rows is None:
153|         n_rows = x.shape[0]
```
**EN:** Checks `n_rows is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `n_rows is None`. 真分支主要准备中间值.

### Block 58 — Lines 154-154 (topk_torch)
```python
154|     has_user_provided_indx = y_indx is not None
```
**EN:** Assigns `has_user_provided_indx` and evaluates `y_indx is not None`.

**CN:** 将 `has_user_provided_indx` and 计算 `y_indx is not None`.

### Block 59 — Lines 155-155 (topk_torch)
```python
155|     cdiv = lambda a, b: (a + b - 1) // b
```
**EN:** Assigns `cdiv` and defines a lambda.

**CN:** 将 `cdiv` and 定义一个 lambda.

### Block 60 — Lines 156-156 (topk_torch)
```python
156|     device = x.device
```
**EN:** Assigns `device` and references `x.device`.

**CN:** 将 `device` and 引用 `x.device`.

### Block 61 — Lines 157-157 (topk_torch)
```python
157|     assert dim == 1
```
**EN:** Asserts `dim == 1` to enforce invariants.

**CN:** 断言 `dim == 1` 以确保不变量成立。

### Block 62 — Lines 158-158 (topk_torch)
```python
158|     assert not isinstance(x, Tensor)
```
**EN:** Asserts `not isinstance(x, Tensor)` to enforce invariants.

**CN:** 断言 `not isinstance(x, Tensor)` 以确保不变量成立。

### Block 63 — Lines 159-160 (topk_torch)
```python
159|     if not has_user_provided_indx:
160|         y_indx = torch.argsort(-x, dim=1, stable=True)[:, :k]
```
**EN:** Checks `not has_user_provided_indx`. The true branch mainly prepares intermediate values.

**CN:** 检查 `not has_user_provided_indx`. 真分支主要准备中间值.

### Block 64 — Lines 161-161 (topk_torch)
```python
161|     y_indx = y_indx.long()
```
**EN:** Assigns `y_indx` and calls `y_indx.long`.

**CN:** 将 `y_indx`，并调用 `y_indx.long`.

### Block 65 — Lines 162-162 (topk_torch)
```python
162|     y_vals = torch.take_along_dim(x[:n_rows, :], y_indx[:n_rows, :], dim=1)
```
**EN:** Assigns `y_vals` and calls `torch.take_along_dim`.

**CN:** 将 `y_vals`，并调用 `torch.take_along_dim`.

### Block 66 — Lines 163-163 (topk_torch)
```python
163|     y_vals = torch.cat([y_vals, x[n_rows:, :k]], dim=0)
```
**EN:** Assigns `y_vals` and calls `torch.cat`.

**CN:** 将 `y_vals`，并调用 `torch.cat`.

### Block 67 — Lines 164-165 (topk_torch)
```python
164|     y_indx = y_indx.int()
165|     # compute bitmatrix
```
**EN:** Assigns `y_indx` and calls `y_indx.int`.

**CN:** 将 `y_indx`，并调用 `y_indx.int`.

### Block 68 — Lines 166-166 (topk_torch)
```python
166|     _, n_cols = x.shape
```
**EN:** Assigns `_`, `n_cols` and references `x.shape`.

**CN:** 将 `_`, `n_cols` and 引用 `x.shape`.

### Block 69 — Lines 167-167 (topk_torch)
```python
167|     bitmatrix_data = torch.zeros((cdiv(n_cols, 32), cdiv(x.shape[0], 32) * 32), dtype=torch.int32, device=device)
```
**EN:** Assigns `bitmatrix_data` and calls `torch.zeros`.

**CN:** 将 `bitmatrix_data`，并调用 `torch.zeros`.

### Block 70 — Lines 168-169 (topk_torch)
```python
168|     bitmatrix_data = torch.transpose(bitmatrix_data, 0, 1)[:x.shape[0]]
169|     # fill bitmatrix
```
**EN:** Assigns `bitmatrix_data` and evaluates `torch.transpose(bitmatrix_data, 0, 1)[:x.shape[0]]`.

**CN:** 将 `bitmatrix_data` and 计算 `torch.transpose(bitmatrix_data, 0, 1)[:x.shape[0]]`.

### Block 71 — Lines 170-171 (topk_torch)
```python
170|     if apply_softmax:
171|         y_vals = torch.softmax(y_vals.float(), dim=-1).to(x.dtype)
```
**EN:** Checks `apply_softmax`. The true branch mainly prepares intermediate values.

**CN:** 检查 `apply_softmax`. 真分支主要准备中间值.

### Block 72 — Lines 172-174 (topk_torch)
```python
172|     if not has_user_provided_indx:
173|         y_vals, sort_indices = torch.sort(y_vals.float(), dim=1, descending=True, stable=True)
174|         y_indx = torch.gather(y_indx, 1, sort_indices)
```
**EN:** Checks `not has_user_provided_indx`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `not has_user_provided_indx`. 真分支主要准备中间值; 准备中间值.

### Block 73 — Lines 175-175 (topk_torch)
```python
175|     y_indx[n_rows:, :] = -1
```
**EN:** Assigns `y_indx[n_rows:, :]` and evaluates `-1`.

**CN:** 将 `y_indx[n_rows:, :]` and 计算 `-1`.

### Block 74 — Lines 176-176 (topk_torch)
```python
176|     rows = torch.arange(x.shape[0], device=device).unsqueeze(1).expand(-1, y_indx.shape[1]).reshape(-1)
```
**EN:** Assigns `rows` and calls `torch.arange(x.shape[0], device=device).unsquee...`.

**CN:** 将 `rows`，并调用 `torch.arange(x.shape[0], device=device).unsquee...`.

### Block 75 — Lines 177-177 (topk_torch)
```python
177|     cols = y_indx.reshape(-1)  # 64-bit safe for div/mod
```
**EN:** Assigns `cols` and calls `y_indx.reshape`.

**CN:** 将 `cols`，并调用 `y_indx.reshape`.

### Block 76 — Lines 178-178 (topk_torch)
```python
178|     word_idx = torch.div(cols, 32, rounding_mode='floor')
```
**EN:** Assigns `word_idx` and calls `torch.div`.

**CN:** 将 `word_idx`，并调用 `torch.div`.

### Block 77 — Lines 179-179 (topk_torch)
```python
179|     bit_idx = cols % 32
```
**EN:** Assigns `bit_idx` and evaluates `cols % 32`.

**CN:** 将 `bit_idx` and 计算 `cols % 32`.

### Block 78 — Lines 180-180 (topk_torch)
```python
180|     masks = torch.ones_like(bit_idx) << bit_idx
```
**EN:** Assigns `masks` and evaluates `torch.ones_like(bit_idx) << bit_idx`.

**CN:** 将 `masks` and 计算 `torch.ones_like(bit_idx) << bit_idx`.

### Block 79 — Lines 181-181 (topk_torch)
```python
181|     bitmatrix_data.index_put_((rows, word_idx), masks, accumulate=True)
```
**EN:** Calls `bitmatrix_data.index_put_` for side effects, registration, or validation.

**CN:** 调用 `bitmatrix_data.index_put_` ，用于副作用、注册或校验。

### Block 80 — Lines 182-183 (topk_torch)
```python
182|     bitmatrix_data = bitmatrix_data.view(torch.uint32)
183| 
```
**EN:** Assigns `bitmatrix_data` and calls `bitmatrix_data.view`.

**CN:** 将 `bitmatrix_data`，并调用 `bitmatrix_data.view`.

### Block 81 — Lines 184-184 (topk_torch)
```python
184|     bitmatrix = wrap_torch_tensor(bitmatrix_data, dtype=BIT, shape=x.shape)
```
**EN:** Assigns `bitmatrix` and calls `wrap_torch_tensor`.

**CN:** 将 `bitmatrix`，并调用 `wrap_torch_tensor`.

### Block 82 — Lines 185-185 (topk_torch)
```python
185|     return SparseMatrix(vals=y_vals, indx=y_indx, mask=bitmatrix)
```
**EN:** Returns `SparseMatrix(vals=y_vals, indx=y_indx, mask=bitmatrix)`.

**CN:** 返回 `SparseMatrix(vals=y_vals, indx=y_indx, mask=bitmatrix)`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `make_empty`, `topk_forward`, `topk_backward`, `TopK`, `topk`, `topk_torch`.
  **CN:** 主要符号：`make_empty`, `topk_forward`, `topk_backward`, `TopK`, `topk`, `topk_torch`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** Distributed execution concepts such as meshes, ranks, or shard mapping appear in this file.
  **CN:** 该文件涉及 mesh、rank 或分片映射等分布式执行概念。
- **EN:** Top-k selection and, where relevant, gradient propagation are key operations here.
  **CN:** 这里的关键操作是 Top-k 选择以及相关的梯度传播。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`, `triton`, `typing (Optional, Union)`.
  **CN:** 外部模块：`torch`, `triton`, `typing (Optional, Union)`。
- **EN:** Internal modules: `triton_kernels.topk_details._topk_forward (_topk_forward)`, `triton_kernels.topk_details._topk_backward (_topk_backward)`, `triton_kernels.tensor (SparseMatrix, Tensor)`, `triton_kernels.tensor_details.dtype (BIT)`, `triton_kernels.distributed (SymmetricMemoryPool)`, `triton_kernels.tensor (wrap_torch_tensor, dtype_to_torch_dtype)`.
  **CN:** 内部模块：`triton_kernels.topk_details._topk_forward (_topk_forward)`, `triton_kernels.topk_details._topk_backward (_topk_backward)`, `triton_kernels.tensor (SparseMatrix, Tensor)`, `triton_kernels.tensor_details.dtype (BIT)`, `triton_kernels.distributed (SymmetricMemoryPool)`, `triton_kernels.tensor (wrap_torch_tensor, dtype_to_torch_dtype)`。
