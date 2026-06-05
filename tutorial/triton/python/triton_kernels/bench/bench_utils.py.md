# bench_utils.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/bench/bench_utils.py`
- **Purpose / 用途:** Benchmark driver for bench utils; it measures performance, sweeps configurations, and reports results for Triton kernels. / 用于 bench utils 的基准测试驱动；它会测量性能、扫描配置，并报告 Triton 内核的结果。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-14 (module)
```python
 1| from copy import deepcopy
 2| from dataclasses import dataclass
 3| 
 4| import triton_kernels
 5| import triton_kernels.swiglu
 6| from triton_kernels.matmul import PrecisionConfig, FlexCtx, FnSpecs, FusedActivation
 7| from triton_kernels.numerics import InFlexData
 8| from triton_kernels.numerics_details.mxfp import MXFP_BLOCK_SIZE, downcast_to_mxfp
 9| from triton_kernels.tensor import convert_layout, wrap_torch_tensor, FP4, Tensor
10| from triton_kernels.target_info import is_cuda, get_cdna_version, cuda_capability_geq, is_hip
11| from triton_kernels.tensor_details import layout
12| import torch
13| 
14| 
```
**EN:** This block imports `copy (deepcopy)`, `dataclasses (dataclass)`, `triton_kernels`, `triton_kernels.swiglu`, `triton_kernels.matmul (PrecisionConfig, FlexCtx, FnSpecs, FusedActivation)`, `triton_kernels.numerics (InFlexData)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, Tensor)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `copy (deepcopy)`, `dataclasses (dataclass)`, `triton_kernels`, `triton_kernels.swiglu`, `triton_kernels.matmul (PrecisionConfig, FlexCtx, FnSpecs, FusedActivation)`, `triton_kernels.numerics (InFlexData)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, Tensor)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 15-15 (_quantize_weight)
```python
15| def _quantize_weight(w, dtype, **opt):
```
**EN:** Defines function `_quantize_weight(w, dtype, **opt)` for this module. The body mainly branches on runtime conditions. It uses calls such as `w.to(torch.bfloat16).transpose(-1, -2...`, `InFlexData`, `w.to`, `downcast_to_mxfp`, `is_cuda` to implement its workflow.

**CN:** 定义函数 `_quantize_weight(w, dtype, **opt)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `w.to(torch.bfloat16).transpose(-1, -2...`, `InFlexData`, `w.to`, `downcast_to_mxfp`, `is_cuda` 来实现其工作流程.

### Block 3 — Lines 16-33 (_quantize_weight)
```python
16|     if dtype == "bf16":
17|         wq = w.to(torch.bfloat16).transpose(-1, -2).contiguous().transpose(-1, -2)
18|         return wq, InFlexData(), None
19|     elif dtype == "fp8":
20|         fp8e4_dtype = torch.float8_e4m3fn if get_cdna_version() != 3 else torch.float8_e4m3fnuz
21|         wq = w.to(fp8e4_dtype)
22|         if is_cuda() and not cuda_capability_geq(10, 0):
23|             wq = wq.transpose(-1, -2).contiguous().transpose(-1, -2)
24|         return wq, InFlexData(dtype=wq.dtype, scale=w.abs().max().unsqueeze(0)), None
25|     else:
26|         assert dtype == "mx4", f"{dtype=}"
27|         w, w_scale = downcast_to_mxfp(w.to(torch.bfloat16), torch.uint8, axis=1)
28|         if opt:
29|             w = convert_layout(wrap_torch_tensor(w, dtype=FP4), opt["value_layout"])
30|             w_scale = convert_layout(wrap_torch_tensor(w_scale), opt["scale_layout"])
31|         return w, InFlexData(), w_scale
32| 
33| 
```
**EN:** Checks `dtype == 'bf16'`. The true branch mainly prepares intermediate values; returns the computed result, while the else branch branches on runtime conditions.

**CN:** 检查 `dtype == 'bf16'`. 真分支主要准备中间值; 返回计算结果；而 else 分支根据运行时条件分支.

### Block 4 — Lines 34-35 (MlpNumerics)
```python
34| @dataclass
35| class MlpNumerics:
```
**EN:** Defines class `MlpNumerics` with decorators `dataclass` to organize related behavior. Key fields include `wg`, `w1`, `w2`, `pcg`, `pc1`, `pc2`, `activation`.

**CN:** 定义类 `MlpNumerics`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `wg`, `w1`, `w2`, `pcg`, `pc1`, `pc2`, `activation`.

### Block 5 — Lines 36-36 (MlpNumerics)
```python
36|     wg: torch.Tensor | Tensor | None
```
**EN:** Annotated assignment stores `wg` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `wg` and 声明该带注解的字段.

### Block 6 — Lines 37-37 (MlpNumerics)
```python
37|     w1: torch.Tensor | Tensor | None
```
**EN:** Annotated assignment stores `w1` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `w1` and 声明该带注解的字段.

### Block 7 — Lines 38-38 (MlpNumerics)
```python
38|     w2: torch.Tensor | Tensor | None
```
**EN:** Annotated assignment stores `w2` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `w2` and 声明该带注解的字段.

### Block 8 — Lines 39-39 (MlpNumerics)
```python
39|     pcg: PrecisionConfig
```
**EN:** Annotated assignment stores `pcg` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `pcg` and 声明该带注解的字段.

### Block 9 — Lines 40-40 (MlpNumerics)
```python
40|     pc1: PrecisionConfig
```
**EN:** Annotated assignment stores `pc1` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `pc1` and 声明该带注解的字段.

### Block 10 — Lines 41-41 (MlpNumerics)
```python
41|     pc2: PrecisionConfig
```
**EN:** Annotated assignment stores `pc2` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `pc2` and 声明该带注解的字段.

### Block 11 — Lines 42-44 (MlpNumerics)
```python
42|     activation: FusedActivation
43| 
44| 
```
**EN:** Annotated assignment stores `activation` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `activation` and 声明该带注解的字段.

### Block 12 — Lines 45-45 (_make_default_mlp_activation)
```python
45| def _make_default_mlp_activation() -> FusedActivation:
```
**EN:** Defines function `_make_default_mlp_activation()` for this module. The body mainly returns the computed result. It uses calls such as `FusedActivation`, `FnSpecs` to implement its workflow.

**CN:** 定义函数 `_make_default_mlp_activation()`，供本模块使用. 主体主要返回计算结果. 其中会调用 `FusedActivation`, `FnSpecs` 来实现其工作流程.

### Block 13 — Lines 46-51 (_make_default_mlp_activation)
```python
46|     return FusedActivation(
47|         FnSpecs("swiglu", triton_kernels.swiglu.swiglu_fn, ("alpha", "limit"), reduction_n=2),
48|         (1.0, 1.0),
49|     )
50| 
51| 
```
**EN:** Returns `FusedActivation(FnSpecs('swiglu', triton_kernels.swiglu.swiglu_fn, ('alpha', ...`.

**CN:** 返回 `FusedActivation(FnSpecs('swiglu', triton_kernels.swiglu.swiglu_fn, ('alpha', ...`.

### Block 14 — Lines 52-52 (_make_mx4_quantization_opts)
```python
52| def _make_mx4_quantization_opts(batch: int, w_dtype: str) -> dict:
```
**EN:** Defines function `_make_mx4_quantization_opts(batch, w_dtype)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `layout.make_default_matmul_mxfp4_w_la...`, `layout.make_default_matmul_mxfp4_w_sc...`, `is_hip`, `cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `_make_mx4_quantization_opts(batch, w_dtype)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `layout.make_default_matmul_mxfp4_w_la...`, `layout.make_default_matmul_mxfp4_w_sc...`, `is_hip`, `cuda_capability_geq` 来实现其工作流程.

### Block 15 — Lines 53-54 (_make_mx4_quantization_opts)
```python
53|     if w_dtype != "mx4" or is_hip():
54|         return {}
```
**EN:** Checks `w_dtype != 'mx4' or is_hip()`. The true branch mainly returns the computed result.

**CN:** 检查 `w_dtype != 'mx4' or is_hip()`. 真分支主要返回计算结果.

### Block 16 — Lines 55-55 (_make_mx4_quantization_opts)
```python
55|     num_warps = 4 if batch <= 512 and cuda_capability_geq(10, 0) else 8
```
**EN:** Assigns `num_warps` and uses conditional expression `4 if batch <= 512 and cuda_capability_geq(10, 0) else 8`.

**CN:** 将 `num_warps` and 使用条件表达式 `4 if batch <= 512 and cuda_capability_geq(10, 0) else 8`.

### Block 17 — Lines 56-56 (_make_mx4_quantization_opts)
```python
56|     value_layout = layout.make_default_matmul_mxfp4_w_layout(mx_axis=-2)
```
**EN:** Assigns `value_layout` and calls `layout.make_default_matmul_mxfp4_w_layout`.

**CN:** 将 `value_layout`，并调用 `layout.make_default_matmul_mxfp4_w_layout`.

### Block 18 — Lines 57-57 (_make_mx4_quantization_opts)
```python
57|     scale_layout = layout.make_default_matmul_mxfp4_w_scale_layout(mx_axis=-2, num_warps=num_warps)
```
**EN:** Assigns `scale_layout` and calls `layout.make_default_matmul_mxfp4_w_scale_layout`.

**CN:** 将 `scale_layout`，并调用 `layout.make_default_matmul_mxfp4_w_scale_layout`.

### Block 19 — Lines 58-63 (_make_mx4_quantization_opts)
```python
58|     return {
59|         "value_layout": value_layout,
60|         "scale_layout": scale_layout,
61|     }
62| 
63| 
```
**EN:** Returns `{'value_layout': value_layout, 'scale_layout': scale_layout}`.

**CN:** 返回 `{'value_layout': value_layout, 'scale_layout': scale_layout}`.

### Block 20 — Lines 64-64 (prepare_mlp_numerics)
```python
64| def prepare_mlp_numerics(batch: int, w_dtype: str, wg, w1, w2) -> MlpNumerics:
```
**EN:** Defines function `prepare_mlp_numerics(batch, w_dtype, wg, w1, w2)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `_make_mx4_quantization_opts`, `_quantize_weight`, `_make_default_mlp_activation`, `MlpNumerics`, `deepcopy` to implement its workflow.

**CN:** 定义函数 `prepare_mlp_numerics(batch, w_dtype, wg, w1, w2)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `_make_mx4_quantization_opts`, `_quantize_weight`, `_make_default_mlp_activation`, `MlpNumerics`, `deepcopy` 来实现其工作流程.

### Block 21 — Lines 65-65 (prepare_mlp_numerics)
```python
65|     quantization_opts = _make_mx4_quantization_opts(batch, w_dtype)
```
**EN:** Assigns `quantization_opts` and calls `_make_mx4_quantization_opts`.

**CN:** 将 `quantization_opts`，并调用 `_make_mx4_quantization_opts`.

### Block 22 — Lines 66-66 (prepare_mlp_numerics)
```python
66|     wg, wg_flex, wg_scale = _quantize_weight(wg, "bf16")
```
**EN:** Assigns `wg`, `wg_flex`, `wg_scale` and calls `_quantize_weight`.

**CN:** 将 `wg`, `wg_flex`, `wg_scale`，并调用 `_quantize_weight`.

### Block 23 — Lines 67-67 (prepare_mlp_numerics)
```python
67|     w1, w1_flex, w1_scale = _quantize_weight(w1, w_dtype, **deepcopy(quantization_opts))
```
**EN:** Assigns `w1`, `w1_flex`, `w1_scale` and calls `_quantize_weight`.

**CN:** 将 `w1`, `w1_flex`, `w1_scale`，并调用 `_quantize_weight`.

### Block 24 — Lines 68-68 (prepare_mlp_numerics)
```python
68|     w2, w2_flex, w2_scale = _quantize_weight(w2, w_dtype, **deepcopy(quantization_opts))
```
**EN:** Assigns `w2`, `w2_flex`, `w2_scale` and calls `_quantize_weight`.

**CN:** 将 `w2`, `w2_flex`, `w2_scale`，并调用 `_quantize_weight`.

### Block 25 — Lines 69-69 (prepare_mlp_numerics)
```python
69|     activation = _make_default_mlp_activation()
```
**EN:** Assigns `activation` and calls `_make_default_mlp_activation`.

**CN:** 将 `activation`，并调用 `_make_default_mlp_activation`.

### Block 26 — Lines 70-92 (prepare_mlp_numerics)
```python
70|     return MlpNumerics(
71|         wg=wg,
72|         w1=w1,
73|         w2=w2,
74|         pcg=PrecisionConfig(
75|             flex_ctx=FlexCtx(rhs_data=wg_flex),
76|             b_mx_scale=wg_scale,
77|             b_microblock_size=MXFP_BLOCK_SIZE.value,
78|         ),
79|         pc1=PrecisionConfig(
80|             flex_ctx=FlexCtx(rhs_data=w1_flex),
81|             b_mx_scale=w1_scale,
82|             b_microblock_size=MXFP_BLOCK_SIZE.value,
83|         ),
84|         pc2=PrecisionConfig(
85|             flex_ctx=FlexCtx(rhs_data=w2_flex),
86|             b_mx_scale=w2_scale,
87|             b_microblock_size=MXFP_BLOCK_SIZE.value,
88|         ),
89|         activation=activation,
90|     )
91| 
92| 
```
**EN:** Returns `MlpNumerics(wg=wg, w1=w1, w2=w2, pcg=PrecisionConfig(flex_ctx=FlexCtx(rhs_dat...`.

**CN:** 返回 `MlpNumerics(wg=wg, w1=w1, w2=w2, pcg=PrecisionConfig(flex_ctx=FlexCtx(rhs_dat...`.

### Block 27 — Lines 93-93 (resolve_x_dtype)
```python
93| def resolve_x_dtype(x_dtype: str) -> torch.dtype:
```
**EN:** Defines function `resolve_x_dtype(x_dtype)` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `get_cdna_version` to implement its workflow.

**CN:** 定义函数 `resolve_x_dtype(x_dtype)`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `get_cdna_version` 来实现其工作流程.

### Block 28 — Lines 94-94 (resolve_x_dtype)
```python
94|     dtype_map = {"fp16": torch.float16, "bf16": torch.bfloat16, "fp8": torch.float8_e4m3fn}
```
**EN:** Assigns `dtype_map` and builds a dictionary.

**CN:** 将 `dtype_map` and 构造一个字典.

### Block 29 — Lines 95-95 (resolve_x_dtype)
```python
95|     dtype = dtype_map[x_dtype]
```
**EN:** Assigns `dtype` and evaluates `dtype_map[x_dtype]`.

**CN:** 将 `dtype` and 计算 `dtype_map[x_dtype]`.

### Block 30 — Lines 96-97 (resolve_x_dtype)
```python
96|     if dtype == torch.float8_e4m3fn and get_cdna_version() == 3:
97|         return torch.float8_e4m3fnuz
```
**EN:** Checks `dtype == torch.float8_e4m3fn and get_cdna_version() == 3`. The true branch mainly returns the computed result.

**CN:** 检查 `dtype == torch.float8_e4m3fn and get_cdna_version() == 3`. 真分支主要返回计算结果.

### Block 31 — Lines 98-98 (resolve_x_dtype)
```python
98|     return dtype
```
**EN:** Returns `dtype`.

**CN:** 返回 `dtype`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `_quantize_weight`, `MlpNumerics`, `_make_default_mlp_activation`, `_make_mx4_quantization_opts`, `prepare_mlp_numerics`, `resolve_x_dtype`.
  **CN:** 主要符号：`_quantize_weight`, `MlpNumerics`, `_make_default_mlp_activation`, `_make_mx4_quantization_opts`, `prepare_mlp_numerics`, `resolve_x_dtype`。
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
- **EN:** SwiGLU fusion or activation-specific handling is part of the computation flow.
  **CN:** 计算流程中包含 SwiGLU 融合或激活函数相关处理。
- **EN:** Benchmark-oriented code sweeps parameter spaces and records performance metrics.
  **CN:** 面向基准测试的代码会扫描参数空间并记录性能指标。

## Dependencies / 依赖关系
- **EN:** External modules: `copy (deepcopy)`, `dataclasses (dataclass)`, `torch`.
  **CN:** 外部模块：`copy (deepcopy)`, `dataclasses (dataclass)`, `torch`。
- **EN:** Internal modules: `triton_kernels`, `triton_kernels.swiglu`, `triton_kernels.matmul (PrecisionConfig, FlexCtx, FnSpecs, FusedActivation)`, `triton_kernels.numerics (InFlexData)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, Tensor)`, `triton_kernels.target_info (is_cuda, get_cdna_version, cuda_capability_geq, is_hip)`, `triton_kernels.tensor_details (layout)`.
  **CN:** 内部模块：`triton_kernels`, `triton_kernels.swiglu`, `triton_kernels.matmul (PrecisionConfig, FlexCtx, FnSpecs, FusedActivation)`, `triton_kernels.numerics (InFlexData)`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, downcast_to_mxfp)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4, Tensor)`, `triton_kernels.target_info (is_cuda, get_cdna_version, cuda_capability_geq, is_hip)`, `triton_kernels.tensor_details (layout)`。
