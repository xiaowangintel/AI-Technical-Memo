# autoheuristic_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/autoheuristic/autoheuristic_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module contains heuristic models and helpers for autotuning decisions. It defines classes such as `AHFeature`, `AHOperation`, `AHContext`, and `AHMetadata`. It exposes functions such as `get_metadata_str_from_log`, `check_minsize`, `pad_mm_precondition`, `get_mixedmm_precondition`, `get_mult_dims_ops`, `get_arith_intensity`, and `...+12`.
- **用途（中文）**: 该模块包含自动调优决策所需的启发式模型与辅助逻辑。其中定义了 `AHFeature`、`AHOperation`、`AHContext`、`AHMetadata` 等类。同时提供 `get_metadata_str_from_log`、`check_minsize`、`pad_mm_precondition`、`get_mixedmm_precondition`、`get_mult_dims_ops`、`get_arith_intensity`、`另有12项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import functools
from collections.abc import Callable
from typing import Any

import torch


Feedback = float
Choice = str
Value = Any

CHOICE_COL = "choice"
FEEDBACK_COL = "feedback"


class AHFeature:
    """
    The context, that AutoHeuristic stores, is a list of features. AutoHeuristic needs to know whether a feature is
    categorical (i.e., not a continuous variable) to learn a machine learning model.
    """
````
- **EN**: Imports dependencies such as `functools`, `collections.abc`, `typing`, and `torch` for the logic in this range. Introduces class `AHFeature`. Initializes or updates values such as `Feedback`, `Choice`, `Value`, `CHOICE_COL`, and `FEEDBACK_COL`.
- **CN**: 这里导入了 `functools`、`collections.abc`、`typing`、`torch` 等依赖，为后续逻辑提供基础能力。这里定义了类`AHFeature`。初始化或更新了 `Feedback`、`Choice`、`Value`、`CHOICE_COL`、`FEEDBACK_COL` 等值。

### Lines 21-40 / 第 21-40 行
````python

    def __init__(self, name: str, value: Value, is_categorical: bool = False) -> None:
        self.name = name
        self.value = value
        self.is_categorical = is_categorical


class AHOperation:
    """
    AHOperation can be used to augment the data collected by AutoHeuristic.
    One might for example store features like m, k, n, but also want to use
    features like m*n, or k*n, to learn a heuristic. Instead of storing features
    that can be created from the collected data, one can use AHOperation to
    create new features from the collected data.
    """

    def __init__(
        self, name: str, func: Callable[[Any], Value], is_categorical: bool = False
    ) -> None:
        self.name = name
````
- **EN**: Introduces function `__init__`, class `AHOperation`, function `__init__`.
- **CN**: 这里定义了函数`__init__`、类`AHOperation`、函数`__init__`。

### Lines 41-60 / 第 41-60 行
````python
        self.func = func
        self.is_categorical = is_categorical

    def apply_operation(self, data: Any) -> None:
        data[self.name] = self.func(data)


class AHContext:
    """
    This class is used to specify which information AutoHeuristic should store. For each choice, AutoHeursitic will
    store the context and the collected feedback. The context could be something like the shape of a tensor, i.e.,
    information that will help to learn a heuristic.
    """

    features: list[AHFeature]
    context_dict: dict[str, Value]

    def __init__(self) -> None:
        self.features = []
        self.context_dict = {}
````
- **EN**: Introduces function `apply_operation`, class `AHContext`, function `__init__`. Initializes or updates values such as `features`, and `context_dict`.
- **CN**: 这里定义了函数`apply_operation`、类`AHContext`、函数`__init__`。初始化或更新了 `features`、`context_dict` 等值。

### Lines 61-80 / 第 61-80 行
````python

    def add_feature(
        self, name: str, value: Value, is_categorical: bool = False
    ) -> None:
        self.features.append(AHFeature(name, value, is_categorical=is_categorical))
        self.context_dict[name] = value

    def get_numerical_and_categorical_features(self) -> tuple[list[str], list[str]]:
        numerical_features = []
        categorical_features = []
        for feature in self.features:
            if feature.is_categorical:
                categorical_features.append(feature.name)
            else:
                numerical_features.append(feature.name)

        return numerical_features, categorical_features

    def get_feature_names_csv(self) -> str:
        return ",".join(feature.name for feature in self.features)
````
- **EN**: Introduces function `add_feature`, function `get_numerical_and_categorical_features`, function `get_feature_names_csv`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `numerical_features`, `categorical_features`, and `else`.
- **CN**: 这里定义了函数`add_feature`、函数`get_numerical_and_categorical_features`、函数`get_feature_names_csv`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `numerical_features`、`categorical_features`、`else` 等值。

### Lines 81-100 / 第 81-100 行
````python

    def get_feature_values_csv(self) -> str:
        return ",".join(str(feature.value) for feature in self.features)

    def get_value(self, name: str) -> Value:
        return self.context_dict[name]

    def apply_operations(self, operations: list[AHOperation]) -> None:
        for op in operations:
            op.apply_operation(self.context_dict)


class AHMetadata:
    def __init__(
        self,
        shared_memory: Any,
        device_capa: tuple[int, int],
        choices: list[Choice],
        name: str,
    ) -> None:
````
- **EN**: Introduces function `get_feature_values_csv`, function `get_value`, function `apply_operations`, class `AHMetadata`, function `__init__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shared_memory`, `device_capa`, `choices`, and `name`.
- **CN**: 这里定义了函数`get_feature_values_csv`、函数`get_value`、函数`apply_operations`、类`AHMetadata`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `shared_memory`、`device_capa`、`choices`、`name` 等值。

### Lines 101-120 / 第 101-120 行
````python
        # use amount of shared_memory and device_capability to identify GPU
        # TODO(AlnisM): there might be a better way to do this
        self.shared_memory = shared_memory
        self.device_capa = device_capa
        self.choices = choices
        self.name = name

    def to_dict(self) -> dict[str, Value]:
        return {
            "shared_memory": self.shared_memory,
            "device_capa": self.device_capa,
            "name": self.name,
        }


def get_metadata_str_from_log(log_path: str) -> str:
    with open(log_path, newline="") as file:
        json_string = file.readline().strip()
        return json_string

````
- **EN**: Introduces function `to_dict`, function `get_metadata_str_from_log`. Touches the filesystem to load, validate, or store compiler artifacts. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`to_dict`、函数`get_metadata_str_from_log`。这一段会访问文件系统，用于加载、校验或保存编译产物。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python

def check_minsize(context: AHContext, minsize: int) -> bool:
    return (
        context.get_value("m") >= minsize
        and context.get_value("k") >= minsize
        and context.get_value("n") >= minsize
    )


def pad_mm_precondition(metadata: AHMetadata, context: AHContext) -> bool:
    if metadata.shared_memory == 166912 and metadata.device_capa == (8, 0):
        # A100 precondition
        return check_minsize(context, 512)
    elif metadata.shared_memory == 232448 and metadata.device_capa == (9, 0):
        # H100 precondition
        return check_minsize(context, 768)
    return True


def get_mixedmm_precondition(metadata: AHMetadata, context: AHContext) -> bool:
````
- **EN**: Introduces function `check_minsize`, function `pad_mm_precondition`, function `get_mixedmm_precondition`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`check_minsize`、函数`pad_mm_precondition`、函数`get_mixedmm_precondition`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
    m = context.get_value("m")
    k = context.get_value("k")
    n = context.get_value("n")
    if m > 128 or k < 1024 or n < 1024:
        return False
    mat1_iscontig = context.get_value("mat1_iscontig")
    mat2_iscontig = context.get_value("mat2_iscontig")
    return mat1_iscontig and not mat2_iscontig


def get_mult_dims_ops() -> list[AHOperation]:
    m_times_k_op = AHOperation("m*k", lambda data: data["m"] * data["k"])
    m_times_n_op = AHOperation("m*n", lambda data: data["m"] * data["n"])
    k_times_n_op = AHOperation("k*n", lambda data: data["k"] * data["n"])
    return [m_times_k_op, m_times_n_op, k_times_n_op]


def get_arith_intensity(data: Any) -> float:
    m = data["m"]
    k = data["k"]
````
- **EN**: Introduces function `get_mult_dims_ops`, function `get_arith_intensity`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `m`, `k`, `n`, `mat1_iscontig`, `mat2_iscontig`, `m_times_k_op`, and `...+2`.
- **CN**: 这里定义了函数`get_mult_dims_ops`、函数`get_arith_intensity`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `m`、`k`、`n`、`mat1_iscontig`、`mat2_iscontig`、`m_times_k_op`、`另有2项` 等值。

### Lines 161-180 / 第 161-180 行
````python
    n = data["n"]
    if m == 0 or k == 0 or n == 0:
        return 0.0
    return m * k * n / (m * k + k * n + m * n)


def pad_mm_operations() -> list[AHOperation]:
    mult_dims_ops = get_mult_dims_ops()
    k_div_m_times_n_op = AHOperation(
        "k/(m*n)", lambda data: data["k"] / (data["m"] * data["n"])
    )

    def bfloat_perf_hit(data: Any) -> bool:
        m = data["m"]
        k = data["k"]
        n = data["n"]
        is_bfloat = str(data["mat1_dtype"]) == "torch.bfloat16"
        return k > (m * 1024) and k > (n * 1024) and is_bfloat

    bfloat_perf_hit_op = AHOperation(
````
- **EN**: Introduces function `pad_mm_operations`, function `bfloat_perf_hit`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`pad_mm_operations`、函数`bfloat_perf_hit`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
        "bfloat_perf_hit", bfloat_perf_hit, is_categorical=True
    )

    arith_intensity_op = AHOperation("arith_intensity", get_arith_intensity)
    dims_need_padding_ops = get_dims_need_padding_ops()
    dims_multiple_ops = get_dims_multiple_ops()
    is_contig_ops = get_is_contig_ops()

    ah_operations = mult_dims_ops + [
        k_div_m_times_n_op,
        bfloat_perf_hit_op,
        arith_intensity_op,
    ]
    ah_operations.extend(dims_need_padding_ops)
    ah_operations.extend(dims_multiple_ops)
    ah_operations.extend(is_contig_ops)
    return ah_operations


def between_op(data: Any, dim: str, lower: int, upper: int) -> bool:
````
- **EN**: Introduces function `between_op`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `arith_intensity_op`, `dims_need_padding_ops`, `dims_multiple_ops`, `is_contig_ops`, and `ah_operations`.
- **CN**: 这里定义了函数`between_op`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `arith_intensity_op`、`dims_need_padding_ops`、`dims_multiple_ops`、`is_contig_ops`、`ah_operations` 等值。

### Lines 201-220 / 第 201-220 行
````python
    return data[dim] >= lower and data[dim] <= upper


def between_ops() -> list[AHOperation]:
    dims = ["m", "k", "n"]
    limits = [(1, 16), (17, 32), (33, 64), (65, 128), (129, 256)]
    ah_operations = []
    for dim in dims:
        for lower, upper in limits:
            between_op_fn = functools.partial(
                between_op, dim=dim, lower=lower, upper=upper
            )
            # using 'LEQ' instead of '<=' because '<=' cannot be exported to dot
            between_op_name = f"{lower}LEQ{dim}LEQ{upper}"
            ah_operations.append(
                AHOperation(between_op_name, between_op_fn, is_categorical=True)
            )
    return ah_operations


````
- **EN**: Introduces function `between_ops`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dims`, `limits`, `ah_operations`, `between_op_fn`, and `between_op_name`.
- **CN**: 这里定义了函数`between_ops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dims`、`limits`、`ah_operations`、`between_op_fn`、`between_op_name` 等值。

### Lines 221-240 / 第 221-240 行
````python
def pow2_op(data: Any, dim: str, exponent: int) -> bool:
    return data[dim] == 2**exponent


def mm_operations() -> list[AHOperation]:
    mult_dims_ops = get_mult_dims_ops()
    arith_intensity_op = AHOperation("arith_intensity", get_arith_intensity)
    return mult_dims_ops + [arith_intensity_op]


def mixed_mm_operations() -> list[AHOperation]:
    return mm_operations() + between_ops()


def is_multiple(data: Any, dim: str, mult: int) -> bool:
    return data[dim] % mult == 0


def get_dims_multiple_ops() -> list[AHOperation]:
    multiples = [2, 4, 8, 16, 32]
````
- **EN**: Introduces function `pow2_op`, function `mm_operations`, function `mixed_mm_operations`, function `is_multiple`, function `get_dims_multiple_ops`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `mult_dims_ops`, `arith_intensity_op`, and `multiples`.
- **CN**: 这里定义了函数`pow2_op`、函数`mm_operations`、函数`mixed_mm_operations`、函数`is_multiple`、函数`get_dims_multiple_ops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `mult_dims_ops`、`arith_intensity_op`、`multiples` 等值。

### Lines 241-260 / 第 241-260 行
````python
    dims = ["m", "k", "n"]
    dims_multiple_ops = []
    for dim in dims:
        for mult in multiples:
            is_multiple_fn = functools.partial(is_multiple, dim=dim, mult=mult)
            dims_multiple_op = AHOperation(
                f"{dim}_multiple_{mult}", is_multiple_fn, is_categorical=True
            )
            dims_multiple_ops.append(dims_multiple_op)
    return dims_multiple_ops


def get_dims_need_padding_ops() -> list[AHOperation]:
    def mat1_innermost_needs_padding_fn(data: Any) -> bool:
        mat1_stride_0 = data["mat1_stride_0"]
        mat1_stride_1 = data["mat1_stride_1"]
        m_padded_length = data["m_padded_length"]
        k_padded_length = data["k_padded_length"]
        mat1_innermost_needs_padding = False
        if mat1_stride_0 == 1 and m_padded_length != 0:
````
- **EN**: Introduces function `get_dims_need_padding_ops`, function `mat1_innermost_needs_padding_fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dims`, `dims_multiple_ops`, `is_multiple_fn`, `dims_multiple_op`, `mat1_stride_0`, `mat1_stride_1`, and `...+3`.
- **CN**: 这里定义了函数`get_dims_need_padding_ops`、函数`mat1_innermost_needs_padding_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dims`、`dims_multiple_ops`、`is_multiple_fn`、`dims_multiple_op`、`mat1_stride_0`、`mat1_stride_1`、`另有3项` 等值。

### Lines 261-280 / 第 261-280 行
````python
            mat1_innermost_needs_padding = True
        if mat1_stride_1 == 1 and k_padded_length != 0:
            mat1_innermost_needs_padding = True
        return mat1_innermost_needs_padding

    mat1_innermost_op = AHOperation(
        "mat1_innermost_needs_padding",
        mat1_innermost_needs_padding_fn,
        is_categorical=True,
    )

    def mat2_innermost_needs_padding_fn(data: Any) -> bool:
        mat2_stride_0 = data["mat2_stride_0"]
        mat2_stride_1 = data["mat2_stride_1"]
        k_padded_length = data["k_padded_length"]
        n_padded_length = data["n_padded_length"]
        mat2_innermost_needs_padding = False
        if mat2_stride_0 == 1 and k_padded_length != 0:
            mat2_innermost_needs_padding = True
        if mat2_stride_1 == 1 and n_padded_length != 0:
````
- **EN**: Introduces function `mat2_innermost_needs_padding_fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mat1_innermost_needs_padding`, `mat1_innermost_op`, `is_categorical`, `mat2_stride_0`, `mat2_stride_1`, `k_padded_length`, and `...+2`.
- **CN**: 这里定义了函数`mat2_innermost_needs_padding_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `mat1_innermost_needs_padding`、`mat1_innermost_op`、`is_categorical`、`mat2_stride_0`、`mat2_stride_1`、`k_padded_length`、`另有2项` 等值。

### Lines 281-300 / 第 281-300 行
````python
            mat2_innermost_needs_padding = True
        return mat2_innermost_needs_padding

    mat2_innermost_op = AHOperation(
        "mat2_innermost_needs_padding",
        mat2_innermost_needs_padding_fn,
        is_categorical=True,
    )

    def num_dims_needs_padding_fn(data: Any) -> int:
        m_padded_length = data["m_padded_length"]
        k_padded_length = data["k_padded_length"]
        n_padded_length = data["n_padded_length"]
        num_dims_needs_padding = 0
        if m_padded_length != 0:
            num_dims_needs_padding += 1
        if k_padded_length != 0:
            num_dims_needs_padding += 1
        if n_padded_length != 0:
            num_dims_needs_padding += 1
````
- **EN**: Introduces function `num_dims_needs_padding_fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mat2_innermost_needs_padding`, `mat2_innermost_op`, `is_categorical`, `m_padded_length`, `k_padded_length`, `n_padded_length`, and `...+1`.
- **CN**: 这里定义了函数`num_dims_needs_padding_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `mat2_innermost_needs_padding`、`mat2_innermost_op`、`is_categorical`、`m_padded_length`、`k_padded_length`、`n_padded_length`、`另有1项` 等值。

### Lines 301-320 / 第 301-320 行
````python
        return num_dims_needs_padding

    num_dims_op = AHOperation("num_dims_needs_padding", num_dims_needs_padding_fn)
    return [mat1_innermost_op, mat2_innermost_op, num_dims_op]


def get_is_contig_ops() -> list[AHOperation]:
    def mat1_is_contig_fn(data: Any) -> bool:
        stride_0 = data["mat1_stride_0"]
        stride_1 = data["mat1_stride_1"]
        k = data["k"]
        return stride_0 == k and stride_1 == 1

    mat1_is_contig_op = AHOperation(
        "mat1_iscontig", mat1_is_contig_fn, is_categorical=True
    )

    def mat2_is_contig_fn(data: Any) -> bool:
        stride_0 = data["mat2_stride_0"]
        stride_1 = data["mat2_stride_1"]
````
- **EN**: Introduces function `get_is_contig_ops`, function `mat1_is_contig_fn`, function `mat2_is_contig_fn`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `num_dims_op`, `stride_0`, `stride_1`, `k`, and `mat1_is_contig_op`.
- **CN**: 这里定义了函数`get_is_contig_ops`、函数`mat1_is_contig_fn`、函数`mat2_is_contig_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `num_dims_op`、`stride_0`、`stride_1`、`k`、`mat1_is_contig_op` 等值。

### Lines 321-340 / 第 321-340 行
````python
        n = data["n"]
        return stride_0 == n and stride_1 == 1

    mat2_is_contig_op = AHOperation(
        "mat2_iscontig", mat2_is_contig_fn, is_categorical=True
    )

    return [mat1_is_contig_op, mat2_is_contig_op]


def context_add_strides(context: AHContext, name: str, stride: tuple[int, ...]) -> None:
    for i, s in enumerate(stride):
        context.add_feature(f"{name}_stride_{i}", s)


def context_add_using_tf32(context: AHContext, dtype: torch.dtype) -> None:
    using_tf32 = "not_float_32"
    if dtype == torch.float32:
        using_tf32 = torch.backends.cuda.matmul.fp32_precision == "tf32"
    context.add_feature("using_tf32", using_tf32, is_categorical=True)
````
- **EN**: Introduces function `context_add_strides`, function `context_add_using_tf32`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`context_add_strides`、函数`context_add_using_tf32`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Contains heuristic models and helpers for autotuning decisions  
  **CN**: 包含自动调优决策所需的启发式模型与辅助逻辑
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `AHFeature`, `AHOperation`, `AHContext`, and `AHMetadata`  
  **CN**: 主要类：`AHFeature`、`AHOperation`、`AHContext`、`AHMetadata`
- **EN**: Primary functions: `get_metadata_str_from_log`, `check_minsize`, `pad_mm_precondition`, `get_mixedmm_precondition`, `get_mult_dims_ops`, `get_arith_intensity`, and `...+12`  
  **CN**: 主要函数：`get_metadata_str_from_log`、`check_minsize`、`pad_mm_precondition`、`get_mixedmm_precondition`、`get_mult_dims_ops`、`get_arith_intensity`、`另有12项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`
