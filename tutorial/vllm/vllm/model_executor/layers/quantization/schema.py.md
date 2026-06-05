# schema.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/schema.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines schemas or backend schemes for quantization backends, schemes, and utilities. / 定义量化后端、方案与工具的模式或后端方案。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-14)
```python
"""
This file contains the Pydantic schemas for various quantization-related
parameters. When a relevant quantization technique is specified, these
parameters are loaded in the form of a JSON alongside the model weights
and augment the model with additional information needed for use of that
technique. The format of this JSON should be specified by one or more
schemas contained here.

For example, when the KV cache is quantized to FP8-E4M3 (currently only
possible on ROCm), the model can be optionally augmented with KV cache
scaling factors.
"""
```
**EN:** This docstring gives the module author's high-level intent: This file contains the Pydantic schemas for various quantization-related parameters. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：This file contains the Pydantic schemas for various quantization-related parameters. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 16-16)
```python
from pydantic import BaseModel, ConfigDict, ValidationInfo, model_validator
```
**EN:** This opening block pulls in external dependencies such as `pydantic` and internal modules such as no internal imports. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `pydantic`）以及内部模块（如 no internal imports）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `KVCacheQuantSchema` overview (lines 19-69)
```python
class KVCacheQuantSchema(BaseModel):
    dtype: str
    # Each key is a TP rank. Each value is a dictionary mapping a TP rank's
    # layer indices to their per-tensor KV cache scaling factor.
    # TODO: Consider pulling this and its validation methods out into its
    # own schema class (tricky as its members are variable)
    scaling_factor: dict[int, dict[int, float]]

    @model_validator(mode="after")
    def check_is_fp8(self) -> "KVCacheQuantSchema":
        assert self.dtype == "float8_e4m3fn", (
            "Loaded scaling factors intended for KV cache dtype = "
            f"{self.dtype} rather than float8_e4m3fn!"
        )
        return self

    @model_validator(mode="after")
    def check_tp_ranks(self, info: ValidationInfo) -> "KVCacheQuantSchema":
        context = info.context
        if context:
            tp_size = context["tp_size"]
            num_hidden_layers = context["num_hidden_layers"]
            assert len(self.scaling_factor) == tp_size, (
                f"Loaded dictionary has TP size {len(self.scaling_factor)} "
                f"but LLM engine is currently running with TP size {tp_size}."
```
**EN:** Defines class `KVCacheQuantSchema` with base classes `BaseModel` and decorators none. It acts as a quantization-oriented module building block and exposes 3 direct methods, with notable entries `check_is_fp8`, `check_tp_ranks`, `check_current_rank`.
**CN:** 定义类 `KVCacheQuantSchema`，其基类为 `BaseModel`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 3 个方法，较重要的包括 `check_is_fp8`, `check_tp_ranks`, `check_current_rank`。

### Method `KVCacheQuantSchema.check_is_fp8` (lines 28-33)
```python
    def check_is_fp8(self) -> "KVCacheQuantSchema":
        assert self.dtype == "float8_e4m3fn", (
            "Loaded scaling factors intended for KV cache dtype = "
            f"{self.dtype} rather than float8_e4m3fn!"
        )
        return self
```
**EN:** Defines function `KVCacheQuantSchema.check_is_fp8` with signature `check_is_fp8(self) -> 'KVCacheQuantSchema'`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `model_validator`.
**CN:** 定义函数 `KVCacheQuantSchema.check_is_fp8`，其签名为 `check_is_fp8(self) -> 'KVCacheQuantSchema'`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `model_validator`。

### Method `KVCacheQuantSchema.check_tp_ranks` (lines 36-55)
```python
    def check_tp_ranks(self, info: ValidationInfo) -> "KVCacheQuantSchema":
        context = info.context
        if context:
            tp_size = context["tp_size"]
            num_hidden_layers = context["num_hidden_layers"]
            assert len(self.scaling_factor) == tp_size, (
                f"Loaded dictionary has TP size {len(self.scaling_factor)} "
                f"but LLM engine is currently running with TP size {tp_size}."
            )
            for tp_rank, layer_maps in self.scaling_factor.items():
                assert len(layer_maps) == num_hidden_layers, (
                    f"KV cache scales map for TP rank {tp_rank} is malformed. "
                    f"Expected {num_hidden_layers} layers, got "
                    f"{len(layer_maps)}."
                )
            for i in range(tp_size):
                assert i in self.scaling_factor, (
                    f"KV cache scales map for TP rank {i} not found."
                )
        return self
```
**EN:** Defines function `KVCacheQuantSchema.check_tp_ranks` with signature `check_tp_ranks(self, info: ValidationInfo) -> 'KVCacheQuantSchema'`. It mainly works with `info`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling. Key calls include `model_validator`, `self.scaling_factor.items`, `range`, `len`.
**CN:** 定义函数 `KVCacheQuantSchema.check_tp_ranks`，其签名为 `check_tp_ranks(self, info: ValidationInfo) -> 'KVCacheQuantSchema'`。它主要围绕 `info` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `model_validator`, `self.scaling_factor.items`, `range`, `len`。

### Method `KVCacheQuantSchema.check_current_rank` (lines 58-69)
```python
    def check_current_rank(self, info: ValidationInfo) -> "KVCacheQuantSchema":
        context = info.context
        if context:
            tp_rank = context["tp_rank"]
            num_hidden_layers = context["num_hidden_layers"]
            layer_scales_map = self.scaling_factor[tp_rank]
            for i in range(num_hidden_layers):
                assert i in layer_scales_map, (
                    f"Could not find KV cache scales for layer {i} in "
                    f"TP rank {tp_rank}."
                )
        return self
```
**EN:** Defines function `KVCacheQuantSchema.check_current_rank` with signature `check_current_rank(self, info: ValidationInfo) -> 'KVCacheQuantSchema'`. It mainly works with `info`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling. Key calls include `model_validator`, `range`.
**CN:** 定义函数 `KVCacheQuantSchema.check_current_rank`，其签名为 `check_current_rank(self, info: ValidationInfo) -> 'KVCacheQuantSchema'`。它主要围绕 `info` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `model_validator`, `range`。

### Class `QuantParamSchema` overview (lines 72-90)
```python
class QuantParamSchema(BaseModel):
    # TODO: Generalize and extend with more fields
    # (e.g. weights/activations params) once functionality is enabled
    model_config = ConfigDict(protected_namespaces=())
    model_type: str | None
    kv_cache: KVCacheQuantSchema

    @model_validator(mode="after")
    def check_model_type(self, info: ValidationInfo) -> "QuantParamSchema":
        context = info.context
        if context:
            model_type = context.get("model_type", None)
            if model_type is not None:
                assert model_type == self.model_type, (
                    f"Model type is {model_type} but loaded "
                    f"scaling factors belonging to different "
                    f"model type {self.model_type}!"
                )
        return self
```
**EN:** Defines class `QuantParamSchema` with base classes `BaseModel` and decorators none. It acts as a quantization-oriented module building block and exposes 1 direct methods, with notable entries `check_model_type`.
**CN:** 定义类 `QuantParamSchema`，其基类为 `BaseModel`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 1 个方法，较重要的包括 `check_model_type`。

### Method `QuantParamSchema.check_model_type` (lines 80-90)
```python
    def check_model_type(self, info: ValidationInfo) -> "QuantParamSchema":
        context = info.context
        if context:
            model_type = context.get("model_type", None)
            if model_type is not None:
                assert model_type == self.model_type, (
                    f"Model type is {model_type} but loaded "
                    f"scaling factors belonging to different "
                    f"model type {self.model_type}!"
                )
        return self
```
**EN:** Defines function `QuantParamSchema.check_model_type` with signature `check_model_type(self, info: ValidationInfo) -> 'QuantParamSchema'`. It mainly works with `info`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `model_validator`, `context.get`.
**CN:** 定义函数 `QuantParamSchema.check_model_type`，其签名为 `check_model_type(self, info: ValidationInfo) -> 'QuantParamSchema'`。它主要围绕 `info` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `model_validator`, `context.get`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `KVCacheQuantSchema`, `QuantParamSchema`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `KVCacheQuantSchema`, `QuantParamSchema`，它们组织了主要的可复用抽象。

## Dependencies / 依赖关系
- **External / 外部**: `pydantic`
- **Internal / 内部**: None / 无
