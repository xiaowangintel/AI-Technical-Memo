# numeric_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/numeric_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `set_deterministic`, `clean_memory`, `compare_dict_tensors`, `compare_tuple_tensors`, `compare_parameters`, `compare_forward_output`, and `...+3`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `set_deterministic`、`clean_memory`、`compare_dict_tensors`、`compare_tuple_tensors`、`compare_parameters`、`compare_forward_output`、`另有3项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs
import gc
import logging
import os
import random
import traceback

import numpy

import torch
import torch.optim as optim
from torch.utils._ordered_set import OrderedSet

from .. import config
````
- **EN**: Imports dependencies such as `gc`, `logging`, `os`, `random`, `traceback`, `numpy`, and `...+4` for the logic in this range.
- **CN**: 这里导入了 `gc`、`logging`、`os`、`random`、`traceback`、`numpy`、`另有4项` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python


logger: logging.Logger = logging.getLogger(__name__)

MAIN_RANDOM_SEED = 1337

# Set the CUBLAS_WORKSPACE_CONFIG environment variable
os.environ["CUBLAS_WORKSPACE_CONFIG"] = ":4096:8"


# If the two forward functions involve any non-deterministic operations,
# such as certain types of parallelism or asynchronous execution,
# this can also lead to different outputs.
def set_deterministic() -> None:
````
- **EN**: Introduces function `set_deterministic`. Initializes or updates values such as `logger`, and `MAIN_RANDOM_SEED`.
- **CN**: 这里定义了函数`set_deterministic`。初始化或更新了 `logger`、`MAIN_RANDOM_SEED` 等值。

### Lines 29-42 / 第 29-42 行
````python
    """Make torch manual seed deterministic."""

    torch.manual_seed(MAIN_RANDOM_SEED)
    random.seed(MAIN_RANDOM_SEED)
    numpy.random.seed(MAIN_RANDOM_SEED)
    torch.use_deterministic_algorithms(True)


def clean_memory() -> None:
    """Clean memory to avoid OOM."""
    gc.collect()
    torch.cuda.empty_cache()


````
- **EN**: Introduces function `clean_memory`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`clean_memory`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 43-56 / 第 43-56 行
````python
# We compare the numerical results before and after pre/post grad fx passes
# transformation to make sure the numerical results are the same.
def compare_dict_tensors(dict_base, dict_control, precision):
    if len(OrderedSet(dict_base.keys())) != len(OrderedSet(dict_control.keys())):
        logger.warning("Mismatch keys found before and after pre/post grad fx passes.")
        logger.debug("keys before pre/post grad fx passes %s", dict_base.keys())
        logger.debug("keys after pre/post grad fx passes %s", dict_control.keys())
        return False
    is_allclose = True
    for key in dict_base:
        if key not in dict_control:
            logger.warning(
                "Mismatch parameter name %s does not exist after pre/post grad fx passes",
                key,
````
- **EN**: Introduces function `compare_dict_tensors`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_allclose`.
- **CN**: 这里定义了函数`compare_dict_tensors`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_allclose` 等值。

### Lines 57-70 / 第 57-70 行
````python
            )
        # Some parameters have `None`, and not every param has a valid .grad field, we skip them
        if dict_base[key] is None or dict_control[key] is None:
            continue
        if not torch.allclose(
            dict_base[key],
            dict_control[key],
            rtol=precision,
            atol=precision,
            equal_nan=True,
        ):
            logger.warning(
                "Mismatch parameter values found before and after pre/post grad fx passes."
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `rtol`, `atol`, and `equal_nan`. This range continues the implementation of function `compare_dict_tensors`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `rtol`、`atol`、`equal_nan` 等值。这一段延续了函数`compare_dict_tensors` 的具体实现。

### Lines 71-84 / 第 71-84 行
````python
            logger.debug("value before pre/post grad fx passes %s", dict_base[key])
            logger.debug("value after pre/post grad fx passes %s", dict_control[key])
            is_allclose = False
    return is_allclose


def compare_tuple_tensors(tuple_base, tuple_control, precision):
    if len(tuple_base) != len(tuple_control):
        logger.warning(
            "Mismatch fw output length. before transformation: %s, after transformation: %s",
            len(tuple_base),
            len(tuple_control),
        )
        return False
````
- **EN**: Introduces function `compare_tuple_tensors`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_allclose`.
- **CN**: 这里定义了函数`compare_tuple_tensors`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_allclose` 等值。

### Lines 85-98 / 第 85-98 行
````python
    is_allclose = True
    for i in range(len(tuple_base)):
        # Some parameters have `None`, we skip them
        if tuple_base[i] is None or tuple_control[i] is None:
            continue
        if not torch.allclose(
            tuple_base[i],
            tuple_control[i],
            rtol=precision,
            atol=precision,
            equal_nan=True,
        ):
            logger.debug(
                "forward output before pre/post grad fx passes %s", tuple_base[i]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_allclose`, `rtol`, `atol`, and `equal_nan`. This range continues the implementation of function `compare_tuple_tensors`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `is_allclose`、`rtol`、`atol`、`equal_nan` 等值。这一段延续了函数`compare_tuple_tensors` 的具体实现。

### Lines 99-112 / 第 99-112 行
````python
            )
            logger.debug(
                "forward output after pre/post grad fx passes %s", tuple_control[i]
            )
            is_allclose = False
    return is_allclose


def compare_parameters(model_base, model_control, precision):
    return compare_dict_tensors(
        dict(model_base.named_parameters()),
        dict(model_control.named_parameters()),
        precision,
    )
````
- **EN**: Introduces function `compare_parameters`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `is_allclose`.
- **CN**: 这里定义了函数`compare_parameters`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `is_allclose` 等值。

### Lines 113-126 / 第 113-126 行
````python


def compare_forward_output(pred_base, pred_control, precision):
    return compare_tuple_tensors(
        pred_base,
        pred_control,
        precision,
    )


def compare_gradients(model_base, model_control, precision):
    grad_base = {key: param.grad for key, param in model_base.named_parameters()}
    grad_pt2 = {key: param.grad for key, param in model_control.named_parameters()}
    return compare_dict_tensors(
````
- **EN**: Introduces function `compare_forward_output`, function `compare_gradients`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `grad_base`, and `grad_pt2`.
- **CN**: 这里定义了函数`compare_forward_output`、函数`compare_gradients`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `grad_base`、`grad_pt2` 等值。

### Lines 127-140 / 第 127-140 行
````python
        grad_base,
        grad_pt2,
        precision,
    )


def run_model(
    model_base, model_control, model_input, num_iterations=10, precision=1e-4
):
    clean_memory()
    for i in range(num_iterations):
        logger.info("start %s iteration", i)
        set_deterministic()
        pred_base = model_base(*model_input)
````
- **EN**: Introduces function `run_model`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pred_base`.
- **CN**: 这里定义了函数`run_model`。包含分支、循环或上下文管理等控制流。初始化或更新了 `pred_base` 等值。

### Lines 141-154 / 第 141-154 行
````python
        set_deterministic()
        pred_control = model_control(*model_input)

        res = compare_parameters(model_base, model_control, precision)
        logger.info("compare parameters. Numerical result : %s", res)

        res = compare_forward_output(pred_base, pred_control, precision)
        logger.info("compare loss/predict. Numerical result : %s", res)
        # tensor may not have a grad_fn
        try:
            _ = pred_base[0].sum().backward(retain_graph=True)
            _ = pred_control[0].sum().backward(retain_graph=True)
            res = compare_gradients(model_base, model_control, precision)
            logger.info("compare param grad. Numerical result : %s", res)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pred_control`, `res`, `try`, and `_`. This range continues the implementation of function `run_model`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `pred_control`、`res`、`try`、`_` 等值。这一段延续了函数`run_model` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python
        except Exception:
            logger.exception("Exception when comparing gradients")
            traceback.print_exc()

        if config.fx_passes_numeric_check["requires_optimizer"]:
            try:
                optimizer_base = optim.SGD(
                    [param for name, param in model_base.named_parameters()], lr=0.01
                )
                optimizer_base.step()

                optimizer_control = optim.SGD(
                    [param for name, param in model_control.named_parameters()], lr=0.01
                )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, `optimizer_base`, and `optimizer_control`. This range continues the implementation of function `run_model`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`optimizer_base`、`optimizer_control` 等值。这一段延续了函数`run_model` 的具体实现。

### Lines 169-182 / 第 169-182 行
````python
                optimizer_control.step()

                res = compare_parameters(model_base, model_control, precision)
                logger.info(
                    "compare parameters with optimizer added. Numerical result : %s",
                    res,
                )
            except Exception:
                logger.exception(
                    "Exception when optimizer is added to check parameter names"
                )
                traceback.print_exc()
        else:
            logger.warning(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `res`, and `else`. This range continues the implementation of function `run_model`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `res`、`else` 等值。这一段延续了函数`run_model` 的具体实现。

### Lines 183-196 / 第 183-196 行
````python
                "no parameter with optimizer to compare with length %s before transformation"
                " and the length %s after transformation",
                len(dict(model_base.named_parameters())),
                len(dict(model_control.named_parameters())),
            )


def numeric_check_if_enabled(
    gm_before_fx_passes,
    gm_after_fx_passes,
    example_inputs,
    num_iterations,
    precision,
):
````
- **EN**: Introduces function `numeric_check_if_enabled`.
- **CN**: 这里定义了函数`numeric_check_if_enabled`。

### Lines 197-210 / 第 197-210 行
````python
    # need to topo-sort graphmodule before we run the model,
    # otherwise it may fail as refer before def
    # fail silently in order not to block the model run
    try:
        with torch.autograd.set_detect_anomaly(True):
            run_model(
                gm_before_fx_passes,
                gm_after_fx_passes,
                example_inputs,
                num_iterations=num_iterations,
                precision=precision,
            )
    except Exception as e:
        logger.warning(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, `num_iterations`, and `precision`. This range continues the implementation of function `numeric_check_if_enabled`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`num_iterations`、`precision` 等值。这一段延续了函数`numeric_check_if_enabled` 的具体实现。

### Lines 211-213 / 第 211-213 行
````python
            "Runtime numeric check failed in pre grad fx passes with error: %s", e
        )
        traceback.print_exc()
````
- **EN**: This range continues the implementation of function `numeric_check_if_enabled`.
- **CN**: 这一段延续了函数`numeric_check_if_enabled` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary functions: `set_deterministic`, `clean_memory`, `compare_dict_tensors`, `compare_tuple_tensors`, `compare_parameters`, `compare_forward_output`, and `...+3`  
  **CN**: 主要函数：`set_deterministic`、`clean_memory`、`compare_dict_tensors`、`compare_tuple_tensors`、`compare_parameters`、`compare_forward_output`、`另有3项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `gc`, `logging`, `os`, `random`, `traceback`
- **Third-party / 第三方**: `numpy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.optim`, `torch.utils._ordered_set`, `..`
