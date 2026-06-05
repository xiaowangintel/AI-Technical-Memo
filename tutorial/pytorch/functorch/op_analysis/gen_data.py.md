# gen_data.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/op_analysis/gen_data.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Generates or collects operator-analysis data used to study transform support and operator properties.
- **Purpose (CN)**: 生成或收集算子分析数据，用于研究变换支持情况与算子属性。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import csv
from collections import defaultdict

import yaml

import torch


def get_ops_for_key(key):
    # Needs modified PyTorch C++ code to work
    if key is None:
        ops = torch._C._dispatch_get_registrations_for_dispatch_key()
    else:
        ops = torch._C._dispatch_get_registrations_for_dispatch_key(key)
    cleaned_ops = []
    for i in ops:
        if "aten::" not in i:
            continue
```
- **EN**: The import section wires together PyTorch-local modules such as torch; third-party modules such as yaml; standard-library modules such as csv, collections for the logic below. This chunk defines `get_ops_for_key`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 torch；第三方模块，如 yaml；标准库模块，如 csv、collections组织在一起，供下方逻辑使用。 这一段定义了 `get_ops_for_key`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 19-31
```python
        cleaned_ops.append(i[6:].strip())
    return set(cleaned_ops)


def gen_data(special_op_lists, analysis_name):
    all_ops = get_ops_for_key(None)
    composite_ops = get_ops_for_key("CompositeImplicitAutograd")
    noncomposite_ops = all_ops - composite_ops
    with open("../../aten/src/ATen/native/native_functions.yaml") as f:
        ops = yaml.load(f.read(), Loader=yaml.CLoader)

    with open("annotated_ops") as f:
        annotated_ops = {a.strip(): b.strip() for a, b in csv.reader(f)}
```
- **EN**: This chunk defines `gen_data`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `gen_data`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 33-50
```python
    uniq_ops = []
    uniq_names = set()
    overload_types = defaultdict(list)
    cnt = 0
    for op in ops:
        func_str = op["func"]
        name = func_str[: func_str.index("(")]
        if "." in name:
            uniq_name = name[: name.index(".")]
            overload_types[name[name.index(".") + 1 :]].append(name)
        else:
            uniq_name = name
        op["name"] = uniq_name
        full_name = func_str[: func_str.index("(")]
        op["full_name"] = full_name
        ret_type = func_str[func_str.index("->") + 3 :]
        op["ret_type"] = ret_type
        cnt += 1
```
- **EN**: This chunk continues `gen_data` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `gen_data`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 51-68
```python
        if uniq_name in uniq_names:
            continue
        uniq_names.add(uniq_name)
        uniq_ops.append(op)

    def annotate_ops(ops, is_unique):
        categorization = defaultdict(int)
        for op in ops:
            if op["name"][-1] == "_":
                categorization["inplace"] += 1
                op["meta"] = "inplace"
                continue
            if not is_unique and "a!" in op["func"].lower():
                categorization["out"] += 1
                op["meta"] = "out"
                continue
            if "conv" in op["name"]:
                categorization["conv"] += 1
```
- **EN**: This chunk defines `annotate_ops`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `annotate_ops`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 69-86
```python
                op["meta"] = "conv"
                continue
            if "pool" in op["name"]:
                categorization["pool"] += 1
                op["meta"] = "pool"
                continue
            if "backward" in op["name"]:
                categorization["backward"] += 1
                op["meta"] = "backward"
                continue
            if op["name"][0] == "_" and op["name"][1] != "_":
                categorization["private"] += 1
                op["meta"] = "private"
                continue
            if "batch_norm" in op["name"]:
                categorization["batch_norm"] += 1
                op["meta"] = "batch_norm"
                continue
```
- **EN**: This chunk continues `annotate_ops` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `annotate_ops`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 87-104
```python
            if "Tensor" not in op["func"] or "Tensor" not in op["ret_type"]:
                categorization["non_tensor"] += 1
                op["meta"] = "non_tensor"
                continue
            if (
                "cudnn" in op["name"]
                or "mkldnn" in op["name"]
                or "miopen" in op["name"]
                or "native" in op["name"]
                or "thnn" in op["name"]
                or "slow" in op["name"]
            ):
                categorization["backend"] += 1
                op["meta"] = "backend"
                continue
            if op["name"] in annotated_ops:
                categorization["core"] += 1
                op["meta"] = "core " + annotated_ops[op["name"]]
```
- **EN**: This chunk continues `annotate_ops` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `annotate_ops`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 105-118
```python
                continue
            categorization["core"] += 1
            op["meta"] = "core unknown"
        return categorization

    annotate_ops(ops, is_unique=False)
    with open(f"{analysis_name}", "w") as f:
        for op in ops:
            info = [
                op["full_name"],
                op["meta"],
                op["full_name"] not in noncomposite_ops,
            ] + [check(op) for check in special_op_lists]
            f.write(",".join([str(i) for i in info]) + "\n")
```
- **EN**: This chunk continues `annotate_ops` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `annotate_ops`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 121-136
```python
def name_check(lst):
    return lambda x: x["name"] in lst


def full_name_check(lst):
    return lambda x: x["full_name"] in lst


# Generates batching rule data
gen_data([full_name_check(get_ops_for_key("FuncTorchBatched"))], "vmap.txt")


def remove_suffix(input_string, suffix):
    if suffix and input_string.endswith(suffix):
        return input_string[: -len(suffix)]
    return input_string
```
- **EN**: This chunk defines `remove_suffix`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `remove_suffix`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 139-153
```python
def remove_prefix(input_string, prefix):
    if prefix and input_string.startswith(prefix):
        return input_string[len(prefix) :]
    return input_string


if True:
    with open("run_ops.txt") as f:
        opinfo_ops = [remove_suffix(i.strip(), ".default") for i in f]
    with open("count_ops.txt") as f:
        opinfo_counts = [i.strip() for i in f]
        opinfo_counts = defaultdict(int, dict(zip(opinfo_ops, opinfo_counts)))

    def count_fn(x):
        return opinfo_counts[x["full_name"]]
```
- **EN**: This chunk defines `count_fn`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `count_fn`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 155-168
```python
    with open("run_decompositions.txt") as f:
        decomposed_ops = [remove_suffix(i.strip(), ".default") for i in f]

    with open("public_api") as f:
        ref_api = [i.strip() for i in f]

    def has_ref_impl(x):
        name = x["name"]
        for prefix in ["linalg_", "special_"]:
            name = remove_prefix(name, prefix)
        prefixes = ["nn.functional", "fft", "special", "linalg"]
        return (
            any(f"{prefix}.{name}" in ref_api for prefix in prefixes) or name in ref_api
        )
```
- **EN**: This chunk defines `has_ref_impl`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `has_ref_impl`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 170-178
```python
    gen_data(
        [
            full_name_check(opinfo_ops),
            full_name_check(decomposed_ops),
            count_fn,
            has_ref_impl,
        ],
        "decompositions.txt",
    )
```
- **EN**: This chunk continues `has_ref_impl` and expands its control flow, data preparation, or emitted structure.
- **CN**: 这一段延续了 `has_ref_impl`，继续展开其控制流、数据准备或生成结构。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **YAML schema loading**
  - EN: Parses YAML operator metadata that drives later code generation.
  - CN: 解析驱动后续代码生成的 YAML 算子元数据。
- **Dispatch-key plumbing**
  - EN: Models dispatch keys, backend selection, or registration routing.
  - CN: 建模 dispatch key、后端选择或注册路由。
- **Autograd generation**
  - EN: Produces or organizes derivative-related wrappers, bindings, and metadata.
  - CN: 生成或组织与求导相关的包装器、绑定与元数据。
- **Batching / vmap**
  - EN: Coordinates vectorized mapping rules and batched operator behavior.
  - CN: 协调向量化映射规则与 batched 算子行为。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`
- **Third-party modules / 第三方模块**: `yaml`
- **Standard library / 标准库**: `csv`, `collections`
- **Primary symbols / 核心符号**: `get_ops_for_key`, `gen_data`, `annotate_ops`, `name_check`, `full_name_check`, `remove_suffix`, `remove_prefix`, `count_fn`, `has_ref_impl`
