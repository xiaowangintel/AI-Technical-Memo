# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/compressed_tensors/transform/linear.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CompressedTensorsLinearTransformMethod` for quantization backends, schemes, and utilities. / 实现 `CompressedTensorsLinearTransformMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-27)
```python
from collections.abc import Callable, Generator
from itertools import accumulate

import torch
from compressed_tensors.transform import (
    TransformArgs,
    TransformConfig,
    TransformLocation,
    TransformScheme,
)
from compressed_tensors.utils import is_match

from vllm.model_executor.layers.linear import (
    WEIGHT_LOADER_V2_SUPPORTED,
    LinearMethodBase,
)
from vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors import (  # noqa: E501
    CompressedTensorsScheme,
)
from vllm.model_executor.layers.quantization.compressed_tensors.transform.module import (  # noqa: E501
    HadamardTransform,
)
from vllm.model_executor.layers.quantization.compressed_tensors.transform.utils import (  # noqa: E501
    TransformTuple,
)
```
**EN:** This opening block pulls in external dependencies such as `collections`, `itertools`, `torch`, `compressed_tensors` and internal modules such as `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.module`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `itertools`, `torch`, `compressed_tensors`）以及内部模块（如 `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.module`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `get_linear_transform_schemes` (lines 193-224)
```python
def get_linear_transform_schemes(
    layer: torch.nn.Module,
    layer_name: str,
    transform_config: TransformConfig | None,
    packed_modules_mapping: dict[str, list[str]],
) -> tuple[
    dict[int, TransformTuple], dict[int, TransformTuple]
]:  # [input_transform, [output_transform, ...]]
    # there can only be one transform input scheme per (fused) module
    input_tfms = {}
    output_tfms = {}

    partition_names = get_layer_partition_names(layer_name, packed_modules_mapping)

    for scheme_name, scheme, args in get_schemes_args(transform_config):
        for part_index, part_name in enumerate(partition_names):
            if (
                is_match(part_name, layer, args.targets, args.ignore)
                and args.is_online()
            ):
                if args.location == TransformLocation.INPUT:
                    input_tfms[part_index] = TransformTuple(scheme_name, scheme, args)

                elif args.location == TransformLocation.OUTPUT:
                    output_tfms[part_index] = TransformTuple(scheme_name, scheme, args)

                else:
                    raise ValueError(
                        f"Cannot apply `{args.location}` transform to `{layer_name}`"
                    )

    return (input_tfms, output_tfms)
```
**EN:** Defines function `get_linear_transform_schemes` with signature `get_linear_transform_schemes(layer: torch.nn.Module, layer_name: str, transform_config: TransformConfig | None, packed_modules_mapping: dict[str, list[str]]) -> tuple[dict[int, TransformTuple], dict[int, TransformTuple]]`. It mainly works with `layer`, `layer_name`, `transform_config`, `packed_modules_mapping`; returns a derived property or capability check. The body uses branching, iteration, validation/error handling. Key calls include `get_layer_partition_names`, `get_schemes_args`, `enumerate`, `is_match`, `args.is_online`, `TransformTuple`.
**CN:** 定义函数 `get_linear_transform_schemes`，其签名为 `get_linear_transform_schemes(layer: torch.nn.Module, layer_name: str, transform_config: TransformConfig | None, packed_modules_mapping: dict[str, list[str]]) -> tuple[dict[int, TransformTuple], dict[int, TransformTuple]]`。它主要围绕 `layer`, `layer_name`, `transform_config`, `packed_modules_mapping` 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `get_layer_partition_names`, `get_schemes_args`, `enumerate`, `is_match`, `args.is_online`, `TransformTuple`。

### Function `get_schemes_args` (lines 227-235)
```python
def get_schemes_args(
    transform_config: TransformConfig | None,
) -> Generator[tuple[str, TransformScheme, TransformArgs]]:
    if transform_config is None:
        return

    for scheme_name, scheme in transform_config.config_groups.items():
        for args in scheme.apply:
            yield (scheme_name, scheme, args)
```
**EN:** Defines function `get_schemes_args` with signature `get_schemes_args(transform_config: TransformConfig | None) -> Generator[tuple[str, TransformScheme, TransformArgs]]`. It mainly works with `transform_config`; returns a derived property or capability check. The body uses branching, iteration. Key calls include `transform_config.config_groups.items`.
**CN:** 定义函数 `get_schemes_args`，其签名为 `get_schemes_args(transform_config: TransformConfig | None) -> Generator[tuple[str, TransformScheme, TransformArgs]]`。它主要围绕 `transform_config` 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理。关键调用包括 `transform_config.config_groups.items`。

### Function `get_layer_partition_names` (lines 238-260)
```python
def get_layer_partition_names(
    layer_name: str, packed_modules_mapping: dict[str, list[str]]
) -> list[str]:
    """
    Get all partition names associated with this layer.
    Names are returned in order of their partition indices.

    ```python
    mapping = {"gate_up_proj", "gate_proj", "up_proj"}

    assert get_layer_partition_names("mlp.gate_up_proj", mapping) == [
        "gate_proj",
        "up_proj",
    ]
    assert get_layer_partition_names("mlp.down_proj", mapping) == ["down_proj"]"""
    for fused_suffix, part_suffixes in packed_modules_mapping.items():
        if layer_name.endswith(fused_suffix):
            return [
                layer_name.removesuffix(fused_suffix) + part_suffix
                for part_suffix in part_suffixes
            ]

    return [layer_name]
```
**EN:** Defines function `get_layer_partition_names` with signature `get_layer_partition_names(layer_name: str, packed_modules_mapping: dict[str, list[str]]) -> list[str]`. It mainly works with `layer_name`, `packed_modules_mapping`; returns a derived property or capability check. The body uses branching, iteration, comprehensions. Key calls include `packed_modules_mapping.items`, `layer_name.endswith`, `layer_name.removesuffix`.
**CN:** 定义函数 `get_layer_partition_names`，其签名为 `get_layer_partition_names(layer_name: str, packed_modules_mapping: dict[str, list[str]]) -> list[str]`。它主要围绕 `layer_name`, `packed_modules_mapping` 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理、推导式。关键调用包括 `packed_modules_mapping.items`, `layer_name.endswith`, `layer_name.removesuffix`。

### Class `CompressedTensorsLinearTransformMethod` overview (lines 30-190)
```python
class CompressedTensorsLinearTransformMethod(LinearMethodBase):
    """
    Wraps `CompressedTensorsLinearMethod` or `UnquantizedLinearMethod` and adds
    input and output transforms to either side of the original apply method
    """

    @classmethod
    def from_schemes(
        cls,
        quant_method: LinearMethodBase,
        quant_scheme: CompressedTensorsScheme | None,
        input_tfms: dict[int, TransformTuple],
        output_tfms: dict[int, TransformTuple],
    ) -> "CompressedTensorsLinearTransformMethod":
        from vllm.model_executor.layers.quantization.compressed_tensors.transform.schemes.linear_qutlass_nvfp4 import (  # noqa: E501
            QutlassNvFP4LinearMethod,
            is_qutlass_fp4_scheme,
        )

        assert input_tfms or output_tfms

        if is_qutlass_fp4_scheme(quant_scheme, input_tfms):
            return QutlassNvFP4LinearMethod(quant_method, input_tfms, output_tfms)

        # hadacore or dense gemm is selected by Transform module
```
**EN:** Defines class `CompressedTensorsLinearTransformMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 6 direct methods, with notable entries `from_schemes`, `__init__`, `create_weights`, `process_weights_after_loading`, `apply`, `_validate_tfm_schemes`. Its docstring says: Wraps `CompressedTensorsLinearMethod` or `UnquantizedLinearMethod` and adds input and output transforms to either side of the original apply method
**CN:** 定义类 `CompressedTensorsLinearTransformMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 6 个方法，较重要的包括 `from_schemes`, `__init__`, `create_weights`, `process_weights_after_loading`, `apply`, `_validate_tfm_schemes`。 文档字符串进一步说明了该类的定位。

### Method `CompressedTensorsLinearTransformMethod.from_schemes` (lines 37-56)
```python
    def from_schemes(
        cls,
        quant_method: LinearMethodBase,
        quant_scheme: CompressedTensorsScheme | None,
        input_tfms: dict[int, TransformTuple],
        output_tfms: dict[int, TransformTuple],
    ) -> "CompressedTensorsLinearTransformMethod":
        from vllm.model_executor.layers.quantization.compressed_tensors.transform.schemes.linear_qutlass_nvfp4 import (  # noqa: E501
            QutlassNvFP4LinearMethod,
            is_qutlass_fp4_scheme,
        )

        assert input_tfms or output_tfms

        if is_qutlass_fp4_scheme(quant_scheme, input_tfms):
            return QutlassNvFP4LinearMethod(quant_method, input_tfms, output_tfms)

        # hadacore or dense gemm is selected by Transform module

        return cls(quant_method, input_tfms, output_tfms)
```
**EN:** Defines function `CompressedTensorsLinearTransformMethod.from_schemes` with signature `from_schemes(cls, quant_method: LinearMethodBase, quant_scheme: CompressedTensorsScheme | None, input_tfms: dict[int, TransformTuple], output_tfms: dict[int, TransformTuple]) -> 'CompressedTensorsLinearTransformMethod'`. It mainly works with `quant_method`, `quant_scheme`, `input_tfms`, `output_tfms`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `is_qutlass_fp4_scheme`, `cls`, `QutlassNvFP4LinearMethod`.
**CN:** 定义函数 `CompressedTensorsLinearTransformMethod.from_schemes`，其签名为 `from_schemes(cls, quant_method: LinearMethodBase, quant_scheme: CompressedTensorsScheme | None, input_tfms: dict[int, TransformTuple], output_tfms: dict[int, TransformTuple]) -> 'CompressedTensorsLinearTransformMethod'`。它主要围绕 `quant_method`, `quant_scheme`, `input_tfms`, `output_tfms` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `is_qutlass_fp4_scheme`, `cls`, `QutlassNvFP4LinearMethod`。

### Method `CompressedTensorsLinearTransformMethod.__init__` (lines 58-69)
```python
    def __init__(
        self,
        quant_method: LinearMethodBase,
        input_tfms: dict[int, TransformTuple],
        output_tfms: dict[int, TransformTuple],
    ):
        self.quant_method = quant_method
        self.input_tfms = input_tfms
        self.output_tfms = output_tfms

        self.input_transform: HadamardTransform | None = None
        self.output_transform: HadamardTransform | None = None
```
**EN:** Defines function `CompressedTensorsLinearTransformMethod.__init__` with signature `__init__(self, quant_method: LinearMethodBase, input_tfms: dict[int, TransformTuple], output_tfms: dict[int, TransformTuple])`. It mainly works with `quant_method`, `input_tfms`, `output_tfms`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `CompressedTensorsLinearTransformMethod.__init__`，其签名为 `__init__(self, quant_method: LinearMethodBase, input_tfms: dict[int, TransformTuple], output_tfms: dict[int, TransformTuple])`。它主要围绕 `quant_method`, `input_tfms`, `output_tfms` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `CompressedTensorsLinearTransformMethod.create_weights` (lines 71-140)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        # get weight loader for transforms
        weight_loader: Callable = extra_weight_attrs.get("weight_loader")  # type: ignore[assignment]

        # HACK: UnquantizedLinearMethod does not support weight loader v2, but
        # transforms (specifically SharedWeightParameter) requires
        # weight loader v2. Until UnquantizedLinearMethod supports v2, we must
        # hack around this by getting weight loader v1 so ULM can load correctly
        quant_method_name = self.quant_method.__class__.__name__
        if quant_method_name not in WEIGHT_LOADER_V2_SUPPORTED:
            weight_loader_v1 = layer.weight_loader
            extra_weight_attrs["weight_loader"] = weight_loader_v1

        self.quant_method.create_weights(
            layer=layer,
            input_size_per_partition=input_size_per_partition,
            output_partition_sizes=output_partition_sizes,
            input_size=input_size,
            output_size=output_size,
            params_dtype=params_dtype,
            **extra_weight_attrs,
        )

        # validate schemes
        num_partitions = len(output_partition_sizes)
        self._validate_tfm_schemes(num_partitions)

        # create submodules for weight loading
        if len(self.input_tfms) > 0:
# ... truncated for analysis ...

            transform = HadamardTransform(
                self.output_tfms,
                layer,
                weight_loader,
                input_size_per_partition,
                output_partition_sizes,
            )
            layer.register_module(transform_name, transform)
            self.output_transform = transform

        # compute partition ranges for slicing activations
        starts = [0] + list(accumulate(output_partition_sizes))[:-1]
        self.partition_ranges = list(zip(starts, output_partition_sizes))
```
**EN:** Defines function `CompressedTensorsLinearTransformMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching. Key calls include `extra_weight_attrs.get`, `self.quant_method.create_weights`, `len`, `self._validate_tfm_schemes`, `list`, `HadamardTransform`.
**CN:** 定义函数 `CompressedTensorsLinearTransformMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断。关键调用包括 `extra_weight_attrs.get`, `self.quant_method.create_weights`, `len`, `self._validate_tfm_schemes`, `list`, `HadamardTransform`。

### Method `CompressedTensorsLinearTransformMethod.process_weights_after_loading` (lines 142-147)
```python
    def process_weights_after_loading(self, layer):
        self.quant_method.process_weights_after_loading(layer)

        for submodule in layer.children():
            if isinstance(submodule, HadamardTransform):
                submodule.process_weights_after_loading()
```
**EN:** Defines function `CompressedTensorsLinearTransformMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer)`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration. Key calls include `self.quant_method.process_weights_after_loading`, `layer.children`, `isinstance`, `submodule.process_weights_after_loading`.
**CN:** 定义函数 `CompressedTensorsLinearTransformMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer)`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理。关键调用包括 `self.quant_method.process_weights_after_loading`, `layer.children`, `isinstance`, `submodule.process_weights_after_loading`。

### Method `CompressedTensorsLinearTransformMethod.apply` (lines 149-169)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if self.input_transform is not None:
            x = self.input_transform(x)

        assert bias is None
        x = self.quant_method.apply(layer, x, bias)

        # In most cases, input transforms are preferred over output transforms
        # (@ksayers): confirm that this is done concurrently
        if self.output_transform is not None:
            for part_id, (start, length) in enumerate(self.partition_ranges):
                x[:, start : start + length] = self.output_transform(
                    x[:, start : start + length].clone(), part_id=part_id
                )

        return x
```
**EN:** Defines function `CompressedTensorsLinearTransformMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling. Key calls include `self.quant_method.apply`, `self.input_transform`, `enumerate`, `self.output_transform`, `x.clone`.
**CN:** 定义函数 `CompressedTensorsLinearTransformMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `self.quant_method.apply`, `self.input_transform`, `enumerate`, `self.output_transform`, `x.clone`。

### Method `CompressedTensorsLinearTransformMethod._validate_tfm_schemes` (lines 171-190)
```python
    def _validate_tfm_schemes(self, num_partitions: int):
        if len(self.input_tfms) > 0:
            if 0 not in self.input_tfms:
                raise ValueError("Must have same input")

            for part_index in range(num_partitions):
                if self.input_tfms[part_index] != self.input_tfms[0]:
                    raise ValueError("Must have same input")

        if len(self.output_tfms) > 0:
            scheme_name = list(self.output_tfms.values())[0].scheme_name
            location = list(self.output_tfms.values())[0].args.location

            for tfm in self.output_tfms.values():
                if tfm.scheme_name != scheme_name:
                    raise ValueError("Must have same scheme name")
                if tfm.args.location != location:
                    raise ValueError("Must have same location")

        return self.input_tfms, self.output_tfms
```
**EN:** Defines function `CompressedTensorsLinearTransformMethod._validate_tfm_schemes` with signature `_validate_tfm_schemes(self, num_partitions: int)`. It mainly works with `num_partitions`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, validation/error handling. Key calls include `len`, `range`, `self.output_tfms.values`, `ValueError`, `list`.
**CN:** 定义函数 `CompressedTensorsLinearTransformMethod._validate_tfm_schemes`，其签名为 `_validate_tfm_schemes(self, num_partitions: int)`。它主要围绕 `num_partitions` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `len`, `range`, `self.output_tfms.values`, `ValueError`, `list`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `CompressedTensorsLinearTransformMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `CompressedTensorsLinearTransformMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_linear_transform_schemes`, `get_schemes_args`, `get_layer_partition_names` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_linear_transform_schemes`, `get_schemes_args`, `get_layer_partition_names` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `itertools`, `torch`, `compressed_tensors`
- **Internal / 内部**: `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.module`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.utils`
