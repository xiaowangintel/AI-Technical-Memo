# test_matmul.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/tests/test_matmul.py`
- **Purpose / 用途:** Pytest coverage for test matmul; it builds representative inputs and checks correctness, edge cases, or target-specific behavior. / 用于 test matmul 的 Pytest 测试覆盖；它会构造代表性输入，并检查正确性、边界情况或特定目标行为。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-2 (module)
```python
1| # isort: off
2| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 3-30 (module)
```python
 3| from dataclasses import dataclass, fields
 4| import itertools
 5| import pytest
 6| import torch
 7| from typing import Union
 8| import triton
 9| from triton._internal_testing import is_hopper
10| # matmul utilities
11| import triton_kernels.matmul_details.opt_flags as opt_flags
12| from triton_kernels.matmul import FlexCtx, PrecisionConfig, FusedActivation, FnSpecs, FnName, Epilogue
13| from triton_kernels.matmul import matmul_set_idle_sms, matmul, matmul_torch
14| # numerics utilities
15| from triton_kernels.numerics import InFlexData, OutFlexData
16| from triton_kernels.numerics_details.mxfp import upcast_from_mxfp, quantize_mxfp8_fn, quantize_nvfp4_fn, downcast_to_mxfp_torch, upcast_from_mxfp_torch, MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE
17| # testing utilities
18| from triton_kernels.testing import assert_close, make_random_tensor
19| # target-specific utilities
20| from triton_kernels.target_info import is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_gfx1250
21| from triton_kernels.swiglu import swiglu, swiglu_fn
22| from triton_kernels.swiglu import PrecisionConfig as SwiGLUPrecisionConfig
23| from triton_kernels.tensor_details import layout
24| from triton_kernels.tensor import Tensor, convert_layout, wrap_torch_tensor
25| from triton_kernels.tensor_details.dtype import FP32
26| 
27| # ---------------
28| # numerics stuff
29| # ---------------
30| 
```
**EN:** This block imports `dataclasses (dataclass, fields)`, `itertools`, `pytest`, `torch`, `typing (Union)`, `triton`, `triton._internal_testing (is_hopper)`, `triton_kernels.matmul_details.opt_flags` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass, fields)`, `itertools`, `pytest`, `torch`, `typing (Union)`, `triton`, `triton._internal_testing (is_hopper)`, `triton_kernels.matmul_details.opt_flags` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 31-32 (DType)
```python
31| class DType:
32| 
```
**EN:** Defines class `DType` to organize related behavior. Main methods are `__init__`.

**CN:** 定义类 `DType`，用于组织相关行为。主要方法有 `__init__`.

### Block 4 — Lines 33-33 (__init__)
```python
33|     def __init__(self, dtype_str):
```
**EN:** Defines function `__init__(self, dtype_str)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `dtype_str.startswith`, `getattr`, `dtype_str.strip` to implement its workflow.

**CN:** 定义函数 `__init__(self, dtype_str)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `dtype_str.startswith`, `getattr`, `dtype_str.strip` 来实现其工作流程.

### Block 5 — Lines 34-36 (__init__)
```python
34|         self.name = dtype_str
35|         # This tracks the regular fp8 flex scale path. NVFP4 has a tensor scale,
36|         # but it is handled separately because it also has MX microscale storage.
```
**EN:** Assigns `self.name` and references `dtype_str`.

**CN:** 将 `self.name` and 引用 `dtype_str`.

### Block 6 — Lines 37-37 (__init__)
```python
37|         self.has_global_scale = dtype_str.startswith("float8")
```
**EN:** Assigns `self.has_global_scale` and calls `dtype_str.startswith`.

**CN:** 将 `self.has_global_scale`，并调用 `dtype_str.startswith`.

### Block 7 — Lines 38-38 (__init__)
```python
38|         self.is_nvfp4 = dtype_str == "nvfp4_e2m1"
```
**EN:** Assigns `self.is_nvfp4` and evaluates `dtype_str == 'nvfp4_e2m1'`.

**CN:** 将 `self.is_nvfp4` and 计算 `dtype_str == 'nvfp4_e2m1'`.

### Block 8 — Lines 39-39 (__init__)
```python
39|         self.has_mx_scale = dtype_str.startswith("mx") or self.is_nvfp4
```
**EN:** Assigns `self.has_mx_scale` and evaluates `dtype_str.startswith('mx') or self.is_nvfp4`.

**CN:** 将 `self.has_mx_scale` and 计算 `dtype_str.startswith('mx') or self.is_nvfp4`.

### Block 9 — Lines 40-40 (__init__)
```python
40|         self.is_any_float8 = "float8" in dtype_str
```
**EN:** Assigns `self.is_any_float8` and evaluates `'float8' in dtype_str`.

**CN:** 将 `self.is_any_float8` and 计算 `'float8' in dtype_str`.

### Block 10 — Lines 41-41 (__init__)
```python
41|         self.uses_fp8e4nv = dtype_str in {"mxfloat8_e4m3fn", "nvfp4_e2m1"}
```
**EN:** Assigns `self.uses_fp8e4nv` and evaluates `dtype_str in {'mxfloat8_e4m3fn', 'nvfp4_e2m1'}`.

**CN:** 将 `self.uses_fp8e4nv` and 计算 `dtype_str in {'mxfloat8_e4m3fn', 'nvfp4_e2m1'}`.

### Block 11 — Lines 42-45 (__init__)
```python
42|         if dtype_str in {"float4_e2m1", "mxfloat4_e2m1", "nvfp4_e2m1"}:
43|             self.torch_dtype = torch.uint8
44|         else:
45|             self.torch_dtype = getattr(torch, dtype_str.strip("mx"))
```
**EN:** Checks `dtype_str in {'float4_e2m1', 'mxfloat4_e2m1', 'nvfp4_e2m1'}`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `dtype_str in {'float4_e2m1', 'mxfloat4_e2m1', 'nvfp4_e2m1'}`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 12 — Lines 46-46 (__init__)
```python
46|         self.is_mxfloat4 = self.has_mx_scale and ("float4" in dtype_str or self.is_nvfp4)
```
**EN:** Assigns `self.is_mxfloat4` and evaluates `self.has_mx_scale and ('float4' in dtype_str or self.is_nvfp4)`.

**CN:** 将 `self.is_mxfloat4` and 计算 `self.has_mx_scale and ('float4' in dtype_str or self.is_nvfp4)`.

### Block 13 — Lines 47-47 (__init__)
```python
47|         self.scale_dtype = torch.float8_e4m3fn if self.is_nvfp4 else torch.uint8 if self.has_mx_scale else None
```
**EN:** Assigns `self.scale_dtype` and uses conditional expression `torch.float8_e4m3fn if self.is_nvfp4 else torch.uint8 if ...`.

**CN:** 将 `self.scale_dtype` and 使用条件表达式 `torch.float8_e4m3fn if self.is_nvfp4 else torch.uint8 if ...`.

### Block 14 — Lines 48-51 (__init__)
```python
48|         self.microblock_size = NVFP_BLOCK_SIZE.value if self.is_nvfp4 else MXFP_BLOCK_SIZE.value if self.has_mx_scale else None
49| 
50| 
51| # Scope to ensure that the opt_flags_constraints are reset after the test
```
**EN:** Assigns `self.microblock_size` and uses conditional expression `NVFP_BLOCK_SIZE.value if self.is_nvfp4 else MXFP_BLOCK_SI...`.

**CN:** 将 `self.microblock_size` and 使用条件表达式 `NVFP_BLOCK_SIZE.value if self.is_nvfp4 else MXFP_BLOCK_SI...`.

### Block 15 — Lines 52-53 (opt_flags_scope)
```python
52| @pytest.fixture
53| def opt_flags_scope(request):
```
**EN:** Defines function `opt_flags_scope(request)` with decorators `pytest.fixture` for this module. The body mainly invokes `opt_flags.reset_opt_flags_constr...`. It uses calls such as `opt_flags.reset_opt_flags_constraints` to implement its workflow.

**CN:** 定义函数 `opt_flags_scope(request)`，带有装饰器 `pytest.fixture`，供本模块使用. 主体主要invokes `opt_flags.reset_opt_flags_constr...`. 其中会调用 `opt_flags.reset_opt_flags_constraints` 来实现其工作流程.

### Block 16 — Lines 54-54 (opt_flags_scope)
```python
54|     yield
```
**EN:** Evaluates expression `(yield)` for its side effects.

**CN:** Evaluates expression `(yield)` 以产生副作用。

### Block 17 — Lines 55-57 (opt_flags_scope)
```python
55|     opt_flags.reset_opt_flags_constraints()
56| 
57| 
```
**EN:** Calls `opt_flags.reset_opt_flags_constraints` for side effects, registration, or validation.

**CN:** 调用 `opt_flags.reset_opt_flags_constraints` ，用于副作用、注册或校验。

### Block 18 — Lines 58-58 (make_constraints)
```python
58| def make_constraints(block_m, split_k, is_persistent, epilogue_subtile, hbm_swizzling, weight_dtype_str, num_warps):
```
**EN:** Defines function `make_constraints(block_m, split_k, is_persistent, epilogue_subtile, hbm_swizzling, weight_dtype_str, num_warps)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; returns the computed result. It uses calls such as `is_hip`, `is_hip_gfx1250`, `constraints.update` to implement its workflow.

**CN:** 定义函数 `make_constraints(block_m, split_k, is_persistent, epilogue_subtile, hbm_swizzling, weight_dtype_str, num_warps)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 返回计算结果. 其中会调用 `is_hip`, `is_hip_gfx1250`, `constraints.update` 来实现其工作流程.

### Block 19 — Lines 59-65 (make_constraints)
```python
59|     constraints = {
60|         "block_m": block_m,
61|         "split_k": split_k,
62|         "is_persistent": is_persistent,
63|         "epilogue_subtile": epilogue_subtile,
64|         "num_warps": num_warps,
65|     }
```
**EN:** Assigns `constraints` and builds a dictionary.

**CN:** 将 `constraints` and 构造一个字典.

### Block 20 — Lines 66-79 (make_constraints)
```python
66|     if is_hip() and hbm_swizzling and "float4" in weight_dtype_str:
67|         # Minimum block size to satisfy scale preshuffling
68|         if is_hip_gfx1250():
69|             constraints.update({
70|                 "block_m": 128,
71|                 "block_n": 128,
72|                 "block_k": 128
73|             })
74|         else:
75|             constraints.update({
76|                 "block_m": 32,
77|                 "block_n": 32,
78|                 "block_k": 256
79|             })
```
**EN:** Checks `is_hip() and hbm_swizzling and ('float4' in weight_dtype_str)`. The true branch mainly branches on runtime conditions.

**CN:** 检查 `is_hip() and hbm_swizzling and ('float4' in weight_dtype_str)`. 真分支主要根据运行时条件分支.

### Block 21 — Lines 80-86 (make_constraints)
```python
80|     return constraints
81| 
82| # ---------------
83| # unit tests
84| # ---------------
85| 
86| 
```
**EN:** Returns `constraints`.

**CN:** 返回 `constraints`.

### Block 22 — Lines 87-88 (Case)
```python
87| @dataclass
88| class Case:
```
**EN:** Defines class `Case` with decorators `dataclass` to organize related behavior. Key fields include `m`, `n`, `k`, `mode`, `act_dtype_str`, `weight_dtype_str`, `output_dtype_str`, `n_slices`. Main methods are `__post_init__`.

**CN:** 定义类 `Case`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `m`, `n`, `k`, `mode`, `act_dtype_str`, `weight_dtype_str`, `output_dtype_str`, `n_slices`。主要方法有 `__post_init__`.

### Block 23 — Lines 89-89 (Case)
```python
89|     m: int
```
**EN:** Annotated assignment stores `m` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `m` and 声明该带注解的字段.

### Block 24 — Lines 90-90 (Case)
```python
90|     n: int
```
**EN:** Annotated assignment stores `n` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `n` and 声明该带注解的字段.

### Block 25 — Lines 91-91 (Case)
```python
91|     k: int
```
**EN:** Annotated assignment stores `k` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `k` and 声明该带注解的字段.

### Block 26 — Lines 92-92 (Case)
```python
92|     mode: str
```
**EN:** Annotated assignment stores `mode` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mode` and 声明该带注解的字段.

### Block 27 — Lines 93-93 (Case)
```python
93|     act_dtype_str: str
```
**EN:** Annotated assignment stores `act_dtype_str` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `act_dtype_str` and 声明该带注解的字段.

### Block 28 — Lines 94-94 (Case)
```python
94|     weight_dtype_str: str
```
**EN:** Annotated assignment stores `weight_dtype_str` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `weight_dtype_str` and 声明该带注解的字段.

### Block 29 — Lines 95-95 (Case)
```python
95|     output_dtype_str: Union[str, None] = None
```
**EN:** Annotated assignment stores `output_dtype_str` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `output_dtype_str` and 保存常量 `None`.

### Block 30 — Lines 96-96 (Case)
```python
96|     n_slices: int = None
```
**EN:** Annotated assignment stores `n_slices` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `n_slices` and 保存常量 `None`.

### Block 31 — Lines 97-97 (Case)
```python
97|     split_k: int = 1
```
**EN:** Annotated assignment stores `split_k` and stores constant `1`.

**CN:** 带类型注解的赋值保存 `split_k` and 保存常量 `1`.

### Block 32 — Lines 98-98 (Case)
```python
98|     a_hbm_swizzling: bool = False
```
**EN:** Annotated assignment stores `a_hbm_swizzling` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `a_hbm_swizzling` and 保存常量 `False`.

### Block 33 — Lines 99-99 (Case)
```python
99|     b_hbm_swizzling: bool = False
```
**EN:** Annotated assignment stores `b_hbm_swizzling` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `b_hbm_swizzling` and 保存常量 `False`.

### Block 34 — Lines 100-100 (Case)
```python
100|     c_hbm_swizzling: bool = False
```
**EN:** Annotated assignment stores `c_hbm_swizzling` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `c_hbm_swizzling` and 保存常量 `False`.

### Block 35 — Lines 101-101 (Case)
```python
101|     shuffle_mxfp4_w_layout: bool = False
```
**EN:** Annotated assignment stores `shuffle_mxfp4_w_layout` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `shuffle_mxfp4_w_layout` and 保存常量 `False`.

### Block 36 — Lines 102-102 (Case)
```python
102|     epilogue_subtile: Union[int, None] = None
```
**EN:** Annotated assignment stores `epilogue_subtile` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `epilogue_subtile` and 保存常量 `None`.

### Block 37 — Lines 103-103 (Case)
```python
103|     a_transpose: bool = False
```
**EN:** Annotated assignment stores `a_transpose` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `a_transpose` and 保存常量 `False`.

### Block 38 — Lines 104-104 (Case)
```python
104|     b_transpose: bool = False
```
**EN:** Annotated assignment stores `b_transpose` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `b_transpose` and 保存常量 `False`.

### Block 39 — Lines 105-105 (Case)
```python
105|     c_transpose: bool = False
```
**EN:** Annotated assignment stores `c_transpose` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `c_transpose` and 保存常量 `False`.

### Block 40 — Lines 106-106 (Case)
```python
106|     colmajor_mxfp_weight: bool = True
```
**EN:** Annotated assignment stores `colmajor_mxfp_weight` and stores constant `True`.

**CN:** 带类型注解的赋值保存 `colmajor_mxfp_weight` and 保存常量 `True`.

### Block 41 — Lines 107-108 (Case)
```python
107|     swiglu_opts: tuple[float, float] = None
108| 
```
**EN:** Annotated assignment stores `swiglu_opts` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `swiglu_opts` and 保存常量 `None`.

### Block 42 — Lines 109-109 (__post_init__)
```python
109|     def __post_init__(self):
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly branches on runtime conditions.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要根据运行时条件分支.

### Block 43 — Lines 110-112 (__post_init__)
```python
110|         if self.n_slices is None:
111|             self.n_slices = 1 if self.mode == "plain" else 10
112| 
```
**EN:** Checks `self.n_slices is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `self.n_slices is None`. 真分支主要准备中间值.

### Block 44 — Lines 113-113 (_build_test_op_cases)
```python
113| def _build_test_op_cases():
```
**EN:** Defines function `_build_test_op_cases()` for this module. The body mainly prepares intermediate values; invokes `test_cases.extend`; prepares intermediate values. It uses calls such as `test_cases.extend`, `test_cases.append`, `Case`, `itertools.product` to implement its workflow.

**CN:** 定义函数 `_build_test_op_cases()`，供本模块使用. 主体主要准备中间值; invokes `test_cases.extend`; 准备中间值. 其中会调用 `test_cases.extend`, `test_cases.append`, `Case`, `itertools.product` 来实现其工作流程.

### Block 45 — Lines 114-115 (_build_test_op_cases)
```python
114|     test_cases = []
115|     # zero-sized
```
**EN:** Assigns `test_cases` and builds a list.

**CN:** 将 `test_cases` and 构造一个列表.

### Block 46 — Lines 116-120 (_build_test_op_cases)
```python
116|     test_cases.extend([
117|         Case(m, n, k, mode, "float16", "float16")
118|         for mode in ("ragged", "batched")
119|         for (m, n, k) in ((0, 5, 7), (5, 0, 7), (5, 7, 0))
120|     ])
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 47 — Lines 121-121 (_build_test_op_cases)
```python
121|     odd_shape1 = (727, 577, 859)
```
**EN:** Assigns `odd_shape1` and builds a tuple.

**CN:** 将 `odd_shape1` and 构造一个元组.

### Block 48 — Lines 122-122 (_build_test_op_cases)
```python
122|     odd_shape2 = (720, 576, 768)
```
**EN:** Assigns `odd_shape2` and builds a tuple.

**CN:** 将 `odd_shape2` and 构造一个元组.

### Block 49 — Lines 123-124 (_build_test_op_cases)
```python
123|     even_shape = (768, 512, 1024)
124|     # canonical float16
```
**EN:** Assigns `even_shape` and builds a tuple.

**CN:** 将 `even_shape` and 构造一个元组.

### Block 50 — Lines 125-129 (_build_test_op_cases)
```python
125|     test_cases.extend([
126|         Case(*shape, mode, "float16", "float16", split_k=split_k)
127|       for shape in [odd_shape1, even_shape] for mode in ["ragged", "batched"] for split_k in [1, 5]
128|     ])
129|     # native float8
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 51 — Lines 130-133 (_build_test_op_cases)
```python
130|     test_cases.extend([
131|         Case(*shape, mode, "float8_e5m2", "float8_e5m2", split_k=split_k)
132|      for shape in [odd_shape1, even_shape] for mode in ["ragged", "batched"] for split_k in [1, 5]
133|     ])
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 52 — Lines 134-138 (_build_test_op_cases)
```python
134|     test_cases.extend([
135|         Case(*even_shape, "ragged", "float8_e5m2", "float8_e5m2", epilogue_subtile=val)
136|         for val in (1, 2, 4)
137|     ])
138|     # fp32
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 53 — Lines 139-142 (_build_test_op_cases)
```python
139|     test_cases.extend([
140|         Case(1024, 1000, 2048, "ragged", "float32", "float32", b_transpose=True)
141|     ])
142|     # bfloat16 x mx
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 54 — Lines 143-155 (_build_test_op_cases)
```python
143|     for shape in [odd_shape2, even_shape]:
144|         test_cases.extend([
145|             Case(*shape, "plain", "bfloat16", "mxfloat4_e2m1"),
146|             Case(*shape, "plain", "bfloat16", "mxfloat4_e2m1", b_hbm_swizzling=True),
147|             Case(*shape, "batched", "bfloat16", "mxfloat4_e2m1"),
148|             Case(*shape, "batched", "bfloat16", "mxfloat4_e2m1", b_hbm_swizzling=True),
149|             Case(*shape, "ragged", "bfloat16", "mxfloat4_e2m1"),
150|             Case(*shape, "ragged", "bfloat16", "mxfloat4_e2m1", b_hbm_swizzling=True),
151|             Case(*shape, "ragged", "bfloat16", "mxfloat4_e2m1", split_k=9),
152|             Case(*shape, "ragged", "bfloat16", "mxfloat4_e2m1", split_k=9, b_hbm_swizzling=True),
153|             Case(*shape, "ragged", "bfloat16", "mxfloat8_e4m3fn"),
154|             Case(*shape, "ragged", "bfloat16", "mxfloat8_e4m3fn", b_hbm_swizzling=True)
155|         ])
```
**EN:** Loops over `[odd_shape2, even_shape]` with target `shape`. The loop body mainly invokes `test_cases.extend`.

**CN:** 遍历 `[odd_shape2, even_shape]` ，目标变量为 `shape`. 循环体主要invokes `test_cases.extend`.

### Block 55 — Lines 156-157 (_build_test_op_cases)
```python
156|     test_cases.append(Case(64, 256, 32, "plain", "bfloat16", "mxfloat4_e2m1", b_hbm_swizzling=True))
157|     # float8 x mxfloat
```
**EN:** Calls `test_cases.append` for side effects, registration, or validation.

**CN:** 调用 `test_cases.append` ，用于副作用、注册或校验。

### Block 56 — Lines 158-172 (_build_test_op_cases)
```python
158|     test_cases.extend([
159|         Case(16, 256, 256, "ragged", "float8_e5m2", "mxfloat4_e2m1", b_hbm_swizzling=True),
160|         Case(16, 256, 256, "ragged", "float8_e5m2", "mxfloat4_e2m1", b_hbm_swizzling=True, shuffle_mxfp4_w_layout=True),
161|         Case(1024, 1024, 1024, "batched", "float8_e5m2", "mxfloat4_e2m1", b_hbm_swizzling=True),
162|         Case(1024, 1024, 1024, "batched", "float8_e5m2", "mxfloat4_e2m1", b_hbm_swizzling=True, shuffle_mxfp4_w_layout=True),
163|         Case(1024, 1024, 1024, "batched", "float8_e5m2", "mxfloat4_e2m1"),
164|         Case(1024, 1024, 1024, "ragged", "float8_e5m2", "mxfloat4_e2m1", split_k=9),
165|         Case(1024, 1024, 1024, "ragged", "float8_e5m2", "mxfloat4_e2m1", split_k=9, b_hbm_swizzling=True),
166|         Case(1024, 1024, 1024, "ragged", "float8_e5m2", "mxfloat4_e2m1", split_k=9, b_hbm_swizzling=True, shuffle_mxfp4_w_layout=True),
167|         Case(300, 400, 416, "ragged", "float8_e5m2", "mxfloat8_e4m3fn"),
168|         Case(300, 400, 832, "ragged", "float8_e5m2", "mxfloat4_e2m1"),
169|         Case(300, 400, 832, "ragged", "float8_e5m2", "mxfloat4_e2m1", b_hbm_swizzling=True, shuffle_mxfp4_w_layout=True),
170|         Case(300, 400, 416, "batched", "float8_e5m2", "mxfloat8_e4m3fn"),
171|     ])
172|     # mxfloat x mxfloat
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 57 — Lines 173-216 (_build_test_op_cases)
```python
173|     test_cases.extend([
174|         Case(16, 256, 256, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1"),
175|         Case(16, 256, 256, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", b_hbm_swizzling=True),
176|         Case(1024, 1024, 1024, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", split_k=9),
177|         Case(1024, 1024, 1024, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", split_k=9, b_hbm_swizzling=True),
178|         Case(1024, 1024, 1024, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", split_k=9, colmajor_mxfp_weight=False),
179|         Case(1000, 704, 800, "batched", "mxfloat8_e4m3fn", "mxfloat4_e2m1", b_hbm_swizzling=True, a_hbm_swizzling=True),
180|         Case(1000, 704, 800, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", b_hbm_swizzling=True, a_hbm_swizzling=True),
181|         Case(300, 400, 416, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", b_hbm_swizzling=True, a_hbm_swizzling=True),
182|         Case(256, 1024, 512, "ragged", "mxfloat8_e4m3fn", "mxfloat4_e2m1", b_hbm_swizzling=True, a_hbm_swizzling=True),
183|         Case(300, 400, 416, "ragged", "mxfloat8_e4m3fn", "mxfloat8_e4m3fn"),
184|         Case(300, 400, 416, "ragged", "mxfloat8_e4m3fn", "mxfloat8_e4m3fn", b_hbm_swizzling=True),
185|         Case(300, 400, 416, "batched", "mxfloat8_e4m3fn", "mxfloat8_e4m3fn"),
186|         Case(64, 128, 96, "ragged", "mxfloat8_e4m3fn", "bfloat16", "bfloat16"),
187|         Case(64, 128, 96, "batched", "mxfloat8_e4m3fn", "bfloat16", "bfloat16"),
188|         Case(1024, 1024, 1024, "batched", "mxfloat8_e4m3fn", "bfloat16", "bfloat16", split_k=9),
189|         Case(64, 128, 96, "ragged", "mxfloat8_e4m3fn", "float16", "bfloat16"),
190|         Case(64, 128, 96, "batched", "mxfloat8_e4m3fn", "float16", "bfloat16"),
191|         Case(1024, 1024, 1024, "batched", "mxfloat8_e4m3fn", "float16", "bfloat16", split_k=9),
192|         Case(64, 128, 96, "ragged", "mxfloat8_e4m3fn", "bfloat16", "bfloat16", a_hbm_swizzling=True),
193|         Case(64, 128, 96, "ragged", "mxfloat8_e4m3fn", "float16", "bfloat16", a_hbm_swizzling=True),
194|         Case(64, 128, 96, "ragged", "mxfloat4_e2m1", "bfloat16", "bfloat16"),
195|         Case(64, 128, 96, "batched", "mxfloat4_e2m1", "bfloat16", "bfloat16"),
196|         Case(1024, 1024, 1024, "batched", "mxfloat4_e2m1", "bfloat16", "bfloat16", split_k=9),
197|         Case(64, 128, 96, "ragged", "mxfloat4_e2m1", "float16", "bfloat16"),
198|         Case(64, 128, 96, "batched", "mxfloat4_e2m1", "float16", "bfloat16"),
199|         Case(1024, 1024, 1024, "batched", "mxfloat4_e2m1", "float16", "bfloat16", split_k=9),
200|         Case(64, 128, 96, "ragged", "mxfloat4_e2m1", "bfloat16", "bfloat16", a_hbm_swizzling=True),
201|         Case(64, 128, 96, "ragged", "mxfloat4_e2m1", "float16", "bfloat16", a_hbm_swizzling=True),
202|         Case(1024, 1024, 1024, "batched", "mxfloat8_e4m3fn", "mxfloat4_e2m1", b_hbm_swizzling=True),
203|         Case(256, 256, 256, "plain", "mxfloat4_e2m1", "mxfloat4_e2m1", "bfloat16"),
204|         Case(256, 256, 256, "plain", "mxfloat4_e2m1", "mxfloat4_e2m1", "bfloat16", b_hbm_swizzling=True),
205|         Case(16, 256, 256, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", b_hbm_swizzling=True),
206|         Case(1024, 1024, 1024, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", split_k=9, b_hbm_swizzling=True),
207|         Case(1024, 1024, 1024, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", split_k=9, colmajor_mxfp_weight=False),
208|         Case(1000, 704, 800, "batched", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", b_hbm_swizzling=True, a_hbm_swizzling=True),
209|         Case(1000, 704, 800, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", b_hbm_swizzling=True, a_hbm_swizzling=True),
210|         Case(300, 400, 416, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", b_hbm_swizzling=True, a_hbm_swizzling=True),
211|         Case(256, 1024, 512, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", b_hbm_swizzling=True, a_hbm_swizzling=True),
212|         Case(128, 256, 256, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "nvfp4_e2m1"),
213|         Case(128, 256, 256, "ragged", "nvfp4_e2m1", "nvfp4_e2m1", "nvfp4_e2m1", c_hbm_swizzling=True, b_hbm_swizzling=True, a_hbm_swizzling=True),
214|         Case(1024, 1024, 1024, "batched", "nvfp4_e2m1", "nvfp4_e2m1", "bfloat16", b_hbm_swizzling=True),
215|     ])
216|     # amd-specific float8
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 58 — Lines 217-223 (_build_test_op_cases)
```python
217|     test_cases.extend([
218|         Case(300, 400, 400, "ragged", "float8_e4m3fnuz", "float8_e4m3fnuz"),
219|         Case(1000, 400, 400, "ragged", "float8_e4m3fnuz", "float8_e4m3fnuz"),
220|         Case(600, 400, 400, "ragged", "float8_e4m3fnuz", "float8_e4m3fnuz", split_k=2),
221|         Case(300, 400, 400, "ragged", "float8_e4m3fn", "float8_e4m3fn"),
222|     ])
223|     # transposes / permutes
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 59 — Lines 224-228 (_build_test_op_cases)
```python
224|     test_cases.extend([
225|         Case(320, 400, 400, "batched", "float16", "float16",
226|                 a_transpose=a_tr, b_transpose=b_tr, c_transpose=c_tr)
227|         for a_tr, b_tr, c_tr in itertools.product((False, True), repeat=3)
228|     ])
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 60 — Lines 229-235 (_build_test_op_cases)
```python
229|     test_cases.extend([
230|         Case(320, 400, 400, "ragged", "float8_e5m2", "float8_e5m2",
231|                 a_transpose=False, b_transpose=True, c_transpose=False),
232|         Case(320, 400, 400, "ragged", "float8_e5m2", "float8_e5m2",
233|                 a_transpose=True, b_transpose=True, c_transpose=True),
234|     ])
235|     # swiglu
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 61 — Lines 236-239 (_build_test_op_cases)
```python
236|     test_cases.extend([
237|         Case(*shape, mode, "bfloat16", "bfloat16", split_k=split_k, swiglu_opts=(1.1, 1.4))
238|      for shape in [odd_shape2, even_shape] for mode in ["ragged", "batched"] for split_k in [1, 5]
239|     ])
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 62 — Lines 240-244 (_build_test_op_cases)
```python
240|     test_cases.extend([
241|         Case(*even_shape, "ragged", "bfloat16", "bfloat16", epilogue_subtile=val, swiglu_opts=(1.1, 1.4))
242|         for val in (1, 2, 4)
243|     ])
244|     # swiglu together with mxfp8 downcastepilogue
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 63 — Lines 245-249 (_build_test_op_cases)
```python
245|     test_cases.extend([
246|         Case(*shape, mode, "mxfloat8_e4m3fn", "mxfloat4_e2m1", a_hbm_swizzling=True, b_hbm_swizzling=True, split_k=split_k, swiglu_opts=(1.1, 7))
247|      for shape in [odd_shape2, even_shape] for mode in ["ragged", "batched"] for split_k in [1, 5]
248|     ])
249|     # swiglu together with nvfp4 downcast epilogue
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 64 — Lines 250-254 (_build_test_op_cases)
```python
250|     test_cases.extend([
251|         Case(*shape, mode, "bfloat16", "bfloat16", "nvfp4_e2m1", swiglu_opts=(1.1, 7.0))
252|         for shape in [even_shape]
253|         for mode in ["ragged", "batched"]
254|     ])
```
**EN:** Calls `test_cases.extend` for side effects, registration, or validation.

**CN:** 调用 `test_cases.extend` ，用于副作用、注册或校验。

### Block 65 — Lines 255-256 (_build_test_op_cases)
```python
255|     test_cases.append(Case(256, 2048, 1024, "plain", "bfloat16", "bfloat16", "nvfp4_e2m1", swiglu_opts=(1.1, 7.0)))
256| 
```
**EN:** Calls `test_cases.append` for side effects, registration, or validation.

**CN:** 调用 `test_cases.append` ，用于副作用、注册或校验。

### Block 66 — Lines 257-258 (_build_test_op_cases)
```python
257|     return test_cases
258| 
```
**EN:** Returns `test_cases`.

**CN:** 返回 `test_cases`.

### Block 67 — Lines 259-283 (test_op)
```python
259| @pytest.mark.parametrize(
260|     ", ".join(f.name for f in fields(Case)),
261|     [
262|         tuple(getattr(case, f.name) for f in fields(Case))
263|         for case in _build_test_op_cases()
264|     ],
265| )
266| @pytest.mark.parametrize("block_m", [16, 128])
267| @pytest.mark.parametrize("do_gather, do_scatter, inner_expt_opt", [
268|     (False, False, None),
269|     (True, False, None),
270|     (False, True, None),
271|     (True, True, None),
272|     (False, False, "pad_b"),
273|     (False, False, "pad_a"),
274| ])
275| @pytest.mark.parametrize("do_gamma", [False,True])
276| @pytest.mark.parametrize("is_persistent", [False,True])
277| @pytest.mark.parametrize("num_warps", [4, 8] if is_hopper() else [None])
278| def test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices,
279|             mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile,
280|             a_transpose, b_transpose, c_transpose,
281|             swiglu_opts, c_hbm_swizzling, device, opt_flags_scope):
282|     # We catch and re-invoke pytest.skip(), because otherwise pytest may hold a reference to
283|     # the frame that called pytest.skip, including all the tensors, leading to OOM.
```
**EN:** Defines function `test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices, mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile, a_transpose, b_transpose, c_transpose, swiglu_opts, c_hbm_swizzling, device, opt_flags_scope)` with decorators `pytest.mark.parametrize(', '.join((f....`, `pytest.mark.parametrize('block_m', [1...`, `pytest.mark.parametrize('do_gather, d...`, `pytest.mark.parametrize('do_gamma', [...`, `pytest.mark.parametrize('is_persisten...`, `pytest.mark.parametrize('num_warps', ...` for this module. The body mainly prepares intermediate values; handles exceptional paths; branches on runtime conditions. It uses calls such as `pytest.mark.parametrize`, `', '.join`, `_test_op`, `pytest.skip`, `tuple` to implement its workflow.

**CN:** 定义函数 `test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices, mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile, a_transpose, b_transpose, c_transpose, swiglu_opts, c_hbm_swizzling, device, opt_flags_scope)`，带有装饰器 `pytest.mark.parametrize(', '.join((f....`, `pytest.mark.parametrize('block_m', [1...`, `pytest.mark.parametrize('do_gather, d...`, `pytest.mark.parametrize('do_gamma', [...`, `pytest.mark.parametrize('is_persisten...`, `pytest.mark.parametrize('num_warps', ...`，供本模块使用. 主体主要准备中间值; 处理异常路径; 根据运行时条件分支. 其中会调用 `pytest.mark.parametrize`, `', '.join`, `_test_op`, `pytest.skip`, `tuple` 来实现其工作流程.

### Block 68 — Lines 284-284 (test_op)
```python
284|     skip_message = None
```
**EN:** Assigns `skip_message` and stores constant `None`.

**CN:** 将 `skip_message` and 保存常量 `None`.

### Block 69 — Lines 285-292 (test_op)
```python
285|     try:
286|         _test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices,
287|                  mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile,
288|                  a_transpose, b_transpose, c_transpose,
289|                  swiglu_opts, c_hbm_swizzling, device, opt_flags_scope)
290|     except pytest.skip.Exception as e:
291|         skip_message = str(e)
292| 
```
**EN:** Wraps code that mainly invokes `_test_op` with exception/finally handling.

**CN:** 包装代码 that mainly invokes `_test_op` 以及异常/收尾处理.

### Block 70 — Lines 293-295 (test_op)
```python
293|     if skip_message is not None:
294|         pytest.skip(skip_message)
295| 
```
**EN:** Checks `skip_message is not None`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `skip_message is not None`. 真分支主要invokes `pytest.skip`.

### Block 71 — Lines 296-299 (_test_op)
```python
296| def _test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices,
297|             mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile,
298|             a_transpose, b_transpose, c_transpose,
299|             swiglu_opts, c_hbm_swizzling, device, opt_flags_scope):
```
**EN:** Defines function `_test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices, mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile, a_transpose, b_transpose, c_transpose, swiglu_opts, c_hbm_swizzling, device, opt_flags_scope)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `DType`, `is_cuda`, `torch.manual_seed`, `make_constraints`, `opt_flags.update_opt_flags_constraints` to implement its workflow.

**CN:** 定义函数 `_test_op(m, n, k, split_k, do_gather, do_scatter, inner_expt_opt, do_gamma, is_persistent, num_warps, n_slices, mode, act_dtype_str, weight_dtype_str, output_dtype_str, block_m, b_hbm_swizzling, shuffle_mxfp4_w_layout, a_hbm_swizzling, colmajor_mxfp_weight, epilogue_subtile, a_transpose, b_transpose, c_transpose, swiglu_opts, c_hbm_swizzling, device, opt_flags_scope)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `DType`, `is_cuda`, `torch.manual_seed`, `make_constraints`, `opt_flags.update_opt_flags_constraints` 来实现其工作流程.

### Block 72 — Lines 300-300 (_test_op)
```python
300|     a_dtype = DType(act_dtype_str)
```
**EN:** Assigns `a_dtype` and calls `DType`.

**CN:** 将 `a_dtype`，并调用 `DType`.

### Block 73 — Lines 301-301 (_test_op)
```python
301|     b_dtype = DType(weight_dtype_str)
```
**EN:** Assigns `b_dtype` and calls `DType`.

**CN:** 将 `b_dtype`，并调用 `DType`.

### Block 74 — Lines 302-302 (_test_op)
```python
302|     c_dtype = DType(output_dtype_str or act_dtype_str)
```
**EN:** Assigns `c_dtype` and calls `DType`.

**CN:** 将 `c_dtype`，并调用 `DType`.

### Block 75 — Lines 303-304 (_test_op)
```python
303|     device_capability = torch.cuda.get_device_capability()[0]
304|     # TODO: remove when Triton FP8 supports proper RTNE
```
**EN:** Assigns `device_capability` and evaluates `torch.cuda.get_device_capability()[0]`.

**CN:** 将 `device_capability` and 计算 `torch.cuda.get_device_capability()[0]`.

### Block 76 — Lines 305-335 (_test_op)
```python
305|     if is_cuda():
306|         if device_capability < 9 and (a_dtype.uses_fp8e4nv or b_dtype.uses_fp8e4nv or c_dtype.uses_fp8e4nv):
307|             pytest.skip("MXFP8/NVFP4 tensors use fp8e4nv, which is not supported on A100")
308|         if b_dtype.is_any_float8 and device_capability < 9:
309|             pytest.skip("Float8 not tested on A100")
310|         if act_dtype_str == "float16" and b_dtype.has_mx_scale and device_capability >= 10:
311|             pytest.skip("float16 x mx not supported with cuda capability >= 10")
312|         if b_dtype.has_mx_scale and a_dtype.has_global_scale and device_capability < 10:
313|             pytest.skip("float8 x mx not supported with cuda capability < 10")
314|         if swiglu_opts is not None and do_gamma:
315|             pytest.skip("NYI: swiglu and gamma not supported together")
316| 
317|     elif is_hip():
318|         if a_dtype.is_any_float8 and b_dtype.has_mx_scale and not (is_hip_cdna4() or is_hip_gfx1250()):
319|             pytest.skip("float8 x mx only supported on CDNA4 and gfx1250")
320|         if a_dtype.is_any_float8 and b_dtype.name == "mxfloat8_e4m3fn":
321|             pytest.skip("NYI: float8 x mxfloat8 not tested on AMD GPU")
322|         if a_dtype.has_mx_scale and b_dtype.has_mx_scale:
323|             pytest.skip("NYI: mx x mx not tested on AMD GPU")
324|         if a_dtype.name == "mxfloat4_e2m1" and weight_dtype_str in {"bfloat16", "float16"}:
325|             pytest.skip("NYI: MXFP4 x dense FP16/BF16 not tested on AMD GPU")
326|         if is_persistent:
327|             pytest.skip("NYI: Persistent kernel not supported on AMD GPU")
328|         # FIXME: this works on nvidia; looks like some sort of bug on AMD?
329|         if do_gamma and swiglu_opts is not None:
330|             pytest.skip("NYI: gamma and swiglu not supported together on AMD GPU")
331|         if split_k is not None and split_k > 1:
332|             pytest.skip("splitK hasn't been fully tested on AMD GPU.")
333|         if act_dtype_str == "float32":
334|             pytest.skip("float32 not fully tested on AMD GPU")
335| 
```
**EN:** Checks `is_cuda()`. The true branch mainly branches on runtime conditions; branches on runtime conditions, while the else branch branches on runtime conditions.

**CN:** 检查 `is_cuda()`. 真分支主要根据运行时条件分支; 根据运行时条件分支；而 else 分支根据运行时条件分支.

### Block 77 — Lines 336-338 (_test_op)
```python
336|     if "float8_e4m3fnuz" in (weight_dtype_str, act_dtype_str) and not is_hip_cdna3():
337|         pytest.skip("float8_e4m3fnuz only tested on AMD CDNA3 Platform")
338| 
```
**EN:** Checks `'float8_e4m3fnuz' in (weight_dtype_str, act_dtype_str) and (not is_hip_cdna3())`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `'float8_e4m3fnuz' in (weight_dtype_str, act_dtype_str) and (not is_hip_cdna3())`. 真分支主要invokes `pytest.skip`.

### Block 78 — Lines 339-352 (_test_op)
```python
339|     if b_hbm_swizzling:
340|         if is_hip():
341|             if not (is_hip_cdna4() or is_hip_gfx1250()):
342|                 pytest.skip("Scale preshuffling on AMD GPU has not been emulated on archs other than CDNA4 and gfx1250 yet.")
343|             if not b_dtype.has_mx_scale:
344|                 pytest.skip("Non-scale swizzling not supported on CDNA4 yet")
345|         if device_capability < 9:
346|             pytest.skip("NYI. Ampere swizzling.")
347|         if device_capability < 10:
348|             if b_dtype.name != "mxfloat4_e2m1":
349|                 pytest.skip("NYI. Hopper swizzling just implemented for mxfp4.")
350|             if a_dtype.is_mxfloat4:
351|                 pytest.skip("Hopper mxfp4 swizzled weights do not support FP4 microscaled lhs.")
352| 
```
**EN:** Checks `b_hbm_swizzling`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `b_hbm_swizzling`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 79 — Lines 353-367 (_test_op)
```python
353|     if a_hbm_swizzling:
354|         # current x scale swizzling requires B200, batched input, microscaled act and persistent case
355|         if is_hip():
356|             pytest.skip("NYI. X swizzling not tested on AMD GPU yet.")
357|         if device_capability < 10:
358|             pytest.skip("NYI. X swizzling only implemented for B200 for now.")
359|         if not a_dtype.has_mx_scale:
360|             pytest.skip(f"NYI. X swizzling only implemented for microscaled activations for now. Got {act_dtype_str}")
361|         if not is_persistent:
362|             pytest.skip("NYI. X swizzling only implemented for persistent case for now.")
363|         if block_m < 128:
364|             pytest.skip("X swizzling requires block_m >= 128")
365|         if do_gather:
366|             pytest.skip("X swizzling does not support gathered activations")
367| 
```
**EN:** Checks `a_hbm_swizzling`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `a_hbm_swizzling`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 80 — Lines 368-373 (_test_op)
```python
368|     if c_hbm_swizzling:
369|         if is_hip() or torch.cuda.get_device_capability()[0] < 10:
370|             pytest.skip("NYI. Output scale swizzling is only implemented on Blackwell")
371|         if do_scatter:
372|             pytest.skip("NYI. Output scale swizzling does not support fused scatter")
373| 
```
**EN:** Checks `c_hbm_swizzling`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `c_hbm_swizzling`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 81 — Lines 374-374 (_test_op)
```python
374|     expt_is_inner = (inner_expt_opt is not None)
```
**EN:** Assigns `expt_is_inner` and evaluates `inner_expt_opt is not None`.

**CN:** 将 `expt_is_inner` and 计算 `inner_expt_opt is not None`.

### Block 82 — Lines 375-389 (_test_op)
```python
375|     if expt_is_inner:
376|         if mode != "ragged":
377|             pytest.skip("inner_expt_opt only meaningful with ragged")
378|         if a_dtype.has_mx_scale and inner_expt_opt != "pad_a":
379|             pytest.skip("inner_expt_opt and act mx only supported with pad_a")
380|         if b_dtype.has_mx_scale:
381|             if inner_expt_opt != "pad_b":
382|                 pytest.skip("inner_expt_opt and weight mx only supported with pad_b")
383|             if is_persistent and not b_hbm_swizzling:
384|                 pytest.skip("FIXME: Fatal Python error: Aborted")
385|             if is_hip():
386|                 if act_dtype_str == "bfloat16":
387|                     pytest.skip("FIXME: failed to translate module to LLVM IR")
388|                 if b_hbm_swizzling:
389|                     pytest.skip("NYI: nner_expt_opt and HBM swizzling")
```
**EN:** Checks `expt_is_inner`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `expt_is_inner`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 83 — Lines 390-393 (_test_op)
```python
390|     if not colmajor_mxfp_weight:
391|         if block_m == 16:
392|             pytest.skip("PassManager::run failed from Triton compiler")
393|     # TODO: should construct the test case differently rather than overriding here
```
**EN:** Checks `not colmajor_mxfp_weight`. The true branch mainly branches on runtime conditions.

**CN:** 检查 `not colmajor_mxfp_weight`. 真分支主要根据运行时条件分支.

### Block 84 — Lines 394-396 (_test_op)
```python
394|     if b_dtype.is_any_float8 and device_capability < 10:
395|         b_transpose = True
396| 
```
**EN:** Checks `b_dtype.is_any_float8 and device_capability < 10`. The true branch mainly prepares intermediate values.

**CN:** 检查 `b_dtype.is_any_float8 and device_capability < 10`. 真分支主要准备中间值.

### Block 85 — Lines 397-399 (_test_op)
```python
397|     torch.manual_seed(0)
398| 
399|     # set opt flags constraints
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 86 — Lines 400-400 (_test_op)
```python
400|     constraints = make_constraints(block_m, split_k, is_persistent, epilogue_subtile, b_hbm_swizzling, weight_dtype_str, num_warps)
```
**EN:** Assigns `constraints` and calls `make_constraints`.

**CN:** 将 `constraints`，并调用 `make_constraints`.

### Block 87 — Lines 401-401 (_test_op)
```python
401|     use_blackwell_shuffled_w_layout = shuffle_mxfp4_w_layout and b_hbm_swizzling
```
**EN:** Assigns `use_blackwell_shuffled_w_layout` and evaluates `shuffle_mxfp4_w_layout and b_hbm_swizzling`.

**CN:** 将 `use_blackwell_shuffled_w_layout` and 计算 `shuffle_mxfp4_w_layout and b_hbm_swizzling`.

### Block 88 — Lines 402-414 (_test_op)
```python
402|     if shuffle_mxfp4_w_layout:
403|         if not b_hbm_swizzling:
404|             pytest.skip("Shuffled MXFP4 weight layout only applies with b_hbm_swizzling")
405|         if is_hip() or device_capability < 10:
406|             pytest.skip("Shuffled MXFP4 weight layout requires Blackwell or newer")
407|         if b_dtype.name != "mxfloat4_e2m1":
408|             pytest.skip("Shuffled MXFP4 weight layout only supports mxfloat4_e2m1 weights")
409|         if not a_dtype.has_global_scale:
410|             pytest.skip("Shuffled MXFP4 weight layout is only tested with FP8 activations")
411|         if not colmajor_mxfp_weight:
412|             pytest.skip("Shuffled MXFP4 weight layout requires column-major MXFP weights")
413|         if not is_persistent:
414|             pytest.skip("Shuffled MXFP4 weight layout requires the persistent TMA kernel")
```
**EN:** Checks `shuffle_mxfp4_w_layout`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `shuffle_mxfp4_w_layout`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 89 — Lines 415-417 (_test_op)
```python
415|     opt_flags.update_opt_flags_constraints(constraints)
416| 
417|     # --- create conditionals ---
```
**EN:** Calls `opt_flags.update_opt_flags_constraints` for side effects, registration, or validation.

**CN:** 调用 `opt_flags.update_opt_flags_constraints` ，用于副作用、注册或校验。

### Block 90 — Lines 418-418 (_test_op)
```python
418|     do_bias = inner_expt_opt is None
```
**EN:** Assigns `do_bias` and evaluates `inner_expt_opt is None`.

**CN:** 将 `do_bias` and 计算 `inner_expt_opt is None`.

### Block 91 — Lines 419-419 (_test_op)
```python
419|     do_gather = do_gather and mode != "batched"
```
**EN:** Assigns `do_gather` and evaluates `do_gather and mode != 'batched'`.

**CN:** 将 `do_gather` and 计算 `do_gather and mode != 'batched'`.

### Block 92 — Lines 420-420 (_test_op)
```python
420|     do_scatter = do_scatter and mode != "batched"
```
**EN:** Assigns `do_scatter` and evaluates `do_scatter and mode != 'batched'`.

**CN:** 将 `do_scatter` and 计算 `do_scatter and mode != 'batched'`.

### Block 93 — Lines 421-421 (_test_op)
```python
421|     b_value_hbm_swizzling = None
```
**EN:** Assigns `b_value_hbm_swizzling` and stores constant `None`.

**CN:** 将 `b_value_hbm_swizzling` and 保存常量 `None`.

### Block 94 — Lines 422-428 (_test_op)
```python
422|     if b_hbm_swizzling and colmajor_mxfp_weight and b_dtype.is_mxfloat4:
423|         b_value_hbm_swizzling = layout.make_default_matmul_mxfp4_w_layout(
424|             mx_axis=-2,
425|             allow_blackwell_value_shuffle=use_blackwell_shuffled_w_layout,
426|         )
427| 
428|     # --- create inputs ---
```
**EN:** Checks `b_hbm_swizzling and colmajor_mxfp_weight and b_dtype.is_mxfloat4`. The true branch mainly prepares intermediate values.

**CN:** 检查 `b_hbm_swizzling and colmajor_mxfp_weight and b_dtype.is_mxfloat4`. 真分支主要准备中间值.

### Block 95 — Lines 429-440 (_test_op)
```python
429|     a, a_scales, a_ragged_metadata = make_random_tensor(
430|         shape=(m, k),
431|         n_slices = n_slices,
432|         dtype = a_dtype,
433|         device = device,
434|         ragged_dim = None if mode != "ragged" else 1 if expt_is_inner else 0,
435|         mxfp_dim = -1 if a_dtype.has_mx_scale else None,
436|         transpose = a_transpose,
437|         ragged_padding = inner_expt_opt is not None and "pad_a" in inner_expt_opt,
438|         squeeze_batch_dim = mode == "plain",
439|         scale_hbm_swizzling = layout.make_default_matmul_mx_act_scale_layout if a_hbm_swizzling else None,
440|     )
```
**EN:** Assigns `a`, `a_scales`, `a_ragged_metadata` and calls `make_random_tensor`.

**CN:** 将 `a`, `a_scales`, `a_ragged_metadata`，并调用 `make_random_tensor`.

### Block 96 — Lines 441-454 (_test_op)
```python
441|     b, b_scale_tri, b_ragged_metadata = make_random_tensor(
442|         shape=(k, n),
443|         n_slices = n_slices,
444|         dtype = b_dtype,
445|         device = device,
446|         ragged_dim = None if mode != "ragged" or inner_expt_opt is None else 0,
447|         mxfp_dim = -2 if b_dtype.has_mx_scale else None,
448|         transpose = b_transpose,
449|         ragged_padding = inner_expt_opt is not None and "pad_b" in inner_expt_opt,
450|         squeeze_batch_dim = mode == "plain",
451|         is_mx_rowmajor = not colmajor_mxfp_weight,
452|         value_hbm_swizzling = b_value_hbm_swizzling,
453|         scale_hbm_swizzling = layout.make_default_matmul_mxfp4_w_scale_layout(mx_axis=-2, num_warps=num_warps) if b_hbm_swizzling and colmajor_mxfp_weight and b_dtype.is_mxfloat4 else None,
454|     )
```
**EN:** Assigns `b`, `b_scale_tri`, `b_ragged_metadata` and calls `make_random_tensor`.

**CN:** 将 `b`, `b_scale_tri`, `b_ragged_metadata`，并调用 `make_random_tensor`.

### Block 97 — Lines 455-456 (_test_op)
```python
455|     if use_blackwell_shuffled_w_layout:
456|         assert isinstance(b.storage.layout, layout.BlackwellMX4ValueShuffledLayout)
```
**EN:** Checks `use_blackwell_shuffled_w_layout`. The true branch mainly checks invariants.

**CN:** 检查 `use_blackwell_shuffled_w_layout`. 真分支主要检查不变量.

### Block 98 — Lines 457-457 (_test_op)
```python
457|     gather_indx  = None if not do_gather  else torch.randint(0, max(m, 1), (m, ), dtype=torch.int32, device=device)
```
**EN:** Assigns `gather_indx` and uses conditional expression `None if not do_gather else torch.randint(0, max(m, 1), (m...`.

**CN:** 将 `gather_indx` and 使用条件表达式 `None if not do_gather else torch.randint(0, max(m, 1), (m...`.

### Block 99 — Lines 458-458 (_test_op)
```python
458|     scatter_indx = None if not do_scatter else torch.randperm(m, dtype=torch.int32, device=device)
```
**EN:** Assigns `scatter_indx` and uses conditional expression `None if not do_scatter else torch.randperm(m, dtype=torch...`.

**CN:** 将 `scatter_indx` and 使用条件表达式 `None if not do_scatter else torch.randperm(m, dtype=torch...`.

### Block 100 — Lines 459-459 (_test_op)
```python
459|     bias         = None if not do_bias    else torch.randn(b.shape[:-2] + b.shape[-1:], dtype=torch.float32, device=device)
```
**EN:** Assigns `bias` and uses conditional expression `None if not do_bias else torch.randn(b.shape[:-2] + b.sha...`.

**CN:** 将 `bias` and 使用条件表达式 `None if not do_bias else torch.randn(b.shape[:-2] + b.sha...`.

### Block 101 — Lines 460-462 (_test_op)
```python
460|     gammas       = None if not do_gamma   else 2**torch.randint(-5, 0, (m, ), dtype=torch.float32, device=device)
461| 
462|     # --- create fused activation ---
```
**EN:** Assigns `gammas` and uses conditional expression `None if not do_gamma else 2 ** torch.randint(-5, 0, (m,),...`.

**CN:** 将 `gammas` and 使用条件表达式 `None if not do_gamma else 2 ** torch.randint(-5, 0, (m,),...`.

### Block 102 — Lines 463-463 (_test_op)
```python
463|     fused_activation = None
```
**EN:** Assigns `fused_activation` and stores constant `None`.

**CN:** 将 `fused_activation` and 保存常量 `None`.

### Block 103 — Lines 464-467 (_test_op)
```python
464|     if swiglu_opts is not None:
465|         fused_activation = FusedActivation(FnSpecs("swiglu", swiglu_fn, ("alpha", "limit"), reduction_n=2), swiglu_opts)
466| 
467|     # --- initialize output ---
```
**EN:** Checks `swiglu_opts is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `swiglu_opts is not None`. 真分支主要准备中间值.

### Block 104 — Lines 468-468 (_test_op)
```python
468|     c_shape = (n_slices,) if mode == "batched" or inner_expt_opt is not None else tuple() # batch dim
```
**EN:** Assigns `c_shape` and uses conditional expression `(n_slices,) if mode == 'batched' or inner_expt_opt is not...`.

**CN:** 将 `c_shape` and 使用条件表达式 `(n_slices,) if mode == 'batched' or inner_expt_opt is not...`.

### Block 105 — Lines 469-469 (_test_op)
```python
469|     c_shape += (scatter_indx.shape[0] if do_scatter else a.shape[-2],) # row dim
```
**EN:** Updates `c_shape` with operator `Add` using `(scatter_indx.shape[0] if do_scatter else a.shape[-2],)`.

**CN:** 更新 `c_shape`，使用运算符 `Add`，并使用 `(scatter_indx.shape[0] if do_scatter else a.shape[-2],)`.

### Block 106 — Lines 470-470 (_test_op)
```python
470|     c_shape += (b.shape[-1] // (1 if fused_activation is None else fused_activation.specs.reduction_n) ,) # col dim
```
**EN:** Updates `c_shape` with operator `Add` using `(b.shape[-1] // (1 if fused_activation is None else fused...`.

**CN:** 更新 `c_shape`，使用运算符 `Add`，并使用 `(b.shape[-1] // (1 if fused_activation is None else fused...`.

### Block 107 — Lines 471-471 (_test_op)
```python
471|     c_storage_shape = c_shape[:-1] + (c_shape[-1] // 2,) if c_dtype.has_mx_scale and c_dtype.is_mxfloat4 else c_shape
```
**EN:** Assigns `c_storage_shape` and uses conditional expression `c_shape[:-1] + (c_shape[-1] // 2,) if c_dtype.has_mx_scal...`.

**CN:** 将 `c_storage_shape` and 使用条件表达式 `c_shape[:-1] + (c_shape[-1] // 2,) if c_dtype.has_mx_scal...`.

### Block 108 — Lines 472-472 (_test_op)
```python
472|     c = torch.empty(c_storage_shape, dtype=c_dtype.torch_dtype, device=device)
```
**EN:** Assigns `c` and calls `torch.empty`.

**CN:** 将 `c`，并调用 `torch.empty`.

### Block 109 — Lines 473-476 (_test_op)
```python
473|     if c_transpose:
474|         c = c.mT.contiguous().mT
475| 
476|     # --- create precision config ---
```
**EN:** Checks `c_transpose`. The true branch mainly prepares intermediate values.

**CN:** 检查 `c_transpose`. 真分支主要准备中间值.

### Block 110 — Lines 477-477 (_test_op)
```python
477|     wrap_list = lambda vals: torch.tensor(vals, dtype=torch.float32, device=device)
```
**EN:** Assigns `wrap_list` and defines a lambda.

**CN:** 将 `wrap_list` and 定义一个 lambda.

### Block 111 — Lines 478-478 (_test_op)
```python
478|     flex_a = InFlexData(c_dtype.torch_dtype, wrap_list([1.25])) if c_dtype.has_global_scale else InFlexData()
```
**EN:** Assigns `flex_a` and uses conditional expression `InFlexData(c_dtype.torch_dtype, wrap_list([1.25])) if c_d...`.

**CN:** 将 `flex_a` and 使用条件表达式 `InFlexData(c_dtype.torch_dtype, wrap_list([1.25])) if c_d...`.

### Block 112 — Lines 479-479 (_test_op)
```python
479|     flex_b = InFlexData(b_dtype.torch_dtype, wrap_list([1.25])) if b_dtype.has_global_scale else InFlexData()
```
**EN:** Assigns `flex_b` and uses conditional expression `InFlexData(b_dtype.torch_dtype, wrap_list([1.25])) if b_d...`.

**CN:** 将 `flex_b` and 使用条件表达式 `InFlexData(b_dtype.torch_dtype, wrap_list([1.25])) if b_d...`.

### Block 113 — Lines 480-485 (_test_op)
```python
480|     if c_dtype.has_global_scale:
481|         flex_c = OutFlexData(c_dtype.torch_dtype, wrap_list([4.00]), wrap_list([0]), None)
482|     elif c_dtype.is_nvfp4:
483|         flex_c = OutFlexData(c_dtype.torch_dtype, wrap_list([0.125]), None, None)
484|     else:
485|         flex_c = OutFlexData(c_dtype.torch_dtype, None, None, None)
```
**EN:** Checks `c_dtype.has_global_scale`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `c_dtype.has_global_scale`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 114 — Lines 486-496 (_test_op)
```python
486|     precision_opt = PrecisionConfig(
487|         flex_ctx=FlexCtx(flex_a, flex_b, flex_c),
488|         acc_scale=2.0 if c_dtype.has_global_scale or b_dtype.has_global_scale else 1.0,
489|         out_dtype=c_dtype.torch_dtype,
490|         a_mx_scale=a_scales,
491|         a_microblock_size=a_dtype.microblock_size,
492|         b_mx_scale=b_scale_tri,
493|         b_microblock_size=b_dtype.microblock_size,
494|     )
495| 
496|     # --- create epilogue ---
```
**EN:** Assigns `precision_opt` and calls `PrecisionConfig`.

**CN:** 将 `precision_opt`，并调用 `PrecisionConfig`.

### Block 115 — Lines 497-497 (_test_op)
```python
497|     epilogue = None
```
**EN:** Assigns `epilogue` and stores constant `None`.

**CN:** 将 `epilogue` and 保存常量 `None`.

### Block 116 — Lines 498-516 (_test_op)
```python
498|     if c_dtype.has_mx_scale:
499|         c_scale_shape = c_shape[:-1] + (triton.cdiv(c_shape[-1], c_dtype.microblock_size),)
500|         c_scale = torch.empty(c_scale_shape, dtype=c_dtype.scale_dtype, device=a.device)
501|         if c_hbm_swizzling:
502|             c_scale = wrap_torch_tensor(c_scale)
503|             c_ragged_metadata = a_ragged_metadata if mode == "ragged" else None
504|             c_scale = convert_layout(c_scale, layout.BlackwellActMXScaleLayout(c_ragged_metadata))
505|         precision_opt.c_mx_scale = c_scale
506|         precision_opt.c_microblock_size = c_dtype.microblock_size
507|         precision_opt.c_value_pack_factor = 2 if c_dtype.is_mxfloat4 else 1
508|         epilogue_spec = (
509|             FnSpecs(FnName.QUANTIZE_NVFP4.name, quantize_nvfp4_fn, (), ())
510|             if c_dtype.is_nvfp4
511|             else FnSpecs(FnName.QUANTIZE_MXFP8.name, quantize_mxfp8_fn, (), ())
512|         )
513|         epilogue = Epilogue(epilogue_spec, tuple(), tuple(), effective_itemsize=2.0 if c_dtype.is_nvfp4 else 6.0)
514| 
515| 
516|     # --- triton implementation ---
```
**EN:** Checks `c_dtype.has_mx_scale`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `c_dtype.has_mx_scale`. 真分支主要准备中间值; 准备中间值.

### Block 117 — Lines 517-531 (_test_op)
```python
517|     try:
518|         tri_y = matmul(a, b, bias,
519|                            a_ragged_metadata, b_ragged_metadata,
520|                            gather_indx, scatter_indx, precision_opt,
521|                            gammas=gammas, epilogue=epilogue, c=c,
522|                            fused_activation=fused_activation)
523|         if c_dtype.has_global_scale:
524|             tri_y_scale = precision_opt.flex_ctx.out_data.actual_scale.clone()
525|     except (opt_flags.InapplicableConstraint, NotImplementedError) as e:
526|         pytest.skip(f"inapplicable opt_flags constraint {e}")
527|     # --- torch implementation ---
528|     # Fused NVFP4 output quantizes the float32 activation result and applies
529|     # expected_scale inside downcast_to_mxfp_torch, so keep the reference in
530|     # float32 until that final downcast instead of letting matmul_torch
531|     # return bf16 and apply the output scale early.
```
**EN:** Wraps code that mainly prepares intermediate values; branches on runtime conditions with exception/finally handling.

**CN:** 包装代码 that mainly 准备中间值; 根据运行时条件分支 以及异常/收尾处理.

### Block 118 — Lines 532-547 (_test_op)
```python
532|     ref_y = matmul_torch(
533|         a.float() if c_dtype.is_nvfp4 and not a_dtype.is_nvfp4 else a,
534|         b.float() if c_dtype.is_nvfp4 and not b_dtype.is_nvfp4 else b,
535|         bias,
536|         a_ragged_metadata,
537|         b_ragged_metadata,
538|         gather_indx,
539|         scatter_indx,
540|         PrecisionConfig(
541|             a_mx_scale=a_scales,
542|             a_microblock_size=a_dtype.microblock_size,
543|             b_mx_scale=b_scale_tri,
544|             b_microblock_size=b_dtype.microblock_size,
545|         ) if c_dtype.is_nvfp4 else precision_opt,
546|         gammas=gammas,
547|     )
```
**EN:** Assigns `ref_y` and calls `matmul_torch`.

**CN:** 将 `ref_y`，并调用 `matmul_torch`.

### Block 119 — Lines 548-549 (_test_op)
```python
548|     if swiglu_opts is not None:
549|         ref_y = swiglu(ref_y, alpha=swiglu_opts[0], precision_config=SwiGLUPrecisionConfig(swiglu_opts[1]))
```
**EN:** Checks `swiglu_opts is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `swiglu_opts is not None`. 真分支主要准备中间值.

### Block 120 — Lines 550-553 (_test_op)
```python
550|     if c_dtype.has_global_scale:
551|         ref_y_scale = precision_opt.flex_ctx.out_data.actual_scale.clone()
552| 
553|     # --- check results ---
```
**EN:** Checks `c_dtype.has_global_scale`. The true branch mainly prepares intermediate values.

**CN:** 检查 `c_dtype.has_global_scale`. 真分支主要准备中间值.

### Block 121 — Lines 554-568 (_test_op)
```python
554|     if c_dtype.has_mx_scale:
555|         tri_y_scale = precision_opt.c_mx_scale
556|         if isinstance(tri_y_scale, Tensor):
557|             tri_y_scale = convert_layout(tri_y_scale, layout.StridedLayout()).storage.data
558|         tri_y = upcast_from_mxfp(tri_y, tri_y_scale, target_dtype=torch.bfloat16, axis=-1).to(ref_y.dtype)
559|         ref_target_dtype = ref_y.dtype
560|         ref_y, ref_scale = downcast_to_mxfp_torch(
561|             ref_y,
562|             c_dtype.torch_dtype,
563|             axis=-1,
564|             scale_dtype=c_dtype.scale_dtype,
565|             microblock_size=c_dtype.microblock_size,
566|             expected_scale=precision_opt.flex_ctx.out_data.expected_scale,
567|         )
568|         ref_y = upcast_from_mxfp_torch(ref_y, ref_scale, target_dtype=ref_target_dtype, axis=-1)
```
**EN:** Checks `c_dtype.has_mx_scale`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `c_dtype.has_mx_scale`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 122 — Lines 569-569 (_test_op)
```python
569|     maxtol, rmstol = None, None
```
**EN:** Assigns `maxtol`, `rmstol` and builds a tuple.

**CN:** 将 `maxtol`, `rmstol` and 构造一个元组.

### Block 123 — Lines 570-575 (_test_op)
```python
570|     if c_dtype.is_nvfp4 and a_dtype.is_nvfp4 and b_dtype.is_nvfp4:
571|         maxtol, rmstol = 6e-1, 4e-2
572|     elif c_dtype.has_mx_scale:
573|         maxtol, rmstol = 4e-1, 4e-2
574|     elif b_dtype.is_mxfloat4:
575|         maxtol, rmstol = 3e-2, None
```
**EN:** Checks `c_dtype.is_nvfp4 and a_dtype.is_nvfp4 and b_dtype.is_nvfp4`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `c_dtype.is_nvfp4 and a_dtype.is_nvfp4 and b_dtype.is_nvfp4`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 124 — Lines 576-576 (_test_op)
```python
576|     assert_close(ref_y, tri_y, maxtol=maxtol, rmstol=rmstol)
```
**EN:** Calls `assert_close` for side effects, registration, or validation.

**CN:** 调用 `assert_close` ，用于副作用、注册或校验。

### Block 125 — Lines 577-581 (_test_op)
```python
577|     if c_dtype.has_global_scale:
578|         assert torch.all((ref_y_scale - tri_y_scale).abs() < 1e-10), \
579|                f"ref_y_scale: {ref_y_scale}, tri_y_scale: {tri_y_scale.item()}"
580| 
581| 
```
**EN:** Checks `c_dtype.has_global_scale`. The true branch mainly checks invariants.

**CN:** 检查 `c_dtype.has_global_scale`. 真分支主要检查不变量.

### Block 126 — Lines 582-582 (test_set_idle_sms)
```python
582| def test_set_idle_sms():
```
**EN:** Defines function `test_set_idle_sms()` for this module. The body mainly branches on runtime conditions; prepares intermediate values; invokes `matmul_set_idle_sms`. It uses calls such as `matmul_set_idle_sms`, `make_opt_flags`, `is_cuda`, `pytest.skip`, `PrecisionConfig` to implement its workflow.

**CN:** 定义函数 `test_set_idle_sms()`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; invokes `matmul_set_idle_sms`. 其中会调用 `matmul_set_idle_sms`, `make_opt_flags`, `is_cuda`, `pytest.skip`, `PrecisionConfig` 来实现其工作流程.

### Block 127 — Lines 583-584 (test_set_idle_sms)
```python
583|     if not is_cuda():
584|         pytest.skip("Only supported on CUDA")
```
**EN:** Checks `not is_cuda()`. The true branch mainly invokes `pytest.skip`.

**CN:** 检查 `not is_cuda()`. 真分支主要invokes `pytest.skip`.

### Block 128 — Lines 585-585 (test_set_idle_sms)
```python
585|     from triton_kernels.matmul_details.opt_flags import make_opt_flags
```
**EN:** This block imports `triton_kernels.matmul_details.opt_flags (make_opt_flags)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `triton_kernels.matmul_details.opt_flags (make_opt_flags)` ，为模块提供所需的外部库和内部辅助工具。

### Block 129 — Lines 586-586 (test_set_idle_sms)
```python
586|     num_idle_sms = 24
```
**EN:** Assigns `num_idle_sms` and stores constant `24`.

**CN:** 将 `num_idle_sms` and 保存常量 `24`.

### Block 130 — Lines 587-587 (test_set_idle_sms)
```python
587|     matmul_set_idle_sms(num_idle_sms)
```
**EN:** Calls `matmul_set_idle_sms` for side effects, registration, or validation.

**CN:** 调用 `matmul_set_idle_sms` ，用于副作用、注册或校验。

### Block 131 — Lines 588-589 (test_set_idle_sms)
```python
588|     flags = make_opt_flags(FP32, FP32, FP32, PrecisionConfig(), \
589|                            1, 1024, 1024, 1024, None, True, False, 1, False, False, None)
```
**EN:** Assigns `flags` and calls `make_opt_flags`.

**CN:** 将 `flags`，并调用 `make_opt_flags`.

### Block 132 — Lines 590-590 (test_set_idle_sms)
```python
590|     assert flags.idle_sms == num_idle_sms
```
**EN:** Asserts `flags.idle_sms == num_idle_sms` to enforce invariants.

**CN:** 断言 `flags.idle_sms == num_idle_sms` 以确保不变量成立。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `DType`, `opt_flags_scope`, `make_constraints`, `Case`, `_build_test_op_cases`, `test_op`, `_test_op`, `test_set_idle_sms`.
  **CN:** 主要符号：`DType`, `opt_flags_scope`, `make_constraints`, `Case`, `_build_test_op_cases`, `test_op`, `_test_op`, `test_set_idle_sms`。
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
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** SwiGLU fusion or activation-specific handling is part of the computation flow.
  **CN:** 计算流程中包含 SwiGLU 融合或激活函数相关处理。

## Dependencies / 依赖关系
- **EN:** External modules: `dataclasses (dataclass, fields)`, `itertools`, `pytest`, `torch`, `typing (Union)`, `triton`, `triton._internal_testing (is_hopper)`.
  **CN:** 外部模块：`dataclasses (dataclass, fields)`, `itertools`, `pytest`, `torch`, `typing (Union)`, `triton`, `triton._internal_testing (is_hopper)`。
- **EN:** Internal modules: `triton_kernels.matmul_details.opt_flags`, `triton_kernels.matmul (FlexCtx, PrecisionConfig, FusedActivation, FnSpecs, FnName, Epilogue)`, `triton_kernels.matmul (matmul_set_idle_sms, matmul, matmul_torch)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.numerics_details.mxfp (upcast_from_mxfp, quantize_mxfp8_fn, quantize_nvfp4_fn, downcast_to_mxfp_torch, upcast_from_mxfp_torch, MXFP_BLOCK_SIZE)`, `triton_kernels.testing (assert_close, make_random_tensor)`, `triton_kernels.target_info (is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_gfx1250)`, `triton_kernels.swiglu (swiglu, swiglu_fn)`, `triton_kernels.swiglu (PrecisionConfig)`, `triton_kernels.tensor_details (layout)`, `triton_kernels.tensor (Tensor, convert_layout, wrap_torch_tensor)`, `triton_kernels.tensor_details.dtype (FP32)`.
  **CN:** 内部模块：`triton_kernels.matmul_details.opt_flags`, `triton_kernels.matmul (FlexCtx, PrecisionConfig, FusedActivation, FnSpecs, FnName, Epilogue)`, `triton_kernels.matmul (matmul_set_idle_sms, matmul, matmul_torch)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.numerics_details.mxfp (upcast_from_mxfp, quantize_mxfp8_fn, quantize_nvfp4_fn, downcast_to_mxfp_torch, upcast_from_mxfp_torch, MXFP_BLOCK_SIZE)`, `triton_kernels.testing (assert_close, make_random_tensor)`, `triton_kernels.target_info (is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_gfx1250)`, `triton_kernels.swiglu (swiglu, swiglu_fn)`, `triton_kernels.swiglu (PrecisionConfig)`, `triton_kernels.tensor_details (layout)`, `triton_kernels.tensor (Tensor, convert_layout, wrap_torch_tensor)`, `triton_kernels.tensor_details.dtype (FP32)`。
