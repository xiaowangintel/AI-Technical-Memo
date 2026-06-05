# _triton_ops_meta.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/sparse/_triton_ops_meta.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements sparse-tensor namespace wrappers, sparse utilities, and sparse runtime helpers.
- **Purpose (CN)**: 实现稀疏张量命名空间包装层、稀疏工具以及稀疏运行时辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-34 / 第 1-34 行
````python
# mypy: allow-untyped-defs
"""Provides optimal triton kernel parameters.

Aim
---

The usage of optimal triton kernel parameters may increase the
performance of operations several times. For example, for large tensor
shapes, the usage of a bsr tensor as mat1 argument in addmm-based
operations typically outperforms the corresponding operation with
strided-only inputs when the blocked representation of a tensor
provides a better alignment with memory access than what the strided
representation would provide.

Pre-computed kernel parameters
------------------------------

This script finds and stores the optimal triton kernel parameters for
a specific set of shape configurations. For instance, the set of shape
configurations of the bsr_dense_addmm kernel is defined as

  input, out: M x N strided tensor
  mat1: M x K bsr tensor with blocksize (BM, BK) and given sparsity
  mat2: M x N strided tensor
  dtype = float16, bfloat16, float32
  sparsity = 0.5
  M = 256, 512, ..., 16384
  K = M
  N = 256, 512, ..., 131072
  BM = 16, 32, ..., 128
  BK = BM
  alpha = 1
  beta = 0, 1
  GPUs: NVIDIA A100-SXM4-80GB
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 36-64 / 第 36-64 行
````python
Approximations
--------------

It is practically infeasible to pre-compute optimal kernel parameter
for all possible shape configurations as well as for all existing
GPUs. Therefore, we'll assume that the pre-computed optimal parameters
are good enough approximations when
1) the used GPU is any of NVIDIA A100 Tensor Core GPUs,
2) the actual sparsity of mat1 is different from sparsity value 0.5.

If a particular shape configuration does not fall in the set of
pre-computed kernel parameters, or it does not match with the listed
approximations above, or the used GPU device is not a NVIDIA A100 GPU,
then a reference set of triton kernel parameters will be used when
executing operations. The reference kernel parameters are defined in
torch/sparse/_triton_ops.py, see bsr_dense_addmm_meta function, for
instance.

Computing optimal kernel parameters
-----------------------------------

If the approximations listed above are unacceptable, e.g. when one
seeks a maximal performance possible, the optimal kernel parameters
for a particular GPU can be computed by simply running this script in
the pytorch development tree::

  cd /path/to/pytorch
  python -m pip install --no-build-isolation -v -e .
  python torch/sparse/_triton_ops_meta.py
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 66-99 / 第 66-99 行
````python
This will compute the optimal kernel parameters for the GPU device
available in the host system for all shape configurations listed in
"Pre-computed kernel parameters" above. The results will be stored in
the database of kernel parameters. Currently, this database is defined
as this module (see "BEGIN GENERATED DATA" comment below) that will be
modified when the script is run. Create a pytorch PR with the
corresponding modifications in this file to make the computed optimal
kernel parameters available for other users as pre-computed kernel
parameters.

Moreover, one can compute the optimal kernel parameters for a specific
set of shape configurations and specific sparsity patterns. For that,
use tuning functions provided by this module:

  tune_bsr_dense_addmm(input, mat1, mat2, beta=1, alpha=1, out=None, verbose=False, store=False) -> meta

The tuning functions return a dictionary of optimal kernel parameters
that can be passed to the corresponding operation, e.g.

  bsr_dense_addmm(..., meta=meta)

Or, when store==True, the optimal kernel parameters will be stored in
the database of pre-computed kernel parameters in runtime so that all
addmm-based operations such as torch.addmm, torch.mm,
torch.nn.functional.linear will benefit from using the computed
optimal set of kernel parameters.

Note that running tune_bsr_dense_addmm can take several minutes. So,
use it wisely, e.g. by implementing persistent storage of optimized
kernel parameters. See the source code of get_meta and
tune_bsr_dense_addmm to learn how to register a custom set of optimal
kernel parameters for addmm-based operations.

"""
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 101-124 / 第 101-124 行
````python
__all__ = ["get_meta", "tune_bsr_dense_addmm", "tune__int_bsr_dense_addmm"]

import inspect
import itertools
import re
import warnings
from typing import Any

import torch
from torch.hub import tqdm
from torch.testing import make_tensor


def _get_device_name() -> str:
    """Return the current accelerator device name for use as a Triton tuning cache key."""
    if torch.cuda.is_available():
        return torch.cuda.get_device_name()
    if torch.xpu.is_available():
        return torch.xpu.get_device_name()
    return ""


def get_meta(op, key, device_name=None, version=(0, torch.float16, 0.5), exact=False):
    """Return triton kernel meta parameters of the specified op and its inputs key.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.hub, torch.testing; standard-library helpers such as inspect, itertools, re, .... `__all__` defines the public symbols that this module chooses to export. This chunk defines `get_meta`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.hub、torch.testing；标准库辅助模块，如 inspect、itertools、re、...。 `__all__` 定义了本模块选择导出的公共符号。 这一段定义了 `get_meta`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 126-161 / 第 126-161 行
````python
    Parameters
    ----------
    op (str): The name of an operation that implementation uses meta parameters.
    key (tuple): A tuple of op input parameters, e.g. shapes, etc.
    device_name (optional, str): The name of a device for which op
      parameters are provided.
    version (optional, hashable): Specifies the version of parameters.
    exact (optional, bool): When True, the returned data (if
      available) corresponds exactly to the specified device_name and
      version information. Otherwise, if the corresponding data is not
      available but there exists a data set that is computed for a
      similar GPU device, then this data set will be returned.

    Returns
    -------
    result (dict): The requested mapping of parameter names and
      values, or None when no data is available. If the input `key`
      contains `"*"`, the result will be a dictionary of keys and
      mappings that match with the given `key`.
    """
    if device_name is None:
        device_name = _get_device_name()

    op_data = _operation_device_version_data.get((op, device_name, version))
    if op_data is None and not exact:
        # A lack of op data could be due to using a (slightly)
        # different GPU model compared to a model for which optimal
        # meta parameters have been computed. In the following we'll
        # assume that there is a set of GPU models that all have
        # a similar set of optimal meta parameters.
        if (
            device_name is not None
            and re.match(r"NVIDIA A100[^\d]", device_name) is not None
        ):
            device_name = "NVIDIA A100-SXM4-80GB"
        else:
````
- **EN**: This chunk continues `get_meta` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `get_meta`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 162-197 / 第 162-197 行
````python
            return
        op_data = _operation_device_version_data.get((op, device_name, version))
    if op_data is None:
        return

    matching_data = {}
    if "*" in key:
        for op_key in op_data:
            if [
                None
                for k1, k2 in zip(op_key, key, strict=True)
                if k2 != "*" and k1 != k2
            ]:
                continue
            matching_data[op_key] = op_data[op_key]
    else:
        values = op_data.get(key)
        if values is not None:
            matching_data[key] = values
    matching_meta = {}
    for op_key, values in matching_data.items():
        if op == "scatter_mm":
            names = (
                "GROUP_SIZE",
                "SPLIT_N",
                "TILE_M",
                "TILE_N",
                "num_stages",
                "num_warps",
            )
            meta = dict(zip(names, values, strict=True))
        elif op in {"bsr_dense_addmm", "_int_bsr_dense_addmm"}:
            meta = dict(
                zip(
                    ("GROUP_SIZE_ROW", "SPLIT_N", "num_stages", "num_warps"),
                    values,
````
- **EN**: This chunk continues `get_meta` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_meta`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 198-226 / 第 198-226 行
````python
                    strict=True,
                )
            )
        else:
            raise NotImplementedError(f"names for {op=}")
        if "*" not in key:
            return meta

        matching_meta[op_key] = meta

    if "*" in key:
        return matching_meta


def update(op, device_name, version, key, value):
    """Update the db of op parameters."""
    # skip storing possible optimization failures:
    if not value:
        warnings.warn(
            f"skipping empty value for {op}: {device_name=} {version=} {key=}",
            stacklevel=2,
        )
        return
    if (op, device_name, version) in _operation_device_version_data:
        if _operation_device_version_data[op, device_name, version].get(key) == value:
            return
        _operation_device_version_data[op, device_name, version][key] = value
    else:
        _operation_device_version_data[op, device_name, version] = {key: value}
````
- **EN**: This chunk defines `update`, which advances mutable state using the current inputs, gradients, or counters. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `update`，其作用是利用当前输入、梯度或计数器推进可变状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 229-263 / 第 229-263 行
````python
def dump():
    """Store the current runtime db state to the module file."""
    current_file = inspect.getfile(dump)
    with open(current_file) as f:
        current_content = f.read()
    begin_data_str = "# BEGIN GENERATED DATA\n"
    begin_data_index = current_content.find(begin_data_str)
    end_data_index = current_content.find("    # END GENERATED DATA\n")
    if begin_data_index == -1 or end_data_index == -1:
        warnings.warn(
            f"{current_file} cannot be updated:"
            " BEGIN/END GENERATED DATA comment blocks appear to be corrupted",
            stacklevel=2,
        )
        return

    def sort_key(key):
        op, device_name, version = key
        version = tuple(
            (str(item) if isinstance(item, torch.dtype) else item) for item in version
        )
        return (op, device_name, version)

    part1 = current_content[: begin_data_index + len(begin_data_str)]
    part2 = current_content[end_data_index:]
    data_part = []
    for op_key in sorted(_operation_device_version_data, key=sort_key):
        data_part.append("    " + repr(op_key).replace("'", '"') + ": {")
        op_data = _operation_device_version_data[op_key]
        data_part.extend(f"        {key}: {op_data[key]}," for key in sorted(op_data))
        data_part.append("    },")
    new_content = part1 + "\n".join(data_part) + "\n" + part2
    if current_content != new_content:
        with open(current_file, "w") as f:
            f.write(new_content)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `sort_key`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `sort_key`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 266-292 / 第 266-292 行
````python
def minimize(
    target_func,
    initial_parameters,
    reference_parameters,
    step_func,
    max_step=2,
    verbose=False,
    all_values=None,
):
    """Find a dict of parameters that minimizes the target function using
    the initial dict of parameters and a step function that progresses
    a specified parameter in a dict of parameters.

    Parameters
    ----------
    target_func (callable): a functional with the signature
      ``target_func(parameters: dict) -> float``
    initial_parameters (dict): a set of parameters used as an initial
      value to the minimization process.
    reference_parameters (dict): a set of parameters used as an
      reference value with respect to which the speed up is computed.
    step_func (callable): a functional with the signature
      ``step_func(parameter_name:str, parameter_value:int, direction:int, parameters:dict) -> int``
      that increments or decrements (when ``direction`` is positive or
      negative, respectively) the parameter with given name and value.
      When return value is equal to ``parameter_value``, it means that
      no step along the given direction can be made.
````
- **EN**: This chunk defines `minimize`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `minimize`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 294-327 / 第 294-327 行
````python
    Returns
    -------
    parameters (dict): a set of parameters that minimizes the target
      function.
    speedup_incr (float): a speedup change given in percentage.
    timing (float): the value of the target function at the parameters.
    sensitivity_message (str): a message containing sensitivity.
      information of parameters around the target function minimizer.
    """

    def to_key(parameters):
        return tuple(parameters[k] for k in sorted(parameters))

    def from_key(key, parameters):
        return dict(zip(sorted(parameters), key, strict=True))

    if all_values is None:
        all_values = {}

    directions = list(range(-max_step, max_step + 1))
    names = sorted(initial_parameters)
    all_directions = []
    for d_tuple in itertools.product(*((directions,) * len(names))):
        dist = sum(map(abs, d_tuple))
        if dist > 0 and dist <= max_step:
            all_directions.append((dist, d_tuple))
    all_directions.sort()

    try:
        reference_target = target_func(reference_parameters)
    except Exception as msg:
        if verbose and "out of resource" not in str(msg):
            print(f"{reference_parameters=} lead to failure: {msg}.")
        reference_target = None
````
- **EN**: This chunk defines `from_key`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `from_key`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 329-352 / 第 329-352 行
````python
    if reference_target is not None:
        all_values[to_key(reference_parameters)] = reference_target

    parameters = initial_parameters
    try:
        initial_target = target_func(parameters)
    except Exception as msg:
        if reference_target is None:
            if verbose:
                print(
                    f"{initial_parameters=} lead to failure: {msg}. Optimization failed!"
                )
            return {}, -1, -1, f"{msg}"
        if verbose and "out of resource" not in str(msg):
            print(
                f"{initial_parameters=} lead to failure: {msg}. Using reference parameters instead of initial parameters."
            )
        parameters = reference_parameters
        initial_target = reference_target

    if reference_target is None:
        if verbose:
            print("Using initial parameters instead of reference parameters.")
        reference_target = initial_target
````
- **EN**: This chunk continues `from_key` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `from_key`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 354-385 / 第 354-385 行
````python
    initial_key = to_key(parameters)
    minimal_target = all_values[initial_key] = initial_target
    pbar = tqdm(
        total=len(all_directions),
        desc="Tuning...",
        disable=not verbose,
        ncols=75,
    )
    while True:
        for i, (_, d_tuple) in enumerate(all_directions):
            pbar.update(1)
            next_parameters = parameters.copy()
            for name, direction in zip(names, d_tuple, strict=True):
                value = next_parameters[name]
                if direction == 0:
                    continue
                next_value = step_func(name, value, direction, parameters)
                if next_value == value:
                    break
                next_parameters[name] = next_value
            else:
                next_key = to_key(next_parameters)
                if next_key in all_values:
                    continue
                try:
                    next_target = target_func(next_parameters)
                except Exception as msg:
                    all_values[next_key] = str(msg)
                    if verbose and "out of resource" not in str(msg):
                        print(f"{next_parameters=} lead to failure: {msg}. Skipping.")
                    continue
                all_values[next_key] = next_target
````
- **EN**: This chunk continues `from_key` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `from_key`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 387-422 / 第 387-422 行
````python
                if next_target < minimal_target:
                    minimal_target = next_target
                    parameters = next_parameters
                    # pyrefly: ignore [unsupported-operation]
                    pbar.total += i + 1
                    break
        else:
            # ensure stable minimizer:
            minimizer_keys = {
                k
                for k, v in all_values.items()
                if isinstance(v, float) and abs(1 - v / minimal_target) < 0.001
            }
            minimizer_key = (
                initial_key if initial_key in minimizer_keys else min(minimizer_keys)
            )
            parameters = from_key(minimizer_key, parameters)
            speedup_incr = (1 - minimal_target / reference_target) * 100
            if speedup_incr < 0:
                if verbose:
                    print(
                        f"{speedup_incr=} is negative. Rerunning minimize with reference parameters as initial parameters."
                    )
                return minimize(
                    target_func,
                    reference_parameters,
                    reference_parameters,
                    step_func,
                    max_step=max_step,
                    verbose=verbose,
                    all_values=all_values,
                )
            sensitivity = []
            for name in parameters:
                value = parameters[name]
                rel_diffs = []
````
- **EN**: This chunk continues `from_key` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `from_key`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 423-453 / 第 423-453 行
````python
                for direction in range(-max_step, max_step + 1):
                    if direction == 0:
                        continue
                    next_value = step_func(name, value, direction, parameters)
                    if next_value == value:
                        rel_diffs.append(0)
                        continue
                    next_parameters = parameters.copy()
                    next_parameters[name] = next_value
                    next_key = to_key(next_parameters)
                    next_target = all_values.get(next_key)
                    if next_target is None or isinstance(next_target, str):
                        rel_diffs.append(0)
                        continue
                    rel_diff = (next_target / minimal_target - 1) * 100
                    rel_diffs.append(rel_diff)
                sensitivity.append((max(rel_diffs), rel_diffs, name))

            sensitivity_message = [f"timing0={initial_target:.3f}"]
            for _, rel_diffs, name in sorted(sensitivity, reverse=True):
                left_diffs = "|".join(
                    [f"{rel_diff:.1f}" for rel_diff in rel_diffs[:max_step]]
                )
                right_diffs = "|".join(
                    [f"{rel_diff:.1f}" for rel_diff in rel_diffs[max_step:]]
                )
                sensitivity_message.append(
                    f"{name}={parameters[name]} ({left_diffs}...{right_diffs} %)"
                )
            sensitivity_message = ", ".join(sensitivity_message)
            return parameters, speedup_incr, minimal_target, sensitivity_message
````
- **EN**: This chunk continues `from_key` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `from_key`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 456-485 / 第 456-485 行
````python
def create_blocked_tensor(B, M, N, blocksize, sparsity, dtype, device):
    if sparsity < 0.0 or sparsity > 1.0:
        raise AssertionError(f"sparsity should be between 0 and 1, got {sparsity}")
    if M % blocksize[0] != 0:
        raise AssertionError(
            f"M ({M}) must be divisible by blocksize[0] ({blocksize[0]})"
        )
    if N % blocksize[1] != 0:
        raise AssertionError(
            f"N ({N}) must be divisible by blocksize[1] ({blocksize[1]})"
        )
    shape = (B, M // blocksize[0], N // blocksize[1])[int(B == 0) :]
    A = torch.bernoulli(
        torch.full(shape, 1 - sparsity, dtype=torch.float32, device=device)
    ).to(dtype)
    expected_nnz = int((1 - sparsity) * M * N / (blocksize[0] * blocksize[1]))
    nonzero_indices = A.flatten().nonzero()
    actual_nnz = nonzero_indices.shape[0]
    if actual_nnz > expected_nnz:
        selected_nonzeros = torch.randperm(actual_nnz)[: actual_nnz - expected_nnz]
        A.flatten()[nonzero_indices[selected_nonzeros]] = 0
    elif actual_nnz < expected_nnz:
        zero_indices = (A == 0).flatten().nonzero()
        selected_zeros = torch.randperm(zero_indices.shape[0])[
            : expected_nnz - actual_nnz
        ]
        A.flatten()[zero_indices[selected_zeros]] = 1
    A = torch.repeat_interleave(A, blocksize[0], dim=-2)
    A = torch.repeat_interleave(A, blocksize[1], dim=-1)
    return A
````
- **EN**: This chunk defines `create_blocked_tensor`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `create_blocked_tensor`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 488-523 / 第 488-523 行
````python
def optimize_scatter_mm(
    m, k, n, bm, bk, dtype=torch.float16, device="cuda", sparsity=0.5, force=False
):
    import triton

    from torch.sparse._triton_ops import bsr_scatter_mm, bsr_scatter_mm_indices_data

    key = (m, k, n, bm, bk)

    version = (0, dtype, sparsity)
    device_name = _get_device_name()

    reference_meta = dict(
        GROUP_SIZE=1,
        TILE_M=16,
        TILE_N=16,
        SPLIT_N=n // 16,
        num_stages=1,
        num_warps=1,
    )

    initial_meta = get_meta(
        "scatter_mm", key, device_name=device_name, version=version, exact=True
    )
    if initial_meta is None:
        initial_meta = get_meta(
            "bsr_dense_addmm",
            key,
            device_name=device_name,
            version=(0, dtype, 0.5),
            exact=True,
        )
        if initial_meta is None:
            initial_meta = reference_meta
    elif not force:
        return
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.sparse._triton_ops; other helper packages such as triton. This chunk defines `optimize_scatter_mm`, which coordinates collective-style data movement or aggregation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.sparse._triton_ops；其他辅助包，如 triton。 这一段定义了 `optimize_scatter_mm`，其作用是协调类似集合通信的数据移动或聚合。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 525-560 / 第 525-560 行
````python
    torch.manual_seed(0)
    bsr = create_blocked_tensor(
        0, m, k, (bm, bk), sparsity, dtype, device
    ).to_sparse_bsr((bm, bk))
    dense = make_tensor(k, n, dtype=dtype, device=device)

    def bench(meta, bsr=bsr, dense=dense):
        indices_data = bsr_scatter_mm_indices_data(
            bsr, dense, indices_format="bsr_strided_mm_compressed", **meta
        )

        def test_func():
            return bsr_scatter_mm(bsr, dense, indices_data=indices_data)

        ms_min = triton.testing.do_bench(test_func, warmup=500, rep=100)

        return ms_min

    def step_meta_parameter(name, value, direction, meta, m=m, n=n, k=k, bm=bm, bk=bk):
        # return next value in positive or negative direction, or
        # input value if the step will result an invalid
        # value. The input value is assumed to be valid.

        is_log = name in {"SPLIT_N", "TILE_M", "TILE_N", "num_warps"}
        min_value = dict(
            SPLIT_N=1, TILE_M=16, TILE_N=16, num_warps=1, num_stages=1, GROUP_SIZE=1
        )[name]
        max_value = dict(
            SPLIT_N=n // meta["TILE_N"], TILE_M=bm, TILE_N=n // meta["SPLIT_N"]
        ).get(name)
        value_step = dict(
            SPLIT_N=2, TILE_M=2, TILE_N=2, num_warps=2, num_stages=1, GROUP_SIZE=1
        )[name]
        if is_log:
            next_value = (
                value * value_step**direction
````
- **EN**: This chunk defines `step_meta_parameter`, which advances mutable state using the current inputs, gradients, or counters. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `step_meta_parameter`，其作用是利用当前输入、梯度或计数器推进可变状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 561-595 / 第 561-595 行
````python
                if direction > 0
                else value // (value_step ** abs(direction))
            )
        else:
            next_value = value + value_step * direction
        if min_value is not None:
            next_value = max(next_value, min_value)
        if max_value is not None:
            next_value = min(next_value, max_value)
        if name == "SPLIT_N" and n % next_value != 0:
            return value
        # Hard-skip parameter combinations that break CUDA state for pytorch:
        if (dtype, name, next_value, m, n, k, bm, bk) in {
            (torch.float32, "num_warps", 32, 256, 256, 256, 16, 16),
            (torch.float32, "num_warps", 16, 256, 256, 256, 32, 32),
            (torch.float32, "num_warps", 16, 256, 256, 256, 64, 64),
            (torch.float32, "num_warps", 16, 256, 256, 256, 128, 128),
            (torch.float32, "num_warps", 16, 512, 512, 256, 128, 128),
        } and re.match(r"NVIDIA A100[^\d]", device_name) is not None:
            return value
        return next_value

    meta, speedup, timing, _sensitivity_message = minimize(
        bench, initial_meta, reference_meta, step_meta_parameter
    )
    if initial_meta is not reference_meta and initial_meta == meta and not force:
        return
    print(f"{meta=} {speedup=:.1f} % {timing=:.3f} ms")
    if speedup < 0:
        return
    device_name = _get_device_name()

    update(
        "scatter_mm", device_name, version, key, tuple(meta[k] for k in sorted(meta))
    )
````
- **EN**: This chunk continues `step_meta_parameter` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `step_meta_parameter`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 598-621 / 第 598-621 行
````python
def tune__int_bsr_dense_addmm(
    input,
    bsr,
    dense,
    *,
    beta=1,
    alpha=1,
    out=None,
    store=False,
    verbose=False,
    force=False,
):
    return tune_bsr_dense_addmm(
        input,
        bsr,
        dense,
        beta=beta,
        alpha=alpha,
        out=out,
        store=store,
        verbose=verbose,
        force=force,
        opname="_int_bsr_dense_addmm",
    )
````
- **EN**: This chunk defines `tune__int_bsr_dense_addmm`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `tune__int_bsr_dense_addmm`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 624-652 / 第 624-652 行
````python
def tune_bsr_dense_addmm(
    input,
    bsr,
    dense,
    *,
    beta=1,
    alpha=1,
    left_alpha=None,
    right_alpha=None,
    out=None,
    store=False,
    verbose=False,
    force=False,
    opname=None,
):
    """Tune bsr_dense_addmm kernel parameters against the given inputs.

    When store is True, the tuning results will be stored in the
    database of kernel parameters.
    """
    import triton

    if opname is None:
        opname = "bsr_dense_addmm"

    if opname == "_int_bsr_dense_addmm":
        from torch.sparse._triton_ops import _int_bsr_dense_addmm as bsr_dense_addmm
    else:
        from torch.sparse._triton_ops import bsr_dense_addmm
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.sparse._triton_ops; other helper packages such as triton. This chunk defines `tune_bsr_dense_addmm`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.sparse._triton_ops；其他辅助包，如 triton。 这一段定义了 `tune_bsr_dense_addmm`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 654-681 / 第 654-681 行
````python
    N = dense.shape[-1]
    values = bsr.values()
    crow_indices = bsr.crow_indices()
    batch_ndim = crow_indices.dim() - 1
    M, K = bsr.shape[batch_ndim : batch_ndim + 2]
    BM, BK = values.shape[batch_ndim + 1 : batch_ndim + 3]

    # Reference parameters is a set of parameters that leads to a
    # successful kernel call and the corresponding timing is used as a
    # reference for computing speedups. Avoid changing the reference
    # parameters when possible.
    reference_meta = dict(
        GROUP_SIZE_ROW=1, num_stages=1, num_warps=4, SPLIT_N=max(N // BM, 1)
    )

    # Compute the key of parameters:
    sparsity = round(1 - bsr._nnz() * BM * BK / (M * K), 2)
    dtype = bsr.dtype
    if out is None:
        out_dtype = dtype
    else:
        out_dtype = out.dtype
    if out_dtype is dtype:
        version_dtype = dtype
    else:
        version_dtype = (dtype, out_dtype)
    version = (0, version_dtype, sparsity)
    key = (M, K, N, BM, BK, beta == 0, beta == 1, alpha == 1)
````
- **EN**: This chunk continues `tune_bsr_dense_addmm` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `tune_bsr_dense_addmm`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 683-711 / 第 683-711 行
````python
    # For tuning, for an initial state, use parameters from the
    # database if available, otherwise, use the reference parameters.
    initial_meta = get_meta(opname, key, version=version, exact=True)
    if initial_meta is None:
        may_skip_update = False
        initial_meta = get_meta(opname, key, version=(0, dtype, 0.5), exact=True)
        if initial_meta is None:
            initial_meta = reference_meta
    elif not force:
        return initial_meta
    else:
        may_skip_update = True

    # The target function that is minimized in the tuning process:
    def bench(meta, input=input, bsr=bsr, dense=dense, alpha=alpha, out=out):
        def test_func():
            return bsr_dense_addmm(
                input,
                bsr,
                dense,
                beta=beta,
                alpha=alpha,
                left_alpha=left_alpha,
                right_alpha=right_alpha,
                meta=meta,
                out=out,
            )

        return triton.testing.do_bench(test_func, warmup=500, rep=100)
````
- **EN**: This chunk defines `test_func`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `test_func`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 713-736 / 第 713-736 行
````python
    # The step function that increments a specified meta parameter:
    def step_meta_parameter(name, value, direction, meta, M=M, N=N, K=K, BM=BM, BK=BK):
        # return next value in positive or negative direction, or
        # input value if the step will result an invalid
        # value. The input value is assumed to be valid.
        is_log = name in {"SPLIT_N", "num_warps"}
        min_value = dict(SPLIT_N=1, num_warps=1, num_stages=1, GROUP_SIZE_ROW=1)[name]
        max_value = dict(SPLIT_N=max(N // BM, 1)).get(name)
        value_step = dict(SPLIT_N=2, num_warps=2, num_stages=1, GROUP_SIZE_ROW=1)[name]
        if is_log:
            next_value = (
                value * value_step**direction
                if direction > 0
                else value // (value_step ** abs(direction))
            )
        else:
            next_value = value + value_step * direction
        if min_value is not None:
            next_value = max(next_value, min_value)
        if max_value is not None:
            next_value = min(next_value, max_value)
        if name == "SPLIT_N" and N % next_value != 0:
            return value
        return next_value
````
- **EN**: This chunk defines `step_meta_parameter`, which advances mutable state using the current inputs, gradients, or counters. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `step_meta_parameter`，其作用是利用当前输入、梯度或计数器推进可变状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 738-762 / 第 738-762 行
````python
    # Tune:
    meta, speedup, timing, sensitivity_message = minimize(
        bench,
        initial_meta,
        reference_meta,
        step_meta_parameter,
        max_step=2,
        verbose=verbose,
    )
    if verbose:
        print(f"-> {sensitivity_message}, {speedup=:.1f} %, {timing=:.3f} ms")

    if store and not (
        may_skip_update and meta == initial_meta and initial_meta is not reference_meta
    ):
        device_name = _get_device_name()
        update(
            opname,
            device_name,
            version,
            key,
            tuple(meta[k] for k in sorted(meta)),
        )

    return meta
````
- **EN**: This chunk continues `step_meta_parameter` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `step_meta_parameter`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 765-800 / 第 765-800 行
````python
def optimize_bsr_dense_addmm(
    m,
    k,
    n,
    bm,
    bk,
    beta=1,
    alpha=1,
    use_left_alpha=False,
    use_right_alpha=False,
    dtype=torch.float16,
    out_dtype=None,
    device="cuda",
    sparsity=0.5,
    force=False,
    verbose=False,
    opname=None,
):
    torch.manual_seed(0)
    bsr = create_blocked_tensor(
        0, m, k, (bm, bk), sparsity, dtype, device
    ).to_sparse_bsr((bm, bk))
    dense = make_tensor(k, n, dtype=dtype, device=device)
    input = make_tensor(m, n, dtype=dtype, device=device)
    left_alpha = make_tensor(m, dtype=dtype, device=device) if use_left_alpha else None
    right_alpha = (
        make_tensor(n, dtype=dtype, device=device) if use_right_alpha else None
    )
    if out_dtype is not None:
        out = dense.new_empty((m, n), dtype=out_dtype)
    else:
        out = None
    tune_bsr_dense_addmm(
        input,
        bsr,
        dense,
````
- **EN**: This chunk defines `optimize_bsr_dense_addmm`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `optimize_bsr_dense_addmm`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 801-836 / 第 801-836 行
````python
        beta=beta,
        alpha=alpha,
        left_alpha=left_alpha,
        right_alpha=right_alpha,
        out=out,
        store=True,
        force=force,
        verbose=verbose,
        opname=opname,
    )


def main(op="scatter_mm", force=False, dtype=torch.float16, verbose=True):
    import itertools

    sizes_lst = [
        256,
        512,
        1024,
        2048,
        4096,
        8192,
        16384,
        32768,
        65536,
        131072,
        50432,
        65792,
    ]
    sizes3_lst = [3 * sz for sz in [64, 128] + sizes_lst if sz <= 2048]
    shapes_lst = [(sz, sz) for sz in sizes_lst[:-4] + sizes3_lst]
    shapes_lst.extend([(3072, 768), (768, 3072)])
    shapes_lst.extend([(5120, 1280), (1280, 5120)])
    if dtype is torch.int8:
        # triton does not support smaller blocks than 32
        blocksize_lst = [(32, 32), (64, 64), (128, 128), (256, 256)]
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as itertools. This chunk defines `main`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 itertools。 这一段定义了 `main`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 837-872 / 第 837-872 行
````python
    else:
        blocksize_lst = [(16, 16), (32, 32), (64, 64), (128, 128)]
    sparsity_lst = [0.5, 0.7, 0.3][:1]
    for sparsity in sparsity_lst:
        print(f"{op, dtype, sparsity=}")
        try:
            for (M, K), N, (BM, BK) in itertools.product(
                shapes_lst, sizes_lst, blocksize_lst
            ):
                if not (BM <= M and BK <= K and M % BM == 0 and K % BK == 0):
                    continue
                if op == "scatter_mm":
                    optimize_scatter_mm(
                        M, K, N, BM, BK, force=force, sparsity=sparsity, dtype=dtype
                    )
                elif op in {"bsr_dense_addmm", "_int_bsr_dense_addmm"}:
                    if M == K and N == 50432:
                        continue
                    print(f"{M, K, N, (BM, BK)=}")
                    for alpha, beta in [(1, 1), (1, 0)]:
                        optimize_bsr_dense_addmm(
                            M,
                            K,
                            N,
                            BM,
                            BK,
                            beta=beta,
                            alpha=alpha,
                            force=force,
                            sparsity=sparsity,
                            dtype=dtype,
                            verbose=verbose,
                            opname=op,
                        )
                else:
                    raise NotImplementedError(op)
````
- **EN**: This chunk continues `main` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 873-907 / 第 873-907 行
````python
        except KeyboardInterrupt:
            break
        except Exception:
            dump()
            raise
    dump()

    if 0:
        # Check performance dependence on sparsity and apply
        # adjustments when differences are noticeable (more than 10%).
        #
        # When using NVIDIA A100 GPU, the performance dependence on
        # sparsity is insignificant (0 % ... 10 %) for majority of
        # shapes/blocksizes combinations. However, for a very few
        # specific size combinations, the effect of sparsity on
        # performance can be up to 20 %.
        for (M, K), N, (BM, BK) in itertools.product(
            shapes_lst, sizes_lst, blocksize_lst
        ):
            meta_lst: list = []
            key = (M, K, N, BM, BK)
            for sparsity1 in sparsity_lst:
                torch.manual_seed(0)
                bsr = create_blocked_tensor(
                    0, M, K, (BM, BK), sparsity1, dtype, device="cuda"
                ).to_sparse_bsr((BM, BK))
                dense = make_tensor(K, N, dtype=dtype, device="cuda")
                meta_lst = []
                for sparsity in sparsity_lst:
                    meta = get_meta(op, key, version=(0, dtype, sparsity), exact=True)
                    if meta is None:
                        continue

                    def bench(meta, bsr=bsr, dense=dense):
                        import triton
````
- **EN**: This block assembles module dependencies, pulling in other helper packages such as triton. This chunk defines `bench`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段组织模块依赖，引入了其他辅助包，如 triton。 这一段定义了 `bench`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 909-932 / 第 909-932 行
````python
                        if op == "scatter_mm":
                            from torch.sparse._triton_ops import (
                                bsr_scatter_mm,
                                bsr_scatter_mm_indices_data,
                            )

                            indices_data = bsr_scatter_mm_indices_data(
                                bsr,
                                dense,
                                indices_format="bsr_strided_mm_compressed",
                                **meta,
                            )

                            def test_func():
                                return bsr_scatter_mm(
                                    bsr, dense, indices_data=indices_data
                                )

                        else:
                            raise NotImplementedError(op)

                        ms_min = triton.testing.do_bench(test_func, warmup=500, rep=100)

                        return ms_min
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.sparse._triton_ops. This chunk defines `test_func`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.sparse._triton_ops。 这一段定义了 `test_func`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 934-964 / 第 934-964 行
````python
                    meta_lst.append(
                        (bench(meta), sparsity, tuple(meta[k] for k in sorted(meta)))
                    )
                if not meta_lst:
                    continue
                meta_lst = sorted(meta_lst)
                index = next(
                    i for i, item in enumerate(meta_lst) if item[1] == sparsity1
                )
                if meta_lst[0][2] == meta_lst[index][2]:
                    continue
                speeddiff = (1 - meta_lst[index][0] / meta_lst[0][0]) * 100
                if abs(speeddiff) < 10:
                    continue

                print(sparsity1, index, key, meta_lst, speeddiff)

                if index > 0:
                    device_name = _get_device_name()
                    meta = get_meta(
                        op, key, version=(0, dtype, meta_lst[0][1]), exact=True
                    )
                    update(
                        op,
                        device_name,
                        (0, dtype, sparsity1),
                        key,
                        tuple(meta[k] for k in sorted(meta)),
                    )
                    print("update")
                    dump()
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 967-1002 / 第 967-1002 行
````python
_operation_device_version_data: dict[Any, dict] = {
    # Warning: the data in between the BEGIN/END DATA comment lines
    # below is generated. It can be updated either manually or via
    # calling dump function defined above.
    #
    # Legend [op: key -> data]:
    #   scatter_mm : M, K, N, Ms, Ks -> GROUP_SIZE, SPLIT_N, TILE_M, TILE_N, num_stages, num_warps
    #   bsr_dense_addmm : M, K, N, Ms, Ks, beta==0, beta==1, alpha==1  -> GROUP_SIZE_ROW, SPLIT_N, num_stages, num_warps
    #
    # BEGIN GENERATED DATA
    ("_int_bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.int8, 0.5)): {
        (192, 192, 256, 32, 32, False, True, True): (2, 8, 1, 4),
        (192, 192, 256, 32, 32, True, False, True): (2, 8, 5, 4),
        (192, 192, 512, 32, 32, False, True, True): (1, 16, 1, 4),
        (192, 192, 512, 32, 32, True, False, True): (1, 16, 5, 4),
        (192, 192, 1024, 32, 32, False, True, True): (1, 32, 1, 4),
        (192, 192, 1024, 32, 32, True, False, True): (4, 32, 4, 4),
        (192, 192, 2048, 32, 32, False, True, True): (2, 64, 1, 4),
        (192, 192, 2048, 32, 32, True, False, True): (3, 16, 5, 4),
        (192, 192, 4096, 32, 32, False, True, True): (1, 128, 1, 4),
        (192, 192, 4096, 32, 32, True, False, True): (1, 128, 1, 4),
        (192, 192, 8192, 32, 32, False, True, True): (1, 256, 1, 4),
        (192, 192, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (192, 192, 16384, 32, 32, False, True, True): (2, 512, 1, 4),
        (192, 192, 16384, 32, 32, True, False, True): (5, 128, 1, 4),
        (192, 192, 32768, 32, 32, False, True, True): (1, 1024, 1, 4),
        (192, 192, 32768, 32, 32, True, False, True): (1, 256, 1, 4),
        (192, 192, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (192, 192, 65536, 32, 32, True, False, True): (1, 512, 1, 4),
        (192, 192, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (192, 192, 131072, 32, 32, True, False, True): (2, 512, 1, 4),
        (256, 256, 256, 32, 32, False, True, True): (4, 8, 1, 4),
        (256, 256, 256, 32, 32, True, False, True): (1, 8, 6, 4),
        (256, 256, 256, 64, 64, False, True, True): (1, 4, 1, 16),
        (256, 256, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (256, 256, 256, 128, 128, False, True, True): (3, 2, 1, 16),
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1003-1038 / 第 1003-1038 行
````python
        (256, 256, 256, 128, 128, True, False, True): (1, 2, 1, 4),
        (256, 256, 512, 32, 32, False, True, True): (2, 16, 1, 4),
        (256, 256, 512, 32, 32, True, False, True): (2, 16, 4, 4),
        (256, 256, 512, 64, 64, False, True, True): (7, 8, 1, 16),
        (256, 256, 512, 64, 64, True, False, True): (3, 8, 3, 4),
        (256, 256, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (256, 256, 512, 128, 128, True, False, True): (1, 4, 1, 4),
        (256, 256, 1024, 32, 32, False, True, True): (1, 32, 1, 4),
        (256, 256, 1024, 32, 32, True, False, True): (1, 8, 6, 4),
        (256, 256, 1024, 64, 64, False, True, True): (2, 16, 1, 16),
        (256, 256, 1024, 64, 64, True, False, True): (1, 16, 5, 4),
        (256, 256, 1024, 128, 128, False, True, True): (4, 8, 1, 32),
        (256, 256, 1024, 128, 128, True, False, True): (1, 8, 2, 4),
        (256, 256, 2048, 32, 32, False, True, True): (1, 64, 1, 4),
        (256, 256, 2048, 32, 32, True, False, True): (2, 32, 3, 2),
        (256, 256, 2048, 64, 64, False, True, True): (2, 32, 1, 16),
        (256, 256, 2048, 64, 64, True, False, True): (1, 16, 3, 4),
        (256, 256, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (256, 256, 2048, 128, 128, True, False, True): (1, 16, 2, 4),
        (256, 256, 4096, 32, 32, False, True, True): (2, 128, 1, 4),
        (256, 256, 4096, 32, 32, True, False, True): (1, 32, 3, 2),
        (256, 256, 4096, 64, 64, False, True, True): (2, 64, 1, 8),
        (256, 256, 4096, 64, 64, True, False, True): (1, 64, 3, 2),
        (256, 256, 4096, 128, 128, False, True, True): (2, 32, 1, 32),
        (256, 256, 4096, 128, 128, True, False, True): (3, 32, 2, 8),
        (256, 256, 8192, 32, 32, False, True, True): (1, 256, 1, 4),
        (256, 256, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (256, 256, 8192, 64, 64, False, True, True): (1, 128, 1, 8),
        (256, 256, 8192, 64, 64, True, False, True): (2, 128, 1, 4),
        (256, 256, 8192, 128, 128, False, True, True): (4, 64, 1, 32),
        (256, 256, 8192, 128, 128, True, False, True): (3, 64, 1, 4),
        (256, 256, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (256, 256, 16384, 32, 32, True, False, True): (3, 128, 1, 4),
        (256, 256, 16384, 64, 64, False, True, True): (2, 256, 1, 8),
        (256, 256, 16384, 64, 64, True, False, True): (2, 256, 1, 4),
        (256, 256, 16384, 128, 128, False, True, True): (2, 128, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1039-1074 / 第 1039-1074 行
````python
        (256, 256, 16384, 128, 128, True, False, True): (4, 128, 2, 4),
        (256, 256, 32768, 32, 32, False, True, True): (2, 512, 1, 8),
        (256, 256, 32768, 32, 32, True, False, True): (1, 256, 1, 4),
        (256, 256, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (256, 256, 32768, 64, 64, True, False, True): (1, 512, 1, 4),
        (256, 256, 32768, 128, 128, False, True, True): (2, 256, 1, 32),
        (256, 256, 32768, 128, 128, True, False, True): (1, 256, 2, 4),
        (256, 256, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (256, 256, 65536, 32, 32, True, False, True): (1, 512, 1, 4),
        (256, 256, 65536, 64, 64, False, True, True): (1, 1024, 1, 8),
        (256, 256, 65536, 64, 64, True, False, True): (1, 512, 1, 4),
        (256, 256, 65536, 128, 128, False, True, True): (2, 512, 1, 16),
        (256, 256, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (256, 256, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (256, 256, 65792, 32, 32, True, False, True): (1, 514, 1, 4),
        (256, 256, 65792, 64, 64, False, True, True): (1, 1028, 1, 8),
        (256, 256, 65792, 64, 64, True, False, True): (4, 257, 1, 4),
        (256, 256, 65792, 128, 128, False, True, True): (2, 514, 1, 16),
        (256, 256, 65792, 128, 128, True, False, True): (3, 514, 1, 4),
        (256, 256, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (256, 256, 131072, 32, 32, True, False, True): (2, 1024, 1, 4),
        (256, 256, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (256, 256, 131072, 64, 64, True, False, True): (2, 512, 1, 4),
        (256, 256, 131072, 128, 128, False, True, True): (2, 1024, 1, 16),
        (256, 256, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (384, 384, 256, 32, 32, False, True, True): (1, 8, 1, 4),
        (384, 384, 256, 32, 32, True, False, True): (5, 8, 5, 4),
        (384, 384, 256, 64, 64, False, True, True): (2, 4, 1, 16),
        (384, 384, 256, 64, 64, True, False, True): (1, 4, 5, 4),
        (384, 384, 512, 32, 32, False, True, True): (2, 16, 1, 4),
        (384, 384, 512, 32, 32, True, False, True): (1, 16, 4, 4),
        (384, 384, 512, 64, 64, False, True, True): (3, 8, 1, 16),
        (384, 384, 512, 64, 64, True, False, True): (3, 8, 3, 4),
        (384, 384, 1024, 32, 32, False, True, True): (2, 32, 1, 4),
        (384, 384, 1024, 32, 32, True, False, True): (1, 8, 6, 4),
        (384, 384, 1024, 64, 64, False, True, True): (2, 16, 1, 16),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1075-1110 / 第 1075-1110 行
````python
        (384, 384, 1024, 64, 64, True, False, True): (1, 16, 5, 4),
        (384, 384, 2048, 32, 32, False, True, True): (1, 64, 1, 4),
        (384, 384, 2048, 32, 32, True, False, True): (3, 16, 4, 4),
        (384, 384, 2048, 64, 64, False, True, True): (2, 32, 1, 16),
        (384, 384, 2048, 64, 64, True, False, True): (1, 16, 4, 4),
        (384, 384, 4096, 32, 32, False, True, True): (4, 64, 1, 8),
        (384, 384, 4096, 32, 32, True, False, True): (4, 32, 1, 4),
        (384, 384, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
        (384, 384, 4096, 64, 64, True, False, True): (1, 64, 1, 4),
        (384, 384, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (384, 384, 8192, 32, 32, True, False, True): (3, 64, 1, 1),
        (384, 384, 8192, 64, 64, False, True, True): (2, 128, 1, 8),
        (384, 384, 8192, 64, 64, True, False, True): (1, 64, 2, 2),
        (384, 384, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (384, 384, 16384, 32, 32, True, False, True): (1, 128, 1, 4),
        (384, 384, 16384, 64, 64, False, True, True): (2, 256, 1, 8),
        (384, 384, 16384, 64, 64, True, False, True): (2, 128, 1, 4),
        (384, 384, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (384, 384, 32768, 32, 32, True, False, True): (1, 256, 1, 4),
        (384, 384, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (384, 384, 32768, 64, 64, True, False, True): (1, 256, 3, 2),
        (384, 384, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (384, 384, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (384, 384, 65536, 64, 64, False, True, True): (2, 1024, 1, 8),
        (384, 384, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (384, 384, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (384, 384, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (384, 384, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (384, 384, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (512, 512, 256, 32, 32, False, True, True): (1, 8, 1, 4),
        (512, 512, 256, 32, 32, True, False, True): (4, 8, 4, 4),
        (512, 512, 256, 64, 64, False, True, True): (3, 4, 1, 16),
        (512, 512, 256, 64, 64, True, False, True): (2, 4, 5, 4),
        (512, 512, 256, 128, 128, False, True, True): (4, 2, 1, 16),
        (512, 512, 256, 128, 128, True, False, True): (1, 2, 3, 4),
        (512, 512, 256, 256, 256, False, True, True): (1, 1, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1111-1146 / 第 1111-1146 行
````python
        (512, 512, 256, 256, 256, True, False, True): (2, 1, 1, 32),
        (512, 512, 512, 32, 32, False, True, True): (3, 16, 1, 4),
        (512, 512, 512, 32, 32, True, False, True): (1, 8, 4, 2),
        (512, 512, 512, 64, 64, False, True, True): (2, 8, 1, 16),
        (512, 512, 512, 64, 64, True, False, True): (2, 8, 5, 4),
        (512, 512, 512, 128, 128, False, True, True): (3, 4, 1, 16),
        (512, 512, 512, 128, 128, True, False, True): (1, 4, 3, 4),
        (512, 512, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (512, 512, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (512, 512, 1024, 32, 32, False, True, True): (2, 32, 1, 4),
        (512, 512, 1024, 32, 32, True, False, True): (4, 16, 3, 2),
        (512, 512, 1024, 64, 64, False, True, True): (4, 16, 1, 16),
        (512, 512, 1024, 64, 64, True, False, True): (1, 8, 4, 4),
        (512, 512, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (512, 512, 1024, 128, 128, True, False, True): (1, 8, 3, 4),
        (512, 512, 1024, 256, 256, False, True, True): (4, 4, 1, 32),
        (512, 512, 1024, 256, 256, True, False, True): (2, 4, 1, 32),
        (512, 512, 2048, 32, 32, False, True, True): (3, 32, 1, 8),
        (512, 512, 2048, 32, 32, True, False, True): (1, 16, 3, 4),
        (512, 512, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
        (512, 512, 2048, 64, 64, True, False, True): (1, 32, 3, 2),
        (512, 512, 2048, 128, 128, False, True, True): (4, 16, 1, 32),
        (512, 512, 2048, 128, 128, True, False, True): (1, 16, 3, 4),
        (512, 512, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (512, 512, 2048, 256, 256, True, False, True): (3, 8, 1, 32),
        (512, 512, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (512, 512, 4096, 32, 32, True, False, True): (5, 32, 1, 4),
        (512, 512, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
        (512, 512, 4096, 64, 64, True, False, True): (1, 64, 1, 4),
        (512, 512, 4096, 128, 128, False, True, True): (5, 32, 1, 32),
        (512, 512, 4096, 128, 128, True, False, True): (2, 32, 3, 4),
        (512, 512, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (512, 512, 4096, 256, 256, True, False, True): (3, 16, 1, 32),
        (512, 512, 8192, 32, 32, False, True, True): (3, 128, 1, 8),
        (512, 512, 8192, 32, 32, True, False, True): (3, 64, 1, 4),
        (512, 512, 8192, 64, 64, False, True, True): (4, 128, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1147-1182 / 第 1147-1182 行
````python
        (512, 512, 8192, 64, 64, True, False, True): (1, 64, 3, 2),
        (512, 512, 8192, 128, 128, False, True, True): (5, 64, 1, 32),
        (512, 512, 8192, 128, 128, True, False, True): (1, 64, 2, 4),
        (512, 512, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (512, 512, 8192, 256, 256, True, False, True): (1, 32, 1, 32),
        (512, 512, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (512, 512, 16384, 32, 32, True, False, True): (2, 128, 1, 4),
        (512, 512, 16384, 64, 64, False, True, True): (2, 256, 1, 8),
        (512, 512, 16384, 64, 64, True, False, True): (1, 128, 3, 2),
        (512, 512, 16384, 128, 128, False, True, True): (4, 128, 1, 16),
        (512, 512, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (512, 512, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (512, 512, 16384, 256, 256, True, False, True): (2, 64, 1, 32),
        (512, 512, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (512, 512, 32768, 32, 32, True, False, True): (2, 256, 1, 4),
        (512, 512, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (512, 512, 32768, 64, 64, True, False, True): (1, 256, 3, 2),
        (512, 512, 32768, 128, 128, False, True, True): (4, 256, 1, 16),
        (512, 512, 32768, 128, 128, True, False, True): (2, 256, 1, 4),
        (512, 512, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (512, 512, 32768, 256, 256, True, False, True): (2, 128, 1, 32),
        (512, 512, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (512, 512, 65536, 32, 32, True, False, True): (2, 512, 1, 2),
        (512, 512, 65536, 64, 64, False, True, True): (1, 1024, 1, 8),
        (512, 512, 65536, 64, 64, True, False, True): (1, 512, 3, 2),
        (512, 512, 65536, 128, 128, False, True, True): (4, 512, 1, 16),
        (512, 512, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (512, 512, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (512, 512, 65536, 256, 256, True, False, True): (1, 256, 1, 32),
        (512, 512, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (512, 512, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (512, 512, 65792, 64, 64, False, True, True): (1, 1028, 1, 8),
        (512, 512, 65792, 64, 64, True, False, True): (2, 257, 3, 4),
        (512, 512, 65792, 128, 128, False, True, True): (4, 514, 1, 16),
        (512, 512, 65792, 128, 128, True, False, True): (1, 514, 1, 4),
        (512, 512, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1183-1218 / 第 1183-1218 行
````python
        (512, 512, 65792, 256, 256, True, False, True): (2, 257, 1, 32),
        (512, 512, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (512, 512, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (512, 512, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (512, 512, 131072, 64, 64, True, False, True): (1, 1024, 3, 2),
        (512, 512, 131072, 128, 128, False, True, True): (4, 1024, 1, 16),
        (512, 512, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (512, 512, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (512, 512, 131072, 256, 256, True, False, True): (2, 512, 1, 32),
        (768, 768, 256, 32, 32, False, True, True): (1, 8, 1, 4),
        (768, 768, 256, 32, 32, True, False, True): (2, 8, 4, 4),
        (768, 768, 256, 64, 64, False, True, True): (3, 4, 1, 16),
        (768, 768, 256, 64, 64, True, False, True): (2, 4, 4, 4),
        (768, 768, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (768, 768, 256, 128, 128, True, False, True): (1, 2, 3, 4),
        (768, 768, 512, 32, 32, False, True, True): (1, 16, 1, 4),
        (768, 768, 512, 32, 32, True, False, True): (1, 4, 5, 4),
        (768, 768, 512, 64, 64, False, True, True): (1, 8, 3, 32),
        (768, 768, 512, 64, 64, True, False, True): (4, 8, 4, 4),
        (768, 768, 512, 128, 128, False, True, True): (4, 4, 1, 16),
        (768, 768, 512, 128, 128, True, False, True): (4, 4, 3, 4),
        (768, 768, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
        (768, 768, 1024, 32, 32, True, False, True): (1, 8, 3, 4),
        (768, 768, 1024, 64, 64, False, True, True): (3, 16, 1, 16),
        (768, 768, 1024, 64, 64, True, False, True): (1, 8, 4, 4),
        (768, 768, 1024, 128, 128, False, True, True): (3, 8, 1, 32),
        (768, 768, 1024, 128, 128, True, False, True): (1, 8, 3, 4),
        (768, 768, 2048, 32, 32, False, True, True): (2, 32, 1, 8),
        (768, 768, 2048, 32, 32, True, False, True): (3, 16, 1, 4),
        (768, 768, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
        (768, 768, 2048, 64, 64, True, False, True): (4, 8, 3, 4),
        (768, 768, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (768, 768, 2048, 128, 128, True, False, True): (1, 16, 3, 4),
        (768, 768, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (768, 768, 4096, 32, 32, True, False, True): (1, 32, 1, 1),
        (768, 768, 4096, 64, 64, False, True, True): (2, 64, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1219-1254 / 第 1219-1254 行
````python
        (768, 768, 4096, 64, 64, True, False, True): (1, 32, 2, 2),
        (768, 768, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (768, 768, 4096, 128, 128, True, False, True): (6, 32, 1, 4),
        (768, 768, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (768, 768, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (768, 768, 8192, 64, 64, False, True, True): (1, 128, 1, 8),
        (768, 768, 8192, 64, 64, True, False, True): (4, 32, 3, 4),
        (768, 768, 8192, 128, 128, False, True, True): (2, 64, 1, 16),
        (768, 768, 8192, 128, 128, True, False, True): (2, 64, 3, 4),
        (768, 768, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (768, 768, 16384, 32, 32, True, False, True): (1, 128, 1, 4),
        (768, 768, 16384, 64, 64, False, True, True): (1, 256, 1, 8),
        (768, 768, 16384, 64, 64, True, False, True): (1, 128, 3, 2),
        (768, 768, 16384, 128, 128, False, True, True): (2, 128, 1, 16),
        (768, 768, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (768, 768, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (768, 768, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (768, 768, 32768, 64, 64, False, True, True): (2, 512, 1, 8),
        (768, 768, 32768, 64, 64, True, False, True): (1, 256, 3, 2),
        (768, 768, 32768, 128, 128, False, True, True): (2, 256, 1, 16),
        (768, 768, 32768, 128, 128, True, False, True): (3, 256, 1, 4),
        (768, 768, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (768, 768, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (768, 768, 65536, 64, 64, False, True, True): (2, 512, 1, 4),
        (768, 768, 65536, 64, 64, True, False, True): (1, 512, 3, 2),
        (768, 768, 65536, 128, 128, False, True, True): (2, 512, 1, 16),
        (768, 768, 65536, 128, 128, True, False, True): (2, 512, 1, 4),
        (768, 768, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (768, 768, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (768, 768, 131072, 64, 64, False, True, True): (2, 1024, 1, 4),
        (768, 768, 131072, 64, 64, True, False, True): (2, 1024, 3, 2),
        (768, 768, 131072, 128, 128, False, True, True): (2, 1024, 1, 16),
        (768, 768, 131072, 128, 128, True, False, True): (2, 1024, 1, 4),
        (768, 3072, 256, 32, 32, False, True, True): (3, 8, 4, 8),
        (768, 3072, 256, 32, 32, True, False, True): (3, 8, 5, 4),
        (768, 3072, 256, 64, 64, False, True, True): (1, 4, 4, 16),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1255-1290 / 第 1255-1290 行
````python
        (768, 3072, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (768, 3072, 256, 128, 128, False, True, True): (2, 2, 1, 8),
        (768, 3072, 256, 128, 128, True, False, True): (2, 2, 4, 4),
        (768, 3072, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (768, 3072, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (768, 3072, 512, 32, 32, False, True, True): (1, 16, 1, 4),
        (768, 3072, 512, 32, 32, True, False, True): (2, 4, 4, 4),
        (768, 3072, 512, 64, 64, False, True, True): (3, 8, 4, 16),
        (768, 3072, 512, 64, 64, True, False, True): (1, 8, 4, 4),
        (768, 3072, 512, 128, 128, False, True, True): (2, 4, 1, 8),
        (768, 3072, 512, 128, 128, True, False, True): (4, 4, 3, 4),
        (768, 3072, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (768, 3072, 512, 256, 256, True, False, True): (1, 2, 1, 32),
        (768, 3072, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
        (768, 3072, 1024, 32, 32, True, False, True): (3, 8, 3, 4),
        (768, 3072, 1024, 64, 64, False, True, True): (2, 16, 1, 16),
        (768, 3072, 1024, 64, 64, True, False, True): (1, 8, 3, 4),
        (768, 3072, 1024, 128, 128, False, True, True): (1, 8, 1, 8),
        (768, 3072, 1024, 128, 128, True, False, True): (3, 8, 4, 4),
        (768, 3072, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (768, 3072, 1024, 256, 256, True, False, True): (4, 4, 1, 32),
        (768, 3072, 2048, 32, 32, False, True, True): (3, 32, 1, 8),
        (768, 3072, 2048, 32, 32, True, False, True): (4, 8, 3, 4),
        (768, 3072, 2048, 64, 64, False, True, True): (5, 16, 1, 16),
        (768, 3072, 2048, 64, 64, True, False, True): (6, 8, 3, 4),
        (768, 3072, 2048, 128, 128, False, True, True): (2, 16, 1, 16),
        (768, 3072, 2048, 128, 128, True, False, True): (1, 16, 4, 4),
        (768, 3072, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (768, 3072, 2048, 256, 256, True, False, True): (1, 8, 1, 32),
        (768, 3072, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (768, 3072, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
        (768, 3072, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
        (768, 3072, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (768, 3072, 4096, 128, 128, False, True, True): (1, 32, 1, 8),
        (768, 3072, 4096, 128, 128, True, False, True): (2, 32, 2, 4),
        (768, 3072, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1291-1326 / 第 1291-1326 行
````python
        (768, 3072, 4096, 256, 256, True, False, True): (1, 16, 1, 32),
        (768, 3072, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (768, 3072, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (768, 3072, 8192, 64, 64, False, True, True): (1, 128, 1, 8),
        (768, 3072, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (768, 3072, 8192, 128, 128, False, True, True): (2, 64, 1, 16),
        (768, 3072, 8192, 128, 128, True, False, True): (2, 64, 3, 4),
        (768, 3072, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (768, 3072, 8192, 256, 256, True, False, True): (1, 32, 1, 32),
        (768, 3072, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (768, 3072, 16384, 32, 32, True, False, True): (1, 128, 1, 4),
        (768, 3072, 16384, 64, 64, False, True, True): (1, 256, 1, 8),
        (768, 3072, 16384, 64, 64, True, False, True): (2, 64, 3, 4),
        (768, 3072, 16384, 128, 128, False, True, True): (2, 128, 1, 16),
        (768, 3072, 16384, 128, 128, True, False, True): (2, 128, 3, 4),
        (768, 3072, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (768, 3072, 16384, 256, 256, True, False, True): (1, 64, 1, 32),
        (768, 3072, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (768, 3072, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (768, 3072, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (768, 3072, 32768, 64, 64, True, False, True): (3, 128, 3, 4),
        (768, 3072, 32768, 128, 128, False, True, True): (2, 256, 1, 16),
        (768, 3072, 32768, 128, 128, True, False, True): (2, 256, 3, 4),
        (768, 3072, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (768, 3072, 32768, 256, 256, True, False, True): (1, 128, 1, 32),
        (768, 3072, 50432, 32, 32, False, True, True): (1, 788, 1, 8),
        (768, 3072, 50432, 32, 32, True, False, True): (1, 394, 3, 2),
        (768, 3072, 50432, 64, 64, False, True, True): (1, 788, 1, 8),
        (768, 3072, 50432, 64, 64, True, False, True): (2, 197, 3, 4),
        (768, 3072, 50432, 128, 128, False, True, True): (2, 394, 1, 16),
        (768, 3072, 50432, 128, 128, True, False, True): (2, 394, 3, 4),
        (768, 3072, 50432, 256, 256, False, True, True): (1, 197, 1, 32),
        (768, 3072, 50432, 256, 256, True, False, True): (1, 197, 1, 32),
        (768, 3072, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (768, 3072, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (768, 3072, 65536, 64, 64, False, True, True): (1, 1024, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1327-1362 / 第 1327-1362 行
````python
        (768, 3072, 65536, 64, 64, True, False, True): (2, 256, 3, 4),
        (768, 3072, 65536, 128, 128, False, True, True): (2, 512, 1, 16),
        (768, 3072, 65536, 128, 128, True, False, True): (2, 512, 3, 4),
        (768, 3072, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (768, 3072, 65536, 256, 256, True, False, True): (1, 256, 1, 32),
        (768, 3072, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (768, 3072, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (768, 3072, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (768, 3072, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (768, 3072, 131072, 128, 128, False, True, True): (2, 1024, 1, 16),
        (768, 3072, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (768, 3072, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (768, 3072, 131072, 256, 256, True, False, True): (1, 512, 1, 32),
        (1024, 1024, 256, 32, 32, False, True, True): (1, 8, 1, 4),
        (1024, 1024, 256, 32, 32, True, False, True): (1, 8, 5, 4),
        (1024, 1024, 256, 64, 64, False, True, True): (1, 4, 1, 16),
        (1024, 1024, 256, 64, 64, True, False, True): (4, 4, 4, 4),
        (1024, 1024, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (1024, 1024, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (1024, 1024, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (1024, 1024, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (1024, 1024, 512, 32, 32, False, True, True): (5, 16, 1, 4),
        (1024, 1024, 512, 32, 32, True, False, True): (2, 8, 4, 2),
        (1024, 1024, 512, 64, 64, False, True, True): (4, 8, 1, 16),
        (1024, 1024, 512, 64, 64, True, False, True): (1, 4, 3, 4),
        (1024, 1024, 512, 128, 128, False, True, True): (3, 4, 1, 16),
        (1024, 1024, 512, 128, 128, True, False, True): (1, 4, 2, 4),
        (1024, 1024, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (1024, 1024, 512, 256, 256, True, False, True): (1, 2, 1, 32),
        (1024, 1024, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
        (1024, 1024, 1024, 32, 32, True, False, True): (1, 8, 3, 4),
        (1024, 1024, 1024, 64, 64, False, True, True): (3, 16, 1, 8),
        (1024, 1024, 1024, 64, 64, True, False, True): (1, 16, 3, 2),
        (1024, 1024, 1024, 128, 128, False, True, True): (1, 8, 1, 16),
        (1024, 1024, 1024, 128, 128, True, False, True): (2, 8, 3, 8),
        (1024, 1024, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1363-1398 / 第 1363-1398 行
````python
        (1024, 1024, 1024, 256, 256, True, False, True): (2, 4, 1, 32),
        (1024, 1024, 2048, 32, 32, False, True, True): (2, 32, 1, 8),
        (1024, 1024, 2048, 32, 32, True, False, True): (3, 16, 1, 4),
        (1024, 1024, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
        (1024, 1024, 2048, 64, 64, True, False, True): (3, 32, 1, 4),
        (1024, 1024, 2048, 128, 128, False, True, True): (4, 16, 1, 16),
        (1024, 1024, 2048, 128, 128, True, False, True): (1, 16, 3, 4),
        (1024, 1024, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (1024, 1024, 2048, 256, 256, True, False, True): (1, 8, 1, 32),
        (1024, 1024, 4096, 32, 32, False, True, True): (4, 64, 1, 8),
        (1024, 1024, 4096, 32, 32, True, False, True): (3, 32, 1, 4),
        (1024, 1024, 4096, 64, 64, False, True, True): (3, 64, 1, 8),
        (1024, 1024, 4096, 64, 64, True, False, True): (1, 32, 3, 2),
        (1024, 1024, 4096, 128, 128, False, True, True): (4, 32, 1, 16),
        (1024, 1024, 4096, 128, 128, True, False, True): (2, 32, 2, 4),
        (1024, 1024, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (1024, 1024, 4096, 256, 256, True, False, True): (7, 16, 1, 32),
        (1024, 1024, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (1024, 1024, 8192, 32, 32, True, False, True): (4, 64, 1, 4),
        (1024, 1024, 8192, 64, 64, False, True, True): (2, 128, 1, 8),
        (1024, 1024, 8192, 64, 64, True, False, True): (3, 32, 3, 4),
        (1024, 1024, 8192, 128, 128, False, True, True): (4, 64, 1, 16),
        (1024, 1024, 8192, 128, 128, True, False, True): (2, 64, 2, 4),
        (1024, 1024, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (1024, 1024, 8192, 256, 256, True, False, True): (1, 32, 1, 32),
        (1024, 1024, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (1024, 1024, 16384, 32, 32, True, False, True): (1, 128, 1, 4),
        (1024, 1024, 16384, 64, 64, False, True, True): (1, 256, 1, 8),
        (1024, 1024, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (1024, 1024, 16384, 128, 128, False, True, True): (4, 128, 1, 16),
        (1024, 1024, 16384, 128, 128, True, False, True): (1, 128, 3, 4),
        (1024, 1024, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (1024, 1024, 16384, 256, 256, True, False, True): (1, 64, 1, 32),
        (1024, 1024, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (1024, 1024, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (1024, 1024, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1399-1434 / 第 1399-1434 行
````python
        (1024, 1024, 32768, 64, 64, True, False, True): (4, 128, 3, 4),
        (1024, 1024, 32768, 128, 128, False, True, True): (4, 256, 1, 16),
        (1024, 1024, 32768, 128, 128, True, False, True): (2, 256, 3, 4),
        (1024, 1024, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (1024, 1024, 32768, 256, 256, True, False, True): (2, 128, 1, 32),
        (1024, 1024, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (1024, 1024, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (1024, 1024, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (1024, 1024, 65536, 64, 64, True, False, True): (2, 256, 3, 4),
        (1024, 1024, 65536, 128, 128, False, True, True): (4, 512, 1, 16),
        (1024, 1024, 65536, 128, 128, True, False, True): (4, 512, 3, 4),
        (1024, 1024, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (1024, 1024, 65536, 256, 256, True, False, True): (1, 256, 1, 32),
        (1024, 1024, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (1024, 1024, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (1024, 1024, 65792, 64, 64, False, True, True): (2, 514, 1, 4),
        (1024, 1024, 65792, 64, 64, True, False, True): (4, 257, 3, 4),
        (1024, 1024, 65792, 128, 128, False, True, True): (2, 514, 1, 16),
        (1024, 1024, 65792, 128, 128, True, False, True): (2, 514, 2, 4),
        (1024, 1024, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (1024, 1024, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (1024, 1024, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (1024, 1024, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (1024, 1024, 131072, 64, 64, False, True, True): (2, 1024, 1, 4),
        (1024, 1024, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (1024, 1024, 131072, 128, 128, False, True, True): (4, 1024, 1, 16),
        (1024, 1024, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (1024, 1024, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (1024, 1024, 131072, 256, 256, True, False, True): (1, 512, 1, 32),
        (1280, 5120, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (1280, 5120, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (1280, 5120, 65792, 64, 64, False, True, True): (1, 1028, 1, 8),
        (1280, 5120, 65792, 64, 64, True, False, True): (2, 257, 3, 4),
        (1280, 5120, 65792, 128, 128, False, True, True): (2, 514, 1, 16),
        (1280, 5120, 65792, 128, 128, True, False, True): (1, 514, 3, 4),
        (1280, 5120, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1435-1470 / 第 1435-1470 行
````python
        (1280, 5120, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (1536, 1536, 256, 32, 32, False, True, True): (1, 8, 1, 4),
        (1536, 1536, 256, 32, 32, True, False, True): (2, 8, 1, 8),
        (1536, 1536, 256, 64, 64, False, True, True): (4, 4, 1, 16),
        (1536, 1536, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (1536, 1536, 256, 128, 128, False, True, True): (2, 2, 1, 16),
        (1536, 1536, 256, 128, 128, True, False, True): (2, 2, 3, 4),
        (1536, 1536, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (1536, 1536, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (1536, 1536, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (1536, 1536, 512, 32, 32, True, False, True): (3, 4, 4, 4),
        (1536, 1536, 512, 64, 64, False, True, True): (3, 8, 1, 16),
        (1536, 1536, 512, 64, 64, True, False, True): (1, 4, 3, 4),
        (1536, 1536, 512, 128, 128, False, True, True): (1, 4, 1, 16),
        (1536, 1536, 512, 128, 128, True, False, True): (2, 4, 4, 4),
        (1536, 1536, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (1536, 1536, 512, 256, 256, True, False, True): (1, 2, 1, 32),
        (1536, 1536, 1024, 32, 32, False, True, True): (4, 16, 1, 8),
        (1536, 1536, 1024, 32, 32, True, False, True): (2, 8, 1, 4),
        (1536, 1536, 1024, 64, 64, False, True, True): (2, 16, 1, 16),
        (1536, 1536, 1024, 64, 64, True, False, True): (2, 4, 3, 4),
        (1536, 1536, 1024, 128, 128, False, True, True): (3, 8, 1, 32),
        (1536, 1536, 1024, 128, 128, True, False, True): (4, 8, 3, 4),
        (1536, 1536, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (1536, 1536, 1024, 256, 256, True, False, True): (1, 4, 1, 32),
        (1536, 1536, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (1536, 1536, 2048, 32, 32, True, False, True): (1, 16, 1, 4),
        (1536, 1536, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
        (1536, 1536, 2048, 64, 64, True, False, True): (1, 16, 2, 2),
        (1536, 1536, 2048, 128, 128, False, True, True): (2, 16, 1, 16),
        (1536, 1536, 2048, 128, 128, True, False, True): (4, 16, 2, 4),
        (1536, 1536, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (1536, 1536, 2048, 256, 256, True, False, True): (1, 8, 1, 32),
        (1536, 1536, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (1536, 1536, 4096, 32, 32, True, False, True): (1, 32, 1, 4),
        (1536, 1536, 4096, 64, 64, False, True, True): (3, 64, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1471-1506 / 第 1471-1506 行
````python
        (1536, 1536, 4096, 64, 64, True, False, True): (1, 32, 3, 2),
        (1536, 1536, 4096, 128, 128, False, True, True): (1, 32, 1, 8),
        (1536, 1536, 4096, 128, 128, True, False, True): (2, 32, 2, 4),
        (1536, 1536, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (1536, 1536, 4096, 256, 256, True, False, True): (2, 16, 1, 32),
        (1536, 1536, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (1536, 1536, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (1536, 1536, 8192, 64, 64, False, True, True): (3, 128, 1, 8),
        (1536, 1536, 8192, 64, 64, True, False, True): (1, 64, 3, 2),
        (1536, 1536, 8192, 128, 128, False, True, True): (1, 64, 1, 8),
        (1536, 1536, 8192, 128, 128, True, False, True): (1, 64, 2, 4),
        (1536, 1536, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (1536, 1536, 8192, 256, 256, True, False, True): (2, 32, 1, 32),
        (1536, 1536, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (1536, 1536, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (1536, 1536, 16384, 64, 64, False, True, True): (2, 128, 1, 4),
        (1536, 1536, 16384, 64, 64, True, False, True): (2, 64, 3, 4),
        (1536, 1536, 16384, 128, 128, False, True, True): (1, 128, 1, 8),
        (1536, 1536, 16384, 128, 128, True, False, True): (2, 128, 2, 4),
        (1536, 1536, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (1536, 1536, 16384, 256, 256, True, False, True): (2, 64, 1, 32),
        (1536, 1536, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (1536, 1536, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (1536, 1536, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (1536, 1536, 32768, 64, 64, True, False, True): (3, 128, 3, 4),
        (1536, 1536, 32768, 128, 128, False, True, True): (1, 256, 1, 8),
        (1536, 1536, 32768, 128, 128, True, False, True): (1, 256, 2, 4),
        (1536, 1536, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (1536, 1536, 32768, 256, 256, True, False, True): (2, 128, 1, 32),
        (1536, 1536, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (1536, 1536, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (1536, 1536, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (1536, 1536, 65536, 64, 64, True, False, True): (1, 512, 3, 2),
        (1536, 1536, 65536, 128, 128, False, True, True): (1, 512, 1, 8),
        (1536, 1536, 65536, 128, 128, True, False, True): (1, 512, 3, 4),
        (1536, 1536, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1507-1542 / 第 1507-1542 行
````python
        (1536, 1536, 65536, 256, 256, True, False, True): (2, 256, 1, 32),
        (1536, 1536, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (1536, 1536, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (1536, 1536, 131072, 64, 64, False, True, True): (3, 1024, 1, 4),
        (1536, 1536, 131072, 64, 64, True, False, True): (3, 512, 3, 4),
        (1536, 1536, 131072, 128, 128, False, True, True): (1, 1024, 1, 8),
        (1536, 1536, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (1536, 1536, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (1536, 1536, 131072, 256, 256, True, False, True): (2, 512, 1, 32),
        (2048, 2048, 256, 32, 32, False, True, True): (3, 8, 1, 4),
        (2048, 2048, 256, 32, 32, True, False, True): (1, 4, 4, 2),
        (2048, 2048, 256, 64, 64, False, True, True): (2, 4, 1, 16),
        (2048, 2048, 256, 64, 64, True, False, True): (1, 2, 3, 4),
        (2048, 2048, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (2048, 2048, 256, 128, 128, True, False, True): (1, 2, 4, 4),
        (2048, 2048, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (2048, 2048, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (2048, 2048, 512, 32, 32, False, True, True): (3, 8, 1, 8),
        (2048, 2048, 512, 32, 32, True, False, True): (4, 4, 3, 2),
        (2048, 2048, 512, 64, 64, False, True, True): (1, 8, 1, 8),
        (2048, 2048, 512, 64, 64, True, False, True): (1, 8, 3, 4),
        (2048, 2048, 512, 128, 128, False, True, True): (1, 4, 1, 8),
        (2048, 2048, 512, 128, 128, True, False, True): (1, 4, 4, 4),
        (2048, 2048, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (2048, 2048, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (2048, 2048, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
        (2048, 2048, 1024, 32, 32, True, False, True): (3, 8, 1, 4),
        (2048, 2048, 1024, 64, 64, False, True, True): (4, 16, 1, 8),
        (2048, 2048, 1024, 64, 64, True, False, True): (1, 8, 3, 2),
        (2048, 2048, 1024, 128, 128, False, True, True): (4, 8, 1, 16),
        (2048, 2048, 1024, 128, 128, True, False, True): (2, 8, 2, 4),
        (2048, 2048, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (2048, 2048, 1024, 256, 256, True, False, True): (3, 4, 1, 32),
        (2048, 2048, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (2048, 2048, 2048, 32, 32, True, False, True): (1, 16, 1, 4),
        (2048, 2048, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1543-1578 / 第 1543-1578 行
````python
        (2048, 2048, 2048, 64, 64, True, False, True): (1, 16, 3, 2),
        (2048, 2048, 2048, 128, 128, False, True, True): (4, 16, 1, 16),
        (2048, 2048, 2048, 128, 128, True, False, True): (2, 16, 2, 4),
        (2048, 2048, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (2048, 2048, 2048, 256, 256, True, False, True): (1, 8, 1, 32),
        (2048, 2048, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (2048, 2048, 4096, 32, 32, True, False, True): (1, 32, 1, 4),
        (2048, 2048, 4096, 64, 64, False, True, True): (4, 64, 1, 8),
        (2048, 2048, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (2048, 2048, 4096, 128, 128, False, True, True): (4, 32, 1, 8),
        (2048, 2048, 4096, 128, 128, True, False, True): (1, 32, 2, 4),
        (2048, 2048, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (2048, 2048, 4096, 256, 256, True, False, True): (4, 16, 1, 32),
        (2048, 2048, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (2048, 2048, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (2048, 2048, 8192, 64, 64, False, True, True): (2, 64, 1, 4),
        (2048, 2048, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (2048, 2048, 8192, 128, 128, False, True, True): (4, 64, 1, 8),
        (2048, 2048, 8192, 128, 128, True, False, True): (2, 64, 2, 4),
        (2048, 2048, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (2048, 2048, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (2048, 2048, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (2048, 2048, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (2048, 2048, 16384, 64, 64, False, True, True): (2, 128, 1, 4),
        (2048, 2048, 16384, 64, 64, True, False, True): (2, 64, 3, 4),
        (2048, 2048, 16384, 128, 128, False, True, True): (1, 128, 1, 8),
        (2048, 2048, 16384, 128, 128, True, False, True): (2, 128, 2, 4),
        (2048, 2048, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (2048, 2048, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (2048, 2048, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (2048, 2048, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (2048, 2048, 32768, 64, 64, False, True, True): (2, 256, 1, 4),
        (2048, 2048, 32768, 64, 64, True, False, True): (2, 128, 3, 4),
        (2048, 2048, 32768, 128, 128, False, True, True): (1, 256, 1, 8),
        (2048, 2048, 32768, 128, 128, True, False, True): (2, 256, 2, 4),
        (2048, 2048, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1579-1614 / 第 1579-1614 行
````python
        (2048, 2048, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (2048, 2048, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (2048, 2048, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (2048, 2048, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (2048, 2048, 65536, 64, 64, True, False, True): (2, 256, 3, 4),
        (2048, 2048, 65536, 128, 128, False, True, True): (1, 512, 1, 8),
        (2048, 2048, 65536, 128, 128, True, False, True): (1, 512, 2, 4),
        (2048, 2048, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (2048, 2048, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (2048, 2048, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (2048, 2048, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (2048, 2048, 65792, 64, 64, False, True, True): (1, 514, 1, 4),
        (2048, 2048, 65792, 64, 64, True, False, True): (2, 257, 3, 4),
        (2048, 2048, 65792, 128, 128, False, True, True): (1, 514, 1, 8),
        (2048, 2048, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (2048, 2048, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (2048, 2048, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (2048, 2048, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (2048, 2048, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (2048, 2048, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (2048, 2048, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (2048, 2048, 131072, 128, 128, False, True, True): (1, 1024, 1, 8),
        (2048, 2048, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (2048, 2048, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (2048, 2048, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (3072, 768, 256, 32, 32, False, True, True): (5, 4, 1, 8),
        (3072, 768, 256, 32, 32, True, False, True): (2, 2, 4, 4),
        (3072, 768, 256, 64, 64, False, True, True): (1, 4, 1, 16),
        (3072, 768, 256, 64, 64, True, False, True): (2, 2, 3, 4),
        (3072, 768, 256, 128, 128, False, True, True): (5, 2, 1, 16),
        (3072, 768, 256, 128, 128, True, False, True): (1, 2, 5, 4),
        (3072, 768, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (3072, 768, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (3072, 768, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (3072, 768, 512, 32, 32, True, False, True): (5, 4, 1, 4),
        (3072, 768, 512, 64, 64, False, True, True): (1, 8, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1615-1650 / 第 1615-1650 行
````python
        (3072, 768, 512, 64, 64, True, False, True): (3, 2, 3, 4),
        (3072, 768, 512, 128, 128, False, True, True): (3, 4, 1, 32),
        (3072, 768, 512, 128, 128, True, False, True): (2, 4, 3, 4),
        (3072, 768, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (3072, 768, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (3072, 768, 1024, 32, 32, False, True, True): (2, 16, 1, 8),
        (3072, 768, 1024, 32, 32, True, False, True): (3, 8, 1, 4),
        (3072, 768, 1024, 64, 64, False, True, True): (4, 16, 1, 8),
        (3072, 768, 1024, 64, 64, True, False, True): (1, 8, 3, 2),
        (3072, 768, 1024, 128, 128, False, True, True): (2, 8, 1, 32),
        (3072, 768, 1024, 128, 128, True, False, True): (3, 8, 2, 4),
        (3072, 768, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (3072, 768, 1024, 256, 256, True, False, True): (4, 4, 1, 32),
        (3072, 768, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (3072, 768, 2048, 32, 32, True, False, True): (1, 16, 1, 4),
        (3072, 768, 2048, 64, 64, False, True, True): (2, 32, 1, 8),
        (3072, 768, 2048, 64, 64, True, False, True): (2, 8, 3, 4),
        (3072, 768, 2048, 128, 128, False, True, True): (2, 16, 1, 16),
        (3072, 768, 2048, 128, 128, True, False, True): (2, 16, 1, 4),
        (3072, 768, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (3072, 768, 2048, 256, 256, True, False, True): (2, 8, 1, 32),
        (3072, 768, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (3072, 768, 4096, 32, 32, True, False, True): (1, 32, 1, 2),
        (3072, 768, 4096, 64, 64, False, True, True): (2, 64, 1, 8),
        (3072, 768, 4096, 64, 64, True, False, True): (2, 32, 2, 2),
        (3072, 768, 4096, 128, 128, False, True, True): (1, 32, 1, 8),
        (3072, 768, 4096, 128, 128, True, False, True): (2, 32, 2, 4),
        (3072, 768, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (3072, 768, 4096, 256, 256, True, False, True): (4, 16, 1, 32),
        (3072, 768, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (3072, 768, 8192, 32, 32, True, False, True): (3, 64, 1, 2),
        (3072, 768, 8192, 64, 64, False, True, True): (1, 128, 1, 8),
        (3072, 768, 8192, 64, 64, True, False, True): (2, 64, 2, 2),
        (3072, 768, 8192, 128, 128, False, True, True): (1, 64, 1, 8),
        (3072, 768, 8192, 128, 128, True, False, True): (2, 64, 2, 4),
        (3072, 768, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1651-1686 / 第 1651-1686 行
````python
        (3072, 768, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (3072, 768, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (3072, 768, 16384, 32, 32, True, False, True): (1, 128, 1, 2),
        (3072, 768, 16384, 64, 64, False, True, True): (2, 128, 1, 4),
        (3072, 768, 16384, 64, 64, True, False, True): (1, 128, 2, 2),
        (3072, 768, 16384, 128, 128, False, True, True): (1, 128, 1, 8),
        (3072, 768, 16384, 128, 128, True, False, True): (1, 128, 1, 4),
        (3072, 768, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (3072, 768, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (3072, 768, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (3072, 768, 32768, 32, 32, True, False, True): (1, 256, 1, 2),
        (3072, 768, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (3072, 768, 32768, 64, 64, True, False, True): (2, 256, 2, 2),
        (3072, 768, 32768, 128, 128, False, True, True): (1, 256, 1, 8),
        (3072, 768, 32768, 128, 128, True, False, True): (2, 256, 1, 4),
        (3072, 768, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (3072, 768, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (3072, 768, 50432, 32, 32, False, True, True): (1, 788, 1, 8),
        (3072, 768, 50432, 32, 32, True, False, True): (1, 394, 1, 2),
        (3072, 768, 50432, 64, 64, False, True, True): (2, 394, 1, 4),
        (3072, 768, 50432, 64, 64, True, False, True): (2, 394, 2, 2),
        (3072, 768, 50432, 128, 128, False, True, True): (1, 394, 1, 8),
        (3072, 768, 50432, 128, 128, True, False, True): (2, 394, 1, 4),
        (3072, 768, 50432, 256, 256, False, True, True): (1, 197, 1, 32),
        (3072, 768, 50432, 256, 256, True, False, True): (1, 197, 1, 32),
        (3072, 768, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (3072, 768, 65536, 32, 32, True, False, True): (1, 512, 1, 2),
        (3072, 768, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (3072, 768, 65536, 64, 64, True, False, True): (2, 512, 2, 2),
        (3072, 768, 65536, 128, 128, False, True, True): (1, 512, 1, 8),
        (3072, 768, 65536, 128, 128, True, False, True): (2, 512, 1, 4),
        (3072, 768, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (3072, 768, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (3072, 768, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (3072, 768, 131072, 32, 32, True, False, True): (1, 1024, 1, 2),
        (3072, 768, 131072, 64, 64, False, True, True): (2, 1024, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1687-1722 / 第 1687-1722 行
````python
        (3072, 768, 131072, 64, 64, True, False, True): (2, 1024, 2, 2),
        (3072, 768, 131072, 128, 128, False, True, True): (1, 1024, 1, 8),
        (3072, 768, 131072, 128, 128, True, False, True): (2, 1024, 1, 4),
        (3072, 768, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (3072, 768, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (3072, 3072, 256, 32, 32, False, True, True): (1, 4, 1, 8),
        (3072, 3072, 256, 32, 32, True, False, True): (2, 2, 5, 4),
        (3072, 3072, 256, 64, 64, False, True, True): (2, 4, 1, 16),
        (3072, 3072, 256, 64, 64, True, False, True): (3, 2, 3, 4),
        (3072, 3072, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (3072, 3072, 256, 128, 128, True, False, True): (1, 2, 5, 4),
        (3072, 3072, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (3072, 3072, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (3072, 3072, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (3072, 3072, 512, 32, 32, True, False, True): (3, 2, 3, 4),
        (3072, 3072, 512, 64, 64, False, True, True): (1, 8, 1, 8),
        (3072, 3072, 512, 64, 64, True, False, True): (3, 2, 3, 4),
        (3072, 3072, 512, 128, 128, False, True, True): (2, 4, 1, 8),
        (3072, 3072, 512, 128, 128, True, False, True): (2, 4, 4, 4),
        (3072, 3072, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (3072, 3072, 512, 256, 256, True, False, True): (1, 2, 1, 32),
        (3072, 3072, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
        (3072, 3072, 1024, 32, 32, True, False, True): (3, 8, 3, 4),
        (3072, 3072, 1024, 64, 64, False, True, True): (2, 16, 1, 8),
        (3072, 3072, 1024, 64, 64, True, False, True): (2, 4, 3, 4),
        (3072, 3072, 1024, 128, 128, False, True, True): (1, 8, 1, 8),
        (3072, 3072, 1024, 128, 128, True, False, True): (3, 8, 2, 4),
        (3072, 3072, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (3072, 3072, 1024, 256, 256, True, False, True): (3, 4, 1, 32),
        (3072, 3072, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (3072, 3072, 2048, 32, 32, True, False, True): (1, 16, 1, 4),
        (3072, 3072, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
        (3072, 3072, 2048, 64, 64, True, False, True): (1, 16, 3, 2),
        (3072, 3072, 2048, 128, 128, False, True, True): (1, 16, 1, 8),
        (3072, 3072, 2048, 128, 128, True, False, True): (2, 16, 2, 4),
        (3072, 3072, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1723-1758 / 第 1723-1758 行
````python
        (3072, 3072, 2048, 256, 256, True, False, True): (3, 8, 1, 32),
        (3072, 3072, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (3072, 3072, 4096, 32, 32, True, False, True): (1, 32, 1, 4),
        (3072, 3072, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
        (3072, 3072, 4096, 64, 64, True, False, True): (3, 16, 3, 4),
        (3072, 3072, 4096, 128, 128, False, True, True): (1, 32, 1, 8),
        (3072, 3072, 4096, 128, 128, True, False, True): (2, 32, 2, 4),
        (3072, 3072, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (3072, 3072, 4096, 256, 256, True, False, True): (2, 16, 1, 32),
        (3072, 3072, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (3072, 3072, 8192, 32, 32, True, False, True): (1, 64, 1, 2),
        (3072, 3072, 8192, 64, 64, False, True, True): (1, 64, 1, 4),
        (3072, 3072, 8192, 64, 64, True, False, True): (1, 64, 3, 2),
        (3072, 3072, 8192, 128, 128, False, True, True): (1, 64, 1, 8),
        (3072, 3072, 8192, 128, 128, True, False, True): (2, 64, 2, 4),
        (3072, 3072, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (3072, 3072, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (3072, 3072, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (3072, 3072, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (3072, 3072, 16384, 64, 64, False, True, True): (1, 128, 1, 4),
        (3072, 3072, 16384, 64, 64, True, False, True): (2, 64, 3, 4),
        (3072, 3072, 16384, 128, 128, False, True, True): (1, 128, 1, 8),
        (3072, 3072, 16384, 128, 128, True, False, True): (1, 128, 2, 4),
        (3072, 3072, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (3072, 3072, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (3072, 3072, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (3072, 3072, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (3072, 3072, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (3072, 3072, 32768, 64, 64, True, False, True): (1, 256, 3, 2),
        (3072, 3072, 32768, 128, 128, False, True, True): (1, 256, 1, 8),
        (3072, 3072, 32768, 128, 128, True, False, True): (1, 256, 2, 4),
        (3072, 3072, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (3072, 3072, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (3072, 3072, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (3072, 3072, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (3072, 3072, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1759-1794 / 第 1759-1794 行
````python
        (3072, 3072, 65536, 64, 64, True, False, True): (2, 256, 3, 4),
        (3072, 3072, 65536, 128, 128, False, True, True): (1, 512, 1, 8),
        (3072, 3072, 65536, 128, 128, True, False, True): (1, 512, 3, 4),
        (3072, 3072, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (3072, 3072, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (3072, 3072, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (3072, 3072, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (3072, 3072, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (3072, 3072, 131072, 64, 64, True, False, True): (1, 1024, 3, 2),
        (3072, 3072, 131072, 128, 128, False, True, True): (1, 1024, 1, 8),
        (3072, 3072, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (3072, 3072, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (3072, 3072, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (4096, 4096, 256, 32, 32, False, True, True): (1, 4, 1, 8),
        (4096, 4096, 256, 32, 32, True, False, True): (5, 2, 3, 4),
        (4096, 4096, 256, 64, 64, False, True, True): (3, 4, 1, 8),
        (4096, 4096, 256, 64, 64, True, False, True): (3, 4, 3, 2),
        (4096, 4096, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (4096, 4096, 256, 128, 128, True, False, True): (2, 2, 4, 4),
        (4096, 4096, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (4096, 4096, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (4096, 4096, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (4096, 4096, 512, 32, 32, True, False, True): (1, 4, 1, 4),
        (4096, 4096, 512, 64, 64, False, True, True): (1, 8, 1, 8),
        (4096, 4096, 512, 64, 64, True, False, True): (3, 4, 2, 2),
        (4096, 4096, 512, 128, 128, False, True, True): (2, 4, 1, 8),
        (4096, 4096, 512, 128, 128, True, False, True): (2, 4, 2, 4),
        (4096, 4096, 512, 256, 256, False, True, True): (2, 2, 1, 32),
        (4096, 4096, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (4096, 4096, 1024, 32, 32, False, True, True): (4, 16, 1, 8),
        (4096, 4096, 1024, 32, 32, True, False, True): (1, 8, 1, 4),
        (4096, 4096, 1024, 64, 64, False, True, True): (1, 16, 1, 8),
        (4096, 4096, 1024, 64, 64, True, False, True): (4, 4, 3, 4),
        (4096, 4096, 1024, 128, 128, False, True, True): (2, 8, 1, 8),
        (4096, 4096, 1024, 128, 128, True, False, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1795-1830 / 第 1795-1830 行
````python
        (4096, 4096, 1024, 256, 256, True, False, True): (6, 4, 1, 32),
        (4096, 4096, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (4096, 4096, 2048, 32, 32, True, False, True): (1, 16, 1, 4),
        (4096, 4096, 2048, 64, 64, False, True, True): (4, 32, 1, 8),
        (4096, 4096, 2048, 64, 64, True, False, True): (4, 8, 3, 4),
        (4096, 4096, 2048, 128, 128, False, True, True): (2, 16, 1, 8),
        (4096, 4096, 2048, 128, 128, True, False, True): (1, 16, 3, 4),
        (4096, 4096, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (4096, 4096, 2048, 256, 256, True, False, True): (4, 8, 1, 32),
        (4096, 4096, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (4096, 4096, 4096, 32, 32, True, False, True): (1, 32, 1, 4),
        (4096, 4096, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
        (4096, 4096, 4096, 64, 64, True, False, True): (1, 32, 3, 2),
        (4096, 4096, 4096, 128, 128, False, True, True): (1, 32, 1, 8),
        (4096, 4096, 4096, 128, 128, True, False, True): (2, 32, 3, 4),
        (4096, 4096, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (4096, 4096, 4096, 256, 256, True, False, True): (4, 16, 1, 32),
        (4096, 4096, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (4096, 4096, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (4096, 4096, 8192, 64, 64, False, True, True): (1, 128, 1, 8),
        (4096, 4096, 8192, 64, 64, True, False, True): (1, 64, 3, 2),
        (4096, 4096, 8192, 128, 128, False, True, True): (1, 64, 1, 8),
        (4096, 4096, 8192, 128, 128, True, False, True): (1, 64, 3, 4),
        (4096, 4096, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (4096, 4096, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (4096, 4096, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (4096, 4096, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (4096, 4096, 16384, 64, 64, False, True, True): (1, 128, 1, 4),
        (4096, 4096, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (4096, 4096, 16384, 128, 128, False, True, True): (1, 128, 1, 8),
        (4096, 4096, 16384, 128, 128, True, False, True): (1, 128, 3, 4),
        (4096, 4096, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (4096, 4096, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (4096, 4096, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (4096, 4096, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (4096, 4096, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1831-1866 / 第 1831-1866 行
````python
        (4096, 4096, 32768, 64, 64, True, False, True): (1, 256, 3, 2),
        (4096, 4096, 32768, 128, 128, False, True, True): (1, 256, 1, 8),
        (4096, 4096, 32768, 128, 128, True, False, True): (1, 256, 3, 4),
        (4096, 4096, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (4096, 4096, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (4096, 4096, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (4096, 4096, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (4096, 4096, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (4096, 4096, 65536, 64, 64, True, False, True): (4, 256, 3, 4),
        (4096, 4096, 65536, 128, 128, False, True, True): (1, 512, 1, 8),
        (4096, 4096, 65536, 128, 128, True, False, True): (1, 512, 3, 4),
        (4096, 4096, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (4096, 4096, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (4096, 4096, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (4096, 4096, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (4096, 4096, 65792, 64, 64, False, True, True): (1, 1028, 1, 8),
        (4096, 4096, 65792, 64, 64, True, False, True): (1, 514, 3, 2),
        (4096, 4096, 65792, 128, 128, False, True, True): (1, 514, 1, 8),
        (4096, 4096, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (4096, 4096, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (4096, 4096, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (4096, 4096, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (4096, 4096, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (4096, 4096, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (4096, 4096, 131072, 64, 64, True, False, True): (1, 1024, 3, 2),
        (4096, 4096, 131072, 128, 128, False, True, True): (1, 1024, 1, 8),
        (4096, 4096, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (4096, 4096, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (4096, 4096, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (5120, 1280, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (5120, 1280, 65792, 32, 32, True, False, True): (1, 514, 1, 2),
        (5120, 1280, 65792, 64, 64, False, True, True): (1, 514, 1, 4),
        (5120, 1280, 65792, 64, 64, True, False, True): (1, 514, 2, 2),
        (5120, 1280, 65792, 128, 128, False, True, True): (1, 514, 1, 8),
        (5120, 1280, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (5120, 1280, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1867-1902 / 第 1867-1902 行
````python
        (5120, 1280, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (6144, 6144, 256, 32, 32, False, True, True): (2, 4, 1, 8),
        (6144, 6144, 256, 32, 32, True, False, True): (2, 1, 4, 4),
        (6144, 6144, 256, 64, 64, False, True, True): (1, 4, 1, 8),
        (6144, 6144, 256, 64, 64, True, False, True): (5, 1, 3, 4),
        (6144, 6144, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (6144, 6144, 256, 128, 128, True, False, True): (1, 2, 3, 4),
        (6144, 6144, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (6144, 6144, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (6144, 6144, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (6144, 6144, 512, 32, 32, True, False, True): (1, 4, 4, 2),
        (6144, 6144, 512, 64, 64, False, True, True): (2, 8, 1, 8),
        (6144, 6144, 512, 64, 64, True, False, True): (2, 2, 3, 4),
        (6144, 6144, 512, 128, 128, False, True, True): (3, 4, 1, 8),
        (6144, 6144, 512, 128, 128, True, False, True): (2, 4, 3, 4),
        (6144, 6144, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (6144, 6144, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (6144, 6144, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
        (6144, 6144, 1024, 32, 32, True, False, True): (1, 8, 1, 4),
        (6144, 6144, 1024, 64, 64, False, True, True): (1, 16, 1, 8),
        (6144, 6144, 1024, 64, 64, True, False, True): (4, 4, 3, 4),
        (6144, 6144, 1024, 128, 128, False, True, True): (1, 8, 1, 8),
        (6144, 6144, 1024, 128, 128, True, False, True): (3, 8, 3, 4),
        (6144, 6144, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (6144, 6144, 1024, 256, 256, True, False, True): (1, 4, 1, 32),
        (6144, 6144, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (6144, 6144, 2048, 32, 32, True, False, True): (1, 16, 1, 4),
        (6144, 6144, 2048, 64, 64, False, True, True): (1, 32, 1, 8),
        (6144, 6144, 2048, 64, 64, True, False, True): (4, 8, 3, 4),
        (6144, 6144, 2048, 128, 128, False, True, True): (1, 16, 1, 8),
        (6144, 6144, 2048, 128, 128, True, False, True): (3, 16, 3, 4),
        (6144, 6144, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (6144, 6144, 2048, 256, 256, True, False, True): (4, 8, 1, 32),
        (6144, 6144, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (6144, 6144, 4096, 32, 32, True, False, True): (1, 32, 1, 4),
        (6144, 6144, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1903-1938 / 第 1903-1938 行
````python
        (6144, 6144, 4096, 64, 64, True, False, True): (4, 16, 3, 4),
        (6144, 6144, 4096, 128, 128, False, True, True): (1, 32, 1, 8),
        (6144, 6144, 4096, 128, 128, True, False, True): (4, 32, 3, 4),
        (6144, 6144, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (6144, 6144, 4096, 256, 256, True, False, True): (4, 16, 1, 32),
        (6144, 6144, 8192, 32, 32, False, True, True): (1, 128, 1, 8),
        (6144, 6144, 8192, 32, 32, True, False, True): (1, 64, 1, 4),
        (6144, 6144, 8192, 64, 64, False, True, True): (1, 128, 1, 8),
        (6144, 6144, 8192, 64, 64, True, False, True): (4, 32, 3, 4),
        (6144, 6144, 8192, 128, 128, False, True, True): (1, 64, 1, 8),
        (6144, 6144, 8192, 128, 128, True, False, True): (1, 64, 3, 4),
        (6144, 6144, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (6144, 6144, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (6144, 6144, 16384, 32, 32, False, True, True): (1, 256, 1, 8),
        (6144, 6144, 16384, 32, 32, True, False, True): (1, 128, 1, 4),
        (6144, 6144, 16384, 64, 64, False, True, True): (1, 256, 1, 8),
        (6144, 6144, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (6144, 6144, 16384, 128, 128, False, True, True): (1, 128, 1, 8),
        (6144, 6144, 16384, 128, 128, True, False, True): (4, 128, 3, 4),
        (6144, 6144, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (6144, 6144, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (6144, 6144, 32768, 32, 32, False, True, True): (1, 512, 1, 8),
        (6144, 6144, 32768, 32, 32, True, False, True): (1, 256, 1, 4),
        (6144, 6144, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (6144, 6144, 32768, 64, 64, True, False, True): (4, 128, 3, 4),
        (6144, 6144, 32768, 128, 128, False, True, True): (1, 256, 1, 8),
        (6144, 6144, 32768, 128, 128, True, False, True): (1, 256, 3, 4),
        (6144, 6144, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (6144, 6144, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (6144, 6144, 65536, 32, 32, False, True, True): (1, 1024, 1, 8),
        (6144, 6144, 65536, 32, 32, True, False, True): (1, 512, 1, 4),
        (6144, 6144, 65536, 64, 64, False, True, True): (1, 1024, 1, 8),
        (6144, 6144, 65536, 64, 64, True, False, True): (4, 256, 3, 4),
        (6144, 6144, 65536, 128, 128, False, True, True): (1, 512, 1, 8),
        (6144, 6144, 65536, 128, 128, True, False, True): (1, 512, 3, 4),
        (6144, 6144, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1939-1974 / 第 1939-1974 行
````python
        (6144, 6144, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (6144, 6144, 131072, 32, 32, False, True, True): (1, 2048, 1, 8),
        (6144, 6144, 131072, 32, 32, True, False, True): (1, 1024, 1, 4),
        (6144, 6144, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (6144, 6144, 131072, 64, 64, True, False, True): (4, 512, 3, 4),
        (6144, 6144, 131072, 128, 128, False, True, True): (1, 1024, 1, 8),
        (6144, 6144, 131072, 128, 128, True, False, True): (1, 1024, 3, 4),
        (6144, 6144, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (6144, 6144, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (8192, 8192, 256, 32, 32, False, True, True): (1, 4, 1, 8),
        (8192, 8192, 256, 32, 32, True, False, True): (3, 2, 3, 4),
        (8192, 8192, 256, 64, 64, False, True, True): (1, 4, 1, 4),
        (8192, 8192, 256, 64, 64, True, False, True): (1, 4, 1, 4),
        (8192, 8192, 256, 128, 128, False, True, True): (1, 2, 1, 8),
        (8192, 8192, 256, 128, 128, True, False, True): (2, 2, 3, 4),
        (8192, 8192, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (8192, 8192, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (8192, 8192, 512, 32, 32, False, True, True): (4, 8, 1, 8),
        (8192, 8192, 512, 32, 32, True, False, True): (2, 4, 4, 2),
        (8192, 8192, 512, 64, 64, False, True, True): (4, 4, 1, 4),
        (8192, 8192, 512, 64, 64, True, False, True): (3, 2, 3, 4),
        (8192, 8192, 512, 128, 128, False, True, True): (1, 4, 1, 8),
        (8192, 8192, 512, 128, 128, True, False, True): (1, 4, 3, 4),
        (8192, 8192, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (8192, 8192, 512, 256, 256, True, False, True): (1, 2, 1, 32),
        (8192, 8192, 1024, 32, 32, False, True, True): (4, 16, 1, 8),
        (8192, 8192, 1024, 32, 32, True, False, True): (1, 8, 3, 2),
        (8192, 8192, 1024, 64, 64, False, True, True): (4, 8, 1, 4),
        (8192, 8192, 1024, 64, 64, True, False, True): (4, 4, 3, 4),
        (8192, 8192, 1024, 128, 128, False, True, True): (1, 8, 1, 8),
        (8192, 8192, 1024, 128, 128, True, False, True): (1, 8, 3, 4),
        (8192, 8192, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (8192, 8192, 1024, 256, 256, True, False, True): (4, 4, 1, 32),
        (8192, 8192, 2048, 32, 32, False, True, True): (4, 32, 1, 8),
        (8192, 8192, 2048, 32, 32, True, False, True): (1, 16, 3, 2),
        (8192, 8192, 2048, 64, 64, False, True, True): (4, 32, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 1975-2010 / 第 1975-2010 行
````python
        (8192, 8192, 2048, 64, 64, True, False, True): (4, 8, 3, 4),
        (8192, 8192, 2048, 128, 128, False, True, True): (4, 16, 1, 8),
        (8192, 8192, 2048, 128, 128, True, False, True): (4, 16, 3, 4),
        (8192, 8192, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (8192, 8192, 2048, 256, 256, True, False, True): (4, 8, 1, 32),
        (8192, 8192, 4096, 32, 32, False, True, True): (4, 64, 1, 8),
        (8192, 8192, 4096, 32, 32, True, False, True): (2, 32, 3, 2),
        (8192, 8192, 4096, 64, 64, False, True, True): (4, 64, 1, 8),
        (8192, 8192, 4096, 64, 64, True, False, True): (4, 16, 3, 4),
        (8192, 8192, 4096, 128, 128, False, True, True): (4, 32, 1, 8),
        (8192, 8192, 4096, 128, 128, True, False, True): (4, 32, 3, 4),
        (8192, 8192, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (8192, 8192, 4096, 256, 256, True, False, True): (2, 16, 1, 32),
        (8192, 8192, 8192, 32, 32, False, True, True): (4, 128, 1, 8),
        (8192, 8192, 8192, 32, 32, True, False, True): (1, 64, 3, 2),
        (8192, 8192, 8192, 64, 64, False, True, True): (4, 64, 1, 4),
        (8192, 8192, 8192, 64, 64, True, False, True): (4, 32, 3, 4),
        (8192, 8192, 8192, 128, 128, False, True, True): (4, 64, 1, 16),
        (8192, 8192, 8192, 128, 128, True, False, True): (4, 64, 3, 4),
        (8192, 8192, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (8192, 8192, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (8192, 8192, 16384, 32, 32, False, True, True): (4, 256, 1, 8),
        (8192, 8192, 16384, 32, 32, True, False, True): (4, 128, 3, 2),
        (8192, 8192, 16384, 64, 64, False, True, True): (4, 128, 1, 4),
        (8192, 8192, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (8192, 8192, 16384, 128, 128, False, True, True): (4, 128, 1, 16),
        (8192, 8192, 16384, 128, 128, True, False, True): (4, 128, 3, 4),
        (8192, 8192, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (8192, 8192, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (8192, 8192, 32768, 32, 32, False, True, True): (4, 512, 1, 8),
        (8192, 8192, 32768, 32, 32, True, False, True): (2, 256, 3, 2),
        (8192, 8192, 32768, 64, 64, False, True, True): (4, 256, 1, 4),
        (8192, 8192, 32768, 64, 64, True, False, True): (4, 128, 3, 4),
        (8192, 8192, 32768, 128, 128, False, True, True): (4, 256, 1, 16),
        (8192, 8192, 32768, 128, 128, True, False, True): (4, 256, 3, 4),
        (8192, 8192, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2011-2046 / 第 2011-2046 行
````python
        (8192, 8192, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (8192, 8192, 65536, 32, 32, False, True, True): (4, 1024, 1, 8),
        (8192, 8192, 65536, 32, 32, True, False, True): (4, 512, 3, 2),
        (8192, 8192, 65536, 64, 64, False, True, True): (4, 512, 1, 4),
        (8192, 8192, 65536, 64, 64, True, False, True): (4, 256, 3, 4),
        (8192, 8192, 65536, 128, 128, False, True, True): (4, 512, 1, 16),
        (8192, 8192, 65536, 128, 128, True, False, True): (4, 512, 3, 4),
        (8192, 8192, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (8192, 8192, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (8192, 8192, 65792, 32, 32, False, True, True): (4, 1028, 1, 8),
        (8192, 8192, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (8192, 8192, 65792, 64, 64, False, True, True): (4, 1028, 1, 8),
        (8192, 8192, 65792, 64, 64, True, False, True): (2, 257, 3, 4),
        (8192, 8192, 65792, 128, 128, False, True, True): (4, 514, 1, 16),
        (8192, 8192, 65792, 128, 128, True, False, True): (2, 514, 3, 4),
        (8192, 8192, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (8192, 8192, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (8192, 8192, 131072, 32, 32, False, True, True): (4, 2048, 1, 8),
        (8192, 8192, 131072, 32, 32, True, False, True): (4, 1024, 3, 2),
        (8192, 8192, 131072, 64, 64, False, True, True): (4, 1024, 1, 4),
        (8192, 8192, 131072, 64, 64, True, False, True): (4, 512, 3, 4),
        (8192, 8192, 131072, 128, 128, False, True, True): (4, 1024, 1, 16),
        (8192, 8192, 131072, 128, 128, True, False, True): (4, 1024, 3, 4),
        (8192, 8192, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (8192, 8192, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (16384, 16384, 256, 32, 32, False, True, True): (4, 4, 1, 8),
        (16384, 16384, 256, 32, 32, True, False, True): (2, 2, 4, 2),
        (16384, 16384, 256, 64, 64, False, True, True): (2, 2, 1, 4),
        (16384, 16384, 256, 64, 64, True, False, True): (5, 1, 3, 4),
        (16384, 16384, 256, 128, 128, False, True, True): (6, 2, 1, 8),
        (16384, 16384, 256, 128, 128, True, False, True): (6, 2, 3, 4),
        (16384, 16384, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (16384, 16384, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (16384, 16384, 512, 32, 32, False, True, True): (4, 8, 1, 8),
        (16384, 16384, 512, 32, 32, True, False, True): (1, 4, 4, 2),
        (16384, 16384, 512, 64, 64, False, True, True): (4, 4, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2047-2082 / 第 2047-2082 行
````python
        (16384, 16384, 512, 64, 64, True, False, True): (2, 2, 3, 4),
        (16384, 16384, 512, 128, 128, False, True, True): (4, 4, 1, 8),
        (16384, 16384, 512, 128, 128, True, False, True): (4, 4, 3, 4),
        (16384, 16384, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (16384, 16384, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (16384, 16384, 1024, 32, 32, False, True, True): (4, 16, 1, 8),
        (16384, 16384, 1024, 32, 32, True, False, True): (1, 8, 3, 2),
        (16384, 16384, 1024, 64, 64, False, True, True): (4, 8, 1, 4),
        (16384, 16384, 1024, 64, 64, True, False, True): (4, 4, 3, 4),
        (16384, 16384, 1024, 128, 128, False, True, True): (4, 4, 1, 8),
        (16384, 16384, 1024, 128, 128, True, False, True): (4, 8, 3, 4),
        (16384, 16384, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (16384, 16384, 1024, 256, 256, True, False, True): (4, 4, 1, 32),
        (16384, 16384, 2048, 32, 32, False, True, True): (4, 32, 1, 8),
        (16384, 16384, 2048, 32, 32, True, False, True): (2, 16, 3, 2),
        (16384, 16384, 2048, 64, 64, False, True, True): (4, 16, 1, 4),
        (16384, 16384, 2048, 64, 64, True, False, True): (4, 8, 3, 4),
        (16384, 16384, 2048, 128, 128, False, True, True): (4, 16, 1, 8),
        (16384, 16384, 2048, 128, 128, True, False, True): (4, 16, 3, 4),
        (16384, 16384, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (16384, 16384, 2048, 256, 256, True, False, True): (4, 8, 1, 32),
        (16384, 16384, 4096, 32, 32, False, True, True): (4, 64, 1, 8),
        (16384, 16384, 4096, 32, 32, True, False, True): (2, 32, 3, 2),
        (16384, 16384, 4096, 64, 64, False, True, True): (2, 32, 1, 4),
        (16384, 16384, 4096, 64, 64, True, False, True): (4, 16, 3, 4),
        (16384, 16384, 4096, 128, 128, False, True, True): (4, 32, 1, 8),
        (16384, 16384, 4096, 128, 128, True, False, True): (4, 32, 3, 4),
        (16384, 16384, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (16384, 16384, 4096, 256, 256, True, False, True): (4, 16, 1, 32),
        (16384, 16384, 8192, 32, 32, False, True, True): (4, 128, 1, 8),
        (16384, 16384, 8192, 32, 32, True, False, True): (2, 64, 3, 2),
        (16384, 16384, 8192, 64, 64, False, True, True): (4, 64, 1, 4),
        (16384, 16384, 8192, 64, 64, True, False, True): (4, 32, 3, 4),
        (16384, 16384, 8192, 128, 128, False, True, True): (4, 64, 1, 16),
        (16384, 16384, 8192, 128, 128, True, False, True): (4, 64, 3, 4),
        (16384, 16384, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2083-2118 / 第 2083-2118 行
````python
        (16384, 16384, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (16384, 16384, 16384, 32, 32, False, True, True): (4, 256, 1, 8),
        (16384, 16384, 16384, 32, 32, True, False, True): (2, 128, 3, 2),
        (16384, 16384, 16384, 64, 64, False, True, True): (4, 128, 1, 4),
        (16384, 16384, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (16384, 16384, 16384, 128, 128, False, True, True): (1, 64, 1, 8),
        (16384, 16384, 16384, 128, 128, True, False, True): (4, 128, 3, 4),
        (16384, 16384, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (16384, 16384, 16384, 256, 256, True, False, True): (4, 64, 1, 32),
        (16384, 16384, 32768, 32, 32, False, True, True): (4, 512, 1, 8),
        (16384, 16384, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (16384, 16384, 32768, 64, 64, False, True, True): (4, 256, 1, 4),
        (16384, 16384, 32768, 64, 64, True, False, True): (4, 128, 3, 4),
        (16384, 16384, 32768, 128, 128, False, True, True): (4, 256, 1, 16),
        (16384, 16384, 32768, 128, 128, True, False, True): (4, 256, 3, 4),
        (16384, 16384, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (16384, 16384, 32768, 256, 256, True, False, True): (4, 128, 1, 32),
        (16384, 16384, 65536, 32, 32, False, True, True): (4, 1024, 1, 8),
        (16384, 16384, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (16384, 16384, 65536, 64, 64, False, True, True): (2, 512, 1, 4),
        (16384, 16384, 65536, 64, 64, True, False, True): (4, 256, 3, 4),
        (16384, 16384, 65536, 128, 128, False, True, True): (4, 512, 1, 16),
        (16384, 16384, 65536, 128, 128, True, False, True): (4, 512, 3, 4),
        (16384, 16384, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (16384, 16384, 65536, 256, 256, True, False, True): (4, 256, 1, 32),
        (16384, 16384, 65792, 32, 32, False, True, True): (4, 1028, 1, 8),
        (16384, 16384, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (16384, 16384, 65792, 64, 64, False, True, True): (2, 514, 1, 4),
        (16384, 16384, 65792, 64, 64, True, False, True): (2, 257, 3, 4),
        (16384, 16384, 65792, 128, 128, False, True, True): (2, 514, 1, 16),
        (16384, 16384, 65792, 128, 128, True, False, True): (2, 514, 3, 4),
        (16384, 16384, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (16384, 16384, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (16384, 16384, 131072, 32, 32, False, True, True): (4, 1024, 1, 8),
        (16384, 16384, 131072, 32, 32, True, False, True): (4, 512, 3, 4),
        (16384, 16384, 131072, 64, 64, False, True, True): (4, 1024, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2119-2154 / 第 2119-2154 行
````python
        (16384, 16384, 131072, 64, 64, True, False, True): (4, 1024, 3, 2),
        (16384, 16384, 131072, 128, 128, False, True, True): (2, 1024, 3, 8),
        (16384, 16384, 131072, 128, 128, True, False, True): (4, 1024, 3, 4),
        (16384, 16384, 131072, 256, 256, False, True, True): (4, 512, 1, 32),
        (16384, 16384, 131072, 256, 256, True, False, True): (4, 512, 1, 32),
        (32768, 32768, 256, 32, 32, False, True, True): (4, 4, 1, 8),
        (32768, 32768, 256, 32, 32, True, False, True): (1, 2, 4, 2),
        (32768, 32768, 256, 64, 64, False, True, True): (2, 2, 1, 4),
        (32768, 32768, 256, 64, 64, True, False, True): (2, 1, 3, 4),
        (32768, 32768, 256, 128, 128, False, True, True): (4, 2, 1, 8),
        (32768, 32768, 256, 128, 128, True, False, True): (4, 2, 3, 4),
        (32768, 32768, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (32768, 32768, 256, 256, 256, True, False, True): (1, 1, 1, 32),
        (32768, 32768, 512, 32, 32, False, True, True): (4, 8, 1, 8),
        (32768, 32768, 512, 32, 32, True, False, True): (1, 4, 3, 2),
        (32768, 32768, 512, 64, 64, False, True, True): (4, 4, 1, 4),
        (32768, 32768, 512, 64, 64, True, False, True): (4, 2, 3, 4),
        (32768, 32768, 512, 128, 128, False, True, True): (1, 2, 1, 8),
        (32768, 32768, 512, 128, 128, True, False, True): (4, 4, 3, 4),
        (32768, 32768, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (32768, 32768, 512, 256, 256, True, False, True): (2, 2, 1, 32),
        (32768, 32768, 1024, 32, 32, False, True, True): (4, 16, 1, 8),
        (32768, 32768, 1024, 32, 32, True, False, True): (1, 8, 4, 2),
        (32768, 32768, 1024, 64, 64, False, True, True): (4, 8, 1, 4),
        (32768, 32768, 1024, 64, 64, True, False, True): (4, 4, 3, 4),
        (32768, 32768, 1024, 128, 128, False, True, True): (1, 4, 1, 8),
        (32768, 32768, 1024, 128, 128, True, False, True): (4, 8, 3, 4),
        (32768, 32768, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (32768, 32768, 1024, 256, 256, True, False, True): (1, 4, 1, 32),
        (32768, 32768, 2048, 32, 32, False, True, True): (2, 32, 1, 8),
        (32768, 32768, 2048, 32, 32, True, False, True): (1, 16, 4, 2),
        (32768, 32768, 2048, 64, 64, False, True, True): (2, 16, 1, 4),
        (32768, 32768, 2048, 64, 64, True, False, True): (4, 8, 3, 4),
        (32768, 32768, 2048, 128, 128, False, True, True): (1, 8, 1, 8),
        (32768, 32768, 2048, 128, 128, True, False, True): (4, 16, 3, 4),
        (32768, 32768, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2155-2190 / 第 2155-2190 行
````python
        (32768, 32768, 2048, 256, 256, True, False, True): (4, 8, 1, 32),
        (32768, 32768, 4096, 32, 32, False, True, True): (2, 64, 1, 8),
        (32768, 32768, 4096, 32, 32, True, False, True): (2, 32, 3, 2),
        (32768, 32768, 4096, 64, 64, False, True, True): (2, 32, 1, 4),
        (32768, 32768, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (32768, 32768, 4096, 128, 128, False, True, True): (1, 16, 1, 8),
        (32768, 32768, 4096, 128, 128, True, False, True): (2, 32, 3, 4),
        (32768, 32768, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (32768, 32768, 4096, 256, 256, True, False, True): (4, 16, 1, 32),
        (32768, 32768, 8192, 32, 32, False, True, True): (2, 128, 1, 8),
        (32768, 32768, 8192, 32, 32, True, False, True): (2, 64, 3, 2),
        (32768, 32768, 8192, 64, 64, False, True, True): (2, 64, 1, 4),
        (32768, 32768, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (32768, 32768, 8192, 128, 128, False, True, True): (1, 32, 1, 8),
        (32768, 32768, 8192, 128, 128, True, False, True): (4, 64, 3, 4),
        (32768, 32768, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (32768, 32768, 8192, 256, 256, True, False, True): (4, 32, 1, 32),
        (32768, 32768, 16384, 32, 32, False, True, True): (2, 256, 1, 8),
        (32768, 32768, 16384, 32, 32, True, False, True): (2, 128, 4, 2),
        (32768, 32768, 16384, 64, 64, False, True, True): (2, 128, 1, 4),
        (32768, 32768, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (32768, 32768, 16384, 128, 128, False, True, True): (1, 64, 1, 8),
        (32768, 32768, 16384, 128, 128, True, False, True): (4, 128, 3, 4),
        (32768, 32768, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (32768, 32768, 16384, 256, 256, True, False, True): (2, 64, 1, 32),
        (32768, 32768, 32768, 32, 32, False, True, True): (2, 512, 1, 8),
        (32768, 32768, 32768, 32, 32, True, False, True): (4, 256, 3, 2),
        (32768, 32768, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (32768, 32768, 32768, 64, 64, True, False, True): (2, 128, 3, 4),
        (32768, 32768, 32768, 128, 128, False, True, True): (1, 128, 1, 8),
        (32768, 32768, 32768, 128, 128, True, False, True): (2, 256, 3, 4),
        (32768, 32768, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (32768, 32768, 32768, 256, 256, True, False, True): (1, 128, 1, 32),
        (32768, 32768, 65536, 32, 32, False, True, True): (2, 512, 1, 8),
        (32768, 32768, 65536, 32, 32, True, False, True): (3, 512, 4, 2),
        (32768, 32768, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2191-2226 / 第 2191-2226 行
````python
        (32768, 32768, 65536, 64, 64, True, False, True): (2, 512, 3, 2),
        (32768, 32768, 65536, 128, 128, False, True, True): (1, 256, 1, 8),
        (32768, 32768, 65536, 128, 128, True, False, True): (2, 512, 3, 4),
        (32768, 32768, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (32768, 32768, 65536, 256, 256, True, False, True): (1, 256, 1, 32),
    },
    ("_int_bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.int8, 0.56)): {
        (192, 192, 256, 64, 64, False, True, True): (3, 4, 3, 32),
        (192, 192, 256, 64, 64, True, False, True): (1, 4, 3, 4),
        (192, 192, 512, 64, 64, False, True, True): (1, 8, 1, 16),
        (192, 192, 512, 64, 64, True, False, True): (1, 8, 5, 4),
        (192, 192, 1024, 64, 64, False, True, True): (4, 16, 1, 16),
        (192, 192, 1024, 64, 64, True, False, True): (3, 16, 3, 4),
        (192, 192, 2048, 64, 64, False, True, True): (5, 32, 1, 8),
        (192, 192, 2048, 64, 64, True, False, True): (2, 32, 4, 4),
        (192, 192, 4096, 64, 64, False, True, True): (4, 64, 1, 16),
        (192, 192, 4096, 64, 64, True, False, True): (1, 32, 4, 4),
        (192, 192, 8192, 64, 64, False, True, True): (2, 128, 1, 8),
        (192, 192, 8192, 64, 64, True, False, True): (3, 64, 1, 4),
        (192, 192, 16384, 64, 64, False, True, True): (2, 256, 1, 8),
        (192, 192, 16384, 64, 64, True, False, True): (1, 128, 3, 2),
        (192, 192, 32768, 64, 64, False, True, True): (2, 512, 1, 8),
        (192, 192, 32768, 64, 64, True, False, True): (3, 128, 1, 4),
        (192, 192, 65536, 64, 64, False, True, True): (3, 1024, 1, 8),
        (192, 192, 65536, 64, 64, True, False, True): (1, 512, 3, 4),
        (192, 192, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (192, 192, 131072, 64, 64, True, False, True): (1, 512, 1, 4),
        (384, 384, 256, 128, 128, False, True, True): (4, 2, 1, 16),
        (384, 384, 256, 128, 128, True, False, True): (1, 2, 3, 4),
        (384, 384, 512, 128, 128, False, True, True): (2, 4, 1, 16),
        (384, 384, 512, 128, 128, True, False, True): (2, 4, 3, 4),
        (384, 384, 1024, 128, 128, False, True, True): (3, 8, 1, 32),
        (384, 384, 1024, 128, 128, True, False, True): (3, 8, 3, 4),
        (384, 384, 2048, 128, 128, False, True, True): (3, 16, 1, 32),
        (384, 384, 2048, 128, 128, True, False, True): (2, 16, 3, 4),
        (384, 384, 4096, 128, 128, False, True, True): (3, 32, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2227-2262 / 第 2227-2262 行
````python
        (384, 384, 4096, 128, 128, True, False, True): (3, 32, 3, 4),
        (384, 384, 8192, 128, 128, False, True, True): (2, 64, 1, 32),
        (384, 384, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (384, 384, 16384, 128, 128, False, True, True): (2, 128, 1, 32),
        (384, 384, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (384, 384, 32768, 128, 128, False, True, True): (3, 256, 1, 16),
        (384, 384, 32768, 128, 128, True, False, True): (1, 256, 1, 4),
        (384, 384, 65536, 128, 128, False, True, True): (4, 512, 1, 16),
        (384, 384, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (384, 384, 131072, 128, 128, False, True, True): (4, 1024, 1, 16),
        (384, 384, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (768, 768, 256, 256, 256, False, True, True): (1, 1, 1, 32),
        (768, 768, 256, 256, 256, True, False, True): (3, 1, 1, 32),
        (768, 768, 512, 256, 256, False, True, True): (1, 2, 1, 32),
        (768, 768, 512, 256, 256, True, False, True): (1, 2, 1, 32),
        (768, 768, 1024, 256, 256, False, True, True): (1, 4, 1, 32),
        (768, 768, 1024, 256, 256, True, False, True): (2, 4, 1, 32),
        (768, 768, 2048, 256, 256, False, True, True): (1, 8, 1, 32),
        (768, 768, 2048, 256, 256, True, False, True): (2, 8, 1, 32),
        (768, 768, 4096, 256, 256, False, True, True): (1, 16, 1, 32),
        (768, 768, 4096, 256, 256, True, False, True): (1, 16, 1, 32),
        (768, 768, 8192, 256, 256, False, True, True): (1, 32, 1, 32),
        (768, 768, 8192, 256, 256, True, False, True): (2, 32, 1, 32),
        (768, 768, 16384, 256, 256, False, True, True): (1, 64, 1, 32),
        (768, 768, 16384, 256, 256, True, False, True): (7, 64, 1, 32),
        (768, 768, 32768, 256, 256, False, True, True): (1, 128, 1, 32),
        (768, 768, 32768, 256, 256, True, False, True): (1, 128, 1, 32),
        (768, 768, 65536, 256, 256, False, True, True): (1, 256, 1, 32),
        (768, 768, 65536, 256, 256, True, False, True): (1, 256, 1, 32),
        (768, 768, 131072, 256, 256, False, True, True): (1, 512, 1, 32),
        (768, 768, 131072, 256, 256, True, False, True): (1, 512, 1, 32),
    },
    ("_int_bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.int8, 1.0)): {
        (256, 256, 256, 256, 256, False, True, True): (2, 1, 1, 4),
        (256, 256, 256, 256, 256, True, False, True): (2, 1, 2, 1),
        (256, 256, 512, 256, 256, False, True, True): (2, 1, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2263-2298 / 第 2263-2298 行
````python
        (256, 256, 512, 256, 256, True, False, True): (2, 2, 2, 8),
        (256, 256, 1024, 256, 256, False, True, True): (1, 4, 1, 4),
        (256, 256, 1024, 256, 256, True, False, True): (1, 2, 2, 4),
        (256, 256, 2048, 256, 256, False, True, True): (1, 4, 1, 2),
        (256, 256, 2048, 256, 256, True, False, True): (1, 8, 1, 2),
        (256, 256, 4096, 256, 256, False, True, True): (1, 16, 1, 4),
        (256, 256, 4096, 256, 256, True, False, True): (1, 16, 1, 2),
        (256, 256, 8192, 256, 256, False, True, True): (1, 16, 3, 4),
        (256, 256, 8192, 256, 256, True, False, True): (1, 8, 1, 4),
        (256, 256, 16384, 256, 256, False, True, True): (2, 16, 1, 8),
        (256, 256, 16384, 256, 256, True, False, True): (1, 32, 1, 2),
        (256, 256, 32768, 256, 256, False, True, True): (1, 128, 1, 8),
        (256, 256, 32768, 256, 256, True, False, True): (1, 128, 1, 4),
        (256, 256, 65536, 256, 256, False, True, True): (1, 4, 1, 1),
        (256, 256, 65536, 256, 256, True, False, True): (1, 128, 1, 4),
        (256, 256, 65792, 256, 256, False, True, True): (1, 128, 2, 16),
        (256, 256, 65792, 256, 256, True, False, True): (1, 16, 3, 4),
        (256, 256, 131072, 256, 256, False, True, True): (1, 512, 1, 4),
        (256, 256, 131072, 256, 256, True, False, True): (1, 512, 1, 2),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.bfloat16, 0.5)): {
        (16, 16, 16, 16, 16, False, False, False): (2, 1, 1, 2),
        (16, 16, 16, 16, 16, False, False, True): (1, 1, 1, 4),
        (16, 16, 16, 16, 16, False, True, False): (1, 1, 3, 16),
        (16, 16, 16, 16, 16, False, True, True): (1, 1, 1, 8),
        (16, 16, 16, 16, 16, True, False, False): (2, 1, 1, 8),
        (16, 16, 16, 16, 16, True, False, True): (1, 1, 1, 8),
        (16, 16, 32, 16, 16, False, False, False): (1, 2, 1, 8),
        (16, 16, 32, 16, 16, False, False, True): (1, 2, 2, 4),
        (16, 16, 32, 16, 16, False, True, False): (1, 1, 2, 4),
        (16, 16, 32, 16, 16, False, True, True): (1, 1, 2, 4),
        (16, 16, 32, 16, 16, True, False, False): (1, 1, 2, 4),
        (16, 16, 32, 16, 16, True, False, True): (2, 2, 1, 2),
        (16, 16, 64, 16, 16, False, False, False): (1, 4, 2, 4),
        (16, 16, 64, 16, 16, False, False, True): (1, 2, 1, 2),
        (16, 16, 64, 16, 16, False, True, False): (2, 1, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2299-2334 / 第 2299-2334 行
````python
        (16, 16, 64, 16, 16, False, True, True): (1, 4, 1, 8),
        (16, 16, 64, 16, 16, True, False, False): (1, 4, 1, 1),
        (16, 16, 64, 16, 16, True, False, True): (1, 4, 2, 4),
        (16, 32, 16, 16, 16, False, False, False): (1, 1, 2, 2),
        (16, 32, 16, 16, 16, False, False, True): (1, 1, 1, 4),
        (16, 32, 16, 16, 16, False, True, False): (1, 1, 1, 2),
        (16, 32, 16, 16, 16, False, True, True): (1, 1, 1, 1),
        (16, 32, 16, 16, 16, True, False, False): (1, 1, 1, 2),
        (16, 32, 16, 16, 16, True, False, True): (2, 1, 1, 2),
        (16, 32, 16, 16, 32, False, False, False): (1, 1, 1, 4),
        (16, 32, 16, 16, 32, False, False, True): (1, 1, 1, 8),
        (16, 32, 16, 16, 32, False, True, False): (1, 1, 1, 8),
        (16, 32, 16, 16, 32, False, True, True): (1, 1, 2, 4),
        (16, 32, 16, 16, 32, True, False, False): (1, 1, 1, 2),
        (16, 32, 16, 16, 32, True, False, True): (1, 1, 1, 1),
        (16, 32, 32, 16, 16, False, False, False): (2, 2, 1, 4),
        (16, 32, 32, 16, 16, False, False, True): (2, 2, 1, 2),
        (16, 32, 32, 16, 16, False, True, False): (1, 1, 2, 8),
        (16, 32, 32, 16, 16, False, True, True): (1, 2, 1, 1),
        (16, 32, 32, 16, 16, True, False, False): (1, 1, 1, 8),
        (16, 32, 32, 16, 16, True, False, True): (1, 2, 1, 4),
        (16, 32, 32, 16, 32, False, False, False): (1, 1, 2, 8),
        (16, 32, 32, 16, 32, False, False, True): (2, 1, 1, 8),
        (16, 32, 32, 16, 32, False, True, False): (1, 1, 1, 4),
        (16, 32, 32, 16, 32, False, True, True): (1, 1, 1, 4),
        (16, 32, 32, 16, 32, True, False, False): (1, 2, 1, 8),
        (16, 32, 32, 16, 32, True, False, True): (1, 1, 1, 4),
        (16, 32, 64, 16, 16, False, False, False): (1, 4, 3, 8),
        (16, 32, 64, 16, 16, False, False, True): (1, 4, 1, 4),
        (16, 32, 64, 16, 16, False, True, False): (1, 4, 1, 4),
        (16, 32, 64, 16, 16, False, True, True): (2, 4, 1, 4),
        (16, 32, 64, 16, 16, True, False, False): (1, 2, 1, 4),
        (16, 32, 64, 16, 16, True, False, True): (1, 2, 1, 4),
        (16, 32, 64, 16, 32, False, False, False): (1, 4, 1, 8),
        (16, 32, 64, 16, 32, False, False, True): (1, 4, 1, 4),
        (16, 32, 64, 16, 32, False, True, False): (1, 4, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2335-2370 / 第 2335-2370 行
````python
        (16, 32, 64, 16, 32, False, True, True): (1, 2, 1, 4),
        (16, 32, 64, 16, 32, True, False, False): (1, 2, 1, 4),
        (16, 32, 64, 16, 32, True, False, True): (1, 2, 1, 2),
        (16, 64, 16, 16, 32, False, False, False): (1, 1, 1, 2),
        (16, 64, 16, 16, 32, False, False, True): (1, 1, 2, 2),
        (16, 64, 16, 16, 32, False, True, False): (1, 1, 2, 8),
        (16, 64, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (16, 64, 16, 16, 32, True, False, False): (1, 1, 1, 8),
        (16, 64, 16, 16, 32, True, False, True): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, False, False, False): (1, 2, 1, 2),
        (16, 64, 32, 16, 32, False, False, True): (1, 2, 1, 4),
        (16, 64, 32, 16, 32, False, True, False): (1, 2, 1, 4),
        (16, 64, 32, 16, 32, False, True, True): (2, 2, 1, 4),
        (16, 64, 32, 16, 32, True, False, False): (1, 2, 1, 4),
        (16, 64, 32, 16, 32, True, False, True): (1, 2, 1, 8),
        (16, 64, 64, 16, 32, False, False, False): (1, 2, 1, 4),
        (16, 64, 64, 16, 32, False, False, True): (1, 4, 2, 2),
        (16, 64, 64, 16, 32, False, True, False): (1, 1, 1, 4),
        (16, 64, 64, 16, 32, False, True, True): (1, 4, 1, 2),
        (16, 64, 64, 16, 32, True, False, False): (1, 2, 1, 4),
        (16, 64, 64, 16, 32, True, False, True): (1, 4, 1, 4),
        (32, 16, 16, 16, 16, False, False, False): (1, 1, 1, 8),
        (32, 16, 16, 16, 16, False, False, True): (1, 1, 2, 4),
        (32, 16, 16, 16, 16, False, True, False): (1, 1, 1, 4),
        (32, 16, 16, 16, 16, False, True, True): (1, 1, 2, 4),
        (32, 16, 16, 16, 16, True, False, False): (1, 1, 1, 2),
        (32, 16, 16, 16, 16, True, False, True): (1, 1, 1, 4),
        (32, 16, 32, 16, 16, False, False, False): (1, 1, 1, 4),
        (32, 16, 32, 16, 16, False, False, True): (2, 2, 1, 4),
        (32, 16, 32, 16, 16, False, True, False): (1, 2, 2, 2),
        (32, 16, 32, 16, 16, False, True, True): (2, 2, 1, 4),
        (32, 16, 32, 16, 16, True, False, False): (1, 2, 2, 8),
        (32, 16, 32, 16, 16, True, False, True): (1, 2, 1, 2),
        (32, 16, 64, 16, 16, False, False, False): (1, 4, 1, 4),
        (32, 16, 64, 16, 16, False, False, True): (1, 4, 2, 4),
        (32, 16, 64, 16, 16, False, True, False): (1, 2, 2, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2371-2406 / 第 2371-2406 行
````python
        (32, 16, 64, 16, 16, False, True, True): (3, 4, 1, 4),
        (32, 16, 64, 16, 16, True, False, False): (1, 2, 1, 2),
        (32, 16, 64, 16, 16, True, False, True): (1, 2, 1, 4),
        (32, 32, 16, 16, 16, False, False, False): (1, 1, 3, 4),
        (32, 32, 16, 16, 16, False, False, True): (1, 1, 1, 4),
        (32, 32, 16, 16, 16, False, True, False): (1, 1, 1, 2),
        (32, 32, 16, 16, 16, False, True, True): (1, 1, 1, 4),
        (32, 32, 16, 16, 16, True, False, False): (1, 1, 1, 4),
        (32, 32, 16, 16, 16, True, False, True): (1, 1, 2, 2),
        (32, 32, 16, 16, 32, False, False, False): (2, 1, 1, 4),
        (32, 32, 16, 16, 32, False, False, True): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, False, True, False): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, False, True, True): (3, 1, 2, 4),
        (32, 32, 16, 16, 32, True, False, False): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, True, False, True): (1, 1, 1, 4),
        (32, 32, 16, 32, 32, False, False, False): (1, 1, 1, 8),
        (32, 32, 16, 32, 32, False, False, True): (1, 1, 1, 4),
        (32, 32, 16, 32, 32, False, True, False): (1, 1, 2, 1),
        (32, 32, 16, 32, 32, False, True, True): (2, 1, 2, 2),
        (32, 32, 16, 32, 32, True, False, False): (1, 1, 1, 8),
        (32, 32, 16, 32, 32, True, False, True): (2, 1, 3, 4),
        (32, 32, 32, 16, 16, False, False, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 16, False, False, True): (2, 2, 1, 4),
        (32, 32, 32, 16, 16, False, True, False): (1, 1, 1, 8),
        (32, 32, 32, 16, 16, False, True, True): (2, 2, 1, 4),
        (32, 32, 32, 16, 16, True, False, False): (1, 1, 1, 4),
        (32, 32, 32, 16, 16, True, False, True): (2, 2, 2, 4),
        (32, 32, 32, 16, 32, False, False, False): (2, 2, 1, 8),
        (32, 32, 32, 16, 32, False, False, True): (1, 2, 1, 2),
        (32, 32, 32, 16, 32, False, True, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, True, True): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, True, False, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, True, False, True): (1, 2, 1, 2),
        (32, 32, 32, 32, 32, False, False, False): (1, 1, 3, 8),
        (32, 32, 32, 32, 32, False, False, True): (1, 1, 1, 8),
        (32, 32, 32, 32, 32, False, True, False): (2, 1, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2407-2442 / 第 2407-2442 行
````python
        (32, 32, 32, 32, 32, False, True, True): (2, 1, 1, 2),
        (32, 32, 32, 32, 32, True, False, False): (1, 1, 1, 2),
        (32, 32, 32, 32, 32, True, False, True): (4, 1, 1, 1),
        (32, 32, 64, 16, 16, False, False, False): (1, 4, 1, 4),
        (32, 32, 64, 16, 16, False, False, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 16, False, True, False): (1, 2, 1, 8),
        (32, 32, 64, 16, 16, False, True, True): (1, 4, 1, 2),
        (32, 32, 64, 16, 16, True, False, False): (2, 4, 1, 2),
        (32, 32, 64, 16, 16, True, False, True): (1, 4, 1, 2),
        (32, 32, 64, 16, 32, False, False, False): (1, 2, 1, 8),
        (32, 32, 64, 16, 32, False, False, True): (1, 4, 2, 2),
        (32, 32, 64, 16, 32, False, True, False): (1, 2, 1, 4),
        (32, 32, 64, 16, 32, False, True, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 32, True, False, False): (1, 4, 2, 2),
        (32, 32, 64, 16, 32, True, False, True): (3, 4, 2, 2),
        (32, 32, 64, 32, 32, False, False, False): (2, 2, 1, 4),
        (32, 32, 64, 32, 32, False, False, True): (1, 2, 1, 4),
        (32, 32, 64, 32, 32, False, True, False): (1, 1, 1, 8),
        (32, 32, 64, 32, 32, False, True, True): (1, 1, 1, 4),
        (32, 32, 64, 32, 32, True, False, False): (1, 2, 1, 2),
        (32, 32, 64, 32, 32, True, False, True): (3, 2, 1, 8),
        (32, 64, 16, 16, 32, False, False, False): (1, 1, 2, 2),
        (32, 64, 16, 16, 32, False, False, True): (1, 1, 1, 4),
        (32, 64, 16, 16, 32, False, True, False): (1, 1, 2, 4),
        (32, 64, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (32, 64, 16, 16, 32, True, False, False): (1, 1, 1, 2),
        (32, 64, 16, 16, 32, True, False, True): (2, 1, 2, 2),
        (32, 64, 16, 32, 32, False, False, False): (1, 1, 1, 1),
        (32, 64, 16, 32, 32, False, False, True): (2, 1, 1, 4),
        (32, 64, 16, 32, 32, False, True, False): (1, 1, 1, 1),
        (32, 64, 16, 32, 32, False, True, True): (1, 1, 2, 2),
        (32, 64, 16, 32, 32, True, False, False): (1, 1, 2, 4),
        (32, 64, 16, 32, 32, True, False, True): (1, 1, 1, 4),
        (32, 64, 32, 16, 32, False, False, False): (2, 2, 1, 4),
        (32, 64, 32, 16, 32, False, False, True): (1, 2, 1, 4),
        (32, 64, 32, 16, 32, False, True, False): (1, 1, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2443-2478 / 第 2443-2478 行
````python
        (32, 64, 32, 16, 32, False, True, True): (2, 2, 3, 4),
        (32, 64, 32, 16, 32, True, False, False): (1, 1, 1, 2),
        (32, 64, 32, 16, 32, True, False, True): (1, 2, 1, 2),
        (32, 64, 32, 32, 32, False, False, False): (1, 1, 1, 2),
        (32, 64, 32, 32, 32, False, False, True): (2, 1, 1, 4),
        (32, 64, 32, 32, 32, False, True, False): (1, 1, 1, 8),
        (32, 64, 32, 32, 32, False, True, True): (1, 1, 2, 4),
        (32, 64, 32, 32, 32, True, False, False): (2, 1, 1, 4),
        (32, 64, 32, 32, 32, True, False, True): (1, 1, 2, 4),
        (32, 64, 64, 16, 32, False, False, False): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, False, False, True): (1, 4, 2, 4),
        (32, 64, 64, 16, 32, False, True, False): (1, 4, 2, 2),
        (32, 64, 64, 16, 32, False, True, True): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, True, False, False): (1, 4, 1, 8),
        (32, 64, 64, 16, 32, True, False, True): (1, 4, 2, 1),
        (32, 64, 64, 32, 32, False, False, False): (1, 1, 1, 4),
        (32, 64, 64, 32, 32, False, False, True): (2, 2, 1, 4),
        (32, 64, 64, 32, 32, False, True, False): (1, 1, 1, 4),
        (32, 64, 64, 32, 32, False, True, True): (2, 2, 1, 4),
        (32, 64, 64, 32, 32, True, False, False): (1, 2, 2, 4),
        (32, 64, 64, 32, 32, True, False, True): (2, 2, 3, 4),
        (64, 32, 16, 32, 32, False, False, False): (1, 1, 1, 4),
        (64, 32, 16, 32, 32, False, False, True): (1, 1, 1, 4),
        (64, 32, 16, 32, 32, False, True, False): (1, 1, 1, 8),
        (64, 32, 16, 32, 32, False, True, True): (1, 1, 1, 4),
        (64, 32, 16, 32, 32, True, False, False): (1, 1, 1, 16),
        (64, 32, 16, 32, 32, True, False, True): (2, 1, 1, 4),
        (64, 32, 32, 32, 32, False, False, False): (1, 1, 3, 4),
        (64, 32, 32, 32, 32, False, False, True): (2, 1, 1, 4),
        (64, 32, 32, 32, 32, False, True, False): (1, 1, 2, 4),
        (64, 32, 32, 32, 32, False, True, True): (2, 1, 1, 4),
        (64, 32, 32, 32, 32, True, False, False): (2, 1, 1, 16),
        (64, 32, 32, 32, 32, True, False, True): (2, 1, 1, 4),
        (64, 32, 64, 32, 32, False, False, False): (1, 2, 1, 4),
        (64, 32, 64, 32, 32, False, False, True): (2, 2, 1, 4),
        (64, 32, 64, 32, 32, False, True, False): (1, 1, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2479-2514 / 第 2479-2514 行
````python
        (64, 32, 64, 32, 32, False, True, True): (2, 2, 1, 4),
        (64, 32, 64, 32, 32, True, False, False): (1, 2, 1, 8),
        (64, 32, 64, 32, 32, True, False, True): (2, 2, 3, 4),
        (64, 64, 16, 32, 32, False, False, False): (1, 1, 2, 16),
        (64, 64, 16, 32, 32, False, False, True): (1, 1, 3, 4),
        (64, 64, 16, 32, 32, False, True, False): (1, 1, 1, 2),
        (64, 64, 16, 32, 32, False, True, True): (2, 1, 1, 4),
        (64, 64, 16, 32, 32, True, False, False): (2, 1, 3, 2),
        (64, 64, 16, 32, 32, True, False, True): (1, 1, 2, 4),
        (64, 64, 32, 32, 32, False, False, False): (1, 1, 1, 8),
        (64, 64, 32, 32, 32, False, False, True): (2, 1, 2, 4),
        (64, 64, 32, 32, 32, False, True, False): (2, 1, 1, 4),
        (64, 64, 32, 32, 32, False, True, True): (1, 1, 2, 4),
        (64, 64, 32, 32, 32, True, False, False): (2, 1, 1, 4),
        (64, 64, 32, 32, 32, True, False, True): (1, 1, 2, 4),
        (64, 64, 64, 32, 32, False, False, False): (1, 2, 2, 4),
        (64, 64, 64, 32, 32, False, False, True): (1, 2, 2, 2),
        (64, 64, 64, 32, 32, False, True, False): (1, 2, 1, 2),
        (64, 64, 64, 32, 32, False, True, True): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, True, False, False): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, True, False, True): (1, 2, 1, 4),
        (192, 192, 256, 16, 16, False, True, True): (1, 8, 5, 4),
        (192, 192, 256, 16, 16, True, False, True): (2, 8, 5, 2),
        (192, 192, 256, 32, 32, False, True, True): (1, 8, 6, 4),
        (192, 192, 256, 32, 32, True, False, True): (3, 8, 5, 2),
        (192, 192, 512, 16, 16, False, True, True): (1, 16, 5, 2),
        (192, 192, 512, 16, 16, True, False, True): (1, 8, 4, 2),
        (192, 192, 512, 32, 32, False, True, True): (2, 16, 5, 4),
        (192, 192, 512, 32, 32, True, False, True): (2, 8, 5, 2),
        (192, 192, 1024, 16, 16, False, True, True): (1, 16, 3, 4),
        (192, 192, 1024, 16, 16, True, False, True): (1, 16, 6, 2),
        (192, 192, 1024, 32, 32, False, True, True): (1, 32, 3, 4),
        (192, 192, 1024, 32, 32, True, False, True): (1, 16, 4, 2),
        (192, 192, 2048, 16, 16, False, True, True): (1, 32, 1, 4),
        (192, 192, 2048, 16, 16, True, False, True): (4, 32, 4, 2),
        (192, 192, 2048, 32, 32, False, True, True): (1, 16, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2515-2550 / 第 2515-2550 行
````python
        (192, 192, 2048, 32, 32, True, False, True): (2, 32, 4, 2),
        (192, 192, 4096, 16, 16, False, True, True): (2, 64, 1, 4),
        (192, 192, 4096, 16, 16, True, False, True): (1, 32, 3, 2),
        (192, 192, 4096, 32, 32, False, True, True): (1, 64, 1, 8),
        (192, 192, 4096, 32, 32, True, False, True): (2, 32, 4, 4),
        (192, 192, 8192, 16, 16, False, True, True): (1, 64, 1, 4),
        (192, 192, 8192, 16, 16, True, False, True): (2, 32, 3, 1),
        (192, 192, 8192, 32, 32, False, True, True): (3, 128, 1, 4),
        (192, 192, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (192, 192, 16384, 16, 16, False, True, True): (1, 128, 1, 4),
        (192, 192, 16384, 16, 16, True, False, True): (4, 64, 3, 1),
        (192, 192, 16384, 32, 32, False, True, True): (1, 128, 1, 4),
        (192, 192, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (192, 192, 32768, 16, 16, False, True, True): (2, 256, 1, 2),
        (192, 192, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (192, 192, 32768, 32, 32, False, True, True): (2, 256, 1, 4),
        (192, 192, 32768, 32, 32, True, False, True): (4, 128, 3, 4),
        (192, 192, 65536, 16, 16, False, True, True): (2, 512, 1, 2),
        (192, 192, 65536, 16, 16, True, False, True): (2, 256, 3, 2),
        (192, 192, 65536, 32, 32, False, True, True): (2, 512, 1, 4),
        (192, 192, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (192, 192, 131072, 16, 16, False, True, True): (4, 1024, 1, 2),
        (192, 192, 131072, 16, 16, True, False, True): (3, 512, 3, 2),
        (192, 192, 131072, 32, 32, False, True, True): (1, 1024, 1, 2),
        (192, 192, 131072, 32, 32, True, False, True): (1, 512, 3, 4),
        (256, 256, 256, 16, 16, False, True, True): (4, 8, 5, 1),
        (256, 256, 256, 16, 16, True, False, True): (2, 8, 4, 2),
        (256, 256, 256, 32, 32, False, True, True): (2, 8, 5, 2),
        (256, 256, 256, 32, 32, True, False, True): (1, 8, 5, 4),
        (256, 256, 256, 64, 64, False, True, True): (2, 4, 4, 4),
        (256, 256, 256, 64, 64, True, False, True): (1, 4, 3, 4),
        (256, 256, 256, 128, 128, False, True, True): (4, 2, 2, 8),
        (256, 256, 256, 128, 128, True, False, True): (1, 2, 2, 8),
        (256, 256, 512, 16, 16, False, True, True): (1, 16, 5, 1),
        (256, 256, 512, 16, 16, True, False, True): (3, 16, 3, 2),
        (256, 256, 512, 32, 32, False, True, True): (2, 8, 5, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2551-2586 / 第 2551-2586 行
````python
        (256, 256, 512, 32, 32, True, False, True): (1, 16, 4, 4),
        (256, 256, 512, 64, 64, False, True, True): (1, 8, 4, 4),
        (256, 256, 512, 64, 64, True, False, True): (3, 8, 3, 4),
        (256, 256, 512, 128, 128, False, True, True): (1, 4, 2, 8),
        (256, 256, 512, 128, 128, True, False, True): (1, 4, 2, 8),
        (256, 256, 1024, 16, 16, False, True, True): (1, 16, 5, 4),
        (256, 256, 1024, 16, 16, True, False, True): (5, 16, 4, 2),
        (256, 256, 1024, 32, 32, False, True, True): (1, 32, 5, 2),
        (256, 256, 1024, 32, 32, True, False, True): (2, 16, 5, 2),
        (256, 256, 1024, 64, 64, False, True, True): (1, 16, 4, 4),
        (256, 256, 1024, 64, 64, True, False, True): (1, 16, 4, 4),
        (256, 256, 1024, 128, 128, False, True, True): (1, 8, 2, 8),
        (256, 256, 1024, 128, 128, True, False, True): (1, 8, 2, 8),
        (256, 256, 2048, 16, 16, False, True, True): (1, 16, 4, 4),
        (256, 256, 2048, 16, 16, True, False, True): (2, 32, 5, 1),
        (256, 256, 2048, 32, 32, False, True, True): (1, 64, 4, 1),
        (256, 256, 2048, 32, 32, True, False, True): (2, 32, 4, 2),
        (256, 256, 2048, 64, 64, False, True, True): (8, 16, 5, 4),
        (256, 256, 2048, 64, 64, True, False, True): (1, 16, 4, 4),
        (256, 256, 2048, 128, 128, False, True, True): (2, 16, 2, 8),
        (256, 256, 2048, 128, 128, True, False, True): (1, 16, 2, 8),
        (256, 256, 4096, 16, 16, False, True, True): (1, 64, 1, 4),
        (256, 256, 4096, 16, 16, True, False, True): (1, 16, 3, 2),
        (256, 256, 4096, 32, 32, False, True, True): (6, 32, 3, 2),
        (256, 256, 4096, 32, 32, True, False, True): (4, 32, 4, 2),
        (256, 256, 4096, 64, 64, False, True, True): (6, 64, 3, 4),
        (256, 256, 4096, 64, 64, True, False, True): (2, 64, 3, 4),
        (256, 256, 4096, 128, 128, False, True, True): (1, 32, 2, 8),
        (256, 256, 4096, 128, 128, True, False, True): (1, 32, 2, 8),
        (256, 256, 8192, 16, 16, False, True, True): (2, 32, 3, 4),
        (256, 256, 8192, 16, 16, True, False, True): (4, 64, 3, 2),
        (256, 256, 8192, 32, 32, False, True, True): (1, 64, 3, 4),
        (256, 256, 8192, 32, 32, True, False, True): (3, 128, 1, 2),
        (256, 256, 8192, 64, 64, False, True, True): (9, 128, 1, 4),
        (256, 256, 8192, 64, 64, True, False, True): (8, 128, 1, 4),
        (256, 256, 8192, 128, 128, False, True, True): (7, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2587-2622 / 第 2587-2622 行
````python
        (256, 256, 8192, 128, 128, True, False, True): (1, 32, 1, 16),
        (256, 256, 16384, 16, 16, False, True, True): (3, 128, 3, 2),
        (256, 256, 16384, 16, 16, True, False, True): (5, 64, 3, 2),
        (256, 256, 16384, 32, 32, False, True, True): (3, 128, 3, 2),
        (256, 256, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (256, 256, 16384, 64, 64, False, True, True): (3, 128, 1, 4),
        (256, 256, 16384, 64, 64, True, False, True): (2, 128, 1, 4),
        (256, 256, 16384, 128, 128, False, True, True): (7, 128, 1, 4),
        (256, 256, 16384, 128, 128, True, False, True): (1, 128, 2, 8),
        (256, 256, 32768, 16, 16, False, True, True): (2, 128, 3, 2),
        (256, 256, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (256, 256, 32768, 32, 32, False, True, True): (1, 256, 3, 4),
        (256, 256, 32768, 32, 32, True, False, True): (3, 256, 3, 2),
        (256, 256, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (256, 256, 32768, 64, 64, True, False, True): (3, 256, 1, 4),
        (256, 256, 32768, 128, 128, False, True, True): (9, 256, 1, 4),
        (256, 256, 32768, 128, 128, True, False, True): (2, 256, 1, 4),
        (256, 256, 65536, 16, 16, False, True, True): (1, 256, 3, 2),
        (256, 256, 65536, 16, 16, True, False, True): (1, 256, 3, 2),
        (256, 256, 65536, 32, 32, False, True, True): (2, 512, 3, 2),
        (256, 256, 65536, 32, 32, True, False, True): (2, 512, 3, 2),
        (256, 256, 65536, 64, 64, False, True, True): (2, 512, 1, 4),
        (256, 256, 65536, 64, 64, True, False, True): (1, 512, 1, 4),
        (256, 256, 65536, 128, 128, False, True, True): (7, 512, 1, 4),
        (256, 256, 65536, 128, 128, True, False, True): (2, 512, 1, 4),
        (256, 256, 131072, 16, 16, False, True, True): (1, 512, 3, 2),
        (256, 256, 131072, 16, 16, True, False, True): (1, 512, 3, 2),
        (256, 256, 131072, 32, 32, False, True, True): (1, 1024, 3, 2),
        (256, 256, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (256, 256, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (256, 256, 131072, 64, 64, True, False, True): (1, 1024, 1, 4),
        (256, 256, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (256, 256, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (384, 384, 256, 16, 16, False, True, True): (1, 8, 5, 2),
        (384, 384, 256, 16, 16, True, False, True): (3, 4, 5, 2),
        (384, 384, 256, 32, 32, False, True, True): (2, 8, 4, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2623-2658 / 第 2623-2658 行
````python
        (384, 384, 256, 32, 32, True, False, True): (1, 4, 6, 2),
        (384, 384, 256, 64, 64, False, True, True): (2, 4, 4, 4),
        (384, 384, 256, 64, 64, True, False, True): (2, 4, 4, 4),
        (384, 384, 512, 16, 16, False, True, True): (1, 8, 4, 2),
        (384, 384, 512, 16, 16, True, False, True): (1, 4, 5, 4),
        (384, 384, 512, 32, 32, False, True, True): (1, 8, 4, 4),
        (384, 384, 512, 32, 32, True, False, True): (3, 8, 5, 2),
        (384, 384, 512, 64, 64, False, True, True): (3, 8, 3, 4),
        (384, 384, 512, 64, 64, True, False, True): (5, 8, 5, 4),
        (384, 384, 1024, 16, 16, False, True, True): (3, 16, 4, 2),
        (384, 384, 1024, 16, 16, True, False, True): (1, 8, 4, 4),
        (384, 384, 1024, 32, 32, False, True, True): (6, 32, 3, 2),
        (384, 384, 1024, 32, 32, True, False, True): (3, 8, 4, 4),
        (384, 384, 1024, 64, 64, False, True, True): (3, 16, 3, 4),
        (384, 384, 1024, 64, 64, True, False, True): (2, 16, 4, 4),
        (384, 384, 2048, 16, 16, False, True, True): (1, 32, 1, 4),
        (384, 384, 2048, 16, 16, True, False, True): (1, 16, 5, 2),
        (384, 384, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (384, 384, 2048, 32, 32, True, False, True): (1, 8, 4, 4),
        (384, 384, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (384, 384, 2048, 64, 64, True, False, True): (1, 16, 3, 8),
        (384, 384, 4096, 16, 16, False, True, True): (5, 32, 1, 4),
        (384, 384, 4096, 16, 16, True, False, True): (6, 32, 3, 2),
        (384, 384, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
        (384, 384, 4096, 32, 32, True, False, True): (1, 16, 3, 4),
        (384, 384, 4096, 64, 64, False, True, True): (1, 64, 1, 4),
        (384, 384, 4096, 64, 64, True, False, True): (2, 32, 3, 4),
        (384, 384, 8192, 16, 16, False, True, True): (2, 64, 1, 4),
        (384, 384, 8192, 16, 16, True, False, True): (3, 32, 3, 2),
        (384, 384, 8192, 32, 32, False, True, True): (5, 64, 1, 8),
        (384, 384, 8192, 32, 32, True, False, True): (1, 32, 3, 2),
        (384, 384, 8192, 64, 64, False, True, True): (1, 128, 1, 4),
        (384, 384, 8192, 64, 64, True, False, True): (3, 64, 3, 4),
        (384, 384, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (384, 384, 16384, 16, 16, True, False, True): (4, 128, 3, 2),
        (384, 384, 16384, 32, 32, False, True, True): (3, 128, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2659-2694 / 第 2659-2694 行
````python
        (384, 384, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (384, 384, 16384, 64, 64, False, True, True): (3, 256, 1, 4),
        (384, 384, 16384, 64, 64, True, False, True): (2, 128, 3, 4),
        (384, 384, 32768, 16, 16, False, True, True): (1, 256, 1, 2),
        (384, 384, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (384, 384, 32768, 32, 32, False, True, True): (1, 256, 1, 2),
        (384, 384, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (384, 384, 32768, 64, 64, False, True, True): (2, 256, 1, 4),
        (384, 384, 32768, 64, 64, True, False, True): (1, 256, 3, 4),
        (384, 384, 65536, 16, 16, False, True, True): (4, 512, 1, 2),
        (384, 384, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (384, 384, 65536, 32, 32, False, True, True): (1, 512, 1, 2),
        (384, 384, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (384, 384, 65536, 64, 64, False, True, True): (3, 512, 1, 4),
        (384, 384, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (384, 384, 131072, 16, 16, False, True, True): (1, 512, 1, 1),
        (384, 384, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (384, 384, 131072, 32, 32, False, True, True): (1, 512, 1, 4),
        (384, 384, 131072, 32, 32, True, False, True): (1, 512, 3, 4),
        (384, 384, 131072, 64, 64, False, True, True): (3, 1024, 1, 4),
        (384, 384, 131072, 64, 64, True, False, True): (3, 512, 3, 4),
        (512, 512, 256, 16, 16, False, True, True): (2, 4, 5, 4),
        (512, 512, 256, 16, 16, True, False, True): (3, 4, 5, 4),
        (512, 512, 256, 32, 32, False, True, True): (1, 4, 5, 2),
        (512, 512, 256, 32, 32, True, False, True): (4, 8, 5, 1),
        (512, 512, 256, 64, 64, False, True, True): (4, 4, 5, 4),
        (512, 512, 256, 64, 64, True, False, True): (5, 4, 5, 4),
        (512, 512, 256, 128, 128, False, True, True): (3, 2, 2, 8),
        (512, 512, 256, 128, 128, True, False, True): (2, 2, 2, 8),
        (512, 512, 512, 16, 16, False, True, True): (1, 8, 5, 4),
        (512, 512, 512, 16, 16, True, False, True): (4, 8, 5, 2),
        (512, 512, 512, 32, 32, False, True, True): (1, 16, 4, 1),
        (512, 512, 512, 32, 32, True, False, True): (1, 8, 5, 2),
        (512, 512, 512, 64, 64, False, True, True): (4, 8, 5, 4),
        (512, 512, 512, 64, 64, True, False, True): (2, 8, 5, 4),
        (512, 512, 512, 128, 128, False, True, True): (2, 4, 2, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2695-2730 / 第 2695-2730 行
````python
        (512, 512, 512, 128, 128, True, False, True): (1, 4, 2, 8),
        (512, 512, 1024, 16, 16, False, True, True): (2, 8, 4, 4),
        (512, 512, 1024, 16, 16, True, False, True): (1, 8, 4, 4),
        (512, 512, 1024, 32, 32, False, True, True): (3, 16, 4, 2),
        (512, 512, 1024, 32, 32, True, False, True): (1, 16, 5, 2),
        (512, 512, 1024, 64, 64, False, True, True): (2, 8, 3, 4),
        (512, 512, 1024, 64, 64, True, False, True): (2, 16, 3, 4),
        (512, 512, 1024, 128, 128, False, True, True): (2, 8, 2, 8),
        (512, 512, 1024, 128, 128, True, False, True): (3, 8, 2, 8),
        (512, 512, 2048, 16, 16, False, True, True): (4, 16, 3, 2),
        (512, 512, 2048, 16, 16, True, False, True): (1, 16, 4, 2),
        (512, 512, 2048, 32, 32, False, True, True): (3, 32, 3, 2),
        (512, 512, 2048, 32, 32, True, False, True): (2, 32, 3, 2),
        (512, 512, 2048, 64, 64, False, True, True): (6, 32, 3, 2),
        (512, 512, 2048, 64, 64, True, False, True): (1, 32, 3, 2),
        (512, 512, 2048, 128, 128, False, True, True): (4, 16, 2, 8),
        (512, 512, 2048, 128, 128, True, False, True): (1, 16, 2, 8),
        (512, 512, 4096, 16, 16, False, True, True): (1, 16, 3, 2),
        (512, 512, 4096, 16, 16, True, False, True): (4, 32, 3, 2),
        (512, 512, 4096, 32, 32, False, True, True): (3, 32, 3, 2),
        (512, 512, 4096, 32, 32, True, False, True): (2, 32, 3, 2),
        (512, 512, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (512, 512, 4096, 64, 64, True, False, True): (1, 64, 3, 4),
        (512, 512, 4096, 128, 128, False, True, True): (4, 32, 1, 4),
        (512, 512, 4096, 128, 128, True, False, True): (4, 32, 2, 8),
        (512, 512, 8192, 16, 16, False, True, True): (8, 64, 3, 2),
        (512, 512, 8192, 16, 16, True, False, True): (4, 64, 3, 2),
        (512, 512, 8192, 32, 32, False, True, True): (3, 64, 3, 2),
        (512, 512, 8192, 32, 32, True, False, True): (3, 64, 3, 2),
        (512, 512, 8192, 64, 64, False, True, True): (1, 64, 3, 4),
        (512, 512, 8192, 64, 64, True, False, True): (7, 64, 3, 4),
        (512, 512, 8192, 128, 128, False, True, True): (1, 64, 1, 4),
        (512, 512, 8192, 128, 128, True, False, True): (4, 64, 2, 8),
        (512, 512, 16384, 16, 16, False, True, True): (1, 64, 3, 2),
        (512, 512, 16384, 16, 16, True, False, True): (1, 128, 3, 2),
        (512, 512, 16384, 32, 32, False, True, True): (3, 128, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2731-2766 / 第 2731-2766 行
````python
        (512, 512, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (512, 512, 16384, 64, 64, False, True, True): (4, 64, 2, 4),
        (512, 512, 16384, 64, 64, True, False, True): (2, 64, 2, 4),
        (512, 512, 16384, 128, 128, False, True, True): (4, 128, 1, 4),
        (512, 512, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (512, 512, 32768, 16, 16, False, True, True): (1, 128, 3, 2),
        (512, 512, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (512, 512, 32768, 32, 32, False, True, True): (1, 256, 3, 2),
        (512, 512, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (512, 512, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (512, 512, 32768, 64, 64, True, False, True): (2, 256, 3, 4),
        (512, 512, 32768, 128, 128, False, True, True): (5, 256, 1, 4),
        (512, 512, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (512, 512, 65536, 16, 16, False, True, True): (1, 256, 3, 2),
        (512, 512, 65536, 16, 16, True, False, True): (1, 256, 3, 1),
        (512, 512, 65536, 32, 32, False, True, True): (1, 512, 3, 2),
        (512, 512, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (512, 512, 65536, 64, 64, False, True, True): (4, 256, 2, 4),
        (512, 512, 65536, 64, 64, True, False, True): (2, 512, 3, 4),
        (512, 512, 65536, 128, 128, False, True, True): (6, 512, 1, 4),
        (512, 512, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (512, 512, 131072, 16, 16, False, True, True): (1, 512, 3, 2),
        (512, 512, 131072, 16, 16, True, False, True): (1, 512, 3, 1),
        (512, 512, 131072, 32, 32, False, True, True): (1, 1024, 3, 2),
        (512, 512, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (512, 512, 131072, 64, 64, False, True, True): (4, 512, 2, 4),
        (512, 512, 131072, 64, 64, True, False, True): (4, 1024, 3, 4),
        (512, 512, 131072, 128, 128, False, True, True): (6, 1024, 1, 4),
        (512, 512, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (768, 768, 256, 16, 16, False, True, True): (1, 8, 4, 1),
        (768, 768, 256, 16, 16, True, False, True): (3, 2, 6, 4),
        (768, 768, 256, 32, 32, False, True, True): (3, 8, 3, 4),
        (768, 768, 256, 32, 32, True, False, True): (1, 4, 4, 2),
        (768, 768, 256, 64, 64, False, True, True): (2, 4, 3, 4),
        (768, 768, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (768, 768, 256, 128, 128, False, True, True): (2, 2, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2767-2802 / 第 2767-2802 行
````python
        (768, 768, 256, 128, 128, True, False, True): (4, 2, 3, 8),
        (768, 768, 512, 16, 16, False, True, True): (4, 8, 4, 2),
        (768, 768, 512, 16, 16, True, False, True): (4, 8, 6, 2),
        (768, 768, 512, 32, 32, False, True, True): (1, 8, 4, 4),
        (768, 768, 512, 32, 32, True, False, True): (3, 8, 4, 2),
        (768, 768, 512, 64, 64, False, True, True): (1, 8, 3, 4),
        (768, 768, 512, 64, 64, True, False, True): (1, 8, 4, 4),
        (768, 768, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (768, 768, 512, 128, 128, True, False, True): (4, 4, 3, 8),
        (768, 768, 1024, 16, 16, False, True, True): (3, 16, 1, 4),
        (768, 768, 1024, 16, 16, True, False, True): (1, 8, 5, 2),
        (768, 768, 1024, 32, 32, False, True, True): (3, 16, 1, 8),
        (768, 768, 1024, 32, 32, True, False, True): (1, 16, 3, 2),
        (768, 768, 1024, 64, 64, False, True, True): (1, 8, 3, 4),
        (768, 768, 1024, 64, 64, True, False, True): (2, 8, 3, 8),
        (768, 768, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
        (768, 768, 1024, 128, 128, True, False, True): (1, 8, 3, 8),
        (768, 768, 2048, 16, 16, False, True, True): (2, 16, 1, 2),
        (768, 768, 2048, 16, 16, True, False, True): (1, 16, 3, 2),
        (768, 768, 2048, 32, 32, False, True, True): (5, 32, 1, 4),
        (768, 768, 2048, 32, 32, True, False, True): (3, 8, 3, 4),
        (768, 768, 2048, 64, 64, False, True, True): (1, 16, 1, 8),
        (768, 768, 2048, 64, 64, True, False, True): (3, 16, 3, 4),
        (768, 768, 2048, 128, 128, False, True, True): (2, 16, 3, 8),
        (768, 768, 2048, 128, 128, True, False, True): (1, 16, 3, 8),
        (768, 768, 4096, 16, 16, False, True, True): (3, 32, 1, 4),
        (768, 768, 4096, 16, 16, True, False, True): (2, 32, 3, 1),
        (768, 768, 4096, 32, 32, False, True, True): (2, 64, 1, 4),
        (768, 768, 4096, 32, 32, True, False, True): (1, 16, 4, 4),
        (768, 768, 4096, 64, 64, False, True, True): (3, 64, 3, 4),
        (768, 768, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (768, 768, 4096, 128, 128, False, True, True): (1, 32, 3, 8),
        (768, 768, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (768, 768, 8192, 16, 16, False, True, True): (1, 64, 1, 2),
        (768, 768, 8192, 16, 16, True, False, True): (4, 64, 3, 2),
        (768, 768, 8192, 32, 32, False, True, True): (1, 64, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2803-2838 / 第 2803-2838 行
````python
        (768, 768, 8192, 32, 32, True, False, True): (2, 32, 3, 4),
        (768, 768, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (768, 768, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (768, 768, 8192, 128, 128, False, True, True): (2, 64, 3, 8),
        (768, 768, 8192, 128, 128, True, False, True): (1, 64, 3, 8),
        (768, 768, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (768, 768, 16384, 16, 16, True, False, True): (1, 64, 4, 4),
        (768, 768, 16384, 32, 32, False, True, True): (1, 128, 1, 8),
        (768, 768, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (768, 768, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (768, 768, 16384, 64, 64, True, False, True): (1, 64, 3, 4),
        (768, 768, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (768, 768, 16384, 128, 128, True, False, True): (3, 128, 2, 4),
        (768, 768, 32768, 16, 16, False, True, True): (1, 256, 1, 2),
        (768, 768, 32768, 16, 16, True, False, True): (1, 128, 4, 4),
        (768, 768, 32768, 32, 32, False, True, True): (1, 128, 1, 2),
        (768, 768, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (768, 768, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (768, 768, 32768, 64, 64, True, False, True): (2, 128, 3, 4),
        (768, 768, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (768, 768, 32768, 128, 128, True, False, True): (2, 256, 2, 4),
        (768, 768, 65536, 16, 16, False, True, True): (4, 512, 1, 2),
        (768, 768, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (768, 768, 65536, 32, 32, False, True, True): (1, 256, 1, 2),
        (768, 768, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (768, 768, 65536, 64, 64, False, True, True): (3, 512, 1, 4),
        (768, 768, 65536, 64, 64, True, False, True): (2, 256, 3, 4),
        (768, 768, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (768, 768, 65536, 128, 128, True, False, True): (2, 512, 2, 4),
        (768, 768, 131072, 16, 16, False, True, True): (4, 1024, 1, 2),
        (768, 768, 131072, 16, 16, True, False, True): (1, 512, 4, 1),
        (768, 768, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
        (768, 768, 131072, 32, 32, True, False, True): (1, 512, 3, 4),
        (768, 768, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (768, 768, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (768, 768, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2839-2874 / 第 2839-2874 行
````python
        (768, 768, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
        (768, 3072, 256, 16, 16, False, True, True): (3, 8, 6, 1),
        (768, 3072, 256, 16, 16, True, False, True): (1, 4, 6, 2),
        (768, 3072, 256, 32, 32, False, True, True): (1, 8, 4, 4),
        (768, 3072, 256, 32, 32, True, False, True): (3, 4, 6, 4),
        (768, 3072, 256, 64, 64, False, True, True): (2, 4, 3, 4),
        (768, 3072, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (768, 3072, 256, 128, 128, False, True, True): (2, 2, 3, 8),
        (768, 3072, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (768, 3072, 512, 16, 16, False, True, True): (1, 8, 4, 2),
        (768, 3072, 512, 16, 16, True, False, True): (1, 8, 5, 2),
        (768, 3072, 512, 32, 32, False, True, True): (1, 16, 3, 2),
        (768, 3072, 512, 32, 32, True, False, True): (1, 8, 5, 2),
        (768, 3072, 512, 64, 64, False, True, True): (1, 8, 3, 4),
        (768, 3072, 512, 64, 64, True, False, True): (3, 8, 4, 4),
        (768, 3072, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (768, 3072, 512, 128, 128, True, False, True): (2, 4, 3, 8),
        (768, 3072, 1024, 16, 16, False, True, True): (1, 16, 1, 4),
        (768, 3072, 1024, 16, 16, True, False, True): (5, 4, 4, 4),
        (768, 3072, 1024, 32, 32, False, True, True): (3, 8, 3, 4),
        (768, 3072, 1024, 32, 32, True, False, True): (1, 8, 4, 4),
        (768, 3072, 1024, 64, 64, False, True, True): (2, 16, 3, 4),
        (768, 3072, 1024, 64, 64, True, False, True): (2, 16, 4, 4),
        (768, 3072, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
        (768, 3072, 1024, 128, 128, True, False, True): (5, 8, 3, 8),
        (768, 3072, 2048, 16, 16, False, True, True): (3, 16, 1, 2),
        (768, 3072, 2048, 16, 16, True, False, True): (1, 8, 3, 4),
        (768, 3072, 2048, 32, 32, False, True, True): (4, 16, 1, 8),
        (768, 3072, 2048, 32, 32, True, False, True): (3, 8, 3, 4),
        (768, 3072, 2048, 64, 64, False, True, True): (2, 16, 3, 4),
        (768, 3072, 2048, 64, 64, True, False, True): (2, 16, 3, 4),
        (768, 3072, 2048, 128, 128, False, True, True): (3, 16, 3, 8),
        (768, 3072, 2048, 128, 128, True, False, True): (4, 16, 3, 8),
        (768, 3072, 4096, 16, 16, False, True, True): (1, 32, 1, 4),
        (768, 3072, 4096, 16, 16, True, False, True): (1, 16, 3, 1),
        (768, 3072, 4096, 32, 32, False, True, True): (3, 32, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2875-2910 / 第 2875-2910 行
````python
        (768, 3072, 4096, 32, 32, True, False, True): (3, 16, 4, 4),
        (768, 3072, 4096, 64, 64, False, True, True): (2, 32, 3, 4),
        (768, 3072, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (768, 3072, 4096, 128, 128, False, True, True): (5, 32, 1, 4),
        (768, 3072, 4096, 128, 128, True, False, True): (9, 32, 3, 8),
        (768, 3072, 8192, 16, 16, False, True, True): (1, 32, 1, 4),
        (768, 3072, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (768, 3072, 8192, 32, 32, False, True, True): (1, 64, 1, 8),
        (768, 3072, 8192, 32, 32, True, False, True): (2, 64, 4, 2),
        (768, 3072, 8192, 64, 64, False, True, True): (1, 64, 3, 4),
        (768, 3072, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (768, 3072, 8192, 128, 128, False, True, True): (2, 64, 3, 8),
        (768, 3072, 8192, 128, 128, True, False, True): (2, 64, 3, 8),
        (768, 3072, 16384, 16, 16, False, True, True): (1, 64, 1, 4),
        (768, 3072, 16384, 16, 16, True, False, True): (1, 64, 4, 1),
        (768, 3072, 16384, 32, 32, False, True, True): (1, 128, 1, 8),
        (768, 3072, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (768, 3072, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (768, 3072, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (768, 3072, 16384, 128, 128, False, True, True): (2, 128, 3, 8),
        (768, 3072, 16384, 128, 128, True, False, True): (2, 128, 3, 8),
        (768, 3072, 32768, 16, 16, False, True, True): (1, 128, 1, 4),
        (768, 3072, 32768, 16, 16, True, False, True): (1, 128, 4, 1),
        (768, 3072, 32768, 32, 32, False, True, True): (1, 256, 1, 8),
        (768, 3072, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (768, 3072, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (768, 3072, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (768, 3072, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (768, 3072, 32768, 128, 128, True, False, True): (2, 256, 3, 8),
        (768, 3072, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (768, 3072, 50432, 16, 16, True, False, True): (4, 197, 4, 4),
        (768, 3072, 50432, 32, 32, False, True, True): (1, 197, 1, 4),
        (768, 3072, 50432, 32, 32, True, False, True): (4, 197, 3, 4),
        (768, 3072, 50432, 64, 64, False, True, True): (1, 394, 3, 4),
        (768, 3072, 50432, 64, 64, True, False, True): (3, 197, 3, 4),
        (768, 3072, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2911-2946 / 第 2911-2946 行
````python
        (768, 3072, 50432, 128, 128, True, False, True): (1, 394, 3, 8),
        (768, 3072, 65536, 16, 16, False, True, True): (1, 256, 1, 4),
        (768, 3072, 65536, 16, 16, True, False, True): (5, 256, 4, 1),
        (768, 3072, 65536, 32, 32, False, True, True): (1, 256, 1, 4),
        (768, 3072, 65536, 32, 32, True, False, True): (3, 256, 3, 4),
        (768, 3072, 65536, 64, 64, False, True, True): (2, 512, 3, 4),
        (768, 3072, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (768, 3072, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (768, 3072, 65536, 128, 128, True, False, True): (2, 512, 3, 8),
        (768, 3072, 131072, 16, 16, False, True, True): (1, 512, 1, 4),
        (768, 3072, 131072, 16, 16, True, False, True): (5, 512, 4, 1),
        (768, 3072, 131072, 32, 32, False, True, True): (1, 512, 1, 4),
        (768, 3072, 131072, 32, 32, True, False, True): (4, 512, 3, 4),
        (768, 3072, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (768, 3072, 131072, 64, 64, True, False, True): (1, 512, 3, 4),
        (768, 3072, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (768, 3072, 131072, 128, 128, True, False, True): (1, 1024, 3, 8),
        (1024, 1024, 256, 16, 16, False, True, True): (1, 4, 5, 4),
        (1024, 1024, 256, 16, 16, True, False, True): (3, 4, 4, 4),
        (1024, 1024, 256, 32, 32, False, True, True): (4, 4, 5, 2),
        (1024, 1024, 256, 32, 32, True, False, True): (3, 4, 5, 2),
        (1024, 1024, 256, 64, 64, False, True, True): (1, 4, 5, 4),
        (1024, 1024, 256, 64, 64, True, False, True): (1, 4, 5, 4),
        (1024, 1024, 256, 128, 128, False, True, True): (1, 2, 2, 8),
        (1024, 1024, 256, 128, 128, True, False, True): (2, 2, 2, 8),
        (1024, 1024, 512, 16, 16, False, True, True): (3, 4, 4, 4),
        (1024, 1024, 512, 16, 16, True, False, True): (4, 8, 5, 2),
        (1024, 1024, 512, 32, 32, False, True, True): (1, 8, 4, 2),
        (1024, 1024, 512, 32, 32, True, False, True): (1, 8, 4, 2),
        (1024, 1024, 512, 64, 64, False, True, True): (4, 8, 4, 4),
        (1024, 1024, 512, 64, 64, True, False, True): (2, 8, 3, 4),
        (1024, 1024, 512, 128, 128, False, True, True): (2, 4, 2, 8),
        (1024, 1024, 512, 128, 128, True, False, True): (1, 4, 2, 8),
        (1024, 1024, 1024, 16, 16, False, True, True): (3, 8, 4, 4),
        (1024, 1024, 1024, 16, 16, True, False, True): (4, 8, 4, 2),
        (1024, 1024, 1024, 32, 32, False, True, True): (1, 16, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2947-2982 / 第 2947-2982 行
````python
        (1024, 1024, 1024, 32, 32, True, False, True): (1, 16, 3, 2),
        (1024, 1024, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (1024, 1024, 1024, 64, 64, True, False, True): (3, 16, 3, 2),
        (1024, 1024, 1024, 128, 128, False, True, True): (1, 8, 2, 8),
        (1024, 1024, 1024, 128, 128, True, False, True): (2, 8, 2, 8),
        (1024, 1024, 2048, 16, 16, False, True, True): (3, 8, 3, 4),
        (1024, 1024, 2048, 16, 16, True, False, True): (3, 8, 3, 2),
        (1024, 1024, 2048, 32, 32, False, True, True): (5, 16, 3, 4),
        (1024, 1024, 2048, 32, 32, True, False, True): (1, 16, 3, 2),
        (1024, 1024, 2048, 64, 64, False, True, True): (6, 16, 4, 4),
        (1024, 1024, 2048, 64, 64, True, False, True): (5, 16, 3, 4),
        (1024, 1024, 2048, 128, 128, False, True, True): (4, 16, 2, 8),
        (1024, 1024, 2048, 128, 128, True, False, True): (4, 16, 2, 8),
        (1024, 1024, 4096, 16, 16, False, True, True): (8, 32, 3, 2),
        (1024, 1024, 4096, 16, 16, True, False, True): (4, 32, 3, 2),
        (1024, 1024, 4096, 32, 32, False, True, True): (2, 32, 3, 4),
        (1024, 1024, 4096, 32, 32, True, False, True): (3, 32, 3, 2),
        (1024, 1024, 4096, 64, 64, False, True, True): (3, 32, 3, 4),
        (1024, 1024, 4096, 64, 64, True, False, True): (1, 32, 3, 4),
        (1024, 1024, 4096, 128, 128, False, True, True): (4, 32, 2, 8),
        (1024, 1024, 4096, 128, 128, True, False, True): (1, 32, 2, 8),
        (1024, 1024, 8192, 16, 16, False, True, True): (4, 64, 3, 2),
        (1024, 1024, 8192, 16, 16, True, False, True): (4, 64, 3, 2),
        (1024, 1024, 8192, 32, 32, False, True, True): (8, 64, 3, 4),
        (1024, 1024, 8192, 32, 32, True, False, True): (4, 32, 3, 4),
        (1024, 1024, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (1024, 1024, 8192, 64, 64, True, False, True): (2, 64, 3, 4),
        (1024, 1024, 8192, 128, 128, False, True, True): (4, 64, 2, 8),
        (1024, 1024, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (1024, 1024, 16384, 16, 16, False, True, True): (1, 64, 3, 2),
        (1024, 1024, 16384, 16, 16, True, False, True): (1, 64, 3, 2),
        (1024, 1024, 16384, 32, 32, False, True, True): (1, 128, 3, 2),
        (1024, 1024, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (1024, 1024, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (1024, 1024, 16384, 64, 64, True, False, True): (1, 128, 3, 4),
        (1024, 1024, 16384, 128, 128, False, True, True): (2, 128, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 2983-3018 / 第 2983-3018 行
````python
        (1024, 1024, 16384, 128, 128, True, False, True): (4, 128, 1, 4),
        (1024, 1024, 32768, 16, 16, False, True, True): (1, 128, 3, 2),
        (1024, 1024, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (1024, 1024, 32768, 32, 32, False, True, True): (1, 256, 3, 2),
        (1024, 1024, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (1024, 1024, 32768, 64, 64, False, True, True): (2, 128, 2, 4),
        (1024, 1024, 32768, 64, 64, True, False, True): (1, 256, 3, 4),
        (1024, 1024, 32768, 128, 128, False, True, True): (2, 256, 1, 4),
        (1024, 1024, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (1024, 1024, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (1024, 1024, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (1024, 1024, 65536, 32, 32, False, True, True): (9, 256, 3, 4),
        (1024, 1024, 65536, 32, 32, True, False, True): (7, 256, 3, 4),
        (1024, 1024, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (1024, 1024, 65536, 64, 64, True, False, True): (2, 512, 3, 4),
        (1024, 1024, 65536, 128, 128, False, True, True): (2, 512, 1, 4),
        (1024, 1024, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (1024, 1024, 131072, 16, 16, False, True, True): (11, 512, 3, 2),
        (1024, 1024, 131072, 16, 16, True, False, True): (11, 512, 3, 2),
        (1024, 1024, 131072, 32, 32, False, True, True): (4, 512, 3, 4),
        (1024, 1024, 131072, 32, 32, True, False, True): (6, 512, 3, 4),
        (1024, 1024, 131072, 64, 64, False, True, True): (2, 512, 2, 4),
        (1024, 1024, 131072, 64, 64, True, False, True): (2, 1024, 3, 4),
        (1024, 1024, 131072, 128, 128, False, True, True): (4, 1024, 1, 4),
        (1024, 1024, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (1280, 5120, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (1280, 5120, 65792, 16, 16, True, False, True): (5, 257, 4, 1),
        (1280, 5120, 65792, 32, 32, False, True, True): (1, 514, 1, 8),
        (1280, 5120, 65792, 32, 32, True, False, True): (2, 257, 3, 4),
        (1280, 5120, 65792, 64, 64, False, True, True): (1, 514, 3, 4),
        (1280, 5120, 65792, 64, 64, True, False, True): (1, 257, 3, 4),
        (1280, 5120, 65792, 128, 128, False, True, True): (1, 514, 3, 8),
        (1280, 5120, 65792, 128, 128, True, False, True): (2, 514, 3, 8),
        (1536, 1536, 256, 16, 16, False, True, True): (1, 4, 6, 2),
        (1536, 1536, 256, 16, 16, True, False, True): (3, 4, 5, 2),
        (1536, 1536, 256, 32, 32, False, True, True): (2, 4, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3019-3054 / 第 3019-3054 行
````python
        (1536, 1536, 256, 32, 32, True, False, True): (1, 4, 5, 2),
        (1536, 1536, 256, 64, 64, False, True, True): (2, 4, 3, 4),
        (1536, 1536, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (1536, 1536, 256, 128, 128, False, True, True): (3, 2, 3, 8),
        (1536, 1536, 256, 128, 128, True, False, True): (6, 2, 3, 8),
        (1536, 1536, 512, 16, 16, False, True, True): (1, 8, 1, 4),
        (1536, 1536, 512, 16, 16, True, False, True): (3, 4, 5, 2),
        (1536, 1536, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (1536, 1536, 512, 32, 32, True, False, True): (1, 4, 4, 4),
        (1536, 1536, 512, 64, 64, False, True, True): (3, 8, 5, 4),
        (1536, 1536, 512, 64, 64, True, False, True): (3, 8, 3, 4),
        (1536, 1536, 512, 128, 128, False, True, True): (2, 4, 3, 8),
        (1536, 1536, 512, 128, 128, True, False, True): (3, 4, 3, 8),
        (1536, 1536, 1024, 16, 16, False, True, True): (1, 8, 1, 2),
        (1536, 1536, 1024, 16, 16, True, False, True): (2, 8, 4, 2),
        (1536, 1536, 1024, 32, 32, False, True, True): (8, 16, 1, 4),
        (1536, 1536, 1024, 32, 32, True, False, True): (3, 8, 4, 2),
        (1536, 1536, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (1536, 1536, 1024, 64, 64, True, False, True): (3, 8, 3, 4),
        (1536, 1536, 1024, 128, 128, False, True, True): (3, 8, 3, 8),
        (1536, 1536, 1024, 128, 128, True, False, True): (3, 8, 3, 8),
        (1536, 1536, 2048, 16, 16, False, True, True): (1, 16, 1, 4),
        (1536, 1536, 2048, 16, 16, True, False, True): (1, 8, 3, 1),
        (1536, 1536, 2048, 32, 32, False, True, True): (3, 16, 1, 8),
        (1536, 1536, 2048, 32, 32, True, False, True): (3, 8, 4, 4),
        (1536, 1536, 2048, 64, 64, False, True, True): (1, 16, 3, 4),
        (1536, 1536, 2048, 64, 64, True, False, True): (3, 8, 3, 4),
        (1536, 1536, 2048, 128, 128, False, True, True): (4, 16, 1, 4),
        (1536, 1536, 2048, 128, 128, True, False, True): (6, 16, 3, 8),
        (1536, 1536, 4096, 16, 16, False, True, True): (1, 32, 1, 2),
        (1536, 1536, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (1536, 1536, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
        (1536, 1536, 4096, 32, 32, True, False, True): (5, 32, 4, 2),
        (1536, 1536, 4096, 64, 64, False, True, True): (2, 32, 3, 4),
        (1536, 1536, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (1536, 1536, 4096, 128, 128, False, True, True): (4, 32, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3055-3090 / 第 3055-3090 行
````python
        (1536, 1536, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (1536, 1536, 8192, 16, 16, False, True, True): (1, 64, 1, 2),
        (1536, 1536, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (1536, 1536, 8192, 32, 32, False, True, True): (2, 64, 1, 8),
        (1536, 1536, 8192, 32, 32, True, False, True): (2, 32, 3, 4),
        (1536, 1536, 8192, 64, 64, False, True, True): (1, 64, 3, 4),
        (1536, 1536, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (1536, 1536, 8192, 128, 128, False, True, True): (4, 64, 3, 8),
        (1536, 1536, 8192, 128, 128, True, False, True): (1, 64, 3, 8),
        (1536, 1536, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (1536, 1536, 16384, 16, 16, True, False, True): (1, 64, 4, 4),
        (1536, 1536, 16384, 32, 32, False, True, True): (1, 64, 1, 2),
        (1536, 1536, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (1536, 1536, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (1536, 1536, 16384, 64, 64, True, False, True): (1, 64, 3, 4),
        (1536, 1536, 16384, 128, 128, False, True, True): (1, 128, 1, 4),
        (1536, 1536, 16384, 128, 128, True, False, True): (1, 128, 2, 4),
        (1536, 1536, 32768, 16, 16, False, True, True): (1, 256, 1, 2),
        (1536, 1536, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (1536, 1536, 32768, 32, 32, False, True, True): (1, 128, 1, 2),
        (1536, 1536, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (1536, 1536, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 32768, 128, 128, False, True, True): (1, 256, 1, 4),
        (1536, 1536, 32768, 128, 128, True, False, True): (2, 256, 2, 4),
        (1536, 1536, 65536, 16, 16, False, True, True): (2, 512, 1, 2),
        (1536, 1536, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (1536, 1536, 65536, 32, 32, False, True, True): (1, 256, 1, 2),
        (1536, 1536, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (1536, 1536, 65536, 64, 64, False, True, True): (1, 512, 3, 4),
        (1536, 1536, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (1536, 1536, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (1536, 1536, 65536, 128, 128, True, False, True): (4, 512, 2, 4),
        (1536, 1536, 131072, 16, 16, False, True, True): (2, 1024, 1, 2),
        (1536, 1536, 131072, 16, 16, True, False, True): (9, 512, 4, 4),
        (1536, 1536, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3091-3126 / 第 3091-3126 行
````python
        (1536, 1536, 131072, 32, 32, True, False, True): (5, 512, 3, 4),
        (1536, 1536, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (1536, 1536, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (1536, 1536, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (1536, 1536, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
        (2048, 2048, 256, 16, 16, False, True, True): (1, 4, 5, 2),
        (2048, 2048, 256, 16, 16, True, False, True): (4, 4, 5, 2),
        (2048, 2048, 256, 32, 32, False, True, True): (3, 4, 6, 2),
        (2048, 2048, 256, 32, 32, True, False, True): (2, 4, 5, 2),
        (2048, 2048, 256, 64, 64, False, True, True): (2, 4, 4, 4),
        (2048, 2048, 256, 64, 64, True, False, True): (2, 4, 3, 4),
        (2048, 2048, 256, 128, 128, False, True, True): (3, 2, 2, 8),
        (2048, 2048, 256, 128, 128, True, False, True): (3, 2, 2, 8),
        (2048, 2048, 512, 16, 16, False, True, True): (3, 4, 4, 4),
        (2048, 2048, 512, 16, 16, True, False, True): (1, 4, 4, 4),
        (2048, 2048, 512, 32, 32, False, True, True): (1, 4, 3, 4),
        (2048, 2048, 512, 32, 32, True, False, True): (1, 4, 4, 2),
        (2048, 2048, 512, 64, 64, False, True, True): (1, 8, 3, 4),
        (2048, 2048, 512, 64, 64, True, False, True): (1, 8, 3, 4),
        (2048, 2048, 512, 128, 128, False, True, True): (3, 4, 2, 8),
        (2048, 2048, 512, 128, 128, True, False, True): (2, 4, 2, 8),
        (2048, 2048, 1024, 16, 16, False, True, True): (3, 4, 3, 4),
        (2048, 2048, 1024, 16, 16, True, False, True): (4, 8, 3, 2),
        (2048, 2048, 1024, 32, 32, False, True, True): (3, 8, 3, 4),
        (2048, 2048, 1024, 32, 32, True, False, True): (1, 8, 3, 2),
        (2048, 2048, 1024, 64, 64, False, True, True): (1, 8, 3, 4),
        (2048, 2048, 1024, 64, 64, True, False, True): (1, 8, 3, 4),
        (2048, 2048, 1024, 128, 128, False, True, True): (4, 8, 1, 4),
        (2048, 2048, 1024, 128, 128, True, False, True): (2, 8, 1, 4),
        (2048, 2048, 2048, 16, 16, False, True, True): (4, 16, 3, 2),
        (2048, 2048, 2048, 16, 16, True, False, True): (4, 16, 3, 2),
        (2048, 2048, 2048, 32, 32, False, True, True): (1, 16, 3, 2),
        (2048, 2048, 2048, 32, 32, True, False, True): (1, 16, 3, 2),
        (2048, 2048, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (2048, 2048, 2048, 64, 64, True, False, True): (4, 16, 3, 4),
        (2048, 2048, 2048, 128, 128, False, True, True): (6, 16, 2, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3127-3162 / 第 3127-3162 行
````python
        (2048, 2048, 2048, 128, 128, True, False, True): (3, 16, 1, 4),
        (2048, 2048, 4096, 16, 16, False, True, True): (4, 32, 4, 2),
        (2048, 2048, 4096, 16, 16, True, False, True): (4, 32, 3, 2),
        (2048, 2048, 4096, 32, 32, False, True, True): (4, 16, 3, 8),
        (2048, 2048, 4096, 32, 32, True, False, True): (4, 16, 3, 8),
        (2048, 2048, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (2048, 2048, 4096, 64, 64, True, False, True): (3, 32, 3, 4),
        (2048, 2048, 4096, 128, 128, False, True, True): (2, 32, 1, 4),
        (2048, 2048, 4096, 128, 128, True, False, True): (2, 32, 1, 4),
        (2048, 2048, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (2048, 2048, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (2048, 2048, 8192, 32, 32, False, True, True): (4, 32, 4, 8),
        (2048, 2048, 8192, 32, 32, True, False, True): (4, 32, 3, 8),
        (2048, 2048, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (2048, 2048, 8192, 64, 64, True, False, True): (4, 64, 3, 4),
        (2048, 2048, 8192, 128, 128, False, True, True): (2, 64, 1, 4),
        (2048, 2048, 8192, 128, 128, True, False, True): (2, 64, 1, 4),
        (2048, 2048, 16384, 16, 16, False, True, True): (4, 64, 3, 2),
        (2048, 2048, 16384, 16, 16, True, False, True): (1, 64, 3, 2),
        (2048, 2048, 16384, 32, 32, False, True, True): (4, 64, 3, 4),
        (2048, 2048, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (2048, 2048, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (2048, 2048, 16384, 64, 64, True, False, True): (4, 128, 3, 4),
        (2048, 2048, 16384, 128, 128, False, True, True): (2, 128, 1, 4),
        (2048, 2048, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (2048, 2048, 32768, 16, 16, False, True, True): (8, 128, 3, 2),
        (2048, 2048, 32768, 16, 16, True, False, True): (8, 128, 3, 4),
        (2048, 2048, 32768, 32, 32, False, True, True): (8, 128, 3, 4),
        (2048, 2048, 32768, 32, 32, True, False, True): (8, 128, 3, 4),
        (2048, 2048, 32768, 64, 64, False, True, True): (1, 128, 2, 4),
        (2048, 2048, 32768, 64, 64, True, False, True): (8, 256, 3, 4),
        (2048, 2048, 32768, 128, 128, False, True, True): (2, 256, 1, 4),
        (2048, 2048, 32768, 128, 128, True, False, True): (2, 256, 1, 4),
        (2048, 2048, 65536, 16, 16, False, True, True): (9, 256, 4, 4),
        (2048, 2048, 65536, 16, 16, True, False, True): (7, 256, 4, 4),
        (2048, 2048, 65536, 32, 32, False, True, True): (7, 256, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3163-3198 / 第 3163-3198 行
````python
        (2048, 2048, 65536, 32, 32, True, False, True): (3, 256, 3, 4),
        (2048, 2048, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (2048, 2048, 65536, 64, 64, True, False, True): (6, 512, 3, 4),
        (2048, 2048, 65536, 128, 128, False, True, True): (2, 512, 1, 4),
        (2048, 2048, 65536, 128, 128, True, False, True): (2, 512, 1, 4),
        (2048, 2048, 131072, 16, 16, False, True, True): (9, 512, 4, 4),
        (2048, 2048, 131072, 16, 16, True, False, True): (9, 512, 4, 4),
        (2048, 2048, 131072, 32, 32, False, True, True): (7, 512, 4, 4),
        (2048, 2048, 131072, 32, 32, True, False, True): (3, 512, 3, 4),
        (2048, 2048, 131072, 64, 64, False, True, True): (2, 512, 2, 4),
        (2048, 2048, 131072, 64, 64, True, False, True): (4, 1024, 3, 4),
        (2048, 2048, 131072, 128, 128, False, True, True): (1, 1024, 1, 4),
        (2048, 2048, 131072, 128, 128, True, False, True): (2, 1024, 1, 4),
        (3072, 768, 256, 16, 16, False, True, True): (6, 4, 1, 4),
        (3072, 768, 256, 16, 16, True, False, True): (3, 1, 4, 4),
        (3072, 768, 256, 32, 32, False, True, True): (6, 8, 1, 2),
        (3072, 768, 256, 32, 32, True, False, True): (1, 2, 4, 4),
        (3072, 768, 256, 64, 64, False, True, True): (1, 4, 4, 4),
        (3072, 768, 256, 64, 64, True, False, True): (4, 2, 4, 4),
        (3072, 768, 256, 128, 128, False, True, True): (1, 2, 3, 8),
        (3072, 768, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (3072, 768, 512, 16, 16, False, True, True): (2, 4, 1, 4),
        (3072, 768, 512, 16, 16, True, False, True): (1, 4, 4, 1),
        (3072, 768, 512, 32, 32, False, True, True): (3, 8, 1, 4),
        (3072, 768, 512, 32, 32, True, False, True): (1, 2, 3, 4),
        (3072, 768, 512, 64, 64, False, True, True): (1, 8, 1, 4),
        (3072, 768, 512, 64, 64, True, False, True): (4, 4, 3, 4),
        (3072, 768, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (3072, 768, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (3072, 768, 1024, 16, 16, False, True, True): (1, 8, 1, 4),
        (3072, 768, 1024, 16, 16, True, False, True): (3, 4, 3, 1),
        (3072, 768, 1024, 32, 32, False, True, True): (1, 8, 1, 8),
        (3072, 768, 1024, 32, 32, True, False, True): (1, 4, 4, 4),
        (3072, 768, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (3072, 768, 1024, 64, 64, True, False, True): (1, 4, 3, 4),
        (3072, 768, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3199-3234 / 第 3199-3234 行
````python
        (3072, 768, 1024, 128, 128, True, False, True): (2, 8, 3, 8),
        (3072, 768, 2048, 16, 16, False, True, True): (3, 8, 1, 4),
        (3072, 768, 2048, 16, 16, True, False, True): (2, 8, 3, 4),
        (3072, 768, 2048, 32, 32, False, True, True): (3, 16, 1, 8),
        (3072, 768, 2048, 32, 32, True, False, True): (3, 8, 3, 4),
        (3072, 768, 2048, 64, 64, False, True, True): (1, 16, 1, 4),
        (3072, 768, 2048, 64, 64, True, False, True): (1, 16, 3, 4),
        (3072, 768, 2048, 128, 128, False, True, True): (1, 16, 3, 8),
        (3072, 768, 2048, 128, 128, True, False, True): (2, 16, 2, 4),
        (3072, 768, 4096, 16, 16, False, True, True): (1, 16, 1, 4),
        (3072, 768, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (3072, 768, 4096, 32, 32, False, True, True): (2, 32, 1, 8),
        (3072, 768, 4096, 32, 32, True, False, True): (7, 16, 3, 4),
        (3072, 768, 4096, 64, 64, False, True, True): (2, 32, 1, 4),
        (3072, 768, 4096, 64, 64, True, False, True): (2, 16, 2, 4),
        (3072, 768, 4096, 128, 128, False, True, True): (1, 32, 3, 8),
        (3072, 768, 4096, 128, 128, True, False, True): (3, 32, 2, 4),
        (3072, 768, 8192, 16, 16, False, True, True): (2, 32, 1, 4),
        (3072, 768, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (3072, 768, 8192, 32, 32, False, True, True): (4, 32, 1, 4),
        (3072, 768, 8192, 32, 32, True, False, True): (4, 32, 3, 4),
        (3072, 768, 8192, 64, 64, False, True, True): (2, 64, 1, 4),
        (3072, 768, 8192, 64, 64, True, False, True): (4, 32, 2, 4),
        (3072, 768, 8192, 128, 128, False, True, True): (3, 64, 1, 4),
        (3072, 768, 8192, 128, 128, True, False, True): (6, 64, 2, 4),
        (3072, 768, 16384, 16, 16, False, True, True): (1, 64, 1, 4),
        (3072, 768, 16384, 16, 16, True, False, True): (1, 64, 1, 1),
        (3072, 768, 16384, 32, 32, False, True, True): (1, 64, 1, 4),
        (3072, 768, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (3072, 768, 16384, 64, 64, False, True, True): (4, 128, 1, 4),
        (3072, 768, 16384, 64, 64, True, False, True): (4, 64, 2, 4),
        (3072, 768, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (3072, 768, 16384, 128, 128, True, False, True): (4, 128, 2, 4),
        (3072, 768, 32768, 16, 16, False, True, True): (1, 128, 1, 4),
        (3072, 768, 32768, 16, 16, True, False, True): (8, 128, 4, 1),
        (3072, 768, 32768, 32, 32, False, True, True): (1, 128, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3235-3270 / 第 3235-3270 行
````python
        (3072, 768, 32768, 32, 32, True, False, True): (8, 128, 3, 4),
        (3072, 768, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (3072, 768, 32768, 64, 64, True, False, True): (1, 128, 2, 4),
        (3072, 768, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (3072, 768, 32768, 128, 128, True, False, True): (8, 256, 2, 4),
        (3072, 768, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (3072, 768, 50432, 16, 16, True, False, True): (7, 197, 4, 1),
        (3072, 768, 50432, 32, 32, False, True, True): (1, 197, 1, 4),
        (3072, 768, 50432, 32, 32, True, False, True): (4, 197, 3, 4),
        (3072, 768, 50432, 64, 64, False, True, True): (1, 394, 1, 4),
        (3072, 768, 50432, 64, 64, True, False, True): (3, 197, 2, 4),
        (3072, 768, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (3072, 768, 50432, 128, 128, True, False, True): (8, 394, 2, 4),
        (3072, 768, 65536, 16, 16, False, True, True): (1, 256, 1, 4),
        (3072, 768, 65536, 16, 16, True, False, True): (15, 256, 4, 1),
        (3072, 768, 65536, 32, 32, False, True, True): (1, 256, 1, 4),
        (3072, 768, 65536, 32, 32, True, False, True): (15, 256, 3, 4),
        (3072, 768, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (3072, 768, 65536, 64, 64, True, False, True): (2, 256, 2, 4),
        (3072, 768, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (3072, 768, 65536, 128, 128, True, False, True): (3, 512, 2, 4),
        (3072, 768, 131072, 16, 16, False, True, True): (1, 512, 1, 4),
        (3072, 768, 131072, 16, 16, True, False, True): (15, 512, 4, 1),
        (3072, 768, 131072, 32, 32, False, True, True): (1, 512, 1, 4),
        (3072, 768, 131072, 32, 32, True, False, True): (9, 512, 3, 4),
        (3072, 768, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (3072, 768, 131072, 64, 64, True, False, True): (3, 512, 2, 4),
        (3072, 768, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (3072, 768, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
        (3072, 3072, 256, 16, 16, False, True, True): (5, 4, 1, 4),
        (3072, 3072, 256, 16, 16, True, False, True): (1, 2, 5, 2),
        (3072, 3072, 256, 32, 32, False, True, True): (5, 4, 1, 8),
        (3072, 3072, 256, 32, 32, True, False, True): (1, 4, 4, 2),
        (3072, 3072, 256, 64, 64, False, True, True): (2, 4, 4, 4),
        (3072, 3072, 256, 64, 64, True, False, True): (2, 4, 4, 4),
        (3072, 3072, 256, 128, 128, False, True, True): (1, 2, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3271-3306 / 第 3271-3306 行
````python
        (3072, 3072, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (3072, 3072, 512, 16, 16, False, True, True): (5, 4, 1, 2),
        (3072, 3072, 512, 16, 16, True, False, True): (1, 2, 3, 4),
        (3072, 3072, 512, 32, 32, False, True, True): (3, 8, 1, 4),
        (3072, 3072, 512, 32, 32, True, False, True): (1, 4, 4, 2),
        (3072, 3072, 512, 64, 64, False, True, True): (1, 8, 2, 2),
        (3072, 3072, 512, 64, 64, True, False, True): (2, 4, 3, 4),
        (3072, 3072, 512, 128, 128, False, True, True): (2, 4, 3, 8),
        (3072, 3072, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (3072, 3072, 1024, 16, 16, False, True, True): (1, 8, 1, 4),
        (3072, 3072, 1024, 16, 16, True, False, True): (2, 8, 3, 1),
        (3072, 3072, 1024, 32, 32, False, True, True): (1, 16, 1, 4),
        (3072, 3072, 1024, 32, 32, True, False, True): (1, 4, 4, 4),
        (3072, 3072, 1024, 64, 64, False, True, True): (1, 8, 3, 4),
        (3072, 3072, 1024, 64, 64, True, False, True): (2, 4, 3, 4),
        (3072, 3072, 1024, 128, 128, False, True, True): (1, 8, 1, 4),
        (3072, 3072, 1024, 128, 128, True, False, True): (2, 8, 3, 8),
        (3072, 3072, 2048, 16, 16, False, True, True): (1, 16, 1, 2),
        (3072, 3072, 2048, 16, 16, True, False, True): (2, 16, 4, 2),
        (3072, 3072, 2048, 32, 32, False, True, True): (1, 16, 1, 8),
        (3072, 3072, 2048, 32, 32, True, False, True): (3, 8, 4, 4),
        (3072, 3072, 2048, 64, 64, False, True, True): (3, 16, 3, 4),
        (3072, 3072, 2048, 64, 64, True, False, True): (3, 8, 3, 4),
        (3072, 3072, 2048, 128, 128, False, True, True): (1, 16, 3, 8),
        (3072, 3072, 2048, 128, 128, True, False, True): (5, 16, 3, 8),
        (3072, 3072, 4096, 16, 16, False, True, True): (1, 32, 1, 2),
        (3072, 3072, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (3072, 3072, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
        (3072, 3072, 4096, 32, 32, True, False, True): (3, 16, 3, 4),
        (3072, 3072, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (3072, 3072, 4096, 64, 64, True, False, True): (3, 16, 3, 4),
        (3072, 3072, 4096, 128, 128, False, True, True): (3, 32, 3, 8),
        (3072, 3072, 4096, 128, 128, True, False, True): (3, 32, 3, 8),
        (3072, 3072, 8192, 16, 16, False, True, True): (1, 64, 1, 2),
        (3072, 3072, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (3072, 3072, 8192, 32, 32, False, True, True): (1, 64, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3307-3342 / 第 3307-3342 行
````python
        (3072, 3072, 8192, 32, 32, True, False, True): (6, 32, 3, 4),
        (3072, 3072, 8192, 64, 64, False, True, True): (1, 64, 3, 4),
        (3072, 3072, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (3072, 3072, 8192, 128, 128, False, True, True): (2, 64, 3, 8),
        (3072, 3072, 8192, 128, 128, True, False, True): (1, 64, 3, 8),
        (3072, 3072, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (3072, 3072, 16384, 16, 16, True, False, True): (4, 128, 4, 2),
        (3072, 3072, 16384, 32, 32, False, True, True): (1, 64, 1, 2),
        (3072, 3072, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (3072, 3072, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (3072, 3072, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (3072, 3072, 16384, 128, 128, False, True, True): (1, 128, 1, 4),
        (3072, 3072, 16384, 128, 128, True, False, True): (1, 128, 3, 8),
        (3072, 3072, 32768, 16, 16, False, True, True): (1, 256, 1, 2),
        (3072, 3072, 32768, 16, 16, True, False, True): (8, 128, 4, 4),
        (3072, 3072, 32768, 32, 32, False, True, True): (1, 256, 1, 8),
        (3072, 3072, 32768, 32, 32, True, False, True): (5, 128, 3, 4),
        (3072, 3072, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (3072, 3072, 32768, 64, 64, True, False, True): (3, 128, 3, 4),
        (3072, 3072, 32768, 128, 128, False, True, True): (1, 256, 1, 4),
        (3072, 3072, 32768, 128, 128, True, False, True): (3, 256, 2, 4),
        (3072, 3072, 65536, 16, 16, False, True, True): (1, 512, 1, 2),
        (3072, 3072, 65536, 16, 16, True, False, True): (7, 256, 4, 4),
        (3072, 3072, 65536, 32, 32, False, True, True): (1, 256, 1, 2),
        (3072, 3072, 65536, 32, 32, True, False, True): (5, 256, 3, 4),
        (3072, 3072, 65536, 64, 64, False, True, True): (1, 512, 3, 4),
        (3072, 3072, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (3072, 3072, 65536, 128, 128, False, True, True): (1, 512, 1, 4),
        (3072, 3072, 65536, 128, 128, True, False, True): (3, 512, 2, 4),
        (3072, 3072, 131072, 16, 16, False, True, True): (1, 1024, 1, 2),
        (3072, 3072, 131072, 16, 16, True, False, True): (5, 512, 4, 4),
        (3072, 3072, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
        (3072, 3072, 131072, 32, 32, True, False, True): (5, 512, 3, 4),
        (3072, 3072, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (3072, 3072, 131072, 64, 64, True, False, True): (3, 512, 3, 4),
        (3072, 3072, 131072, 128, 128, False, True, True): (1, 1024, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3343-3378 / 第 3343-3378 行
````python
        (3072, 3072, 131072, 128, 128, True, False, True): (6, 1024, 2, 4),
        (4096, 4096, 256, 16, 16, False, True, True): (2, 2, 5, 4),
        (4096, 4096, 256, 16, 16, True, False, True): (2, 2, 4, 2),
        (4096, 4096, 256, 32, 32, False, True, True): (1, 2, 4, 4),
        (4096, 4096, 256, 32, 32, True, False, True): (3, 2, 4, 2),
        (4096, 4096, 256, 64, 64, False, True, True): (3, 4, 3, 4),
        (4096, 4096, 256, 64, 64, True, False, True): (1, 4, 3, 2),
        (4096, 4096, 256, 128, 128, False, True, True): (1, 2, 2, 8),
        (4096, 4096, 256, 128, 128, True, False, True): (1, 2, 2, 8),
        (4096, 4096, 512, 16, 16, False, True, True): (4, 2, 3, 4),
        (4096, 4096, 512, 16, 16, True, False, True): (1, 2, 3, 4),
        (4096, 4096, 512, 32, 32, False, True, True): (1, 4, 3, 4),
        (4096, 4096, 512, 32, 32, True, False, True): (3, 4, 3, 2),
        (4096, 4096, 512, 64, 64, False, True, True): (4, 4, 4, 4),
        (4096, 4096, 512, 64, 64, True, False, True): (3, 4, 3, 4),
        (4096, 4096, 512, 128, 128, False, True, True): (2, 4, 2, 8),
        (4096, 4096, 512, 128, 128, True, False, True): (2, 4, 1, 4),
        (4096, 4096, 1024, 16, 16, False, True, True): (2, 8, 3, 2),
        (4096, 4096, 1024, 16, 16, True, False, True): (2, 8, 3, 2),
        (4096, 4096, 1024, 32, 32, False, True, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 32, 32, True, False, True): (1, 8, 3, 2),
        (4096, 4096, 1024, 64, 64, False, True, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 64, 64, True, False, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 128, 128, False, True, True): (4, 8, 1, 4),
        (4096, 4096, 1024, 128, 128, True, False, True): (2, 8, 2, 8),
        (4096, 4096, 2048, 16, 16, False, True, True): (2, 8, 4, 4),
        (4096, 4096, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (4096, 4096, 2048, 32, 32, False, True, True): (4, 8, 3, 8),
        (4096, 4096, 2048, 32, 32, True, False, True): (4, 8, 4, 8),
        (4096, 4096, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (4096, 4096, 2048, 64, 64, True, False, True): (4, 16, 3, 4),
        (4096, 4096, 2048, 128, 128, False, True, True): (1, 16, 1, 4),
        (4096, 4096, 2048, 128, 128, True, False, True): (4, 16, 1, 4),
        (4096, 4096, 4096, 16, 16, False, True, True): (4, 32, 4, 4),
        (4096, 4096, 4096, 16, 16, True, False, True): (2, 32, 4, 4),
        (4096, 4096, 4096, 32, 32, False, True, True): (4, 16, 4, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3379-3414 / 第 3379-3414 行
````python
        (4096, 4096, 4096, 32, 32, True, False, True): (4, 16, 4, 8),
        (4096, 4096, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (4096, 4096, 4096, 64, 64, True, False, True): (2, 32, 3, 4),
        (4096, 4096, 4096, 128, 128, False, True, True): (2, 32, 1, 4),
        (4096, 4096, 4096, 128, 128, True, False, True): (2, 32, 1, 4),
        (4096, 4096, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (4096, 4096, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (4096, 4096, 8192, 32, 32, False, True, True): (4, 32, 4, 8),
        (4096, 4096, 8192, 32, 32, True, False, True): (4, 32, 4, 8),
        (4096, 4096, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (4096, 4096, 8192, 64, 64, True, False, True): (4, 64, 3, 4),
        (4096, 4096, 8192, 128, 128, False, True, True): (1, 64, 1, 4),
        (4096, 4096, 8192, 128, 128, True, False, True): (1, 64, 1, 4),
        (4096, 4096, 16384, 16, 16, False, True, True): (4, 64, 4, 4),
        (4096, 4096, 16384, 16, 16, True, False, True): (4, 64, 4, 4),
        (4096, 4096, 16384, 32, 32, False, True, True): (4, 64, 4, 8),
        (4096, 4096, 16384, 32, 32, True, False, True): (4, 64, 4, 8),
        (4096, 4096, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (4096, 4096, 16384, 64, 64, True, False, True): (4, 128, 3, 4),
        (4096, 4096, 16384, 128, 128, False, True, True): (1, 128, 1, 4),
        (4096, 4096, 16384, 128, 128, True, False, True): (1, 128, 1, 4),
        (4096, 4096, 32768, 16, 16, False, True, True): (8, 128, 4, 4),
        (4096, 4096, 32768, 16, 16, True, False, True): (5, 128, 4, 4),
        (4096, 4096, 32768, 32, 32, False, True, True): (5, 128, 4, 4),
        (4096, 4096, 32768, 32, 32, True, False, True): (3, 128, 4, 8),
        (4096, 4096, 32768, 64, 64, False, True, True): (3, 256, 3, 4),
        (4096, 4096, 32768, 64, 64, True, False, True): (2, 256, 3, 4),
        (4096, 4096, 32768, 128, 128, False, True, True): (1, 256, 1, 4),
        (4096, 4096, 32768, 128, 128, True, False, True): (1, 256, 1, 4),
        (4096, 4096, 65536, 16, 16, False, True, True): (5, 256, 4, 4),
        (4096, 4096, 65536, 16, 16, True, False, True): (5, 256, 4, 4),
        (4096, 4096, 65536, 32, 32, False, True, True): (4, 256, 4, 8),
        (4096, 4096, 65536, 32, 32, True, False, True): (4, 256, 4, 8),
        (4096, 4096, 65536, 64, 64, False, True, True): (1, 512, 3, 4),
        (4096, 4096, 65536, 64, 64, True, False, True): (3, 512, 3, 4),
        (4096, 4096, 65536, 128, 128, False, True, True): (1, 512, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3415-3450 / 第 3415-3450 行
````python
        (4096, 4096, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (4096, 4096, 131072, 16, 16, False, True, True): (5, 512, 4, 4),
        (4096, 4096, 131072, 16, 16, True, False, True): (5, 512, 4, 4),
        (4096, 4096, 131072, 32, 32, False, True, True): (4, 512, 4, 4),
        (4096, 4096, 131072, 32, 32, True, False, True): (2, 512, 3, 4),
        (4096, 4096, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (4096, 4096, 131072, 64, 64, True, False, True): (3, 1024, 3, 4),
        (4096, 4096, 131072, 128, 128, False, True, True): (1, 1024, 1, 4),
        (4096, 4096, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (5120, 1280, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (5120, 1280, 65792, 16, 16, True, False, True): (11, 257, 4, 1),
        (5120, 1280, 65792, 32, 32, False, True, True): (1, 257, 1, 4),
        (5120, 1280, 65792, 32, 32, True, False, True): (5, 257, 3, 4),
        (5120, 1280, 65792, 64, 64, False, True, True): (1, 514, 1, 4),
        (5120, 1280, 65792, 64, 64, True, False, True): (5, 257, 2, 4),
        (5120, 1280, 65792, 128, 128, False, True, True): (3, 514, 1, 4),
        (5120, 1280, 65792, 128, 128, True, False, True): (7, 514, 2, 4),
        (6144, 6144, 256, 16, 16, False, True, True): (1, 2, 1, 4),
        (6144, 6144, 256, 16, 16, True, False, True): (3, 1, 4, 4),
        (6144, 6144, 256, 32, 32, False, True, True): (3, 2, 1, 8),
        (6144, 6144, 256, 32, 32, True, False, True): (1, 1, 4, 4),
        (6144, 6144, 256, 64, 64, False, True, True): (4, 2, 3, 4),
        (6144, 6144, 256, 64, 64, True, False, True): (3, 2, 4, 4),
        (6144, 6144, 256, 128, 128, False, True, True): (2, 2, 3, 8),
        (6144, 6144, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (6144, 6144, 512, 16, 16, False, True, True): (4, 4, 1, 4),
        (6144, 6144, 512, 16, 16, True, False, True): (3, 2, 3, 1),
        (6144, 6144, 512, 32, 32, False, True, True): (1, 8, 1, 4),
        (6144, 6144, 512, 32, 32, True, False, True): (1, 2, 3, 2),
        (6144, 6144, 512, 64, 64, False, True, True): (2, 4, 3, 4),
        (6144, 6144, 512, 64, 64, True, False, True): (2, 2, 3, 4),
        (6144, 6144, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (6144, 6144, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (6144, 6144, 1024, 16, 16, False, True, True): (1, 8, 1, 2),
        (6144, 6144, 1024, 16, 16, True, False, True): (4, 8, 4, 4),
        (6144, 6144, 1024, 32, 32, False, True, True): (1, 8, 4, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3451-3486 / 第 3451-3486 行
````python
        (6144, 6144, 1024, 32, 32, True, False, True): (1, 8, 4, 2),
        (6144, 6144, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (6144, 6144, 1024, 64, 64, True, False, True): (1, 4, 3, 4),
        (6144, 6144, 1024, 128, 128, False, True, True): (2, 8, 3, 8),
        (6144, 6144, 1024, 128, 128, True, False, True): (1, 8, 3, 8),
        (6144, 6144, 2048, 16, 16, False, True, True): (4, 4, 1, 4),
        (6144, 6144, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (6144, 6144, 2048, 32, 32, False, True, True): (1, 16, 4, 2),
        (6144, 6144, 2048, 32, 32, True, False, True): (4, 8, 4, 8),
        (6144, 6144, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (6144, 6144, 2048, 64, 64, True, False, True): (2, 8, 3, 4),
        (6144, 6144, 2048, 128, 128, False, True, True): (1, 16, 3, 8),
        (6144, 6144, 2048, 128, 128, True, False, True): (4, 16, 3, 8),
        (6144, 6144, 4096, 16, 16, False, True, True): (4, 8, 1, 4),
        (6144, 6144, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (6144, 6144, 4096, 32, 32, False, True, True): (4, 16, 1, 2),
        (6144, 6144, 4096, 32, 32, True, False, True): (2, 8, 3, 8),
        (6144, 6144, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (6144, 6144, 4096, 64, 64, True, False, True): (4, 16, 3, 4),
        (6144, 6144, 4096, 128, 128, False, True, True): (4, 32, 3, 8),
        (6144, 6144, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (6144, 6144, 8192, 16, 16, False, True, True): (2, 16, 1, 2),
        (6144, 6144, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (6144, 6144, 8192, 32, 32, False, True, True): (4, 32, 1, 2),
        (6144, 6144, 8192, 32, 32, True, False, True): (4, 32, 4, 8),
        (6144, 6144, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (6144, 6144, 8192, 64, 64, True, False, True): (4, 32, 3, 4),
        (6144, 6144, 8192, 128, 128, False, True, True): (4, 64, 3, 8),
        (6144, 6144, 8192, 128, 128, True, False, True): (4, 64, 3, 8),
        (6144, 6144, 16384, 16, 16, False, True, True): (2, 32, 1, 2),
        (6144, 6144, 16384, 16, 16, True, False, True): (4, 64, 4, 4),
        (6144, 6144, 16384, 32, 32, False, True, True): (4, 64, 1, 2),
        (6144, 6144, 16384, 32, 32, True, False, True): (4, 64, 3, 2),
        (6144, 6144, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (6144, 6144, 16384, 64, 64, True, False, True): (2, 32, 3, 8),
        (6144, 6144, 16384, 128, 128, False, True, True): (4, 128, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3487-3522 / 第 3487-3522 行
````python
        (6144, 6144, 16384, 128, 128, True, False, True): (4, 128, 3, 8),
        (6144, 6144, 32768, 16, 16, False, True, True): (2, 64, 1, 2),
        (6144, 6144, 32768, 16, 16, True, False, True): (3, 128, 4, 4),
        (6144, 6144, 32768, 32, 32, False, True, True): (4, 128, 1, 2),
        (6144, 6144, 32768, 32, 32, True, False, True): (3, 128, 3, 4),
        (6144, 6144, 32768, 64, 64, False, True, True): (4, 256, 3, 4),
        (6144, 6144, 32768, 64, 64, True, False, True): (2, 64, 3, 8),
        (6144, 6144, 32768, 128, 128, False, True, True): (4, 256, 3, 8),
        (6144, 6144, 32768, 128, 128, True, False, True): (4, 256, 3, 8),
        (6144, 6144, 65536, 16, 16, False, True, True): (2, 128, 1, 2),
        (6144, 6144, 65536, 16, 16, True, False, True): (4, 256, 4, 4),
        (6144, 6144, 65536, 32, 32, False, True, True): (4, 256, 1, 2),
        (6144, 6144, 65536, 32, 32, True, False, True): (4, 256, 3, 4),
        (6144, 6144, 65536, 64, 64, False, True, True): (4, 512, 3, 4),
        (6144, 6144, 65536, 64, 64, True, False, True): (2, 128, 3, 8),
        (6144, 6144, 65536, 128, 128, False, True, True): (4, 512, 3, 8),
        (6144, 6144, 65536, 128, 128, True, False, True): (4, 512, 3, 8),
        (6144, 6144, 131072, 16, 16, False, True, True): (2, 256, 1, 2),
        (6144, 6144, 131072, 16, 16, True, False, True): (5, 512, 4, 1),
        (6144, 6144, 131072, 32, 32, False, True, True): (4, 512, 1, 2),
        (6144, 6144, 131072, 32, 32, True, False, True): (4, 512, 3, 2),
        (6144, 6144, 131072, 64, 64, False, True, True): (4, 1024, 3, 4),
        (6144, 6144, 131072, 64, 64, True, False, True): (2, 256, 3, 8),
        (6144, 6144, 131072, 128, 128, False, True, True): (4, 1024, 3, 8),
        (6144, 6144, 131072, 128, 128, True, False, True): (4, 1024, 3, 8),
        (8192, 8192, 256, 16, 16, False, True, True): (1, 1, 3, 4),
        (8192, 8192, 256, 16, 16, True, False, True): (4, 1, 3, 4),
        (8192, 8192, 256, 32, 32, False, True, True): (1, 2, 3, 4),
        (8192, 8192, 256, 32, 32, True, False, True): (1, 2, 3, 4),
        (8192, 8192, 256, 64, 64, False, True, True): (6, 2, 3, 8),
        (8192, 8192, 256, 64, 64, True, False, True): (4, 2, 3, 8),
        (8192, 8192, 256, 128, 128, False, True, True): (1, 2, 1, 4),
        (8192, 8192, 256, 128, 128, True, False, True): (1, 2, 1, 4),
        (8192, 8192, 512, 16, 16, False, True, True): (4, 4, 3, 2),
        (8192, 8192, 512, 16, 16, True, False, True): (4, 4, 3, 4),
        (8192, 8192, 512, 32, 32, False, True, True): (1, 4, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3523-3558 / 第 3523-3558 行
````python
        (8192, 8192, 512, 32, 32, True, False, True): (3, 4, 3, 2),
        (8192, 8192, 512, 64, 64, False, True, True): (1, 4, 3, 4),
        (8192, 8192, 512, 64, 64, True, False, True): (1, 4, 3, 4),
        (8192, 8192, 512, 128, 128, False, True, True): (4, 4, 2, 8),
        (8192, 8192, 512, 128, 128, True, False, True): (4, 4, 2, 8),
        (8192, 8192, 1024, 16, 16, False, True, True): (4, 8, 4, 4),
        (8192, 8192, 1024, 16, 16, True, False, True): (2, 8, 4, 4),
        (8192, 8192, 1024, 32, 32, False, True, True): (2, 4, 4, 8),
        (8192, 8192, 1024, 32, 32, True, False, True): (1, 4, 3, 4),
        (8192, 8192, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (8192, 8192, 1024, 64, 64, True, False, True): (2, 8, 3, 4),
        (8192, 8192, 1024, 128, 128, False, True, True): (4, 8, 1, 4),
        (8192, 8192, 1024, 128, 128, True, False, True): (4, 8, 1, 4),
        (8192, 8192, 2048, 16, 16, False, True, True): (2, 8, 4, 4),
        (8192, 8192, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (8192, 8192, 2048, 32, 32, False, True, True): (2, 8, 4, 8),
        (8192, 8192, 2048, 32, 32, True, False, True): (2, 8, 4, 8),
        (8192, 8192, 2048, 64, 64, False, True, True): (4, 8, 2, 4),
        (8192, 8192, 2048, 64, 64, True, False, True): (4, 16, 3, 4),
        (8192, 8192, 2048, 128, 128, False, True, True): (4, 16, 1, 4),
        (8192, 8192, 2048, 128, 128, True, False, True): (4, 16, 1, 4),
        (8192, 8192, 4096, 16, 16, False, True, True): (4, 16, 4, 4),
        (8192, 8192, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (8192, 8192, 4096, 32, 32, False, True, True): (2, 16, 4, 8),
        (8192, 8192, 4096, 32, 32, True, False, True): (2, 16, 4, 8),
        (8192, 8192, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (8192, 8192, 4096, 64, 64, True, False, True): (4, 16, 2, 4),
        (8192, 8192, 4096, 128, 128, False, True, True): (4, 32, 1, 4),
        (8192, 8192, 4096, 128, 128, True, False, True): (4, 32, 1, 4),
        (8192, 8192, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (8192, 8192, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (8192, 8192, 8192, 32, 32, False, True, True): (2, 32, 4, 8),
        (8192, 8192, 8192, 32, 32, True, False, True): (2, 32, 4, 8),
        (8192, 8192, 8192, 64, 64, False, True, True): (4, 32, 3, 8),
        (8192, 8192, 8192, 64, 64, True, False, True): (4, 32, 2, 4),
        (8192, 8192, 8192, 128, 128, False, True, True): (4, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3559-3594 / 第 3559-3594 行
````python
        (8192, 8192, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (8192, 8192, 16384, 16, 16, False, True, True): (4, 64, 4, 4),
        (8192, 8192, 16384, 16, 16, True, False, True): (4, 64, 4, 4),
        (8192, 8192, 16384, 32, 32, False, True, True): (4, 64, 3, 4),
        (8192, 8192, 16384, 32, 32, True, False, True): (4, 64, 4, 8),
        (8192, 8192, 16384, 64, 64, False, True, True): (4, 64, 2, 4),
        (8192, 8192, 16384, 64, 64, True, False, True): (4, 64, 2, 4),
        (8192, 8192, 16384, 128, 128, False, True, True): (4, 128, 1, 4),
        (8192, 8192, 16384, 128, 128, True, False, True): (4, 128, 1, 4),
        (8192, 8192, 32768, 16, 16, False, True, True): (3, 128, 4, 4),
        (8192, 8192, 32768, 16, 16, True, False, True): (3, 128, 4, 4),
        (8192, 8192, 32768, 32, 32, False, True, True): (2, 128, 4, 8),
        (8192, 8192, 32768, 32, 32, True, False, True): (2, 128, 4, 8),
        (8192, 8192, 32768, 64, 64, False, True, True): (2, 128, 2, 4),
        (8192, 8192, 32768, 64, 64, True, False, True): (2, 128, 2, 4),
        (8192, 8192, 32768, 128, 128, False, True, True): (4, 256, 1, 4),
        (8192, 8192, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (8192, 8192, 65536, 16, 16, False, True, True): (3, 256, 4, 4),
        (8192, 8192, 65536, 16, 16, True, False, True): (3, 256, 4, 4),
        (8192, 8192, 65536, 32, 32, False, True, True): (2, 256, 3, 4),
        (8192, 8192, 65536, 32, 32, True, False, True): (2, 256, 3, 4),
        (8192, 8192, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (8192, 8192, 65536, 64, 64, True, False, True): (2, 256, 3, 8),
        (8192, 8192, 65536, 128, 128, False, True, True): (4, 512, 1, 4),
        (8192, 8192, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (8192, 8192, 131072, 16, 16, False, True, True): (3, 512, 4, 4),
        (8192, 8192, 131072, 16, 16, True, False, True): (3, 512, 4, 4),
        (8192, 8192, 131072, 32, 32, False, True, True): (2, 512, 4, 4),
        (8192, 8192, 131072, 32, 32, True, False, True): (2, 512, 3, 4),
        (8192, 8192, 131072, 64, 64, False, True, True): (4, 512, 2, 4),
        (8192, 8192, 131072, 64, 64, True, False, True): (2, 512, 2, 4),
        (8192, 8192, 131072, 128, 128, False, True, True): (4, 1024, 1, 4),
        (8192, 8192, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (16384, 16384, 256, 16, 16, False, True, True): (2, 2, 6, 4),
        (16384, 16384, 256, 16, 16, True, False, True): (2, 2, 6, 4),
        (16384, 16384, 256, 32, 32, False, True, True): (4, 2, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3595-3630 / 第 3595-3630 行
````python
        (16384, 16384, 256, 32, 32, True, False, True): (4, 2, 3, 2),
        (16384, 16384, 256, 64, 64, False, True, True): (2, 2, 4, 4),
        (16384, 16384, 256, 64, 64, True, False, True): (4, 2, 3, 8),
        (16384, 16384, 256, 128, 128, False, True, True): (4, 2, 2, 8),
        (16384, 16384, 256, 128, 128, True, False, True): (4, 2, 2, 8),
        (16384, 16384, 512, 16, 16, False, True, True): (1, 2, 4, 4),
        (16384, 16384, 512, 16, 16, True, False, True): (1, 2, 4, 4),
        (16384, 16384, 512, 32, 32, False, True, True): (2, 2, 4, 8),
        (16384, 16384, 512, 32, 32, True, False, True): (2, 2, 4, 8),
        (16384, 16384, 512, 64, 64, False, True, True): (4, 4, 3, 4),
        (16384, 16384, 512, 64, 64, True, False, True): (4, 4, 3, 4),
        (16384, 16384, 512, 128, 128, False, True, True): (4, 4, 2, 8),
        (16384, 16384, 512, 128, 128, True, False, True): (4, 4, 2, 8),
        (16384, 16384, 1024, 16, 16, False, True, True): (3, 4, 4, 4),
        (16384, 16384, 1024, 16, 16, True, False, True): (2, 8, 4, 4),
        (16384, 16384, 1024, 32, 32, False, True, True): (2, 4, 4, 8),
        (16384, 16384, 1024, 32, 32, True, False, True): (1, 4, 4, 8),
        (16384, 16384, 1024, 64, 64, False, True, True): (2, 8, 3, 4),
        (16384, 16384, 1024, 64, 64, True, False, True): (2, 8, 3, 4),
        (16384, 16384, 1024, 128, 128, False, True, True): (4, 8, 1, 4),
        (16384, 16384, 1024, 128, 128, True, False, True): (4, 8, 1, 4),
        (16384, 16384, 2048, 16, 16, False, True, True): (2, 8, 4, 4),
        (16384, 16384, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (16384, 16384, 2048, 32, 32, False, True, True): (1, 8, 4, 8),
        (16384, 16384, 2048, 32, 32, True, False, True): (2, 8, 4, 8),
        (16384, 16384, 2048, 64, 64, False, True, True): (2, 8, 2, 4),
        (16384, 16384, 2048, 64, 64, True, False, True): (2, 8, 2, 4),
        (16384, 16384, 2048, 128, 128, False, True, True): (4, 16, 1, 4),
        (16384, 16384, 2048, 128, 128, True, False, True): (4, 16, 1, 4),
        (16384, 16384, 4096, 16, 16, False, True, True): (2, 16, 4, 4),
        (16384, 16384, 4096, 16, 16, True, False, True): (2, 16, 4, 4),
        (16384, 16384, 4096, 32, 32, False, True, True): (1, 8, 3, 8),
        (16384, 16384, 4096, 32, 32, True, False, True): (2, 16, 3, 4),
        (16384, 16384, 4096, 64, 64, False, True, True): (2, 16, 2, 4),
        (16384, 16384, 4096, 64, 64, True, False, True): (2, 16, 2, 4),
        (16384, 16384, 4096, 128, 128, False, True, True): (4, 32, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3631-3666 / 第 3631-3666 行
````python
        (16384, 16384, 4096, 128, 128, True, False, True): (4, 32, 1, 4),
        (16384, 16384, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (16384, 16384, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (16384, 16384, 8192, 32, 32, False, True, True): (2, 32, 4, 8),
        (16384, 16384, 8192, 32, 32, True, False, True): (2, 32, 3, 4),
        (16384, 16384, 8192, 64, 64, False, True, True): (2, 32, 4, 8),
        (16384, 16384, 8192, 64, 64, True, False, True): (2, 32, 3, 8),
        (16384, 16384, 8192, 128, 128, False, True, True): (4, 64, 1, 4),
        (16384, 16384, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (16384, 16384, 16384, 16, 16, False, True, True): (1, 64, 4, 4),
        (16384, 16384, 16384, 16, 16, True, False, True): (1, 64, 4, 4),
        (16384, 16384, 16384, 32, 32, False, True, True): (1, 64, 3, 8),
        (16384, 16384, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (16384, 16384, 16384, 64, 64, False, True, True): (1, 64, 2, 4),
        (16384, 16384, 16384, 64, 64, True, False, True): (1, 64, 4, 8),
        (16384, 16384, 16384, 128, 128, False, True, True): (4, 128, 1, 4),
        (16384, 16384, 16384, 128, 128, True, False, True): (4, 128, 1, 4),
        (16384, 16384, 32768, 16, 16, False, True, True): (1, 128, 4, 4),
        (16384, 16384, 32768, 16, 16, True, False, True): (1, 128, 4, 4),
        (16384, 16384, 32768, 32, 32, False, True, True): (1, 128, 4, 2),
        (16384, 16384, 32768, 32, 32, True, False, True): (1, 128, 3, 8),
        (16384, 16384, 32768, 64, 64, False, True, True): (2, 128, 2, 4),
        (16384, 16384, 32768, 64, 64, True, False, True): (1, 128, 3, 8),
        (16384, 16384, 32768, 128, 128, False, True, True): (4, 256, 1, 4),
        (16384, 16384, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (16384, 16384, 65536, 16, 16, False, True, True): (1, 256, 4, 4),
        (16384, 16384, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (16384, 16384, 65536, 32, 32, False, True, True): (1, 256, 3, 4),
        (16384, 16384, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (16384, 16384, 65536, 64, 64, False, True, True): (1, 256, 2, 4),
        (16384, 16384, 65536, 64, 64, True, False, True): (2, 256, 2, 4),
        (16384, 16384, 65536, 128, 128, False, True, True): (4, 512, 1, 4),
        (16384, 16384, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (16384, 16384, 131072, 16, 16, False, True, True): (2, 512, 4, 4),
        (16384, 16384, 131072, 16, 16, True, False, True): (1, 512, 4, 4),
        (16384, 16384, 131072, 32, 32, False, True, True): (1, 512, 4, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3667-3702 / 第 3667-3702 行
````python
        (16384, 16384, 131072, 32, 32, True, False, True): (1, 512, 3, 4),
        (16384, 16384, 131072, 64, 64, False, True, True): (2, 512, 2, 4),
        (16384, 16384, 131072, 64, 64, True, False, True): (1, 512, 2, 4),
        (16384, 16384, 131072, 128, 128, False, True, True): (4, 1024, 1, 4),
        (16384, 16384, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.bfloat16, 0.56)): {
        (192, 192, 256, 64, 64, False, True, True): (3, 4, 3, 4),
        (192, 192, 256, 64, 64, True, False, True): (1, 4, 4, 4),
        (192, 192, 512, 64, 64, False, True, True): (2, 8, 3, 4),
        (192, 192, 512, 64, 64, True, False, True): (2, 8, 3, 4),
        (192, 192, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (192, 192, 1024, 64, 64, True, False, True): (1, 16, 5, 4),
        (192, 192, 2048, 64, 64, False, True, True): (3, 32, 3, 4),
        (192, 192, 2048, 64, 64, True, False, True): (5, 32, 3, 4),
        (192, 192, 4096, 64, 64, False, True, True): (1, 64, 4, 4),
        (192, 192, 4096, 64, 64, True, False, True): (2, 32, 3, 4),
        (192, 192, 8192, 64, 64, False, True, True): (1, 128, 2, 4),
        (192, 192, 8192, 64, 64, True, False, True): (1, 64, 3, 4),
        (192, 192, 16384, 64, 64, False, True, True): (1, 256, 1, 4),
        (192, 192, 16384, 64, 64, True, False, True): (1, 64, 3, 4),
        (192, 192, 32768, 64, 64, False, True, True): (2, 512, 1, 2),
        (192, 192, 32768, 64, 64, True, False, True): (2, 256, 2, 4),
        (192, 192, 65536, 64, 64, False, True, True): (3, 512, 1, 4),
        (192, 192, 65536, 64, 64, True, False, True): (1, 512, 2, 4),
        (192, 192, 131072, 64, 64, False, True, True): (5, 1024, 1, 4),
        (192, 192, 131072, 64, 64, True, False, True): (4, 512, 2, 4),
        (384, 384, 256, 128, 128, False, True, True): (3, 2, 3, 8),
        (384, 384, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (384, 384, 512, 128, 128, False, True, True): (4, 4, 3, 8),
        (384, 384, 512, 128, 128, True, False, True): (3, 4, 3, 8),
        (384, 384, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
        (384, 384, 1024, 128, 128, True, False, True): (2, 8, 3, 8),
        (384, 384, 2048, 128, 128, False, True, True): (5, 16, 3, 8),
        (384, 384, 2048, 128, 128, True, False, True): (5, 16, 3, 8),
        (384, 384, 4096, 128, 128, False, True, True): (3, 32, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3703-3738 / 第 3703-3738 行
````python
        (384, 384, 4096, 128, 128, True, False, True): (6, 32, 3, 8),
        (384, 384, 8192, 128, 128, False, True, True): (2, 64, 3, 8),
        (384, 384, 8192, 128, 128, True, False, True): (4, 32, 2, 8),
        (384, 384, 16384, 128, 128, False, True, True): (2, 128, 3, 8),
        (384, 384, 16384, 128, 128, True, False, True): (5, 128, 2, 4),
        (384, 384, 32768, 128, 128, False, True, True): (2, 256, 3, 8),
        (384, 384, 32768, 128, 128, True, False, True): (3, 256, 2, 4),
        (384, 384, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (384, 384, 65536, 128, 128, True, False, True): (1, 512, 2, 4),
        (384, 384, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (384, 384, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.float16, 0.5)): {
        (16, 16, 16, 16, 16, False, False, False): (1, 1, 1, 1),
        (16, 16, 16, 16, 16, False, False, True): (1, 1, 2, 2),
        (16, 16, 16, 16, 16, False, True, False): (1, 1, 1, 1),
        (16, 16, 16, 16, 16, False, True, True): (1, 1, 1, 8),
        (16, 16, 16, 16, 16, True, False, False): (3, 1, 3, 4),
        (16, 16, 16, 16, 16, True, False, True): (1, 1, 2, 1),
        (16, 16, 32, 16, 16, False, False, False): (1, 2, 1, 8),
        (16, 16, 32, 16, 16, False, False, True): (1, 2, 1, 2),
        (16, 16, 32, 16, 16, False, True, False): (2, 1, 1, 4),
        (16, 16, 32, 16, 16, False, True, True): (1, 2, 1, 4),
        (16, 16, 32, 16, 16, True, False, False): (1, 1, 1, 4),
        (16, 16, 32, 16, 16, True, False, True): (1, 2, 1, 2),
        (16, 16, 64, 16, 16, False, False, False): (1, 4, 1, 1),
        (16, 16, 64, 16, 16, False, False, True): (1, 2, 2, 4),
        (16, 16, 64, 16, 16, False, True, False): (1, 4, 1, 4),
        (16, 16, 64, 16, 16, False, True, True): (1, 2, 1, 4),
        (16, 16, 64, 16, 16, True, False, False): (1, 4, 1, 2),
        (16, 16, 64, 16, 16, True, False, True): (1, 1, 1, 2),
        (16, 32, 16, 16, 16, False, False, False): (1, 1, 2, 4),
        (16, 32, 16, 16, 16, False, False, True): (1, 1, 1, 4),
        (16, 32, 16, 16, 16, False, True, False): (1, 1, 1, 2),
        (16, 32, 16, 16, 16, False, True, True): (1, 1, 1, 2),
        (16, 32, 16, 16, 16, True, False, False): (1, 1, 2, 16),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3739-3774 / 第 3739-3774 行
````python
        (16, 32, 16, 16, 16, True, False, True): (1, 1, 1, 4),
        (16, 32, 16, 16, 32, False, False, False): (2, 1, 1, 8),
        (16, 32, 16, 16, 32, False, False, True): (2, 1, 1, 8),
        (16, 32, 16, 16, 32, False, True, False): (1, 1, 2, 1),
        (16, 32, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (16, 32, 16, 16, 32, True, False, False): (2, 1, 1, 8),
        (16, 32, 16, 16, 32, True, False, True): (1, 1, 2, 4),
        (16, 32, 32, 16, 16, False, False, False): (1, 1, 1, 16),
        (16, 32, 32, 16, 16, False, False, True): (1, 2, 1, 2),
        (16, 32, 32, 16, 16, False, True, False): (1, 2, 1, 8),
        (16, 32, 32, 16, 16, False, True, True): (3, 2, 1, 4),
        (16, 32, 32, 16, 16, True, False, False): (1, 2, 1, 4),
        (16, 32, 32, 16, 16, True, False, True): (1, 2, 1, 2),
        (16, 32, 32, 16, 32, False, False, False): (1, 2, 1, 2),
        (16, 32, 32, 16, 32, False, False, True): (1, 1, 1, 4),
        (16, 32, 32, 16, 32, False, True, False): (1, 1, 2, 4),
        (16, 32, 32, 16, 32, False, True, True): (1, 2, 1, 2),
        (16, 32, 32, 16, 32, True, False, False): (1, 2, 1, 2),
        (16, 32, 32, 16, 32, True, False, True): (1, 2, 1, 16),
        (16, 32, 64, 16, 16, False, False, False): (1, 4, 1, 4),
        (16, 32, 64, 16, 16, False, False, True): (2, 4, 1, 4),
        (16, 32, 64, 16, 16, False, True, False): (1, 4, 1, 4),
        (16, 32, 64, 16, 16, False, True, True): (1, 4, 1, 4),
        (16, 32, 64, 16, 16, True, False, False): (3, 4, 1, 2),
        (16, 32, 64, 16, 16, True, False, True): (1, 4, 1, 1),
        (16, 32, 64, 16, 32, False, False, False): (1, 4, 1, 16),
        (16, 32, 64, 16, 32, False, False, True): (1, 2, 1, 2),
        (16, 32, 64, 16, 32, False, True, False): (1, 4, 2, 2),
        (16, 32, 64, 16, 32, False, True, True): (1, 4, 1, 8),
        (16, 32, 64, 16, 32, True, False, False): (1, 4, 1, 8),
        (16, 32, 64, 16, 32, True, False, True): (1, 2, 1, 4),
        (16, 64, 16, 16, 32, False, False, False): (1, 1, 1, 2),
        (16, 64, 16, 16, 32, False, False, True): (1, 1, 1, 4),
        (16, 64, 16, 16, 32, False, True, False): (2, 1, 2, 4),
        (16, 64, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (16, 64, 16, 16, 32, True, False, False): (1, 1, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3775-3810 / 第 3775-3810 行
````python
        (16, 64, 16, 16, 32, True, False, True): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, False, False, False): (1, 2, 1, 2),
        (16, 64, 32, 16, 32, False, False, True): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, False, True, False): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, False, True, True): (1, 2, 3, 2),
        (16, 64, 32, 16, 32, True, False, False): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, True, False, True): (1, 1, 2, 4),
        (16, 64, 64, 16, 32, False, False, False): (1, 4, 1, 8),
        (16, 64, 64, 16, 32, False, False, True): (1, 4, 1, 4),
        (16, 64, 64, 16, 32, False, True, False): (1, 4, 1, 1),
        (16, 64, 64, 16, 32, False, True, True): (2, 4, 1, 4),
        (16, 64, 64, 16, 32, True, False, False): (1, 4, 1, 4),
        (16, 64, 64, 16, 32, True, False, True): (1, 4, 1, 4),
        (32, 16, 16, 16, 16, False, False, False): (2, 1, 2, 4),
        (32, 16, 16, 16, 16, False, False, True): (2, 1, 1, 2),
        (32, 16, 16, 16, 16, False, True, False): (1, 1, 2, 4),
        (32, 16, 16, 16, 16, False, True, True): (1, 1, 1, 2),
        (32, 16, 16, 16, 16, True, False, False): (1, 1, 1, 4),
        (32, 16, 16, 16, 16, True, False, True): (2, 1, 1, 2),
        (32, 16, 32, 16, 16, False, False, False): (1, 1, 1, 4),
        (32, 16, 32, 16, 16, False, False, True): (1, 1, 1, 4),
        (32, 16, 32, 16, 16, False, True, False): (1, 2, 1, 4),
        (32, 16, 32, 16, 16, False, True, True): (2, 2, 1, 4),
        (32, 16, 32, 16, 16, True, False, False): (2, 1, 1, 4),
        (32, 16, 32, 16, 16, True, False, True): (2, 2, 1, 2),
        (32, 16, 64, 16, 16, False, False, False): (1, 4, 1, 2),
        (32, 16, 64, 16, 16, False, False, True): (1, 4, 1, 4),
        (32, 16, 64, 16, 16, False, True, False): (1, 2, 1, 4),
        (32, 16, 64, 16, 16, False, True, True): (1, 4, 1, 2),
        (32, 16, 64, 16, 16, True, False, False): (1, 4, 2, 8),
        (32, 16, 64, 16, 16, True, False, True): (1, 4, 1, 1),
        (32, 32, 16, 16, 16, False, False, False): (1, 1, 1, 4),
        (32, 32, 16, 16, 16, False, False, True): (2, 1, 1, 4),
        (32, 32, 16, 16, 16, False, True, False): (1, 1, 2, 4),
        (32, 32, 16, 16, 16, False, True, True): (1, 1, 2, 2),
        (32, 32, 16, 16, 16, True, False, False): (1, 1, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3811-3846 / 第 3811-3846 行
````python
        (32, 32, 16, 16, 16, True, False, True): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, False, False, False): (1, 1, 3, 2),
        (32, 32, 16, 16, 32, False, False, True): (2, 1, 1, 4),
        (32, 32, 16, 16, 32, False, True, False): (3, 1, 1, 4),
        (32, 32, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, True, False, False): (2, 1, 1, 8),
        (32, 32, 16, 16, 32, True, False, True): (1, 1, 3, 2),
        (32, 32, 16, 32, 32, False, False, False): (1, 1, 1, 2),
        (32, 32, 16, 32, 32, False, False, True): (2, 1, 1, 8),
        (32, 32, 16, 32, 32, False, True, False): (1, 1, 1, 2),
        (32, 32, 16, 32, 32, False, True, True): (1, 1, 1, 8),
        (32, 32, 16, 32, 32, True, False, False): (1, 1, 2, 4),
        (32, 32, 16, 32, 32, True, False, True): (1, 1, 1, 2),
        (32, 32, 32, 16, 16, False, False, False): (1, 1, 1, 4),
        (32, 32, 32, 16, 16, False, False, True): (1, 2, 1, 4),
        (32, 32, 32, 16, 16, False, True, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 16, False, True, True): (1, 2, 1, 2),
        (32, 32, 32, 16, 16, True, False, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 16, True, False, True): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, False, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, False, True): (1, 2, 1, 2),
        (32, 32, 32, 16, 32, False, True, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, True, True): (1, 2, 1, 2),
        (32, 32, 32, 16, 32, True, False, False): (1, 2, 1, 1),
        (32, 32, 32, 16, 32, True, False, True): (1, 2, 1, 2),
        (32, 32, 32, 32, 32, False, False, False): (1, 1, 1, 4),
        (32, 32, 32, 32, 32, False, False, True): (2, 1, 1, 4),
        (32, 32, 32, 32, 32, False, True, False): (1, 1, 1, 8),
        (32, 32, 32, 32, 32, False, True, True): (1, 1, 1, 8),
        (32, 32, 32, 32, 32, True, False, False): (1, 1, 3, 4),
        (32, 32, 32, 32, 32, True, False, True): (1, 1, 1, 8),
        (32, 32, 64, 16, 16, False, False, False): (1, 4, 1, 4),
        (32, 32, 64, 16, 16, False, False, True): (1, 4, 1, 2),
        (32, 32, 64, 16, 16, False, True, False): (1, 1, 1, 4),
        (32, 32, 64, 16, 16, False, True, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 16, True, False, False): (1, 4, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3847-3882 / 第 3847-3882 行
````python
        (32, 32, 64, 16, 16, True, False, True): (1, 4, 1, 2),
        (32, 32, 64, 16, 32, False, False, False): (1, 1, 1, 4),
        (32, 32, 64, 16, 32, False, False, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 32, False, True, False): (1, 1, 1, 4),
        (32, 32, 64, 16, 32, False, True, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 32, True, False, False): (2, 2, 1, 8),
        (32, 32, 64, 16, 32, True, False, True): (1, 2, 1, 2),
        (32, 32, 64, 32, 32, False, False, False): (1, 2, 1, 4),
        (32, 32, 64, 32, 32, False, False, True): (1, 2, 1, 1),
        (32, 32, 64, 32, 32, False, True, False): (1, 2, 2, 8),
        (32, 32, 64, 32, 32, False, True, True): (1, 1, 1, 4),
        (32, 32, 64, 32, 32, True, False, False): (1, 2, 1, 4),
        (32, 32, 64, 32, 32, True, False, True): (2, 2, 1, 4),
        (32, 64, 16, 16, 32, False, False, False): (1, 1, 1, 8),
        (32, 64, 16, 16, 32, False, False, True): (1, 1, 1, 4),
        (32, 64, 16, 16, 32, False, True, False): (2, 1, 1, 4),
        (32, 64, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (32, 64, 16, 16, 32, True, False, False): (1, 1, 2, 4),
        (32, 64, 16, 16, 32, True, False, True): (1, 1, 2, 2),
        (32, 64, 16, 32, 32, False, False, False): (1, 1, 1, 8),
        (32, 64, 16, 32, 32, False, False, True): (2, 1, 1, 4),
        (32, 64, 16, 32, 32, False, True, False): (1, 1, 1, 4),
        (32, 64, 16, 32, 32, False, True, True): (1, 1, 2, 2),
        (32, 64, 16, 32, 32, True, False, False): (1, 1, 1, 2),
        (32, 64, 16, 32, 32, True, False, True): (2, 1, 2, 4),
        (32, 64, 32, 16, 32, False, False, False): (1, 1, 1, 4),
        (32, 64, 32, 16, 32, False, False, True): (1, 2, 1, 2),
        (32, 64, 32, 16, 32, False, True, False): (1, 2, 3, 4),
        (32, 64, 32, 16, 32, False, True, True): (2, 2, 1, 4),
        (32, 64, 32, 16, 32, True, False, False): (1, 1, 1, 4),
        (32, 64, 32, 16, 32, True, False, True): (1, 2, 2, 1),
        (32, 64, 32, 32, 32, False, False, False): (1, 1, 1, 8),
        (32, 64, 32, 32, 32, False, False, True): (1, 1, 1, 4),
        (32, 64, 32, 32, 32, False, True, False): (1, 1, 2, 4),
        (32, 64, 32, 32, 32, False, True, True): (1, 1, 1, 4),
        (32, 64, 32, 32, 32, True, False, False): (2, 1, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3883-3918 / 第 3883-3918 行
````python
        (32, 64, 32, 32, 32, True, False, True): (1, 1, 1, 4),
        (32, 64, 64, 16, 32, False, False, False): (1, 4, 2, 1),
        (32, 64, 64, 16, 32, False, False, True): (3, 4, 1, 4),
        (32, 64, 64, 16, 32, False, True, False): (1, 1, 1, 8),
        (32, 64, 64, 16, 32, False, True, True): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, True, False, False): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, True, False, True): (2, 2, 3, 4),
        (32, 64, 64, 32, 32, False, False, False): (1, 2, 1, 4),
        (32, 64, 64, 32, 32, False, False, True): (1, 2, 1, 4),
        (32, 64, 64, 32, 32, False, True, False): (1, 2, 2, 8),
        (32, 64, 64, 32, 32, False, True, True): (1, 2, 1, 4),
        (32, 64, 64, 32, 32, True, False, False): (1, 2, 2, 4),
        (32, 64, 64, 32, 32, True, False, True): (1, 2, 1, 4),
        (64, 32, 16, 32, 32, False, False, False): (1, 1, 1, 1),
        (64, 32, 16, 32, 32, False, False, True): (1, 1, 2, 4),
        (64, 32, 16, 32, 32, False, True, False): (2, 1, 1, 8),
        (64, 32, 16, 32, 32, False, True, True): (1, 1, 1, 4),
        (64, 32, 16, 32, 32, True, False, False): (2, 1, 1, 2),
        (64, 32, 16, 32, 32, True, False, True): (1, 1, 1, 4),
        (64, 32, 32, 32, 32, False, False, False): (3, 1, 1, 4),
        (64, 32, 32, 32, 32, False, False, True): (1, 1, 1, 4),
        (64, 32, 32, 32, 32, False, True, False): (1, 1, 1, 8),
        (64, 32, 32, 32, 32, False, True, True): (1, 1, 1, 2),
        (64, 32, 32, 32, 32, True, False, False): (1, 1, 1, 2),
        (64, 32, 32, 32, 32, True, False, True): (1, 1, 1, 4),
        (64, 32, 64, 32, 32, False, False, False): (1, 2, 1, 2),
        (64, 32, 64, 32, 32, False, False, True): (3, 2, 1, 4),
        (64, 32, 64, 32, 32, False, True, False): (1, 1, 1, 1),
        (64, 32, 64, 32, 32, False, True, True): (1, 2, 1, 4),
        (64, 32, 64, 32, 32, True, False, False): (1, 1, 3, 4),
        (64, 32, 64, 32, 32, True, False, True): (1, 2, 2, 4),
        (64, 64, 16, 32, 32, False, False, False): (1, 1, 2, 2),
        (64, 64, 16, 32, 32, False, False, True): (1, 1, 3, 2),
        (64, 64, 16, 32, 32, False, True, False): (1, 1, 1, 8),
        (64, 64, 16, 32, 32, False, True, True): (1, 1, 2, 4),
        (64, 64, 16, 32, 32, True, False, False): (1, 1, 2, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3919-3954 / 第 3919-3954 行
````python
        (64, 64, 16, 32, 32, True, False, True): (2, 1, 2, 4),
        (64, 64, 32, 32, 32, False, False, False): (1, 1, 2, 8),
        (64, 64, 32, 32, 32, False, False, True): (1, 1, 2, 4),
        (64, 64, 32, 32, 32, False, True, False): (1, 1, 1, 4),
        (64, 64, 32, 32, 32, False, True, True): (1, 1, 1, 4),
        (64, 64, 32, 32, 32, True, False, False): (1, 1, 1, 4),
        (64, 64, 32, 32, 32, True, False, True): (2, 1, 2, 4),
        (64, 64, 64, 32, 32, False, False, False): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, False, False, True): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, False, True, False): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, False, True, True): (3, 2, 1, 4),
        (64, 64, 64, 32, 32, True, False, False): (1, 2, 1, 8),
        (64, 64, 64, 32, 32, True, False, True): (1, 2, 3, 4),
        (192, 192, 256, 16, 16, False, True, True): (1, 8, 4, 2),
        (192, 192, 256, 16, 16, True, False, True): (1, 4, 4, 4),
        (192, 192, 256, 32, 32, False, True, True): (2, 8, 5, 4),
        (192, 192, 256, 32, 32, True, False, True): (2, 8, 5, 1),
        (192, 192, 512, 16, 16, False, True, True): (3, 8, 4, 4),
        (192, 192, 512, 16, 16, True, False, True): (5, 8, 5, 4),
        (192, 192, 512, 32, 32, False, True, True): (1, 16, 5, 4),
        (192, 192, 512, 32, 32, True, False, True): (1, 8, 6, 2),
        (192, 192, 1024, 16, 16, False, True, True): (1, 16, 4, 4),
        (192, 192, 1024, 16, 16, True, False, True): (3, 16, 5, 2),
        (192, 192, 1024, 32, 32, False, True, True): (3, 16, 4, 4),
        (192, 192, 1024, 32, 32, True, False, True): (1, 16, 5, 4),
        (192, 192, 2048, 16, 16, False, True, True): (2, 16, 3, 4),
        (192, 192, 2048, 16, 16, True, False, True): (1, 16, 4, 4),
        (192, 192, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (192, 192, 2048, 32, 32, True, False, True): (3, 16, 4, 4),
        (192, 192, 4096, 16, 16, False, True, True): (1, 64, 1, 4),
        (192, 192, 4096, 16, 16, True, False, True): (1, 16, 3, 4),
        (192, 192, 4096, 32, 32, False, True, True): (1, 128, 1, 4),
        (192, 192, 4096, 32, 32, True, False, True): (2, 32, 4, 2),
        (192, 192, 8192, 16, 16, False, True, True): (1, 64, 1, 4),
        (192, 192, 8192, 16, 16, True, False, True): (2, 64, 3, 2),
        (192, 192, 8192, 32, 32, False, True, True): (1, 128, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3955-3990 / 第 3955-3990 行
````python
        (192, 192, 8192, 32, 32, True, False, True): (4, 32, 3, 4),
        (192, 192, 16384, 16, 16, False, True, True): (1, 128, 1, 4),
        (192, 192, 16384, 16, 16, True, False, True): (1, 64, 3, 2),
        (192, 192, 16384, 32, 32, False, True, True): (1, 128, 1, 4),
        (192, 192, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (192, 192, 32768, 16, 16, False, True, True): (2, 256, 1, 2),
        (192, 192, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (192, 192, 32768, 32, 32, False, True, True): (2, 256, 1, 4),
        (192, 192, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (192, 192, 65536, 16, 16, False, True, True): (2, 512, 1, 2),
        (192, 192, 65536, 16, 16, True, False, True): (1, 256, 3, 2),
        (192, 192, 65536, 32, 32, False, True, True): (2, 512, 1, 4),
        (192, 192, 65536, 32, 32, True, False, True): (2, 256, 3, 4),
        (192, 192, 131072, 16, 16, False, True, True): (4, 1024, 1, 2),
        (192, 192, 131072, 16, 16, True, False, True): (3, 512, 3, 2),
        (192, 192, 131072, 32, 32, False, True, True): (1, 1024, 1, 4),
        (192, 192, 131072, 32, 32, True, False, True): (3, 512, 3, 4),
        (256, 256, 256, 16, 16, False, True, True): (4, 8, 6, 2),
        (256, 256, 256, 16, 16, True, False, True): (5, 16, 5, 1),
        (256, 256, 256, 32, 32, False, True, True): (1, 8, 7, 4),
        (256, 256, 256, 32, 32, True, False, True): (1, 8, 5, 4),
        (256, 256, 256, 64, 64, False, True, True): (1, 4, 5, 4),
        (256, 256, 256, 64, 64, True, False, True): (2, 4, 3, 4),
        (256, 256, 256, 128, 128, False, True, True): (1, 2, 2, 8),
        (256, 256, 256, 128, 128, True, False, True): (1, 2, 2, 8),
        (256, 256, 512, 16, 16, False, True, True): (4, 8, 4, 4),
        (256, 256, 512, 16, 16, True, False, True): (4, 8, 6, 2),
        (256, 256, 512, 32, 32, False, True, True): (3, 8, 5, 4),
        (256, 256, 512, 32, 32, True, False, True): (2, 8, 5, 4),
        (256, 256, 512, 64, 64, False, True, True): (2, 8, 4, 4),
        (256, 256, 512, 64, 64, True, False, True): (1, 8, 7, 4),
        (256, 256, 512, 128, 128, False, True, True): (2, 4, 2, 8),
        (256, 256, 512, 128, 128, True, False, True): (5, 4, 2, 8),
        (256, 256, 1024, 16, 16, False, True, True): (1, 8, 4, 4),
        (256, 256, 1024, 16, 16, True, False, True): (1, 16, 4, 2),
        (256, 256, 1024, 32, 32, False, True, True): (5, 32, 5, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 3991-4026 / 第 3991-4026 行
````python
        (256, 256, 1024, 32, 32, True, False, True): (1, 16, 4, 2),
        (256, 256, 1024, 64, 64, False, True, True): (1, 16, 4, 4),
        (256, 256, 1024, 64, 64, True, False, True): (2, 16, 3, 4),
        (256, 256, 1024, 128, 128, False, True, True): (9, 8, 2, 8),
        (256, 256, 1024, 128, 128, True, False, True): (1, 8, 2, 8),
        (256, 256, 2048, 16, 16, False, True, True): (6, 32, 5, 2),
        (256, 256, 2048, 16, 16, True, False, True): (2, 32, 4, 2),
        (256, 256, 2048, 32, 32, False, True, True): (1, 32, 3, 2),
        (256, 256, 2048, 32, 32, True, False, True): (1, 32, 3, 2),
        (256, 256, 2048, 64, 64, False, True, True): (2, 32, 4, 4),
        (256, 256, 2048, 64, 64, True, False, True): (2, 16, 4, 4),
        (256, 256, 2048, 128, 128, False, True, True): (3, 16, 2, 8),
        (256, 256, 2048, 128, 128, True, False, True): (4, 16, 2, 8),
        (256, 256, 4096, 16, 16, False, True, True): (1, 32, 3, 4),
        (256, 256, 4096, 16, 16, True, False, True): (3, 16, 3, 2),
        (256, 256, 4096, 32, 32, False, True, True): (3, 32, 3, 2),
        (256, 256, 4096, 32, 32, True, False, True): (1, 32, 3, 2),
        (256, 256, 4096, 64, 64, False, True, True): (2, 32, 3, 4),
        (256, 256, 4096, 64, 64, True, False, True): (2, 32, 3, 4),
        (256, 256, 4096, 128, 128, False, True, True): (5, 32, 2, 8),
        (256, 256, 4096, 128, 128, True, False, True): (1, 32, 2, 8),
        (256, 256, 8192, 16, 16, False, True, True): (8, 32, 3, 4),
        (256, 256, 8192, 16, 16, True, False, True): (1, 32, 3, 2),
        (256, 256, 8192, 32, 32, False, True, True): (3, 64, 3, 4),
        (256, 256, 8192, 32, 32, True, False, True): (2, 128, 1, 2),
        (256, 256, 8192, 64, 64, False, True, True): (7, 128, 1, 4),
        (256, 256, 8192, 64, 64, True, False, True): (4, 128, 1, 4),
        (256, 256, 8192, 128, 128, False, True, True): (2, 64, 1, 4),
        (256, 256, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (256, 256, 16384, 16, 16, False, True, True): (4, 128, 3, 2),
        (256, 256, 16384, 16, 16, True, False, True): (5, 64, 3, 2),
        (256, 256, 16384, 32, 32, False, True, True): (5, 128, 3, 2),
        (256, 256, 16384, 32, 32, True, False, True): (5, 128, 3, 2),
        (256, 256, 16384, 64, 64, False, True, True): (1, 256, 1, 4),
        (256, 256, 16384, 64, 64, True, False, True): (5, 128, 3, 4),
        (256, 256, 16384, 128, 128, False, True, True): (11, 128, 2, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4027-4062 / 第 4027-4062 行
````python
        (256, 256, 16384, 128, 128, True, False, True): (3, 128, 1, 4),
        (256, 256, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (256, 256, 32768, 16, 16, True, False, True): (2, 128, 3, 2),
        (256, 256, 32768, 32, 32, False, True, True): (4, 256, 3, 2),
        (256, 256, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (256, 256, 32768, 64, 64, False, True, True): (2, 256, 1, 4),
        (256, 256, 32768, 64, 64, True, False, True): (2, 256, 1, 4),
        (256, 256, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (256, 256, 32768, 128, 128, True, False, True): (2, 256, 1, 4),
        (256, 256, 50432, 16, 16, False, True, True): (4, 197, 1, 4),
        (256, 256, 50432, 16, 16, True, False, True): (4, 197, 3, 2),
        (256, 256, 50432, 32, 32, False, True, True): (1, 394, 1, 2),
        (256, 256, 50432, 32, 32, True, False, True): (4, 197, 3, 4),
        (256, 256, 50432, 64, 64, False, True, True): (6, 394, 1, 4),
        (256, 256, 50432, 64, 64, True, False, True): (4, 394, 2, 4),
        (256, 256, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (256, 256, 50432, 128, 128, True, False, True): (1, 394, 2, 4),
        (256, 256, 65536, 16, 16, False, True, True): (1, 256, 3, 2),
        (256, 256, 65536, 16, 16, True, False, True): (1, 256, 3, 2),
        (256, 256, 65536, 32, 32, False, True, True): (1, 512, 3, 2),
        (256, 256, 65536, 32, 32, True, False, True): (4, 512, 3, 2),
        (256, 256, 65536, 64, 64, False, True, True): (2, 512, 1, 4),
        (256, 256, 65536, 64, 64, True, False, True): (5, 512, 1, 4),
        (256, 256, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (256, 256, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (256, 256, 65792, 16, 16, False, True, True): (2, 257, 1, 4),
        (256, 256, 65792, 16, 16, True, False, True): (1, 257, 3, 2),
        (256, 256, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (256, 256, 65792, 32, 32, True, False, True): (1, 257, 3, 4),
        (256, 256, 65792, 64, 64, False, True, True): (2, 514, 1, 4),
        (256, 256, 65792, 64, 64, True, False, True): (2, 514, 2, 4),
        (256, 256, 65792, 128, 128, False, True, True): (3, 514, 1, 4),
        (256, 256, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (256, 256, 131072, 16, 16, False, True, True): (1, 512, 3, 1),
        (256, 256, 131072, 16, 16, True, False, True): (1, 512, 3, 2),
        (256, 256, 131072, 32, 32, False, True, True): (2, 1024, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4063-4098 / 第 4063-4098 行
````python
        (256, 256, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (256, 256, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (256, 256, 131072, 64, 64, True, False, True): (1, 1024, 1, 4),
        (256, 256, 131072, 128, 128, False, True, True): (7, 1024, 1, 4),
        (256, 256, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (384, 384, 256, 16, 16, False, True, True): (3, 16, 4, 1),
        (384, 384, 256, 16, 16, True, False, True): (2, 4, 6, 2),
        (384, 384, 256, 32, 32, False, True, True): (1, 8, 4, 4),
        (384, 384, 256, 32, 32, True, False, True): (1, 4, 5, 2),
        (384, 384, 256, 64, 64, False, True, True): (3, 4, 3, 4),
        (384, 384, 256, 64, 64, True, False, True): (4, 4, 5, 4),
        (384, 384, 512, 16, 16, False, True, True): (1, 16, 4, 1),
        (384, 384, 512, 16, 16, True, False, True): (1, 8, 5, 2),
        (384, 384, 512, 32, 32, False, True, True): (4, 16, 4, 2),
        (384, 384, 512, 32, 32, True, False, True): (1, 8, 5, 2),
        (384, 384, 512, 64, 64, False, True, True): (2, 8, 3, 4),
        (384, 384, 512, 64, 64, True, False, True): (1, 8, 4, 4),
        (384, 384, 1024, 16, 16, False, True, True): (1, 16, 4, 2),
        (384, 384, 1024, 16, 16, True, False, True): (7, 8, 5, 2),
        (384, 384, 1024, 32, 32, False, True, True): (2, 16, 3, 4),
        (384, 384, 1024, 32, 32, True, False, True): (1, 16, 4, 2),
        (384, 384, 1024, 64, 64, False, True, True): (6, 16, 3, 4),
        (384, 384, 1024, 64, 64, True, False, True): (4, 16, 4, 4),
        (384, 384, 2048, 16, 16, False, True, True): (1, 32, 1, 4),
        (384, 384, 2048, 16, 16, True, False, True): (1, 16, 3, 2),
        (384, 384, 2048, 32, 32, False, True, True): (1, 32, 1, 8),
        (384, 384, 2048, 32, 32, True, False, True): (1, 8, 4, 4),
        (384, 384, 2048, 64, 64, False, True, True): (2, 32, 1, 8),
        (384, 384, 2048, 64, 64, True, False, True): (3, 16, 3, 4),
        (384, 384, 4096, 16, 16, False, True, True): (5, 32, 1, 4),
        (384, 384, 4096, 16, 16, True, False, True): (1, 32, 3, 2),
        (384, 384, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
        (384, 384, 4096, 32, 32, True, False, True): (2, 16, 4, 4),
        (384, 384, 4096, 64, 64, False, True, True): (1, 64, 1, 4),
        (384, 384, 4096, 64, 64, True, False, True): (2, 32, 3, 4),
        (384, 384, 8192, 16, 16, False, True, True): (2, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4099-4134 / 第 4099-4134 行
````python
        (384, 384, 8192, 16, 16, True, False, True): (3, 32, 3, 2),
        (384, 384, 8192, 32, 32, False, True, True): (4, 128, 1, 4),
        (384, 384, 8192, 32, 32, True, False, True): (1, 32, 3, 2),
        (384, 384, 8192, 64, 64, False, True, True): (1, 128, 1, 4),
        (384, 384, 8192, 64, 64, True, False, True): (1, 64, 3, 4),
        (384, 384, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (384, 384, 16384, 16, 16, True, False, True): (1, 64, 3, 2),
        (384, 384, 16384, 32, 32, False, True, True): (1, 128, 1, 4),
        (384, 384, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (384, 384, 16384, 64, 64, False, True, True): (5, 128, 3, 4),
        (384, 384, 16384, 64, 64, True, False, True): (1, 128, 3, 4),
        (384, 384, 32768, 16, 16, False, True, True): (2, 256, 1, 2),
        (384, 384, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (384, 384, 32768, 32, 32, False, True, True): (1, 256, 1, 2),
        (384, 384, 32768, 32, 32, True, False, True): (2, 128, 3, 4),
        (384, 384, 32768, 64, 64, False, True, True): (3, 256, 1, 4),
        (384, 384, 32768, 64, 64, True, False, True): (2, 256, 3, 4),
        (384, 384, 65536, 16, 16, False, True, True): (2, 128, 1, 4),
        (384, 384, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (384, 384, 65536, 32, 32, False, True, True): (1, 512, 1, 2),
        (384, 384, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (384, 384, 65536, 64, 64, False, True, True): (3, 512, 1, 4),
        (384, 384, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (384, 384, 131072, 16, 16, False, True, True): (2, 256, 1, 2),
        (384, 384, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (384, 384, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
        (384, 384, 131072, 32, 32, True, False, True): (1, 512, 3, 4),
        (384, 384, 131072, 64, 64, False, True, True): (3, 1024, 1, 4),
        (384, 384, 131072, 64, 64, True, False, True): (3, 512, 3, 4),
        (512, 512, 256, 16, 16, False, True, True): (1, 8, 5, 1),
        (512, 512, 256, 16, 16, True, False, True): (2, 16, 5, 1),
        (512, 512, 256, 32, 32, False, True, True): (2, 8, 5, 2),
        (512, 512, 256, 32, 32, True, False, True): (4, 4, 5, 2),
        (512, 512, 256, 64, 64, False, True, True): (1, 4, 5, 4),
        (512, 512, 256, 64, 64, True, False, True): (3, 4, 5, 4),
        (512, 512, 256, 128, 128, False, True, True): (1, 2, 2, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4135-4170 / 第 4135-4170 行
````python
        (512, 512, 256, 128, 128, True, False, True): (1, 2, 2, 8),
        (512, 512, 512, 16, 16, False, True, True): (1, 8, 4, 4),
        (512, 512, 512, 16, 16, True, False, True): (4, 16, 5, 1),
        (512, 512, 512, 32, 32, False, True, True): (4, 8, 5, 2),
        (512, 512, 512, 32, 32, True, False, True): (7, 16, 4, 1),
        (512, 512, 512, 64, 64, False, True, True): (3, 8, 5, 4),
        (512, 512, 512, 64, 64, True, False, True): (1, 8, 4, 4),
        (512, 512, 512, 128, 128, False, True, True): (4, 4, 2, 8),
        (512, 512, 512, 128, 128, True, False, True): (4, 4, 2, 8),
        (512, 512, 1024, 16, 16, False, True, True): (2, 8, 4, 4),
        (512, 512, 1024, 16, 16, True, False, True): (2, 16, 4, 2),
        (512, 512, 1024, 32, 32, False, True, True): (3, 16, 4, 2),
        (512, 512, 1024, 32, 32, True, False, True): (3, 16, 3, 2),
        (512, 512, 1024, 64, 64, False, True, True): (5, 8, 5, 4),
        (512, 512, 1024, 64, 64, True, False, True): (4, 16, 3, 4),
        (512, 512, 1024, 128, 128, False, True, True): (6, 8, 2, 8),
        (512, 512, 1024, 128, 128, True, False, True): (4, 8, 2, 8),
        (512, 512, 2048, 16, 16, False, True, True): (2, 16, 3, 4),
        (512, 512, 2048, 16, 16, True, False, True): (1, 16, 4, 2),
        (512, 512, 2048, 32, 32, False, True, True): (2, 32, 3, 2),
        (512, 512, 2048, 32, 32, True, False, True): (2, 32, 3, 2),
        (512, 512, 2048, 64, 64, False, True, True): (1, 32, 3, 4),
        (512, 512, 2048, 64, 64, True, False, True): (1, 32, 3, 2),
        (512, 512, 2048, 128, 128, False, True, True): (3, 16, 2, 8),
        (512, 512, 2048, 128, 128, True, False, True): (1, 16, 2, 8),
        (512, 512, 4096, 16, 16, False, True, True): (4, 32, 3, 2),
        (512, 512, 4096, 16, 16, True, False, True): (1, 32, 3, 2),
        (512, 512, 4096, 32, 32, False, True, True): (3, 32, 3, 2),
        (512, 512, 4096, 32, 32, True, False, True): (3, 32, 3, 2),
        (512, 512, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (512, 512, 4096, 64, 64, True, False, True): (1, 64, 1, 4),
        (512, 512, 4096, 128, 128, False, True, True): (7, 32, 2, 8),
        (512, 512, 4096, 128, 128, True, False, True): (1, 32, 2, 8),
        (512, 512, 8192, 16, 16, False, True, True): (4, 64, 3, 2),
        (512, 512, 8192, 16, 16, True, False, True): (1, 64, 3, 2),
        (512, 512, 8192, 32, 32, False, True, True): (3, 64, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4171-4206 / 第 4171-4206 行
````python
        (512, 512, 8192, 32, 32, True, False, True): (1, 64, 3, 2),
        (512, 512, 8192, 64, 64, False, True, True): (1, 64, 3, 4),
        (512, 512, 8192, 64, 64, True, False, True): (1, 64, 3, 4),
        (512, 512, 8192, 128, 128, False, True, True): (7, 64, 2, 8),
        (512, 512, 8192, 128, 128, True, False, True): (1, 64, 1, 4),
        (512, 512, 16384, 16, 16, False, True, True): (1, 128, 3, 2),
        (512, 512, 16384, 16, 16, True, False, True): (1, 64, 3, 2),
        (512, 512, 16384, 32, 32, False, True, True): (1, 128, 3, 2),
        (512, 512, 16384, 32, 32, True, False, True): (1, 128, 3, 2),
        (512, 512, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (512, 512, 16384, 64, 64, True, False, True): (4, 128, 3, 4),
        (512, 512, 16384, 128, 128, False, True, True): (5, 128, 2, 8),
        (512, 512, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (512, 512, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (512, 512, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (512, 512, 32768, 32, 32, False, True, True): (1, 256, 3, 2),
        (512, 512, 32768, 32, 32, True, False, True): (1, 256, 3, 2),
        (512, 512, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (512, 512, 32768, 64, 64, True, False, True): (1, 256, 3, 4),
        (512, 512, 32768, 128, 128, False, True, True): (5, 256, 1, 4),
        (512, 512, 32768, 128, 128, True, False, True): (1, 256, 1, 4),
        (512, 512, 50432, 16, 16, False, True, True): (4, 197, 1, 4),
        (512, 512, 50432, 16, 16, True, False, True): (4, 197, 3, 2),
        (512, 512, 50432, 32, 32, False, True, True): (2, 197, 1, 4),
        (512, 512, 50432, 32, 32, True, False, True): (4, 197, 3, 4),
        (512, 512, 50432, 64, 64, False, True, True): (2, 394, 1, 4),
        (512, 512, 50432, 64, 64, True, False, True): (4, 197, 2, 4),
        (512, 512, 50432, 128, 128, False, True, True): (5, 394, 1, 4),
        (512, 512, 50432, 128, 128, True, False, True): (6, 394, 2, 4),
        (512, 512, 65536, 16, 16, False, True, True): (1, 256, 3, 2),
        (512, 512, 65536, 16, 16, True, False, True): (1, 256, 3, 1),
        (512, 512, 65536, 32, 32, False, True, True): (1, 512, 3, 2),
        (512, 512, 65536, 32, 32, True, False, True): (1, 512, 3, 2),
        (512, 512, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (512, 512, 65536, 64, 64, True, False, True): (1, 512, 3, 4),
        (512, 512, 65536, 128, 128, False, True, True): (7, 512, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4207-4242 / 第 4207-4242 行
````python
        (512, 512, 65536, 128, 128, True, False, True): (5, 512, 1, 4),
        (512, 512, 65792, 16, 16, False, True, True): (2, 257, 1, 4),
        (512, 512, 65792, 16, 16, True, False, True): (1, 257, 3, 4),
        (512, 512, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (512, 512, 65792, 32, 32, True, False, True): (1, 257, 3, 4),
        (512, 512, 65792, 64, 64, False, True, True): (4, 514, 1, 4),
        (512, 512, 65792, 64, 64, True, False, True): (4, 257, 2, 4),
        (512, 512, 65792, 128, 128, False, True, True): (5, 514, 1, 4),
        (512, 512, 65792, 128, 128, True, False, True): (4, 514, 2, 4),
        (512, 512, 131072, 16, 16, False, True, True): (1, 512, 3, 1),
        (512, 512, 131072, 16, 16, True, False, True): (1, 512, 3, 1),
        (512, 512, 131072, 32, 32, False, True, True): (1, 1024, 3, 2),
        (512, 512, 131072, 32, 32, True, False, True): (1, 1024, 3, 2),
        (512, 512, 131072, 64, 64, False, True, True): (4, 512, 2, 4),
        (512, 512, 131072, 64, 64, True, False, True): (2, 512, 2, 4),
        (512, 512, 131072, 128, 128, False, True, True): (5, 1024, 1, 4),
        (512, 512, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (768, 768, 256, 16, 16, False, True, True): (1, 8, 4, 1),
        (768, 768, 256, 16, 16, True, False, True): (3, 2, 5, 2),
        (768, 768, 256, 32, 32, False, True, True): (1, 8, 4, 2),
        (768, 768, 256, 32, 32, True, False, True): (2, 4, 6, 2),
        (768, 768, 256, 64, 64, False, True, True): (3, 4, 3, 4),
        (768, 768, 256, 64, 64, True, False, True): (2, 4, 4, 4),
        (768, 768, 256, 128, 128, False, True, True): (1, 2, 3, 8),
        (768, 768, 256, 128, 128, True, False, True): (2, 2, 3, 8),
        (768, 768, 512, 16, 16, False, True, True): (1, 8, 4, 2),
        (768, 768, 512, 16, 16, True, False, True): (2, 8, 5, 2),
        (768, 768, 512, 32, 32, False, True, True): (1, 16, 1, 4),
        (768, 768, 512, 32, 32, True, False, True): (3, 8, 5, 2),
        (768, 768, 512, 64, 64, False, True, True): (4, 8, 3, 4),
        (768, 768, 512, 64, 64, True, False, True): (2, 8, 4, 4),
        (768, 768, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (768, 768, 512, 128, 128, True, False, True): (3, 4, 3, 8),
        (768, 768, 1024, 16, 16, False, True, True): (1, 16, 1, 4),
        (768, 768, 1024, 16, 16, True, False, True): (1, 8, 5, 2),
        (768, 768, 1024, 32, 32, False, True, True): (1, 16, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4243-4278 / 第 4243-4278 行
````python
        (768, 768, 1024, 32, 32, True, False, True): (1, 4, 4, 4),
        (768, 768, 1024, 64, 64, False, True, True): (2, 16, 1, 8),
        (768, 768, 1024, 64, 64, True, False, True): (1, 8, 3, 8),
        (768, 768, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
        (768, 768, 1024, 128, 128, True, False, True): (3, 8, 3, 8),
        (768, 768, 2048, 16, 16, False, True, True): (6, 16, 1, 2),
        (768, 768, 2048, 16, 16, True, False, True): (2, 16, 4, 2),
        (768, 768, 2048, 32, 32, False, True, True): (3, 32, 1, 4),
        (768, 768, 2048, 32, 32, True, False, True): (6, 8, 3, 4),
        (768, 768, 2048, 64, 64, False, True, True): (2, 32, 2, 2),
        (768, 768, 2048, 64, 64, True, False, True): (1, 16, 4, 4),
        (768, 768, 2048, 128, 128, False, True, True): (2, 16, 3, 8),
        (768, 768, 2048, 128, 128, True, False, True): (4, 16, 3, 8),
        (768, 768, 4096, 16, 16, False, True, True): (1, 32, 1, 4),
        (768, 768, 4096, 16, 16, True, False, True): (2, 16, 3, 2),
        (768, 768, 4096, 32, 32, False, True, True): (3, 32, 1, 8),
        (768, 768, 4096, 32, 32, True, False, True): (1, 16, 4, 4),
        (768, 768, 4096, 64, 64, False, True, True): (1, 64, 2, 4),
        (768, 768, 4096, 64, 64, True, False, True): (1, 8, 3, 8),
        (768, 768, 4096, 128, 128, False, True, True): (1, 32, 3, 8),
        (768, 768, 4096, 128, 128, True, False, True): (2, 32, 3, 8),
        (768, 768, 8192, 16, 16, False, True, True): (1, 64, 1, 2),
        (768, 768, 8192, 16, 16, True, False, True): (2, 64, 3, 2),
        (768, 768, 8192, 32, 32, False, True, True): (2, 64, 1, 8),
        (768, 768, 8192, 32, 32, True, False, True): (2, 32, 3, 4),
        (768, 768, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (768, 768, 8192, 64, 64, True, False, True): (1, 64, 3, 4),
        (768, 768, 8192, 128, 128, False, True, True): (4, 64, 3, 8),
        (768, 768, 8192, 128, 128, True, False, True): (2, 64, 3, 8),
        (768, 768, 16384, 16, 16, False, True, True): (4, 128, 1, 2),
        (768, 768, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (768, 768, 16384, 32, 32, False, True, True): (1, 128, 1, 8),
        (768, 768, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (768, 768, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (768, 768, 16384, 64, 64, True, False, True): (1, 128, 3, 4),
        (768, 768, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4279-4314 / 第 4279-4314 行
````python
        (768, 768, 16384, 128, 128, True, False, True): (1, 128, 2, 4),
        (768, 768, 32768, 16, 16, False, True, True): (2, 256, 1, 2),
        (768, 768, 32768, 16, 16, True, False, True): (1, 128, 4, 4),
        (768, 768, 32768, 32, 32, False, True, True): (1, 128, 1, 2),
        (768, 768, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (768, 768, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (768, 768, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (768, 768, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (768, 768, 32768, 128, 128, True, False, True): (3, 256, 2, 4),
        (768, 768, 65536, 16, 16, False, True, True): (4, 512, 1, 2),
        (768, 768, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (768, 768, 65536, 32, 32, False, True, True): (1, 256, 1, 2),
        (768, 768, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (768, 768, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (768, 768, 65536, 64, 64, True, False, True): (1, 256, 3, 4),
        (768, 768, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (768, 768, 65536, 128, 128, True, False, True): (2, 512, 2, 4),
        (768, 768, 131072, 16, 16, False, True, True): (1, 512, 1, 1),
        (768, 768, 131072, 16, 16, True, False, True): (1, 512, 4, 4),
        (768, 768, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
        (768, 768, 131072, 32, 32, True, False, True): (1, 512, 3, 4),
        (768, 768, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (768, 768, 131072, 64, 64, True, False, True): (3, 512, 3, 4),
        (768, 768, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (768, 768, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
        (768, 3072, 256, 16, 16, False, True, True): (1, 8, 5, 2),
        (768, 3072, 256, 16, 16, True, False, True): (3, 4, 7, 2),
        (768, 3072, 256, 32, 32, False, True, True): (1, 8, 4, 2),
        (768, 3072, 256, 32, 32, True, False, True): (1, 4, 5, 4),
        (768, 3072, 256, 64, 64, False, True, True): (1, 4, 3, 4),
        (768, 3072, 256, 64, 64, True, False, True): (1, 4, 5, 4),
        (768, 3072, 256, 128, 128, False, True, True): (2, 2, 3, 8),
        (768, 3072, 256, 128, 128, True, False, True): (2, 2, 3, 8),
        (768, 3072, 512, 16, 16, False, True, True): (1, 8, 5, 2),
        (768, 3072, 512, 16, 16, True, False, True): (1, 8, 5, 2),
        (768, 3072, 512, 32, 32, False, True, True): (3, 8, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4315-4350 / 第 4315-4350 行
````python
        (768, 3072, 512, 32, 32, True, False, True): (1, 8, 7, 4),
        (768, 3072, 512, 64, 64, False, True, True): (3, 8, 3, 4),
        (768, 3072, 512, 64, 64, True, False, True): (3, 8, 5, 4),
        (768, 3072, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (768, 3072, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (768, 3072, 1024, 16, 16, False, True, True): (4, 16, 1, 4),
        (768, 3072, 1024, 16, 16, True, False, True): (2, 8, 5, 2),
        (768, 3072, 1024, 32, 32, False, True, True): (1, 16, 6, 2),
        (768, 3072, 1024, 32, 32, True, False, True): (1, 8, 4, 4),
        (768, 3072, 1024, 64, 64, False, True, True): (2, 16, 4, 4),
        (768, 3072, 1024, 64, 64, True, False, True): (2, 16, 4, 4),
        (768, 3072, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
        (768, 3072, 1024, 128, 128, True, False, True): (3, 8, 3, 8),
        (768, 3072, 2048, 16, 16, False, True, True): (1, 16, 1, 2),
        (768, 3072, 2048, 16, 16, True, False, True): (1, 16, 5, 2),
        (768, 3072, 2048, 32, 32, False, True, True): (4, 16, 1, 8),
        (768, 3072, 2048, 32, 32, True, False, True): (2, 8, 3, 4),
        (768, 3072, 2048, 64, 64, False, True, True): (2, 16, 3, 4),
        (768, 3072, 2048, 64, 64, True, False, True): (2, 16, 3, 4),
        (768, 3072, 2048, 128, 128, False, True, True): (3, 16, 3, 8),
        (768, 3072, 2048, 128, 128, True, False, True): (1, 16, 3, 8),
        (768, 3072, 4096, 16, 16, False, True, True): (1, 32, 1, 4),
        (768, 3072, 4096, 16, 16, True, False, True): (1, 16, 3, 1),
        (768, 3072, 4096, 32, 32, False, True, True): (3, 32, 1, 8),
        (768, 3072, 4096, 32, 32, True, False, True): (2, 16, 3, 8),
        (768, 3072, 4096, 64, 64, False, True, True): (2, 32, 3, 4),
        (768, 3072, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (768, 3072, 4096, 128, 128, False, True, True): (5, 32, 1, 4),
        (768, 3072, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (768, 3072, 8192, 16, 16, False, True, True): (1, 32, 1, 4),
        (768, 3072, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (768, 3072, 8192, 32, 32, False, True, True): (1, 64, 1, 8),
        (768, 3072, 8192, 32, 32, True, False, True): (2, 32, 3, 8),
        (768, 3072, 8192, 64, 64, False, True, True): (2, 64, 3, 4),
        (768, 3072, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (768, 3072, 8192, 128, 128, False, True, True): (1, 64, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4351-4386 / 第 4351-4386 行
````python
        (768, 3072, 8192, 128, 128, True, False, True): (2, 64, 3, 8),
        (768, 3072, 16384, 16, 16, False, True, True): (1, 64, 1, 4),
        (768, 3072, 16384, 16, 16, True, False, True): (1, 64, 4, 1),
        (768, 3072, 16384, 32, 32, False, True, True): (1, 128, 1, 8),
        (768, 3072, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (768, 3072, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (768, 3072, 16384, 64, 64, True, False, True): (1, 64, 3, 4),
        (768, 3072, 16384, 128, 128, False, True, True): (2, 128, 3, 8),
        (768, 3072, 16384, 128, 128, True, False, True): (1, 128, 3, 8),
        (768, 3072, 32768, 16, 16, False, True, True): (1, 128, 1, 4),
        (768, 3072, 32768, 16, 16, True, False, True): (1, 128, 4, 1),
        (768, 3072, 32768, 32, 32, False, True, True): (1, 256, 1, 8),
        (768, 3072, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (768, 3072, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (768, 3072, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (768, 3072, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (768, 3072, 32768, 128, 128, True, False, True): (5, 256, 3, 8),
        (768, 3072, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (768, 3072, 50432, 16, 16, True, False, True): (4, 197, 4, 1),
        (768, 3072, 50432, 32, 32, False, True, True): (2, 197, 1, 4),
        (768, 3072, 50432, 32, 32, True, False, True): (4, 197, 3, 4),
        (768, 3072, 50432, 64, 64, False, True, True): (1, 394, 3, 4),
        (768, 3072, 50432, 64, 64, True, False, True): (1, 197, 3, 4),
        (768, 3072, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (768, 3072, 50432, 128, 128, True, False, True): (3, 394, 2, 4),
        (768, 3072, 65536, 16, 16, False, True, True): (1, 256, 1, 4),
        (768, 3072, 65536, 16, 16, True, False, True): (5, 256, 4, 1),
        (768, 3072, 65536, 32, 32, False, True, True): (2, 256, 1, 4),
        (768, 3072, 65536, 32, 32, True, False, True): (3, 256, 3, 4),
        (768, 3072, 65536, 64, 64, False, True, True): (1, 512, 3, 4),
        (768, 3072, 65536, 64, 64, True, False, True): (1, 256, 3, 4),
        (768, 3072, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (768, 3072, 65536, 128, 128, True, False, True): (2, 512, 3, 8),
        (768, 3072, 131072, 16, 16, False, True, True): (1, 512, 1, 4),
        (768, 3072, 131072, 16, 16, True, False, True): (5, 512, 4, 1),
        (768, 3072, 131072, 32, 32, False, True, True): (2, 512, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4387-4422 / 第 4387-4422 行
````python
        (768, 3072, 131072, 32, 32, True, False, True): (2, 512, 3, 4),
        (768, 3072, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (768, 3072, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (768, 3072, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (768, 3072, 131072, 128, 128, True, False, True): (2, 1024, 3, 8),
        (1024, 1024, 256, 16, 16, False, True, True): (3, 4, 5, 4),
        (1024, 1024, 256, 16, 16, True, False, True): (3, 4, 5, 4),
        (1024, 1024, 256, 32, 32, False, True, True): (2, 4, 6, 2),
        (1024, 1024, 256, 32, 32, True, False, True): (2, 4, 6, 2),
        (1024, 1024, 256, 64, 64, False, True, True): (1, 4, 4, 4),
        (1024, 1024, 256, 64, 64, True, False, True): (2, 4, 6, 4),
        (1024, 1024, 256, 128, 128, False, True, True): (1, 2, 2, 8),
        (1024, 1024, 256, 128, 128, True, False, True): (1, 2, 2, 8),
        (1024, 1024, 512, 16, 16, False, True, True): (3, 4, 5, 4),
        (1024, 1024, 512, 16, 16, True, False, True): (3, 8, 4, 2),
        (1024, 1024, 512, 32, 32, False, True, True): (1, 8, 4, 2),
        (1024, 1024, 512, 32, 32, True, False, True): (1, 8, 4, 2),
        (1024, 1024, 512, 64, 64, False, True, True): (2, 8, 3, 4),
        (1024, 1024, 512, 64, 64, True, False, True): (1, 4, 4, 4),
        (1024, 1024, 512, 128, 128, False, True, True): (7, 4, 2, 8),
        (1024, 1024, 512, 128, 128, True, False, True): (1, 4, 2, 8),
        (1024, 1024, 1024, 16, 16, False, True, True): (4, 8, 4, 2),
        (1024, 1024, 1024, 16, 16, True, False, True): (3, 8, 5, 2),
        (1024, 1024, 1024, 32, 32, False, True, True): (1, 8, 4, 4),
        (1024, 1024, 1024, 32, 32, True, False, True): (1, 8, 4, 2),
        (1024, 1024, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (1024, 1024, 1024, 64, 64, True, False, True): (3, 16, 3, 4),
        (1024, 1024, 1024, 128, 128, False, True, True): (6, 8, 2, 8),
        (1024, 1024, 1024, 128, 128, True, False, True): (4, 8, 2, 8),
        (1024, 1024, 2048, 16, 16, False, True, True): (3, 8, 3, 4),
        (1024, 1024, 2048, 16, 16, True, False, True): (3, 8, 3, 4),
        (1024, 1024, 2048, 32, 32, False, True, True): (1, 16, 3, 4),
        (1024, 1024, 2048, 32, 32, True, False, True): (1, 16, 3, 2),
        (1024, 1024, 2048, 64, 64, False, True, True): (5, 16, 3, 4),
        (1024, 1024, 2048, 64, 64, True, False, True): (5, 16, 3, 4),
        (1024, 1024, 2048, 128, 128, False, True, True): (3, 16, 2, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4423-4458 / 第 4423-4458 行
````python
        (1024, 1024, 2048, 128, 128, True, False, True): (4, 16, 2, 16),
        (1024, 1024, 4096, 16, 16, False, True, True): (4, 32, 3, 2),
        (1024, 1024, 4096, 16, 16, True, False, True): (8, 32, 3, 2),
        (1024, 1024, 4096, 32, 32, False, True, True): (9, 32, 3, 2),
        (1024, 1024, 4096, 32, 32, True, False, True): (1, 32, 3, 2),
        (1024, 1024, 4096, 64, 64, False, True, True): (6, 32, 3, 4),
        (1024, 1024, 4096, 64, 64, True, False, True): (1, 32, 3, 4),
        (1024, 1024, 4096, 128, 128, False, True, True): (4, 32, 2, 8),
        (1024, 1024, 4096, 128, 128, True, False, True): (4, 32, 1, 4),
        (1024, 1024, 8192, 16, 16, False, True, True): (4, 64, 3, 2),
        (1024, 1024, 8192, 16, 16, True, False, True): (4, 64, 3, 2),
        (1024, 1024, 8192, 32, 32, False, True, True): (8, 64, 3, 2),
        (1024, 1024, 8192, 32, 32, True, False, True): (6, 64, 3, 2),
        (1024, 1024, 8192, 64, 64, False, True, True): (2, 64, 3, 4),
        (1024, 1024, 8192, 64, 64, True, False, True): (2, 64, 3, 4),
        (1024, 1024, 8192, 128, 128, False, True, True): (3, 64, 1, 4),
        (1024, 1024, 8192, 128, 128, True, False, True): (2, 64, 1, 4),
        (1024, 1024, 16384, 16, 16, False, True, True): (1, 64, 3, 4),
        (1024, 1024, 16384, 16, 16, True, False, True): (1, 64, 3, 2),
        (1024, 1024, 16384, 32, 32, False, True, True): (1, 128, 3, 4),
        (1024, 1024, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (1024, 1024, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (1024, 1024, 16384, 64, 64, True, False, True): (1, 128, 3, 4),
        (1024, 1024, 16384, 128, 128, False, True, True): (11, 128, 1, 4),
        (1024, 1024, 16384, 128, 128, True, False, True): (4, 128, 1, 4),
        (1024, 1024, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (1024, 1024, 32768, 16, 16, True, False, True): (1, 128, 3, 1),
        (1024, 1024, 32768, 32, 32, False, True, True): (1, 256, 3, 2),
        (1024, 1024, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (1024, 1024, 32768, 64, 64, False, True, True): (2, 128, 2, 4),
        (1024, 1024, 32768, 64, 64, True, False, True): (1, 256, 3, 4),
        (1024, 1024, 32768, 128, 128, False, True, True): (7, 256, 1, 4),
        (1024, 1024, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (1024, 1024, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (1024, 1024, 50432, 16, 16, True, False, True): (4, 197, 3, 4),
        (1024, 1024, 50432, 32, 32, False, True, True): (2, 197, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4459-4494 / 第 4459-4494 行
````python
        (1024, 1024, 50432, 32, 32, True, False, True): (1, 197, 3, 4),
        (1024, 1024, 50432, 64, 64, False, True, True): (2, 394, 1, 4),
        (1024, 1024, 50432, 64, 64, True, False, True): (1, 197, 2, 4),
        (1024, 1024, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (1024, 1024, 50432, 128, 128, True, False, True): (2, 394, 2, 4),
        (1024, 1024, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (1024, 1024, 65536, 16, 16, True, False, True): (1, 256, 3, 1),
        (1024, 1024, 65536, 32, 32, False, True, True): (1, 512, 3, 2),
        (1024, 1024, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (1024, 1024, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (1024, 1024, 65536, 64, 64, True, False, True): (1, 512, 3, 4),
        (1024, 1024, 65536, 128, 128, False, True, True): (10, 512, 1, 4),
        (1024, 1024, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (1024, 1024, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (1024, 1024, 65792, 16, 16, True, False, True): (10, 257, 4, 1),
        (1024, 1024, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (1024, 1024, 65792, 32, 32, True, False, True): (1, 257, 3, 4),
        (1024, 1024, 65792, 64, 64, False, True, True): (2, 514, 1, 4),
        (1024, 1024, 65792, 64, 64, True, False, True): (2, 257, 2, 4),
        (1024, 1024, 65792, 128, 128, False, True, True): (6, 514, 1, 4),
        (1024, 1024, 65792, 128, 128, True, False, True): (2, 514, 2, 4),
        (1024, 1024, 131072, 16, 16, False, True, True): (11, 512, 3, 2),
        (1024, 1024, 131072, 16, 16, True, False, True): (11, 512, 3, 2),
        (1024, 1024, 131072, 32, 32, False, True, True): (7, 1024, 3, 2),
        (1024, 1024, 131072, 32, 32, True, False, True): (6, 512, 3, 4),
        (1024, 1024, 131072, 64, 64, False, True, True): (1, 512, 2, 4),
        (1024, 1024, 131072, 64, 64, True, False, True): (4, 1024, 3, 4),
        (1024, 1024, 131072, 128, 128, False, True, True): (12, 1024, 1, 4),
        (1024, 1024, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (1280, 5120, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (1280, 5120, 65792, 16, 16, True, False, True): (5, 257, 4, 1),
        (1280, 5120, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (1280, 5120, 65792, 32, 32, True, False, True): (2, 257, 3, 4),
        (1280, 5120, 65792, 64, 64, False, True, True): (1, 514, 3, 4),
        (1280, 5120, 65792, 64, 64, True, False, True): (2, 257, 3, 4),
        (1280, 5120, 65792, 128, 128, False, True, True): (1, 514, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4495-4530 / 第 4495-4530 行
````python
        (1280, 5120, 65792, 128, 128, True, False, True): (1, 514, 3, 8),
        (1536, 1536, 256, 16, 16, False, True, True): (5, 4, 4, 2),
        (1536, 1536, 256, 16, 16, True, False, True): (3, 4, 5, 2),
        (1536, 1536, 256, 32, 32, False, True, True): (2, 4, 4, 4),
        (1536, 1536, 256, 32, 32, True, False, True): (1, 4, 6, 2),
        (1536, 1536, 256, 64, 64, False, True, True): (5, 4, 4, 4),
        (1536, 1536, 256, 64, 64, True, False, True): (2, 4, 4, 4),
        (1536, 1536, 256, 128, 128, False, True, True): (1, 2, 3, 8),
        (1536, 1536, 256, 128, 128, True, False, True): (2, 2, 3, 8),
        (1536, 1536, 512, 16, 16, False, True, True): (1, 8, 1, 4),
        (1536, 1536, 512, 16, 16, True, False, True): (3, 4, 4, 2),
        (1536, 1536, 512, 32, 32, False, True, True): (1, 8, 1, 8),
        (1536, 1536, 512, 32, 32, True, False, True): (1, 4, 4, 4),
        (1536, 1536, 512, 64, 64, False, True, True): (3, 8, 3, 4),
        (1536, 1536, 512, 64, 64, True, False, True): (5, 8, 3, 4),
        (1536, 1536, 512, 128, 128, False, True, True): (3, 4, 3, 8),
        (1536, 1536, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (1536, 1536, 1024, 16, 16, False, True, True): (6, 8, 1, 2),
        (1536, 1536, 1024, 16, 16, True, False, True): (2, 8, 5, 2),
        (1536, 1536, 1024, 32, 32, False, True, True): (6, 8, 1, 8),
        (1536, 1536, 1024, 32, 32, True, False, True): (2, 4, 3, 4),
        (1536, 1536, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (1536, 1536, 1024, 64, 64, True, False, True): (3, 8, 3, 4),
        (1536, 1536, 1024, 128, 128, False, True, True): (3, 8, 3, 8),
        (1536, 1536, 1024, 128, 128, True, False, True): (3, 8, 3, 8),
        (1536, 1536, 2048, 16, 16, False, True, True): (1, 16, 1, 4),
        (1536, 1536, 2048, 16, 16, True, False, True): (1, 8, 3, 1),
        (1536, 1536, 2048, 32, 32, False, True, True): (1, 16, 1, 8),
        (1536, 1536, 2048, 32, 32, True, False, True): (4, 8, 3, 2),
        (1536, 1536, 2048, 64, 64, False, True, True): (1, 16, 3, 4),
        (1536, 1536, 2048, 64, 64, True, False, True): (3, 8, 3, 4),
        (1536, 1536, 2048, 128, 128, False, True, True): (6, 16, 1, 4),
        (1536, 1536, 2048, 128, 128, True, False, True): (4, 16, 3, 8),
        (1536, 1536, 4096, 16, 16, False, True, True): (1, 32, 1, 2),
        (1536, 1536, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (1536, 1536, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4531-4566 / 第 4531-4566 行
````python
        (1536, 1536, 4096, 32, 32, True, False, True): (3, 16, 3, 4),
        (1536, 1536, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (1536, 1536, 4096, 64, 64, True, False, True): (1, 16, 3, 4),
        (1536, 1536, 4096, 128, 128, False, True, True): (4, 32, 3, 8),
        (1536, 1536, 4096, 128, 128, True, False, True): (2, 32, 3, 8),
        (1536, 1536, 8192, 16, 16, False, True, True): (2, 64, 1, 2),
        (1536, 1536, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (1536, 1536, 8192, 32, 32, False, True, True): (1, 64, 1, 8),
        (1536, 1536, 8192, 32, 32, True, False, True): (12, 32, 3, 4),
        (1536, 1536, 8192, 64, 64, False, True, True): (2, 64, 3, 4),
        (1536, 1536, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (1536, 1536, 8192, 128, 128, False, True, True): (3, 64, 1, 4),
        (1536, 1536, 8192, 128, 128, True, False, True): (4, 64, 3, 8),
        (1536, 1536, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (1536, 1536, 16384, 16, 16, True, False, True): (1, 64, 4, 4),
        (1536, 1536, 16384, 32, 32, False, True, True): (1, 64, 1, 2),
        (1536, 1536, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (1536, 1536, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (1536, 1536, 16384, 64, 64, True, False, True): (1, 64, 3, 4),
        (1536, 1536, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (1536, 1536, 16384, 128, 128, True, False, True): (1, 128, 2, 4),
        (1536, 1536, 32768, 16, 16, False, True, True): (1, 256, 1, 2),
        (1536, 1536, 32768, 16, 16, True, False, True): (1, 128, 3, 2),
        (1536, 1536, 32768, 32, 32, False, True, True): (1, 128, 1, 2),
        (1536, 1536, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 32768, 64, 64, False, True, True): (3, 256, 3, 4),
        (1536, 1536, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (1536, 1536, 32768, 128, 128, True, False, True): (1, 256, 2, 4),
        (1536, 1536, 65536, 16, 16, False, True, True): (4, 512, 1, 2),
        (1536, 1536, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (1536, 1536, 65536, 32, 32, False, True, True): (1, 256, 1, 2),
        (1536, 1536, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (1536, 1536, 65536, 64, 64, False, True, True): (2, 512, 3, 4),
        (1536, 1536, 65536, 64, 64, True, False, True): (1, 256, 3, 4),
        (1536, 1536, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4567-4602 / 第 4567-4602 行
````python
        (1536, 1536, 65536, 128, 128, True, False, True): (2, 512, 2, 4),
        (1536, 1536, 131072, 16, 16, False, True, True): (2, 1024, 1, 2),
        (1536, 1536, 131072, 16, 16, True, False, True): (9, 512, 4, 4),
        (1536, 1536, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
        (1536, 1536, 131072, 32, 32, True, False, True): (9, 512, 3, 4),
        (1536, 1536, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (1536, 1536, 131072, 64, 64, True, False, True): (1, 512, 3, 4),
        (1536, 1536, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (1536, 1536, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
        (2048, 2048, 256, 16, 16, False, True, True): (4, 4, 6, 2),
        (2048, 2048, 256, 16, 16, True, False, True): (2, 8, 4, 1),
        (2048, 2048, 256, 32, 32, False, True, True): (3, 4, 4, 2),
        (2048, 2048, 256, 32, 32, True, False, True): (1, 4, 5, 2),
        (2048, 2048, 256, 64, 64, False, True, True): (2, 4, 4, 4),
        (2048, 2048, 256, 64, 64, True, False, True): (2, 4, 4, 4),
        (2048, 2048, 256, 128, 128, False, True, True): (3, 2, 2, 8),
        (2048, 2048, 256, 128, 128, True, False, True): (5, 2, 2, 8),
        (2048, 2048, 512, 16, 16, False, True, True): (5, 4, 4, 4),
        (2048, 2048, 512, 16, 16, True, False, True): (2, 4, 4, 2),
        (2048, 2048, 512, 32, 32, False, True, True): (1, 4, 3, 4),
        (2048, 2048, 512, 32, 32, True, False, True): (3, 4, 4, 2),
        (2048, 2048, 512, 64, 64, False, True, True): (1, 8, 3, 4),
        (2048, 2048, 512, 64, 64, True, False, True): (1, 8, 3, 2),
        (2048, 2048, 512, 128, 128, False, True, True): (3, 4, 2, 8),
        (2048, 2048, 512, 128, 128, True, False, True): (2, 4, 2, 8),
        (2048, 2048, 1024, 16, 16, False, True, True): (3, 4, 3, 4),
        (2048, 2048, 1024, 16, 16, True, False, True): (2, 8, 3, 2),
        (2048, 2048, 1024, 32, 32, False, True, True): (3, 8, 3, 4),
        (2048, 2048, 1024, 32, 32, True, False, True): (1, 8, 3, 2),
        (2048, 2048, 1024, 64, 64, False, True, True): (1, 8, 3, 4),
        (2048, 2048, 1024, 64, 64, True, False, True): (1, 8, 3, 4),
        (2048, 2048, 1024, 128, 128, False, True, True): (4, 8, 2, 8),
        (2048, 2048, 1024, 128, 128, True, False, True): (4, 8, 1, 4),
        (2048, 2048, 2048, 16, 16, False, True, True): (4, 16, 3, 2),
        (2048, 2048, 2048, 16, 16, True, False, True): (2, 16, 3, 2),
        (2048, 2048, 2048, 32, 32, False, True, True): (1, 16, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4603-4638 / 第 4603-4638 行
````python
        (2048, 2048, 2048, 32, 32, True, False, True): (1, 16, 3, 2),
        (2048, 2048, 2048, 64, 64, False, True, True): (1, 16, 3, 4),
        (2048, 2048, 2048, 64, 64, True, False, True): (1, 16, 3, 4),
        (2048, 2048, 2048, 128, 128, False, True, True): (6, 16, 2, 8),
        (2048, 2048, 2048, 128, 128, True, False, True): (5, 16, 1, 4),
        (2048, 2048, 4096, 16, 16, False, True, True): (4, 32, 4, 2),
        (2048, 2048, 4096, 16, 16, True, False, True): (4, 32, 3, 2),
        (2048, 2048, 4096, 32, 32, False, True, True): (4, 16, 3, 8),
        (2048, 2048, 4096, 32, 32, True, False, True): (4, 16, 3, 4),
        (2048, 2048, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (2048, 2048, 4096, 64, 64, True, False, True): (4, 32, 3, 4),
        (2048, 2048, 4096, 128, 128, False, True, True): (4, 32, 2, 8),
        (2048, 2048, 4096, 128, 128, True, False, True): (2, 32, 1, 4),
        (2048, 2048, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (2048, 2048, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (2048, 2048, 8192, 32, 32, False, True, True): (4, 32, 3, 8),
        (2048, 2048, 8192, 32, 32, True, False, True): (4, 32, 4, 8),
        (2048, 2048, 8192, 64, 64, False, True, True): (2, 64, 3, 4),
        (2048, 2048, 8192, 64, 64, True, False, True): (4, 64, 3, 4),
        (2048, 2048, 8192, 128, 128, False, True, True): (3, 64, 1, 4),
        (2048, 2048, 8192, 128, 128, True, False, True): (2, 64, 1, 4),
        (2048, 2048, 16384, 16, 16, False, True, True): (4, 64, 3, 4),
        (2048, 2048, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (2048, 2048, 16384, 32, 32, False, True, True): (4, 64, 3, 4),
        (2048, 2048, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (2048, 2048, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (2048, 2048, 16384, 64, 64, True, False, True): (4, 128, 3, 4),
        (2048, 2048, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (2048, 2048, 16384, 128, 128, True, False, True): (2, 128, 1, 4),
        (2048, 2048, 32768, 16, 16, False, True, True): (8, 128, 3, 2),
        (2048, 2048, 32768, 16, 16, True, False, True): (8, 128, 3, 4),
        (2048, 2048, 32768, 32, 32, False, True, True): (8, 128, 3, 4),
        (2048, 2048, 32768, 32, 32, True, False, True): (8, 128, 3, 4),
        (2048, 2048, 32768, 64, 64, False, True, True): (8, 256, 3, 4),
        (2048, 2048, 32768, 64, 64, True, False, True): (8, 256, 3, 4),
        (2048, 2048, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4639-4674 / 第 4639-4674 行
````python
        (2048, 2048, 32768, 128, 128, True, False, True): (1, 256, 1, 4),
        (2048, 2048, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (2048, 2048, 50432, 16, 16, True, False, True): (4, 197, 4, 1),
        (2048, 2048, 50432, 32, 32, False, True, True): (2, 197, 1, 4),
        (2048, 2048, 50432, 32, 32, True, False, True): (4, 197, 3, 4),
        (2048, 2048, 50432, 64, 64, False, True, True): (2, 394, 3, 4),
        (2048, 2048, 50432, 64, 64, True, False, True): (4, 197, 2, 4),
        (2048, 2048, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (2048, 2048, 50432, 128, 128, True, False, True): (4, 394, 2, 4),
        (2048, 2048, 65536, 16, 16, False, True, True): (9, 256, 3, 2),
        (2048, 2048, 65536, 16, 16, True, False, True): (9, 256, 4, 4),
        (2048, 2048, 65536, 32, 32, False, True, True): (7, 256, 3, 4),
        (2048, 2048, 65536, 32, 32, True, False, True): (7, 256, 3, 4),
        (2048, 2048, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (2048, 2048, 65536, 64, 64, True, False, True): (9, 512, 3, 4),
        (2048, 2048, 65536, 128, 128, False, True, True): (5, 512, 1, 4),
        (2048, 2048, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (2048, 2048, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (2048, 2048, 65792, 16, 16, True, False, True): (7, 257, 4, 1),
        (2048, 2048, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (2048, 2048, 65792, 32, 32, True, False, True): (7, 257, 3, 4),
        (2048, 2048, 65792, 64, 64, False, True, True): (1, 514, 3, 4),
        (2048, 2048, 65792, 64, 64, True, False, True): (1, 257, 2, 4),
        (2048, 2048, 65792, 128, 128, False, True, True): (3, 514, 1, 4),
        (2048, 2048, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (2048, 2048, 131072, 16, 16, False, True, True): (9, 512, 3, 2),
        (2048, 2048, 131072, 16, 16, True, False, True): (9, 512, 4, 4),
        (2048, 2048, 131072, 32, 32, False, True, True): (7, 512, 3, 4),
        (2048, 2048, 131072, 32, 32, True, False, True): (3, 512, 3, 4),
        (2048, 2048, 131072, 64, 64, False, True, True): (1, 512, 2, 4),
        (2048, 2048, 131072, 64, 64, True, False, True): (2, 1024, 3, 4),
        (2048, 2048, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (2048, 2048, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (3072, 768, 256, 16, 16, False, True, True): (6, 4, 1, 4),
        (3072, 768, 256, 16, 16, True, False, True): (2, 1, 5, 2),
        (3072, 768, 256, 32, 32, False, True, True): (1, 4, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4675-4710 / 第 4675-4710 行
````python
        (3072, 768, 256, 32, 32, True, False, True): (4, 2, 4, 4),
        (3072, 768, 256, 64, 64, False, True, True): (1, 2, 3, 4),
        (3072, 768, 256, 64, 64, True, False, True): (3, 4, 3, 4),
        (3072, 768, 256, 128, 128, False, True, True): (1, 2, 3, 8),
        (3072, 768, 256, 128, 128, True, False, True): (3, 2, 3, 8),
        (3072, 768, 512, 16, 16, False, True, True): (1, 4, 1, 4),
        (3072, 768, 512, 16, 16, True, False, True): (3, 4, 4, 1),
        (3072, 768, 512, 32, 32, False, True, True): (5, 8, 1, 4),
        (3072, 768, 512, 32, 32, True, False, True): (3, 4, 4, 2),
        (3072, 768, 512, 64, 64, False, True, True): (1, 8, 1, 4),
        (3072, 768, 512, 64, 64, True, False, True): (1, 4, 3, 4),
        (3072, 768, 512, 128, 128, False, True, True): (3, 4, 3, 8),
        (3072, 768, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (3072, 768, 1024, 16, 16, False, True, True): (1, 8, 1, 4),
        (3072, 768, 1024, 16, 16, True, False, True): (3, 4, 3, 1),
        (3072, 768, 1024, 32, 32, False, True, True): (1, 16, 1, 4),
        (3072, 768, 1024, 32, 32, True, False, True): (1, 4, 3, 8),
        (3072, 768, 1024, 64, 64, False, True, True): (8, 16, 3, 2),
        (3072, 768, 1024, 64, 64, True, False, True): (1, 4, 3, 4),
        (3072, 768, 1024, 128, 128, False, True, True): (2, 8, 3, 8),
        (3072, 768, 1024, 128, 128, True, False, True): (3, 8, 2, 4),
        (3072, 768, 2048, 16, 16, False, True, True): (1, 8, 1, 4),
        (3072, 768, 2048, 16, 16, True, False, True): (6, 8, 4, 4),
        (3072, 768, 2048, 32, 32, False, True, True): (1, 16, 1, 8),
        (3072, 768, 2048, 32, 32, True, False, True): (6, 8, 3, 4),
        (3072, 768, 2048, 64, 64, False, True, True): (8, 16, 3, 4),
        (3072, 768, 2048, 64, 64, True, False, True): (3, 16, 3, 4),
        (3072, 768, 2048, 128, 128, False, True, True): (1, 16, 3, 8),
        (3072, 768, 2048, 128, 128, True, False, True): (2, 16, 2, 4),
        (3072, 768, 4096, 16, 16, False, True, True): (1, 16, 1, 4),
        (3072, 768, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (3072, 768, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
        (3072, 768, 4096, 32, 32, True, False, True): (4, 16, 3, 4),
        (3072, 768, 4096, 64, 64, False, True, True): (2, 32, 1, 4),
        (3072, 768, 4096, 64, 64, True, False, True): (2, 16, 2, 4),
        (3072, 768, 4096, 128, 128, False, True, True): (2, 32, 1, 16),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4711-4746 / 第 4711-4746 行
````python
        (3072, 768, 4096, 128, 128, True, False, True): (3, 32, 2, 4),
        (3072, 768, 8192, 16, 16, False, True, True): (2, 32, 1, 4),
        (3072, 768, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (3072, 768, 8192, 32, 32, False, True, True): (2, 32, 1, 4),
        (3072, 768, 8192, 32, 32, True, False, True): (6, 32, 3, 4),
        (3072, 768, 8192, 64, 64, False, True, True): (2, 64, 1, 4),
        (3072, 768, 8192, 64, 64, True, False, True): (2, 32, 2, 4),
        (3072, 768, 8192, 128, 128, False, True, True): (3, 64, 1, 4),
        (3072, 768, 8192, 128, 128, True, False, True): (2, 64, 2, 4),
        (3072, 768, 16384, 16, 16, False, True, True): (1, 64, 1, 4),
        (3072, 768, 16384, 16, 16, True, False, True): (1, 64, 1, 1),
        (3072, 768, 16384, 32, 32, False, True, True): (2, 64, 1, 4),
        (3072, 768, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (3072, 768, 16384, 64, 64, False, True, True): (2, 128, 1, 4),
        (3072, 768, 16384, 64, 64, True, False, True): (4, 64, 2, 4),
        (3072, 768, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (3072, 768, 16384, 128, 128, True, False, True): (1, 128, 2, 4),
        (3072, 768, 32768, 16, 16, False, True, True): (1, 128, 1, 4),
        (3072, 768, 32768, 16, 16, True, False, True): (8, 256, 3, 2),
        (3072, 768, 32768, 32, 32, False, True, True): (2, 128, 1, 4),
        (3072, 768, 32768, 32, 32, True, False, True): (8, 128, 3, 4),
        (3072, 768, 32768, 64, 64, False, True, True): (1, 256, 1, 4),
        (3072, 768, 32768, 64, 64, True, False, True): (8, 128, 2, 4),
        (3072, 768, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (3072, 768, 32768, 128, 128, True, False, True): (3, 256, 2, 4),
        (3072, 768, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (3072, 768, 50432, 16, 16, True, False, True): (7, 197, 4, 1),
        (3072, 768, 50432, 32, 32, False, True, True): (2, 197, 1, 4),
        (3072, 768, 50432, 32, 32, True, False, True): (10, 197, 3, 4),
        (3072, 768, 50432, 64, 64, False, True, True): (1, 394, 1, 4),
        (3072, 768, 50432, 64, 64, True, False, True): (3, 197, 2, 4),
        (3072, 768, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (3072, 768, 50432, 128, 128, True, False, True): (2, 394, 2, 4),
        (3072, 768, 65536, 16, 16, False, True, True): (1, 256, 1, 4),
        (3072, 768, 65536, 16, 16, True, False, True): (15, 256, 4, 1),
        (3072, 768, 65536, 32, 32, False, True, True): (2, 256, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4747-4782 / 第 4747-4782 行
````python
        (3072, 768, 65536, 32, 32, True, False, True): (10, 256, 3, 4),
        (3072, 768, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (3072, 768, 65536, 64, 64, True, False, True): (3, 256, 2, 4),
        (3072, 768, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (3072, 768, 65536, 128, 128, True, False, True): (3, 512, 2, 4),
        (3072, 768, 131072, 16, 16, False, True, True): (1, 512, 1, 4),
        (3072, 768, 131072, 16, 16, True, False, True): (15, 512, 4, 1),
        (3072, 768, 131072, 32, 32, False, True, True): (2, 512, 1, 4),
        (3072, 768, 131072, 32, 32, True, False, True): (9, 512, 3, 4),
        (3072, 768, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (3072, 768, 131072, 64, 64, True, False, True): (3, 512, 2, 4),
        (3072, 768, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (3072, 768, 131072, 128, 128, True, False, True): (3, 1024, 2, 4),
        (3072, 3072, 256, 16, 16, False, True, True): (5, 4, 1, 4),
        (3072, 3072, 256, 16, 16, True, False, True): (1, 2, 5, 2),
        (3072, 3072, 256, 32, 32, False, True, True): (1, 4, 1, 8),
        (3072, 3072, 256, 32, 32, True, False, True): (3, 4, 4, 2),
        (3072, 3072, 256, 64, 64, False, True, True): (2, 4, 3, 4),
        (3072, 3072, 256, 64, 64, True, False, True): (3, 4, 4, 4),
        (3072, 3072, 256, 128, 128, False, True, True): (1, 2, 3, 8),
        (3072, 3072, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (3072, 3072, 512, 16, 16, False, True, True): (5, 4, 1, 2),
        (3072, 3072, 512, 16, 16, True, False, True): (1, 2, 4, 4),
        (3072, 3072, 512, 32, 32, False, True, True): (3, 8, 1, 4),
        (3072, 3072, 512, 32, 32, True, False, True): (4, 2, 3, 4),
        (3072, 3072, 512, 64, 64, False, True, True): (1, 8, 2, 2),
        (3072, 3072, 512, 64, 64, True, False, True): (2, 4, 3, 4),
        (3072, 3072, 512, 128, 128, False, True, True): (1, 4, 3, 8),
        (3072, 3072, 512, 128, 128, True, False, True): (4, 4, 3, 8),
        (3072, 3072, 1024, 16, 16, False, True, True): (1, 8, 1, 4),
        (3072, 3072, 1024, 16, 16, True, False, True): (4, 8, 5, 2),
        (3072, 3072, 1024, 32, 32, False, True, True): (1, 8, 1, 8),
        (3072, 3072, 1024, 32, 32, True, False, True): (1, 4, 4, 4),
        (3072, 3072, 1024, 64, 64, False, True, True): (3, 8, 3, 4),
        (3072, 3072, 1024, 64, 64, True, False, True): (2, 4, 3, 4),
        (3072, 3072, 1024, 128, 128, False, True, True): (3, 8, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4783-4818 / 第 4783-4818 行
````python
        (3072, 3072, 1024, 128, 128, True, False, True): (1, 8, 3, 8),
        (3072, 3072, 2048, 16, 16, False, True, True): (1, 16, 1, 2),
        (3072, 3072, 2048, 16, 16, True, False, True): (4, 16, 4, 2),
        (3072, 3072, 2048, 32, 32, False, True, True): (1, 16, 1, 8),
        (3072, 3072, 2048, 32, 32, True, False, True): (3, 8, 4, 4),
        (3072, 3072, 2048, 64, 64, False, True, True): (3, 16, 3, 4),
        (3072, 3072, 2048, 64, 64, True, False, True): (3, 8, 3, 4),
        (3072, 3072, 2048, 128, 128, False, True, True): (4, 16, 3, 8),
        (3072, 3072, 2048, 128, 128, True, False, True): (3, 16, 3, 8),
        (3072, 3072, 4096, 16, 16, False, True, True): (1, 32, 1, 2),
        (3072, 3072, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (3072, 3072, 4096, 32, 32, False, True, True): (1, 32, 1, 8),
        (3072, 3072, 4096, 32, 32, True, False, True): (3, 16, 3, 4),
        (3072, 3072, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (3072, 3072, 4096, 64, 64, True, False, True): (3, 16, 3, 4),
        (3072, 3072, 4096, 128, 128, False, True, True): (1, 32, 3, 8),
        (3072, 3072, 4096, 128, 128, True, False, True): (3, 32, 3, 8),
        (3072, 3072, 8192, 16, 16, False, True, True): (1, 64, 1, 2),
        (3072, 3072, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (3072, 3072, 8192, 32, 32, False, True, True): (1, 64, 1, 8),
        (3072, 3072, 8192, 32, 32, True, False, True): (8, 32, 3, 4),
        (3072, 3072, 8192, 64, 64, False, True, True): (3, 64, 3, 4),
        (3072, 3072, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (3072, 3072, 8192, 128, 128, False, True, True): (2, 64, 3, 8),
        (3072, 3072, 8192, 128, 128, True, False, True): (1, 64, 3, 8),
        (3072, 3072, 16384, 16, 16, False, True, True): (1, 128, 1, 2),
        (3072, 3072, 16384, 16, 16, True, False, True): (4, 128, 4, 2),
        (3072, 3072, 16384, 32, 32, False, True, True): (1, 64, 1, 2),
        (3072, 3072, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (3072, 3072, 16384, 64, 64, False, True, True): (1, 128, 3, 4),
        (3072, 3072, 16384, 64, 64, True, False, True): (4, 64, 3, 4),
        (3072, 3072, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (3072, 3072, 16384, 128, 128, True, False, True): (1, 128, 3, 8),
        (3072, 3072, 32768, 16, 16, False, True, True): (1, 256, 1, 2),
        (3072, 3072, 32768, 16, 16, True, False, True): (8, 128, 4, 4),
        (3072, 3072, 32768, 32, 32, False, True, True): (1, 256, 1, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4819-4854 / 第 4819-4854 行
````python
        (3072, 3072, 32768, 32, 32, True, False, True): (5, 128, 3, 4),
        (3072, 3072, 32768, 64, 64, False, True, True): (1, 256, 3, 4),
        (3072, 3072, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (3072, 3072, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (3072, 3072, 32768, 128, 128, True, False, True): (3, 256, 2, 4),
        (3072, 3072, 65536, 16, 16, False, True, True): (1, 512, 1, 2),
        (3072, 3072, 65536, 16, 16, True, False, True): (7, 256, 4, 4),
        (3072, 3072, 65536, 32, 32, False, True, True): (1, 256, 1, 2),
        (3072, 3072, 65536, 32, 32, True, False, True): (5, 256, 3, 4),
        (3072, 3072, 65536, 64, 64, False, True, True): (1, 512, 3, 4),
        (3072, 3072, 65536, 64, 64, True, False, True): (3, 256, 3, 4),
        (3072, 3072, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (3072, 3072, 65536, 128, 128, True, False, True): (3, 512, 2, 4),
        (3072, 3072, 131072, 16, 16, False, True, True): (1, 1024, 1, 2),
        (3072, 3072, 131072, 16, 16, True, False, True): (5, 512, 4, 4),
        (3072, 3072, 131072, 32, 32, False, True, True): (1, 512, 1, 2),
        (3072, 3072, 131072, 32, 32, True, False, True): (3, 512, 3, 4),
        (3072, 3072, 131072, 64, 64, False, True, True): (1, 1024, 3, 4),
        (3072, 3072, 131072, 64, 64, True, False, True): (3, 512, 3, 4),
        (3072, 3072, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (3072, 3072, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
        (4096, 4096, 256, 16, 16, False, True, True): (2, 2, 6, 4),
        (4096, 4096, 256, 16, 16, True, False, True): (2, 2, 5, 4),
        (4096, 4096, 256, 32, 32, False, True, True): (7, 2, 4, 4),
        (4096, 4096, 256, 32, 32, True, False, True): (1, 2, 4, 4),
        (4096, 4096, 256, 64, 64, False, True, True): (3, 4, 3, 4),
        (4096, 4096, 256, 64, 64, True, False, True): (3, 4, 3, 4),
        (4096, 4096, 256, 128, 128, False, True, True): (1, 2, 2, 8),
        (4096, 4096, 256, 128, 128, True, False, True): (1, 2, 2, 8),
        (4096, 4096, 512, 16, 16, False, True, True): (4, 2, 3, 4),
        (4096, 4096, 512, 16, 16, True, False, True): (2, 4, 3, 2),
        (4096, 4096, 512, 32, 32, False, True, True): (3, 4, 3, 4),
        (4096, 4096, 512, 32, 32, True, False, True): (3, 4, 3, 2),
        (4096, 4096, 512, 64, 64, False, True, True): (3, 4, 3, 4),
        (4096, 4096, 512, 64, 64, True, False, True): (3, 4, 3, 4),
        (4096, 4096, 512, 128, 128, False, True, True): (2, 4, 2, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4855-4890 / 第 4855-4890 行
````python
        (4096, 4096, 512, 128, 128, True, False, True): (2, 4, 1, 4),
        (4096, 4096, 1024, 16, 16, False, True, True): (2, 8, 3, 2),
        (4096, 4096, 1024, 16, 16, True, False, True): (2, 8, 3, 2),
        (4096, 4096, 1024, 32, 32, False, True, True): (3, 8, 3, 4),
        (4096, 4096, 1024, 32, 32, True, False, True): (1, 8, 3, 2),
        (4096, 4096, 1024, 64, 64, False, True, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 64, 64, True, False, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 128, 128, False, True, True): (2, 8, 2, 8),
        (4096, 4096, 1024, 128, 128, True, False, True): (2, 8, 2, 8),
        (4096, 4096, 2048, 16, 16, False, True, True): (2, 8, 4, 4),
        (4096, 4096, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (4096, 4096, 2048, 32, 32, False, True, True): (4, 8, 4, 8),
        (4096, 4096, 2048, 32, 32, True, False, True): (4, 8, 4, 8),
        (4096, 4096, 2048, 64, 64, False, True, True): (1, 16, 3, 4),
        (4096, 4096, 2048, 64, 64, True, False, True): (4, 16, 3, 4),
        (4096, 4096, 2048, 128, 128, False, True, True): (2, 16, 2, 8),
        (4096, 4096, 2048, 128, 128, True, False, True): (4, 16, 1, 4),
        (4096, 4096, 4096, 16, 16, False, True, True): (4, 32, 4, 4),
        (4096, 4096, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (4096, 4096, 4096, 32, 32, False, True, True): (4, 16, 4, 8),
        (4096, 4096, 4096, 32, 32, True, False, True): (4, 16, 3, 8),
        (4096, 4096, 4096, 64, 64, False, True, True): (1, 32, 3, 4),
        (4096, 4096, 4096, 64, 64, True, False, True): (1, 32, 3, 4),
        (4096, 4096, 4096, 128, 128, False, True, True): (3, 32, 1, 4),
        (4096, 4096, 4096, 128, 128, True, False, True): (2, 32, 1, 4),
        (4096, 4096, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (4096, 4096, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (4096, 4096, 8192, 32, 32, False, True, True): (4, 32, 4, 8),
        (4096, 4096, 8192, 32, 32, True, False, True): (4, 32, 4, 8),
        (4096, 4096, 8192, 64, 64, False, True, True): (2, 64, 3, 4),
        (4096, 4096, 8192, 64, 64, True, False, True): (2, 64, 3, 4),
        (4096, 4096, 8192, 128, 128, False, True, True): (3, 64, 1, 4),
        (4096, 4096, 8192, 128, 128, True, False, True): (1, 64, 1, 4),
        (4096, 4096, 16384, 16, 16, False, True, True): (4, 64, 3, 4),
        (4096, 4096, 16384, 16, 16, True, False, True): (4, 64, 4, 4),
        (4096, 4096, 16384, 32, 32, False, True, True): (4, 64, 4, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4891-4926 / 第 4891-4926 行
````python
        (4096, 4096, 16384, 32, 32, True, False, True): (4, 64, 4, 8),
        (4096, 4096, 16384, 64, 64, False, True, True): (1, 64, 2, 4),
        (4096, 4096, 16384, 64, 64, True, False, True): (1, 64, 3, 8),
        (4096, 4096, 16384, 128, 128, False, True, True): (3, 128, 1, 4),
        (4096, 4096, 16384, 128, 128, True, False, True): (1, 128, 1, 4),
        (4096, 4096, 32768, 16, 16, False, True, True): (8, 128, 3, 2),
        (4096, 4096, 32768, 16, 16, True, False, True): (5, 128, 4, 4),
        (4096, 4096, 32768, 32, 32, False, True, True): (3, 128, 4, 4),
        (4096, 4096, 32768, 32, 32, True, False, True): (3, 128, 4, 8),
        (4096, 4096, 32768, 64, 64, False, True, True): (1, 128, 2, 4),
        (4096, 4096, 32768, 64, 64, True, False, True): (3, 256, 3, 4),
        (4096, 4096, 32768, 128, 128, False, True, True): (3, 256, 1, 4),
        (4096, 4096, 32768, 128, 128, True, False, True): (1, 256, 1, 4),
        (4096, 4096, 50432, 16, 16, False, True, True): (1, 197, 1, 4),
        (4096, 4096, 50432, 16, 16, True, False, True): (4, 197, 4, 1),
        (4096, 4096, 50432, 32, 32, False, True, True): (1, 197, 1, 4),
        (4096, 4096, 50432, 32, 32, True, False, True): (2, 197, 3, 4),
        (4096, 4096, 50432, 64, 64, False, True, True): (1, 394, 3, 4),
        (4096, 4096, 50432, 64, 64, True, False, True): (1, 197, 2, 4),
        (4096, 4096, 50432, 128, 128, False, True, True): (3, 394, 1, 4),
        (4096, 4096, 50432, 128, 128, True, False, True): (1, 394, 2, 4),
        (4096, 4096, 65536, 16, 16, False, True, True): (5, 256, 4, 4),
        (4096, 4096, 65536, 16, 16, True, False, True): (5, 256, 4, 4),
        (4096, 4096, 65536, 32, 32, False, True, True): (4, 256, 4, 8),
        (4096, 4096, 65536, 32, 32, True, False, True): (4, 256, 3, 8),
        (4096, 4096, 65536, 64, 64, False, True, True): (1, 256, 2, 4),
        (4096, 4096, 65536, 64, 64, True, False, True): (1, 512, 3, 4),
        (4096, 4096, 65536, 128, 128, False, True, True): (3, 512, 1, 4),
        (4096, 4096, 65536, 128, 128, True, False, True): (1, 512, 1, 4),
        (4096, 4096, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (4096, 4096, 65792, 16, 16, True, False, True): (5, 257, 4, 1),
        (4096, 4096, 65792, 32, 32, False, True, True): (1, 257, 1, 4),
        (4096, 4096, 65792, 32, 32, True, False, True): (1, 257, 3, 4),
        (4096, 4096, 65792, 64, 64, False, True, True): (1, 514, 3, 4),
        (4096, 4096, 65792, 64, 64, True, False, True): (1, 257, 2, 4),
        (4096, 4096, 65792, 128, 128, False, True, True): (3, 514, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4927-4962 / 第 4927-4962 行
````python
        (4096, 4096, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (4096, 4096, 131072, 16, 16, False, True, True): (4, 512, 3, 4),
        (4096, 4096, 131072, 16, 16, True, False, True): (5, 512, 4, 4),
        (4096, 4096, 131072, 32, 32, False, True, True): (1, 512, 4, 8),
        (4096, 4096, 131072, 32, 32, True, False, True): (4, 512, 4, 8),
        (4096, 4096, 131072, 64, 64, False, True, True): (1, 512, 2, 4),
        (4096, 4096, 131072, 64, 64, True, False, True): (1, 512, 2, 4),
        (4096, 4096, 131072, 128, 128, False, True, True): (3, 1024, 1, 4),
        (4096, 4096, 131072, 128, 128, True, False, True): (1, 1024, 1, 4),
        (5120, 1280, 65792, 16, 16, False, True, True): (1, 257, 1, 4),
        (5120, 1280, 65792, 16, 16, True, False, True): (7, 257, 4, 1),
        (5120, 1280, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (5120, 1280, 65792, 32, 32, True, False, True): (5, 257, 3, 4),
        (5120, 1280, 65792, 64, 64, False, True, True): (1, 514, 1, 4),
        (5120, 1280, 65792, 64, 64, True, False, True): (5, 257, 2, 4),
        (5120, 1280, 65792, 128, 128, False, True, True): (3, 514, 1, 4),
        (5120, 1280, 65792, 128, 128, True, False, True): (4, 514, 2, 4),
        (6144, 6144, 256, 16, 16, False, True, True): (1, 2, 1, 4),
        (6144, 6144, 256, 16, 16, True, False, True): (1, 1, 4, 4),
        (6144, 6144, 256, 32, 32, False, True, True): (3, 2, 1, 8),
        (6144, 6144, 256, 32, 32, True, False, True): (2, 1, 3, 4),
        (6144, 6144, 256, 64, 64, False, True, True): (2, 2, 3, 4),
        (6144, 6144, 256, 64, 64, True, False, True): (6, 2, 4, 4),
        (6144, 6144, 256, 128, 128, False, True, True): (2, 2, 3, 8),
        (6144, 6144, 256, 128, 128, True, False, True): (1, 2, 3, 8),
        (6144, 6144, 512, 16, 16, False, True, True): (4, 4, 1, 4),
        (6144, 6144, 512, 16, 16, True, False, True): (3, 2, 3, 1),
        (6144, 6144, 512, 32, 32, False, True, True): (1, 8, 1, 4),
        (6144, 6144, 512, 32, 32, True, False, True): (2, 2, 3, 8),
        (6144, 6144, 512, 64, 64, False, True, True): (4, 4, 3, 4),
        (6144, 6144, 512, 64, 64, True, False, True): (6, 2, 3, 4),
        (6144, 6144, 512, 128, 128, False, True, True): (3, 4, 1, 4),
        (6144, 6144, 512, 128, 128, True, False, True): (4, 4, 3, 8),
        (6144, 6144, 1024, 16, 16, False, True, True): (1, 8, 1, 2),
        (6144, 6144, 1024, 16, 16, True, False, True): (4, 8, 4, 2),
        (6144, 6144, 1024, 32, 32, False, True, True): (1, 8, 4, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4963-4998 / 第 4963-4998 行
````python
        (6144, 6144, 1024, 32, 32, True, False, True): (1, 8, 4, 2),
        (6144, 6144, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (6144, 6144, 1024, 64, 64, True, False, True): (1, 4, 3, 4),
        (6144, 6144, 1024, 128, 128, False, True, True): (3, 8, 1, 4),
        (6144, 6144, 1024, 128, 128, True, False, True): (1, 8, 3, 8),
        (6144, 6144, 2048, 16, 16, False, True, True): (4, 4, 1, 4),
        (6144, 6144, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (6144, 6144, 2048, 32, 32, False, True, True): (4, 8, 3, 4),
        (6144, 6144, 2048, 32, 32, True, False, True): (2, 8, 3, 4),
        (6144, 6144, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (6144, 6144, 2048, 64, 64, True, False, True): (2, 8, 3, 4),
        (6144, 6144, 2048, 128, 128, False, True, True): (3, 16, 1, 4),
        (6144, 6144, 2048, 128, 128, True, False, True): (4, 16, 3, 8),
        (6144, 6144, 4096, 16, 16, False, True, True): (4, 8, 1, 4),
        (6144, 6144, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (6144, 6144, 4096, 32, 32, False, True, True): (4, 16, 1, 2),
        (6144, 6144, 4096, 32, 32, True, False, True): (2, 8, 3, 8),
        (6144, 6144, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (6144, 6144, 4096, 64, 64, True, False, True): (4, 16, 3, 4),
        (6144, 6144, 4096, 128, 128, False, True, True): (6, 32, 1, 4),
        (6144, 6144, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (6144, 6144, 8192, 16, 16, False, True, True): (2, 16, 1, 2),
        (6144, 6144, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (6144, 6144, 8192, 32, 32, False, True, True): (4, 32, 1, 2),
        (6144, 6144, 8192, 32, 32, True, False, True): (4, 32, 3, 4),
        (6144, 6144, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (6144, 6144, 8192, 64, 64, True, False, True): (4, 32, 3, 4),
        (6144, 6144, 8192, 128, 128, False, True, True): (6, 64, 1, 4),
        (6144, 6144, 8192, 128, 128, True, False, True): (4, 64, 3, 8),
        (6144, 6144, 16384, 16, 16, False, True, True): (2, 32, 1, 2),
        (6144, 6144, 16384, 16, 16, True, False, True): (4, 64, 4, 4),
        (6144, 6144, 16384, 32, 32, False, True, True): (4, 64, 1, 2),
        (6144, 6144, 16384, 32, 32, True, False, True): (4, 64, 3, 4),
        (6144, 6144, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (6144, 6144, 16384, 64, 64, True, False, True): (1, 32, 3, 8),
        (6144, 6144, 16384, 128, 128, False, True, True): (4, 128, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 4999-5034 / 第 4999-5034 行
````python
        (6144, 6144, 16384, 128, 128, True, False, True): (4, 128, 3, 8),
        (6144, 6144, 32768, 16, 16, False, True, True): (2, 64, 1, 2),
        (6144, 6144, 32768, 16, 16, True, False, True): (5, 128, 4, 1),
        (6144, 6144, 32768, 32, 32, False, True, True): (4, 128, 1, 2),
        (6144, 6144, 32768, 32, 32, True, False, True): (3, 128, 3, 4),
        (6144, 6144, 32768, 64, 64, False, True, True): (4, 256, 3, 4),
        (6144, 6144, 32768, 64, 64, True, False, True): (2, 64, 3, 8),
        (6144, 6144, 32768, 128, 128, False, True, True): (8, 256, 1, 4),
        (6144, 6144, 32768, 128, 128, True, False, True): (4, 256, 3, 8),
        (6144, 6144, 65536, 16, 16, False, True, True): (2, 128, 1, 2),
        (6144, 6144, 65536, 16, 16, True, False, True): (5, 256, 4, 1),
        (6144, 6144, 65536, 32, 32, False, True, True): (4, 256, 1, 2),
        (6144, 6144, 65536, 32, 32, True, False, True): (2, 256, 3, 4),
        (6144, 6144, 65536, 64, 64, False, True, True): (4, 512, 3, 4),
        (6144, 6144, 65536, 64, 64, True, False, True): (1, 128, 3, 8),
        (6144, 6144, 65536, 128, 128, False, True, True): (4, 512, 1, 4),
        (6144, 6144, 65536, 128, 128, True, False, True): (4, 512, 3, 8),
        (6144, 6144, 131072, 16, 16, False, True, True): (2, 256, 1, 2),
        (6144, 6144, 131072, 16, 16, True, False, True): (3, 512, 4, 4),
        (6144, 6144, 131072, 32, 32, False, True, True): (4, 512, 1, 2),
        (6144, 6144, 131072, 32, 32, True, False, True): (4, 512, 3, 4),
        (6144, 6144, 131072, 64, 64, False, True, True): (4, 1024, 3, 4),
        (6144, 6144, 131072, 64, 64, True, False, True): (2, 256, 3, 8),
        (6144, 6144, 131072, 128, 128, False, True, True): (4, 1024, 1, 4),
        (6144, 6144, 131072, 128, 128, True, False, True): (4, 1024, 3, 8),
        (8192, 8192, 256, 16, 16, False, True, True): (2, 2, 6, 4),
        (8192, 8192, 256, 16, 16, True, False, True): (2, 4, 2, 2),
        (8192, 8192, 256, 32, 32, False, True, True): (4, 2, 3, 4),
        (8192, 8192, 256, 32, 32, True, False, True): (4, 2, 3, 4),
        (8192, 8192, 256, 64, 64, False, True, True): (2, 2, 3, 8),
        (8192, 8192, 256, 64, 64, True, False, True): (6, 2, 3, 8),
        (8192, 8192, 256, 128, 128, False, True, True): (3, 2, 1, 4),
        (8192, 8192, 256, 128, 128, True, False, True): (1, 2, 1, 4),
        (8192, 8192, 512, 16, 16, False, True, True): (4, 4, 3, 2),
        (8192, 8192, 512, 16, 16, True, False, True): (4, 4, 3, 4),
        (8192, 8192, 512, 32, 32, False, True, True): (1, 4, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5035-5070 / 第 5035-5070 行
````python
        (8192, 8192, 512, 32, 32, True, False, True): (5, 4, 3, 2),
        (8192, 8192, 512, 64, 64, False, True, True): (1, 4, 3, 4),
        (8192, 8192, 512, 64, 64, True, False, True): (2, 2, 3, 8),
        (8192, 8192, 512, 128, 128, False, True, True): (4, 4, 2, 8),
        (8192, 8192, 512, 128, 128, True, False, True): (4, 4, 2, 8),
        (8192, 8192, 1024, 16, 16, False, True, True): (4, 8, 4, 4),
        (8192, 8192, 1024, 16, 16, True, False, True): (4, 8, 4, 4),
        (8192, 8192, 1024, 32, 32, False, True, True): (2, 4, 4, 8),
        (8192, 8192, 1024, 32, 32, True, False, True): (1, 4, 3, 4),
        (8192, 8192, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (8192, 8192, 1024, 64, 64, True, False, True): (2, 8, 3, 4),
        (8192, 8192, 1024, 128, 128, False, True, True): (4, 8, 2, 8),
        (8192, 8192, 1024, 128, 128, True, False, True): (4, 8, 1, 4),
        (8192, 8192, 2048, 16, 16, False, True, True): (2, 8, 4, 4),
        (8192, 8192, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (8192, 8192, 2048, 32, 32, False, True, True): (2, 8, 4, 8),
        (8192, 8192, 2048, 32, 32, True, False, True): (2, 8, 4, 8),
        (8192, 8192, 2048, 64, 64, False, True, True): (4, 8, 2, 4),
        (8192, 8192, 2048, 64, 64, True, False, True): (4, 16, 3, 4),
        (8192, 8192, 2048, 128, 128, False, True, True): (6, 16, 1, 4),
        (8192, 8192, 2048, 128, 128, True, False, True): (4, 16, 1, 4),
        (8192, 8192, 4096, 16, 16, False, True, True): (4, 32, 4, 2),
        (8192, 8192, 4096, 16, 16, True, False, True): (4, 32, 4, 2),
        (8192, 8192, 4096, 32, 32, False, True, True): (2, 16, 4, 8),
        (8192, 8192, 4096, 32, 32, True, False, True): (4, 16, 4, 8),
        (8192, 8192, 4096, 64, 64, False, True, True): (4, 16, 2, 4),
        (8192, 8192, 4096, 64, 64, True, False, True): (4, 16, 2, 4),
        (8192, 8192, 4096, 128, 128, False, True, True): (6, 32, 1, 4),
        (8192, 8192, 4096, 128, 128, True, False, True): (4, 32, 1, 4),
        (8192, 8192, 8192, 16, 16, False, True, True): (4, 64, 4, 2),
        (8192, 8192, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (8192, 8192, 8192, 32, 32, False, True, True): (2, 32, 4, 8),
        (8192, 8192, 8192, 32, 32, True, False, True): (2, 32, 4, 8),
        (8192, 8192, 8192, 64, 64, False, True, True): (2, 32, 2, 4),
        (8192, 8192, 8192, 64, 64, True, False, True): (4, 32, 2, 4),
        (8192, 8192, 8192, 128, 128, False, True, True): (6, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5071-5106 / 第 5071-5106 行
````python
        (8192, 8192, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (8192, 8192, 16384, 16, 16, False, True, True): (4, 64, 3, 4),
        (8192, 8192, 16384, 16, 16, True, False, True): (4, 64, 4, 4),
        (8192, 8192, 16384, 32, 32, False, True, True): (4, 64, 4, 8),
        (8192, 8192, 16384, 32, 32, True, False, True): (4, 64, 4, 8),
        (8192, 8192, 16384, 64, 64, False, True, True): (4, 64, 2, 4),
        (8192, 8192, 16384, 64, 64, True, False, True): (4, 64, 3, 8),
        (8192, 8192, 16384, 128, 128, False, True, True): (6, 128, 1, 4),
        (8192, 8192, 16384, 128, 128, True, False, True): (4, 128, 1, 4),
        (8192, 8192, 32768, 16, 16, False, True, True): (3, 128, 4, 4),
        (8192, 8192, 32768, 16, 16, True, False, True): (3, 128, 4, 4),
        (8192, 8192, 32768, 32, 32, False, True, True): (2, 128, 4, 8),
        (8192, 8192, 32768, 32, 32, True, False, True): (2, 128, 4, 8),
        (8192, 8192, 32768, 64, 64, False, True, True): (2, 128, 2, 4),
        (8192, 8192, 32768, 64, 64, True, False, True): (2, 128, 3, 8),
        (8192, 8192, 32768, 128, 128, False, True, True): (6, 256, 1, 4),
        (8192, 8192, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (8192, 8192, 50432, 16, 16, False, True, True): (1, 197, 1, 1),
        (8192, 8192, 50432, 16, 16, True, False, True): (3, 197, 4, 1),
        (8192, 8192, 50432, 32, 32, False, True, True): (2, 197, 1, 4),
        (8192, 8192, 50432, 32, 32, True, False, True): (2, 197, 3, 4),
        (8192, 8192, 50432, 64, 64, False, True, True): (2, 394, 3, 4),
        (8192, 8192, 65536, 16, 16, False, True, True): (3, 256, 4, 4),
        (8192, 8192, 65536, 16, 16, True, False, True): (4, 256, 4, 4),
        (8192, 8192, 65536, 32, 32, False, True, True): (2, 256, 4, 8),
        (8192, 8192, 65536, 32, 32, True, False, True): (2, 256, 3, 8),
        (8192, 8192, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (8192, 8192, 65536, 64, 64, True, False, True): (4, 256, 3, 8),
        (8192, 8192, 65536, 128, 128, False, True, True): (6, 512, 1, 4),
        (8192, 8192, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (8192, 8192, 65792, 16, 16, False, True, True): (1, 257, 1, 1),
        (8192, 8192, 65792, 16, 16, True, False, True): (3, 257, 4, 1),
        (8192, 8192, 65792, 32, 32, False, True, True): (2, 257, 1, 4),
        (8192, 8192, 65792, 32, 32, True, False, True): (1, 257, 3, 4),
        (8192, 8192, 65792, 64, 64, False, True, True): (2, 514, 3, 4),
        (8192, 8192, 65792, 64, 64, True, False, True): (1, 257, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5107-5142 / 第 5107-5142 行
````python
        (8192, 8192, 65792, 128, 128, False, True, True): (2, 514, 1, 4),
        (8192, 8192, 65792, 128, 128, True, False, True): (2, 514, 3, 8),
        (8192, 8192, 131072, 16, 16, False, True, True): (4, 512, 4, 4),
        (8192, 8192, 131072, 16, 16, True, False, True): (3, 512, 4, 4),
        (8192, 8192, 131072, 32, 32, False, True, True): (2, 512, 4, 8),
        (8192, 8192, 131072, 32, 32, True, False, True): (2, 512, 4, 8),
        (8192, 8192, 131072, 64, 64, False, True, True): (2, 512, 2, 4),
        (8192, 8192, 131072, 64, 64, True, False, True): (2, 512, 2, 4),
        (8192, 8192, 131072, 128, 128, False, True, True): (4, 1024, 1, 4),
        (8192, 8192, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (12288, 12288, 256, 16, 16, False, True, True): (4, 2, 1, 4),
        (12288, 12288, 256, 16, 16, True, False, True): (1, 1, 3, 1),
        (12288, 12288, 256, 32, 32, False, True, True): (4, 4, 1, 4),
        (12288, 12288, 256, 32, 32, True, False, True): (2, 1, 3, 2),
        (12288, 12288, 256, 64, 64, False, True, True): (4, 2, 3, 4),
        (12288, 12288, 256, 64, 64, True, False, True): (3, 1, 3, 4),
        (12288, 12288, 256, 128, 128, False, True, True): (6, 2, 1, 4),
        (12288, 12288, 256, 128, 128, True, False, True): (4, 2, 3, 8),
        (12288, 12288, 512, 16, 16, False, True, True): (4, 4, 1, 2),
        (12288, 12288, 512, 16, 16, True, False, True): (4, 4, 4, 2),
        (12288, 12288, 512, 32, 32, False, True, True): (4, 4, 4, 2),
        (12288, 12288, 512, 32, 32, True, False, True): (2, 2, 3, 8),
        (12288, 12288, 512, 64, 64, False, True, True): (4, 4, 3, 4),
        (12288, 12288, 512, 64, 64, True, False, True): (8, 2, 3, 4),
        (12288, 12288, 512, 128, 128, False, True, True): (4, 4, 3, 8),
        (12288, 12288, 512, 128, 128, True, False, True): (4, 4, 3, 8),
        (12288, 12288, 1024, 16, 16, False, True, True): (4, 8, 1, 2),
        (12288, 12288, 1024, 16, 16, True, False, True): (2, 4, 4, 4),
        (12288, 12288, 1024, 32, 32, False, True, True): (4, 4, 3, 4),
        (12288, 12288, 1024, 32, 32, True, False, True): (1, 4, 3, 4),
        (12288, 12288, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (12288, 12288, 1024, 64, 64, True, False, True): (2, 4, 3, 4),
        (12288, 12288, 1024, 128, 128, False, True, True): (4, 8, 3, 8),
        (12288, 12288, 1024, 128, 128, True, False, True): (4, 8, 3, 8),
        (12288, 12288, 2048, 16, 16, False, True, True): (2, 4, 1, 4),
        (12288, 12288, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5143-5178 / 第 5143-5178 行
````python
        (12288, 12288, 2048, 32, 32, False, True, True): (4, 8, 1, 2),
        (12288, 12288, 2048, 32, 32, True, False, True): (2, 8, 4, 8),
        (12288, 12288, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (12288, 12288, 2048, 64, 64, True, False, True): (2, 8, 3, 4),
        (12288, 12288, 2048, 128, 128, False, True, True): (4, 16, 3, 8),
        (12288, 12288, 2048, 128, 128, True, False, True): (4, 16, 3, 8),
        (12288, 12288, 4096, 16, 16, False, True, True): (2, 8, 1, 4),
        (12288, 12288, 4096, 16, 16, True, False, True): (2, 16, 4, 4),
        (12288, 12288, 4096, 32, 32, False, True, True): (2, 16, 1, 2),
        (12288, 12288, 4096, 32, 32, True, False, True): (2, 16, 3, 4),
        (12288, 12288, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (12288, 12288, 4096, 64, 64, True, False, True): (2, 16, 3, 4),
        (12288, 12288, 4096, 128, 128, False, True, True): (4, 32, 1, 4),
        (12288, 12288, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (12288, 12288, 8192, 16, 16, False, True, True): (2, 32, 1, 1),
        (12288, 12288, 8192, 16, 16, True, False, True): (4, 64, 4, 2),
        (12288, 12288, 8192, 32, 32, False, True, True): (2, 32, 1, 2),
        (12288, 12288, 8192, 32, 32, True, False, True): (2, 32, 3, 2),
        (12288, 12288, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (12288, 12288, 8192, 64, 64, True, False, True): (2, 32, 3, 4),
        (12288, 12288, 8192, 128, 128, False, True, True): (4, 64, 3, 8),
        (12288, 12288, 8192, 128, 128, True, False, True): (2, 64, 3, 8),
        (12288, 12288, 16384, 16, 16, False, True, True): (4, 128, 1, 2),
        (12288, 12288, 16384, 16, 16, True, False, True): (4, 128, 4, 2),
        (12288, 12288, 16384, 32, 32, False, True, True): (2, 64, 1, 2),
        (12288, 12288, 16384, 32, 32, True, False, True): (2, 64, 3, 4),
        (12288, 12288, 16384, 64, 64, False, True, True): (4, 128, 3, 4),
        (12288, 12288, 16384, 64, 64, True, False, True): (2, 64, 3, 4),
        (12288, 12288, 16384, 128, 128, False, True, True): (4, 128, 1, 4),
        (12288, 12288, 16384, 128, 128, True, False, True): (4, 128, 3, 8),
        (12288, 12288, 32768, 16, 16, False, True, True): (2, 128, 1, 1),
        (12288, 12288, 32768, 16, 16, True, False, True): (3, 128, 4, 1),
        (12288, 12288, 32768, 32, 32, False, True, True): (2, 128, 1, 2),
        (12288, 12288, 32768, 32, 32, True, False, True): (2, 128, 3, 2),
        (12288, 12288, 32768, 64, 64, False, True, True): (4, 256, 3, 4),
        (12288, 12288, 32768, 64, 64, True, False, True): (1, 64, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5179-5214 / 第 5179-5214 行
````python
        (12288, 12288, 32768, 128, 128, False, True, True): (4, 256, 3, 8),
        (12288, 12288, 32768, 128, 128, True, False, True): (4, 256, 3, 8),
        (12288, 12288, 65536, 16, 16, False, True, True): (4, 512, 1, 2),
        (12288, 12288, 65536, 16, 16, True, False, True): (3, 256, 4, 1),
        (12288, 12288, 65536, 32, 32, False, True, True): (2, 256, 1, 2),
        (12288, 12288, 65536, 32, 32, True, False, True): (2, 256, 3, 2),
        (12288, 12288, 65536, 64, 64, False, True, True): (4, 512, 3, 4),
        (12288, 12288, 65536, 64, 64, True, False, True): (2, 256, 3, 4),
        (12288, 12288, 65536, 128, 128, False, True, True): (4, 512, 1, 4),
        (12288, 12288, 65536, 128, 128, True, False, True): (4, 512, 3, 8),
        (12288, 12288, 131072, 16, 16, False, True, True): (2, 512, 1, 1),
        (12288, 12288, 131072, 16, 16, True, False, True): (2, 512, 4, 4),
        (12288, 12288, 131072, 32, 32, False, True, True): (2, 512, 1, 2),
        (12288, 12288, 131072, 32, 32, True, False, True): (2, 512, 3, 4),
        (12288, 12288, 131072, 64, 64, False, True, True): (4, 1024, 3, 4),
        (12288, 12288, 131072, 64, 64, True, False, True): (2, 512, 3, 4),
        (12288, 12288, 131072, 128, 128, False, True, True): (4, 1024, 3, 8),
        (12288, 12288, 131072, 128, 128, True, False, True): (4, 1024, 3, 8),
        (16384, 16384, 256, 16, 16, False, True, True): (2, 2, 3, 2),
        (16384, 16384, 256, 16, 16, True, False, True): (2, 2, 6, 4),
        (16384, 16384, 256, 32, 32, False, True, True): (4, 2, 3, 4),
        (16384, 16384, 256, 32, 32, True, False, True): (4, 2, 3, 2),
        (16384, 16384, 256, 64, 64, False, True, True): (2, 2, 5, 4),
        (16384, 16384, 256, 64, 64, True, False, True): (2, 2, 3, 8),
        (16384, 16384, 256, 128, 128, False, True, True): (4, 2, 2, 8),
        (16384, 16384, 256, 128, 128, True, False, True): (2, 2, 1, 4),
        (16384, 16384, 512, 16, 16, False, True, True): (1, 2, 4, 4),
        (16384, 16384, 512, 16, 16, True, False, True): (1, 2, 4, 4),
        (16384, 16384, 512, 32, 32, False, True, True): (2, 2, 3, 8),
        (16384, 16384, 512, 32, 32, True, False, True): (2, 2, 4, 8),
        (16384, 16384, 512, 64, 64, False, True, True): (4, 4, 3, 4),
        (16384, 16384, 512, 64, 64, True, False, True): (2, 4, 3, 4),
        (16384, 16384, 512, 128, 128, False, True, True): (4, 4, 2, 8),
        (16384, 16384, 512, 128, 128, True, False, True): (4, 4, 2, 8),
        (16384, 16384, 1024, 16, 16, False, True, True): (4, 8, 4, 4),
        (16384, 16384, 1024, 16, 16, True, False, True): (2, 4, 4, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5215-5250 / 第 5215-5250 行
````python
        (16384, 16384, 1024, 32, 32, False, True, True): (2, 4, 4, 8),
        (16384, 16384, 1024, 32, 32, True, False, True): (2, 4, 4, 8),
        (16384, 16384, 1024, 64, 64, False, True, True): (4, 4, 2, 4),
        (16384, 16384, 1024, 64, 64, True, False, True): (2, 4, 2, 4),
        (16384, 16384, 1024, 128, 128, False, True, True): (6, 8, 1, 4),
        (16384, 16384, 1024, 128, 128, True, False, True): (4, 8, 1, 4),
        (16384, 16384, 2048, 16, 16, False, True, True): (2, 8, 4, 4),
        (16384, 16384, 2048, 16, 16, True, False, True): (2, 8, 4, 4),
        (16384, 16384, 2048, 32, 32, False, True, True): (2, 8, 4, 8),
        (16384, 16384, 2048, 32, 32, True, False, True): (2, 8, 4, 8),
        (16384, 16384, 2048, 64, 64, False, True, True): (2, 8, 2, 4),
        (16384, 16384, 2048, 64, 64, True, False, True): (2, 8, 2, 4),
        (16384, 16384, 2048, 128, 128, False, True, True): (4, 16, 2, 8),
        (16384, 16384, 2048, 128, 128, True, False, True): (4, 16, 1, 4),
        (16384, 16384, 4096, 16, 16, False, True, True): (2, 16, 4, 4),
        (16384, 16384, 4096, 16, 16, True, False, True): (2, 16, 4, 4),
        (16384, 16384, 4096, 32, 32, False, True, True): (1, 16, 4, 8),
        (16384, 16384, 4096, 32, 32, True, False, True): (2, 16, 3, 4),
        (16384, 16384, 4096, 64, 64, False, True, True): (1, 16, 2, 4),
        (16384, 16384, 4096, 64, 64, True, False, True): (2, 16, 2, 4),
        (16384, 16384, 4096, 128, 128, False, True, True): (4, 32, 2, 8),
        (16384, 16384, 4096, 128, 128, True, False, True): (4, 32, 1, 4),
        (16384, 16384, 8192, 16, 16, False, True, True): (2, 64, 4, 2),
        (16384, 16384, 8192, 16, 16, True, False, True): (2, 64, 4, 2),
        (16384, 16384, 8192, 32, 32, False, True, True): (2, 32, 4, 8),
        (16384, 16384, 8192, 32, 32, True, False, True): (2, 32, 4, 8),
        (16384, 16384, 8192, 64, 64, False, True, True): (2, 32, 2, 4),
        (16384, 16384, 8192, 64, 64, True, False, True): (2, 32, 4, 8),
        (16384, 16384, 8192, 128, 128, False, True, True): (4, 64, 2, 8),
        (16384, 16384, 8192, 128, 128, True, False, True): (4, 64, 1, 4),
        (16384, 16384, 16384, 16, 16, False, True, True): (1, 64, 4, 4),
        (16384, 16384, 16384, 16, 16, True, False, True): (1, 64, 4, 4),
        (16384, 16384, 16384, 32, 32, False, True, True): (1, 64, 4, 8),
        (16384, 16384, 16384, 32, 32, True, False, True): (1, 64, 4, 8),
        (16384, 16384, 16384, 64, 64, False, True, True): (1, 64, 2, 4),
        (16384, 16384, 16384, 64, 64, True, False, True): (1, 64, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5251-5286 / 第 5251-5286 行
````python
        (16384, 16384, 16384, 128, 128, False, True, True): (4, 128, 1, 4),
        (16384, 16384, 16384, 128, 128, True, False, True): (4, 128, 1, 4),
        (16384, 16384, 32768, 16, 16, False, True, True): (1, 128, 4, 4),
        (16384, 16384, 32768, 16, 16, True, False, True): (1, 128, 4, 4),
        (16384, 16384, 32768, 32, 32, False, True, True): (1, 128, 3, 4),
        (16384, 16384, 32768, 32, 32, True, False, True): (1, 128, 3, 8),
        (16384, 16384, 32768, 64, 64, False, True, True): (2, 128, 2, 4),
        (16384, 16384, 32768, 64, 64, True, False, True): (1, 128, 4, 8),
        (16384, 16384, 32768, 128, 128, False, True, True): (4, 256, 2, 8),
        (16384, 16384, 32768, 128, 128, True, False, True): (4, 256, 1, 4),
        (16384, 16384, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (16384, 16384, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (16384, 16384, 65536, 32, 32, False, True, True): (1, 256, 4, 8),
        (16384, 16384, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (16384, 16384, 65536, 64, 64, False, True, True): (2, 256, 2, 4),
        (16384, 16384, 65536, 64, 64, True, False, True): (1, 256, 3, 8),
        (16384, 16384, 65536, 128, 128, False, True, True): (4, 512, 2, 8),
        (16384, 16384, 65536, 128, 128, True, False, True): (4, 512, 1, 4),
        (16384, 16384, 65792, 16, 16, False, True, True): (1, 257, 1, 1),
        (16384, 16384, 65792, 16, 16, True, False, True): (1, 257, 4, 1),
        (16384, 16384, 65792, 32, 32, False, True, True): (1, 257, 1, 4),
        (16384, 16384, 65792, 32, 32, True, False, True): (1, 257, 3, 4),
        (16384, 16384, 65792, 64, 64, False, True, True): (2, 514, 3, 4),
        (16384, 16384, 65792, 64, 64, True, False, True): (1, 257, 3, 4),
        (16384, 16384, 65792, 128, 128, False, True, True): (2, 514, 3, 8),
        (16384, 16384, 65792, 128, 128, True, False, True): (2, 514, 3, 8),
        (16384, 16384, 131072, 16, 16, False, True, True): (1, 512, 4, 4),
        (16384, 16384, 131072, 16, 16, True, False, True): (1, 512, 3, 2),
        (16384, 16384, 131072, 32, 32, False, True, True): (1, 512, 4, 8),
        (16384, 16384, 131072, 32, 32, True, False, True): (1, 512, 3, 2),
        (16384, 16384, 131072, 64, 64, False, True, True): (1, 512, 2, 4),
        (16384, 16384, 131072, 64, 64, True, False, True): (1, 512, 2, 4),
        (16384, 16384, 131072, 128, 128, False, True, True): (4, 1024, 1, 4),
        (16384, 16384, 131072, 128, 128, True, False, True): (4, 1024, 1, 4),
        (24576, 24576, 256, 16, 16, False, True, True): (6, 2, 1, 2),
        (24576, 24576, 256, 16, 16, True, False, True): (2, 2, 5, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5287-5322 / 第 5287-5322 行
````python
        (24576, 24576, 256, 32, 32, False, True, True): (4, 4, 1, 4),
        (24576, 24576, 256, 32, 32, True, False, True): (2, 2, 4, 2),
        (24576, 24576, 256, 64, 64, False, True, True): (2, 2, 3, 4),
        (24576, 24576, 256, 64, 64, True, False, True): (1, 1, 3, 4),
        (24576, 24576, 256, 128, 128, False, True, True): (6, 2, 1, 4),
        (24576, 24576, 256, 128, 128, True, False, True): (2, 2, 3, 8),
        (24576, 24576, 512, 16, 16, False, True, True): (4, 4, 1, 2),
        (24576, 24576, 512, 16, 16, True, False, True): (2, 2, 4, 4),
        (24576, 24576, 512, 32, 32, False, True, True): (1, 2, 3, 4),
        (24576, 24576, 512, 32, 32, True, False, True): (1, 2, 3, 4),
        (24576, 24576, 512, 64, 64, False, True, True): (4, 4, 3, 4),
        (24576, 24576, 512, 64, 64, True, False, True): (1, 2, 3, 4),
        (24576, 24576, 512, 128, 128, False, True, True): (4, 4, 3, 8),
        (24576, 24576, 512, 128, 128, True, False, True): (4, 4, 3, 8),
        (24576, 24576, 1024, 16, 16, False, True, True): (2, 8, 1, 2),
        (24576, 24576, 1024, 16, 16, True, False, True): (2, 4, 4, 4),
        (24576, 24576, 1024, 32, 32, False, True, True): (2, 4, 1, 2),
        (24576, 24576, 1024, 32, 32, True, False, True): (1, 4, 3, 4),
        (24576, 24576, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (24576, 24576, 1024, 64, 64, True, False, True): (1, 4, 3, 4),
        (24576, 24576, 1024, 128, 128, False, True, True): (4, 8, 3, 8),
        (24576, 24576, 1024, 128, 128, True, False, True): (4, 8, 3, 8),
        (24576, 24576, 2048, 16, 16, False, True, True): (1, 4, 1, 4),
        (24576, 24576, 2048, 16, 16, True, False, True): (1, 8, 4, 4),
        (24576, 24576, 2048, 32, 32, False, True, True): (2, 8, 1, 2),
        (24576, 24576, 2048, 32, 32, True, False, True): (1, 8, 3, 4),
        (24576, 24576, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (24576, 24576, 2048, 64, 64, True, False, True): (1, 4, 3, 8),
        (24576, 24576, 2048, 128, 128, False, True, True): (4, 16, 3, 8),
        (24576, 24576, 2048, 128, 128, True, False, True): (2, 16, 3, 8),
        (24576, 24576, 4096, 16, 16, False, True, True): (2, 32, 1, 2),
        (24576, 24576, 4096, 16, 16, True, False, True): (1, 16, 4, 4),
        (24576, 24576, 4096, 32, 32, False, True, True): (1, 16, 1, 2),
        (24576, 24576, 4096, 32, 32, True, False, True): (1, 16, 3, 4),
        (24576, 24576, 4096, 64, 64, False, True, True): (4, 32, 3, 4),
        (24576, 24576, 4096, 64, 64, True, False, True): (1, 8, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5323-5358 / 第 5323-5358 行
````python
        (24576, 24576, 4096, 128, 128, False, True, True): (4, 32, 3, 8),
        (24576, 24576, 4096, 128, 128, True, False, True): (2, 32, 3, 8),
        (24576, 24576, 8192, 16, 16, False, True, True): (1, 32, 1, 1),
        (24576, 24576, 8192, 16, 16, True, False, True): (2, 64, 4, 2),
        (24576, 24576, 8192, 32, 32, False, True, True): (1, 32, 1, 2),
        (24576, 24576, 8192, 32, 32, True, False, True): (1, 32, 3, 4),
        (24576, 24576, 8192, 64, 64, False, True, True): (4, 64, 3, 4),
        (24576, 24576, 8192, 64, 64, True, False, True): (1, 32, 3, 4),
        (24576, 24576, 8192, 128, 128, False, True, True): (4, 64, 3, 8),
        (24576, 24576, 8192, 128, 128, True, False, True): (4, 64, 3, 8),
        (24576, 24576, 16384, 16, 16, False, True, True): (2, 128, 1, 2),
        (24576, 24576, 16384, 16, 16, True, False, True): (1, 64, 4, 4),
        (24576, 24576, 16384, 32, 32, False, True, True): (1, 64, 1, 2),
        (24576, 24576, 16384, 32, 32, True, False, True): (1, 64, 3, 2),
        (24576, 24576, 16384, 64, 64, False, True, True): (2, 128, 3, 4),
        (24576, 24576, 16384, 64, 64, True, False, True): (1, 32, 3, 8),
        (24576, 24576, 16384, 128, 128, False, True, True): (4, 128, 3, 8),
        (24576, 24576, 16384, 128, 128, True, False, True): (4, 128, 3, 8),
        (24576, 24576, 32768, 16, 16, False, True, True): (1, 128, 1, 1),
        (24576, 24576, 32768, 16, 16, True, False, True): (1, 128, 4, 4),
        (24576, 24576, 32768, 32, 32, False, True, True): (1, 128, 1, 2),
        (24576, 24576, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (24576, 24576, 32768, 64, 64, False, True, True): (2, 256, 3, 4),
        (24576, 24576, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (24576, 24576, 32768, 128, 128, False, True, True): (4, 256, 3, 8),
        (24576, 24576, 32768, 128, 128, True, False, True): (2, 256, 3, 8),
        (24576, 24576, 65536, 16, 16, False, True, True): (2, 512, 1, 2),
        (24576, 24576, 65536, 16, 16, True, False, True): (1, 256, 4, 4),
        (32768, 32768, 256, 16, 16, False, True, True): (4, 2, 1, 2),
        (32768, 32768, 256, 16, 16, True, False, True): (2, 2, 5, 4),
        (32768, 32768, 256, 32, 32, False, True, True): (4, 2, 4, 2),
        (32768, 32768, 256, 32, 32, True, False, True): (1, 1, 4, 8),
        (32768, 32768, 256, 64, 64, False, True, True): (2, 2, 3, 4),
        (32768, 32768, 256, 64, 64, True, False, True): (1, 1, 3, 8),
        (32768, 32768, 256, 128, 128, False, True, True): (2, 2, 3, 8),
        (32768, 32768, 256, 128, 128, True, False, True): (2, 2, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5359-5394 / 第 5359-5394 行
````python
        (32768, 32768, 512, 16, 16, False, True, True): (2, 2, 1, 4),
        (32768, 32768, 512, 16, 16, True, False, True): (2, 2, 4, 2),
        (32768, 32768, 512, 32, 32, False, True, True): (1, 2, 3, 4),
        (32768, 32768, 512, 32, 32, True, False, True): (1, 2, 4, 8),
        (32768, 32768, 512, 64, 64, False, True, True): (4, 4, 3, 4),
        (32768, 32768, 512, 64, 64, True, False, True): (1, 2, 3, 4),
        (32768, 32768, 512, 128, 128, False, True, True): (4, 4, 3, 8),
        (32768, 32768, 512, 128, 128, True, False, True): (4, 4, 3, 8),
        (32768, 32768, 1024, 16, 16, False, True, True): (2, 4, 1, 1),
        (32768, 32768, 1024, 16, 16, True, False, True): (1, 4, 4, 2),
        (32768, 32768, 1024, 32, 32, False, True, True): (2, 4, 1, 4),
        (32768, 32768, 1024, 32, 32, True, False, True): (1, 4, 3, 4),
        (32768, 32768, 1024, 64, 64, False, True, True): (4, 8, 3, 4),
        (32768, 32768, 1024, 64, 64, True, False, True): (1, 4, 3, 4),
        (32768, 32768, 1024, 128, 128, False, True, True): (4, 8, 3, 8),
        (32768, 32768, 1024, 128, 128, True, False, True): (4, 8, 3, 8),
        (32768, 32768, 2048, 16, 16, False, True, True): (1, 8, 1, 4),
        (32768, 32768, 2048, 16, 16, True, False, True): (1, 8, 4, 4),
        (32768, 32768, 2048, 32, 32, False, True, True): (2, 8, 1, 4),
        (32768, 32768, 2048, 32, 32, True, False, True): (1, 8, 3, 4),
        (32768, 32768, 2048, 64, 64, False, True, True): (4, 16, 3, 4),
        (32768, 32768, 2048, 64, 64, True, False, True): (1, 8, 3, 4),
        (32768, 32768, 2048, 128, 128, False, True, True): (4, 16, 3, 8),
        (32768, 32768, 2048, 128, 128, True, False, True): (2, 16, 3, 8),
        (32768, 32768, 4096, 16, 16, False, True, True): (1, 16, 1, 4),
        (32768, 32768, 4096, 16, 16, True, False, True): (1, 16, 4, 4),
        (32768, 32768, 4096, 32, 32, False, True, True): (2, 16, 1, 4),
        (32768, 32768, 4096, 32, 32, True, False, True): (1, 16, 3, 4),
        (32768, 32768, 4096, 64, 64, False, True, True): (2, 32, 3, 4),
        (32768, 32768, 4096, 64, 64, True, False, True): (1, 16, 3, 4),
        (32768, 32768, 4096, 128, 128, False, True, True): (4, 32, 3, 8),
        (32768, 32768, 4096, 128, 128, True, False, True): (4, 32, 3, 8),
        (32768, 32768, 8192, 16, 16, False, True, True): (1, 32, 1, 4),
        (32768, 32768, 8192, 16, 16, True, False, True): (2, 64, 4, 1),
        (32768, 32768, 8192, 32, 32, False, True, True): (2, 32, 1, 4),
        (32768, 32768, 8192, 32, 32, True, False, True): (1, 32, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5395-5430 / 第 5395-5430 行
````python
        (32768, 32768, 8192, 64, 64, False, True, True): (2, 64, 3, 4),
        (32768, 32768, 8192, 64, 64, True, False, True): (1, 32, 3, 4),
        (32768, 32768, 8192, 128, 128, False, True, True): (4, 64, 3, 8),
        (32768, 32768, 8192, 128, 128, True, False, True): (2, 64, 3, 8),
        (32768, 32768, 16384, 16, 16, False, True, True): (1, 64, 1, 4),
        (32768, 32768, 16384, 16, 16, True, False, True): (1, 64, 4, 1),
        (32768, 32768, 16384, 32, 32, False, True, True): (2, 64, 1, 4),
        (32768, 32768, 16384, 32, 32, True, False, True): (1, 64, 3, 4),
        (32768, 32768, 16384, 64, 64, False, True, True): (2, 128, 3, 4),
        (32768, 32768, 16384, 64, 64, True, False, True): (1, 64, 3, 4),
        (32768, 32768, 16384, 128, 128, False, True, True): (4, 128, 3, 8),
        (32768, 32768, 16384, 128, 128, True, False, True): (2, 128, 3, 8),
        (32768, 32768, 32768, 16, 16, False, True, True): (1, 128, 1, 4),
        (32768, 32768, 32768, 16, 16, True, False, True): (1, 128, 4, 1),
        (32768, 32768, 32768, 32, 32, False, True, True): (2, 128, 1, 4),
        (32768, 32768, 32768, 32, 32, True, False, True): (1, 128, 3, 4),
        (32768, 32768, 32768, 64, 64, False, True, True): (2, 256, 3, 4),
        (32768, 32768, 32768, 64, 64, True, False, True): (1, 128, 3, 4),
        (32768, 32768, 32768, 128, 128, False, True, True): (2, 256, 3, 8),
        (32768, 32768, 32768, 128, 128, True, False, True): (4, 256, 3, 8),
        (32768, 32768, 65536, 16, 16, False, True, True): (1, 256, 1, 4),
        (32768, 32768, 65536, 16, 16, True, False, True): (1, 256, 4, 1),
        (32768, 32768, 65536, 32, 32, False, True, True): (1, 256, 3, 4),
        (32768, 32768, 65536, 32, 32, True, False, True): (1, 256, 3, 4),
        (32768, 32768, 65536, 64, 64, False, True, True): (1, 512, 3, 4),
        (32768, 32768, 65536, 64, 64, True, False, True): (1, 256, 3, 4),
        (32768, 32768, 65536, 128, 128, False, True, True): (4, 512, 1, 4),
        (32768, 32768, 65536, 128, 128, True, False, True): (2, 512, 3, 8),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.float16, 0.56)): {
        (192, 192, 256, 64, 64, False, True, True): (1, 4, 3, 4),
        (192, 192, 256, 64, 64, True, False, True): (1, 4, 3, 4),
        (192, 192, 512, 64, 64, False, True, True): (1, 8, 5, 4),
        (192, 192, 512, 64, 64, True, False, True): (1, 8, 3, 4),
        (192, 192, 1024, 64, 64, False, True, True): (1, 16, 3, 2),
        (192, 192, 1024, 64, 64, True, False, True): (1, 16, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5431-5466 / 第 5431-5466 行
````python
        (192, 192, 2048, 64, 64, False, True, True): (1, 32, 5, 4),
        (192, 192, 2048, 64, 64, True, False, True): (4, 32, 5, 4),
        (192, 192, 4096, 64, 64, False, True, True): (1, 64, 1, 8),
        (192, 192, 4096, 64, 64, True, False, True): (1, 32, 3, 4),
        (192, 192, 8192, 64, 64, False, True, True): (4, 128, 1, 4),
        (192, 192, 8192, 64, 64, True, False, True): (3, 64, 3, 4),
        (192, 192, 16384, 64, 64, False, True, True): (1, 256, 1, 4),
        (192, 192, 16384, 64, 64, True, False, True): (3, 64, 2, 4),
        (192, 192, 32768, 64, 64, False, True, True): (1, 512, 1, 2),
        (192, 192, 32768, 64, 64, True, False, True): (2, 256, 2, 4),
        (192, 192, 65536, 64, 64, False, True, True): (1, 512, 1, 4),
        (192, 192, 65536, 64, 64, True, False, True): (2, 512, 2, 4),
        (192, 192, 131072, 64, 64, False, True, True): (1, 1024, 1, 4),
        (192, 192, 131072, 64, 64, True, False, True): (1, 512, 3, 4),
        (384, 384, 256, 128, 128, False, True, True): (3, 2, 3, 8),
        (384, 384, 256, 128, 128, True, False, True): (5, 2, 3, 8),
        (384, 384, 512, 128, 128, False, True, True): (4, 4, 3, 8),
        (384, 384, 512, 128, 128, True, False, True): (1, 4, 3, 8),
        (384, 384, 1024, 128, 128, False, True, True): (1, 8, 3, 8),
        (384, 384, 1024, 128, 128, True, False, True): (1, 8, 2, 8),
        (384, 384, 2048, 128, 128, False, True, True): (3, 16, 3, 8),
        (384, 384, 2048, 128, 128, True, False, True): (1, 16, 3, 8),
        (384, 384, 4096, 128, 128, False, True, True): (3, 32, 3, 8),
        (384, 384, 4096, 128, 128, True, False, True): (3, 32, 3, 8),
        (384, 384, 8192, 128, 128, False, True, True): (2, 64, 3, 8),
        (384, 384, 8192, 128, 128, True, False, True): (2, 64, 2, 4),
        (384, 384, 16384, 128, 128, False, True, True): (1, 128, 2, 8),
        (384, 384, 16384, 128, 128, True, False, True): (3, 128, 2, 4),
        (384, 384, 32768, 128, 128, False, True, True): (2, 256, 3, 8),
        (384, 384, 32768, 128, 128, True, False, True): (1, 256, 2, 4),
        (384, 384, 65536, 128, 128, False, True, True): (7, 512, 1, 4),
        (384, 384, 65536, 128, 128, True, False, True): (3, 512, 2, 4),
        (384, 384, 131072, 128, 128, False, True, True): (5, 1024, 1, 4),
        (384, 384, 131072, 128, 128, True, False, True): (1, 1024, 2, 4),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.float32, 0.5)): {
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5467-5502 / 第 5467-5502 行
````python
        (16, 16, 16, 16, 16, False, False, False): (2, 1, 1, 16),
        (16, 16, 16, 16, 16, False, False, True): (1, 1, 2, 4),
        (16, 16, 16, 16, 16, False, True, False): (1, 1, 2, 16),
        (16, 16, 16, 16, 16, False, True, True): (2, 1, 2, 8),
        (16, 16, 16, 16, 16, True, False, False): (1, 1, 1, 2),
        (16, 16, 16, 16, 16, True, False, True): (2, 1, 1, 4),
        (16, 16, 32, 16, 16, False, False, False): (1, 1, 1, 2),
        (16, 16, 32, 16, 16, False, False, True): (1, 1, 2, 8),
        (16, 16, 32, 16, 16, False, True, False): (1, 2, 1, 4),
        (16, 16, 32, 16, 16, False, True, True): (1, 2, 2, 4),
        (16, 16, 32, 16, 16, True, False, False): (1, 1, 2, 4),
        (16, 16, 32, 16, 16, True, False, True): (1, 2, 2, 4),
        (16, 16, 64, 16, 16, False, False, False): (1, 4, 1, 4),
        (16, 16, 64, 16, 16, False, False, True): (2, 2, 1, 4),
        (16, 16, 64, 16, 16, False, True, False): (1, 4, 1, 4),
        (16, 16, 64, 16, 16, False, True, True): (1, 4, 1, 8),
        (16, 16, 64, 16, 16, True, False, False): (1, 2, 1, 4),
        (16, 16, 64, 16, 16, True, False, True): (1, 4, 2, 8),
        (16, 32, 16, 16, 16, False, False, False): (1, 1, 2, 8),
        (16, 32, 16, 16, 16, False, False, True): (2, 1, 1, 4),
        (16, 32, 16, 16, 16, False, True, False): (1, 1, 1, 4),
        (16, 32, 16, 16, 16, False, True, True): (1, 1, 1, 4),
        (16, 32, 16, 16, 16, True, False, False): (1, 1, 1, 4),
        (16, 32, 16, 16, 16, True, False, True): (1, 1, 2, 8),
        (16, 32, 16, 16, 32, False, False, False): (1, 1, 2, 4),
        (16, 32, 16, 16, 32, False, False, True): (2, 1, 2, 2),
        (16, 32, 16, 16, 32, False, True, False): (1, 1, 1, 8),
        (16, 32, 16, 16, 32, False, True, True): (1, 1, 1, 2),
        (16, 32, 16, 16, 32, True, False, False): (3, 1, 1, 4),
        (16, 32, 16, 16, 32, True, False, True): (1, 1, 1, 4),
        (16, 32, 32, 16, 16, False, False, False): (1, 2, 1, 4),
        (16, 32, 32, 16, 16, False, False, True): (2, 2, 1, 4),
        (16, 32, 32, 16, 16, False, True, False): (1, 2, 1, 2),
        (16, 32, 32, 16, 16, False, True, True): (1, 2, 1, 4),
        (16, 32, 32, 16, 16, True, False, False): (1, 2, 1, 4),
        (16, 32, 32, 16, 16, True, False, True): (1, 2, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5503-5538 / 第 5503-5538 行
````python
        (16, 32, 32, 16, 32, False, False, False): (1, 1, 2, 4),
        (16, 32, 32, 16, 32, False, False, True): (1, 2, 1, 4),
        (16, 32, 32, 16, 32, False, True, False): (1, 2, 2, 8),
        (16, 32, 32, 16, 32, False, True, True): (1, 2, 1, 1),
        (16, 32, 32, 16, 32, True, False, False): (1, 2, 1, 2),
        (16, 32, 32, 16, 32, True, False, True): (1, 2, 1, 4),
        (16, 32, 64, 16, 16, False, False, False): (1, 2, 1, 4),
        (16, 32, 64, 16, 16, False, False, True): (2, 4, 1, 4),
        (16, 32, 64, 16, 16, False, True, False): (1, 4, 2, 4),
        (16, 32, 64, 16, 16, False, True, True): (1, 4, 1, 4),
        (16, 32, 64, 16, 16, True, False, False): (1, 2, 2, 8),
        (16, 32, 64, 16, 16, True, False, True): (1, 4, 1, 2),
        (16, 32, 64, 16, 32, False, False, False): (1, 4, 1, 4),
        (16, 32, 64, 16, 32, False, False, True): (1, 4, 3, 4),
        (16, 32, 64, 16, 32, False, True, False): (1, 2, 1, 4),
        (16, 32, 64, 16, 32, False, True, True): (1, 4, 1, 4),
        (16, 32, 64, 16, 32, True, False, False): (1, 2, 1, 8),
        (16, 32, 64, 16, 32, True, False, True): (1, 2, 1, 4),
        (16, 64, 16, 16, 32, False, False, False): (1, 1, 1, 2),
        (16, 64, 16, 16, 32, False, False, True): (1, 1, 1, 8),
        (16, 64, 16, 16, 32, False, True, False): (1, 1, 1, 8),
        (16, 64, 16, 16, 32, False, True, True): (1, 1, 1, 4),
        (16, 64, 16, 16, 32, True, False, False): (1, 1, 1, 8),
        (16, 64, 16, 16, 32, True, False, True): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, False, False, False): (1, 2, 1, 4),
        (16, 64, 32, 16, 32, False, False, True): (1, 1, 1, 4),
        (16, 64, 32, 16, 32, False, True, False): (1, 2, 1, 1),
        (16, 64, 32, 16, 32, False, True, True): (1, 2, 1, 8),
        (16, 64, 32, 16, 32, True, False, False): (2, 2, 1, 4),
        (16, 64, 32, 16, 32, True, False, True): (2, 2, 1, 4),
        (16, 64, 64, 16, 32, False, False, False): (1, 2, 1, 4),
        (16, 64, 64, 16, 32, False, False, True): (1, 4, 1, 4),
        (16, 64, 64, 16, 32, False, True, False): (1, 4, 1, 4),
        (16, 64, 64, 16, 32, False, True, True): (1, 4, 1, 4),
        (16, 64, 64, 16, 32, True, False, False): (1, 4, 1, 2),
        (16, 64, 64, 16, 32, True, False, True): (3, 4, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5539-5574 / 第 5539-5574 行
````python
        (32, 16, 16, 16, 16, False, False, False): (1, 1, 2, 4),
        (32, 16, 16, 16, 16, False, False, True): (1, 1, 1, 2),
        (32, 16, 16, 16, 16, False, True, False): (1, 1, 2, 4),
        (32, 16, 16, 16, 16, False, True, True): (1, 1, 2, 4),
        (32, 16, 16, 16, 16, True, False, False): (1, 1, 3, 8),
        (32, 16, 16, 16, 16, True, False, True): (1, 1, 2, 4),
        (32, 16, 32, 16, 16, False, False, False): (1, 2, 1, 4),
        (32, 16, 32, 16, 16, False, False, True): (1, 2, 3, 4),
        (32, 16, 32, 16, 16, False, True, False): (1, 1, 1, 8),
        (32, 16, 32, 16, 16, False, True, True): (1, 2, 1, 4),
        (32, 16, 32, 16, 16, True, False, False): (1, 1, 1, 2),
        (32, 16, 32, 16, 16, True, False, True): (1, 1, 1, 4),
        (32, 16, 64, 16, 16, False, False, False): (1, 4, 1, 4),
        (32, 16, 64, 16, 16, False, False, True): (3, 4, 1, 4),
        (32, 16, 64, 16, 16, False, True, False): (1, 4, 1, 1),
        (32, 16, 64, 16, 16, False, True, True): (1, 4, 1, 4),
        (32, 16, 64, 16, 16, True, False, False): (1, 4, 1, 4),
        (32, 16, 64, 16, 16, True, False, True): (1, 4, 1, 4),
        (32, 32, 16, 16, 16, False, False, False): (1, 1, 1, 2),
        (32, 32, 16, 16, 16, False, False, True): (2, 1, 1, 4),
        (32, 32, 16, 16, 16, False, True, False): (1, 1, 1, 2),
        (32, 32, 16, 16, 16, False, True, True): (2, 1, 1, 4),
        (32, 32, 16, 16, 16, True, False, False): (3, 1, 2, 4),
        (32, 32, 16, 16, 16, True, False, True): (1, 1, 2, 4),
        (32, 32, 16, 16, 32, False, False, False): (2, 1, 1, 2),
        (32, 32, 16, 16, 32, False, False, True): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, False, True, False): (1, 1, 1, 4),
        (32, 32, 16, 16, 32, False, True, True): (1, 1, 1, 8),
        (32, 32, 16, 16, 32, True, False, False): (1, 1, 1, 8),
        (32, 32, 16, 16, 32, True, False, True): (1, 1, 1, 4),
        (32, 32, 16, 32, 32, False, False, False): (2, 1, 1, 4),
        (32, 32, 16, 32, 32, False, False, True): (1, 1, 2, 4),
        (32, 32, 16, 32, 32, False, True, False): (2, 1, 1, 1),
        (32, 32, 16, 32, 32, False, True, True): (2, 1, 2, 4),
        (32, 32, 16, 32, 32, True, False, False): (1, 1, 1, 8),
        (32, 32, 16, 32, 32, True, False, True): (1, 1, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5575-5610 / 第 5575-5610 行
````python
        (32, 32, 32, 16, 16, False, False, False): (1, 1, 1, 4),
        (32, 32, 32, 16, 16, False, False, True): (1, 2, 1, 2),
        (32, 32, 32, 16, 16, False, True, False): (2, 2, 1, 4),
        (32, 32, 32, 16, 16, False, True, True): (1, 2, 2, 4),
        (32, 32, 32, 16, 16, True, False, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 16, True, False, True): (2, 2, 1, 4),
        (32, 32, 32, 16, 32, False, False, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, False, True): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, True, False): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, False, True, True): (1, 2, 1, 4),
        (32, 32, 32, 16, 32, True, False, False): (2, 1, 1, 2),
        (32, 32, 32, 16, 32, True, False, True): (2, 2, 2, 4),
        (32, 32, 32, 32, 32, False, False, False): (1, 1, 1, 4),
        (32, 32, 32, 32, 32, False, False, True): (1, 1, 1, 2),
        (32, 32, 32, 32, 32, False, True, False): (1, 1, 1, 4),
        (32, 32, 32, 32, 32, False, True, True): (1, 1, 2, 2),
        (32, 32, 32, 32, 32, True, False, False): (1, 1, 1, 2),
        (32, 32, 32, 32, 32, True, False, True): (1, 1, 2, 1),
        (32, 32, 64, 16, 16, False, False, False): (2, 4, 1, 4),
        (32, 32, 64, 16, 16, False, False, True): (1, 4, 2, 4),
        (32, 32, 64, 16, 16, False, True, False): (1, 4, 1, 4),
        (32, 32, 64, 16, 16, False, True, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 16, True, False, False): (1, 2, 1, 4),
        (32, 32, 64, 16, 16, True, False, True): (2, 4, 1, 4),
        (32, 32, 64, 16, 32, False, False, False): (1, 4, 1, 8),
        (32, 32, 64, 16, 32, False, False, True): (1, 4, 1, 4),
        (32, 32, 64, 16, 32, False, True, False): (1, 4, 1, 4),
        (32, 32, 64, 16, 32, False, True, True): (2, 4, 1, 4),
        (32, 32, 64, 16, 32, True, False, False): (1, 2, 2, 4),
        (32, 32, 64, 16, 32, True, False, True): (2, 4, 1, 4),
        (32, 32, 64, 32, 32, False, False, False): (2, 2, 1, 4),
        (32, 32, 64, 32, 32, False, False, True): (1, 1, 1, 4),
        (32, 32, 64, 32, 32, False, True, False): (1, 1, 1, 8),
        (32, 32, 64, 32, 32, False, True, True): (2, 1, 1, 4),
        (32, 32, 64, 32, 32, True, False, False): (1, 1, 1, 4),
        (32, 32, 64, 32, 32, True, False, True): (1, 2, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5611-5646 / 第 5611-5646 行
````python
        (32, 64, 16, 16, 32, False, False, False): (1, 1, 2, 2),
        (32, 64, 16, 16, 32, False, False, True): (2, 1, 1, 4),
        (32, 64, 16, 16, 32, False, True, False): (1, 1, 1, 8),
        (32, 64, 16, 16, 32, False, True, True): (1, 1, 3, 4),
        (32, 64, 16, 16, 32, True, False, False): (1, 1, 1, 2),
        (32, 64, 16, 16, 32, True, False, True): (1, 1, 2, 4),
        (32, 64, 16, 32, 32, False, False, False): (1, 1, 1, 2),
        (32, 64, 16, 32, 32, False, False, True): (1, 1, 3, 4),
        (32, 64, 16, 32, 32, False, True, False): (1, 1, 2, 4),
        (32, 64, 16, 32, 32, False, True, True): (1, 1, 1, 8),
        (32, 64, 16, 32, 32, True, False, False): (1, 1, 2, 4),
        (32, 64, 16, 32, 32, True, False, True): (1, 1, 1, 8),
        (32, 64, 32, 16, 32, False, False, False): (1, 2, 1, 4),
        (32, 64, 32, 16, 32, False, False, True): (1, 2, 3, 4),
        (32, 64, 32, 16, 32, False, True, False): (1, 2, 1, 8),
        (32, 64, 32, 16, 32, False, True, True): (3, 2, 1, 4),
        (32, 64, 32, 16, 32, True, False, False): (1, 1, 1, 8),
        (32, 64, 32, 16, 32, True, False, True): (1, 2, 1, 4),
        (32, 64, 32, 32, 32, False, False, False): (1, 1, 1, 1),
        (32, 64, 32, 32, 32, False, False, True): (1, 1, 1, 4),
        (32, 64, 32, 32, 32, False, True, False): (1, 1, 1, 4),
        (32, 64, 32, 32, 32, False, True, True): (1, 1, 1, 4),
        (32, 64, 32, 32, 32, True, False, False): (1, 1, 1, 4),
        (32, 64, 32, 32, 32, True, False, True): (1, 1, 2, 8),
        (32, 64, 64, 16, 32, False, False, False): (2, 4, 1, 4),
        (32, 64, 64, 16, 32, False, False, True): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, False, True, False): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, False, True, True): (2, 4, 1, 4),
        (32, 64, 64, 16, 32, True, False, False): (1, 4, 1, 4),
        (32, 64, 64, 16, 32, True, False, True): (1, 4, 1, 4),
        (32, 64, 64, 32, 32, False, False, False): (2, 2, 1, 4),
        (32, 64, 64, 32, 32, False, False, True): (1, 2, 1, 8),
        (32, 64, 64, 32, 32, False, True, False): (1, 2, 1, 4),
        (32, 64, 64, 32, 32, False, True, True): (1, 2, 1, 4),
        (32, 64, 64, 32, 32, True, False, False): (2, 2, 1, 4),
        (32, 64, 64, 32, 32, True, False, True): (1, 2, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5647-5682 / 第 5647-5682 行
````python
        (64, 32, 16, 32, 32, False, False, False): (1, 1, 1, 4),
        (64, 32, 16, 32, 32, False, False, True): (3, 1, 2, 4),
        (64, 32, 16, 32, 32, False, True, False): (2, 1, 1, 2),
        (64, 32, 16, 32, 32, False, True, True): (1, 1, 1, 8),
        (64, 32, 16, 32, 32, True, False, False): (1, 1, 1, 2),
        (64, 32, 16, 32, 32, True, False, True): (1, 1, 1, 4),
        (64, 32, 32, 32, 32, False, False, False): (1, 1, 1, 4),
        (64, 32, 32, 32, 32, False, False, True): (1, 1, 2, 8),
        (64, 32, 32, 32, 32, False, True, False): (1, 1, 1, 8),
        (64, 32, 32, 32, 32, False, True, True): (1, 1, 1, 4),
        (64, 32, 32, 32, 32, True, False, False): (1, 1, 2, 4),
        (64, 32, 32, 32, 32, True, False, True): (1, 1, 3, 8),
        (64, 32, 64, 32, 32, False, False, False): (1, 2, 1, 4),
        (64, 32, 64, 32, 32, False, False, True): (2, 2, 1, 4),
        (64, 32, 64, 32, 32, False, True, False): (1, 1, 1, 4),
        (64, 32, 64, 32, 32, False, True, True): (1, 2, 1, 8),
        (64, 32, 64, 32, 32, True, False, False): (2, 2, 1, 4),
        (64, 32, 64, 32, 32, True, False, True): (1, 2, 1, 8),
        (64, 64, 16, 32, 32, False, False, False): (1, 1, 2, 8),
        (64, 64, 16, 32, 32, False, False, True): (2, 1, 2, 4),
        (64, 64, 16, 32, 32, False, True, False): (1, 1, 1, 2),
        (64, 64, 16, 32, 32, False, True, True): (1, 1, 2, 4),
        (64, 64, 16, 32, 32, True, False, False): (1, 1, 1, 2),
        (64, 64, 16, 32, 32, True, False, True): (1, 1, 2, 4),
        (64, 64, 32, 32, 32, False, False, False): (1, 1, 1, 4),
        (64, 64, 32, 32, 32, False, False, True): (2, 1, 1, 4),
        (64, 64, 32, 32, 32, False, True, False): (1, 1, 1, 8),
        (64, 64, 32, 32, 32, False, True, True): (2, 1, 1, 4),
        (64, 64, 32, 32, 32, True, False, False): (1, 1, 1, 4),
        (64, 64, 32, 32, 32, True, False, True): (1, 1, 1, 8),
        (64, 64, 64, 32, 32, False, False, False): (2, 2, 1, 4),
        (64, 64, 64, 32, 32, False, False, True): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, False, True, False): (1, 2, 1, 4),
        (64, 64, 64, 32, 32, False, True, True): (2, 2, 1, 4),
        (64, 64, 64, 32, 32, True, False, False): (1, 1, 1, 8),
        (64, 64, 64, 32, 32, True, False, True): (1, 2, 2, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5683-5718 / 第 5683-5718 行
````python
        (192, 192, 256, 16, 16, False, True, True): (1, 16, 3, 2),
        (192, 192, 256, 16, 16, True, False, True): (1, 8, 5, 4),
        (192, 192, 256, 32, 32, False, True, True): (2, 8, 4, 4),
        (192, 192, 256, 32, 32, True, False, True): (1, 8, 5, 4),
        (192, 192, 512, 16, 16, False, True, True): (2, 16, 3, 4),
        (192, 192, 512, 16, 16, True, False, True): (1, 16, 5, 4),
        (192, 192, 512, 32, 32, False, True, True): (1, 16, 3, 4),
        (192, 192, 512, 32, 32, True, False, True): (2, 16, 3, 4),
        (192, 192, 1024, 16, 16, False, True, True): (3, 16, 3, 4),
        (192, 192, 1024, 16, 16, True, False, True): (2, 8, 3, 4),
        (192, 192, 1024, 32, 32, False, True, True): (3, 32, 1, 4),
        (192, 192, 1024, 32, 32, True, False, True): (3, 16, 3, 4),
        (192, 192, 2048, 16, 16, False, True, True): (1, 32, 3, 4),
        (192, 192, 2048, 16, 16, True, False, True): (2, 16, 3, 4),
        (192, 192, 2048, 32, 32, False, True, True): (1, 64, 1, 4),
        (192, 192, 2048, 32, 32, True, False, True): (1, 64, 2, 4),
        (192, 192, 4096, 16, 16, False, True, True): (1, 64, 2, 4),
        (192, 192, 4096, 16, 16, True, False, True): (1, 32, 3, 4),
        (192, 192, 4096, 32, 32, False, True, True): (3, 128, 2, 4),
        (192, 192, 4096, 32, 32, True, False, True): (1, 128, 2, 4),
        (192, 192, 8192, 16, 16, False, True, True): (2, 64, 3, 4),
        (192, 192, 8192, 16, 16, True, False, True): (1, 64, 3, 4),
        (192, 192, 8192, 32, 32, False, True, True): (3, 128, 3, 4),
        (192, 192, 8192, 32, 32, True, False, True): (1, 128, 2, 4),
        (192, 192, 16384, 16, 16, False, True, True): (1, 256, 3, 2),
        (192, 192, 16384, 16, 16, True, False, True): (1, 256, 3, 2),
        (192, 192, 16384, 32, 32, False, True, True): (2, 256, 3, 4),
        (192, 192, 16384, 32, 32, True, False, True): (2, 256, 3, 4),
        (192, 192, 32768, 16, 16, False, True, True): (2, 512, 3, 2),
        (192, 192, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (192, 192, 32768, 32, 32, False, True, True): (2, 512, 3, 4),
        (192, 192, 32768, 32, 32, True, False, True): (2, 512, 3, 4),
        (192, 192, 65536, 16, 16, False, True, True): (2, 1024, 3, 2),
        (192, 192, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (192, 192, 65536, 32, 32, False, True, True): (2, 1024, 3, 4),
        (192, 192, 65536, 32, 32, True, False, True): (2, 1024, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5719-5754 / 第 5719-5754 行
````python
        (192, 192, 131072, 16, 16, False, True, True): (2, 512, 3, 4),
        (192, 192, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (192, 192, 131072, 32, 32, False, True, True): (2, 1024, 3, 4),
        (192, 192, 131072, 32, 32, True, False, True): (2, 1024, 3, 4),
        (256, 256, 256, 16, 16, False, True, True): (1, 16, 3, 4),
        (256, 256, 256, 16, 16, True, False, True): (2, 16, 1, 4),
        (256, 256, 256, 32, 32, False, True, True): (1, 8, 4, 8),
        (256, 256, 256, 32, 32, True, False, True): (4, 8, 4, 4),
        (256, 256, 256, 64, 64, False, True, True): (1, 4, 4, 8),
        (256, 256, 256, 64, 64, True, False, True): (1, 4, 3, 8),
        (256, 256, 256, 128, 128, False, True, True): (7, 2, 1, 32),
        (256, 256, 256, 128, 128, True, False, True): (3, 2, 1, 32),
        (256, 256, 512, 16, 16, False, True, True): (1, 16, 5, 4),
        (256, 256, 512, 16, 16, True, False, True): (1, 16, 3, 2),
        (256, 256, 512, 32, 32, False, True, True): (4, 16, 4, 4),
        (256, 256, 512, 32, 32, True, False, True): (4, 16, 3, 4),
        (256, 256, 512, 64, 64, False, True, True): (1, 8, 3, 8),
        (256, 256, 512, 64, 64, True, False, True): (1, 8, 3, 8),
        (256, 256, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (256, 256, 512, 128, 128, True, False, True): (3, 4, 1, 32),
        (256, 256, 1024, 16, 16, False, True, True): (3, 32, 5, 2),
        (256, 256, 1024, 16, 16, True, False, True): (2, 32, 5, 2),
        (256, 256, 1024, 32, 32, False, True, True): (1, 32, 4, 4),
        (256, 256, 1024, 32, 32, True, False, True): (1, 32, 5, 4),
        (256, 256, 1024, 64, 64, False, True, True): (4, 16, 3, 8),
        (256, 256, 1024, 64, 64, True, False, True): (1, 16, 3, 8),
        (256, 256, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (256, 256, 1024, 128, 128, True, False, True): (3, 8, 1, 32),
        (256, 256, 2048, 16, 16, False, True, True): (3, 32, 3, 4),
        (256, 256, 2048, 16, 16, True, False, True): (1, 64, 3, 2),
        (256, 256, 2048, 32, 32, False, True, True): (1, 64, 3, 4),
        (256, 256, 2048, 32, 32, True, False, True): (1, 64, 3, 4),
        (256, 256, 2048, 64, 64, False, True, True): (2, 32, 1, 8),
        (256, 256, 2048, 64, 64, True, False, True): (2, 32, 1, 8),
        (256, 256, 2048, 128, 128, False, True, True): (4, 16, 1, 32),
        (256, 256, 2048, 128, 128, True, False, True): (4, 16, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5755-5790 / 第 5755-5790 行
````python
        (256, 256, 4096, 16, 16, False, True, True): (1, 32, 2, 4),
        (256, 256, 4096, 16, 16, True, False, True): (1, 32, 3, 4),
        (256, 256, 4096, 32, 32, False, True, True): (1, 128, 2, 4),
        (256, 256, 4096, 32, 32, True, False, True): (1, 128, 2, 4),
        (256, 256, 4096, 64, 64, False, True, True): (2, 64, 4, 8),
        (256, 256, 4096, 64, 64, True, False, True): (3, 64, 2, 8),
        (256, 256, 4096, 128, 128, False, True, True): (3, 32, 1, 32),
        (256, 256, 4096, 128, 128, True, False, True): (2, 32, 1, 32),
        (256, 256, 8192, 16, 16, False, True, True): (1, 64, 3, 4),
        (256, 256, 8192, 16, 16, True, False, True): (2, 128, 3, 2),
        (256, 256, 8192, 32, 32, False, True, True): (3, 128, 3, 4),
        (256, 256, 8192, 32, 32, True, False, True): (1, 128, 3, 4),
        (256, 256, 8192, 64, 64, False, True, True): (3, 128, 1, 4),
        (256, 256, 8192, 64, 64, True, False, True): (4, 128, 2, 8),
        (256, 256, 8192, 128, 128, False, True, True): (6, 64, 1, 32),
        (256, 256, 8192, 128, 128, True, False, True): (2, 64, 1, 32),
        (256, 256, 16384, 16, 16, False, True, True): (4, 128, 3, 4),
        (256, 256, 16384, 16, 16, True, False, True): (3, 128, 3, 4),
        (256, 256, 16384, 32, 32, False, True, True): (4, 256, 3, 4),
        (256, 256, 16384, 32, 32, True, False, True): (2, 256, 3, 4),
        (256, 256, 16384, 64, 64, False, True, True): (3, 256, 1, 4),
        (256, 256, 16384, 64, 64, True, False, True): (2, 256, 2, 4),
        (256, 256, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (256, 256, 16384, 128, 128, True, False, True): (3, 128, 1, 32),
        (256, 256, 32768, 16, 16, False, True, True): (1, 256, 3, 4),
        (256, 256, 32768, 16, 16, True, False, True): (2, 128, 3, 4),
        (256, 256, 32768, 32, 32, False, True, True): (2, 512, 3, 4),
        (256, 256, 32768, 32, 32, True, False, True): (4, 512, 3, 4),
        (256, 256, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (256, 256, 32768, 64, 64, True, False, True): (1, 512, 2, 4),
        (256, 256, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (256, 256, 32768, 128, 128, True, False, True): (1, 256, 1, 32),
        (256, 256, 65536, 16, 16, False, True, True): (2, 512, 3, 4),
        (256, 256, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (256, 256, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (256, 256, 65536, 32, 32, True, False, True): (2, 1024, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5791-5826 / 第 5791-5826 行
````python
        (256, 256, 65536, 64, 64, False, True, True): (1, 1024, 2, 4),
        (256, 256, 65536, 64, 64, True, False, True): (1, 1024, 2, 4),
        (256, 256, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (256, 256, 65536, 128, 128, True, False, True): (2, 512, 1, 32),
        (256, 256, 131072, 16, 16, False, True, True): (1, 1024, 3, 4),
        (256, 256, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (256, 256, 131072, 32, 32, False, True, True): (1, 2048, 3, 4),
        (256, 256, 131072, 32, 32, True, False, True): (1, 2048, 3, 4),
        (256, 256, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (256, 256, 131072, 64, 64, True, False, True): (1, 2048, 2, 4),
        (256, 256, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (256, 256, 131072, 128, 128, True, False, True): (4, 1024, 1, 32),
        (384, 384, 256, 16, 16, False, True, True): (1, 8, 3, 4),
        (384, 384, 256, 16, 16, True, False, True): (1, 8, 3, 4),
        (384, 384, 256, 32, 32, False, True, True): (2, 8, 3, 8),
        (384, 384, 256, 32, 32, True, False, True): (1, 8, 3, 4),
        (384, 384, 256, 64, 64, False, True, True): (1, 4, 4, 8),
        (384, 384, 256, 64, 64, True, False, True): (2, 4, 3, 8),
        (384, 384, 512, 16, 16, False, True, True): (3, 16, 3, 2),
        (384, 384, 512, 16, 16, True, False, True): (3, 16, 3, 2),
        (384, 384, 512, 32, 32, False, True, True): (2, 8, 3, 4),
        (384, 384, 512, 32, 32, True, False, True): (1, 8, 3, 4),
        (384, 384, 512, 64, 64, False, True, True): (2, 8, 3, 8),
        (384, 384, 512, 64, 64, True, False, True): (2, 8, 4, 8),
        (384, 384, 1024, 16, 16, False, True, True): (3, 16, 3, 2),
        (384, 384, 1024, 16, 16, True, False, True): (4, 32, 3, 2),
        (384, 384, 1024, 32, 32, False, True, True): (1, 32, 3, 4),
        (384, 384, 1024, 32, 32, True, False, True): (2, 16, 3, 4),
        (384, 384, 1024, 64, 64, False, True, True): (2, 16, 3, 8),
        (384, 384, 1024, 64, 64, True, False, True): (4, 16, 4, 8),
        (384, 384, 2048, 16, 16, False, True, True): (3, 16, 3, 4),
        (384, 384, 2048, 16, 16, True, False, True): (1, 32, 3, 4),
        (384, 384, 2048, 32, 32, False, True, True): (3, 64, 2, 4),
        (384, 384, 2048, 32, 32, True, False, True): (1, 64, 3, 4),
        (384, 384, 2048, 64, 64, False, True, True): (4, 32, 4, 8),
        (384, 384, 2048, 64, 64, True, False, True): (5, 32, 4, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5827-5862 / 第 5827-5862 行
````python
        (384, 384, 4096, 16, 16, False, True, True): (1, 32, 3, 4),
        (384, 384, 4096, 16, 16, True, False, True): (3, 32, 3, 4),
        (384, 384, 4096, 32, 32, False, True, True): (2, 64, 3, 4),
        (384, 384, 4096, 32, 32, True, False, True): (2, 64, 3, 4),
        (384, 384, 4096, 64, 64, False, True, True): (2, 64, 3, 8),
        (384, 384, 4096, 64, 64, True, False, True): (2, 64, 3, 8),
        (384, 384, 8192, 16, 16, False, True, True): (1, 128, 3, 2),
        (384, 384, 8192, 16, 16, True, False, True): (1, 128, 3, 2),
        (384, 384, 8192, 32, 32, False, True, True): (1, 128, 3, 4),
        (384, 384, 8192, 32, 32, True, False, True): (1, 128, 3, 4),
        (384, 384, 8192, 64, 64, False, True, True): (3, 128, 3, 4),
        (384, 384, 8192, 64, 64, True, False, True): (2, 128, 3, 4),
        (384, 384, 16384, 16, 16, False, True, True): (1, 256, 3, 2),
        (384, 384, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (384, 384, 16384, 32, 32, False, True, True): (2, 256, 3, 4),
        (384, 384, 16384, 32, 32, True, False, True): (4, 256, 3, 4),
        (384, 384, 16384, 64, 64, False, True, True): (2, 256, 3, 4),
        (384, 384, 16384, 64, 64, True, False, True): (1, 256, 3, 4),
        (384, 384, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (384, 384, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (384, 384, 32768, 32, 32, False, True, True): (1, 512, 3, 4),
        (384, 384, 32768, 32, 32, True, False, True): (1, 512, 2, 4),
        (384, 384, 32768, 64, 64, False, True, True): (1, 512, 3, 4),
        (384, 384, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (384, 384, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (384, 384, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (384, 384, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (384, 384, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (384, 384, 65536, 64, 64, False, True, True): (1, 1024, 3, 4),
        (384, 384, 65536, 64, 64, True, False, True): (1, 1024, 3, 4),
        (384, 384, 131072, 16, 16, False, True, True): (1, 512, 3, 4),
        (384, 384, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (384, 384, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (384, 384, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (384, 384, 131072, 64, 64, False, True, True): (1, 2048, 3, 4),
        (384, 384, 131072, 64, 64, True, False, True): (1, 2048, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5863-5898 / 第 5863-5898 行
````python
        (512, 512, 256, 16, 16, False, True, True): (1, 8, 4, 4),
        (512, 512, 256, 16, 16, True, False, True): (1, 8, 3, 2),
        (512, 512, 256, 32, 32, False, True, True): (4, 8, 3, 4),
        (512, 512, 256, 32, 32, True, False, True): (4, 8, 3, 4),
        (512, 512, 256, 64, 64, False, True, True): (3, 4, 3, 8),
        (512, 512, 256, 64, 64, True, False, True): (5, 4, 3, 8),
        (512, 512, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (512, 512, 256, 128, 128, True, False, True): (3, 2, 1, 32),
        (512, 512, 512, 16, 16, False, True, True): (2, 16, 3, 2),
        (512, 512, 512, 16, 16, True, False, True): (1, 8, 4, 4),
        (512, 512, 512, 32, 32, False, True, True): (3, 16, 3, 4),
        (512, 512, 512, 32, 32, True, False, True): (5, 16, 2, 4),
        (512, 512, 512, 64, 64, False, True, True): (1, 8, 3, 8),
        (512, 512, 512, 64, 64, True, False, True): (3, 8, 3, 8),
        (512, 512, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (512, 512, 512, 128, 128, True, False, True): (3, 4, 1, 16),
        (512, 512, 1024, 16, 16, False, True, True): (1, 16, 3, 4),
        (512, 512, 1024, 16, 16, True, False, True): (3, 16, 3, 4),
        (512, 512, 1024, 32, 32, False, True, True): (3, 32, 3, 4),
        (512, 512, 1024, 32, 32, True, False, True): (3, 32, 2, 4),
        (512, 512, 1024, 64, 64, False, True, True): (1, 16, 3, 8),
        (512, 512, 1024, 64, 64, True, False, True): (4, 16, 3, 8),
        (512, 512, 1024, 128, 128, False, True, True): (4, 8, 1, 32),
        (512, 512, 1024, 128, 128, True, False, True): (4, 8, 1, 32),
        (512, 512, 2048, 16, 16, False, True, True): (5, 16, 3, 4),
        (512, 512, 2048, 16, 16, True, False, True): (5, 16, 3, 4),
        (512, 512, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (512, 512, 2048, 32, 32, True, False, True): (1, 32, 4, 4),
        (512, 512, 2048, 64, 64, False, True, True): (4, 32, 3, 8),
        (512, 512, 2048, 64, 64, True, False, True): (4, 32, 3, 8),
        (512, 512, 2048, 128, 128, False, True, True): (3, 16, 1, 32),
        (512, 512, 2048, 128, 128, True, False, True): (3, 16, 1, 32),
        (512, 512, 4096, 16, 16, False, True, True): (4, 32, 3, 4),
        (512, 512, 4096, 16, 16, True, False, True): (4, 64, 3, 2),
        (512, 512, 4096, 32, 32, False, True, True): (3, 64, 3, 4),
        (512, 512, 4096, 32, 32, True, False, True): (3, 64, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5899-5934 / 第 5899-5934 行
````python
        (512, 512, 4096, 64, 64, False, True, True): (4, 64, 2, 4),
        (512, 512, 4096, 64, 64, True, False, True): (1, 64, 2, 4),
        (512, 512, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (512, 512, 4096, 128, 128, True, False, True): (1, 32, 1, 32),
        (512, 512, 8192, 16, 16, False, True, True): (1, 64, 3, 4),
        (512, 512, 8192, 16, 16, True, False, True): (4, 64, 3, 4),
        (512, 512, 8192, 32, 32, False, True, True): (2, 128, 3, 4),
        (512, 512, 8192, 32, 32, True, False, True): (3, 128, 3, 4),
        (512, 512, 8192, 64, 64, False, True, True): (1, 128, 2, 4),
        (512, 512, 8192, 64, 64, True, False, True): (1, 128, 2, 4),
        (512, 512, 8192, 128, 128, False, True, True): (6, 64, 1, 32),
        (512, 512, 8192, 128, 128, True, False, True): (4, 64, 1, 32),
        (512, 512, 16384, 16, 16, False, True, True): (1, 128, 3, 4),
        (512, 512, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (512, 512, 16384, 32, 32, False, True, True): (1, 256, 3, 4),
        (512, 512, 16384, 32, 32, True, False, True): (4, 256, 3, 4),
        (512, 512, 16384, 64, 64, False, True, True): (1, 256, 2, 4),
        (512, 512, 16384, 64, 64, True, False, True): (1, 256, 2, 4),
        (512, 512, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (512, 512, 16384, 128, 128, True, False, True): (2, 128, 1, 32),
        (512, 512, 32768, 16, 16, False, True, True): (1, 256, 3, 4),
        (512, 512, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (512, 512, 32768, 32, 32, False, True, True): (1, 512, 3, 4),
        (512, 512, 32768, 32, 32, True, False, True): (1, 512, 3, 4),
        (512, 512, 32768, 64, 64, False, True, True): (1, 512, 2, 4),
        (512, 512, 32768, 64, 64, True, False, True): (2, 512, 2, 4),
        (512, 512, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (512, 512, 32768, 128, 128, True, False, True): (2, 256, 1, 32),
        (512, 512, 65536, 16, 16, False, True, True): (1, 512, 3, 4),
        (512, 512, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (512, 512, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (512, 512, 65536, 32, 32, True, False, True): (1, 1024, 3, 4),
        (512, 512, 65536, 64, 64, False, True, True): (1, 1024, 2, 4),
        (512, 512, 65536, 64, 64, True, False, True): (1, 1024, 2, 4),
        (512, 512, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (512, 512, 65536, 128, 128, True, False, True): (4, 512, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5935-5970 / 第 5935-5970 行
````python
        (512, 512, 131072, 16, 16, False, True, True): (1, 512, 3, 4),
        (512, 512, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (512, 512, 131072, 32, 32, False, True, True): (1, 2048, 3, 4),
        (512, 512, 131072, 32, 32, True, False, True): (1, 2048, 3, 4),
        (512, 512, 131072, 64, 64, False, True, True): (1, 2048, 2, 4),
        (512, 512, 131072, 64, 64, True, False, True): (1, 2048, 2, 4),
        (512, 512, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (512, 512, 131072, 128, 128, True, False, True): (2, 1024, 1, 32),
        (768, 768, 256, 16, 16, False, True, True): (1, 4, 5, 4),
        (768, 768, 256, 16, 16, True, False, True): (3, 8, 3, 2),
        (768, 768, 256, 32, 32, False, True, True): (2, 4, 3, 4),
        (768, 768, 256, 32, 32, True, False, True): (3, 8, 4, 4),
        (768, 768, 256, 64, 64, False, True, True): (1, 4, 4, 8),
        (768, 768, 256, 64, 64, True, False, True): (3, 4, 3, 8),
        (768, 768, 256, 128, 128, False, True, True): (3, 2, 1, 32),
        (768, 768, 256, 128, 128, True, False, True): (2, 2, 2, 32),
        (768, 768, 512, 16, 16, False, True, True): (2, 4, 5, 4),
        (768, 768, 512, 16, 16, True, False, True): (2, 4, 4, 4),
        (768, 768, 512, 32, 32, False, True, True): (1, 8, 3, 4),
        (768, 768, 512, 32, 32, True, False, True): (3, 8, 4, 4),
        (768, 768, 512, 64, 64, False, True, True): (2, 8, 3, 8),
        (768, 768, 512, 64, 64, True, False, True): (5, 8, 3, 8),
        (768, 768, 512, 128, 128, False, True, True): (2, 4, 1, 32),
        (768, 768, 512, 128, 128, True, False, True): (2, 4, 2, 32),
        (768, 768, 1024, 16, 16, False, True, True): (2, 16, 4, 2),
        (768, 768, 1024, 16, 16, True, False, True): (4, 32, 3, 1),
        (768, 768, 1024, 32, 32, False, True, True): (1, 32, 2, 4),
        (768, 768, 1024, 32, 32, True, False, True): (1, 16, 5, 4),
        (768, 768, 1024, 64, 64, False, True, True): (2, 16, 3, 8),
        (768, 768, 1024, 64, 64, True, False, True): (2, 16, 3, 8),
        (768, 768, 1024, 128, 128, False, True, True): (1, 8, 2, 32),
        (768, 768, 1024, 128, 128, True, False, True): (1, 8, 1, 32),
        (768, 768, 2048, 16, 16, False, True, True): (1, 16, 3, 4),
        (768, 768, 2048, 16, 16, True, False, True): (1, 16, 3, 4),
        (768, 768, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (768, 768, 2048, 32, 32, True, False, True): (5, 32, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 5971-6006 / 第 5971-6006 行
````python
        (768, 768, 2048, 64, 64, False, True, True): (1, 32, 3, 8),
        (768, 768, 2048, 64, 64, True, False, True): (1, 32, 3, 4),
        (768, 768, 2048, 128, 128, False, True, True): (3, 16, 1, 32),
        (768, 768, 2048, 128, 128, True, False, True): (4, 16, 1, 32),
        (768, 768, 4096, 16, 16, False, True, True): (1, 64, 3, 2),
        (768, 768, 4096, 16, 16, True, False, True): (3, 64, 3, 2),
        (768, 768, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (768, 768, 4096, 32, 32, True, False, True): (1, 64, 3, 4),
        (768, 768, 4096, 64, 64, False, True, True): (4, 64, 3, 4),
        (768, 768, 4096, 64, 64, True, False, True): (4, 64, 3, 4),
        (768, 768, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (768, 768, 4096, 128, 128, True, False, True): (1, 32, 2, 32),
        (768, 768, 8192, 16, 16, False, True, True): (1, 128, 3, 2),
        (768, 768, 8192, 16, 16, True, False, True): (2, 32, 3, 4),
        (768, 768, 8192, 32, 32, False, True, True): (2, 128, 3, 4),
        (768, 768, 8192, 32, 32, True, False, True): (1, 128, 2, 4),
        (768, 768, 8192, 64, 64, False, True, True): (1, 128, 3, 4),
        (768, 768, 8192, 64, 64, True, False, True): (2, 128, 3, 4),
        (768, 768, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (768, 768, 8192, 128, 128, True, False, True): (2, 64, 1, 32),
        (768, 768, 16384, 16, 16, False, True, True): (3, 64, 3, 4),
        (768, 768, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (768, 768, 16384, 32, 32, False, True, True): (2, 256, 3, 4),
        (768, 768, 16384, 32, 32, True, False, True): (4, 256, 2, 4),
        (768, 768, 16384, 64, 64, False, True, True): (1, 256, 3, 4),
        (768, 768, 16384, 64, 64, True, False, True): (1, 256, 3, 4),
        (768, 768, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (768, 768, 16384, 128, 128, True, False, True): (2, 128, 1, 32),
        (768, 768, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (768, 768, 32768, 16, 16, True, False, True): (2, 128, 3, 4),
        (768, 768, 32768, 32, 32, False, True, True): (2, 256, 3, 4),
        (768, 768, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (768, 768, 32768, 64, 64, False, True, True): (1, 512, 3, 4),
        (768, 768, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (768, 768, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (768, 768, 32768, 128, 128, True, False, True): (1, 256, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6007-6042 / 第 6007-6042 行
````python
        (768, 768, 50432, 16, 16, False, True, True): (1, 197, 3, 4),
        (768, 768, 50432, 32, 32, False, True, True): (1, 394, 3, 4),
        (768, 768, 50432, 64, 64, False, True, True): (1, 788, 3, 4),
        (768, 768, 50432, 128, 128, False, True, True): (3, 394, 1, 32),
        (768, 768, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (768, 768, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (768, 768, 65536, 32, 32, False, True, True): (1, 512, 3, 4),
        (768, 768, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (768, 768, 65536, 64, 64, False, True, True): (1, 1024, 3, 4),
        (768, 768, 65536, 64, 64, True, False, True): (1, 1024, 3, 4),
        (768, 768, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (768, 768, 65536, 128, 128, True, False, True): (1, 512, 1, 32),
        (768, 768, 131072, 16, 16, False, True, True): (1, 512, 3, 4),
        (768, 768, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (768, 768, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (768, 768, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (768, 768, 131072, 64, 64, False, True, True): (1, 2048, 3, 4),
        (768, 768, 131072, 64, 64, True, False, True): (1, 2048, 3, 4),
        (768, 768, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (768, 768, 131072, 128, 128, True, False, True): (1, 1024, 1, 32),
        (768, 3072, 256, 16, 16, False, True, True): (1, 2, 4, 4),
        (768, 3072, 256, 16, 16, True, False, True): (1, 4, 3, 4),
        (768, 3072, 256, 32, 32, False, True, True): (1, 4, 3, 4),
        (768, 3072, 256, 32, 32, True, False, True): (3, 4, 3, 4),
        (768, 3072, 256, 64, 64, False, True, True): (1, 4, 3, 8),
        (768, 3072, 256, 64, 64, True, False, True): (1, 4, 3, 8),
        (768, 3072, 256, 128, 128, False, True, True): (2, 2, 2, 32),
        (768, 3072, 256, 128, 128, True, False, True): (2, 2, 1, 32),
        (768, 3072, 512, 16, 16, False, True, True): (2, 4, 3, 4),
        (768, 3072, 512, 16, 16, True, False, True): (1, 8, 3, 2),
        (768, 3072, 512, 32, 32, False, True, True): (3, 8, 4, 4),
        (768, 3072, 512, 32, 32, True, False, True): (3, 8, 3, 4),
        (768, 3072, 512, 64, 64, False, True, True): (1, 8, 4, 8),
        (768, 3072, 512, 64, 64, True, False, True): (1, 8, 3, 8),
        (768, 3072, 512, 128, 128, False, True, True): (1, 4, 2, 32),
        (768, 3072, 512, 128, 128, True, False, True): (1, 4, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6043-6078 / 第 6043-6078 行
````python
        (768, 3072, 1024, 16, 16, False, True, True): (4, 16, 3, 2),
        (768, 3072, 1024, 16, 16, True, False, True): (4, 16, 3, 2),
        (768, 3072, 1024, 32, 32, False, True, True): (4, 16, 5, 4),
        (768, 3072, 1024, 32, 32, True, False, True): (4, 16, 5, 4),
        (768, 3072, 1024, 64, 64, False, True, True): (2, 16, 3, 8),
        (768, 3072, 1024, 64, 64, True, False, True): (2, 16, 3, 8),
        (768, 3072, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (768, 3072, 1024, 128, 128, True, False, True): (1, 8, 1, 32),
        (768, 3072, 2048, 16, 16, False, True, True): (2, 16, 3, 4),
        (768, 3072, 2048, 16, 16, True, False, True): (2, 16, 3, 4),
        (768, 3072, 2048, 32, 32, False, True, True): (4, 32, 5, 4),
        (768, 3072, 2048, 32, 32, True, False, True): (2, 32, 3, 4),
        (768, 3072, 2048, 64, 64, False, True, True): (2, 32, 3, 8),
        (768, 3072, 2048, 64, 64, True, False, True): (2, 32, 3, 8),
        (768, 3072, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (768, 3072, 2048, 128, 128, True, False, True): (2, 16, 1, 32),
        (768, 3072, 4096, 16, 16, False, True, True): (1, 32, 5, 4),
        (768, 3072, 4096, 16, 16, True, False, True): (3, 64, 3, 2),
        (768, 3072, 4096, 32, 32, False, True, True): (5, 64, 3, 4),
        (768, 3072, 4096, 32, 32, True, False, True): (5, 64, 3, 4),
        (768, 3072, 4096, 64, 64, False, True, True): (1, 64, 3, 8),
        (768, 3072, 4096, 64, 64, True, False, True): (5, 64, 3, 4),
        (768, 3072, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (768, 3072, 4096, 128, 128, True, False, True): (1, 32, 1, 32),
        (768, 3072, 8192, 16, 16, False, True, True): (1, 128, 3, 2),
        (768, 3072, 8192, 16, 16, True, False, True): (1, 128, 3, 2),
        (768, 3072, 8192, 32, 32, False, True, True): (1, 128, 3, 4),
        (768, 3072, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (768, 3072, 8192, 64, 64, False, True, True): (3, 128, 3, 4),
        (768, 3072, 8192, 64, 64, True, False, True): (3, 128, 3, 4),
        (768, 3072, 8192, 128, 128, False, True, True): (4, 64, 2, 32),
        (768, 3072, 8192, 128, 128, True, False, True): (2, 64, 1, 32),
        (768, 3072, 16384, 16, 16, False, True, True): (1, 256, 2, 2),
        (768, 3072, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (768, 3072, 16384, 32, 32, False, True, True): (8, 128, 3, 4),
        (768, 3072, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6079-6114 / 第 6079-6114 行
````python
        (768, 3072, 16384, 64, 64, False, True, True): (1, 256, 3, 4),
        (768, 3072, 16384, 64, 64, True, False, True): (3, 256, 3, 4),
        (768, 3072, 16384, 128, 128, False, True, True): (3, 128, 1, 32),
        (768, 3072, 16384, 128, 128, True, False, True): (2, 128, 2, 32),
        (768, 3072, 32768, 16, 16, False, True, True): (1, 512, 3, 1),
        (768, 3072, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (768, 3072, 32768, 32, 32, False, True, True): (1, 256, 3, 4),
        (768, 3072, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (768, 3072, 32768, 64, 64, False, True, True): (2, 512, 3, 4),
        (768, 3072, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (768, 3072, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (768, 3072, 32768, 128, 128, True, False, True): (2, 256, 2, 32),
        (768, 3072, 50432, 16, 16, False, True, True): (1, 197, 3, 4),
        (768, 3072, 50432, 16, 16, True, False, True): (1, 197, 3, 4),
        (768, 3072, 50432, 32, 32, False, True, True): (1, 788, 2, 4),
        (768, 3072, 50432, 32, 32, True, False, True): (1, 394, 3, 4),
        (768, 3072, 50432, 64, 64, False, True, True): (1, 788, 3, 4),
        (768, 3072, 50432, 64, 64, True, False, True): (2, 788, 3, 4),
        (768, 3072, 50432, 128, 128, False, True, True): (1, 394, 1, 32),
        (768, 3072, 50432, 128, 128, True, False, True): (2, 394, 2, 32),
        (768, 3072, 65536, 16, 16, False, True, True): (1, 1024, 3, 1),
        (768, 3072, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (768, 3072, 65536, 32, 32, False, True, True): (1, 512, 3, 4),
        (768, 3072, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (768, 3072, 65536, 64, 64, False, True, True): (2, 1024, 3, 4),
        (768, 3072, 65536, 64, 64, True, False, True): (5, 1024, 3, 4),
        (768, 3072, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (768, 3072, 65536, 128, 128, True, False, True): (2, 512, 2, 32),
        (768, 3072, 131072, 16, 16, False, True, True): (1, 2048, 3, 1),
        (768, 3072, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (768, 3072, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (768, 3072, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (768, 3072, 131072, 64, 64, False, True, True): (1, 2048, 3, 4),
        (768, 3072, 131072, 64, 64, True, False, True): (2, 2048, 3, 4),
        (768, 3072, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (768, 3072, 131072, 128, 128, True, False, True): (1, 1024, 2, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6115-6150 / 第 6115-6150 行
````python
        (1024, 1024, 256, 16, 16, False, True, True): (4, 8, 3, 2),
        (1024, 1024, 256, 16, 16, True, False, True): (2, 8, 3, 2),
        (1024, 1024, 256, 32, 32, False, True, True): (1, 8, 3, 4),
        (1024, 1024, 256, 32, 32, True, False, True): (1, 8, 3, 4),
        (1024, 1024, 256, 64, 64, False, True, True): (1, 4, 3, 8),
        (1024, 1024, 256, 64, 64, True, False, True): (2, 4, 3, 8),
        (1024, 1024, 256, 128, 128, False, True, True): (3, 2, 1, 32),
        (1024, 1024, 256, 128, 128, True, False, True): (5, 2, 1, 32),
        (1024, 1024, 512, 16, 16, False, True, True): (3, 8, 3, 4),
        (1024, 1024, 512, 16, 16, True, False, True): (3, 8, 3, 4),
        (1024, 1024, 512, 32, 32, False, True, True): (1, 16, 3, 4),
        (1024, 1024, 512, 32, 32, True, False, True): (3, 16, 3, 4),
        (1024, 1024, 512, 64, 64, False, True, True): (6, 8, 3, 8),
        (1024, 1024, 512, 64, 64, True, False, True): (8, 8, 3, 8),
        (1024, 1024, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (1024, 1024, 512, 128, 128, True, False, True): (1, 4, 1, 32),
        (1024, 1024, 1024, 16, 16, False, True, True): (4, 8, 3, 4),
        (1024, 1024, 1024, 16, 16, True, False, True): (1, 8, 3, 4),
        (1024, 1024, 1024, 32, 32, False, True, True): (4, 16, 4, 4),
        (1024, 1024, 1024, 32, 32, True, False, True): (5, 16, 3, 4),
        (1024, 1024, 1024, 64, 64, False, True, True): (6, 16, 3, 8),
        (1024, 1024, 1024, 64, 64, True, False, True): (3, 16, 2, 4),
        (1024, 1024, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (1024, 1024, 1024, 128, 128, True, False, True): (2, 8, 1, 32),
        (1024, 1024, 2048, 16, 16, False, True, True): (4, 16, 3, 4),
        (1024, 1024, 2048, 16, 16, True, False, True): (1, 16, 3, 4),
        (1024, 1024, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (1024, 1024, 2048, 32, 32, True, False, True): (2, 32, 3, 4),
        (1024, 1024, 2048, 64, 64, False, True, True): (4, 32, 2, 4),
        (1024, 1024, 2048, 64, 64, True, False, True): (8, 32, 2, 4),
        (1024, 1024, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (1024, 1024, 2048, 128, 128, True, False, True): (1, 16, 1, 32),
        (1024, 1024, 4096, 16, 16, False, True, True): (4, 32, 3, 4),
        (1024, 1024, 4096, 16, 16, True, False, True): (1, 64, 3, 2),
        (1024, 1024, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (1024, 1024, 4096, 32, 32, True, False, True): (1, 64, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6151-6186 / 第 6151-6186 行
````python
        (1024, 1024, 4096, 64, 64, False, True, True): (2, 64, 2, 4),
        (1024, 1024, 4096, 64, 64, True, False, True): (2, 64, 2, 4),
        (1024, 1024, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (1024, 1024, 4096, 128, 128, True, False, True): (4, 32, 1, 32),
        (1024, 1024, 8192, 16, 16, False, True, True): (1, 128, 3, 1),
        (1024, 1024, 8192, 16, 16, True, False, True): (1, 128, 3, 1),
        (1024, 1024, 8192, 32, 32, False, True, True): (1, 128, 3, 4),
        (1024, 1024, 8192, 32, 32, True, False, True): (1, 128, 3, 4),
        (1024, 1024, 8192, 64, 64, False, True, True): (2, 128, 2, 4),
        (1024, 1024, 8192, 64, 64, True, False, True): (2, 128, 2, 4),
        (1024, 1024, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (1024, 1024, 8192, 128, 128, True, False, True): (4, 64, 1, 32),
        (1024, 1024, 16384, 16, 16, False, True, True): (1, 128, 2, 4),
        (1024, 1024, 16384, 16, 16, True, False, True): (4, 256, 3, 1),
        (1024, 1024, 16384, 32, 32, False, True, True): (1, 256, 3, 4),
        (1024, 1024, 16384, 32, 32, True, False, True): (1, 256, 3, 4),
        (1024, 1024, 16384, 64, 64, False, True, True): (1, 256, 2, 4),
        (1024, 1024, 16384, 64, 64, True, False, True): (1, 256, 2, 4),
        (1024, 1024, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (1024, 1024, 16384, 128, 128, True, False, True): (4, 128, 1, 32),
        (1024, 1024, 32768, 16, 16, False, True, True): (1, 256, 2, 4),
        (1024, 1024, 32768, 16, 16, True, False, True): (4, 512, 3, 1),
        (1024, 1024, 32768, 32, 32, False, True, True): (1, 512, 3, 4),
        (1024, 1024, 32768, 32, 32, True, False, True): (1, 512, 3, 4),
        (1024, 1024, 32768, 64, 64, False, True, True): (1, 512, 2, 4),
        (1024, 1024, 32768, 64, 64, True, False, True): (1, 512, 2, 4),
        (1024, 1024, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (1024, 1024, 32768, 128, 128, True, False, True): (1, 256, 1, 32),
        (1024, 1024, 65536, 16, 16, False, True, True): (1, 512, 2, 4),
        (1024, 1024, 65536, 16, 16, True, False, True): (1, 1024, 3, 1),
        (1024, 1024, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (1024, 1024, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (1024, 1024, 65536, 64, 64, False, True, True): (1, 1024, 2, 4),
        (1024, 1024, 65536, 64, 64, True, False, True): (1, 1024, 2, 4),
        (1024, 1024, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (1024, 1024, 65536, 128, 128, True, False, True): (1, 512, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6187-6222 / 第 6187-6222 行
````python
        (1024, 1024, 131072, 16, 16, False, True, True): (4, 2048, 3, 1),
        (1024, 1024, 131072, 16, 16, True, False, True): (4, 2048, 3, 1),
        (1024, 1024, 131072, 32, 32, False, True, True): (1, 2048, 3, 4),
        (1024, 1024, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (1024, 1024, 131072, 64, 64, False, True, True): (1, 2048, 2, 4),
        (1024, 1024, 131072, 64, 64, True, False, True): (1, 2048, 2, 4),
        (1024, 1024, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (1024, 1024, 131072, 128, 128, True, False, True): (1, 1024, 1, 32),
        (1280, 5120, 65792, 16, 16, False, True, True): (1, 1028, 3, 1),
        (1280, 5120, 65792, 16, 16, True, False, True): (1, 257, 3, 4),
        (1280, 5120, 65792, 32, 32, False, True, True): (1, 514, 3, 4),
        (1280, 5120, 65792, 32, 32, True, False, True): (1, 514, 3, 4),
        (1280, 5120, 65792, 64, 64, False, True, True): (2, 1028, 3, 4),
        (1280, 5120, 65792, 64, 64, True, False, True): (1, 1028, 3, 4),
        (1280, 5120, 65792, 128, 128, False, True, True): (2, 514, 2, 32),
        (1280, 5120, 65792, 128, 128, True, False, True): (1, 514, 2, 32),
        (1536, 1536, 256, 16, 16, False, True, True): (5, 4, 3, 2),
        (1536, 1536, 256, 16, 16, True, False, True): (2, 2, 3, 4),
        (1536, 1536, 256, 32, 32, False, True, True): (1, 8, 2, 4),
        (1536, 1536, 256, 32, 32, True, False, True): (2, 4, 3, 4),
        (1536, 1536, 256, 64, 64, False, True, True): (1, 4, 3, 8),
        (1536, 1536, 256, 64, 64, True, False, True): (2, 4, 3, 8),
        (1536, 1536, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (1536, 1536, 256, 128, 128, True, False, True): (2, 2, 2, 32),
        (1536, 1536, 512, 16, 16, False, True, True): (1, 8, 3, 2),
        (1536, 1536, 512, 16, 16, True, False, True): (1, 8, 3, 2),
        (1536, 1536, 512, 32, 32, False, True, True): (1, 16, 3, 4),
        (1536, 1536, 512, 32, 32, True, False, True): (1, 16, 3, 4),
        (1536, 1536, 512, 64, 64, False, True, True): (3, 8, 3, 8),
        (1536, 1536, 512, 64, 64, True, False, True): (3, 8, 3, 8),
        (1536, 1536, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (1536, 1536, 512, 128, 128, True, False, True): (2, 4, 2, 32),
        (1536, 1536, 1024, 16, 16, False, True, True): (2, 8, 3, 4),
        (1536, 1536, 1024, 16, 16, True, False, True): (2, 8, 3, 4),
        (1536, 1536, 1024, 32, 32, False, True, True): (1, 16, 3, 4),
        (1536, 1536, 1024, 32, 32, True, False, True): (1, 16, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6223-6258 / 第 6223-6258 行
````python
        (1536, 1536, 1024, 64, 64, False, True, True): (2, 16, 3, 8),
        (1536, 1536, 1024, 64, 64, True, False, True): (2, 16, 3, 8),
        (1536, 1536, 1024, 128, 128, False, True, True): (3, 8, 1, 32),
        (1536, 1536, 1024, 128, 128, True, False, True): (1, 8, 2, 32),
        (1536, 1536, 2048, 16, 16, False, True, True): (1, 32, 3, 2),
        (1536, 1536, 2048, 16, 16, True, False, True): (1, 32, 3, 2),
        (1536, 1536, 2048, 32, 32, False, True, True): (3, 32, 2, 4),
        (1536, 1536, 2048, 32, 32, True, False, True): (4, 32, 3, 4),
        (1536, 1536, 2048, 64, 64, False, True, True): (1, 32, 3, 4),
        (1536, 1536, 2048, 64, 64, True, False, True): (1, 32, 3, 4),
        (1536, 1536, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (1536, 1536, 2048, 128, 128, True, False, True): (2, 16, 1, 32),
        (1536, 1536, 4096, 16, 16, False, True, True): (1, 64, 3, 2),
        (1536, 1536, 4096, 16, 16, True, False, True): (1, 16, 3, 4),
        (1536, 1536, 4096, 32, 32, False, True, True): (1, 64, 2, 4),
        (1536, 1536, 4096, 32, 32, True, False, True): (1, 64, 2, 4),
        (1536, 1536, 4096, 64, 64, False, True, True): (1, 64, 3, 4),
        (1536, 1536, 4096, 64, 64, True, False, True): (1, 64, 3, 4),
        (1536, 1536, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (1536, 1536, 4096, 128, 128, True, False, True): (4, 32, 2, 32),
        (1536, 1536, 8192, 16, 16, False, True, True): (1, 32, 3, 4),
        (1536, 1536, 8192, 16, 16, True, False, True): (5, 32, 3, 4),
        (1536, 1536, 8192, 32, 32, False, True, True): (1, 128, 2, 4),
        (1536, 1536, 8192, 32, 32, True, False, True): (1, 128, 2, 4),
        (1536, 1536, 8192, 64, 64, False, True, True): (1, 128, 3, 4),
        (1536, 1536, 8192, 64, 64, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (1536, 1536, 8192, 128, 128, True, False, True): (4, 64, 2, 32),
        (1536, 1536, 16384, 16, 16, False, True, True): (1, 64, 3, 4),
        (1536, 1536, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (1536, 1536, 16384, 32, 32, False, True, True): (1, 256, 2, 4),
        (1536, 1536, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 16384, 64, 64, False, True, True): (1, 256, 3, 4),
        (1536, 1536, 16384, 64, 64, True, False, True): (3, 256, 3, 4),
        (1536, 1536, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (1536, 1536, 16384, 128, 128, True, False, True): (4, 128, 2, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6259-6294 / 第 6259-6294 行
````python
        (1536, 1536, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (1536, 1536, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (1536, 1536, 32768, 32, 32, False, True, True): (1, 256, 3, 4),
        (1536, 1536, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (1536, 1536, 32768, 64, 64, False, True, True): (1, 512, 3, 4),
        (1536, 1536, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (1536, 1536, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (1536, 1536, 32768, 128, 128, True, False, True): (4, 256, 2, 32),
        (1536, 1536, 65536, 16, 16, False, True, True): (5, 256, 3, 4),
        (1536, 1536, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (1536, 1536, 65536, 32, 32, False, True, True): (1, 512, 3, 4),
        (1536, 1536, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (1536, 1536, 65536, 64, 64, False, True, True): (1, 1024, 3, 4),
        (1536, 1536, 65536, 64, 64, True, False, True): (1, 1024, 3, 4),
        (1536, 1536, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (1536, 1536, 65536, 128, 128, True, False, True): (4, 512, 2, 32),
        (1536, 1536, 131072, 16, 16, False, True, True): (3, 512, 3, 4),
        (1536, 1536, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (1536, 1536, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (1536, 1536, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (1536, 1536, 131072, 64, 64, False, True, True): (1, 2048, 3, 4),
        (1536, 1536, 131072, 64, 64, True, False, True): (1, 2048, 3, 4),
        (1536, 1536, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (1536, 1536, 131072, 128, 128, True, False, True): (4, 1024, 2, 32),
        (2048, 2048, 256, 16, 16, False, True, True): (1, 4, 3, 4),
        (2048, 2048, 256, 16, 16, True, False, True): (1, 4, 3, 4),
        (2048, 2048, 256, 32, 32, False, True, True): (3, 8, 3, 4),
        (2048, 2048, 256, 32, 32, True, False, True): (3, 8, 3, 4),
        (2048, 2048, 256, 64, 64, False, True, True): (4, 4, 4, 8),
        (2048, 2048, 256, 64, 64, True, False, True): (8, 4, 4, 8),
        (2048, 2048, 256, 128, 128, False, True, True): (3, 2, 1, 32),
        (2048, 2048, 256, 128, 128, True, False, True): (3, 2, 1, 32),
        (2048, 2048, 512, 16, 16, False, True, True): (4, 8, 3, 2),
        (2048, 2048, 512, 16, 16, True, False, True): (4, 8, 3, 2),
        (2048, 2048, 512, 32, 32, False, True, True): (3, 8, 3, 4),
        (2048, 2048, 512, 32, 32, True, False, True): (1, 16, 2, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6295-6330 / 第 6295-6330 行
````python
        (2048, 2048, 512, 64, 64, False, True, True): (4, 8, 2, 4),
        (2048, 2048, 512, 64, 64, True, False, True): (4, 8, 2, 4),
        (2048, 2048, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (2048, 2048, 512, 128, 128, True, False, True): (4, 4, 1, 32),
        (2048, 2048, 1024, 16, 16, False, True, True): (4, 8, 3, 4),
        (2048, 2048, 1024, 16, 16, True, False, True): (4, 8, 3, 4),
        (2048, 2048, 1024, 32, 32, False, True, True): (4, 16, 3, 4),
        (2048, 2048, 1024, 32, 32, True, False, True): (1, 16, 3, 4),
        (2048, 2048, 1024, 64, 64, False, True, True): (2, 16, 2, 4),
        (2048, 2048, 1024, 64, 64, True, False, True): (2, 16, 2, 4),
        (2048, 2048, 1024, 128, 128, False, True, True): (8, 8, 1, 32),
        (2048, 2048, 1024, 128, 128, True, False, True): (4, 8, 1, 32),
        (2048, 2048, 2048, 16, 16, False, True, True): (4, 32, 3, 1),
        (2048, 2048, 2048, 16, 16, True, False, True): (3, 32, 3, 2),
        (2048, 2048, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (2048, 2048, 2048, 32, 32, True, False, True): (1, 32, 3, 4),
        (2048, 2048, 2048, 64, 64, False, True, True): (2, 32, 2, 4),
        (2048, 2048, 2048, 64, 64, True, False, True): (2, 32, 2, 4),
        (2048, 2048, 2048, 128, 128, False, True, True): (6, 16, 1, 32),
        (2048, 2048, 2048, 128, 128, True, False, True): (4, 16, 1, 32),
        (2048, 2048, 4096, 16, 16, False, True, True): (4, 64, 3, 1),
        (2048, 2048, 4096, 16, 16, True, False, True): (1, 64, 3, 1),
        (2048, 2048, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (2048, 2048, 4096, 32, 32, True, False, True): (4, 64, 3, 4),
        (2048, 2048, 4096, 64, 64, False, True, True): (2, 64, 2, 4),
        (2048, 2048, 4096, 64, 64, True, False, True): (2, 64, 2, 4),
        (2048, 2048, 4096, 128, 128, False, True, True): (4, 32, 1, 32),
        (2048, 2048, 4096, 128, 128, True, False, True): (4, 32, 1, 32),
        (2048, 2048, 8192, 16, 16, False, True, True): (4, 128, 3, 1),
        (2048, 2048, 8192, 16, 16, True, False, True): (1, 128, 3, 1),
        (2048, 2048, 8192, 32, 32, False, True, True): (4, 128, 3, 4),
        (2048, 2048, 8192, 32, 32, True, False, True): (4, 64, 3, 4),
        (2048, 2048, 8192, 64, 64, False, True, True): (1, 128, 2, 4),
        (2048, 2048, 8192, 64, 64, True, False, True): (2, 128, 2, 4),
        (2048, 2048, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (2048, 2048, 8192, 128, 128, True, False, True): (4, 64, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6331-6366 / 第 6331-6366 行
````python
        (2048, 2048, 16384, 16, 16, False, True, True): (4, 256, 3, 1),
        (2048, 2048, 16384, 16, 16, True, False, True): (1, 256, 3, 1),
        (2048, 2048, 16384, 32, 32, False, True, True): (1, 256, 3, 4),
        (2048, 2048, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (2048, 2048, 16384, 64, 64, False, True, True): (1, 256, 2, 4),
        (2048, 2048, 16384, 64, 64, True, False, True): (1, 256, 2, 4),
        (2048, 2048, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (2048, 2048, 16384, 128, 128, True, False, True): (4, 128, 1, 32),
        (2048, 2048, 32768, 16, 16, False, True, True): (8, 512, 3, 1),
        (2048, 2048, 32768, 16, 16, True, False, True): (1, 512, 3, 1),
        (2048, 2048, 32768, 32, 32, False, True, True): (1, 512, 3, 4),
        (2048, 2048, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (2048, 2048, 32768, 64, 64, False, True, True): (1, 512, 2, 4),
        (2048, 2048, 32768, 64, 64, True, False, True): (1, 512, 2, 4),
        (2048, 2048, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (2048, 2048, 32768, 128, 128, True, False, True): (4, 256, 1, 32),
        (2048, 2048, 65536, 16, 16, False, True, True): (4, 1024, 3, 1),
        (2048, 2048, 65536, 16, 16, True, False, True): (1, 1024, 3, 1),
        (2048, 2048, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (2048, 2048, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (2048, 2048, 65536, 64, 64, False, True, True): (1, 1024, 2, 4),
        (2048, 2048, 65536, 64, 64, True, False, True): (1, 1024, 2, 4),
        (2048, 2048, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (2048, 2048, 65536, 128, 128, True, False, True): (4, 512, 1, 32),
        (2048, 2048, 131072, 16, 16, False, True, True): (4, 2048, 3, 1),
        (2048, 2048, 131072, 16, 16, True, False, True): (1, 2048, 3, 1),
        (2048, 2048, 131072, 32, 32, False, True, True): (1, 2048, 3, 4),
        (2048, 2048, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (2048, 2048, 131072, 64, 64, False, True, True): (1, 2048, 2, 4),
        (2048, 2048, 131072, 64, 64, True, False, True): (1, 2048, 2, 4),
        (2048, 2048, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (2048, 2048, 131072, 128, 128, True, False, True): (4, 1024, 1, 32),
        (3072, 768, 256, 16, 16, False, True, True): (4, 4, 3, 2),
        (3072, 768, 256, 16, 16, True, False, True): (1, 2, 6, 4),
        (3072, 768, 256, 32, 32, False, True, True): (1, 4, 6, 4),
        (3072, 768, 256, 32, 32, True, False, True): (5, 4, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6367-6402 / 第 6367-6402 行
````python
        (3072, 768, 256, 64, 64, False, True, True): (4, 4, 3, 8),
        (3072, 768, 256, 64, 64, True, False, True): (4, 4, 3, 8),
        (3072, 768, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (3072, 768, 256, 128, 128, True, False, True): (5, 2, 1, 32),
        (3072, 768, 512, 16, 16, False, True, True): (4, 4, 3, 4),
        (3072, 768, 512, 16, 16, True, False, True): (1, 4, 3, 4),
        (3072, 768, 512, 32, 32, False, True, True): (3, 8, 3, 4),
        (3072, 768, 512, 32, 32, True, False, True): (3, 8, 3, 4),
        (3072, 768, 512, 64, 64, False, True, True): (2, 8, 3, 8),
        (3072, 768, 512, 64, 64, True, False, True): (2, 8, 3, 8),
        (3072, 768, 512, 128, 128, False, True, True): (1, 4, 2, 32),
        (3072, 768, 512, 128, 128, True, False, True): (1, 4, 1, 32),
        (3072, 768, 1024, 16, 16, False, True, True): (1, 16, 3, 2),
        (3072, 768, 1024, 16, 16, True, False, True): (3, 16, 3, 2),
        (3072, 768, 1024, 32, 32, False, True, True): (1, 16, 3, 4),
        (3072, 768, 1024, 32, 32, True, False, True): (3, 16, 3, 4),
        (3072, 768, 1024, 64, 64, False, True, True): (4, 16, 3, 8),
        (3072, 768, 1024, 64, 64, True, False, True): (4, 16, 3, 4),
        (3072, 768, 1024, 128, 128, False, True, True): (5, 8, 1, 32),
        (3072, 768, 1024, 128, 128, True, False, True): (5, 8, 1, 32),
        (3072, 768, 2048, 16, 16, False, True, True): (4, 32, 3, 2),
        (3072, 768, 2048, 16, 16, True, False, True): (1, 32, 3, 2),
        (3072, 768, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (3072, 768, 2048, 32, 32, True, False, True): (1, 32, 2, 4),
        (3072, 768, 2048, 64, 64, False, True, True): (2, 32, 3, 4),
        (3072, 768, 2048, 64, 64, True, False, True): (4, 32, 3, 4),
        (3072, 768, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (3072, 768, 2048, 128, 128, True, False, True): (1, 16, 1, 32),
        (3072, 768, 4096, 16, 16, False, True, True): (3, 64, 3, 2),
        (3072, 768, 4096, 16, 16, True, False, True): (1, 64, 3, 2),
        (3072, 768, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (3072, 768, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
        (3072, 768, 4096, 64, 64, False, True, True): (2, 64, 3, 4),
        (3072, 768, 4096, 64, 64, True, False, True): (2, 64, 3, 4),
        (3072, 768, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (3072, 768, 4096, 128, 128, True, False, True): (1, 32, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6403-6438 / 第 6403-6438 行
````python
        (3072, 768, 8192, 16, 16, False, True, True): (4, 128, 3, 1),
        (3072, 768, 8192, 16, 16, True, False, True): (1, 32, 3, 4),
        (3072, 768, 8192, 32, 32, False, True, True): (1, 64, 3, 4),
        (3072, 768, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (3072, 768, 8192, 64, 64, False, True, True): (2, 128, 3, 4),
        (3072, 768, 8192, 64, 64, True, False, True): (2, 128, 3, 4),
        (3072, 768, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (3072, 768, 8192, 128, 128, True, False, True): (1, 64, 1, 32),
        (3072, 768, 16384, 16, 16, False, True, True): (4, 256, 3, 1),
        (3072, 768, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (3072, 768, 16384, 32, 32, False, True, True): (1, 128, 3, 4),
        (3072, 768, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (3072, 768, 16384, 64, 64, False, True, True): (2, 256, 3, 4),
        (3072, 768, 16384, 64, 64, True, False, True): (2, 256, 3, 4),
        (3072, 768, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (3072, 768, 16384, 128, 128, True, False, True): (1, 128, 1, 32),
        (3072, 768, 32768, 16, 16, False, True, True): (4, 512, 3, 1),
        (3072, 768, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (3072, 768, 32768, 32, 32, False, True, True): (1, 256, 3, 4),
        (3072, 768, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (3072, 768, 32768, 64, 64, False, True, True): (2, 512, 3, 4),
        (3072, 768, 32768, 64, 64, True, False, True): (2, 512, 3, 4),
        (3072, 768, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (3072, 768, 32768, 128, 128, True, False, True): (1, 256, 1, 32),
        (3072, 768, 50432, 16, 16, False, True, True): (4, 788, 3, 1),
        (3072, 768, 50432, 16, 16, True, False, True): (1, 197, 3, 4),
        (3072, 768, 50432, 32, 32, False, True, True): (1, 394, 3, 4),
        (3072, 768, 50432, 32, 32, True, False, True): (1, 394, 3, 4),
        (3072, 768, 50432, 64, 64, False, True, True): (1, 788, 3, 4),
        (3072, 768, 50432, 64, 64, True, False, True): (2, 788, 3, 4),
        (3072, 768, 50432, 128, 128, False, True, True): (1, 394, 1, 32),
        (3072, 768, 50432, 128, 128, True, False, True): (1, 394, 1, 32),
        (3072, 768, 65536, 16, 16, False, True, True): (4, 1024, 3, 1),
        (3072, 768, 65536, 16, 16, True, False, True): (1, 256, 3, 4),
        (3072, 768, 65536, 32, 32, False, True, True): (1, 512, 3, 4),
        (3072, 768, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6439-6474 / 第 6439-6474 行
````python
        (3072, 768, 65536, 64, 64, False, True, True): (2, 1024, 3, 4),
        (3072, 768, 65536, 64, 64, True, False, True): (2, 1024, 3, 4),
        (3072, 768, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (3072, 768, 65536, 128, 128, True, False, True): (1, 512, 1, 32),
        (3072, 768, 131072, 16, 16, False, True, True): (4, 2048, 3, 1),
        (3072, 768, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (3072, 768, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (3072, 768, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (3072, 768, 131072, 64, 64, False, True, True): (2, 2048, 3, 4),
        (3072, 768, 131072, 64, 64, True, False, True): (2, 2048, 3, 4),
        (3072, 768, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (3072, 768, 131072, 128, 128, True, False, True): (1, 1024, 1, 32),
        (3072, 3072, 256, 16, 16, False, True, True): (1, 4, 5, 2),
        (3072, 3072, 256, 16, 16, True, False, True): (1, 4, 3, 2),
        (3072, 3072, 256, 32, 32, False, True, True): (1, 4, 4, 4),
        (3072, 3072, 256, 32, 32, True, False, True): (1, 4, 3, 4),
        (3072, 3072, 256, 64, 64, False, True, True): (2, 4, 3, 8),
        (3072, 3072, 256, 64, 64, True, False, True): (2, 4, 3, 8),
        (3072, 3072, 256, 128, 128, False, True, True): (6, 2, 1, 32),
        (3072, 3072, 256, 128, 128, True, False, True): (8, 2, 2, 32),
        (3072, 3072, 512, 16, 16, False, True, True): (2, 4, 3, 4),
        (3072, 3072, 512, 16, 16, True, False, True): (2, 4, 3, 4),
        (3072, 3072, 512, 32, 32, False, True, True): (2, 8, 3, 4),
        (3072, 3072, 512, 32, 32, True, False, True): (2, 8, 3, 4),
        (3072, 3072, 512, 64, 64, False, True, True): (2, 8, 3, 8),
        (3072, 3072, 512, 64, 64, True, False, True): (2, 8, 3, 8),
        (3072, 3072, 512, 128, 128, False, True, True): (5, 4, 1, 32),
        (3072, 3072, 512, 128, 128, True, False, True): (5, 4, 2, 32),
        (3072, 3072, 1024, 16, 16, False, True, True): (1, 16, 3, 2),
        (3072, 3072, 1024, 16, 16, True, False, True): (1, 16, 3, 2),
        (3072, 3072, 1024, 32, 32, False, True, True): (2, 16, 3, 4),
        (3072, 3072, 1024, 32, 32, True, False, True): (1, 16, 3, 4),
        (3072, 3072, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (3072, 3072, 1024, 64, 64, True, False, True): (1, 16, 3, 4),
        (3072, 3072, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (3072, 3072, 1024, 128, 128, True, False, True): (3, 8, 2, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6475-6510 / 第 6475-6510 行
````python
        (3072, 3072, 2048, 16, 16, False, True, True): (1, 32, 3, 2),
        (3072, 3072, 2048, 16, 16, True, False, True): (1, 16, 2, 4),
        (3072, 3072, 2048, 32, 32, False, True, True): (1, 32, 2, 4),
        (3072, 3072, 2048, 32, 32, True, False, True): (1, 32, 3, 4),
        (3072, 3072, 2048, 64, 64, False, True, True): (1, 32, 3, 4),
        (3072, 3072, 2048, 64, 64, True, False, True): (1, 32, 3, 4),
        (3072, 3072, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (3072, 3072, 2048, 128, 128, True, False, True): (4, 16, 2, 32),
        (3072, 3072, 4096, 16, 16, False, True, True): (2, 16, 3, 4),
        (3072, 3072, 4096, 16, 16, True, False, True): (2, 16, 3, 4),
        (3072, 3072, 4096, 32, 32, False, True, True): (1, 64, 2, 4),
        (3072, 3072, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
        (3072, 3072, 4096, 64, 64, False, True, True): (1, 64, 3, 4),
        (3072, 3072, 4096, 64, 64, True, False, True): (1, 64, 3, 4),
        (3072, 3072, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (3072, 3072, 4096, 128, 128, True, False, True): (2, 32, 2, 32),
        (3072, 3072, 8192, 16, 16, False, True, True): (2, 32, 3, 4),
        (3072, 3072, 8192, 16, 16, True, False, True): (2, 32, 3, 4),
        (3072, 3072, 8192, 32, 32, False, True, True): (1, 64, 3, 4),
        (3072, 3072, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (3072, 3072, 8192, 64, 64, False, True, True): (1, 128, 3, 4),
        (3072, 3072, 8192, 64, 64, True, False, True): (1, 128, 3, 4),
        (3072, 3072, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (3072, 3072, 8192, 128, 128, True, False, True): (4, 64, 2, 32),
        (3072, 3072, 16384, 16, 16, False, True, True): (2, 64, 3, 4),
        (3072, 3072, 16384, 16, 16, True, False, True): (1, 64, 3, 4),
        (3072, 3072, 16384, 32, 32, False, True, True): (1, 128, 3, 4),
        (3072, 3072, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (3072, 3072, 16384, 64, 64, False, True, True): (1, 256, 3, 4),
        (3072, 3072, 16384, 64, 64, True, False, True): (1, 256, 3, 4),
        (3072, 3072, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (3072, 3072, 16384, 128, 128, True, False, True): (4, 128, 2, 32),
        (3072, 3072, 32768, 16, 16, False, True, True): (3, 128, 3, 4),
        (3072, 3072, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (3072, 3072, 32768, 32, 32, False, True, True): (1, 256, 3, 4),
        (3072, 3072, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6511-6546 / 第 6511-6546 行
````python
        (3072, 3072, 32768, 64, 64, False, True, True): (1, 512, 3, 4),
        (3072, 3072, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (3072, 3072, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (3072, 3072, 32768, 128, 128, True, False, True): (4, 256, 2, 32),
        (3072, 3072, 65536, 16, 16, False, True, True): (5, 256, 3, 4),
        (3072, 3072, 65536, 16, 16, True, False, True): (2, 256, 3, 4),
        (3072, 3072, 65536, 32, 32, False, True, True): (1, 512, 3, 4),
        (3072, 3072, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (3072, 3072, 65536, 64, 64, False, True, True): (1, 1024, 3, 4),
        (3072, 3072, 65536, 64, 64, True, False, True): (1, 1024, 3, 4),
        (3072, 3072, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (3072, 3072, 65536, 128, 128, True, False, True): (4, 512, 2, 32),
        (3072, 3072, 131072, 16, 16, False, True, True): (5, 512, 3, 4),
        (3072, 3072, 131072, 16, 16, True, False, True): (1, 512, 3, 4),
        (3072, 3072, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (3072, 3072, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (3072, 3072, 131072, 64, 64, False, True, True): (1, 2048, 3, 4),
        (3072, 3072, 131072, 64, 64, True, False, True): (1, 2048, 3, 4),
        (3072, 3072, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (3072, 3072, 131072, 128, 128, True, False, True): (4, 1024, 2, 32),
        (4096, 4096, 256, 16, 16, False, True, True): (1, 4, 3, 2),
        (4096, 4096, 256, 16, 16, True, False, True): (1, 2, 3, 4),
        (4096, 4096, 256, 32, 32, False, True, True): (4, 4, 4, 4),
        (4096, 4096, 256, 32, 32, True, False, True): (4, 4, 4, 4),
        (4096, 4096, 256, 64, 64, False, True, True): (1, 4, 3, 8),
        (4096, 4096, 256, 64, 64, True, False, True): (4, 4, 2, 4),
        (4096, 4096, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (4096, 4096, 256, 128, 128, True, False, True): (3, 2, 1, 32),
        (4096, 4096, 512, 16, 16, False, True, True): (1, 4, 3, 4),
        (4096, 4096, 512, 16, 16, True, False, True): (5, 8, 3, 2),
        (4096, 4096, 512, 32, 32, False, True, True): (4, 8, 3, 4),
        (4096, 4096, 512, 32, 32, True, False, True): (4, 8, 3, 4),
        (4096, 4096, 512, 64, 64, False, True, True): (1, 8, 2, 4),
        (4096, 4096, 512, 64, 64, True, False, True): (1, 8, 2, 4),
        (4096, 4096, 512, 128, 128, False, True, True): (4, 4, 1, 32),
        (4096, 4096, 512, 128, 128, True, False, True): (4, 4, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6547-6582 / 第 6547-6582 行
````python
        (4096, 4096, 1024, 16, 16, False, True, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 16, 16, True, False, True): (1, 8, 3, 4),
        (4096, 4096, 1024, 32, 32, False, True, True): (1, 16, 3, 4),
        (4096, 4096, 1024, 32, 32, True, False, True): (1, 16, 3, 4),
        (4096, 4096, 1024, 64, 64, False, True, True): (4, 16, 2, 4),
        (4096, 4096, 1024, 64, 64, True, False, True): (4, 16, 2, 4),
        (4096, 4096, 1024, 128, 128, False, True, True): (4, 8, 1, 32),
        (4096, 4096, 1024, 128, 128, True, False, True): (4, 8, 1, 32),
        (4096, 4096, 2048, 16, 16, False, True, True): (1, 32, 3, 1),
        (4096, 4096, 2048, 16, 16, True, False, True): (6, 8, 3, 4),
        (4096, 4096, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (4096, 4096, 2048, 32, 32, True, False, True): (1, 32, 3, 4),
        (4096, 4096, 2048, 64, 64, False, True, True): (4, 32, 2, 4),
        (4096, 4096, 2048, 64, 64, True, False, True): (4, 32, 2, 4),
        (4096, 4096, 2048, 128, 128, False, True, True): (4, 16, 1, 32),
        (4096, 4096, 2048, 128, 128, True, False, True): (4, 16, 1, 32),
        (4096, 4096, 4096, 16, 16, False, True, True): (1, 16, 3, 4),
        (4096, 4096, 4096, 16, 16, True, False, True): (1, 64, 3, 1),
        (4096, 4096, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (4096, 4096, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
        (4096, 4096, 4096, 64, 64, False, True, True): (4, 64, 2, 4),
        (4096, 4096, 4096, 64, 64, True, False, True): (4, 64, 2, 4),
        (4096, 4096, 4096, 128, 128, False, True, True): (4, 32, 1, 32),
        (4096, 4096, 4096, 128, 128, True, False, True): (4, 32, 1, 32),
        (4096, 4096, 8192, 16, 16, False, True, True): (4, 128, 3, 1),
        (4096, 4096, 8192, 16, 16, True, False, True): (1, 128, 3, 1),
        (4096, 4096, 8192, 32, 32, False, True, True): (1, 128, 3, 4),
        (4096, 4096, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (4096, 4096, 8192, 64, 64, False, True, True): (4, 128, 2, 4),
        (4096, 4096, 8192, 64, 64, True, False, True): (4, 128, 2, 4),
        (4096, 4096, 8192, 128, 128, False, True, True): (4, 64, 1, 32),
        (4096, 4096, 8192, 128, 128, True, False, True): (4, 64, 1, 32),
        (4096, 4096, 16384, 16, 16, False, True, True): (1, 64, 3, 4),
        (4096, 4096, 16384, 16, 16, True, False, True): (1, 256, 3, 1),
        (4096, 4096, 16384, 32, 32, False, True, True): (1, 256, 3, 4),
        (4096, 4096, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6583-6618 / 第 6583-6618 行
````python
        (4096, 4096, 16384, 64, 64, False, True, True): (4, 256, 2, 4),
        (4096, 4096, 16384, 64, 64, True, False, True): (4, 256, 2, 4),
        (4096, 4096, 16384, 128, 128, False, True, True): (4, 128, 1, 32),
        (4096, 4096, 16384, 128, 128, True, False, True): (4, 128, 1, 32),
        (4096, 4096, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (4096, 4096, 32768, 16, 16, True, False, True): (1, 512, 3, 1),
        (4096, 4096, 32768, 32, 32, False, True, True): (1, 512, 3, 4),
        (4096, 4096, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (4096, 4096, 32768, 64, 64, False, True, True): (4, 512, 2, 4),
        (4096, 4096, 32768, 64, 64, True, False, True): (4, 512, 2, 4),
        (4096, 4096, 32768, 128, 128, False, True, True): (4, 256, 1, 32),
        (4096, 4096, 32768, 128, 128, True, False, True): (4, 256, 1, 32),
        (4096, 4096, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (4096, 4096, 65536, 16, 16, True, False, True): (1, 1024, 3, 1),
        (4096, 4096, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (4096, 4096, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (4096, 4096, 65536, 64, 64, False, True, True): (4, 1024, 2, 4),
        (4096, 4096, 65536, 64, 64, True, False, True): (2, 1024, 2, 4),
        (4096, 4096, 65536, 128, 128, False, True, True): (4, 512, 1, 32),
        (4096, 4096, 65536, 128, 128, True, False, True): (4, 512, 1, 32),
        (4096, 4096, 131072, 16, 16, False, True, True): (2, 2048, 3, 1),
        (4096, 4096, 131072, 16, 16, True, False, True): (1, 2048, 3, 1),
        (4096, 4096, 131072, 32, 32, False, True, True): (2, 2048, 3, 4),
        (4096, 4096, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (4096, 4096, 131072, 64, 64, False, True, True): (2, 2048, 2, 4),
        (4096, 4096, 131072, 64, 64, True, False, True): (2, 2048, 2, 4),
        (4096, 4096, 131072, 128, 128, False, True, True): (4, 1024, 1, 32),
        (4096, 4096, 131072, 128, 128, True, False, True): (4, 1024, 1, 32),
        (5120, 1280, 65792, 16, 16, False, True, True): (2, 1028, 3, 1),
        (5120, 1280, 65792, 16, 16, True, False, True): (1, 257, 3, 4),
        (5120, 1280, 65792, 32, 32, False, True, True): (1, 514, 3, 4),
        (5120, 1280, 65792, 32, 32, True, False, True): (1, 514, 3, 4),
        (5120, 1280, 65792, 64, 64, False, True, True): (1, 1028, 3, 4),
        (5120, 1280, 65792, 64, 64, True, False, True): (5, 1028, 3, 4),
        (5120, 1280, 65792, 128, 128, False, True, True): (1, 514, 1, 32),
        (5120, 1280, 65792, 128, 128, True, False, True): (4, 514, 2, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6619-6654 / 第 6619-6654 行
````python
        (6144, 6144, 256, 16, 16, False, True, True): (2, 2, 3, 4),
        (6144, 6144, 256, 16, 16, True, False, True): (2, 2, 3, 4),
        (6144, 6144, 256, 32, 32, False, True, True): (2, 4, 3, 4),
        (6144, 6144, 256, 32, 32, True, False, True): (2, 4, 3, 4),
        (6144, 6144, 256, 64, 64, False, True, True): (1, 4, 3, 4),
        (6144, 6144, 256, 64, 64, True, False, True): (1, 4, 3, 4),
        (6144, 6144, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (6144, 6144, 256, 128, 128, True, False, True): (5, 2, 2, 32),
        (6144, 6144, 512, 16, 16, False, True, True): (4, 8, 3, 2),
        (6144, 6144, 512, 16, 16, True, False, True): (4, 8, 3, 2),
        (6144, 6144, 512, 32, 32, False, True, True): (2, 8, 3, 4),
        (6144, 6144, 512, 32, 32, True, False, True): (2, 8, 3, 4),
        (6144, 6144, 512, 64, 64, False, True, True): (1, 8, 3, 4),
        (6144, 6144, 512, 64, 64, True, False, True): (1, 8, 3, 4),
        (6144, 6144, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (6144, 6144, 512, 128, 128, True, False, True): (4, 4, 2, 32),
        (6144, 6144, 1024, 16, 16, False, True, True): (4, 16, 3, 2),
        (6144, 6144, 1024, 16, 16, True, False, True): (4, 4, 3, 4),
        (6144, 6144, 1024, 32, 32, False, True, True): (1, 16, 3, 4),
        (6144, 6144, 1024, 32, 32, True, False, True): (1, 16, 3, 4),
        (6144, 6144, 1024, 64, 64, False, True, True): (1, 16, 3, 4),
        (6144, 6144, 1024, 64, 64, True, False, True): (1, 16, 3, 4),
        (6144, 6144, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (6144, 6144, 1024, 128, 128, True, False, True): (4, 8, 2, 32),
        (6144, 6144, 2048, 16, 16, False, True, True): (1, 8, 3, 4),
        (6144, 6144, 2048, 16, 16, True, False, True): (4, 8, 3, 4),
        (6144, 6144, 2048, 32, 32, False, True, True): (1, 16, 3, 4),
        (6144, 6144, 2048, 32, 32, True, False, True): (1, 16, 3, 4),
        (6144, 6144, 2048, 64, 64, False, True, True): (1, 32, 3, 4),
        (6144, 6144, 2048, 64, 64, True, False, True): (3, 32, 3, 4),
        (6144, 6144, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (6144, 6144, 2048, 128, 128, True, False, True): (1, 16, 2, 32),
        (6144, 6144, 4096, 16, 16, False, True, True): (3, 16, 3, 4),
        (6144, 6144, 4096, 16, 16, True, False, True): (4, 16, 3, 4),
        (6144, 6144, 4096, 32, 32, False, True, True): (1, 32, 3, 4),
        (6144, 6144, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6655-6690 / 第 6655-6690 行
````python
        (6144, 6144, 4096, 64, 64, False, True, True): (1, 64, 3, 4),
        (6144, 6144, 4096, 64, 64, True, False, True): (1, 64, 3, 4),
        (6144, 6144, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (6144, 6144, 4096, 128, 128, True, False, True): (4, 32, 2, 32),
        (6144, 6144, 8192, 16, 16, False, True, True): (1, 32, 3, 4),
        (6144, 6144, 8192, 16, 16, True, False, True): (4, 32, 3, 4),
        (6144, 6144, 8192, 32, 32, False, True, True): (1, 64, 3, 4),
        (6144, 6144, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (6144, 6144, 8192, 64, 64, False, True, True): (1, 128, 3, 4),
        (6144, 6144, 8192, 64, 64, True, False, True): (1, 128, 3, 4),
        (6144, 6144, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (6144, 6144, 8192, 128, 128, True, False, True): (4, 64, 2, 32),
        (6144, 6144, 16384, 16, 16, False, True, True): (1, 64, 3, 4),
        (6144, 6144, 16384, 16, 16, True, False, True): (4, 64, 3, 4),
        (6144, 6144, 16384, 32, 32, False, True, True): (1, 128, 3, 4),
        (6144, 6144, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (6144, 6144, 16384, 64, 64, False, True, True): (1, 256, 3, 4),
        (6144, 6144, 16384, 64, 64, True, False, True): (1, 256, 3, 4),
        (6144, 6144, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (6144, 6144, 16384, 128, 128, True, False, True): (4, 128, 2, 32),
        (6144, 6144, 32768, 16, 16, False, True, True): (1, 128, 3, 4),
        (6144, 6144, 32768, 16, 16, True, False, True): (4, 128, 3, 4),
        (6144, 6144, 32768, 32, 32, False, True, True): (1, 256, 3, 4),
        (6144, 6144, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (6144, 6144, 32768, 64, 64, False, True, True): (1, 512, 3, 4),
        (6144, 6144, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (6144, 6144, 32768, 128, 128, False, True, True): (1, 256, 1, 32),
        (6144, 6144, 32768, 128, 128, True, False, True): (4, 256, 2, 32),
        (6144, 6144, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (6144, 6144, 65536, 16, 16, True, False, True): (2, 256, 3, 4),
        (6144, 6144, 65536, 32, 32, False, True, True): (1, 512, 3, 4),
        (6144, 6144, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (6144, 6144, 65536, 64, 64, False, True, True): (1, 1024, 3, 4),
        (6144, 6144, 65536, 64, 64, True, False, True): (1, 1024, 3, 4),
        (6144, 6144, 65536, 128, 128, False, True, True): (1, 512, 1, 32),
        (6144, 6144, 65536, 128, 128, True, False, True): (4, 512, 2, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6691-6726 / 第 6691-6726 行
````python
        (6144, 6144, 131072, 16, 16, False, True, True): (1, 512, 3, 4),
        (6144, 6144, 131072, 16, 16, True, False, True): (2, 512, 3, 4),
        (6144, 6144, 131072, 32, 32, False, True, True): (1, 1024, 3, 4),
        (6144, 6144, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (6144, 6144, 131072, 64, 64, False, True, True): (1, 2048, 3, 4),
        (6144, 6144, 131072, 64, 64, True, False, True): (1, 2048, 3, 4),
        (6144, 6144, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (6144, 6144, 131072, 128, 128, True, False, True): (4, 1024, 2, 32),
        (8192, 8192, 256, 16, 16, False, True, True): (2, 2, 4, 4),
        (8192, 8192, 256, 16, 16, True, False, True): (1, 1, 3, 4),
        (8192, 8192, 256, 32, 32, False, True, True): (2, 4, 3, 4),
        (8192, 8192, 256, 32, 32, True, False, True): (2, 4, 3, 4),
        (8192, 8192, 256, 64, 64, False, True, True): (4, 4, 2, 4),
        (8192, 8192, 256, 64, 64, True, False, True): (4, 4, 2, 4),
        (8192, 8192, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (8192, 8192, 256, 128, 128, True, False, True): (4, 2, 1, 32),
        (8192, 8192, 512, 16, 16, False, True, True): (1, 4, 3, 4),
        (8192, 8192, 512, 16, 16, True, False, True): (3, 4, 3, 4),
        (8192, 8192, 512, 32, 32, False, True, True): (1, 8, 3, 4),
        (8192, 8192, 512, 32, 32, True, False, True): (6, 8, 3, 4),
        (8192, 8192, 512, 64, 64, False, True, True): (4, 8, 2, 4),
        (8192, 8192, 512, 64, 64, True, False, True): (4, 8, 2, 4),
        (8192, 8192, 512, 128, 128, False, True, True): (4, 4, 1, 32),
        (8192, 8192, 512, 128, 128, True, False, True): (4, 4, 1, 32),
        (8192, 8192, 1024, 16, 16, False, True, True): (1, 4, 3, 4),
        (8192, 8192, 1024, 16, 16, True, False, True): (1, 32, 3, 1),
        (8192, 8192, 1024, 32, 32, False, True, True): (1, 16, 3, 4),
        (8192, 8192, 1024, 32, 32, True, False, True): (1, 16, 3, 4),
        (8192, 8192, 1024, 64, 64, False, True, True): (4, 16, 2, 4),
        (8192, 8192, 1024, 64, 64, True, False, True): (4, 16, 2, 4),
        (8192, 8192, 1024, 128, 128, False, True, True): (4, 8, 1, 32),
        (8192, 8192, 1024, 128, 128, True, False, True): (4, 8, 1, 32),
        (8192, 8192, 2048, 16, 16, False, True, True): (4, 8, 3, 4),
        (8192, 8192, 2048, 16, 16, True, False, True): (1, 32, 3, 1),
        (8192, 8192, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (8192, 8192, 2048, 32, 32, True, False, True): (1, 16, 4, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6727-6762 / 第 6727-6762 行
````python
        (8192, 8192, 2048, 64, 64, False, True, True): (4, 32, 2, 4),
        (8192, 8192, 2048, 64, 64, True, False, True): (4, 32, 2, 4),
        (8192, 8192, 2048, 128, 128, False, True, True): (4, 16, 1, 32),
        (8192, 8192, 2048, 128, 128, True, False, True): (4, 16, 1, 32),
        (8192, 8192, 4096, 16, 16, False, True, True): (3, 16, 3, 4),
        (8192, 8192, 4096, 16, 16, True, False, True): (2, 64, 3, 1),
        (8192, 8192, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (8192, 8192, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
        (8192, 8192, 4096, 64, 64, False, True, True): (4, 64, 2, 4),
        (8192, 8192, 4096, 64, 64, True, False, True): (2, 64, 2, 4),
        (8192, 8192, 4096, 128, 128, False, True, True): (4, 32, 1, 32),
        (8192, 8192, 4096, 128, 128, True, False, True): (4, 32, 1, 32),
        (8192, 8192, 8192, 16, 16, False, True, True): (2, 128, 3, 1),
        (8192, 8192, 8192, 16, 16, True, False, True): (2, 128, 3, 1),
        (8192, 8192, 8192, 32, 32, False, True, True): (1, 128, 3, 4),
        (8192, 8192, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (8192, 8192, 8192, 64, 64, False, True, True): (4, 128, 2, 4),
        (8192, 8192, 8192, 64, 64, True, False, True): (2, 128, 2, 4),
        (8192, 8192, 8192, 128, 128, False, True, True): (4, 64, 1, 32),
        (8192, 8192, 8192, 128, 128, True, False, True): (4, 64, 1, 32),
        (8192, 8192, 16384, 16, 16, False, True, True): (1, 64, 3, 4),
        (8192, 8192, 16384, 16, 16, True, False, True): (1, 256, 3, 1),
        (8192, 8192, 16384, 32, 32, False, True, True): (1, 256, 3, 4),
        (8192, 8192, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (8192, 8192, 16384, 64, 64, False, True, True): (2, 256, 2, 4),
        (8192, 8192, 16384, 64, 64, True, False, True): (2, 256, 2, 4),
        (8192, 8192, 16384, 128, 128, False, True, True): (4, 128, 1, 32),
        (8192, 8192, 16384, 128, 128, True, False, True): (4, 128, 1, 32),
        (8192, 8192, 32768, 16, 16, False, True, True): (1, 512, 3, 1),
        (8192, 8192, 32768, 16, 16, True, False, True): (1, 512, 3, 1),
        (8192, 8192, 32768, 32, 32, False, True, True): (1, 512, 3, 4),
        (8192, 8192, 32768, 32, 32, True, False, True): (1, 256, 3, 4),
        (8192, 8192, 32768, 64, 64, False, True, True): (2, 512, 2, 4),
        (8192, 8192, 32768, 64, 64, True, False, True): (2, 512, 2, 4),
        (8192, 8192, 32768, 128, 128, False, True, True): (4, 256, 1, 32),
        (8192, 8192, 32768, 128, 128, True, False, True): (4, 256, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6763-6798 / 第 6763-6798 行
````python
        (8192, 8192, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (8192, 8192, 65536, 16, 16, True, False, True): (1, 1024, 3, 1),
        (8192, 8192, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (8192, 8192, 65536, 32, 32, True, False, True): (1, 512, 3, 4),
        (8192, 8192, 65536, 64, 64, False, True, True): (4, 1024, 2, 4),
        (8192, 8192, 65536, 64, 64, True, False, True): (2, 1024, 2, 4),
        (8192, 8192, 65536, 128, 128, False, True, True): (4, 512, 1, 32),
        (8192, 8192, 65536, 128, 128, True, False, True): (4, 512, 1, 32),
        (8192, 8192, 131072, 16, 16, False, True, True): (1, 2048, 3, 1),
        (8192, 8192, 131072, 16, 16, True, False, True): (2, 2048, 3, 1),
        (8192, 8192, 131072, 32, 32, False, True, True): (4, 2048, 3, 4),
        (8192, 8192, 131072, 32, 32, True, False, True): (1, 1024, 3, 4),
        (8192, 8192, 131072, 64, 64, False, True, True): (2, 2048, 2, 4),
        (8192, 8192, 131072, 64, 64, True, False, True): (2, 2048, 2, 4),
        (8192, 8192, 131072, 128, 128, False, True, True): (4, 1024, 1, 32),
        (8192, 8192, 131072, 128, 128, True, False, True): (4, 1024, 1, 32),
        (16384, 16384, 256, 16, 16, False, True, True): (1, 2, 3, 4),
        (16384, 16384, 256, 16, 16, True, False, True): (1, 2, 3, 4),
        (16384, 16384, 256, 32, 32, False, True, True): (1, 4, 3, 4),
        (16384, 16384, 256, 32, 32, True, False, True): (1, 4, 3, 4),
        (16384, 16384, 256, 64, 64, False, True, True): (2, 4, 2, 4),
        (16384, 16384, 256, 64, 64, True, False, True): (2, 4, 2, 4),
        (16384, 16384, 256, 128, 128, False, True, True): (2, 2, 1, 32),
        (16384, 16384, 256, 128, 128, True, False, True): (2, 2, 1, 32),
        (16384, 16384, 512, 16, 16, False, True, True): (1, 2, 3, 4),
        (16384, 16384, 512, 16, 16, True, False, True): (5, 2, 3, 4),
        (16384, 16384, 512, 32, 32, False, True, True): (1, 8, 3, 4),
        (16384, 16384, 512, 32, 32, True, False, True): (1, 4, 3, 4),
        (16384, 16384, 512, 64, 64, False, True, True): (4, 8, 2, 4),
        (16384, 16384, 512, 64, 64, True, False, True): (4, 8, 2, 4),
        (16384, 16384, 512, 128, 128, False, True, True): (4, 4, 1, 32),
        (16384, 16384, 512, 128, 128, True, False, True): (4, 4, 1, 32),
        (16384, 16384, 1024, 16, 16, False, True, True): (1, 4, 3, 4),
        (16384, 16384, 1024, 16, 16, True, False, True): (2, 16, 3, 1),
        (16384, 16384, 1024, 32, 32, False, True, True): (1, 16, 3, 4),
        (16384, 16384, 1024, 32, 32, True, False, True): (1, 8, 3, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6799-6834 / 第 6799-6834 行
````python
        (16384, 16384, 1024, 64, 64, False, True, True): (4, 16, 2, 4),
        (16384, 16384, 1024, 64, 64, True, False, True): (4, 16, 2, 4),
        (16384, 16384, 1024, 128, 128, False, True, True): (4, 8, 1, 32),
        (16384, 16384, 1024, 128, 128, True, False, True): (4, 8, 1, 32),
        (16384, 16384, 2048, 16, 16, False, True, True): (1, 8, 3, 4),
        (16384, 16384, 2048, 16, 16, True, False, True): (2, 32, 3, 1),
        (16384, 16384, 2048, 32, 32, False, True, True): (1, 32, 3, 4),
        (16384, 16384, 2048, 32, 32, True, False, True): (1, 16, 3, 4),
        (16384, 16384, 2048, 64, 64, False, True, True): (4, 32, 2, 4),
        (16384, 16384, 2048, 64, 64, True, False, True): (2, 32, 2, 4),
        (16384, 16384, 2048, 128, 128, False, True, True): (4, 16, 1, 32),
        (16384, 16384, 2048, 128, 128, True, False, True): (4, 16, 1, 32),
        (16384, 16384, 4096, 16, 16, False, True, True): (1, 16, 3, 4),
        (16384, 16384, 4096, 16, 16, True, False, True): (2, 64, 3, 1),
        (16384, 16384, 4096, 32, 32, False, True, True): (1, 64, 3, 4),
        (16384, 16384, 4096, 32, 32, True, False, True): (1, 32, 3, 4),
        (16384, 16384, 4096, 64, 64, False, True, True): (4, 64, 2, 4),
        (16384, 16384, 4096, 64, 64, True, False, True): (2, 64, 2, 4),
        (16384, 16384, 4096, 128, 128, False, True, True): (4, 32, 1, 32),
        (16384, 16384, 4096, 128, 128, True, False, True): (4, 32, 1, 32),
        (16384, 16384, 8192, 16, 16, False, True, True): (1, 128, 3, 1),
        (16384, 16384, 8192, 16, 16, True, False, True): (2, 128, 3, 1),
        (16384, 16384, 8192, 32, 32, False, True, True): (1, 128, 3, 4),
        (16384, 16384, 8192, 32, 32, True, False, True): (1, 64, 3, 4),
        (16384, 16384, 8192, 64, 64, False, True, True): (2, 128, 2, 4),
        (16384, 16384, 8192, 64, 64, True, False, True): (2, 128, 2, 4),
        (16384, 16384, 8192, 128, 128, False, True, True): (4, 64, 1, 32),
        (16384, 16384, 8192, 128, 128, True, False, True): (4, 64, 1, 32),
        (16384, 16384, 16384, 16, 16, False, True, True): (1, 64, 3, 4),
        (16384, 16384, 16384, 16, 16, True, False, True): (2, 256, 3, 1),
        (16384, 16384, 16384, 32, 32, False, True, True): (1, 256, 3, 4),
        (16384, 16384, 16384, 32, 32, True, False, True): (1, 128, 3, 4),
        (16384, 16384, 16384, 64, 64, False, True, True): (2, 256, 2, 4),
        (16384, 16384, 16384, 64, 64, True, False, True): (2, 256, 2, 4),
        (16384, 16384, 16384, 128, 128, False, True, True): (4, 128, 1, 32),
        (16384, 16384, 16384, 128, 128, True, False, True): (4, 128, 1, 32),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6835-6870 / 第 6835-6870 行
````python
        (16384, 16384, 32768, 16, 16, False, True, True): (1, 512, 3, 1),
        (16384, 16384, 32768, 16, 16, True, False, True): (1, 128, 3, 4),
        (16384, 16384, 32768, 32, 32, False, True, True): (2, 512, 3, 4),
        (16384, 16384, 32768, 32, 32, True, False, True): (1, 256, 4, 4),
        (16384, 16384, 32768, 64, 64, False, True, True): (2, 512, 2, 4),
        (16384, 16384, 32768, 64, 64, True, False, True): (2, 512, 2, 4),
        (16384, 16384, 32768, 128, 128, False, True, True): (4, 256, 1, 32),
        (16384, 16384, 32768, 128, 128, True, False, True): (4, 256, 1, 32),
        (16384, 16384, 65536, 16, 16, False, True, True): (1, 256, 3, 4),
        (16384, 16384, 65536, 16, 16, True, False, True): (1, 1024, 3, 1),
        (16384, 16384, 65536, 32, 32, False, True, True): (1, 1024, 3, 4),
        (16384, 16384, 65536, 32, 32, True, False, True): (1, 512, 4, 4),
        (16384, 16384, 65536, 64, 64, False, True, True): (2, 1024, 2, 4),
        (16384, 16384, 65536, 64, 64, True, False, True): (2, 1024, 2, 4),
        (16384, 16384, 65536, 128, 128, False, True, True): (4, 512, 1, 32),
        (16384, 16384, 65536, 128, 128, True, False, True): (4, 512, 1, 32),
        (16384, 16384, 131072, 16, 16, False, True, True): (1, 1024, 4, 4),
        (16384, 16384, 131072, 16, 16, True, False, True): (2, 2048, 3, 1),
        (16384, 16384, 131072, 32, 32, False, True, True): (1, 1024, 2, 4),
        (16384, 16384, 131072, 32, 32, True, False, True): (1, 1024, 2, 4),
        (16384, 16384, 131072, 64, 64, False, True, True): (4, 2048, 2, 4),
        (16384, 16384, 131072, 64, 64, True, False, True): (2, 2048, 2, 4),
        (16384, 16384, 131072, 128, 128, False, True, True): (4, 1024, 1, 32),
        (16384, 16384, 131072, 128, 128, True, False, True): (4, 1024, 1, 32),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.float32, 0.56)): {
        (192, 192, 256, 64, 64, False, True, True): (1, 4, 3, 8),
        (192, 192, 256, 64, 64, True, False, True): (1, 4, 3, 8),
        (192, 192, 512, 64, 64, False, True, True): (2, 8, 3, 8),
        (192, 192, 512, 64, 64, True, False, True): (5, 8, 3, 8),
        (192, 192, 1024, 64, 64, False, True, True): (2, 16, 4, 8),
        (192, 192, 1024, 64, 64, True, False, True): (1, 16, 3, 8),
        (192, 192, 2048, 64, 64, False, True, True): (3, 32, 3, 8),
        (192, 192, 2048, 64, 64, True, False, True): (5, 32, 5, 8),
        (192, 192, 4096, 64, 64, False, True, True): (3, 64, 2, 8),
        (192, 192, 4096, 64, 64, True, False, True): (1, 64, 3, 8),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6871-6906 / 第 6871-6906 行
````python
        (192, 192, 8192, 64, 64, False, True, True): (3, 128, 3, 8),
        (192, 192, 8192, 64, 64, True, False, True): (6, 128, 3, 4),
        (192, 192, 16384, 64, 64, False, True, True): (1, 256, 1, 8),
        (192, 192, 16384, 64, 64, True, False, True): (1, 256, 3, 4),
        (192, 192, 32768, 64, 64, False, True, True): (1, 512, 1, 8),
        (192, 192, 32768, 64, 64, True, False, True): (1, 512, 3, 4),
        (192, 192, 65536, 64, 64, False, True, True): (1, 1024, 1, 8),
        (192, 192, 65536, 64, 64, True, False, True): (1, 1024, 3, 4),
        (192, 192, 131072, 64, 64, False, True, True): (1, 2048, 1, 8),
        (192, 192, 131072, 64, 64, True, False, True): (3, 2048, 1, 4),
        (384, 384, 256, 128, 128, False, True, True): (1, 2, 1, 32),
        (384, 384, 256, 128, 128, True, False, True): (1, 2, 1, 32),
        (384, 384, 512, 128, 128, False, True, True): (1, 4, 1, 32),
        (384, 384, 512, 128, 128, True, False, True): (2, 4, 1, 32),
        (384, 384, 1024, 128, 128, False, True, True): (1, 8, 1, 32),
        (384, 384, 1024, 128, 128, True, False, True): (4, 8, 1, 32),
        (384, 384, 2048, 128, 128, False, True, True): (1, 16, 1, 32),
        (384, 384, 2048, 128, 128, True, False, True): (1, 16, 1, 32),
        (384, 384, 4096, 128, 128, False, True, True): (1, 32, 1, 32),
        (384, 384, 4096, 128, 128, True, False, True): (2, 32, 2, 32),
        (384, 384, 8192, 128, 128, False, True, True): (1, 64, 1, 32),
        (384, 384, 8192, 128, 128, True, False, True): (1, 64, 2, 32),
        (384, 384, 16384, 128, 128, False, True, True): (1, 128, 1, 32),
        (384, 384, 16384, 128, 128, True, False, True): (4, 128, 1, 32),
        (384, 384, 32768, 128, 128, False, True, True): (3, 256, 1, 32),
        (384, 384, 32768, 128, 128, True, False, True): (3, 256, 1, 32),
        (384, 384, 65536, 128, 128, False, True, True): (3, 512, 1, 32),
        (384, 384, 65536, 128, 128, True, False, True): (3, 512, 1, 32),
        (384, 384, 131072, 128, 128, False, True, True): (1, 1024, 1, 32),
        (384, 384, 131072, 128, 128, True, False, True): (3, 1024, 1, 32),
    },
    ("bsr_dense_addmm", "NVIDIA A100-SXM4-80GB", (0, torch.int8, 0.5)): {
        (1280, 5120, 65792, 32, 32, False, True, True): (1, 1028, 1, 8),
        (1280, 5120, 65792, 32, 32, True, False, True): (1, 514, 3, 2),
        (1280, 5120, 65792, 64, 64, False, True, True): (2, 514, 1, 4),
        (1280, 5120, 65792, 64, 64, True, False, True): (1, 514, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6907-6942 / 第 6907-6942 行
````python
        (1280, 5120, 65792, 128, 128, False, True, True): (2, 514, 1, 8),
        (1280, 5120, 65792, 128, 128, True, False, True): (1, 514, 2, 4),
        (1280, 5120, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (1280, 5120, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
        (5120, 1280, 65792, 32, 32, False, True, True): (3, 1028, 1, 8),
        (5120, 1280, 65792, 32, 32, True, False, True): (1, 514, 1, 2),
        (5120, 1280, 65792, 64, 64, False, True, True): (1, 514, 1, 4),
        (5120, 1280, 65792, 64, 64, True, False, True): (2, 514, 2, 2),
        (5120, 1280, 65792, 128, 128, False, True, True): (2, 514, 1, 8),
        (5120, 1280, 65792, 128, 128, True, False, True): (2, 514, 2, 4),
        (5120, 1280, 65792, 256, 256, False, True, True): (1, 257, 1, 32),
        (5120, 1280, 65792, 256, 256, True, False, True): (1, 257, 1, 32),
    },
    ("scatter_mm", "NVIDIA A100-SXM4-80GB", (0, torch.bfloat16, 0.5)): {
        (256, 256, 256, 16, 16): (1, 1, 16, 16, 1, 2),
        (256, 256, 256, 32, 32): (1, 1, 16, 16, 1, 4),
        (256, 256, 256, 64, 64): (1, 1, 16, 16, 1, 1),
        (256, 256, 256, 128, 128): (2, 4, 16, 64, 1, 4),
        (256, 256, 512, 16, 16): (1, 1, 16, 16, 1, 4),
        (256, 256, 512, 32, 32): (1, 1, 16, 32, 1, 4),
        (256, 256, 512, 64, 64): (1, 1, 16, 32, 1, 1),
        (256, 256, 512, 128, 128): (1, 1, 32, 32, 1, 4),
        (256, 256, 1024, 16, 16): (1, 1, 16, 16, 1, 4),
        (256, 256, 1024, 32, 32): (1, 2, 16, 32, 1, 1),
        (256, 256, 1024, 64, 64): (1, 1, 32, 32, 1, 2),
        (256, 256, 1024, 128, 128): (1, 1, 32, 64, 1, 4),
        (256, 256, 2048, 16, 16): (1, 1, 16, 64, 1, 8),
        (256, 256, 2048, 32, 32): (2, 1, 32, 64, 1, 2),
        (256, 256, 2048, 64, 64): (1, 1, 32, 32, 1, 1),
        (256, 256, 2048, 128, 128): (1, 1, 64, 64, 1, 4),
        (256, 256, 4096, 16, 16): (1, 1, 16, 64, 1, 1),
        (256, 256, 4096, 32, 32): (2, 2, 32, 64, 1, 2),
        (256, 256, 4096, 64, 64): (1, 1, 32, 128, 1, 4),
        (256, 256, 4096, 128, 128): (1, 1, 64, 64, 1, 4),
        (256, 256, 8192, 16, 16): (1, 2, 16, 64, 1, 2),
        (256, 256, 8192, 32, 32): (1, 1, 32, 64, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6943-6978 / 第 6943-6978 行
````python
        (256, 256, 8192, 64, 64): (1, 1, 32, 64, 1, 2),
        (256, 256, 8192, 128, 128): (1, 1, 64, 64, 1, 4),
        (256, 256, 16384, 16, 16): (1, 1, 16, 64, 1, 2),
        (256, 256, 16384, 32, 32): (1, 1, 32, 64, 1, 2),
        (256, 256, 16384, 64, 64): (1, 1, 64, 64, 1, 2),
        (256, 256, 16384, 128, 128): (2, 16, 64, 64, 1, 4),
        (256, 256, 32768, 16, 16): (1, 1, 16, 128, 1, 2),
        (256, 256, 32768, 32, 32): (1, 1, 32, 64, 1, 2),
        (256, 256, 32768, 64, 64): (1, 1, 64, 64, 1, 2),
        (256, 256, 32768, 128, 128): (2, 32, 64, 64, 1, 4),
        (256, 256, 65536, 16, 16): (1, 1, 16, 64, 1, 1),
        (256, 256, 65536, 32, 32): (1, 1, 32, 64, 1, 2),
        (256, 256, 65536, 64, 64): (1, 1, 64, 32, 1, 1),
        (256, 256, 65536, 128, 128): (2, 32, 64, 64, 1, 4),
        (256, 256, 131072, 16, 16): (1, 1, 16, 64, 1, 1),
        (256, 256, 131072, 32, 32): (1, 1, 32, 64, 1, 2),
        (256, 256, 131072, 64, 64): (4, 1, 64, 32, 1, 1),
        (256, 256, 131072, 128, 128): (2, 64, 64, 64, 1, 4),
        (512, 512, 256, 16, 16): (1, 1, 16, 16, 1, 2),
        (512, 512, 256, 32, 32): (1, 1, 16, 32, 1, 1),
        (512, 512, 256, 64, 64): (1, 2, 16, 32, 1, 1),
        (512, 512, 256, 128, 128): (2, 16, 64, 16, 2, 4),
        (512, 512, 512, 16, 16): (1, 1, 16, 16, 1, 4),
        (512, 512, 512, 32, 32): (1, 1, 16, 32, 1, 1),
        (512, 512, 512, 64, 64): (1, 1, 32, 32, 1, 2),
        (512, 512, 512, 128, 128): (2, 8, 32, 64, 1, 4),
        (512, 512, 1024, 16, 16): (1, 1, 16, 64, 1, 8),
        (512, 512, 1024, 32, 32): (1, 1, 32, 32, 3, 1),
        (512, 512, 1024, 64, 64): (1, 4, 32, 64, 1, 2),
        (512, 512, 1024, 128, 128): (1, 4, 64, 64, 1, 4),
        (512, 512, 2048, 16, 16): (1, 1, 16, 64, 1, 2),
        (512, 512, 2048, 32, 32): (1, 1, 32, 64, 1, 2),
        (512, 512, 2048, 64, 64): (1, 1, 64, 64, 3, 4),
        (512, 512, 2048, 128, 128): (1, 1, 64, 64, 1, 4),
        (512, 512, 4096, 16, 16): (1, 1, 16, 64, 1, 2),
        (512, 512, 4096, 32, 32): (2, 64, 32, 64, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 6979-7014 / 第 6979-7014 行
````python
        (512, 512, 4096, 64, 64): (1, 1, 64, 64, 3, 4),
        (512, 512, 4096, 128, 128): (1, 1, 64, 64, 1, 4),
        (512, 512, 8192, 16, 16): (1, 2, 16, 128, 1, 2),
        (512, 512, 8192, 32, 32): (1, 1, 32, 64, 1, 2),
        (512, 512, 8192, 64, 64): (1, 1, 64, 64, 1, 2),
        (512, 512, 8192, 128, 128): (1, 1, 64, 64, 1, 4),
        (512, 512, 16384, 16, 16): (1, 2, 16, 128, 1, 2),
        (512, 512, 16384, 32, 32): (1, 1, 32, 64, 1, 2),
        (512, 512, 16384, 64, 64): (1, 1, 64, 64, 3, 2),
        (512, 512, 16384, 128, 128): (2, 1, 64, 64, 1, 4),
        (512, 512, 32768, 16, 16): (1, 2, 16, 128, 1, 2),
        (512, 512, 32768, 32, 32): (1, 1, 32, 64, 1, 2),
        (512, 512, 32768, 64, 64): (1, 1, 64, 64, 3, 4),
        (512, 512, 32768, 128, 128): (2, 1, 64, 64, 1, 4),
        (512, 512, 65536, 16, 16): (1, 2, 16, 128, 1, 2),
        (512, 512, 65536, 32, 32): (1, 1, 32, 64, 1, 2),
        (512, 512, 65536, 64, 64): (1, 1, 64, 64, 3, 4),
        (512, 512, 65536, 128, 128): (2, 1, 64, 64, 1, 4),
        (512, 512, 131072, 16, 16): (1, 1, 16, 64, 1, 1),
        (512, 512, 131072, 32, 32): (1, 1, 32, 64, 1, 2),
        (512, 512, 131072, 64, 64): (1, 1, 64, 64, 3, 4),
        (512, 512, 131072, 128, 128): (2, 4, 64, 64, 1, 4),
        (1024, 1024, 256, 16, 16): (1, 1, 16, 16, 1, 4),
        (1024, 1024, 256, 32, 32): (2, 16, 32, 16, 3, 4),
        (1024, 1024, 256, 64, 64): (1, 4, 32, 32, 1, 2),
        (1024, 1024, 256, 128, 128): (1, 4, 128, 16, 3, 16),
        (1024, 1024, 512, 16, 16): (1, 1, 16, 64, 1, 2),
        (1024, 1024, 512, 32, 32): (2, 2, 32, 64, 1, 2),
        (1024, 1024, 512, 64, 64): (2, 8, 64, 64, 3, 4),
        (1024, 1024, 512, 128, 128): (1, 4, 64, 64, 1, 8),
        (1024, 1024, 1024, 16, 16): (1, 1, 16, 64, 1, 2),
        (1024, 1024, 1024, 32, 32): (1, 1, 32, 64, 1, 2),
        (1024, 1024, 1024, 64, 64): (1, 8, 64, 64, 3, 4),
        (1024, 1024, 1024, 128, 128): (1, 8, 64, 64, 1, 4),
        (1024, 1024, 2048, 16, 16): (1, 2, 16, 64, 1, 2),
        (1024, 1024, 2048, 32, 32): (1, 1, 32, 64, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7015-7050 / 第 7015-7050 行
````python
        (1024, 1024, 2048, 64, 64): (2, 16, 64, 64, 2, 2),
        (1024, 1024, 2048, 128, 128): (2, 32, 64, 64, 1, 4),
        (1024, 1024, 4096, 16, 16): (2, 16, 16, 128, 1, 2),
        (1024, 1024, 4096, 32, 32): (1, 16, 32, 64, 3, 2),
        (1024, 1024, 4096, 64, 64): (1, 1, 64, 64, 3, 4),
        (1024, 1024, 4096, 128, 128): (2, 64, 128, 64, 1, 4),
        (1024, 1024, 8192, 16, 16): (2, 16, 16, 128, 1, 2),
        (1024, 1024, 8192, 32, 32): (1, 16, 32, 64, 3, 2),
        (1024, 1024, 8192, 64, 64): (1, 1, 64, 64, 3, 4),
        (1024, 1024, 8192, 128, 128): (2, 1, 64, 64, 1, 4),
        (1024, 1024, 16384, 16, 16): (1, 2, 16, 128, 1, 2),
        (1024, 1024, 16384, 32, 32): (1, 16, 32, 64, 3, 2),
        (1024, 1024, 16384, 64, 64): (1, 1, 64, 64, 3, 4),
        (1024, 1024, 16384, 128, 128): (2, 16, 128, 64, 1, 4),
        (1024, 1024, 32768, 16, 16): (1, 1, 16, 128, 1, 2),
        (1024, 1024, 32768, 32, 32): (1, 1, 32, 128, 1, 2),
        (1024, 1024, 32768, 64, 64): (1, 32, 64, 32, 2, 1),
        (1024, 1024, 32768, 128, 128): (2, 8, 128, 64, 1, 4),
        (1024, 1024, 65536, 16, 16): (3, 2, 16, 128, 1, 2),
        (1024, 1024, 65536, 32, 32): (1, 1, 32, 128, 1, 2),
        (1024, 1024, 65536, 64, 64): (2, 4, 64, 32, 2, 1),
        (1024, 1024, 65536, 128, 128): (2, 8, 128, 64, 1, 4),
        (1024, 1024, 131072, 16, 16): (2, 1, 16, 128, 1, 2),
        (1024, 1024, 131072, 32, 32): (1, 1, 32, 128, 1, 2),
        (1024, 1024, 131072, 64, 64): (1, 4, 64, 32, 2, 1),
        (1024, 1024, 131072, 128, 128): (4, 1, 128, 64, 1, 4),
        (2048, 2048, 256, 16, 16): (1, 1, 16, 64, 1, 8),
        (2048, 2048, 256, 32, 32): (1, 1, 32, 32, 3, 1),
        (2048, 2048, 256, 64, 64): (1, 1, 32, 32, 2, 1),
        (2048, 2048, 256, 128, 128): (1, 4, 64, 64, 1, 8),
        (2048, 2048, 512, 16, 16): (1, 2, 16, 64, 1, 2),
        (2048, 2048, 512, 32, 32): (1, 2, 32, 64, 1, 4),
        (2048, 2048, 512, 64, 64): (1, 4, 64, 64, 1, 8),
        (2048, 2048, 512, 128, 128): (1, 4, 64, 64, 1, 4),
        (2048, 2048, 1024, 16, 16): (1, 2, 16, 128, 1, 2),
        (2048, 2048, 1024, 32, 32): (1, 1, 32, 64, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7051-7086 / 第 7051-7086 行
````python
        (2048, 2048, 1024, 64, 64): (1, 8, 64, 64, 1, 4),
        (2048, 2048, 1024, 128, 128): (1, 8, 128, 64, 1, 4),
        (2048, 2048, 2048, 16, 16): (3, 4, 16, 128, 1, 2),
        (2048, 2048, 2048, 32, 32): (1, 16, 32, 64, 5, 2),
        (2048, 2048, 2048, 64, 64): (1, 1, 64, 64, 3, 4),
        (2048, 2048, 2048, 128, 128): (1, 8, 128, 64, 1, 4),
        (2048, 2048, 4096, 16, 16): (1, 2, 16, 128, 1, 2),
        (2048, 2048, 4096, 32, 32): (1, 8, 32, 64, 3, 2),
        (2048, 2048, 4096, 64, 64): (1, 1, 64, 64, 3, 4),
        (2048, 2048, 4096, 128, 128): (1, 8, 128, 64, 1, 4),
        (2048, 2048, 8192, 16, 16): (2, 4, 16, 128, 1, 2),
        (2048, 2048, 8192, 32, 32): (1, 4, 32, 128, 3, 2),
        (2048, 2048, 8192, 64, 64): (1, 8, 64, 64, 3, 2),
        (2048, 2048, 8192, 128, 128): (1, 8, 128, 64, 1, 4),
        (2048, 2048, 16384, 16, 16): (1, 2, 16, 128, 1, 2),
        (2048, 2048, 16384, 32, 32): (1, 4, 32, 128, 3, 2),
        (2048, 2048, 16384, 64, 64): (1, 8, 64, 64, 3, 2),
        (2048, 2048, 16384, 128, 128): (1, 4, 128, 64, 1, 4),
        (2048, 2048, 32768, 16, 16): (3, 2, 16, 128, 1, 2),
        (2048, 2048, 32768, 32, 32): (1, 1, 32, 128, 3, 2),
        (2048, 2048, 32768, 64, 64): (1, 1, 64, 64, 3, 2),
        (2048, 2048, 32768, 128, 128): (1, 4, 128, 64, 1, 4),
        (2048, 2048, 65536, 16, 16): (1, 2, 16, 128, 1, 2),
        (2048, 2048, 65536, 32, 32): (1, 4, 32, 128, 1, 2),
        (2048, 2048, 65536, 64, 64): (1, 1, 64, 64, 3, 2),
        (2048, 2048, 65536, 128, 128): (1, 2, 128, 64, 1, 4),
        (2048, 2048, 131072, 16, 16): (4, 2, 16, 128, 1, 2),
        (2048, 2048, 131072, 32, 32): (1, 1, 32, 128, 3, 2),
        (2048, 2048, 131072, 64, 64): (1, 1, 64, 64, 3, 2),
        (2048, 2048, 131072, 128, 128): (1, 2, 128, 64, 1, 4),
        (4096, 4096, 256, 16, 16): (1, 1, 16, 64, 1, 2),
        (4096, 4096, 256, 32, 32): (1, 1, 32, 64, 3, 4),
        (4096, 4096, 256, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 256, 128, 128): (3, 4, 128, 32, 1, 4),
        (4096, 4096, 512, 16, 16): (1, 2, 16, 128, 1, 2),
        (4096, 4096, 512, 32, 32): (1, 2, 32, 64, 3, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7087-7122 / 第 7087-7122 行
````python
        (4096, 4096, 512, 64, 64): (1, 4, 64, 64, 1, 4),
        (4096, 4096, 512, 128, 128): (1, 4, 128, 64, 1, 4),
        (4096, 4096, 1024, 16, 16): (1, 2, 16, 128, 1, 2),
        (4096, 4096, 1024, 32, 32): (1, 8, 32, 64, 3, 2),
        (4096, 4096, 1024, 64, 64): (1, 4, 64, 64, 1, 4),
        (4096, 4096, 1024, 128, 128): (2, 4, 128, 64, 1, 4),
        (4096, 4096, 2048, 16, 16): (1, 1, 16, 128, 1, 2),
        (4096, 4096, 2048, 32, 32): (1, 4, 32, 128, 1, 4),
        (4096, 4096, 2048, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 2048, 128, 128): (1, 16, 128, 64, 1, 4),
        (4096, 4096, 4096, 16, 16): (1, 1, 16, 64, 3, 1),
        (4096, 4096, 4096, 32, 32): (1, 4, 32, 64, 3, 2),
        (4096, 4096, 4096, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 4096, 128, 128): (5, 1, 128, 64, 1, 4),
        (4096, 4096, 8192, 16, 16): (1, 1, 16, 128, 1, 2),
        (4096, 4096, 8192, 32, 32): (1, 1, 32, 128, 3, 2),
        (4096, 4096, 8192, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 8192, 128, 128): (2, 1, 128, 64, 1, 4),
        (4096, 4096, 16384, 16, 16): (1, 1, 16, 128, 1, 2),
        (4096, 4096, 16384, 32, 32): (1, 1, 32, 128, 3, 2),
        (4096, 4096, 16384, 64, 64): (1, 1, 64, 64, 4, 4),
        (4096, 4096, 16384, 128, 128): (2, 1, 128, 64, 1, 4),
        (4096, 4096, 32768, 16, 16): (3, 1, 16, 128, 1, 2),
        (4096, 4096, 32768, 32, 32): (1, 1, 32, 128, 3, 2),
        (4096, 4096, 32768, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 32768, 128, 128): (2, 1, 128, 64, 1, 4),
        (4096, 4096, 65536, 16, 16): (2, 2, 16, 128, 1, 2),
        (4096, 4096, 65536, 32, 32): (1, 1, 32, 128, 4, 2),
        (4096, 4096, 65536, 64, 64): (1, 1, 64, 64, 4, 4),
        (4096, 4096, 65536, 128, 128): (2, 1, 128, 64, 1, 4),
        (4096, 4096, 131072, 16, 16): (2, 1, 16, 128, 1, 2),
        (4096, 4096, 131072, 32, 32): (1, 1, 32, 128, 3, 2),
        (4096, 4096, 131072, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 131072, 128, 128): (2, 1, 128, 64, 1, 4),
        (8192, 8192, 256, 16, 16): (1, 2, 16, 64, 1, 2),
        (8192, 8192, 256, 32, 32): (1, 1, 32, 64, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7123-7158 / 第 7123-7158 行
````python
        (8192, 8192, 256, 64, 64): (1, 2, 64, 64, 1, 4),
        (8192, 8192, 256, 128, 128): (3, 16, 128, 16, 1, 2),
        (8192, 8192, 512, 16, 16): (1, 2, 16, 128, 1, 2),
        (8192, 8192, 512, 32, 32): (1, 4, 32, 64, 3, 2),
        (8192, 8192, 512, 64, 64): (2, 8, 64, 64, 4, 4),
        (8192, 8192, 512, 128, 128): (1, 8, 128, 64, 1, 4),
        (8192, 8192, 1024, 16, 16): (4, 2, 16, 128, 1, 2),
        (8192, 8192, 1024, 32, 32): (1, 8, 32, 128, 1, 2),
        (8192, 8192, 1024, 64, 64): (1, 16, 64, 64, 3, 2),
        (8192, 8192, 1024, 128, 128): (2, 16, 128, 64, 2, 4),
        (8192, 8192, 2048, 16, 16): (2, 1, 16, 64, 4, 1),
        (8192, 8192, 2048, 32, 32): (1, 16, 32, 64, 5, 2),
        (8192, 8192, 2048, 64, 64): (1, 16, 64, 64, 3, 2),
        (8192, 8192, 2048, 128, 128): (2, 16, 128, 64, 2, 4),
        (8192, 8192, 4096, 16, 16): (1, 1, 16, 64, 4, 1),
        (8192, 8192, 4096, 32, 32): (1, 16, 32, 64, 5, 2),
        (8192, 8192, 4096, 64, 64): (1, 16, 64, 64, 3, 2),
        (8192, 8192, 4096, 128, 128): (2, 64, 128, 64, 2, 4),
        (8192, 8192, 8192, 16, 16): (1, 1, 16, 64, 4, 1),
        (8192, 8192, 8192, 32, 32): (1, 8, 32, 128, 5, 4),
        (8192, 8192, 8192, 64, 64): (1, 8, 64, 64, 3, 2),
        (8192, 8192, 8192, 128, 128): (2, 8, 128, 64, 1, 4),
        (8192, 8192, 16384, 16, 16): (1, 1, 16, 64, 4, 1),
        (8192, 8192, 16384, 32, 32): (1, 8, 32, 64, 5, 2),
        (8192, 8192, 16384, 64, 64): (1, 8, 64, 64, 3, 2),
        (8192, 8192, 16384, 128, 128): (1, 8, 128, 64, 1, 4),
        (8192, 8192, 32768, 16, 16): (1, 1, 16, 64, 4, 1),
        (8192, 8192, 32768, 32, 32): (1, 8, 32, 64, 5, 2),
        (8192, 8192, 32768, 64, 64): (3, 8, 64, 64, 3, 2),
        (8192, 8192, 32768, 128, 128): (2, 8, 128, 64, 1, 4),
        (8192, 8192, 65536, 16, 16): (1, 1, 16, 64, 4, 1),
        (8192, 8192, 65536, 32, 32): (5, 4, 32, 64, 3, 2),
        (8192, 8192, 65536, 64, 64): (1, 8, 64, 64, 3, 2),
        (8192, 8192, 65536, 128, 128): (2, 8, 128, 64, 1, 4),
        (8192, 8192, 131072, 16, 16): (2, 1, 16, 64, 4, 1),
        (8192, 8192, 131072, 32, 32): (1, 4, 32, 64, 5, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7159-7194 / 第 7159-7194 行
````python
        (8192, 8192, 131072, 64, 64): (1, 4, 64, 128, 3, 4),
        (8192, 8192, 131072, 128, 128): (2, 8, 128, 64, 1, 4),
        (16384, 16384, 256, 16, 16): (1, 2, 16, 128, 1, 2),
        (16384, 16384, 256, 32, 32): (1, 4, 32, 64, 3, 2),
        (16384, 16384, 256, 64, 64): (2, 4, 64, 64, 4, 4),
        (16384, 16384, 256, 128, 128): (1, 4, 128, 64, 1, 16),
        (16384, 16384, 512, 16, 16): (1, 2, 16, 128, 3, 2),
        (16384, 16384, 512, 32, 32): (1, 4, 32, 128, 5, 4),
        (16384, 16384, 512, 64, 64): (1, 8, 64, 64, 3, 2),
        (16384, 16384, 512, 128, 128): (2, 8, 128, 64, 1, 4),
        (16384, 16384, 1024, 16, 16): (1, 2, 16, 128, 1, 2),
        (16384, 16384, 1024, 32, 32): (1, 8, 32, 64, 5, 2),
        (16384, 16384, 1024, 64, 64): (1, 16, 64, 64, 3, 2),
        (16384, 16384, 1024, 128, 128): (5, 16, 128, 64, 2, 4),
        (16384, 16384, 2048, 16, 16): (1, 2, 16, 128, 1, 2),
        (16384, 16384, 2048, 32, 32): (1, 8, 32, 64, 5, 2),
        (16384, 16384, 2048, 64, 64): (1, 16, 64, 64, 3, 2),
        (16384, 16384, 2048, 128, 128): (4, 32, 128, 64, 2, 4),
        (16384, 16384, 4096, 16, 16): (3, 2, 16, 128, 1, 2),
        (16384, 16384, 4096, 32, 32): (1, 4, 32, 64, 5, 2),
        (16384, 16384, 4096, 64, 64): (2, 16, 64, 64, 3, 2),
        (16384, 16384, 4096, 128, 128): (3, 32, 128, 64, 2, 4),
        (16384, 16384, 8192, 16, 16): (1, 2, 16, 128, 1, 2),
        (16384, 16384, 8192, 32, 32): (1, 4, 32, 64, 5, 2),
        (16384, 16384, 8192, 64, 64): (4, 8, 64, 64, 3, 2),
        (16384, 16384, 8192, 128, 128): (5, 8, 128, 64, 1, 4),
        (16384, 16384, 16384, 16, 16): (1, 2, 16, 128, 1, 2),
        (16384, 16384, 16384, 32, 32): (1, 4, 32, 64, 5, 2),
        (16384, 16384, 16384, 64, 64): (2, 4, 64, 128, 3, 4),
        (16384, 16384, 16384, 128, 128): (4, 8, 128, 64, 1, 4),
        (16384, 16384, 32768, 16, 16): (4, 2, 16, 128, 1, 2),
        (16384, 16384, 32768, 32, 32): (1, 4, 32, 64, 5, 2),
        (16384, 16384, 32768, 64, 64): (1, 8, 64, 64, 3, 2),
        (16384, 16384, 32768, 128, 128): (2, 512, 128, 64, 2, 4),
        (16384, 16384, 65536, 16, 16): (3, 2, 16, 128, 1, 2),
        (16384, 16384, 65536, 32, 32): (1, 4, 32, 64, 5, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7195-7230 / 第 7195-7230 行
````python
        (16384, 16384, 65536, 64, 64): (1, 4, 64, 128, 3, 4),
        (16384, 16384, 65536, 128, 128): (2, 1024, 128, 64, 2, 4),
        (16384, 16384, 131072, 16, 16): (1, 2, 16, 128, 1, 2),
        (16384, 16384, 131072, 32, 32): (1, 4, 32, 64, 5, 2),
        (16384, 16384, 131072, 64, 64): (3, 4, 64, 128, 3, 4),
        (16384, 16384, 131072, 128, 128): (4, 2048, 128, 64, 2, 4),
    },
    ("scatter_mm", "NVIDIA A100-SXM4-80GB", (0, torch.float16, 0.5)): {
        (256, 256, 256, 16, 16): (5, 4, 16, 16, 1, 4),
        (256, 256, 256, 32, 32): (5, 2, 32, 16, 1, 4),
        (256, 256, 256, 64, 64): (4, 1, 32, 32, 1, 8),
        (256, 256, 256, 128, 128): (2, 1, 32, 32, 1, 4),
        (256, 256, 512, 16, 16): (2, 2, 16, 32, 1, 4),
        (256, 256, 512, 32, 32): (4, 8, 32, 32, 1, 8),
        (256, 256, 512, 64, 64): (4, 8, 32, 64, 1, 4),
        (256, 256, 512, 128, 128): (4, 8, 32, 64, 1, 4),
        (256, 256, 1024, 16, 16): (4, 2, 16, 64, 1, 2),
        (256, 256, 1024, 32, 32): (4, 16, 32, 64, 1, 2),
        (256, 256, 1024, 64, 64): (4, 16, 32, 64, 1, 4),
        (256, 256, 1024, 128, 128): (4, 16, 64, 64, 1, 8),
        (256, 256, 2048, 16, 16): (2, 16, 16, 64, 1, 8),
        (256, 256, 2048, 32, 32): (4, 16, 32, 64, 1, 2),
        (256, 256, 2048, 64, 64): (4, 16, 32, 64, 1, 4),
        (256, 256, 2048, 128, 128): (4, 16, 64, 64, 1, 4),
        (256, 256, 4096, 16, 16): (4, 32, 16, 64, 1, 1),
        (256, 256, 4096, 32, 32): (2, 64, 32, 64, 1, 2),
        (256, 256, 4096, 64, 64): (4, 64, 64, 64, 1, 4),
        (256, 256, 4096, 128, 128): (4, 32, 64, 64, 1, 4),
        (256, 256, 8192, 16, 16): (4, 64, 16, 64, 1, 1),
        (256, 256, 8192, 32, 32): (4, 128, 32, 64, 1, 2),
        (256, 256, 8192, 64, 64): (4, 64, 64, 64, 1, 4),
        (256, 256, 8192, 128, 128): (4, 64, 64, 64, 1, 4),
        (256, 256, 16384, 16, 16): (4, 128, 16, 64, 1, 1),
        (256, 256, 16384, 32, 32): (2, 128, 32, 64, 1, 2),
        (256, 256, 16384, 64, 64): (4, 32, 32, 128, 1, 4),
        (256, 256, 16384, 128, 128): (4, 16, 64, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7231-7266 / 第 7231-7266 行
````python
        (256, 256, 32768, 16, 16): (4, 64, 16, 64, 1, 1),
        (256, 256, 32768, 32, 32): (2, 256, 32, 64, 1, 2),
        (256, 256, 32768, 64, 64): (4, 32, 32, 128, 1, 4),
        (256, 256, 32768, 128, 128): (4, 32, 64, 64, 1, 4),
        (256, 256, 65536, 16, 16): (4, 128, 16, 64, 1, 1),
        (256, 256, 65536, 32, 32): (4, 1, 32, 64, 1, 2),
        (256, 256, 65536, 64, 64): (2, 1, 64, 64, 1, 2),
        (256, 256, 65536, 128, 128): (4, 32, 64, 64, 1, 4),
        (256, 256, 131072, 16, 16): (4, 64, 16, 64, 1, 1),
        (256, 256, 131072, 32, 32): (2, 1, 32, 64, 1, 2),
        (256, 256, 131072, 64, 64): (4, 32, 32, 128, 1, 4),
        (256, 256, 131072, 128, 128): (4, 32, 64, 64, 1, 4),
        (512, 512, 256, 16, 16): (4, 16, 16, 16, 1, 4),
        (512, 512, 256, 32, 32): (2, 4, 32, 16, 1, 4),
        (512, 512, 256, 64, 64): (2, 16, 64, 16, 3, 8),
        (512, 512, 256, 128, 128): (4, 16, 64, 16, 1, 4),
        (512, 512, 512, 16, 16): (1, 1, 16, 64, 1, 8),
        (512, 512, 512, 32, 32): (2, 4, 16, 32, 1, 1),
        (512, 512, 512, 64, 64): (2, 1, 32, 32, 1, 2),
        (512, 512, 512, 128, 128): (4, 8, 32, 64, 1, 4),
        (512, 512, 1024, 16, 16): (2, 8, 16, 64, 1, 8),
        (512, 512, 1024, 32, 32): (4, 16, 32, 64, 1, 2),
        (512, 512, 1024, 64, 64): (4, 16, 64, 64, 1, 4),
        (512, 512, 1024, 128, 128): (2, 8, 64, 64, 1, 4),
        (512, 512, 2048, 16, 16): (4, 16, 16, 64, 1, 4),
        (512, 512, 2048, 32, 32): (4, 16, 32, 64, 1, 2),
        (512, 512, 2048, 64, 64): (4, 16, 64, 64, 1, 8),
        (512, 512, 2048, 128, 128): (4, 16, 64, 64, 1, 4),
        (512, 512, 4096, 16, 16): (4, 32, 16, 128, 1, 2),
        (512, 512, 4096, 32, 32): (4, 32, 32, 64, 1, 2),
        (512, 512, 4096, 64, 64): (4, 32, 64, 64, 1, 4),
        (512, 512, 4096, 128, 128): (4, 32, 64, 64, 1, 4),
        (512, 512, 8192, 16, 16): (2, 32, 16, 128, 1, 2),
        (512, 512, 8192, 32, 32): (4, 64, 32, 64, 1, 2),
        (512, 512, 8192, 64, 64): (4, 128, 64, 64, 1, 2),
        (512, 512, 8192, 128, 128): (4, 64, 64, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7267-7302 / 第 7267-7302 行
````python
        (512, 512, 16384, 16, 16): (4, 32, 16, 64, 1, 1),
        (512, 512, 16384, 32, 32): (4, 64, 32, 64, 1, 2),
        (512, 512, 16384, 64, 64): (4, 16, 64, 64, 1, 4),
        (512, 512, 16384, 128, 128): (4, 32, 64, 64, 1, 4),
        (512, 512, 32768, 16, 16): (7, 16, 16, 128, 1, 2),
        (512, 512, 32768, 32, 32): (4, 64, 32, 64, 1, 2),
        (512, 512, 32768, 64, 64): (2, 32, 64, 64, 3, 2),
        (512, 512, 32768, 128, 128): (2, 32, 64, 64, 1, 4),
        (512, 512, 65536, 16, 16): (2, 32, 16, 64, 1, 1),
        (512, 512, 65536, 32, 32): (4, 64, 32, 64, 1, 2),
        (512, 512, 65536, 64, 64): (3, 32, 64, 64, 3, 2),
        (512, 512, 65536, 128, 128): (4, 16, 64, 64, 1, 4),
        (512, 512, 131072, 16, 16): (3, 32, 16, 128, 1, 2),
        (512, 512, 131072, 32, 32): (4, 64, 32, 64, 1, 2),
        (512, 512, 131072, 64, 64): (2, 32, 64, 64, 3, 2),
        (512, 512, 131072, 128, 128): (3, 1, 64, 64, 1, 4),
        (1024, 1024, 256, 16, 16): (4, 16, 16, 16, 1, 4),
        (1024, 1024, 256, 32, 32): (4, 16, 32, 16, 1, 4),
        (1024, 1024, 256, 64, 64): (4, 4, 64, 32, 1, 16),
        (1024, 1024, 256, 128, 128): (4, 16, 64, 16, 1, 8),
        (1024, 1024, 512, 16, 16): (2, 8, 16, 64, 1, 8),
        (1024, 1024, 512, 32, 32): (3, 2, 32, 64, 1, 2),
        (1024, 1024, 512, 64, 64): (4, 8, 32, 64, 1, 8),
        (1024, 1024, 512, 128, 128): (4, 8, 64, 64, 1, 8),
        (1024, 1024, 1024, 16, 16): (2, 2, 16, 64, 1, 2),
        (1024, 1024, 1024, 32, 32): (2, 8, 32, 64, 1, 2),
        (1024, 1024, 1024, 64, 64): (2, 8, 32, 128, 1, 4),
        (1024, 1024, 1024, 128, 128): (2, 8, 64, 64, 1, 4),
        (1024, 1024, 2048, 16, 16): (2, 16, 16, 128, 3, 2),
        (1024, 1024, 2048, 32, 32): (4, 32, 32, 64, 1, 2),
        (1024, 1024, 2048, 64, 64): (4, 16, 64, 64, 1, 4),
        (1024, 1024, 2048, 128, 128): (4, 32, 64, 64, 1, 4),
        (1024, 1024, 4096, 16, 16): (4, 16, 16, 128, 1, 2),
        (1024, 1024, 4096, 32, 32): (3, 32, 32, 64, 1, 2),
        (1024, 1024, 4096, 64, 64): (4, 32, 64, 64, 1, 4),
        (1024, 1024, 4096, 128, 128): (4, 32, 64, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7303-7338 / 第 7303-7338 行
````python
        (1024, 1024, 8192, 16, 16): (5, 16, 16, 128, 1, 2),
        (1024, 1024, 8192, 32, 32): (2, 32, 32, 64, 3, 2),
        (1024, 1024, 8192, 64, 64): (1, 16, 64, 64, 3, 2),
        (1024, 1024, 8192, 128, 128): (4, 32, 64, 64, 1, 4),
        (1024, 1024, 16384, 16, 16): (4, 16, 16, 128, 1, 2),
        (1024, 1024, 16384, 32, 32): (1, 32, 32, 64, 3, 2),
        (1024, 1024, 16384, 64, 64): (4, 16, 64, 64, 3, 2),
        (1024, 1024, 16384, 128, 128): (4, 32, 128, 64, 1, 4),
        (1024, 1024, 32768, 16, 16): (3, 16, 16, 128, 1, 2),
        (1024, 1024, 32768, 32, 32): (1, 8, 32, 64, 3, 2),
        (1024, 1024, 32768, 64, 64): (4, 16, 64, 64, 3, 2),
        (1024, 1024, 32768, 128, 128): (4, 8, 128, 64, 2, 4),
        (1024, 1024, 65536, 16, 16): (1, 2, 16, 128, 1, 2),
        (1024, 1024, 65536, 32, 32): (2, 4, 32, 64, 3, 2),
        (1024, 1024, 65536, 64, 64): (5, 16, 64, 64, 3, 2),
        (1024, 1024, 65536, 128, 128): (5, 8, 128, 64, 2, 4),
        (1024, 1024, 131072, 16, 16): (5, 2, 16, 128, 1, 2),
        (1024, 1024, 131072, 32, 32): (1, 2, 32, 64, 3, 2),
        (1024, 1024, 131072, 64, 64): (5, 16, 64, 64, 3, 2),
        (1024, 1024, 131072, 128, 128): (2, 1, 128, 64, 2, 4),
        (2048, 2048, 256, 16, 16): (4, 4, 16, 64, 1, 8),
        (2048, 2048, 256, 32, 32): (4, 8, 32, 32, 1, 8),
        (2048, 2048, 256, 64, 64): (4, 16, 64, 16, 1, 8),
        (2048, 2048, 256, 128, 128): (4, 4, 128, 32, 3, 8),
        (2048, 2048, 512, 16, 16): (2, 2, 16, 64, 1, 2),
        (2048, 2048, 512, 32, 32): (2, 4, 32, 64, 3, 2),
        (2048, 2048, 512, 64, 64): (4, 4, 64, 64, 1, 8),
        (2048, 2048, 512, 128, 128): (4, 8, 64, 64, 1, 4),
        (2048, 2048, 1024, 16, 16): (1, 8, 16, 64, 1, 2),
        (2048, 2048, 1024, 32, 32): (2, 16, 32, 64, 3, 2),
        (2048, 2048, 1024, 64, 64): (4, 8, 64, 64, 1, 4),
        (2048, 2048, 1024, 128, 128): (4, 8, 128, 64, 1, 4),
        (2048, 2048, 2048, 16, 16): (5, 4, 16, 128, 1, 2),
        (2048, 2048, 2048, 32, 32): (1, 16, 32, 64, 3, 2),
        (2048, 2048, 2048, 64, 64): (2, 8, 64, 64, 1, 4),
        (2048, 2048, 2048, 128, 128): (2, 8, 128, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7339-7374 / 第 7339-7374 行
````python
        (2048, 2048, 4096, 16, 16): (4, 2, 16, 128, 1, 2),
        (2048, 2048, 4096, 32, 32): (2, 16, 32, 64, 3, 2),
        (2048, 2048, 4096, 64, 64): (2, 8, 64, 64, 3, 2),
        (2048, 2048, 4096, 128, 128): (4, 8, 128, 64, 1, 4),
        (2048, 2048, 8192, 16, 16): (5, 4, 16, 128, 1, 2),
        (2048, 2048, 8192, 32, 32): (2, 8, 32, 64, 3, 2),
        (2048, 2048, 8192, 64, 64): (4, 8, 64, 64, 3, 2),
        (2048, 2048, 8192, 128, 128): (4, 8, 128, 64, 1, 4),
        (2048, 2048, 16384, 16, 16): (3, 2, 16, 128, 1, 2),
        (2048, 2048, 16384, 32, 32): (2, 4, 32, 128, 3, 2),
        (2048, 2048, 16384, 64, 64): (4, 8, 64, 64, 3, 2),
        (2048, 2048, 16384, 128, 128): (4, 4, 128, 64, 1, 4),
        (2048, 2048, 32768, 16, 16): (3, 2, 16, 128, 1, 2),
        (2048, 2048, 32768, 32, 32): (3, 4, 32, 128, 3, 2),
        (2048, 2048, 32768, 64, 64): (6, 4, 64, 64, 3, 2),
        (2048, 2048, 32768, 128, 128): (3, 4, 128, 64, 1, 4),
        (2048, 2048, 65536, 16, 16): (6, 2, 16, 128, 1, 2),
        (2048, 2048, 65536, 32, 32): (1, 2, 32, 128, 1, 2),
        (2048, 2048, 65536, 64, 64): (5, 4, 64, 64, 3, 2),
        (2048, 2048, 65536, 128, 128): (5, 1, 128, 64, 2, 4),
        (2048, 2048, 131072, 16, 16): (3, 2, 16, 128, 1, 2),
        (2048, 2048, 131072, 32, 32): (2, 1, 32, 128, 3, 2),
        (2048, 2048, 131072, 64, 64): (4, 1, 64, 64, 3, 2),
        (2048, 2048, 131072, 128, 128): (3, 1, 128, 64, 2, 4),
        (4096, 4096, 256, 16, 16): (5, 8, 16, 32, 1, 4),
        (4096, 4096, 256, 32, 32): (4, 16, 32, 16, 2, 4),
        (4096, 4096, 256, 64, 64): (2, 1, 64, 64, 3, 4),
        (4096, 4096, 256, 128, 128): (4, 4, 128, 32, 1, 4),
        (4096, 4096, 512, 16, 16): (4, 2, 16, 128, 1, 2),
        (4096, 4096, 512, 32, 32): (4, 8, 32, 64, 1, 2),
        (4096, 4096, 512, 64, 64): (4, 4, 64, 64, 1, 4),
        (4096, 4096, 512, 128, 128): (4, 8, 128, 64, 2, 4),
        (4096, 4096, 1024, 16, 16): (1, 2, 16, 128, 1, 2),
        (4096, 4096, 1024, 32, 32): (6, 8, 32, 64, 3, 2),
        (4096, 4096, 1024, 64, 64): (2, 16, 64, 64, 4, 4),
        (4096, 4096, 1024, 128, 128): (2, 4, 128, 64, 2, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7375-7410 / 第 7375-7410 行
````python
        (4096, 4096, 2048, 16, 16): (3, 1, 16, 128, 1, 2),
        (4096, 4096, 2048, 32, 32): (1, 4, 32, 64, 5, 2),
        (4096, 4096, 2048, 64, 64): (3, 16, 64, 64, 3, 2),
        (4096, 4096, 2048, 128, 128): (4, 32, 128, 64, 2, 4),
        (4096, 4096, 4096, 16, 16): (1, 2, 16, 128, 1, 2),
        (4096, 4096, 4096, 32, 32): (1, 4, 32, 64, 3, 2),
        (4096, 4096, 4096, 64, 64): (1, 1, 64, 64, 4, 4),
        (4096, 4096, 4096, 128, 128): (2, 1, 128, 128, 1, 8),
        (4096, 4096, 8192, 16, 16): (3, 1, 16, 128, 1, 2),
        (4096, 4096, 8192, 32, 32): (2, 2, 32, 64, 5, 2),
        (4096, 4096, 8192, 64, 64): (4, 16, 64, 64, 3, 2),
        (4096, 4096, 8192, 128, 128): (4, 16, 128, 64, 2, 4),
        (4096, 4096, 16384, 16, 16): (1, 2, 16, 128, 1, 2),
        (4096, 4096, 16384, 32, 32): (4, 2, 32, 64, 5, 2),
        (4096, 4096, 16384, 64, 64): (4, 16, 64, 64, 3, 2),
        (4096, 4096, 16384, 128, 128): (4, 16, 128, 64, 2, 4),
        (4096, 4096, 32768, 16, 16): (3, 1, 16, 128, 1, 2),
        (4096, 4096, 32768, 32, 32): (3, 1, 32, 128, 1, 4),
        (4096, 4096, 32768, 64, 64): (3, 1, 64, 64, 3, 4),
        (4096, 4096, 32768, 128, 128): (5, 16, 128, 64, 2, 4),
        (4096, 4096, 65536, 16, 16): (5, 1, 16, 128, 1, 2),
        (4096, 4096, 65536, 32, 32): (5, 1, 32, 128, 1, 4),
        (4096, 4096, 65536, 64, 64): (1, 1, 64, 64, 3, 4),
        (4096, 4096, 65536, 128, 128): (3, 16, 128, 64, 2, 4),
        (4096, 4096, 131072, 16, 16): (3, 1, 16, 128, 1, 2),
        (4096, 4096, 131072, 32, 32): (3, 1, 32, 128, 3, 2),
        (4096, 4096, 131072, 64, 64): (2, 1, 64, 64, 3, 4),
        (4096, 4096, 131072, 128, 128): (1, 1, 128, 64, 1, 4),
        (8192, 8192, 256, 16, 16): (4, 16, 16, 16, 1, 4),
        (8192, 8192, 256, 32, 32): (1, 16, 32, 16, 4, 4),
        (8192, 8192, 256, 64, 64): (4, 16, 64, 16, 3, 8),
        (8192, 8192, 256, 128, 128): (4, 16, 128, 16, 1, 2),
        (8192, 8192, 512, 16, 16): (2, 8, 16, 64, 1, 4),
        (8192, 8192, 512, 32, 32): (4, 8, 32, 64, 3, 2),
        (8192, 8192, 512, 64, 64): (2, 8, 64, 64, 4, 4),
        (8192, 8192, 512, 128, 128): (4, 8, 128, 64, 2, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7411-7446 / 第 7411-7446 行
````python
        (8192, 8192, 1024, 16, 16): (4, 16, 16, 64, 1, 8),
        (8192, 8192, 1024, 32, 32): (2, 8, 32, 64, 5, 2),
        (8192, 8192, 1024, 64, 64): (1, 16, 64, 64, 3, 2),
        (8192, 8192, 1024, 128, 128): (5, 16, 128, 64, 2, 4),
        (8192, 8192, 2048, 16, 16): (7, 2, 16, 128, 1, 2),
        (8192, 8192, 2048, 32, 32): (1, 16, 32, 64, 5, 2),
        (8192, 8192, 2048, 64, 64): (4, 16, 64, 64, 3, 2),
        (8192, 8192, 2048, 128, 128): (6, 16, 128, 64, 2, 4),
        (8192, 8192, 4096, 16, 16): (4, 2, 16, 128, 1, 2),
        (8192, 8192, 4096, 32, 32): (2, 8, 32, 64, 5, 2),
        (8192, 8192, 4096, 64, 64): (3, 16, 64, 64, 3, 2),
        (8192, 8192, 4096, 128, 128): (3, 64, 128, 64, 2, 4),
        (8192, 8192, 8192, 16, 16): (4, 2, 16, 128, 1, 2),
        (8192, 8192, 8192, 32, 32): (1, 4, 32, 128, 5, 4),
        (8192, 8192, 8192, 64, 64): (4, 4, 64, 64, 1, 4),
        (8192, 8192, 8192, 128, 128): (2, 2, 128, 128, 3, 8),
        (8192, 8192, 16384, 16, 16): (1, 2, 16, 128, 1, 2),
        (8192, 8192, 16384, 32, 32): (4, 8, 32, 64, 5, 2),
        (8192, 8192, 16384, 64, 64): (5, 8, 64, 64, 3, 2),
        (8192, 8192, 16384, 128, 128): (3, 16, 128, 64, 2, 4),
        (8192, 8192, 32768, 16, 16): (7, 2, 16, 128, 1, 2),
        (8192, 8192, 32768, 32, 32): (3, 4, 32, 64, 3, 2),
        (8192, 8192, 32768, 64, 64): (2, 8, 64, 64, 3, 2),
        (8192, 8192, 32768, 128, 128): (6, 16, 128, 64, 2, 4),
        (8192, 8192, 65536, 16, 16): (9, 2, 16, 128, 1, 2),
        (8192, 8192, 65536, 32, 32): (7, 4, 32, 64, 5, 2),
        (8192, 8192, 65536, 64, 64): (4, 8, 64, 64, 3, 2),
        (8192, 8192, 65536, 128, 128): (3, 16, 128, 64, 2, 4),
        (8192, 8192, 131072, 16, 16): (9, 2, 16, 128, 1, 2),
        (8192, 8192, 131072, 32, 32): (1, 8, 32, 64, 5, 2),
        (8192, 8192, 131072, 64, 64): (1, 8, 64, 64, 3, 2),
        (8192, 8192, 131072, 128, 128): (4, 16, 128, 64, 2, 4),
        (16384, 16384, 256, 16, 16): (5, 16, 16, 16, 1, 4),
        (16384, 16384, 256, 32, 32): (4, 16, 32, 16, 4, 4),
        (16384, 16384, 256, 64, 64): (4, 16, 64, 16, 3, 8),
        (16384, 16384, 256, 128, 128): (4, 16, 128, 16, 1, 2),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7447-7482 / 第 7447-7482 行
````python
        (16384, 16384, 512, 16, 16): (2, 8, 16, 64, 1, 4),
        (16384, 16384, 512, 32, 32): (1, 4, 32, 64, 5, 2),
        (16384, 16384, 512, 64, 64): (4, 8, 64, 64, 1, 4),
        (16384, 16384, 512, 128, 128): (3, 8, 128, 64, 2, 4),
        (16384, 16384, 1024, 16, 16): (4, 2, 16, 128, 1, 2),
        (16384, 16384, 1024, 32, 32): (4, 8, 32, 64, 5, 2),
        (16384, 16384, 1024, 64, 64): (6, 16, 64, 64, 3, 2),
        (16384, 16384, 1024, 128, 128): (3, 16, 128, 64, 2, 4),
        (16384, 16384, 2048, 16, 16): (3, 2, 16, 128, 1, 2),
        (16384, 16384, 2048, 32, 32): (1, 8, 32, 64, 5, 2),
        (16384, 16384, 2048, 64, 64): (5, 16, 64, 64, 3, 2),
        (16384, 16384, 2048, 128, 128): (2, 32, 128, 64, 2, 4),
        (16384, 16384, 4096, 16, 16): (2, 2, 16, 128, 1, 2),
        (16384, 16384, 4096, 32, 32): (1, 4, 32, 64, 3, 2),
        (16384, 16384, 4096, 64, 64): (2, 8, 64, 64, 3, 2),
        (16384, 16384, 4096, 128, 128): (3, 16, 128, 64, 2, 4),
        (16384, 16384, 8192, 16, 16): (3, 2, 16, 128, 1, 2),
        (16384, 16384, 8192, 32, 32): (2, 4, 32, 64, 5, 2),
        (16384, 16384, 8192, 64, 64): (4, 8, 64, 64, 3, 2),
        (16384, 16384, 8192, 128, 128): (8, 32, 128, 64, 2, 4),
        (16384, 16384, 16384, 16, 16): (1, 2, 16, 256, 1, 4),
        (16384, 16384, 16384, 32, 32): (1, 4, 32, 128, 3, 4),
        (16384, 16384, 16384, 64, 64): (5, 4, 64, 64, 1, 4),
        (16384, 16384, 16384, 128, 128): (4, 8, 128, 64, 2, 4),
        (16384, 16384, 32768, 16, 16): (2, 2, 16, 128, 1, 2),
        (16384, 16384, 32768, 32, 32): (1, 4, 32, 64, 3, 2),
        (16384, 16384, 32768, 64, 64): (5, 4, 64, 64, 1, 4),
        (16384, 16384, 32768, 128, 128): (5, 8, 128, 64, 2, 4),
        (16384, 16384, 65536, 16, 16): (8, 2, 16, 128, 1, 2),
        (16384, 16384, 65536, 32, 32): (6, 4, 32, 64, 5, 2),
        (16384, 16384, 65536, 64, 64): (2, 4, 64, 64, 1, 4),
        (16384, 16384, 65536, 128, 128): (4, 8, 128, 64, 2, 4),
        (16384, 16384, 131072, 16, 16): (3, 1, 16, 128, 1, 2),
        (16384, 16384, 131072, 32, 32): (1, 4, 32, 64, 3, 2),
        (16384, 16384, 131072, 64, 64): (4, 4, 64, 64, 1, 4),
        (16384, 16384, 131072, 128, 128): (1, 8, 128, 64, 2, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7483-7518 / 第 7483-7518 行
````python
        (32768, 32768, 256, 16, 16): (4, 16, 16, 16, 1, 4),
        (32768, 32768, 512, 16, 16): (4, 2, 16, 128, 1, 2),
        (32768, 32768, 1024, 16, 16): (3, 2, 16, 128, 1, 2),
        (32768, 32768, 2048, 16, 16): (4, 2, 16, 128, 1, 2),
        (32768, 32768, 4096, 16, 16): (5, 4, 16, 64, 1, 1),
        (32768, 32768, 8192, 16, 16): (4, 4, 16, 64, 1, 1),
        (32768, 32768, 16384, 16, 16): (4, 4, 16, 64, 1, 1),
        (32768, 32768, 32768, 16, 16): (5, 4, 16, 64, 1, 1),
    },
    ("scatter_mm", "NVIDIA A100-SXM4-80GB", (0, torch.float32, 0.5)): {
        (256, 256, 256, 16, 16): (1, 1, 16, 16, 1, 8),
        (256, 256, 256, 32, 32): (1, 1, 16, 16, 1, 4),
        (256, 256, 256, 64, 64): (1, 1, 16, 16, 1, 4),
        (256, 256, 256, 128, 128): (1, 1, 16, 16, 1, 1),
        (256, 256, 512, 16, 16): (1, 1, 16, 16, 1, 4),
        (256, 256, 512, 32, 32): (1, 16, 16, 16, 1, 1),
        (256, 256, 512, 64, 64): (1, 1, 16, 16, 1, 1),
        (256, 256, 512, 128, 128): (1, 1, 32, 32, 1, 4),
        (256, 256, 1024, 16, 16): (1, 1, 16, 32, 1, 2),
        (256, 256, 1024, 32, 32): (1, 4, 16, 16, 1, 1),
        (256, 256, 1024, 64, 64): (1, 1, 32, 32, 1, 4),
        (256, 256, 1024, 128, 128): (1, 1, 32, 32, 1, 4),
        (256, 256, 2048, 16, 16): (1, 2, 16, 32, 1, 2),
        (256, 256, 2048, 32, 32): (1, 1, 16, 32, 1, 2),
        (256, 256, 2048, 64, 64): (2, 1, 16, 32, 1, 2),
        (256, 256, 2048, 128, 128): (1, 1, 16, 16, 1, 1),
        (256, 256, 4096, 16, 16): (1, 1, 16, 32, 1, 2),
        (256, 256, 4096, 32, 32): (1, 1, 16, 32, 1, 2),
        (256, 256, 4096, 64, 64): (1, 1, 32, 32, 1, 4),
        (256, 256, 4096, 128, 128): (3, 1, 32, 64, 1, 4),
        (256, 256, 8192, 16, 16): (1, 32, 16, 64, 1, 2),
        (256, 256, 8192, 32, 32): (1, 1, 32, 64, 1, 4),
        (256, 256, 8192, 64, 64): (1, 1, 32, 64, 1, 4),
        (256, 256, 8192, 128, 128): (2, 1, 64, 32, 1, 4),
        (256, 256, 16384, 16, 16): (1, 1, 16, 64, 1, 2),
        (256, 256, 16384, 32, 32): (1, 1, 32, 64, 1, 4),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7519-7554 / 第 7519-7554 行
````python
        (256, 256, 16384, 64, 64): (1, 128, 64, 64, 1, 4),
        (256, 256, 16384, 128, 128): (2, 1, 64, 32, 1, 4),
        (256, 256, 32768, 16, 16): (2, 128, 16, 64, 1, 1),
        (256, 256, 32768, 32, 32): (1, 1, 32, 64, 1, 4),
        (256, 256, 32768, 64, 64): (1, 128, 64, 64, 1, 4),
        (256, 256, 32768, 128, 128): (2, 1, 64, 64, 1, 4),
        (256, 256, 65536, 16, 16): (1, 1, 16, 64, 1, 2),
        (256, 256, 65536, 32, 32): (1, 1, 32, 64, 1, 4),
        (256, 256, 65536, 64, 64): (2, 1, 64, 64, 1, 4),
        (256, 256, 65536, 128, 128): (1, 1, 128, 32, 1, 4),
        (256, 256, 131072, 16, 16): (3, 128, 16, 64, 1, 1),
        (256, 256, 131072, 32, 32): (1, 1, 32, 64, 1, 4),
        (256, 256, 131072, 64, 64): (2, 1, 64, 64, 1, 4),
        (256, 256, 131072, 128, 128): (1, 8192, 64, 16, 1, 4),
        (512, 512, 256, 16, 16): (1, 2, 16, 16, 1, 1),
        (512, 512, 256, 32, 32): (1, 4, 16, 16, 1, 1),
        (512, 512, 256, 64, 64): (1, 16, 16, 16, 1, 1),
        (512, 512, 256, 128, 128): (1, 1, 16, 32, 1, 4),
        (512, 512, 512, 16, 16): (1, 8, 16, 32, 1, 2),
        (512, 512, 512, 32, 32): (1, 8, 16, 32, 1, 2),
        (512, 512, 512, 64, 64): (1, 2, 16, 32, 1, 2),
        (512, 512, 512, 128, 128): (1, 1, 32, 32, 1, 4),
        (512, 512, 1024, 16, 16): (1, 1, 16, 32, 1, 2),
        (512, 512, 1024, 32, 32): (1, 1, 16, 32, 1, 2),
        (512, 512, 1024, 64, 64): (1, 1, 16, 32, 1, 2),
        (512, 512, 1024, 128, 128): (1, 1, 64, 32, 1, 4),
        (512, 512, 2048, 16, 16): (1, 16, 16, 64, 1, 2),
        (512, 512, 2048, 32, 32): (1, 1, 32, 32, 1, 4),
        (512, 512, 2048, 64, 64): (1, 1, 32, 32, 1, 4),
        (512, 512, 2048, 128, 128): (2, 1, 32, 32, 1, 4),
        (512, 512, 4096, 16, 16): (2, 64, 16, 64, 1, 1),
        (512, 512, 4096, 32, 32): (1, 64, 32, 64, 1, 4),
        (512, 512, 4096, 64, 64): (1, 1, 32, 32, 1, 4),
        (512, 512, 4096, 128, 128): (1, 1, 64, 32, 1, 4),
        (512, 512, 8192, 16, 16): (2, 64, 16, 64, 1, 1),
        (512, 512, 8192, 32, 32): (1, 256, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7555-7590 / 第 7555-7590 行
````python
        (512, 512, 8192, 64, 64): (1, 64, 64, 64, 1, 4),
        (512, 512, 8192, 128, 128): (2, 1, 64, 32, 1, 8),
        (512, 512, 16384, 16, 16): (2, 64, 16, 64, 1, 1),
        (512, 512, 16384, 32, 32): (1, 128, 32, 32, 1, 1),
        (512, 512, 16384, 64, 64): (1, 64, 64, 64, 1, 4),
        (512, 512, 16384, 128, 128): (3, 1, 64, 32, 1, 8),
        (512, 512, 32768, 16, 16): (2, 64, 16, 64, 1, 1),
        (512, 512, 32768, 32, 32): (1, 128, 32, 32, 1, 1),
        (512, 512, 32768, 64, 64): (1, 64, 64, 64, 1, 4),
        (512, 512, 32768, 128, 128): (2, 1, 64, 32, 1, 8),
        (512, 512, 65536, 16, 16): (2, 32, 16, 64, 1, 1),
        (512, 512, 65536, 32, 32): (1, 128, 32, 32, 1, 1),
        (512, 512, 65536, 64, 64): (1, 64, 64, 64, 1, 4),
        (512, 512, 65536, 128, 128): (2, 1, 64, 32, 1, 8),
        (512, 512, 131072, 16, 16): (2, 32, 16, 64, 1, 1),
        (512, 512, 131072, 32, 32): (1, 128, 32, 32, 1, 1),
        (512, 512, 131072, 64, 64): (3, 64, 64, 64, 1, 4),
        (512, 512, 131072, 128, 128): (1, 8192, 64, 16, 1, 4),
        (1024, 1024, 256, 16, 16): (1, 4, 16, 32, 1, 2),
        (1024, 1024, 256, 32, 32): (1, 4, 16, 32, 1, 2),
        (1024, 1024, 256, 64, 64): (1, 1, 16, 32, 1, 2),
        (1024, 1024, 256, 128, 128): (1, 1, 16, 16, 1, 1),
        (1024, 1024, 512, 16, 16): (1, 8, 16, 32, 1, 2),
        (1024, 1024, 512, 32, 32): (1, 8, 16, 32, 1, 1),
        (1024, 1024, 512, 64, 64): (1, 8, 32, 32, 1, 4),
        (1024, 1024, 512, 128, 128): (2, 1, 32, 32, 1, 4),
        (1024, 1024, 1024, 16, 16): (1, 16, 16, 32, 1, 2),
        (1024, 1024, 1024, 32, 32): (1, 16, 32, 64, 1, 4),
        (1024, 1024, 1024, 64, 64): (1, 16, 32, 64, 1, 4),
        (1024, 1024, 1024, 128, 128): (1, 1, 32, 32, 1, 4),
        (1024, 1024, 2048, 16, 16): (2, 32, 16, 64, 1, 1),
        (1024, 1024, 2048, 32, 32): (1, 32, 32, 64, 1, 4),
        (1024, 1024, 2048, 64, 64): (1, 32, 64, 64, 1, 4),
        (1024, 1024, 2048, 128, 128): (1, 1, 32, 64, 1, 4),
        (1024, 1024, 4096, 16, 16): (2, 16, 16, 64, 1, 1),
        (1024, 1024, 4096, 32, 32): (1, 64, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7591-7626 / 第 7591-7626 行
````python
        (1024, 1024, 4096, 64, 64): (1, 64, 64, 64, 1, 4),
        (1024, 1024, 4096, 128, 128): (2, 64, 64, 32, 1, 8),
        (1024, 1024, 8192, 16, 16): (2, 16, 16, 64, 1, 1),
        (1024, 1024, 8192, 32, 32): (1, 64, 32, 32, 1, 1),
        (1024, 1024, 8192, 64, 64): (1, 64, 64, 64, 1, 4),
        (1024, 1024, 8192, 128, 128): (4, 1, 32, 64, 1, 4),
        (1024, 1024, 16384, 16, 16): (2, 16, 16, 64, 1, 1),
        (1024, 1024, 16384, 32, 32): (1, 64, 32, 32, 1, 1),
        (1024, 1024, 16384, 64, 64): (1, 32, 64, 64, 1, 4),
        (1024, 1024, 16384, 128, 128): (2, 64, 64, 32, 1, 4),
        (1024, 1024, 32768, 16, 16): (2, 16, 16, 64, 1, 1),
        (1024, 1024, 32768, 32, 32): (1, 64, 32, 32, 1, 1),
        (1024, 1024, 32768, 64, 64): (1, 32, 64, 64, 1, 4),
        (1024, 1024, 32768, 128, 128): (4, 1, 32, 64, 1, 4),
        (1024, 1024, 65536, 16, 16): (2, 16, 16, 64, 1, 1),
        (1024, 1024, 65536, 32, 32): (1, 32, 32, 32, 1, 1),
        (1024, 1024, 65536, 64, 64): (2, 32, 64, 64, 1, 4),
        (1024, 1024, 65536, 128, 128): (4, 1, 64, 32, 1, 4),
        (1024, 1024, 131072, 16, 16): (2, 16, 16, 64, 1, 1),
        (1024, 1024, 131072, 32, 32): (1, 32, 32, 32, 1, 1),
        (1024, 1024, 131072, 64, 64): (1, 16, 64, 64, 1, 4),
        (1024, 1024, 131072, 128, 128): (1, 8192, 64, 16, 1, 4),
        (2048, 2048, 256, 16, 16): (1, 4, 16, 32, 1, 2),
        (2048, 2048, 256, 32, 32): (1, 8, 16, 32, 1, 1),
        (2048, 2048, 256, 64, 64): (1, 8, 32, 32, 1, 4),
        (2048, 2048, 256, 128, 128): (1, 4, 64, 64, 1, 8),
        (2048, 2048, 512, 16, 16): (2, 8, 16, 32, 1, 2),
        (2048, 2048, 512, 32, 32): (2, 8, 32, 64, 1, 4),
        (2048, 2048, 512, 64, 64): (2, 4, 64, 64, 1, 4),
        (2048, 2048, 512, 128, 128): (1, 8, 32, 64, 1, 4),
        (2048, 2048, 1024, 16, 16): (2, 16, 16, 64, 3, 1),
        (2048, 2048, 1024, 32, 32): (1, 32, 32, 32, 1, 1),
        (2048, 2048, 1024, 64, 64): (1, 16, 64, 64, 1, 4),
        (2048, 2048, 1024, 128, 128): (2, 4, 64, 64, 1, 8),
        (2048, 2048, 2048, 16, 16): (2, 16, 16, 64, 1, 1),
        (2048, 2048, 2048, 32, 32): (1, 32, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7627-7662 / 第 7627-7662 行
````python
        (2048, 2048, 2048, 64, 64): (1, 16, 64, 64, 1, 4),
        (2048, 2048, 2048, 128, 128): (2, 32, 32, 64, 1, 4),
        (2048, 2048, 4096, 16, 16): (3, 2, 16, 64, 1, 1),
        (2048, 2048, 4096, 32, 32): (3, 4, 32, 32, 1, 1),
        (2048, 2048, 4096, 64, 64): (1, 16, 64, 64, 1, 4),
        (2048, 2048, 4096, 128, 128): (2, 32, 64, 32, 1, 4),
        (2048, 2048, 8192, 16, 16): (3, 4, 16, 64, 1, 1),
        (2048, 2048, 8192, 32, 32): (2, 4, 32, 32, 1, 1),
        (2048, 2048, 8192, 64, 64): (2, 32, 64, 32, 1, 2),
        (2048, 2048, 8192, 128, 128): (4, 1, 32, 64, 1, 4),
        (2048, 2048, 16384, 16, 16): (3, 4, 16, 64, 1, 1),
        (2048, 2048, 16384, 32, 32): (1, 4, 32, 32, 1, 1),
        (2048, 2048, 16384, 64, 64): (2, 8, 64, 32, 1, 2),
        (2048, 2048, 16384, 128, 128): (2, 8, 64, 32, 1, 4),
        (2048, 2048, 32768, 16, 16): (2, 4, 16, 64, 1, 1),
        (2048, 2048, 32768, 32, 32): (2, 8, 32, 32, 1, 1),
        (2048, 2048, 32768, 64, 64): (1, 16, 64, 32, 1, 2),
        (2048, 2048, 32768, 128, 128): (4, 1, 32, 64, 1, 4),
        (2048, 2048, 65536, 16, 16): (3, 4, 16, 64, 1, 1),
        (2048, 2048, 65536, 32, 32): (1, 8, 32, 32, 1, 1),
        (2048, 2048, 65536, 64, 64): (1, 8, 64, 32, 1, 2),
        (2048, 2048, 65536, 128, 128): (4, 1, 64, 32, 1, 4),
        (2048, 2048, 131072, 16, 16): (2, 4, 16, 64, 1, 1),
        (2048, 2048, 131072, 32, 32): (1, 8, 32, 32, 1, 1),
        (2048, 2048, 131072, 64, 64): (3, 1, 64, 32, 1, 2),
        (2048, 2048, 131072, 128, 128): (1, 8192, 128, 16, 1, 8),
        (4096, 4096, 256, 16, 16): (2, 4, 16, 32, 1, 2),
        (4096, 4096, 256, 32, 32): (1, 4, 32, 64, 1, 4),
        (4096, 4096, 256, 64, 64): (1, 4, 64, 64, 1, 4),
        (4096, 4096, 256, 128, 128): (1, 4, 32, 64, 1, 4),
        (4096, 4096, 512, 16, 16): (2, 8, 16, 64, 3, 1),
        (4096, 4096, 512, 32, 32): (2, 16, 32, 32, 1, 1),
        (4096, 4096, 512, 64, 64): (1, 8, 64, 64, 1, 4),
        (4096, 4096, 512, 128, 128): (1, 8, 32, 64, 1, 4),
        (4096, 4096, 1024, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 1024, 32, 32): (1, 16, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7663-7698 / 第 7663-7698 行
````python
        (4096, 4096, 1024, 64, 64): (1, 16, 64, 32, 1, 2),
        (4096, 4096, 1024, 128, 128): (1, 16, 32, 64, 1, 4),
        (4096, 4096, 2048, 16, 16): (1, 16, 16, 64, 3, 1),
        (4096, 4096, 2048, 32, 32): (1, 16, 32, 32, 1, 1),
        (4096, 4096, 2048, 64, 64): (3, 16, 64, 32, 1, 2),
        (4096, 4096, 2048, 128, 128): (4, 8, 32, 64, 1, 4),
        (4096, 4096, 4096, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 4096, 32, 32): (1, 1, 32, 32, 1, 1),
        (4096, 4096, 4096, 64, 64): (2, 16, 64, 32, 1, 2),
        (4096, 4096, 4096, 128, 128): (4, 8, 32, 64, 1, 4),
        (4096, 4096, 8192, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 8192, 32, 32): (2, 1, 32, 32, 1, 1),
        (4096, 4096, 8192, 64, 64): (1, 16, 64, 32, 1, 2),
        (4096, 4096, 8192, 128, 128): (2, 1, 32, 64, 1, 4),
        (4096, 4096, 16384, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 16384, 32, 32): (1, 1, 32, 32, 1, 1),
        (4096, 4096, 16384, 64, 64): (2, 8, 64, 32, 1, 2),
        (4096, 4096, 16384, 128, 128): (2, 1, 32, 64, 1, 4),
        (4096, 4096, 32768, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 32768, 32, 32): (1, 1, 32, 32, 1, 1),
        (4096, 4096, 32768, 64, 64): (1, 8, 64, 32, 1, 2),
        (4096, 4096, 32768, 128, 128): (2, 1, 32, 64, 1, 4),
        (4096, 4096, 65536, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 65536, 32, 32): (3, 1, 32, 32, 1, 1),
        (4096, 4096, 65536, 64, 64): (3, 4, 64, 32, 1, 2),
        (4096, 4096, 65536, 128, 128): (2, 1, 32, 64, 1, 4),
        (4096, 4096, 131072, 16, 16): (1, 8, 16, 64, 3, 1),
        (4096, 4096, 131072, 32, 32): (1, 1, 32, 32, 1, 1),
        (4096, 4096, 131072, 64, 64): (2, 8, 64, 32, 1, 2),
        (4096, 4096, 131072, 128, 128): (1, 8192, 128, 16, 1, 8),
        (8192, 8192, 256, 16, 16): (2, 4, 16, 64, 3, 1),
        (8192, 8192, 256, 32, 32): (1, 8, 32, 32, 1, 1),
        (8192, 8192, 256, 64, 64): (1, 4, 64, 64, 1, 4),
        (8192, 8192, 256, 128, 128): (1, 4, 32, 64, 1, 4),
        (8192, 8192, 512, 16, 16): (1, 4, 16, 64, 3, 1),
        (8192, 8192, 512, 32, 32): (1, 16, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7699-7734 / 第 7699-7734 行
````python
        (8192, 8192, 512, 64, 64): (2, 4, 64, 64, 1, 4),
        (8192, 8192, 512, 128, 128): (2, 1, 32, 64, 1, 4),
        (8192, 8192, 1024, 16, 16): (3, 8, 16, 64, 3, 1),
        (8192, 8192, 1024, 32, 32): (1, 16, 32, 32, 1, 1),
        (8192, 8192, 1024, 64, 64): (1, 8, 64, 32, 1, 2),
        (8192, 8192, 1024, 128, 128): (2, 4, 32, 64, 1, 4),
        (8192, 8192, 2048, 16, 16): (1, 8, 16, 64, 3, 1),
        (8192, 8192, 2048, 32, 32): (1, 16, 32, 32, 1, 1),
        (8192, 8192, 2048, 64, 64): (2, 8, 64, 32, 1, 2),
        (8192, 8192, 2048, 128, 128): (4, 1, 32, 64, 1, 4),
        (8192, 8192, 4096, 16, 16): (1, 8, 16, 64, 3, 1),
        (8192, 8192, 4096, 32, 32): (1, 16, 32, 32, 1, 1),
        (8192, 8192, 4096, 64, 64): (1, 4, 64, 32, 1, 2),
        (8192, 8192, 4096, 128, 128): (3, 1, 32, 64, 1, 4),
        (8192, 8192, 8192, 16, 16): (1, 8, 16, 64, 3, 1),
        (8192, 8192, 8192, 32, 32): (1, 8, 32, 32, 1, 1),
        (8192, 8192, 8192, 64, 64): (1, 8, 64, 32, 1, 2),
        (8192, 8192, 8192, 128, 128): (4, 1, 32, 64, 1, 4),
        (8192, 8192, 16384, 16, 16): (3, 4, 16, 64, 3, 1),
        (8192, 8192, 16384, 32, 32): (1, 8, 32, 32, 1, 1),
        (8192, 8192, 16384, 64, 64): (2, 2, 64, 32, 1, 2),
        (8192, 8192, 16384, 128, 128): (7, 1, 32, 64, 1, 4),
        (8192, 8192, 32768, 16, 16): (1, 4, 16, 64, 3, 1),
        (8192, 8192, 32768, 32, 32): (1, 8, 32, 32, 1, 1),
        (8192, 8192, 32768, 64, 64): (3, 2, 64, 32, 1, 2),
        (8192, 8192, 32768, 128, 128): (6, 1, 32, 64, 1, 4),
        (8192, 8192, 65536, 16, 16): (1, 4, 16, 64, 3, 1),
        (8192, 8192, 65536, 32, 32): (4, 8, 32, 32, 1, 1),
        (8192, 8192, 65536, 64, 64): (1, 2, 64, 32, 1, 2),
        (8192, 8192, 65536, 128, 128): (4, 1, 32, 64, 1, 4),
        (8192, 8192, 131072, 16, 16): (1, 4, 16, 64, 3, 1),
        (8192, 8192, 131072, 32, 32): (1, 8, 32, 32, 1, 1),
        (8192, 8192, 131072, 64, 64): (5, 4, 64, 32, 1, 2),
        (8192, 8192, 131072, 128, 128): (1, 4096, 128, 16, 1, 8),
        (16384, 16384, 256, 16, 16): (1, 4, 16, 64, 3, 1),
        (16384, 16384, 256, 32, 32): (1, 8, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7735-7770 / 第 7735-7770 行
````python
        (16384, 16384, 256, 64, 64): (1, 4, 64, 32, 1, 2),
        (16384, 16384, 256, 128, 128): (1, 4, 32, 64, 1, 4),
        (16384, 16384, 512, 16, 16): (1, 8, 16, 64, 3, 1),
        (16384, 16384, 512, 32, 32): (1, 16, 32, 32, 1, 1),
        (16384, 16384, 512, 64, 64): (1, 4, 64, 32, 1, 2),
        (16384, 16384, 512, 128, 128): (3, 1, 32, 64, 1, 4),
        (16384, 16384, 1024, 16, 16): (1, 8, 16, 64, 3, 1),
        (16384, 16384, 1024, 32, 32): (1, 16, 32, 32, 1, 1),
        (16384, 16384, 1024, 64, 64): (2, 4, 64, 32, 1, 2),
        (16384, 16384, 1024, 128, 128): (1, 2, 32, 64, 1, 4),
        (16384, 16384, 2048, 16, 16): (1, 4, 16, 64, 3, 1),
        (16384, 16384, 2048, 32, 32): (1, 16, 32, 32, 1, 1),
        (16384, 16384, 2048, 64, 64): (3, 4, 64, 32, 1, 2),
        (16384, 16384, 2048, 128, 128): (2, 1, 32, 64, 1, 4),
        (16384, 16384, 4096, 16, 16): (4, 8, 16, 64, 3, 1),
        (16384, 16384, 4096, 32, 32): (5, 16, 32, 32, 1, 1),
        (16384, 16384, 4096, 64, 64): (3, 2, 64, 32, 1, 2),
        (16384, 16384, 4096, 128, 128): (2, 1, 32, 64, 1, 4),
        (16384, 16384, 8192, 16, 16): (1, 4, 16, 64, 3, 1),
        (16384, 16384, 8192, 32, 32): (1, 4, 32, 32, 1, 1),
        (16384, 16384, 8192, 64, 64): (1, 2, 64, 32, 1, 2),
        (16384, 16384, 8192, 128, 128): (2, 1, 32, 64, 1, 4),
        (16384, 16384, 16384, 16, 16): (1, 8, 16, 64, 3, 1),
        (16384, 16384, 16384, 32, 32): (1, 4, 32, 32, 1, 1),
        (16384, 16384, 16384, 64, 64): (1, 2, 64, 32, 1, 2),
        (16384, 16384, 16384, 128, 128): (3, 1, 32, 64, 1, 4),
        (16384, 16384, 32768, 16, 16): (1, 4, 16, 64, 3, 1),
        (16384, 16384, 32768, 32, 32): (1, 2, 32, 32, 1, 1),
        (16384, 16384, 32768, 64, 64): (3, 2, 64, 32, 1, 2),
        (16384, 16384, 32768, 128, 128): (3, 1, 32, 64, 1, 4),
        (16384, 16384, 65536, 16, 16): (1, 8, 16, 64, 3, 1),
        (16384, 16384, 65536, 32, 32): (1, 4, 32, 32, 1, 1),
        (16384, 16384, 65536, 64, 64): (4, 4, 64, 32, 1, 2),
        (16384, 16384, 65536, 128, 128): (5, 1, 32, 64, 1, 4),
        (16384, 16384, 131072, 16, 16): (1, 2, 16, 64, 3, 1),
        (16384, 16384, 131072, 32, 32): (1, 4, 32, 32, 1, 1),
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。

### Lines 7771-7783 / 第 7771-7783 行
````python
        (16384, 16384, 131072, 64, 64): (1, 2, 64, 32, 1, 2),
        (16384, 16384, 131072, 128, 128): (1, 4096, 128, 16, 1, 8),
    },
    # END GENERATED DATA
}

if __name__ == "__main__":
    for dtype in [torch.int8]:
        for op in ["_int_bsr_dense_addmm"]:
            main(op=op, force=False, dtype=dtype)
    for dtype in [torch.float16, torch.bfloat16, torch.float32, torch.int8]:
        for op in ["bsr_dense_addmm"]:
            main(op=op, force=False, dtype=dtype)
````
- **EN**: This chunk continues `test_func` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `test_func`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

## Key Concepts / 关键概念

- **Sparse tensors**
  - EN: Carries sparse layouts, sparse operator wrappers, and utilities that respect sparse semantics.
  - CN: 承载稀疏布局、稀疏算子包装层以及遵守稀疏语义的工具。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **_get_device_name**
  - EN: `_get_device_name` is one of the main symbols declared or implemented in this file.
  - CN: `_get_device_name` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.hub`, `torch.testing`, `torch.sparse._triton_ops`
- **Standard library / 标准库**: `inspect`, `itertools`, `re`, `warnings`, `typing`
- **Other helper packages / 其他辅助包**: `triton`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `_get_device_name`, `get_meta`, `update`, `dump`, `minimize`, `create_blocked_tensor`, `optimize_scatter_mm`, `tune__int_bsr_dense_addmm`, `tune_bsr_dense_addmm`
