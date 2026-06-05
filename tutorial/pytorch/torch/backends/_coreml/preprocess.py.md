# preprocess.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/backends/_coreml/preprocess.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Exposes backend-specific switches, capability probes, and preprocessing helpers for optional runtimes.
- **Purpose (CN)**: 暴露后端相关的开关、能力探测以及面向可选运行时的预处理辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行
````python
# mypy: allow-untyped-defs
import hashlib
import json

import coremltools as ct  # type: ignore[import]
from coremltools.converters.mil.input_types import TensorType  # type: ignore[import]
from coremltools.converters.mil.mil import types  # type: ignore[import]
from coremltools.models.neural_network import quantization_utils  # type: ignore[import]

import torch
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as hashlib, json; other helper packages such as coremltools, coremltools.converters.mil.input_types, coremltools.converters.mil.mil, ....
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 hashlib、json；其他辅助包，如 coremltools、coremltools.converters.mil.input_types、coremltools.converters.mil.mil、...。

### Lines 13-22 / 第 13-22 行
````python
CT_METADATA_VERSION = "com.github.apple.coremltools.version"
CT_METADATA_SOURCE = "com.github.apple.coremltools.source"


class ScalarType:
    Float = 0
    Double = 1
    Int = 2
    Long = 3
    Undefined = 4
````
- **EN**: It introduces or extends `ScalarType`, which hold the main object-oriented state for this portion of the file.
- **CN**: 它引入或扩展了 `ScalarType`，这些类承载了本段涉及的主要面向对象状态。

### Lines 25-32 / 第 25-32 行
````python
# Supported Tensor types in coremltools:
# https://github.com/apple/coremltools/blob/main/coremltools/converters/mil/frontend/torch/converter.py#L28
torch_to_mil_types = {
    ScalarType.Float: types.fp32,
    ScalarType.Double: types.fp64,
    ScalarType.Int: types.int32,
    ScalarType.Long: types.int64,
}
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 35-44 / 第 35-44 行
````python
class CoreMLComputeUnit:
    CPU = "cpuOnly"
    CPUAndGPU = "cpuAndGPU"
    ALL = "all"


class CoreMLQuantizationMode:
    LINEAR = "linear"
    LINEAR_SYMMETRIC = "linear_symmetric"
    NONE = "none"
````
- **EN**: It introduces or extends `CoreMLComputeUnit`, `CoreMLQuantizationMode`, which hold the main object-oriented state for this portion of the file.
- **CN**: 它引入或扩展了 `CoreMLComputeUnit`、`CoreMLQuantizationMode`，这些类承载了本段涉及的主要面向对象状态。

### Lines 47-60 / 第 47-60 行
````python
def TensorSpec(shape, dtype=ScalarType.Float):
    return (shape, dtype)


def CompileSpec(
    inputs,
    outputs,
    backend=CoreMLComputeUnit.CPU,
    allow_low_precision=True,
    quantization_mode=CoreMLQuantizationMode.NONE,
    mlmodel_export_path=None,
    convert_to=None,
):
    return (
````
- **EN**: This chunk defines `CompileSpec`, which coordinates graph-oriented state needed for tracing, capture, or compilation. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `CompileSpec`，其作用是协调 tracing、捕获或编译所需的图相关状态。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 61-68 / 第 61-68 行
````python
        inputs,
        outputs,
        backend,
        allow_low_precision,
        quantization_mode,
        mlmodel_export_path,
        convert_to,
    )
````
- **EN**: This chunk continues `CompileSpec` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `CompileSpec`，进一步展开其内部控制流或状态更新。

### Lines 71-84 / 第 71-84 行
````python
def _check_enumerated_shape(shape):
    for s in shape:
        if not isinstance(s, (list, tuple)):
            return False
    return True


def _convert_to_mil_type(shape, dtype, name: str):
    mil_shape = shape
    if _check_enumerated_shape(shape):
        mil_shape = ct.EnumeratedShapes(shape)
    ml_type = TensorType(shape=mil_shape, dtype=torch_to_mil_types[dtype])
    ml_type.name = name
    return ml_type
````
- **EN**: This chunk defines `_convert_to_mil_type`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_convert_to_mil_type`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 87-100 / 第 87-100 行
````python
def preprocess(script_module: torch._C.ScriptObject, compile_spec: dict[str, tuple]):
    spec = compile_spec["forward"]
    (
        input_specs,
        output_specs,
        backend,
        allow_low_precision,
        quantization_mode,
        mlmodel_export_path,
        convert_to,
    ) = spec
    mil_inputs = []
    inputs = []
    for index, input in enumerate(input_specs):
````
- **EN**: This chunk defines `preprocess`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `preprocess`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 101-113 / 第 101-113 行
````python
        shape, dtype = input
        name = "input_" + str(index)
        inputs.append([name, str(dtype), str(shape)])
        ml_type = _convert_to_mil_type(shape, dtype, name)
        mil_inputs.append(ml_type)
    model = torch.jit.RecursiveScriptModule._construct(script_module, lambda x: None)
    mlmodel = ct.convert(model, inputs=mil_inputs, convert_to=convert_to)

    if quantization_mode != CoreMLQuantizationMode.NONE:
        quant_model_spec = quantization_utils.quantize_weights(
            mlmodel, nbits=8, quantization_mode=quantization_mode
        )
        mlmodel = ct.models.MLModel(quant_model_spec)
````
- **EN**: This chunk continues `preprocess` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `preprocess`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 115-127 / 第 115-127 行
````python
    spec = mlmodel.get_spec()
    if len(spec.description.output) != len(output_specs):  # type: ignore[attr-defined]
        raise AssertionError(
            f"Number of outputs in spec ({len(spec.description.output)}) "  # type: ignore[attr-defined]
            f"does not match output_specs ({len(output_specs)})"
        )
    outputs = []
    for index, output in enumerate(output_specs):
        shape, dtype = output
        name = spec.description.output[index].name  # type: ignore[attr-defined]
        outputs.append([name, str(dtype), str(shape)])
    mlmodel = ct.models.model.MLModel(spec)
    print(mlmodel)
````
- **EN**: This chunk continues `preprocess` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `preprocess`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 129-142 / 第 129-142 行
````python
    if mlmodel_export_path is not None:
        print(f"Saving CoreML .mlmodel file to {mlmodel_export_path}")
        mlmodel.save(mlmodel_export_path)

    config = {
        "spec_ver": str(spec.specificationVersion),  # type: ignore[attr-defined]
        "backend": backend,
        "allow_low_precision": str(allow_low_precision),
    }
    metadata = {
        "coremltool_ver": mlmodel.user_defined_metadata[CT_METADATA_VERSION],
        "torch_ver": mlmodel.user_defined_metadata[CT_METADATA_SOURCE],
    }
    coreml_compile_spec = {
````
- **EN**: This chunk continues `preprocess` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `preprocess`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 143-154 / 第 143-154 行
````python
        "inputs": inputs,
        "outputs": outputs,
        "config": config,
        "metadata": metadata,
    }
    mlmodel = spec.SerializeToString()  # type: ignore[attr-defined]

    return {
        "model": mlmodel,
        "hash": str(hashlib.sha256(mlmodel).hexdigest()),
        "extra": json.dumps(coreml_compile_spec),
    }
````
- **EN**: This chunk continues `preprocess` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `preprocess`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Backend configuration**
  - EN: Exposes optional-runtime toggles and capability checks that alter low-level execution behavior.
  - CN: 暴露可选运行时开关与能力检查，以改变底层执行行为。
- **CT_METADATA_VERSION**
  - EN: `CT_METADATA_VERSION` is one of the main symbols declared or implemented in this file.
  - CN: `CT_METADATA_VERSION` 是本文件声明或实现的主要符号之一。
- **CT_METADATA_SOURCE**
  - EN: `CT_METADATA_SOURCE` is one of the main symbols declared or implemented in this file.
  - CN: `CT_METADATA_SOURCE` 是本文件声明或实现的主要符号之一。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`
- **Standard library / 标准库**: `hashlib`, `json`
- **Other helper packages / 其他辅助包**: `coremltools`, `coremltools.converters.mil.input_types`, `coremltools.converters.mil.mil`, `coremltools.models.neural_network`
- **Primary symbols in this file / 本文件核心符号**: `CT_METADATA_VERSION`, `CT_METADATA_SOURCE`, `ScalarType`, `CoreMLComputeUnit`, `CoreMLQuantizationMode`, `TensorSpec`, `CompileSpec`, `_check_enumerated_shape`, `_convert_to_mil_type`, `preprocess`
