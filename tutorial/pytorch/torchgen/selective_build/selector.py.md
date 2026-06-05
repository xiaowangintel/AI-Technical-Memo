# selector.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/selective_build/selector.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements selective-build models and filters so generated code can be limited to chosen operator subsets.
- **Purpose (CN)**: 实现 selective build 模型与过滤逻辑，使生成代码可以限制在指定算子子集上。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
from __future__ import annotations

from collections import defaultdict
from collections.abc import Iterable
from dataclasses import dataclass
from typing import TYPE_CHECKING

import yaml

from torchgen.selective_build.operator import (
    merge_debug_info,
    merge_operator_dicts,
    SelectiveBuildOperator,
    strip_operator_overload_name,
)
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.selective_build.operator; third-party modules such as yaml; standard-library modules such as __future__, collections, collections.abc, dataclasses, and 1 more for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.selective_build.operator；第三方模块，如 yaml；标准库模块，如 __future__、collections、collections.abc、dataclasses 等共 5 项组织在一起，供下方逻辑使用。

### Lines 18-33
```python
if TYPE_CHECKING:
    from torchgen.model import NativeFunction


# A SelectiveBuilder holds information extracted from the selective build
# YAML specification.
#
# It includes information about the build's selectivity, the debug_info
# associated with this selective build (opaque string), and the set of
# operators that should be included in the build.
#
@dataclass(frozen=True)
class SelectiveBuilder:
    # If true, then the build is not selective, and includes all
    # operators.
    include_all_operators: bool
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.model for the logic below. Decorators such as @dataclass(frozen=True) modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends SelectiveBuilder, which hold the primary data model or public surface for this slice of the file. This chunk continues `SelectiveBuilder` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.model组织在一起，供下方逻辑使用。 像 @dataclass(frozen=True) 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 SelectiveBuilder，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `SelectiveBuilder`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 35-47
```python
    # Debug Information at the selective/custom build level.
    _debug_info: tuple[str, ...] | None

    # A dictionary of operator -> operator metadata.
    operators: dict[str, SelectiveBuildOperator]

    # A dictionary of selected kernel tags and dtypes. Typically a
    # PyTorch Operator Kernel (function) may have many code paths
    # that are specialized for many many Tensor dtypes, so it's not
    # one per kernel function, but there could be many per kernel
    # function. The tag isn't a kernel function name, but some fragment
    # of the kernel function implementation itself.
    kernel_metadata: dict[str, list[str]]
```
- **EN**: This chunk continues `SelectiveBuilder` and expands its control flow, data preparation, or emitted structure.
- **CN**: 这一段延续了 `SelectiveBuilder`，继续展开其控制流、数据准备或生成结构。

### Lines 49-62
```python
    # ExecuTorch only. A dictionary of kernel tag -> list of (list of input
    # dtypes for tensor-like input args).
    # This is from selective.yaml
    et_kernel_metadata: dict[str, list[str]]

    # A set of all the custom torch bind classes used by the selected models
    # Stored as a set internally to remove duplicates proactively, but written
    # as a list to yamls
    custom_classes: set[str]

    # A set of all the build features used by the selected models
    # Stored as a set internally to remove duplicates proactively, but written
    # as a list to yamls
    build_features: set[str]
```
- **EN**: This chunk continues `SelectiveBuilder` and expands its control flow, data preparation, or emitted structure.
- **CN**: 这一段延续了 `SelectiveBuilder`，继续展开其控制流、数据准备或生成结构。

### Lines 64-74
```python
    # If true, then fragments for all dtypes for all kernel functions
    # are included as well as all custom classes. This is typically set when any one of the
    # operator lists is generated from a mechanism other than
    # tracing based selective build.
    include_all_non_op_selectives: bool

    @staticmethod
    def get_nop_selector() -> SelectiveBuilder:
        return SelectiveBuilder.from_yaml_dict({"include_all_operators": True})

    @staticmethod
```
- **EN**: Decorators such as @staticmethod, @staticmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This region behaves like generated or templated source, so the important signal is the emitted schema/layout rather than bespoke control flow. This chunk defines `get_nop_selector`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @staticmethod、@staticmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段更像生成代码或模板，因此重点在于最终发射的 schema/布局，而不是手写控制流。 这一段定义了 `get_nop_selector`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 75-92
```python
    def from_yaml_dict(data: dict[str, object]) -> SelectiveBuilder:
        valid_top_level_keys = {
            "include_all_non_op_selectives",
            "include_all_operators",
            "debug_info",
            "operators",
            "kernel_metadata",
            "et_kernel_metadata",
            "custom_classes",
            "build_features",
        }
        top_level_keys = set(data.keys())
        if len(top_level_keys - valid_top_level_keys) > 0:
            raise Exception(  # noqa: TRY002
                "Got unexpected top level keys: {}".format(
                    ",".join(top_level_keys - valid_top_level_keys),
                )
            )
```
- **EN**: This chunk defines `from_yaml_dict`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `from_yaml_dict`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 93-107
```python
        include_all_operators = data.get("include_all_operators", False)
        if not isinstance(include_all_operators, bool):
            raise AssertionError(
                f"Expected 'include_all_operators' to be bool, got {type(include_all_operators)}"
            )

        debug_info = None
        if "debug_info" in data:
            di_list = data["debug_info"]
            if not isinstance(di_list, list):
                raise AssertionError(
                    f"Expected 'debug_info' to be list, got {type(di_list)}"
                )

            debug_info = tuple(str(x) for x in di_list)
```
- **EN**: This chunk continues `from_yaml_dict` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `from_yaml_dict`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 109-124
```python
        operators = {}
        operators_dict = data.get("operators", {})
        if not isinstance(operators_dict, dict):
            raise AssertionError(
                f"Expected 'operators' to be dict, got {type(operators_dict)}"
            )

        for k, v in operators_dict.items():
            operators[k] = SelectiveBuildOperator.from_yaml_dict(k, v)

        kernel_metadata = {}
        kernel_metadata_dict = data.get("kernel_metadata", {})
        if not isinstance(kernel_metadata_dict, dict):
            raise AssertionError(
                f"Expected 'kernel_metadata' to be dict, got {type(kernel_metadata_dict)}"
            )
```
- **EN**: This chunk continues `from_yaml_dict` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `from_yaml_dict`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 126-140
```python
        for k, v in kernel_metadata_dict.items():
            kernel_metadata[str(k)] = [str(dtype) for dtype in v]

        et_kernel_metadata = data.get("et_kernel_metadata", {})
        if not isinstance(et_kernel_metadata, dict):
            raise AssertionError(
                f"Expected 'et_kernel_metadata' to be dict, got {type(et_kernel_metadata)}"
            )

        custom_classes = data.get("custom_classes", [])
        if not isinstance(custom_classes, Iterable):
            raise AssertionError(
                f"Expected 'custom_classes' to be Iterable, got {type(custom_classes)}"
            )
        custom_classes = set(custom_classes)
```
- **EN**: This chunk continues `from_yaml_dict` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `from_yaml_dict`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 142-154
```python
        build_features = data.get("build_features", [])
        if not isinstance(build_features, Iterable):
            raise AssertionError(
                f"Expected 'build_features' to be Iterable, got {type(build_features)}"
            )
        build_features = set(build_features)

        include_all_non_op_selectives = data.get("include_all_non_op_selectives", False)
        if not isinstance(include_all_non_op_selectives, bool):
            raise AssertionError(
                f"Expected 'include_all_non_op_selectives' to be bool, "
                f"got {type(include_all_non_op_selectives)}"
            )
```
- **EN**: This chunk continues `from_yaml_dict` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `from_yaml_dict`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 156-172
```python
        return SelectiveBuilder(
            include_all_operators,
            debug_info,
            operators,
            kernel_metadata,
            et_kernel_metadata,
            custom_classes,  # type: ignore[arg-type]
            build_features,  # type: ignore[arg-type]
            include_all_non_op_selectives,
        )

    @staticmethod
    def from_yaml_str(config_contents: str) -> SelectiveBuilder:
        contents = yaml.safe_load(config_contents)
        return SelectiveBuilder.from_yaml_dict(contents)

    @staticmethod
```
- **EN**: Decorators such as @staticmethod, @staticmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `from_yaml_str`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @staticmethod、@staticmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `from_yaml_str`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 173-190
```python
    def from_yaml_path(config_path: str) -> SelectiveBuilder:
        with open(config_path) as f:
            contents = yaml.safe_load(f)
            return SelectiveBuilder.from_yaml_dict(contents)

    @staticmethod
    def from_legacy_op_registration_allow_list(
        allow_list: set[str], is_root_operator: bool, is_used_for_training: bool
    ) -> SelectiveBuilder:
        operators = {}
        for op in allow_list:
            operators[op] = {
                "name": op,
                "is_root_operator": is_root_operator,
                "is_used_for_training": is_used_for_training,
                "include_all_overloads": True,
            }
        return SelectiveBuilder.from_yaml_dict(
```
- **EN**: Decorators such as @staticmethod modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `from_legacy_op_registration_allow_list`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @staticmethod 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `from_legacy_op_registration_allow_list`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 191-204
```python
            {
                "operators": operators,
                "include_all_non_op_selectives": True,
            }
        )

    def is_operator_selected(self, name: str) -> bool:
        if self.include_all_operators:
            return True

        if name in self.operators:
            return True
        name = strip_operator_overload_name(name)
        return name in self.operators and self.operators[name].include_all_overloads
```
- **EN**: This chunk defines `is_operator_selected`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `is_operator_selected`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 206-223
```python
    def is_native_function_selected(self, func: NativeFunction) -> bool:
        op_name = op_name_from_native_function(func)
        return self.is_operator_selected(op_name)

    def is_operator_selected_for_training(self, name: str) -> bool:
        if not self.is_operator_selected(name):
            return False
        if self.include_all_operators:
            return True

        not_training_op = SelectiveBuildOperator(
            name="",
            is_root_operator=False,
            is_used_for_training=False,
            include_all_overloads=False,
            _debug_info=None,
        )
        op = not_training_op
```
- **EN**: This chunk defines `is_operator_selected_for_training`, which runs a training procedure that fits a heuristic or predictive model. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `is_operator_selected_for_training`，其作用是执行训练过程以拟合启发式或预测模型。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 224-238
```python
        if name in self.operators:
            op = self.operators[name]

        name = strip_operator_overload_name(name)
        base_op = not_training_op
        if name in self.operators:
            base_op = self.operators[name]

        return op.is_used_for_training or (
            base_op.include_all_overloads and base_op.is_used_for_training
        )

    def is_native_function_selected_for_training(self, func: NativeFunction) -> bool:
        op_name = op_name_from_native_function(func)
        return self.is_operator_selected_for_training(op_name)
```
- **EN**: This chunk defines `is_native_function_selected_for_training`, which runs a training procedure that fits a heuristic or predictive model. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `is_native_function_selected_for_training`，其作用是执行训练过程以拟合启发式或预测模型。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 240-253
```python
    def is_root_operator(self, name: str) -> bool:
        if not self.is_operator_selected(name):
            return False
        if self.include_all_operators:
            return True

        if name in self.operators:
            op: SelectiveBuildOperator = self.operators[name]
            return op.is_root_operator
        name = strip_operator_overload_name(name)
        if name not in self.operators:
            return False
        base_op: SelectiveBuildOperator = self.operators[name]
        return base_op.include_all_overloads and base_op.is_root_operator
```
- **EN**: This chunk defines `is_root_operator`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `is_root_operator`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 255-271
```python
    def is_kernel_dtype_selected(self, kernel_tag: str, dtype: str) -> bool:
        if self.include_all_operators or self.include_all_non_op_selectives:
            return True

        return (
            kernel_tag in self.kernel_metadata
            and dtype in self.kernel_metadata[kernel_tag]
        )

    def et_get_selected_kernels(self, op_name: str, kernel_key: list[str]) -> list[str]:
        """
        Return a list of kernel keys that cover the used ops
        """
        # If no kernel metadata, either it's implied by include_all_operators=True or the op is not used.
        if op_name not in self.et_kernel_metadata:
            return kernel_key if self.include_all_operators else []
        # Otherwise, only return the specific kernel keys.
```
- **EN**: This chunk defines `et_get_selected_kernels`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `et_get_selected_kernels`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 273-290
```python
        result_set = set()

        for model_kernel_keys in self.et_kernel_metadata[op_name]:
            key_found = False
            for key in kernel_key:
                # Don't compare the version for now
                if (
                    key != "default"
                    and key.split("/")[1] == model_kernel_keys.split("/")[1]
                ):
                    result_set.add(key)
                    key_found = True
                    break
            if not key_found:
                if "default" not in kernel_key:
                    raise Exception("Missing kernel for the model")  # noqa: TRY002
                else:
                    result_set.add("default")
```
- **EN**: This chunk continues `et_get_selected_kernels` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `et_get_selected_kernels`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 292-305
```python
        return list(result_set)

    def to_dict(self) -> dict[str, object]:
        ret: dict[str, object] = {
            "include_all_non_op_selectives": self.include_all_non_op_selectives,
            "include_all_operators": self.include_all_operators,
        }
        operators = {}
        for op_name, op in self.operators.items():
            operators[op_name] = op.to_dict()
        ret["operators"] = operators

        if self._debug_info is not None:
            ret["debug_info"] = sorted(self._debug_info)
```
- **EN**: This chunk defines `to_dict`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `to_dict`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 307-317
```python
        ret["kernel_metadata"] = {
            k: sorted(v) for (k, v) in self.kernel_metadata.items()
        }

        ret["et_kernel_metadata"] = self.et_kernel_metadata

        ret["custom_classes"] = sorted(self.custom_classes)

        ret["build_features"] = sorted(self.build_features)

        return ret
```
- **EN**: This chunk continues `to_dict` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `to_dict`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 320-332
```python
def merge_kernel_metadata(
    lhs: dict[str, list[str]],
    rhs: dict[str, list[str]],
) -> dict[str, list[str]]:
    kernel_metadata: dict[str, list[str]] = {}
    for tag_name, dtypes in list(lhs.items()) + list(rhs.items()):
        dtypes_copy = set(dtypes)
        if tag_name in kernel_metadata:
            dtypes_copy |= set(kernel_metadata[tag_name])

        kernel_metadata[tag_name] = list(dtypes_copy)

    return kernel_metadata
```
- **EN**: This chunk defines `merge_kernel_metadata`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `merge_kernel_metadata`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 335-344
```python
def merge_et_kernel_metadata(
    lhs: dict[str, list[str]],
    rhs: dict[str, list[str]],
) -> dict[str, list[str]]:
    merge_et_kernel_metadata: dict[str, set[str]] = defaultdict(set)
    for op in list(lhs.keys()) + list(rhs.keys()):
        merge_et_kernel_metadata[op].update(lhs.get(op, []))
        merge_et_kernel_metadata[op].update(rhs.get(op, []))

    return {op: sorted(val) for op, val in merge_et_kernel_metadata.items()}
```
- **EN**: This chunk defines `merge_et_kernel_metadata`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `merge_et_kernel_metadata`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 347-364
```python
def combine_selective_builders(
    lhs: SelectiveBuilder, rhs: SelectiveBuilder
) -> SelectiveBuilder:
    include_all_operators = lhs.include_all_operators or rhs.include_all_operators
    debug_info = merge_debug_info(lhs._debug_info, rhs._debug_info)
    operators = merge_operator_dicts(lhs.operators, rhs.operators)
    kernel_metadata = merge_kernel_metadata(lhs.kernel_metadata, rhs.kernel_metadata)
    et_kernel_metadata = merge_et_kernel_metadata(
        lhs.et_kernel_metadata, rhs.et_kernel_metadata
    )
    include_all_non_op_selectives = (
        lhs.include_all_non_op_selectives or rhs.include_all_non_op_selectives
    )
    custom_classes = lhs.custom_classes.union(rhs.custom_classes)
    build_features = lhs.build_features.union(rhs.build_features)
    return SelectiveBuilder(
        include_all_operators,
        debug_info,
```
- **EN**: This chunk defines `combine_selective_builders`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `combine_selective_builders`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 365-377
```python
        operators,
        kernel_metadata,
        et_kernel_metadata,
        custom_classes,
        build_features,
        include_all_non_op_selectives,
    )


def op_name_from_native_function(f: NativeFunction) -> str:
    # This was originally read from the 'operator_name_with_overload' field in the
    # declaration dict, which was the part before the first '(' in 'schema_string'.
    return f"{f.namespace}::{f.func.name}"
```
- **EN**: This chunk defines `op_name_from_native_function`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `op_name_from_native_function`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **YAML schema loading**
  - EN: Parses YAML operator metadata that drives later code generation.
  - CN: 解析驱动后续代码生成的 YAML 算子元数据。
- **Selective build**
  - EN: Keeps build outputs focused on a chosen operator subset.
  - CN: 使构建产物聚焦于选定的算子子集。
- **SelectiveBuilder**
  - EN: `SelectiveBuilder` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `SelectiveBuilder` 是本文件声明、导出或驱动的显著符号之一。
- **get_nop_selector**
  - EN: `get_nop_selector` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `get_nop_selector` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torchgen.selective_build.operator`, `torchgen.model`
- **Third-party modules / 第三方模块**: `yaml`
- **Standard library / 标准库**: `__future__`, `collections`, `collections.abc`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `SelectiveBuilder`, `get_nop_selector`, `from_yaml_dict`, `from_yaml_str`, `from_yaml_path`, `from_legacy_op_registration_allow_list`, `is_operator_selected`, `is_native_function_selected`, `is_operator_selected_for_training`, `is_native_function_selected_for_training`
