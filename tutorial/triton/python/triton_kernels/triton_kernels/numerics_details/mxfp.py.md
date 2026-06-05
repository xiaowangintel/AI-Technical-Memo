# mxfp.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/numerics_details/mxfp.py`
- **Purpose / 用途:** Implementation module for mxfp; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols DequantScaleRoundingMode, downcast_to_mxfp, upcast_from_mxfp, right_shift_unsigned, get_max_quant_val. / 用于 mxfp 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 DequantScaleRoundingMode、downcast_to_mxfp、upcast_from_mxfp、right_shift_unsigned、get_max_quant_val。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-2 (module)
```python
1| # isort: off
2| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 3-17 (module)
```python
 3| from enum import Enum
 4| import math
 5| import triton
 6| import torch
 7| import torch.nn.functional as F
 8| from .mxfp_details._upcast_from_mxfp import _upcast_from_mxfp
 9| from .mxfp_details._downcast_to_mxfp import _downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE, _quantize_mxfp8_fn, _quantize_mxfp4_fn, _quantize_nvfp4_fn
10| from triton.tools.tensor_descriptor import TensorDescriptor
11| from triton_kernels.tensor import Tensor, wrap_torch_tensor, empty
12| from triton_kernels.tensor_details.layout import StridedLayout
13| from triton_kernels.tensor_details.dtype import FP4, FP8_E4M3FN, FP8_E5M2, UINT8
14| # -----------------------------------------------------------------------------
15| #                      Dequantization / Quantization Utilities
16| # -----------------------------------------------------------------------------
17| 
```
**EN:** This block imports `enum (Enum)`, `math`, `triton`, `torch`, `torch.nn.functional`, `.mxfp_details._upcast_from_mxfp (_upcast_from_mxfp)`, `.mxfp_details._downcast_to_mxfp (_downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE, _quantize_mxfp8_fn)`, `triton.tools.tensor_descriptor (TensorDescriptor)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `enum (Enum)`, `math`, `triton`, `torch`, `torch.nn.functional`, `.mxfp_details._upcast_from_mxfp (_upcast_from_mxfp)`, `.mxfp_details._downcast_to_mxfp (_downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE, _quantize_mxfp8_fn)`, `triton.tools.tensor_descriptor (TensorDescriptor)` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 18-19 (DequantScaleRoundingMode)
```python
18| class DequantScaleRoundingMode(Enum):
19|     # 2^round_up(log2(max/max_q)) avoids clipping the max value
```
**EN:** Defines class `DequantScaleRoundingMode` inheriting from `Enum` to organize related behavior. Key fields include `ROUND_UP`, `ROUND_DOWN`.

**CN:** 定义类 `DequantScaleRoundingMode`，继承自 `Enum`，用于组织相关行为。关键字段包括 `ROUND_UP`, `ROUND_DOWN`.

### Block 4 — Lines 20-22 (DequantScaleRoundingMode)
```python
20|     ROUND_UP = 0
21|     # 2^round_down(log2(max/max_power_of_2_q)) follows the OCP standard ~50% of
22|     # chance of clipping the max value.
```
**EN:** Assigns `ROUND_UP` and stores constant `0`.

**CN:** 将 `ROUND_UP` and 保存常量 `0`.

### Block 5 — Lines 23-24 (DequantScaleRoundingMode)
```python
23|     ROUND_DOWN = 1
24| 
```
**EN:** Assigns `ROUND_DOWN` and stores constant `1`.

**CN:** 将 `ROUND_DOWN` and 保存常量 `1`.

### Block 6 — Lines 25-28 (downcast_to_mxfp)
```python
25| def downcast_to_mxfp(x: torch.Tensor, out_dtype: torch.dtype, axis: int,
26|                      scale_dtype: torch.dtype = torch.uint8,
27|                      microblock_size: int = MXFP_BLOCK_SIZE.value,
28|                      DEQUANT_SCALE_ROUNDING_MODE: DequantScaleRoundingMode = DequantScaleRoundingMode.ROUND_UP):
```
**EN:** Defines function `downcast_to_mxfp(x, out_dtype, axis, scale_dtype, microblock_size, DEQUANT_SCALE_ROUNDING_MODE)` for this module. The body mainly branches on runtime conditions; checks invariants; branches on runtime conditions. It uses calls such as `isinstance`, `StridedLayout`, `empty`, `wrap_torch_tensor`, `triton.cdiv` to implement its workflow.

**CN:** 定义函数 `downcast_to_mxfp(x, out_dtype, axis, scale_dtype, microblock_size, DEQUANT_SCALE_ROUNDING_MODE)`，供本模块使用. 主体主要根据运行时条件分支; 检查不变量; 根据运行时条件分支. 其中会调用 `isinstance`, `StridedLayout`, `empty`, `wrap_torch_tensor`, `triton.cdiv` 来实现其工作流程.

### Block 7 — Lines 29-37 (downcast_to_mxfp)
```python
29|     """
30|          Convert the src weights to mx format. The src weight is quantized along the axis dimension.
31| 
32|          If weight_quant_type is torch.uint8, we output mxfp4/NVFP4 where two e2m1 values are packed into a single byte.
33|          Note that this means the k_dim of the tensor will be half of the logical k_dim.
34| 
35|          If weight_quant_type is torch.float8_e4m3fn or torch.float8_e5m2, we output mxfp8 with the float8s are stored
36|          in their respective formats.
37|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 8 — Lines 38-39 (downcast_to_mxfp)
```python
38|     if not isinstance(x, Tensor):
39|         x = wrap_torch_tensor(x)
```
**EN:** Checks `not isinstance(x, Tensor)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `not isinstance(x, Tensor)`. 真分支主要准备中间值.

### Block 9 — Lines 40-40 (downcast_to_mxfp)
```python
40|     assert scale_dtype == torch.uint8 or scale_dtype == torch.float8_e4m3fn, f"Invalid scale dtype {scale_dtype=}"
```
**EN:** Asserts `scale_dtype == torch.uint8 or scale_dtype == torch.float8_e4m3fn` to enforce invariants.

**CN:** 断言 `scale_dtype == torch.uint8 or scale_dtype == torch.float8_e4m3fn` 以确保不变量成立。

### Block 10 — Lines 41-46 (downcast_to_mxfp)
```python
41|     if isinstance(out_dtype, torch.dtype):
42|         out_dtype = {
43|             torch.uint8: FP4,
44|             torch.float8_e4m3fn: FP8_E4M3FN,
45|             torch.float8_e5m2: FP8_E5M2,
46|         }[out_dtype]
```
**EN:** Checks `isinstance(out_dtype, torch.dtype)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `isinstance(out_dtype, torch.dtype)`. 真分支主要准备中间值.

### Block 11 — Lines 47-47 (downcast_to_mxfp)
```python
47|     assert x.shape[axis] % microblock_size == 0, f"axis dim must be divisible by {microblock_size}. Got {x.shape[axis]}"
```
**EN:** Asserts `x.shape[axis] % microblock_size == 0` to enforce invariants.

**CN:** 断言 `x.shape[axis] % microblock_size == 0` 以确保不变量成立。

### Block 12 — Lines 48-48 (downcast_to_mxfp)
```python
48|     assert isinstance(x.storage.layout, StridedLayout), "input data must be strided"
```
**EN:** Asserts `isinstance(x.storage.layout, StridedLayout)` to enforce invariants.

**CN:** 断言 `isinstance(x.storage.layout, StridedLayout)` 以确保不变量成立。

### Block 13 — Lines 49-49 (downcast_to_mxfp)
```python
49|     assert -x.ndim <= axis < x.ndim, f"Invalid axis {axis=}"
```
**EN:** Asserts `-x.ndim <= axis < x.ndim` to enforce invariants.

**CN:** 断言 `-x.ndim <= axis < x.ndim` 以确保不变量成立。

### Block 14 — Lines 50-50 (downcast_to_mxfp)
```python
50|     assert out_dtype in (FP4, FP8_E4M3FN, FP8_E5M2), f"Invalid output dtype {out_dtype=}"
```
**EN:** Asserts `out_dtype in (FP4, FP8_E4M3FN, FP8_E5M2)` to enforce invariants.

**CN:** 断言 `out_dtype in (FP4, FP8_E4M3FN, FP8_E5M2)` 以确保不变量成立。

### Block 15 — Lines 51-55 (downcast_to_mxfp)
```python
51|     if scale_dtype == torch.float8_e4m3fn:
52|         assert out_dtype == FP4, f"Direct float8 scales are only supported for FP4 values. Got {out_dtype=}"
53|         assert DEQUANT_SCALE_ROUNDING_MODE == DequantScaleRoundingMode.ROUND_UP, \
54|             "Direct float8 scales only support ROUND_UP in downcast_to_mxfp"
55|     # handle negative `axis``
```
**EN:** Checks `scale_dtype == torch.float8_e4m3fn`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `scale_dtype == torch.float8_e4m3fn`. 真分支主要检查不变量; 检查不变量.

### Block 16 — Lines 56-57 (downcast_to_mxfp)
```python
56|     axis = axis if axis >= 0 else axis + x.ndim
57|     # downcast
```
**EN:** Assigns `axis` and uses conditional expression `axis if axis >= 0 else axis + x.ndim`.

**CN:** 将 `axis` and 使用条件表达式 `axis if axis >= 0 else axis + x.ndim`.

### Block 17 — Lines 58-60 (downcast_to_mxfp)
```python
58|     L = x.shape[axis]
59|     # Ensure last dimension is a multiple of the microblock size. This is expected by the kernel.
60|     # output value storage
```
**EN:** Assigns `L` and evaluates `x.shape[axis]`.

**CN:** 将 `L` and 计算 `x.shape[axis]`.

### Block 18 — Lines 61-61 (downcast_to_mxfp)
```python
61|     y_layout = StridedLayout(major_dim=axis - x.ndim)
```
**EN:** Assigns `y_layout` and calls `StridedLayout`.

**CN:** 将 `y_layout`，并调用 `StridedLayout`.

### Block 19 — Lines 62-62 (downcast_to_mxfp)
```python
62|     y_scale_shape = (*x.shape[:axis], triton.cdiv(L, microblock_size), *x.shape[axis+1:])
```
**EN:** Assigns `y_scale_shape` and builds a tuple.

**CN:** 将 `y_scale_shape` and 构造一个元组.

### Block 20 — Lines 63-63 (downcast_to_mxfp)
```python
63|     y_value = empty(x.shape, out_dtype, x.device, y_layout)
```
**EN:** Assigns `y_value` and calls `empty`.

**CN:** 将 `y_value`，并调用 `empty`.

### Block 21 — Lines 64-64 (downcast_to_mxfp)
```python
64|     y_scale = empty(y_scale_shape, UINT8 if scale_dtype == torch.uint8 else FP8_E4M3FN, x.device, y_layout)
```
**EN:** Assigns `y_scale` and calls `empty`.

**CN:** 将 `y_scale`，并调用 `empty`.

### Block 22 — Lines 65-87 (downcast_to_mxfp)
```python
65|     if x.numel() > 0:
66|         # canonicalize to a 2D tensor that paxks 4-bit values on its inner-most dimension
67|         x_storage = x.storage.data.transpose(axis, -1).reshape(-1, x.shape[axis])
68|         y_storage_value = y_value.storage.data.transpose(axis, -1).view(-1, y_value.storage.data.shape[axis])
69|         y_storage_scale = y_scale.storage.data.transpose(axis, -1).view(-1, y_scale.storage.data.shape[axis])
70|         # performance hyper-parameters
71|         BLOCK_OUT_DIM = 32
72|         BLOCK_QUANT_DIM = microblock_size * 4
73|         NUM_WARPS = 4 if x.dtype == torch.float32 else 8
74|         # launch kernel
75|         blocks_out_dim = triton.cdiv(x_storage.shape[0], BLOCK_OUT_DIM)
76|         blocks_quant_dim = triton.cdiv(x_storage.shape[1], BLOCK_QUANT_DIM)
77|         _downcast_to_mxfp[(blocks_out_dim, blocks_quant_dim)](
78|             y_storage_value, *y_storage_value.stride(),
79|             y_storage_scale, *y_storage_scale.stride(),
80|             x_storage, *x_storage.stride(), *x_storage.shape,
81|             BLOCK_OUT_DIM,
82|             BLOCK_QUANT_DIM,
83|             microblock_size,
84|             DEQUANT_SCALE_ROUNDING_MODE.value,
85|             num_warps=NUM_WARPS,
86|         )
87|     # TODO: return tensor object instead of its storage
```
**EN:** Checks `x.numel() > 0`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `x.numel() > 0`. 真分支主要准备中间值; 准备中间值.

### Block 23 — Lines 88-89 (downcast_to_mxfp)
```python
88|     return y_value.storage.data, y_scale.storage.data
89| 
```
**EN:** Returns `(y_value.storage.data, y_scale.storage.data)`.

**CN:** 返回 `(y_value.storage.data, y_scale.storage.data)`.

### Block 24 — Lines 90-90 (upcast_from_mxfp)
```python
90| def upcast_from_mxfp(tensor: torch.Tensor, scale: torch.Tensor, target_dtype: torch.dtype, axis: int):
```
**EN:** Defines function `upcast_from_mxfp(tensor, scale, target_dtype, axis)` for this module. The body mainly prepares intermediate values; checks invariants; prepares intermediate values. It uses calls such as `tensor.transpose(axis, tensor.ndim - ...`, `scale.transpose(axis, scale.ndim - 1)...`, `out.transpose(axis, scale.ndim - 1).c...`, `tensor.numel`, `tensor.view` to implement its workflow.

**CN:** 定义函数 `upcast_from_mxfp(tensor, scale, target_dtype, axis)`，供本模块使用. 主体主要准备中间值; 检查不变量; 准备中间值. 其中会调用 `tensor.transpose(axis, tensor.ndim - ...`, `scale.transpose(axis, scale.ndim - 1)...`, `out.transpose(axis, scale.ndim - 1).c...`, `tensor.numel`, `tensor.view` 来实现其工作流程.

### Block 25 — Lines 91-97 (upcast_from_mxfp)
```python
91|     """
92|     Upcasts an mxfp (packed) weight tensor back to float16 or bfloat16.
93| 
94|     The function assumes that the tensors were quantized along the given axis.
95|     It permutes the tensor so that the quantized axis is last, reshapes to 2D,
96|     launches the Triton upcast kernel, and then unpermutes back to the original order.
97|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 26 — Lines 98-98 (upcast_from_mxfp)
```python
98|     ndim = tensor.ndim
```
**EN:** Assigns `ndim` and references `tensor.ndim`.

**CN:** 将 `ndim` and 引用 `tensor.ndim`.

### Block 27 — Lines 99-99 (upcast_from_mxfp)
```python
99|     assert -ndim <= axis < ndim, f"Invalid axis {axis=}"
```
**EN:** Asserts `-ndim <= axis < ndim` to enforce invariants.

**CN:** 断言 `-ndim <= axis < ndim` 以确保不变量成立。

### Block 28 — Lines 100-100 (upcast_from_mxfp)
```python
100|     axis = axis if axis >= 0 else axis + ndim
```
**EN:** Assigns `axis` and uses conditional expression `axis if axis >= 0 else axis + ndim`.

**CN:** 将 `axis` and 使用条件表达式 `axis if axis >= 0 else axis + ndim`.

### Block 29 — Lines 101-103 (upcast_from_mxfp)
```python
101|     assert tensor.ndim == scale.ndim, (f"Weight and scale must have the same number of dimensions. "
102|                                        f"Got {tensor.ndim=} and {scale.ndim=}")
103|     # dtype checks
```
**EN:** Asserts `tensor.ndim == scale.ndim` to enforce invariants.

**CN:** 断言 `tensor.ndim == scale.ndim` 以确保不变量成立。

### Block 30 — Lines 104-105 (upcast_from_mxfp)
```python
104|     assert tensor.dtype in {torch.uint8, torch.float8_e5m2, torch.float8_e4m3fn}, \
105|         f"Invalid tensor dtype {tensor.dtype=}"
```
**EN:** Asserts `tensor.dtype in {torch.uint8, torch.float8_e5m2, torch.float8_e4m3fn}` to enforce invariants.

**CN:** 断言 `tensor.dtype in {torch.uint8, torch.float8_e5m2, torch.float8_e4m3fn}` 以确保不变量成立。

### Block 31 — Lines 106-107 (upcast_from_mxfp)
```python
106|     assert scale.dtype == torch.uint8 or scale.dtype == torch.float8_e4m3fn, \
107|         f"Invalid scale dtype {scale.dtype=}"
```
**EN:** Asserts `scale.dtype == torch.uint8 or scale.dtype == torch.float8_e4m3fn` to enforce invariants.

**CN:** 断言 `scale.dtype == torch.uint8 or scale.dtype == torch.float8_e4m3fn` 以确保不变量成立。

### Block 32 — Lines 108-109 (upcast_from_mxfp)
```python
108|     assert target_dtype in (torch.float16, torch.bfloat16, torch.float32), f"Invalid output dtype {target_dtype=}"
109|     # upcast
```
**EN:** Asserts `target_dtype in (torch.float16, torch.bfloat16, torch.float32)` to enforce invariants.

**CN:** 断言 `target_dtype in (torch.float16, torch.bfloat16, torch.float32)` 以确保不变量成立。

### Block 33 — Lines 110-110 (upcast_from_mxfp)
```python
110|     pack_multiple = 2 if tensor.dtype == torch.uint8 else 1
```
**EN:** Assigns `pack_multiple` and uses conditional expression `2 if tensor.dtype == torch.uint8 else 1`.

**CN:** 将 `pack_multiple` and 使用条件表达式 `2 if tensor.dtype == torch.uint8 else 1`.

### Block 34 — Lines 111-111 (upcast_from_mxfp)
```python
111|     logical_quant_dim = tensor.shape[axis] * pack_multiple
```
**EN:** Assigns `logical_quant_dim` and evaluates `tensor.shape[axis] * pack_multiple`.

**CN:** 将 `logical_quant_dim` and 计算 `tensor.shape[axis] * pack_multiple`.

### Block 35 — Lines 112-112 (upcast_from_mxfp)
```python
112|     tensor = tensor.transpose(axis, tensor.ndim - 1).contiguous()
```
**EN:** Assigns `tensor` and calls `tensor.transpose(axis, tensor.ndim - 1).contiguous`.

**CN:** 将 `tensor`，并调用 `tensor.transpose(axis, tensor.ndim - 1).contiguous`.

### Block 36 — Lines 113-113 (upcast_from_mxfp)
```python
113|     scale = scale.transpose(axis, scale.ndim - 1).contiguous()
```
**EN:** Assigns `scale` and calls `scale.transpose(axis, scale.ndim - 1).contiguous`.

**CN:** 将 `scale`，并调用 `scale.transpose(axis, scale.ndim - 1).contiguous`.

### Block 37 — Lines 114-115 (upcast_from_mxfp)
```python
114|     original_out_shape = tensor.shape[:-1] + (logical_quant_dim, )
115| 
```
**EN:** Assigns `original_out_shape` and evaluates `tensor.shape[:-1] + (logical_quant_dim,)`.

**CN:** 将 `original_out_shape` and 计算 `tensor.shape[:-1] + (logical_quant_dim,)`.

### Block 38 — Lines 116-161 (upcast_from_mxfp)
```python
116|     if tensor.numel() > 0:
117|         reshaped_tensor = tensor.view(-1, tensor.shape[-1])
118|         reshaped_scale = scale.view(-1, scale.shape[-1])
119| 
120|         # Pad the tensor and output if needed for tensor descriptor spec requirements.
121|         TENSOR_DESC_PAD_REQ = 16
122|         needs_padding = reshaped_tensor.shape[-1] % TENSOR_DESC_PAD_REQ != 0
123|         if needs_padding:
124|             tensor_pad_amount = TENSOR_DESC_PAD_REQ - (reshaped_tensor.shape[-1] % TENSOR_DESC_PAD_REQ)
125|             reshaped_tensor = F.pad(reshaped_tensor, (0, tensor_pad_amount), "constant", 0)
126|             pad_elems_count = tensor_pad_amount * pack_multiple
127|             out_shape = original_out_shape[:-1] + (original_out_shape[-1] + pad_elems_count, )
128|         else:
129|             out_shape = original_out_shape
130|         out = torch.empty(out_shape, dtype=target_dtype, device=tensor.device)
131|         reshaped_out = out.view(-1, out.shape[-1])
132| 
133|         is_fp4 = reshaped_tensor.dtype == torch.uint8
134|         scale_block_size = MXFP_BLOCK_SIZE.value if scale.dtype == torch.uint8 else NVFP_BLOCK_SIZE.value
135| 
136|         # performance hyper-parameters
137|         BLOCK_OUT_DIM = 64
138|         BLOCK_QUANT_DIM = scale_block_size * 4
139|         NUM_WARPS = 4
140| 
141|         blocks_out_dim = triton.cdiv(reshaped_out.shape[0], BLOCK_OUT_DIM)
142|         blocks_quant_dim = triton.cdiv(reshaped_out.shape[1], BLOCK_QUANT_DIM)
143|         k_divisor = 2 if is_fp4 else 1
144|         block_size_quant_mx_tensor = BLOCK_QUANT_DIM // k_divisor
145|         out_desc = TensorDescriptor.from_tensor(reshaped_out, [BLOCK_OUT_DIM, BLOCK_QUANT_DIM])
146|         tensor_desc = TensorDescriptor.from_tensor(reshaped_tensor, [BLOCK_OUT_DIM, block_size_quant_mx_tensor])
147|         _upcast_from_mxfp[(blocks_out_dim, blocks_quant_dim)](
148|             out_desc,
149|             tensor_desc,
150|             reshaped_scale,
151|             *reshaped_scale.stride(),
152|             *reshaped_out.shape,
153|             BLOCK_OUT_DIM,
154|             BLOCK_QUANT_DIM,
155|             scale_block_size,
156|             num_warps=NUM_WARPS,
157|         )
158|         if needs_padding:
159|             out = out[..., :original_out_shape[-1]]
160|     else:
161|         out = torch.empty(original_out_shape, dtype=target_dtype, device=tensor.device)
```
**EN:** Checks `tensor.numel() > 0`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `tensor.numel() > 0`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 39 — Lines 162-162 (upcast_from_mxfp)
```python
162|     out = out.transpose(axis, scale.ndim - 1).contiguous()
```
**EN:** Assigns `out` and calls `out.transpose(axis, scale.ndim - 1).contiguous`.

**CN:** 将 `out`，并调用 `out.transpose(axis, scale.ndim - 1).contiguous`.

### Block 40 — Lines 163-168 (upcast_from_mxfp)
```python
163|     return out
164| 
165| 
166| # ------------
167| 
168| 
```
**EN:** Returns `out`.

**CN:** 返回 `out`.

### Block 41 — Lines 169-170 (right_shift_unsigned)
```python
169| def right_shift_unsigned(x, shift):
170|     # CUDA torch does not support bit ops on uint32, so we need to mask to get unsigned right shift
```
**EN:** Defines function `right_shift_unsigned(x, shift)` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `right_shift_unsigned(x, shift)`，供本模块使用. 主体主要返回计算结果.

### Block 42 — Lines 171-173 (right_shift_unsigned)
```python
171|     return (x >> shift) & ((1 << (32 - shift)) - 1)
172| 
173| 
```
**EN:** Returns `x >> shift & (1 << 32 - shift) - 1`.

**CN:** 返回 `x >> shift & (1 << 32 - shift) - 1`.

### Block 43 — Lines 174-174 (get_max_quant_val)
```python
174| def get_max_quant_val(dtype: torch.dtype):
```
**EN:** Defines function `get_max_quant_val(dtype)` for this module. The body mainly prepares intermediate values; checks invariants; returns the computed result.

**CN:** 定义函数 `get_max_quant_val(dtype)`，供本模块使用. 主体主要准备中间值; 检查不变量; 返回计算结果.

### Block 44 — Lines 175-175 (get_max_quant_val)
```python
175|     d = {torch.uint8: 6.0, torch.float8_e5m2: 57344.0, torch.float8_e4m3fn: 448.0}
```
**EN:** Assigns `d` and builds a dictionary.

**CN:** 将 `d` and 构造一个字典.

### Block 45 — Lines 176-176 (get_max_quant_val)
```python
176|     assert dtype in d
```
**EN:** Asserts `dtype in d` to enforce invariants.

**CN:** 断言 `dtype in d` 以确保不变量成立。

### Block 46 — Lines 177-179 (get_max_quant_val)
```python
177|     return d[dtype]
178| 
179| 
```
**EN:** Returns `d[dtype]`.

**CN:** 返回 `d[dtype]`.

### Block 47 — Lines 180-184 (downcast_to_mxfp_torch)
```python
180| def downcast_to_mxfp_torch(src_tensor: torch.Tensor, out_quant_type: torch.dtype, axis: int,
181|                            scale_dtype: torch.dtype = torch.uint8,
182|                            microblock_size: int = MXFP_BLOCK_SIZE.value,
183|                            expected_scale: torch.Tensor | None = None,
184|                            DEQUANT_SCALE_ROUNDING_MODE: DequantScaleRoundingMode = DequantScaleRoundingMode.ROUND_UP):
```
**EN:** Defines function `downcast_to_mxfp_torch(src_tensor, out_quant_type, axis, scale_dtype, microblock_size, expected_scale, DEQUANT_SCALE_ROUNDING_MODE)` for this module. The body mainly prepares intermediate values; checks invariants; checks invariants. It uses calls such as `src_tensor.transpose(axis, src_tensor...`, `F.pad`, `padded_src.size`, `torch.abs`, `torch.where` to implement its workflow.

**CN:** 定义函数 `downcast_to_mxfp_torch(src_tensor, out_quant_type, axis, scale_dtype, microblock_size, expected_scale, DEQUANT_SCALE_ROUNDING_MODE)`，供本模块使用. 主体主要准备中间值; 检查不变量; 检查不变量. 其中会调用 `src_tensor.transpose(axis, src_tensor...`, `F.pad`, `padded_src.size`, `torch.abs`, `torch.where` 来实现其工作流程.

### Block 48 — Lines 185-199 (downcast_to_mxfp_torch)
```python
185|     """
186|     Converts the src tensor to the output format specified by out_quant_type.
187|       axis: The axis along which the tensors are contiguous and quantization is applied.
188|       DEQUANT_SCALE_ROUNDING_MODE: 0 for ROUND_UP, 1 for ROUND_DOWN.
189| 
190|     Returns:
191|       out_quant_tensor: Quantized tensor in mx format.
192|          • For mxfp8, the output has the same shape as src_tensor.
193|          • For FP4, the size along the axis is halved, and the tensor is returned as a torch.uint8.
194|       scale: Scale tensor computed per microblock along the axis.
195| 
196|     If expected_scale is provided, apply the same preconditioning as float_to_flex
197|     before microscaled output quantization by dividing the source tensor by it.
198|     """
199|     # This should probably be packed into its own tiny class
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 49 — Lines 200-200 (downcast_to_mxfp_torch)
```python
200|     ndim = src_tensor.ndim
```
**EN:** Assigns `ndim` and references `src_tensor.ndim`.

**CN:** 将 `ndim` and 引用 `src_tensor.ndim`.

### Block 50 — Lines 201-201 (downcast_to_mxfp_torch)
```python
201|     assert -ndim <= axis < ndim, f"Invalid axis {axis=}"
```
**EN:** Asserts `-ndim <= axis < ndim` to enforce invariants.

**CN:** 断言 `-ndim <= axis < ndim` 以确保不变量成立。

### Block 51 — Lines 202-204 (downcast_to_mxfp_torch)
```python
202|     assert src_tensor.dtype in {torch.float32, torch.bfloat16,
203|                                 torch.float16}, f"Invalid input tensor dtype {src_tensor.dtype}"
204| 
```
**EN:** Asserts `src_tensor.dtype in {torch.float32, torch.bfloat16, torch.float16}` to enforce invariants.

**CN:** 断言 `src_tensor.dtype in {torch.float32, torch.bfloat16, torch.float16}` 以确保不变量成立。

### Block 52 — Lines 205-205 (downcast_to_mxfp_torch)
```python
205|     axis = axis if axis >= 0 else axis + ndim
```
**EN:** Assigns `axis` and uses conditional expression `axis if axis >= 0 else axis + ndim`.

**CN:** 将 `axis` and 使用条件表达式 `axis if axis >= 0 else axis + ndim`.

### Block 53 — Lines 206-206 (downcast_to_mxfp_torch)
```python
206|     is_fp4 = out_quant_type == torch.uint8
```
**EN:** Assigns `is_fp4` and evaluates `out_quant_type == torch.uint8`.

**CN:** 将 `is_fp4` and 计算 `out_quant_type == torch.uint8`.

### Block 54 — Lines 207-207 (downcast_to_mxfp_torch)
```python
207|     is_fp8 = "float8" in str(out_quant_type)
```
**EN:** Assigns `is_fp8` and evaluates `'float8' in str(out_quant_type)`.

**CN:** 将 `is_fp8` and 计算 `'float8' in str(out_quant_type)`.

### Block 55 — Lines 208-208 (downcast_to_mxfp_torch)
```python
208|     assert is_fp4 or is_fp8, f"Invalid input tensor dtype {out_quant_type}"
```
**EN:** Asserts `is_fp4 or is_fp8` to enforce invariants.

**CN:** 断言 `is_fp4 or is_fp8` 以确保不变量成立。

### Block 56 — Lines 209-209 (downcast_to_mxfp_torch)
```python
209|     assert scale_dtype == torch.uint8 or scale_dtype == torch.float8_e4m3fn, f"Invalid scale dtype {scale_dtype=}"
```
**EN:** Asserts `scale_dtype == torch.uint8 or scale_dtype == torch.float8_e4m3fn` to enforce invariants.

**CN:** 断言 `scale_dtype == torch.uint8 or scale_dtype == torch.float8_e4m3fn` 以确保不变量成立。

### Block 57 — Lines 210-214 (downcast_to_mxfp_torch)
```python
210|     if scale_dtype == torch.float8_e4m3fn:
211|         assert is_fp4, f"Direct float8 scales are only supported for FP4 values. Got {out_quant_type=}"
212|         assert DEQUANT_SCALE_ROUNDING_MODE == DequantScaleRoundingMode.ROUND_UP, \
213|             "Direct float8 scales only support ROUND_UP in downcast_to_mxfp_torch"
214| 
```
**EN:** Checks `scale_dtype == torch.float8_e4m3fn`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `scale_dtype == torch.float8_e4m3fn`. 真分支主要检查不变量; 检查不变量.

### Block 58 — Lines 215-217 (downcast_to_mxfp_torch)
```python
215|     device = src_tensor.device
216| 
217|     # For mxfp4 conversion, we assume the contiguous axis length is even.
```
**EN:** Assigns `device` and references `src_tensor.device`.

**CN:** 将 `device` and 引用 `src_tensor.device`.

### Block 59 — Lines 218-222 (downcast_to_mxfp_torch)
```python
218|     if is_fp4:
219|         axis_shape = src_tensor.size(axis)
220|         assert axis_shape % 2 == 0, "For mxfp4 conversion the contiguous axis length must be even."
221| 
222|     # Permute the tensor so that the contiguous axis becomes the last dimension.
```
**EN:** Checks `is_fp4`. The true branch mainly prepares intermediate values; checks invariants.

**CN:** 检查 `is_fp4`. 真分支主要准备中间值; 检查不变量.

### Block 60 — Lines 223-223 (downcast_to_mxfp_torch)
```python
223|     src = src_tensor.transpose(axis, src_tensor.ndim - 1).to(torch.float32)
```
**EN:** Assigns `src` and calls `src_tensor.transpose(axis, src_tensor.ndim - 1).to`.

**CN:** 将 `src`，并调用 `src_tensor.transpose(axis, src_tensor.ndim - 1).to`.

### Block 61 — Lines 224-229 (downcast_to_mxfp_torch)
```python
224|     if expected_scale is not None:
225|         if expected_scale.numel() == 1:
226|             src = src / expected_scale.to(device=device, dtype=src.dtype)
227|         else:
228|             assert src.ndim == 3
229|             src = src / expected_scale.to(device=device, dtype=src.dtype)[:, None, None]
```
**EN:** Checks `expected_scale is not None`. The true branch mainly branches on runtime conditions.

**CN:** 检查 `expected_scale is not None`. 真分支主要根据运行时条件分支.

### Block 62 — Lines 230-232 (downcast_to_mxfp_torch)
```python
230|     axis_shape = src.shape[-1]
231| 
232|     # Pad the axis to be divisible by the microblock size, in case it is not.
```
**EN:** Assigns `axis_shape` and evaluates `src.shape[-1]`.

**CN:** 将 `axis_shape` and 计算 `src.shape[-1]`.

### Block 63 — Lines 233-233 (downcast_to_mxfp_torch)
```python
233|     next_multiple = triton.cdiv(axis_shape, microblock_size) * microblock_size
```
**EN:** Assigns `next_multiple` and evaluates `triton.cdiv(axis_shape, microblock_size) * microblock_size`.

**CN:** 将 `next_multiple` and 计算 `triton.cdiv(axis_shape, microblock_size) * microblock_size`.

### Block 64 — Lines 234-234 (downcast_to_mxfp_torch)
```python
234|     pad_amount = next_multiple - axis_shape
```
**EN:** Assigns `pad_amount` and evaluates `next_multiple - axis_shape`.

**CN:** 将 `pad_amount` and 计算 `next_multiple - axis_shape`.

### Block 65 — Lines 235-235 (downcast_to_mxfp_torch)
```python
235|     padded_src = F.pad(src, (0, pad_amount))
```
**EN:** Assigns `padded_src` and calls `F.pad`.

**CN:** 将 `padded_src`，并调用 `F.pad`.

### Block 66 — Lines 236-236 (downcast_to_mxfp_torch)
```python
236|     valid_mask = F.pad(torch.ones_like(src, dtype=torch.bool), (0, pad_amount))
```
**EN:** Assigns `valid_mask` and calls `F.pad`.

**CN:** 将 `valid_mask`，并调用 `F.pad`.

### Block 67 — Lines 237-240 (downcast_to_mxfp_torch)
```python
237|     padded_axis_shape = padded_src.size(-1)
238| 
239|     # --- Compute per-group maximums for scale ---
240|     # Set padded entries to -1 so they don’t affect the max.
```
**EN:** Assigns `padded_axis_shape` and calls `padded_src.size`.

**CN:** 将 `padded_axis_shape`，并调用 `padded_src.size`.

### Block 68 — Lines 241-241 (downcast_to_mxfp_torch)
```python
241|     abs_f = torch.abs(padded_src)
```
**EN:** Assigns `abs_f` and calls `torch.abs`.

**CN:** 将 `abs_f`，并调用 `torch.abs`.

### Block 69 — Lines 242-243 (downcast_to_mxfp_torch)
```python
242|     abs_f = torch.where(valid_mask, abs_f, torch.tensor(-1.0, device=device, dtype=padded_src.dtype))
243|     # Reshape the last dimension into microblocks.
```
**EN:** Assigns `abs_f` and calls `torch.where`.

**CN:** 将 `abs_f`，并调用 `torch.where`.

### Block 70 — Lines 244-244 (downcast_to_mxfp_torch)
```python
244|     new_shape = padded_src.shape[:-1] + (padded_axis_shape // microblock_size, microblock_size)
```
**EN:** Assigns `new_shape` and evaluates `padded_src.shape[:-1] + (padded_axis_shape // microblock_size, micr...`.

**CN:** 将 `new_shape` and 计算 `padded_src.shape[:-1] + (padded_axis_shape // microblock_size, micr...`.

### Block 71 — Lines 245-246 (downcast_to_mxfp_torch)
```python
245|     abs_groups = abs_f.view(*new_shape)
246|     # Compute maximum along the microblock dimension.
```
**EN:** Assigns `abs_groups` and calls `abs_f.view`.

**CN:** 将 `abs_groups`，并调用 `abs_f.view`.

### Block 72 — Lines 247-249 (downcast_to_mxfp_torch)
```python
247|     max_val, _ = abs_groups.max(dim=-1, keepdim=True)
248| 
249|     # Choose a max quantization value depending on type.
```
**EN:** Assigns `max_val`, `_` and calls `abs_groups.max`.

**CN:** 将 `max_val`, `_`，并调用 `abs_groups.max`.

### Block 73 — Lines 250-250 (downcast_to_mxfp_torch)
```python
250|     max_quant_val = get_max_quant_val(out_quant_type)
```
**EN:** Assigns `max_quant_val` and calls `get_max_quant_val`.

**CN:** 将 `max_quant_val`，并调用 `get_max_quant_val`.

### Block 74 — Lines 251-254 (downcast_to_mxfp_torch)
```python
251|     if DEQUANT_SCALE_ROUNDING_MODE == DequantScaleRoundingMode.ROUND_UP:
252|         dequant_scale = max_val / max_quant_val
253|     else:
254|         dequant_scale = max_val / (2 ** math.floor(math.log2(max_quant_val)))
```
**EN:** Checks `DEQUANT_SCALE_ROUNDING_MODE == DequantScaleRoundingMode.ROUND_UP`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `DEQUANT_SCALE_ROUNDING_MODE == DequantScaleRoundingMode.ROUND_UP`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 75 — Lines 255-269 (downcast_to_mxfp_torch)
```python
255|     if scale_dtype == torch.uint8:
256|         # Convert to int to round the FP32 scale, prior to quantization!
257|         ds_int = dequant_scale.view(torch.int32)
258|         if DEQUANT_SCALE_ROUNDING_MODE == DequantScaleRoundingMode.ROUND_UP:
259|             ds_int_rounded = (ds_int + 0x007FFFFF) & 0x7F800000
260|         else:
261|             ds_int_rounded = ds_int & 0x7F800000
262|         dequant_scale_rounded = ds_int_rounded.view(torch.float32)
263|     else:
264|         # Direct fp8 scales keep the existing plain cast semantics here: the
265|         # stored scale is rounded by the fp8 conversion rather than forced
266|         # upward despite the ROUND_UP mode name.
267|         dequant_scale_rounded = dequant_scale.to(scale_dtype).to(torch.float32)
268| 
269|     # Compute the quantization scale.
```
**EN:** Checks `scale_dtype == torch.uint8`. The true branch mainly prepares intermediate values; branches on runtime conditions, while the else branch prepares intermediate values.

**CN:** 检查 `scale_dtype == torch.uint8`. 真分支主要准备中间值; 根据运行时条件分支；而 else 分支准备中间值.

### Block 76 — Lines 270-272 (downcast_to_mxfp_torch)
```python
270|     quant_scale = torch.where(dequant_scale_rounded == 0, torch.tensor(0.0, device=device), 1.0 / dequant_scale_rounded)
271| 
272|     # Quantize the tensor
```
**EN:** Assigns `quant_scale` and calls `torch.where`.

**CN:** 将 `quant_scale`，并调用 `torch.where`.

### Block 77 — Lines 273-273 (downcast_to_mxfp_torch)
```python
273|     orig_padded_shape = padded_src.shape
```
**EN:** Assigns `orig_padded_shape` and references `padded_src.shape`.

**CN:** 将 `orig_padded_shape` and 引用 `padded_src.shape`.

### Block 78 — Lines 274-274 (downcast_to_mxfp_torch)
```python
274|     padded_src_groups = padded_src.view(*new_shape)
```
**EN:** Assigns `padded_src_groups` and calls `padded_src.view`.

**CN:** 将 `padded_src_groups`，并调用 `padded_src.view`.

### Block 79 — Lines 275-276 (downcast_to_mxfp_torch)
```python
275|     quant_tensor = padded_src_groups * quant_scale
276|     # Reshape back to the original shape and trim padding
```
**EN:** Assigns `quant_tensor` and evaluates `padded_src_groups * quant_scale`.

**CN:** 将 `quant_tensor` and 计算 `padded_src_groups * quant_scale`.

### Block 80 — Lines 277-277 (downcast_to_mxfp_torch)
```python
277|     quant_tensor = quant_tensor.view(orig_padded_shape)
```
**EN:** Assigns `quant_tensor` and calls `quant_tensor.view`.

**CN:** 将 `quant_tensor`，并调用 `quant_tensor.view`.

### Block 81 — Lines 278-280 (downcast_to_mxfp_torch)
```python
278|     quant_tensor = quant_tensor[..., :axis_shape]
279| 
280|     # Finally, convert the quantized tensor to the target format
```
**EN:** Assigns `quant_tensor` and evaluates `quant_tensor[..., :axis_shape]`.

**CN:** 将 `quant_tensor` and 计算 `quant_tensor[..., :axis_shape]`.

### Block 82 — Lines 281-323 (downcast_to_mxfp_torch)
```python
281|     if is_fp8:
282|         # Conversion must use satfinite PTX, so clamp before the conversion in torch to emulate this behavior
283|         quant_tensor = torch.clamp(quant_tensor, -max_quant_val, max_quant_val)
284|         out_weight = quant_tensor.to(out_quant_type)
285|     else:
286|         assert is_fp4, f"Invalid output quantization type {out_quant_type}"
287|         # For mxfp4, perform bit-level manipulation and pack two 4-bit values per uint8.
288|         # First, reinterpret the quantized tensor bits.
289|         q_int = quant_tensor.contiguous().view(torch.int32)
290|         # Extract sign, exponent, and mantissa.
291|         signs = q_int & 0x80000000
292|         exponents = right_shift_unsigned(q_int, 23) & 0xFF
293|         mantissas_orig = q_int & 0x7FFFFF
294| 
295|         E8_BIAS = 127
296|         E2_BIAS = 1
297|         # Adjust mantissas for subnormals.
298|         is_subnormal = exponents < E8_BIAS
299|         shift = E8_BIAS - exponents - 1
300|         mantissas_pre = (0x400000 | right_shift_unsigned(mantissas_orig, 1))
301|         bit0_dropped = (mantissas_orig & 0x1) != 0
302|         mask = (1 << shift.clamp(max=31)) - 1
303|         dropped_post = (mantissas_pre & mask) != 0
304|         sticky = is_subnormal & (bit0_dropped | dropped_post)
305|         mantissas = torch.where(is_subnormal, mantissas_pre >> shift, mantissas_orig)
306|         exponents = torch.maximum(exponents, torch.tensor(E8_BIAS - E2_BIAS, device=device)) - (E8_BIAS - E2_BIAS)
307|         # Round to nearest, ties to even (RTNE)
308|         m2bits = right_shift_unsigned(mantissas, 21) & 0x3
309|         lsb_keep = right_shift_unsigned(m2bits, 1) & 0x1
310|         guard = m2bits & 0x1
311|         sticky |= (mantissas & ((1 << 21) - 1)) != 0
312|         round_inc = guard & (sticky.to(torch.int32) | lsb_keep)
313|         e2m1_tmp = right_shift_unsigned(((exponents << 2) | m2bits) + round_inc, 1)
314|         e2m1_tmp = torch.minimum(e2m1_tmp, torch.tensor(0x7, device=device))
315|         e2m1_value = (right_shift_unsigned(signs, 28) | e2m1_tmp).to(torch.uint8)  # shape: (..., even_axis_shape)
316| 
317|         # Pack pairs of 4-bit values along the last dimension.
318|         e2m1_value = e2m1_value.view(*e2m1_value.shape[:-1], axis_shape // 2, 2)
319|         evens = e2m1_value[..., 0]
320|         odds = e2m1_value[..., 1]
321|         out_weight = evens | (odds << 4)  # shape: (..., axis_shape//2)
322| 
323|     # --- Process and output the scale ---
```
**EN:** Checks `is_fp8`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch checks invariants; prepares intermediate values.

**CN:** 检查 `is_fp8`. 真分支主要准备中间值; 准备中间值；而 else 分支检查不变量; 准备中间值.

### Block 83 — Lines 324-328 (downcast_to_mxfp_torch)
```python
324|     if scale_dtype == torch.uint8:
325|         dq_scale = (ds_int_rounded.view(*dequant_scale.shape) >> 23).to(torch.uint8)
326|         dq_scale = dq_scale.squeeze(-1)
327|     else:
328|         dq_scale = dequant_scale_rounded.to(scale_dtype).squeeze(-1)
```
**EN:** Checks `scale_dtype == torch.uint8`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `scale_dtype == torch.uint8`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 84 — Lines 329-329 (downcast_to_mxfp_torch)
```python
329|     out_weight = out_weight.transpose(axis, src_tensor.ndim - 1)
```
**EN:** Assigns `out_weight` and calls `out_weight.transpose`.

**CN:** 将 `out_weight`，并调用 `out_weight.transpose`.

### Block 85 — Lines 330-330 (downcast_to_mxfp_torch)
```python
330|     dq_scale = dq_scale.transpose(axis, src_tensor.ndim - 1)
```
**EN:** Assigns `dq_scale` and calls `dq_scale.transpose`.

**CN:** 将 `dq_scale`，并调用 `dq_scale.transpose`.

### Block 86 — Lines 331-333 (downcast_to_mxfp_torch)
```python
331|     return out_weight, dq_scale
332| 
333| 
```
**EN:** Returns `(out_weight, dq_scale)`.

**CN:** 返回 `(out_weight, dq_scale)`.

### Block 87 — Lines 334-334 (cvt_e2m1_to_fp32)
```python
334| def cvt_e2m1_to_fp32(input_tensor):
```
**EN:** Defines function `cvt_e2m1_to_fp32(input_tensor)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `input_tensor.to`, `torch.tensor`, `torch.cat`, `torch.stack`, `output_tensor.view` to implement its workflow.

**CN:** 定义函数 `cvt_e2m1_to_fp32(input_tensor)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `input_tensor.to`, `torch.tensor`, `torch.cat`, `torch.stack`, `output_tensor.view` 来实现其工作流程.

### Block 88 — Lines 335-336 (cvt_e2m1_to_fp32)
```python
335|     assert input_tensor.dtype == torch.uint8
336| 
```
**EN:** Asserts `input_tensor.dtype == torch.uint8` to enforce invariants.

**CN:** 断言 `input_tensor.dtype == torch.uint8` 以确保不变量成立。

### Block 89 — Lines 337-337 (cvt_e2m1_to_fp32)
```python
337|     input_tensor = input_tensor.to(torch.int32)
```
**EN:** Assigns `input_tensor` and calls `input_tensor.to`.

**CN:** 将 `input_tensor`，并调用 `input_tensor.to`.

### Block 90 — Lines 338-338 (cvt_e2m1_to_fp32)
```python
338|     evens = input_tensor & 0xF
```
**EN:** Assigns `evens` and evaluates `input_tensor & 15`.

**CN:** 将 `evens` and 计算 `input_tensor & 15`.

### Block 91 — Lines 339-340 (cvt_e2m1_to_fp32)
```python
339|     odds = (input_tensor >> 4) & 0xF
340| 
```
**EN:** Assigns `odds` and evaluates `input_tensor >> 4 & 15`.

**CN:** 将 `odds` and 计算 `input_tensor >> 4 & 15`.

### Block 92 — Lines 341-341 (cvt_e2m1_to_fp32)
```python
341|     vals = [0.0, 0.5, 1, 1.5, 2, 3, 4, 6]
```
**EN:** Assigns `vals` and builds a list.

**CN:** 将 `vals` and 构造一个列表.

### Block 93 — Lines 342-342 (cvt_e2m1_to_fp32)
```python
342|     outputs = torch.tensor(vals, dtype=torch.float32, device=input_tensor.device)
```
**EN:** Assigns `outputs` and calls `torch.tensor`.

**CN:** 将 `outputs`，并调用 `torch.tensor`.

### Block 94 — Lines 343-344 (cvt_e2m1_to_fp32)
```python
343|     outputs = torch.cat([outputs, -outputs])
344| 
```
**EN:** Assigns `outputs` and calls `torch.cat`.

**CN:** 将 `outputs`，并调用 `torch.cat`.

### Block 95 — Lines 345-345 (cvt_e2m1_to_fp32)
```python
345|     even_floats = outputs[evens]
```
**EN:** Assigns `even_floats` and evaluates `outputs[evens]`.

**CN:** 将 `even_floats` and 计算 `outputs[evens]`.

### Block 96 — Lines 346-346 (cvt_e2m1_to_fp32)
```python
346|     odd_floats = outputs[odds]
```
**EN:** Assigns `odd_floats` and evaluates `outputs[odds]`.

**CN:** 将 `odd_floats` and 计算 `outputs[odds]`.

### Block 97 — Lines 347-347 (cvt_e2m1_to_fp32)
```python
347|     output_tensor = torch.stack([even_floats, odd_floats], dim=-1)
```
**EN:** Assigns `output_tensor` and calls `torch.stack`.

**CN:** 将 `output_tensor`，并调用 `torch.stack`.

### Block 98 — Lines 348-348 (cvt_e2m1_to_fp32)
```python
348|     output_tensor = output_tensor.view(*input_tensor.shape[:-1], input_tensor.shape[-1] * 2)
```
**EN:** Assigns `output_tensor` and calls `output_tensor.view`.

**CN:** 将 `output_tensor`，并调用 `output_tensor.view`.

### Block 99 — Lines 349-351 (cvt_e2m1_to_fp32)
```python
349|     return output_tensor
350| 
351| 
```
**EN:** Returns `output_tensor`.

**CN:** 返回 `output_tensor`.

### Block 100 — Lines 352-352 (upcast_from_mxfp_torch)
```python
352| def upcast_from_mxfp_torch(tensor: torch.Tensor, scale: torch.Tensor, target_dtype: torch.dtype, axis: int):
```
**EN:** Defines function `upcast_from_mxfp_torch(tensor, scale, target_dtype, axis)` for this module. The body mainly prepares intermediate values; checks invariants; prepares intermediate values. It uses calls such as `scale.transpose`, `tensor.transpose`, `fp32_tensor.size`, `F.pad`, `padded_tensor.view` to implement its workflow.

**CN:** 定义函数 `upcast_from_mxfp_torch(tensor, scale, target_dtype, axis)`，供本模块使用. 主体主要准备中间值; 检查不变量; 准备中间值. 其中会调用 `scale.transpose`, `tensor.transpose`, `fp32_tensor.size`, `F.pad`, `padded_tensor.view` 来实现其工作流程.

### Block 101 — Lines 353-360 (upcast_from_mxfp_torch)
```python
353|     """
354|     Converts a microscaled tensor to the target format specified by target_dtype.
355|       axis: The axis along which dequantization is applied.
356| 
357|     Returns:
358|       out_weight: Tensor in the target format.
359|     """
360| 
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 102 — Lines 361-361 (upcast_from_mxfp_torch)
```python
361|     ndim = tensor.ndim
```
**EN:** Assigns `ndim` and references `tensor.ndim`.

**CN:** 将 `ndim` and 引用 `tensor.ndim`.

### Block 103 — Lines 362-362 (upcast_from_mxfp_torch)
```python
362|     assert -ndim <= axis < ndim, f"Invalid axis {axis=}"
```
**EN:** Asserts `-ndim <= axis < ndim` to enforce invariants.

**CN:** 断言 `-ndim <= axis < ndim` 以确保不变量成立。

### Block 104 — Lines 363-363 (upcast_from_mxfp_torch)
```python
363|     is_fp8 = tensor.dtype == torch.float8_e4m3fn or tensor.dtype == torch.float8_e5m2
```
**EN:** Assigns `is_fp8` and evaluates `tensor.dtype == torch.float8_e4m3fn or tensor.dtype == torch.float8...`.

**CN:** 将 `is_fp8` and 计算 `tensor.dtype == torch.float8_e4m3fn or tensor.dtype == torch.float8...`.

### Block 105 — Lines 364-364 (upcast_from_mxfp_torch)
```python
364|     assert is_fp8 or tensor.dtype == torch.uint8, f"Invalid input quantization type {tensor.dtype}"
```
**EN:** Asserts `is_fp8 or tensor.dtype == torch.uint8` to enforce invariants.

**CN:** 断言 `is_fp8 or tensor.dtype == torch.uint8` 以确保不变量成立。

### Block 106 — Lines 365-368 (upcast_from_mxfp_torch)
```python
365|     assert scale.dtype == torch.uint8 or scale.dtype == torch.float8_e4m3fn, \
366|         f"Invalid scale dtype {scale.dtype}"
367| 
368|     # Permute the tensor and scale so that the quantization axis becomes the last dimension
```
**EN:** Asserts `scale.dtype == torch.uint8 or scale.dtype == torch.float8_e4m3fn` to enforce invariants.

**CN:** 断言 `scale.dtype == torch.uint8 or scale.dtype == torch.float8_e4m3fn` 以确保不变量成立。

### Block 107 — Lines 369-369 (upcast_from_mxfp_torch)
```python
369|     axis = axis if axis >= 0 else axis + ndim
```
**EN:** Assigns `axis` and uses conditional expression `axis if axis >= 0 else axis + ndim`.

**CN:** 将 `axis` and 使用条件表达式 `axis if axis >= 0 else axis + ndim`.

### Block 108 — Lines 370-370 (upcast_from_mxfp_torch)
```python
370|     scale = scale.transpose(axis, scale.ndim - 1)
```
**EN:** Assigns `scale` and calls `scale.transpose`.

**CN:** 将 `scale`，并调用 `scale.transpose`.

### Block 109 — Lines 371-372 (upcast_from_mxfp_torch)
```python
371|     tensor = tensor.transpose(axis, tensor.ndim - 1)
372| 
```
**EN:** Assigns `tensor` and calls `tensor.transpose`.

**CN:** 将 `tensor`，并调用 `tensor.transpose`.

### Block 110 — Lines 373-377 (upcast_from_mxfp_torch)
```python
373|     if tensor.dtype == torch.uint8:
374|         fp32_tensor = cvt_e2m1_to_fp32(tensor)
375|     else:
376|         fp32_tensor = tensor.to(torch.float32)
377| 
```
**EN:** Checks `tensor.dtype == torch.uint8`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `tensor.dtype == torch.uint8`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 111 — Lines 378-378 (upcast_from_mxfp_torch)
```python
378|     logical_quant_dim = tensor.shape[-1] * (2 if tensor.dtype == torch.uint8 else 1)
```
**EN:** Assigns `logical_quant_dim` and evaluates `tensor.shape[-1] * (2 if tensor.dtype == torch.uint8 else 1)`.

**CN:** 将 `logical_quant_dim` and 计算 `tensor.shape[-1] * (2 if tensor.dtype == torch.uint8 else 1)`.

### Block 112 — Lines 379-385 (upcast_from_mxfp_torch)
```python
379|     if scale.dtype == torch.uint8:
380|         dq_scale = (scale.to(torch.int32) << 23).view(torch.float32)
381|         scale_block_size = MXFP_BLOCK_SIZE.value
382|     else:
383|         dq_scale = scale.to(torch.float32)
384|         scale_block_size = logical_quant_dim // scale.shape[-1]
385|         assert scale_block_size in (NVFP_BLOCK_SIZE.value, MXFP_BLOCK_SIZE.value), f"Unsupported direct scale block size {scale_block_size}"
```
**EN:** Checks `scale.dtype == torch.uint8`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `scale.dtype == torch.uint8`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 113 — Lines 386-386 (upcast_from_mxfp_torch)
```python
386|     axis_shape = fp32_tensor.size(-1)
```
**EN:** Assigns `axis_shape` and calls `fp32_tensor.size`.

**CN:** 将 `axis_shape`，并调用 `fp32_tensor.size`.

### Block 114 — Lines 387-387 (upcast_from_mxfp_torch)
```python
387|     padded_axis_shape = triton.cdiv(logical_quant_dim, scale_block_size) * scale_block_size
```
**EN:** Assigns `padded_axis_shape` and evaluates `triton.cdiv(logical_quant_dim, scale_block_size) * scale_block_size`.

**CN:** 将 `padded_axis_shape` and 计算 `triton.cdiv(logical_quant_dim, scale_block_size) * scale_block_size`.

### Block 115 — Lines 388-388 (upcast_from_mxfp_torch)
```python
388|     pad_size = padded_axis_shape - axis_shape
```
**EN:** Assigns `pad_size` and evaluates `padded_axis_shape - axis_shape`.

**CN:** 将 `pad_size` and 计算 `padded_axis_shape - axis_shape`.

### Block 116 — Lines 389-390 (upcast_from_mxfp_torch)
```python
389|     padded_tensor = F.pad(fp32_tensor, (0, pad_size))
390| 
```
**EN:** Assigns `padded_tensor` and calls `F.pad`.

**CN:** 将 `padded_tensor`，并调用 `F.pad`.

### Block 117 — Lines 391-391 (upcast_from_mxfp_torch)
```python
391|     new_axis_shape = padded_tensor.shape[-1]
```
**EN:** Assigns `new_axis_shape` and evaluates `padded_tensor.shape[-1]`.

**CN:** 将 `new_axis_shape` and 计算 `padded_tensor.shape[-1]`.

### Block 118 — Lines 392-392 (upcast_from_mxfp_torch)
```python
392|     new_shape = padded_tensor.shape[:-1] + (new_axis_shape // scale_block_size, scale_block_size)
```
**EN:** Assigns `new_shape` and evaluates `padded_tensor.shape[:-1] + (new_axis_shape // scale_block_size, sca...`.

**CN:** 将 `new_shape` and 计算 `padded_tensor.shape[:-1] + (new_axis_shape // scale_block_size, sca...`.

### Block 119 — Lines 393-393 (upcast_from_mxfp_torch)
```python
393|     padded_tensor = padded_tensor.view(*new_shape)
```
**EN:** Assigns `padded_tensor` and calls `padded_tensor.view`.

**CN:** 将 `padded_tensor`，并调用 `padded_tensor.view`.

### Block 120 — Lines 394-394 (upcast_from_mxfp_torch)
```python
394|     dq_scale_padded = dq_scale.unsqueeze(-1)  # shape: [..., ceil(axis_shape/32), 1]
```
**EN:** Assigns `dq_scale_padded` and calls `dq_scale.unsqueeze`.

**CN:** 将 `dq_scale_padded`，并调用 `dq_scale.unsqueeze`.

### Block 121 — Lines 395-401 (upcast_from_mxfp_torch)
```python
395|     out_padded = padded_tensor * dq_scale_padded
396|     # Need to clamp since due to rounding, we can have overflow that was within
397|     # the range before quantization.
398|     # e.g., 3.3895e+38 -> log2(3.3895e+38 / max_fp8e4m3=448) ~= 119.17 -> round
399|     # up to 120 + exp_bias=127 -> scale=247
400|     # 3.3895e+38 / 2**120 ~= 254.9976 -> round to 256 in fp8e4m3fn
401|     # Dequantization: 256 * 2**120 > 3.4e38 overflowing 3.38953139e38
```
**EN:** Assigns `out_padded` and evaluates `padded_tensor * dq_scale_padded`.

**CN:** 将 `out_padded` and 计算 `padded_tensor * dq_scale_padded`.

### Block 122 — Lines 402-402 (upcast_from_mxfp_torch)
```python
402|     finfo = torch.finfo(target_dtype)
```
**EN:** Assigns `finfo` and calls `torch.finfo`.

**CN:** 将 `finfo`，并调用 `torch.finfo`.

### Block 123 — Lines 403-403 (upcast_from_mxfp_torch)
```python
403|     out_padded = (padded_tensor * dq_scale_padded).clamp(finfo.min, finfo.max)
```
**EN:** Assigns `out_padded` and calls `(padded_tensor * dq_scale_padded).clamp`.

**CN:** 将 `out_padded`，并调用 `(padded_tensor * dq_scale_padded).clamp`.

### Block 124 — Lines 404-408 (upcast_from_mxfp_torch)
```python
404|     if tensor.dtype == torch.float8_e5m2:
405|         # fp8e5m2 can have inf and we want to preserve so separately handle
406|         out_padded = out_padded.where(~padded_tensor.isinf(), padded_tensor.to(target_dtype))
407| 
408|     # Flatten back and remove the padded tail
```
**EN:** Checks `tensor.dtype == torch.float8_e5m2`. The true branch mainly prepares intermediate values.

**CN:** 检查 `tensor.dtype == torch.float8_e5m2`. 真分支主要准备中间值.

### Block 125 — Lines 409-409 (upcast_from_mxfp_torch)
```python
409|     out_padded = out_padded.view(*fp32_tensor.shape[:-1], new_axis_shape)
```
**EN:** Assigns `out_padded` and calls `out_padded.view`.

**CN:** 将 `out_padded`，并调用 `out_padded.view`.

### Block 126 — Lines 410-411 (upcast_from_mxfp_torch)
```python
410|     out_tensor = out_padded[..., :axis_shape]
411| 
```
**EN:** Assigns `out_tensor` and evaluates `out_padded[..., :axis_shape]`.

**CN:** 将 `out_tensor` and 计算 `out_padded[..., :axis_shape]`.

### Block 127 — Lines 412-412 (upcast_from_mxfp_torch)
```python
412|     out_tensor = out_tensor.to(target_dtype).contiguous()
```
**EN:** Assigns `out_tensor` and calls `out_tensor.to(target_dtype).contiguous`.

**CN:** 将 `out_tensor`，并调用 `out_tensor.to(target_dtype).contiguous`.

### Block 128 — Lines 413-414 (upcast_from_mxfp_torch)
```python
413|     out_tensor = out_tensor.transpose(axis, tensor.ndim - 1)
414| 
```
**EN:** Assigns `out_tensor` and calls `out_tensor.transpose`.

**CN:** 将 `out_tensor`，并调用 `out_tensor.transpose`.

### Block 129 — Lines 415-417 (upcast_from_mxfp_torch)
```python
415|     return out_tensor
416| 
417| 
```
**EN:** Returns `out_tensor`.

**CN:** 返回 `out_tensor`.

### Block 130 — Lines 418-418 (module)
```python
418| quantize_mxfp8_fn = _quantize_mxfp8_fn
```
**EN:** Assigns `quantize_mxfp8_fn` and references `_quantize_mxfp8_fn`.

**CN:** 将 `quantize_mxfp8_fn` and 引用 `_quantize_mxfp8_fn`.

### Block 131 — Lines 419-419 (module)
```python
419| quantize_mxfp4_fn = _quantize_mxfp4_fn
```
**EN:** Assigns `quantize_mxfp4_fn` and references `_quantize_mxfp4_fn`.

**CN:** 将 `quantize_mxfp4_fn` and 引用 `_quantize_mxfp4_fn`.

### Block 132 — Lines 420-420 (module)
```python
420| quantize_nvfp4_fn = _quantize_nvfp4_fn
```
**EN:** Assigns `quantize_nvfp4_fn` and references `_quantize_nvfp4_fn`.

**CN:** 将 `quantize_nvfp4_fn` and 引用 `_quantize_nvfp4_fn`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `DequantScaleRoundingMode`, `downcast_to_mxfp`, `upcast_from_mxfp`, `right_shift_unsigned`, `get_max_quant_val`, `downcast_to_mxfp_torch`, `cvt_e2m1_to_fp32`, `upcast_from_mxfp_torch`.
  **CN:** 主要符号：`DequantScaleRoundingMode`, `downcast_to_mxfp`, `upcast_from_mxfp`, `right_shift_unsigned`, `get_max_quant_val`, `downcast_to_mxfp_torch`, `cvt_e2m1_to_fp32`, `upcast_from_mxfp_torch`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Flex-point style metadata or scaling is used to control quantized arithmetic.
  **CN:** 使用 Flex-point 风格的元数据或缩放来控制量化计算。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `enum (Enum)`, `math`, `triton`, `torch`, `torch.nn.functional`, `triton.tools.tensor_descriptor (TensorDescriptor)`.
  **CN:** 外部模块：`enum (Enum)`, `math`, `triton`, `torch`, `torch.nn.functional`, `triton.tools.tensor_descriptor (TensorDescriptor)`。
- **EN:** Internal modules: `.mxfp_details._upcast_from_mxfp (_upcast_from_mxfp)`, `.mxfp_details._downcast_to_mxfp (_downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE, _quantize_mxfp8_fn, _quantize_mxfp4_fn, _quantize_nvfp4_fn)`, `triton_kernels.tensor (Tensor, wrap_torch_tensor, empty)`, `triton_kernels.tensor_details.layout (StridedLayout)`, `triton_kernels.tensor_details.dtype (FP4, FP8_E4M3FN, FP8_E5M2, UINT8)`.
  **CN:** 内部模块：`.mxfp_details._upcast_from_mxfp (_upcast_from_mxfp)`, `.mxfp_details._downcast_to_mxfp (_downcast_to_mxfp, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE, _quantize_mxfp8_fn, _quantize_mxfp4_fn, _quantize_nvfp4_fn)`, `triton_kernels.tensor (Tensor, wrap_torch_tensor, empty)`, `triton_kernels.tensor_details.layout (StridedLayout)`, `triton_kernels.tensor_details.dtype (FP4, FP8_E4M3FN, FP8_E5M2, UINT8)`。
