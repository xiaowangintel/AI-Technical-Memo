# swiglu.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/swiglu.py`
- **Purpose / 用途:** Implementation module for swiglu; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols FlexCtx, PrecisionConfig, SwiGLU, swiglu, swiglu_torch. / 用于 swiglu 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 FlexCtx、PrecisionConfig、SwiGLU、swiglu、swiglu_torch。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-8 (module)
```python
1| from dataclasses import dataclass
2| from triton_kernels.numerics import InFlexData, OutFlexData
3| import torch
4| import triton
5| from .swiglu_details._swiglu import _swiglu, _swiglu_fn
6| from triton_kernels import target_info
7| 
8| 
```
**EN:** This block imports `dataclasses (dataclass)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `torch`, `triton`, `.swiglu_details._swiglu (_swiglu, _swiglu_fn)`, `triton_kernels (target_info)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `torch`, `triton`, `.swiglu_details._swiglu (_swiglu, _swiglu_fn)`, `triton_kernels (target_info)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 9-10 (FlexCtx)
```python
 9| @dataclass(frozen=True)
10| class FlexCtx:
```
**EN:** Defines class `FlexCtx` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `out_data`, `inp_data`, `saturate_inf`.

**CN:** 定义类 `FlexCtx`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `out_data`, `inp_data`, `saturate_inf`.

### Block 3 — Lines 11-11 (FlexCtx)
```python
11|     out_data: OutFlexData = OutFlexData()
```
**EN:** Annotated assignment stores `out_data` and calls `OutFlexData`.

**CN:** 带类型注解的赋值保存 `out_data`，并调用 `OutFlexData`.

### Block 4 — Lines 12-12 (FlexCtx)
```python
12|     inp_data: InFlexData = InFlexData()
```
**EN:** Annotated assignment stores `inp_data` and calls `InFlexData`.

**CN:** 带类型注解的赋值保存 `inp_data`，并调用 `InFlexData`.

### Block 5 — Lines 13-15 (FlexCtx)
```python
13|     saturate_inf: bool = False
14| 
15| 
```
**EN:** Annotated assignment stores `saturate_inf` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `saturate_inf` and 保存常量 `False`.

### Block 6 — Lines 16-17 (PrecisionConfig)
```python
16| @dataclass(frozen=True)
17| class PrecisionConfig:
```
**EN:** Defines class `PrecisionConfig` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `limit`, `flex_ctx`.

**CN:** 定义类 `PrecisionConfig`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `limit`, `flex_ctx`.

### Block 7 — Lines 18-18 (PrecisionConfig)
```python
18|     limit: float
```
**EN:** Annotated assignment stores `limit` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `limit` and 声明该带注解的字段.

### Block 8 — Lines 19-21 (PrecisionConfig)
```python
19|     flex_ctx: FlexCtx = FlexCtx()
20| 
21| 
```
**EN:** Annotated assignment stores `flex_ctx` and calls `FlexCtx`.

**CN:** 带类型注解的赋值保存 `flex_ctx`，并调用 `FlexCtx`.

### Block 9 — Lines 22-24 (module)
```python
22| swiglu_fn = _swiglu_fn
23| 
24| 
```
**EN:** Assigns `swiglu_fn` and references `_swiglu_fn`.

**CN:** 将 `swiglu_fn` and 引用 `_swiglu_fn`.

### Block 10 — Lines 25-26 (SwiGLU)
```python
25| class SwiGLU(torch.autograd.Function):
26| 
```
**EN:** Defines class `SwiGLU` inheriting from `torch.autograd.Function` to organize related behavior. Main methods are `forward`.

**CN:** 定义类 `SwiGLU`，继承自 `torch.autograd.Function`，用于组织相关行为。主要方法有 `forward`.

### Block 11 — Lines 27-28 (forward)
```python
27|     @staticmethod
28|     def forward(ctx, a, alpha, precision_config, routing_data):
```
**EN:** Defines function `forward(ctx, a, alpha, precision_config, routing_data)` with decorators `staticmethod` for this module. The body mainly prepares intermediate values; prepares intermediate values; checks invariants. It uses calls such as `torch.empty`, `triton.cdiv`, `target_info.num_sms`, `_swiglu[grid]`, `out.view` to implement its workflow.

**CN:** 定义函数 `forward(ctx, a, alpha, precision_config, routing_data)`，带有装饰器 `staticmethod`，供本模块使用. 主体主要准备中间值; 准备中间值; 检查不变量. 其中会调用 `torch.empty`, `triton.cdiv`, `target_info.num_sms`, `_swiglu[grid]`, `out.view` 来实现其工作流程.

### Block 12 — Lines 29-29 (forward)
```python
29|         N = a.shape[-1]
```
**EN:** Assigns `N` and evaluates `a.shape[-1]`.

**CN:** 将 `N` and 计算 `a.shape[-1]`.

### Block 13 — Lines 30-30 (forward)
```python
30|         M = a.numel() // N
```
**EN:** Assigns `M` and evaluates `a.numel() // N`.

**CN:** 将 `M` and 计算 `a.numel() // N`.

### Block 14 — Lines 31-31 (forward)
```python
31|         assert a.stride()[-1] == 1
```
**EN:** Asserts `a.stride()[-1] == 1` to enforce invariants.

**CN:** 断言 `a.stride()[-1] == 1` 以确保不变量成立。

### Block 15 — Lines 32-32 (forward)
```python
32|         assert a.shape[-1] % 2 == 0
```
**EN:** Asserts `a.shape[-1] % 2 == 0` to enforce invariants.

**CN:** 断言 `a.shape[-1] % 2 == 0` 以确保不变量成立。

### Block 16 — Lines 33-33 (forward)
```python
33|         out = torch.empty(size=(M, N // 2), dtype=a.dtype, device=a.device)
```
**EN:** Assigns `out` and calls `torch.empty`.

**CN:** 将 `out`，并调用 `torch.empty`.

### Block 17 — Lines 34-35 (forward)
```python
34|         flex_ctx = precision_config.flex_ctx
35|         # optimization hyperparameters
```
**EN:** Assigns `flex_ctx` and references `precision_config.flex_ctx`.

**CN:** 将 `flex_ctx` and 引用 `precision_config.flex_ctx`.

### Block 18 — Lines 36-36 (forward)
```python
36|         BLOCK_M, BLOCK_N = 32 // a.itemsize, 128
```
**EN:** Assigns `BLOCK_M`, `BLOCK_N` and builds a tuple.

**CN:** 将 `BLOCK_M`, `BLOCK_N` and 构造一个元组.

### Block 19 — Lines 37-37 (forward)
```python
37|         num_warps = 4
```
**EN:** Assigns `num_warps` and stores constant `4`.

**CN:** 将 `num_warps` and 保存常量 `4`.

### Block 20 — Lines 38-39 (forward)
```python
38|         kwargs = {'maxnreg': 64} if not target_info.is_hip() else {}
39|         # launch semi-persistent kernel
```
**EN:** Assigns `kwargs` and uses conditional expression `{'maxnreg': 64} if not target_info.is_hip() else {}`.

**CN:** 将 `kwargs` and 使用条件表达式 `{'maxnreg': 64} if not target_info.is_hip() else {}`.

### Block 21 — Lines 40-40 (forward)
```python
40|         N_BLOCKS = triton.cdiv(N // 2, BLOCK_N)
```
**EN:** Assigns `N_BLOCKS` and calls `triton.cdiv`.

**CN:** 将 `N_BLOCKS`，并调用 `triton.cdiv`.

### Block 22 — Lines 41-41 (forward)
```python
41|         num_sms = target_info.num_sms()
```
**EN:** Assigns `num_sms` and calls `target_info.num_sms`.

**CN:** 将 `num_sms`，并调用 `target_info.num_sms`.

### Block 23 — Lines 42-52 (forward)
```python
42|         if routing_data is not None:
43|             waves_per_sm = 32 if target_info.is_hip() else 128
44|             num_pid = num_sms * (waves_per_sm // num_warps)
45|             M_BLOCKS = max(1, triton.cdiv(num_pid, N_BLOCKS))
46|             grid = (min(M_BLOCKS * N_BLOCKS, 4 * num_sms), )
47|         else:
48|             M_BLOCKS = triton.cdiv(M, BLOCK_M)
49|             if M_BLOCKS * N_BLOCKS >= 8 * num_sms:
50|                 grid = (8 * num_sms, )
51|             else:
52|                 grid = (min(M_BLOCKS * N_BLOCKS, 4 * num_sms), )
```
**EN:** Checks `routing_data is not None`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `routing_data is not None`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值; 根据运行时条件分支.

### Block 24 — Lines 53-53 (forward)
```python
53|         n_tokens = None
```
**EN:** Assigns `n_tokens` and stores constant `None`.

**CN:** 将 `n_tokens` and 保存常量 `None`.

### Block 25 — Lines 54-55 (forward)
```python
54|         if routing_data is not None:
55|             n_tokens = routing_data.expt_data.token_offs[routing_data.n_expts_tot]
```
**EN:** Checks `routing_data is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `routing_data is not None`. 真分支主要准备中间值.

### Block 26 — Lines 56-80 (forward)
```python
56|         _swiglu[grid](
57|             flex_ctx.out_data.reinterpret(out),
58|             flex_ctx.out_data.expected_scale,
59|             flex_ctx.out_data.actual_scale,
60|             flex_ctx.out_data.checksum_scale,
61|             flex_ctx.inp_data.reinterpret(a),
62|             flex_ctx.inp_data.scale,
63|             alpha,
64|             M,
65|             N // 2,
66|             a.shape[-1],
67|             1,
68|             out.shape[-1],
69|             1,
70|             precision_config.limit,
71|             n_tokens,
72|             BLOCK_M=BLOCK_M,
73|             BLOCK_N=BLOCK_N,
74|             EVEN_N=(N // 2) % BLOCK_N == 0,
75|             M_BLOCKS=M_BLOCKS,
76|             N_BLOCKS=N_BLOCKS,
77|             flexpoint_saturate_inf=flex_ctx.saturate_inf,
78|             num_warps=num_warps,
79|             **kwargs,
80|         )
```
**EN:** Calls `_swiglu[grid]` for side effects, registration, or validation.

**CN:** 调用 `_swiglu[grid]` ，用于副作用、注册或校验。

### Block 27 — Lines 81-81 (forward)
```python
81|         out = out.view(a.shape[:-1] + out.shape[-1:])
```
**EN:** Assigns `out` and calls `out.view`.

**CN:** 将 `out`，并调用 `out.view`.

### Block 28 — Lines 82-84 (forward)
```python
82|         return out
83| 
84| 
```
**EN:** Returns `out`.

**CN:** 返回 `out`.

### Block 29 — Lines 85-85 (swiglu)
```python
85| def swiglu(a, alpha, precision_config, routing_data=None):
```
**EN:** Defines function `swiglu(a, alpha, precision_config, routing_data)` for this module. The body mainly returns the computed result. It uses calls such as `SwiGLU.apply` to implement its workflow.

**CN:** 定义函数 `swiglu(a, alpha, precision_config, routing_data)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `SwiGLU.apply` 来实现其工作流程.

### Block 30 — Lines 86-88 (swiglu)
```python
86|     return SwiGLU.apply(a, alpha, precision_config, routing_data)
87| 
88| 
```
**EN:** Returns `SwiGLU.apply(a, alpha, precision_config, routing_data)`.

**CN:** 返回 `SwiGLU.apply(a, alpha, precision_config, routing_data)`.

### Block 31 — Lines 89-89 (swiglu_torch)
```python
89| def swiglu_torch(a, alpha, precision_config):
```
**EN:** Defines function `swiglu_torch(a, alpha, precision_config)` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `a_gelu.clamp`, `a_linear.clamp`, `torch.sigmoid` to implement its workflow.

**CN:** 定义函数 `swiglu_torch(a, alpha, precision_config)`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `a_gelu.clamp`, `a_linear.clamp`, `torch.sigmoid` 来实现其工作流程.

### Block 32 — Lines 90-90 (swiglu_torch)
```python
90|     limit = precision_config.limit
```
**EN:** Assigns `limit` and references `precision_config.limit`.

**CN:** 将 `limit` and 引用 `precision_config.limit`.

### Block 33 — Lines 91-91 (swiglu_torch)
```python
91|     a_gelu = a[..., ::2]
```
**EN:** Assigns `a_gelu` and evaluates `a[..., ::2]`.

**CN:** 将 `a_gelu` and 计算 `a[..., ::2]`.

### Block 34 — Lines 92-93 (swiglu_torch)
```python
92|     if limit is not None:
93|         a_gelu = a_gelu.clamp(max=limit)
```
**EN:** Checks `limit is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `limit is not None`. 真分支主要准备中间值.

### Block 35 — Lines 94-94 (swiglu_torch)
```python
94|     a_linear = a[..., 1::2]
```
**EN:** Assigns `a_linear` and evaluates `a[..., 1::2]`.

**CN:** 将 `a_linear` and 计算 `a[..., 1::2]`.

### Block 36 — Lines 95-97 (swiglu_torch)
```python
95|     if limit is not None:
96|         a_linear = a_linear.clamp(min=-limit, max=limit)
97| 
```
**EN:** Checks `limit is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `limit is not None`. 真分支主要准备中间值.

### Block 37 — Lines 98-98 (swiglu_torch)
```python
98|     out_gelu = a_gelu * torch.sigmoid(alpha * a_gelu)
```
**EN:** Assigns `out_gelu` and evaluates `a_gelu * torch.sigmoid(alpha * a_gelu)`.

**CN:** 将 `out_gelu` and 计算 `a_gelu * torch.sigmoid(alpha * a_gelu)`.

### Block 38 — Lines 99-99 (swiglu_torch)
```python
99|     out = out_gelu * (a_linear + 1)
```
**EN:** Assigns `out` and evaluates `out_gelu * (a_linear + 1)`.

**CN:** 将 `out` and 计算 `out_gelu * (a_linear + 1)`.

### Block 39 — Lines 100-100 (swiglu_torch)
```python
100|     return out
```
**EN:** Returns `out`.

**CN:** 返回 `out`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `FlexCtx`, `PrecisionConfig`, `SwiGLU`, `swiglu`, `swiglu_torch`.
  **CN:** 主要符号：`FlexCtx`, `PrecisionConfig`, `SwiGLU`, `swiglu`, `swiglu_torch`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Flex-point style metadata or scaling is used to control quantized arithmetic.
  **CN:** 使用 Flex-point 风格的元数据或缩放来控制量化计算。
- **EN:** SwiGLU fusion or activation-specific handling is part of the computation flow.
  **CN:** 计算流程中包含 SwiGLU 融合或激活函数相关处理。

## Dependencies / 依赖关系
- **EN:** External modules: `dataclasses (dataclass)`, `torch`, `triton`.
  **CN:** 外部模块：`dataclasses (dataclass)`, `torch`, `triton`。
- **EN:** Internal modules: `triton_kernels.numerics (InFlexData, OutFlexData)`, `.swiglu_details._swiglu (_swiglu, _swiglu_fn)`, `triton_kernels (target_info)`.
  **CN:** 内部模块：`triton_kernels.numerics (InFlexData, OutFlexData)`, `.swiglu_details._swiglu (_swiglu, _swiglu_fn)`, `triton_kernels (target_info)`。
