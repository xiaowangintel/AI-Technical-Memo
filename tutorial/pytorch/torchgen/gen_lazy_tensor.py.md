# gen_lazy_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/gen_lazy_tensor.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Drives top-level torchgen code generation, orchestrating YAML parsing, model building, and file emission.
- **Purpose (CN)**: 驱动 torchgen 顶层代码生成流程，负责 YAML 解析、模型构建与文件输出。
## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```python
from __future__ import annotations

import argparse
import os
from collections import namedtuple
from pathlib import Path
from typing import Any, TYPE_CHECKING

import yaml

import torchgen.dest as dest
from torchgen.api.lazy import setValueT
from torchgen.api.types import BaseCppType
from torchgen.dest.lazy_ir import GenLazyIR, GenLazyNativeFuncDefinition, GenTSLazyIR
from torchgen.gen import get_grouped_native_functions, parse_native_yaml
from torchgen.gen_backend_stubs import (
    error_on_missing_kernels,
    gen_dispatcher_registrations,
    gen_dispatchkey_nativefunc_headers,
    parse_backend_yaml,
)
from torchgen.model import NativeFunction, NativeFunctionsGroup, OperatorName
from torchgen.selective_build.selector import SelectiveBuilder
from torchgen.utils import FileManager, NamespaceHelper
from torchgen.yaml_utils import YamlLoader
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.dest, torchgen.api.lazy, torchgen.api.types, torchgen.dest.lazy_ir, and 6 more; third-party modules such as yaml; standard-library modules such as __future__, argparse, os, collections, and 2 more for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.dest、torchgen.api.lazy、torchgen.api.types、torchgen.dest.lazy_ir 等共 10 项；第三方模块，如 yaml；标准库模块，如 __future__、argparse、os、collections 等共 6 项组织在一起，供下方逻辑使用。

### Lines 28-55
```python
if TYPE_CHECKING:
    from collections.abc import Callable, Iterable, Iterator, Sequence


# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #
#
#                        Lazy Tensor Codegen
#
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #
# Overview
# ~~~~~~~~
#
# This codegen script builds on existing data models and helpers used
# by all ATen backends, and adds new functionality specific to lazy
# tensor backends.
#
# Inputs:
# - <backend>_native_functions.yaml: controls which operators are
#   supported by the backend.
#
# Outputs:
# (for all backends)
# <DispatchKey>Ir.h defines Lazy IR classes to be constructed during tracing
# - opt-in: also generate 'lowering' methods for the TorchScript backend only
# <DispatchKey>NativeFunctions.cpp defines implementations of native functions which perform lazy tracing
# - opt-in: 'full_codegen' section of backend yaml; 'supported' section omits these implementations
# <DispatchKey>NativeFunctions.h declares implementations of native functions for both 'supported' and 'full_codegen'
# ops
```
- **EN**: The import section wires together standard-library modules such as collections.abc for the logic below. Conditional branches preserve special cases and keep invariants explicit.
- **CN**: 导入区把标准库模块，如 collections.abc组织在一起，供下方逻辑使用。 条件分支用于保留特殊情况，并显式维护不变量。

### Lines 56-83
```python
#
# Register<DispatchKey>.cpp registers all op implementations with the dispatcher
# RegisterAutograd<DispatchKey>.cpp registers all autograd implementations with the dispatcher
#
# Validation Helpers:
# - Shape Inference: errs if any ops in backend yaml require shape inference not provided by meta kernels or
#   implementations in torch/csrc/lazy/core/shape_inference.*
# - native function impls: errs if any 'supported' ops do not have an implementation defined in the backend
#   (non-codegen) implementation file
#
#
# About the Data Model
# ~~~~~~~~~~~~~~~~~~~~
#
# Modeled after ATen codegen, the first step is to parse yaml and build a data model for the operators
# we care about.  In this case, the <backend>_native_functions yaml defines a subset of the core operators
# (defined in more detail in the main native_functions.yaml), which will be supported by your backend.
# Backends can list ops in two categories:
#  - `supported` ops require hand-implementations but still get codegenned declarations and registrations
#  - `full_codegen` ops get implementations (and IR classes) generated too
#
# Each native function is modeled as an object with a schema, and each schema has objects representing their
# arguments.  Much of the codegen is manipulation of the arguments and their types.  For example, lazy tensor
# backends need to transform 'at::Tensor' arguments into 'lazy::Value' objects, as well as replacing reference
# types (stringref) with actual string objects, and this is done by manipulating the data model objects.
# - see api/lazy.py for the lazy data model
#
# Once the data model is set up, the rest of this script processes a number of templates for output CPP file
```
- **EN**: This comment block records assumptions, caveats, or usage guidance that frame the implementation below.
- **CN**: 这一段注释记录了后续实现所依赖的假设、注意事项或使用说明。

### Lines 84-105
```python
# and fills in the template values using helpers in `dest/lazy_ir.py` and `dest/lazy_ts_lowering.py`.  These
# helpers mostly iterate over functions and their arguments, outputting different c++ snippets.
#
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #


# Parses the external backend's yaml, and adds a new BackendIndex for the backend's dispatch key.
# Returns a Tuple of (backend_key, autograd_key, cpp_namespace, updated BackendIndex mapping, full_codegen)
ParsedExternalYaml = namedtuple(
    "ParsedExternalYaml",
    ["backend_key", "autograd_key", "cpp_namespace", "backend_indices", "full_codegen"],
)


def parse_native_functions_keys(
    backend_yaml_path: str,
    grouped_native_functions: Sequence[NativeFunction | NativeFunctionsGroup],
) -> tuple[list[OperatorName], list[Any], list[OperatorName]]:
    with open(backend_yaml_path) as f:
        yaml_values = yaml.load(f, Loader=YamlLoader)
    if not isinstance(yaml_values, dict):
        raise AssertionError(f"Expected dict from YAML, got {type(yaml_values)}")
```
- **EN**: This chunk defines `parse_native_functions_keys`, which parses textual or structured input into internal representations. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `parse_native_functions_keys`，其作用是把文本或结构化输入解析为内部表示。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 107-133
```python
    full_codegen = yaml_values.pop("full_codegen", [])
    non_native = yaml_values.pop("non_native", [])
    ir_gen = yaml_values.pop("ir_gen", [])
    if not isinstance(full_codegen, list):
        raise AssertionError(
            f"Expected full_codegen to be list, got {type(full_codegen)}"
        )
    if not isinstance(non_native, list):
        raise AssertionError(f"Expected non_native to be list, got {type(non_native)}")
    if not isinstance(ir_gen, list):
        raise AssertionError(f"Expected ir_gen to be list, got {type(ir_gen)}")
    full_codegen_opnames = [OperatorName.parse(name) for name in full_codegen]
    ir_gen_opnames = [OperatorName.parse(name) for name in ir_gen]
    return full_codegen_opnames, non_native, ir_gen_opnames


def validate_shape_inference_header(
    shape_inference_hdr: str, expected_shape_infr_decls: list[str]
) -> None:
    try:
        with open(shape_inference_hdr) as f:
            shape_infr_decls = f.read()
            shape_infr_decl_lines = set(shape_infr_decls.split("\n"))
    except OSError as e:
        raise AssertionError(
            f"Unable to read from the specified shape_inference_hdr file: {shape_inference_hdr}"
        ) from e
```
- **EN**: This chunk defines `validate_shape_inference_header`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `validate_shape_inference_header`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 135-162
```python
    # TODO(whc) add a check for shape inference functions that have meta kernels implement and should be retired.

    missing_decls = [
        decl for decl in expected_shape_infr_decls if decl not in shape_infr_decl_lines
    ]
    if missing_decls:
        raise Exception(  # noqa: TRY002
            f"""Missing shape inference function.\n
Please add declare this function in {shape_inference_hdr}:\n
and implement it in the corresponding shape_inference.cpp file.\n
{os.linesep.join(missing_decls)}"""
        )


# Some helper functions for the codegen.
def get_ltc_helper_fns() -> str:
    return """\
at::Tensor to_meta(const at::Tensor& tensor) {
  // undefined tensors can't be converted to the meta device, since they don't have sizes/strides
  if (!tensor.defined()) return tensor;
  auto out = at::native::empty_strided_meta_symint(tensor.sym_sizes(), tensor.sym_strides(), \
/*dtype=*/tensor.scalar_type(), /*layout=*/tensor.layout(), \
/*device=*/c10::Device(c10::kMeta), /*pin_memory=*/std::nullopt);
  // needs to handle wrapped numbers, so dtype promotion works properly.
  if (tensor.unsafeGetTensorImpl()->is_wrapped_number()) {
    out.unsafeGetTensorImpl()->set_wrapped_number(true);
  }
  return out;
```
- **EN**: This chunk defines `to_meta`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `to_meta`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 163-179
```python
}
std::optional<at::Tensor> to_meta(const std::optional<at::Tensor>& tensor) {
  if (tensor.has_value()) {
    return to_meta(*tensor);
  }
  return std::nullopt;
}

std::vector<at::Tensor> to_meta(at::ITensorListRef t_list) {
  std::vector<at::Tensor> outs;
  outs.reserve(t_list.size());
  for (const auto& tensor : t_list) {
    outs.push_back(to_meta(tensor));
  }
  return outs;
}
"""
```
- **EN**: This chunk defines `to_meta`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `to_meta`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 182-209
```python
class default_args:
    node_base: str = "Node"
    node_base_hdr: str | None = None
    shape_inference_hdr: str = "torch/csrc/lazy/core/shape_inference.h"
    tensor_class: str = "torch::lazy::LazyTensor"
    tensor_class_hdr: str = "torch/csrc/lazy/core/tensor.h"
    lazy_ir_generator: type[GenLazyIR] = GenLazyIR
    native_func_definition_generator: type[GenLazyNativeFuncDefinition] = (
        GenLazyNativeFuncDefinition
    )
    backend_name: str = "TorchScript"


def main() -> None:
    parser = argparse.ArgumentParser(description="Generate Lazy Tensor backend files")
    parser.add_argument(
        "-s",
        "--source-yaml",
        "--source_yaml",
        help="path to source yaml file containing operator external definitions",
    )
    parser.add_argument("-o", "--output-dir", "--output_dir", help="output directory")
    parser.add_argument(
        "--dry-run", "--dry_run", type=bool, default=False, help="output directory"
    )
    parser.add_argument(
        "--impl-path",
        "--impl_path",
```
- **EN**: It introduces or extends default_args, which hold the primary data model or public surface for this slice of the file. This chunk defines `main`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 default_args，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `main`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 210-237
```python
        type=str,
        default=None,
        help="path to the source C++ file containing kernel definitions",
    )
    parser.add_argument(
        "--gen-ts-lowerings",
        "--gen_ts_lowerings",
        action="store_true",
        help="Generate TorchScript lowerings in addition to Lazy IR and NativeFunctions",
    )
    parser.add_argument(
        "--node-base",
        "--node_base",
        type=str,
        default=default_args.node_base,
        help="Name of backend specific custom Lazy IR Node base class",
    )
    parser.add_argument(
        "--node-base-hdr",
        "--node_base_hdr",
        type=str,
        default=default_args.node_base_hdr,
        help="Path to header file defining custom Lazy IR Node base class",
    )
    parser.add_argument(
        "--shape-inference-hdr",
        "--shape_inference_hdr",
        type=str,
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 238-262
```python
        default=default_args.shape_inference_hdr,
        help="Path to header file defining custom Lazy shape inference functions",
    )
    parser.add_argument(
        "--tensor-class",
        "--tensor_class",
        type=str,
        default=default_args.tensor_class,
        help="Name of backend specific custom Lazy Tensor class",
    )
    parser.add_argument(
        "--tensor-class-hdr",
        "--tensor_class_hdr",
        type=str,
        default=default_args.tensor_class_hdr,
        help="Path to header file defining custom Lazy Tensor class",
    )
    parser.add_argument(
        "--backend-name",
        "--backend_name",
        type=str,
        default=default_args.backend_name,
        help="Name of the backend to generate",
    )
    options = parser.parse_args()
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 264-288
```python
    # Assumes that this file lives at PYTORCH_ROOT/torchgen/gen_backend_stubs.py
    torch_root = Path(__file__).absolute().parents[2]
    aten_path = str(torch_root / "aten" / "src" / "ATen")
    lazy_ir_generator: type[GenLazyIR] = default_args.lazy_ir_generator
    if options.gen_ts_lowerings:
        lazy_ir_generator = GenTSLazyIR
    native_func_definition_generator: type[GenLazyNativeFuncDefinition] = (
        default_args.native_func_definition_generator
    )

    run_gen_lazy_tensor(
        aten_path,
        options.source_yaml,
        options.output_dir,
        options.dry_run,
        options.impl_path,
        options.node_base,
        options.node_base_hdr,
        options.tensor_class,
        options.tensor_class_hdr,
        options.shape_inference_hdr,
        lazy_ir_generator,
        native_func_definition_generator,
        options.backend_name,
    )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 291-318
```python
def run_gen_lazy_tensor(
    aten_path: str,
    source_yaml: str,
    output_dir: str,
    dry_run: bool,
    impl_path: str | None,
    node_base: str = default_args.node_base,
    node_base_hdr: str | None = default_args.node_base_hdr,
    tensor_class: str = default_args.tensor_class,
    tensor_class_hdr: str = default_args.tensor_class_hdr,
    shape_inference_hdr: str = default_args.shape_inference_hdr,
    lazy_ir_generator: type[GenLazyIR] = default_args.lazy_ir_generator,
    native_func_definition_generator: type[
        GenLazyNativeFuncDefinition
    ] = default_args.native_func_definition_generator,
    # build_in_tree is true for TS backend and affects include paths
    build_in_tree: bool = False,
    # per_operator_headers changes whether ATen/Functions.h or individual operator headers are used
    # it must match how ATen was built
    per_operator_headers: bool = False,
    backend_name: str = default_args.backend_name,
    gen_forced_fallback_code: bool = False,
    use_lazy_shape: bool = True,
    # the following arguments are temporary customization points for xla backend migration.
    # do not rely on them otherwise, they should be removed once migration is complete
    backend_namespace: str = "torch::lazy",
    get_tensorlist: str = "GetTensorList",
    get_tensor_or_wrap_number: str = "GetLtcTensorOrCreateForWrappedNumber",
```
- **EN**: This chunk defines `run_gen_lazy_tensor`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `run_gen_lazy_tensor`，其作用是根据高层元数据生成派生代码、表或分析产物。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 319-340
```python
    try_get_tensor: str = "TryGetLtcTensor",
    metrics_counter: str = 'TORCH_LAZY_FN_COUNTER("lazy::")',
    create_tensor: str = "LazyTensor::Create",
    create_from_first_tensor: bool = False,
    create_aten_from_ltc_tensor: str = "torch::lazy::CreateAtenFromLtcTensor",
    tuple_aten_from_ltc_tensors: str = "torch::lazy::TupleAtenFromLtcTensors",
    lazy_value_class: str = "torch::lazy::Value",
    lazy_tensor_ptr: str = "LazyTensorPtr",
    get_device_fn: str = "torch::lazy::GetBackendDevice",
) -> None:
    lv_tokens = lazy_value_class.split("::")
    lv_class = lv_tokens[-1]
    lv_ns = "::".join(lv_tokens[:-1])
    setValueT(BaseCppType(lv_ns, lv_class))
    template_dir = os.path.join(aten_path, "templates")

    def make_file_manager(install_dir: str) -> FileManager:
        return FileManager(
            install_dir=install_dir, template_dir=template_dir, dry_run=dry_run
        )

    fm = make_file_manager(output_dir)
```
- **EN**: This chunk defines `make_file_manager`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `make_file_manager`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 342-361
```python
    native_yaml_path = os.path.join(aten_path, "native/native_functions.yaml")
    tags_yaml_path = os.path.join(aten_path, "native/tags.yaml")
    parsed_yaml = parse_native_yaml(native_yaml_path, tags_yaml_path)
    native_functions, backend_indices = (
        parsed_yaml.native_functions,
        parsed_yaml.backend_indices,
    )
    grouped_native_functions = get_grouped_native_functions(native_functions)

    def sort_native_function(f: NativeFunctionsGroup | NativeFunction) -> str:
        """
        We sort the native function because of the note in concat_map_codegen.
        TODO(alanwaketan): Remove this sorting hack once all ops are grouped properly.
        """
        func = f.functional.func if isinstance(f, NativeFunctionsGroup) else f.func
        return str(func.name.name)

    grouped_native_functions = sorted(
        grouped_native_functions, key=sort_native_function
    )
```
- **EN**: This chunk defines `sort_native_function`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `sort_native_function`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 363-387
```python
    parsed_backend_yaml = parse_backend_yaml(
        source_yaml, grouped_native_functions, backend_indices
    )
    backend_key = parsed_backend_yaml.backend_key
    autograd_key = parsed_backend_yaml.autograd_key
    cpp_namespace = parsed_backend_yaml.cpp_namespace
    backend_indices = parsed_backend_yaml.backend_indices
    # the following 3 keys are all processed differently
    # for full_codegen, we generate IR, kernels, etc
    # for ir_gen, we generate only IR
    # non_native is used to register kernels not declared in
    # native_functions.yaml
    full_codegen, non_native, ir_gen = parse_native_functions_keys(
        source_yaml, grouped_native_functions
    )

    def concat_map_codegen(
        func: Callable[[NativeFunction], Sequence[str]],
        xs: Iterable[NativeFunctionsGroup | NativeFunction],
        ops_list: list[OperatorName] = full_codegen,
    ) -> Iterator[str]:
        """
        We code-gen for the functional variant, which is all we need for IR classes/lowerings/shape inferences, but we
        only code-gen additional entries for the inplace variant for the native functions.
        """
```
- **EN**: This chunk defines `concat_map_codegen`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `concat_map_codegen`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 389-412
```python
        for x in xs:
            fs = list(x.functions()) if isinstance(x, NativeFunctionsGroup) else [x]
            for f in fs:
                if f.func.name in ops_list:
                    yield from func(f)

    selector = SelectiveBuilder.get_nop_selector()

    if backend_key is None:
        raise AssertionError("backend_key must be non-None")
    class_name = backend_indices[backend_key].native_function_class_name()

    if impl_path is not None:
        error_on_missing_kernels(
            native_functions,
            backend_indices,
            backend_key,
            autograd_key,
            class_name,
            impl_path,
            full_codegen,
        )

    """ Validate Shape Inference Definitions
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 414-438
```python
    Generated lazy native functions all perform shape inference, by first using a meta:: kernel
    if available for that op, and otherwise using a 'compute_shape_{op}' function instead.  The generator
    knows the call signature for compute_shape_{op} because it matches the nativefunction (and meta::) signature,
    so it just has to check whether the op is structured and generate a call for one or the other.  It's up to the dev
    to supply the missing compute_shape_{op} function, but the codegen at least warns you about this and provides
    the expected signature which can be copy-pasted into shape_inference.h.

    compute_shape_{op} functions are handwritten and should be replaced over time as ops get ported
    to structured kernels.

    See torch/csrc/lazy/core/shape_inference.cpp #READ THIS! for more information.
    """
    if shape_inference_hdr is not None:
        expected_shape_infr_decls = list(
            concat_map_codegen(
                dest.GenLazyShapeInferenceDefinition(
                    backend_indices[backend_key], tensor_class
                ),
                grouped_native_functions,
            )
        )

        validate_shape_inference_header(shape_inference_hdr, expected_shape_infr_decls)
    if class_name is None:
        raise AssertionError("class_name must be non-None")
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 440-467
```python
    # Generate nativefunction declarations
    # Note, eager registrations is set to False for the lazy TS backend as another LTC backend
    # may want to register their own lazy kernels instead of registering the TS ones.
    # The registration will lazily happen when init_ts_backend is called.
    gen_dispatchkey_nativefunc_headers(
        fm,
        class_name,
        cpp_namespace,
        backend_indices,
        grouped_native_functions,
        backend_key,
        autograd_key,
        backend_name,
    )

    # Generate Dispatcher registrations which hook up the nativefunctions
    for dispatch_key in (
        [backend_key] if autograd_key is None else [backend_key, autograd_key]
    ):
        gen_dispatcher_registrations(
            fm,
            output_dir,
            class_name,
            backend_indices,
            grouped_native_functions,
            backend_key,
            dispatch_key,
            selector,
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。

### Lines 468-495
```python
            build_in_tree=build_in_tree,
            per_operator_headers=per_operator_headers,
            backend_name=backend_name,
            eager_registration=False,
        )

    # Generate native function impls that build IR nodes
    ns_helper = NamespaceHelper(cpp_namespace)
    fm.write_with_template(
        f"{backend_key}NativeFunctions.cpp",
        "DispatchKeyNativeFunctions.cpp",
        lambda: {
            "includes": [
                f"#include <{path}>"
                for path in [
                    tensor_class_hdr,
                    shape_inference_hdr,
                    "ATen/Functions.h",
                    "ATen/native/TensorConversions.h",
                    "ATen/NativeFunctions.h",
                    "ATen/CompositeExplicitAutogradNonFunctionalFunctions.h",
                    "ATen/MetaFunctions.h",
                    "ATen/Operators.h",
                    "ATen/native/CPUFallback.h",
                    "torch/csrc/lazy/core/ir_builder.h",
                    "torch/csrc/lazy/core/lazy_graph_executor.h",
                    "torch/csrc/lazy/core/metrics.h",
                    "torch/csrc/lazy/core/shape.h",
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 496-523
```python
                    f"{output_dir}/{backend_key}NativeFunctions.h",
                    f"{output_dir}/LazyIr.h",
                ]
                + (
                    ["torch/csrc/lazy/ts_backend/ts_eager_fallback.h"]
                    if gen_forced_fallback_code
                    else []
                )
            ],
            "helper_fns": get_ltc_helper_fns(),
            "native_functions_include": "",
            "namespace_prologue": ns_helper.prologue,
            "namespace_epilogue": ns_helper.epilogue,
            "native_function_definitions": list(
                concat_map_codegen(
                    native_func_definition_generator(
                        f"{backend_key}NativeFunctions",
                        backend_indices[backend_key],
                        tensor_class,
                        gen_forced_fallback_code,
                        backend_namespace,
                        get_tensorlist,
                        get_tensor_or_wrap_number,
                        try_get_tensor,
                        metrics_counter,
                        create_tensor,
                        create_from_first_tensor,
                        create_aten_from_ltc_tensor,
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。

### Lines 524-551
```python
                        tuple_aten_from_ltc_tensors,
                        lazy_tensor_ptr,
                        get_device_fn,
                    ),
                    grouped_native_functions,
                )
            ),
        },
    )
    # Generate IR node classes
    lazy_ir_obj = lazy_ir_generator(
        backend_indices[backend_key], backend_name, node_base, use_lazy_shape
    )

    fm.write_with_template(
        "LazyIr.h",
        "LazyIr.h",
        lambda: {
            "lazy_ir_sysinc": [
                f"#include <{path}>"
                for path in [
                    "ATen/core/Formatting.h",
                    "c10/core/ScalarType.h",
                    "torch/csrc/lazy/core/hash.h",
                    "torch/csrc/lazy/core/ir.h",
                    "torch/csrc/lazy/core/shape.h",
                    "optional",
                    "vector",
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 552-579
```python
                ]
            ],
            "lazy_ir_inc": [f'#include "{node_base_hdr}"']
            if node_base_hdr is not None
            else [],
            "ir_declarations": list(
                concat_map_codegen(
                    lazy_ir_obj, grouped_native_functions, full_codegen + ir_gen
                )
            ),
            "namespace_prologue": ns_helper.prologue,
            "namespace_epilogue": ns_helper.epilogue,
        },
    )

    # Generate Non Native IR Node classes
    fm.write_with_template(
        "LazyNonNativeIr.h",
        "LazyNonNativeIr.h",
        lambda: {
            "lazy_non_native_ir_inc": [
                f"#include <{path}>"
                for path in [
                    "torch/csrc/lazy/core/ir.h",
                    "torch/csrc/lazy/core/ir_builder.h",
                    "torch/csrc/lazy/core/internal_ops/ltc_ops.h",
                    "torch/csrc/lazy/core/shape_inference.h",
                ]
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。

### Lines 580-593
```python
                + ([node_base_hdr] if node_base_hdr else [])
                if path
            ],
            "non_native_ir_nodes": dest.generate_non_native_lazy_ir_nodes(
                non_native, lazy_ir_obj
            ),
            "namespace_prologue": ns_helper.prologue,
            "namespace_epilogue": ns_helper.epilogue,
        },
    )


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `concat_map_codegen` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `concat_map_codegen`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **YAML schema loading**
  - EN: Parses YAML operator metadata that drives later code generation.
  - CN: 解析驱动后续代码生成的 YAML 算子元数据。
- **Dispatch-key plumbing**
  - EN: Models dispatch keys, backend selection, or registration routing.
  - CN: 建模 dispatch key、后端选择或注册路由。
- **Autograd generation**
  - EN: Produces or organizes derivative-related wrappers, bindings, and metadata.
  - CN: 生成或组织与求导相关的包装器、绑定与元数据。
- **Lazy tensor support**
  - EN: Builds lazy IR or lowering support for deferred execution backends.
  - CN: 为延迟执行后端构建 lazy IR 或降级支持。
- **Code-generation templates**
  - EN: Uses placeholders and boilerplate patterns that are later materialized into concrete sources.
  - CN: 使用占位符与样板模式，随后会被实例化为具体源码。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torchgen.dest`, `torchgen.api.lazy`, `torchgen.api.types`, `torchgen.dest.lazy_ir`, `torchgen.gen`, `torchgen.gen_backend_stubs`, `torchgen.model`, `torchgen.selective_build.selector`, `torchgen.utils`, `torchgen.yaml_utils`
- **Third-party modules / 第三方模块**: `yaml`
- **Standard library / 标准库**: `__future__`, `argparse`, `os`, `collections`, `pathlib`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `parse_native_functions_keys`, `validate_shape_inference_header`, `get_ltc_helper_fns`, `default_args`, `main`, `run_gen_lazy_tensor`
- **Note / 说明**: 18 imports were detected; only the first few are listed for readability. / 检测到 18 个导入项，为便于阅读这里只展示前若干项。
