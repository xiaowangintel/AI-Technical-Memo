# fused_moe_triton_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/triton_utils/fused_moe_triton_config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `get_config_file_name`, `get_moe_configs`, `get_default_config`, and `try_get_optimal_moe_config` and connects them to backend-specific paths such as `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `get_config_file_name`、`get_moe_configs`、`get_default_config` 以及 `try_get_optimal_moe_config` 等符号，并把这些符号连接到 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports, constants, and runtime setup
```python
from __future__ import annotations

import functools
import json
import logging
import os
from typing import Any, Dict, List, Optional, Tuple

import torch
import triton

from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import get_device_name, is_hip

logger = logging.getLogger(__name__)
_is_hip = is_hip()
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `functools`, `json`, `logging`, `os`, and `typing.Any`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` and `_is_hip` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`functools`、`json`、`logging`、`os` 以及 `typing.Any`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 和 `_is_hip` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 19-36: `get_config_file_name` getter for config file name
```python
def get_config_file_name(
    E: int,
    N: int,
    dtype: Optional[str],
    block_shape: Optional[int] = None,
    per_channel_quant: bool = False,
    down_moe: bool = False,
) -> str:
    device_name = get_device_name().replace(" ", "_")
    dtype_selector = "" if not dtype else f",dtype={dtype}"
    block_shape_selector = (
        "" if not block_shape or not all(block_shape) else f",block_shape={block_shape}"
    )
    per_channel_quant_selector = ",per_channel_quant=True" if per_channel_quant else ""
    down_moe_selector = "_down" if down_moe else ""
    return f"E={E},N={N},device_name={device_name}{dtype_selector}{block_shape_selector}{per_channel_quant_selector}{down_moe_selector}.json"
```
**EN:** This block defines `get_config_file_name` and contains the main logic for this step. It mainly invokes `get_device_name.replace`, `get_device_name`, and `all`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `device_name`, `dtype_selector`, `block_shape_selector`, `per_channel_quant_selector`, and `down_moe_selector` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_config_file_name`，并承载这一阶段的核心逻辑。 它主要调用 `get_device_name.replace`、`get_device_name` 以及 `all`，说明该流程会编排底层辅助函数或计算内核。 像 `device_name`、`dtype_selector`、`block_shape_selector`、`per_channel_quant_selector` 以及 `down_moe_selector` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 37-145: `get_moe_configs` getter for Mixture-of-Experts configs
```python
@functools.lru_cache
def get_moe_configs(
    E: int,
    N: int,
    dtype: Optional[str],
    block_n: Optional[int] = 0,
    block_k: Optional[int] = 0,
    per_channel_quant: bool = False,
    down_moe: bool = False,
) -> Optional[Dict[int, Any]]:
    """
    Return optimized configurations for the fused MoE kernel.

    The return value will be a dictionary that maps an irregular grid of
    batch sizes to configurations of the fused_moe kernel. To evaluate the
    kernel on a given batch size bs, the closest batch size in the grid should
    be picked and the associated configuration chosen to invoke the kernel.
    """
    if get_global_server_args().enable_deterministic_inference:
        logger.warning(
            "Deterministic inference is enabled, using default MoE kernel config."
        )
        return None

    # First look up if an optimized configuration is available in the configs
    # directory
    json_file_name = get_config_file_name(
        E,
        N,
        dtype,
        [block_n, block_k],
        per_channel_quant,
        down_moe=down_moe,
    )

    # We found that using the fused_moe_kernel config from Triton 3.1.0 with Triton 3.2.0 results in negative performance gains,
    # so we also include the Triton version as a key for finding the fused_moe_kernel config to achieve the best performance.
    config_dir = os.environ.get(
        "SGLANG_MOE_CONFIG_DIR", os.path.dirname(os.path.realpath(__file__))
    )

    triton_version = triton.__version__
    version_dir = f"triton_{triton_version.replace('.', '_')}"
    config_file_path = os.path.join(
        config_dir,
        "configs",
        version_dir,
        json_file_name,
    )
    if os.path.exists(config_file_path):
        with open(config_file_path) as f:
            # Please note that although we find the config files, performance might still be suboptimal.
            # This is because the tuning environment might differ from your current environment.
            # For example, updating the Triton version might cause all old configs to become suboptimal.
            # To achieve the best performance, consider re-tuning the Triton fused MOE kernel in your environment.
            # For the tuning method, refer to: https://github.com/sgl-project/sglang/tree/main/benchmark/kernels/fused_moe_triton
            logger.info(f"Using MoE kernel config from {config_file_path}.")
            # If a configuration has been found, return it
            return {int(key): val for key, val in json.load(f).items()}

    # Discover available triton config dirs on disk and search newest-first.
    configs_root = os.path.join(config_dir, "configs")
    available_versions = sorted(
        (
            d.removeprefix("triton_").replace("_", ".")
            for d in os.listdir(configs_root)
            if d.startswith("triton_")
        ),
        key=lambda v: tuple(int(x) for x in v.split(".")),
        reverse=True,
    )

    for try_triton_version in available_versions:
        if try_triton_version == triton_version:
            continue
        try_config_file_path = os.path.join(
            configs_root,
            f"triton_{try_triton_version.replace('.', '_')}",
            json_file_name,
        )
        if os.path.exists(try_config_file_path):
            with open(try_config_file_path) as f:
                logger.warning(
                    f"Config file not found at {config_file_path}. Fallback to triton version {try_triton_version} and use MoE kernel config from {try_config_file_path}. Performance might be sub-optimal!",
                )
                # If a configuration has been found, return it
                return {int(key): val for key, val in json.load(f).items()}

    # If no optimized configuration is available, we will use the default configuration when down_moe is False
    # When down_moe is True, we will try to use the config for down_moe=False
    if down_moe:
        logger.warning(
            (
                "Using MoE kernel config with down_moe=False. Performance might be sub-optimal! "
                "Config file not found at %s, you can create them with https://github.com/sgl-project/sglang/tree/main/benchmark/kernels/fused_moe_triton"
            ),
            config_file_path,
        )
    else:
        logger.warning(
            (
                "Using default MoE kernel config. Performance might be sub-optimal! "
                "Config file not found at %s, you can create them with https://github.com/sgl-project/sglang/tree/main/benchmark/kernels/fused_moe_triton"
            ),
            config_file_path,
        )
    return None
```
**EN:** This block defines `get_moe_configs` and contains the main logic for this step. Decorators like `functools.lru_cache` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_config_file_name`, `os.environ.get`, `os.path.join`, `os.path.exists`, and `sorted`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `json_file_name`, `config_dir`, `triton_version`, `version_dir`, and `config_file_path` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_configs`，并承载这一阶段的核心逻辑。 像 `functools.lru_cache` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_config_file_name`、`os.environ.get`、`os.path.join`、`os.path.exists` 以及 `sorted`，说明该流程会编排底层辅助函数或计算内核。 像 `json_file_name`、`config_dir`、`triton_version`、`version_dir` 以及 `config_file_path` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 146-210: `get_default_config` getter for default config
```python
def get_default_config(
    M: int,
    E: int,
    N: int,
    K: int,
    topk: int,
    dtype: Optional[str],
    is_marlin: bool,
    block_shape: Optional[List[int]] = None,
) -> Dict[str, int]:
    if get_global_server_args().enable_deterministic_inference:
        config = {
            "BLOCK_SIZE_M": 64,
            "BLOCK_SIZE_N": 64,
            "BLOCK_SIZE_K": 32,
            "GROUP_SIZE_M": 8,
        }
        return config
    if dtype == "fp8_w8a8":
        if block_shape is None:
            config = {
                "BLOCK_SIZE_M": 128,
                "BLOCK_SIZE_N": 256,
                "BLOCK_SIZE_K": 128,
                "GROUP_SIZE_M": 32,
                "num_warps": 8,
                "num_stages": 2 if _is_hip else 4,
            }
            if M <= E:
                config = {
                    "BLOCK_SIZE_M": 64,
                    "BLOCK_SIZE_N": 128,
                    "BLOCK_SIZE_K": 128,
                    "GROUP_SIZE_M": 1,
                    "num_warps": 4,
                    "num_stages": 2 if _is_hip else 4,
                }
        else:
            # Block-wise quant: BLOCK_SIZE_K must be divisible by block_shape[1]
            config = {
                "BLOCK_SIZE_M": 64,
                "BLOCK_SIZE_N": block_shape[0],
                "BLOCK_SIZE_K": block_shape[1],
                "GROUP_SIZE_M": 32,
                "num_warps": 4,
                "num_stages": 2 if _is_hip else 3,
            }
    else:
        config = {
            "BLOCK_SIZE_M": 64,
            "BLOCK_SIZE_N": 64,
            "BLOCK_SIZE_K": 32,
            "GROUP_SIZE_M": 8,
        }
        # A heuristic: fused marlin works faster with this config for small M
        if M <= E or (is_marlin and M <= 32):
            config = {
                "BLOCK_SIZE_M": 16,
                "BLOCK_SIZE_N": 32,
                "BLOCK_SIZE_K": 64,
                "GROUP_SIZE_M": 1,
            }
    return config
```
**EN:** This block defines `get_default_config` and contains the main logic for this step. It mainly invokes `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `config` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_default_config`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 211-278: Function `try_get_optimal_moe_config` and its core logic
```python
def try_get_optimal_moe_config(
    w1_shape: Tuple[int, ...],
    w2_shape: Tuple[int, ...],
    top_k: int,
    dtype: Optional[str],
    M: int,
    is_marlin: bool = False,
    block_shape: Optional[List[int]] = None,
    per_channel_quant: bool = False,
    return_down_config: bool = False,
):
    from sglang.srt.layers.moe.moe_runner.triton_utils import get_config

    down_config = None
    max_block_m = None
    override_config = get_config()
    if override_config:
        config = override_config
    else:
        # First try to load optimal config from the file
        E, _, N = w2_shape
        block_n = block_shape[0] if block_shape else 0
        block_k = block_shape[1] if block_shape else 0
        configs = get_moe_configs(
            E,
            N,
            dtype,
            block_n,
            block_k,
            per_channel_quant=per_channel_quant,
            down_moe=False,
        )

        if configs:
            # If an optimal configuration map has been found, look up the
            # optimal config
            config = configs[min(configs.keys(), key=lambda x: abs(x - M))]
        else:
            # Else use the default config
            config = get_default_config(
                M, E, N, w1_shape[2], top_k, dtype, is_marlin, block_shape
            )
        if return_down_config:
            down_configs = get_moe_configs(
                E,
                N,
                dtype,
                block_n,
                block_k,
                per_channel_quant=per_channel_quant,
                down_moe=True,
            )
            if down_configs:
                down_config = down_configs[
                    min(down_configs.keys(), key=lambda x: abs(x - M))
                ]
                down_config = dict(**down_config)
                max_block_m = max(
                    [cfg["BLOCK_SIZE_M"] for cfg in down_configs.values()]
                )
    if return_down_config:
        assert (
            down_config is None or config["BLOCK_SIZE_M"] == down_config["BLOCK_SIZE_M"]
        )
        return config, (down_config, max_block_m)
    return config
```
**EN:** This block defines `try_get_optimal_moe_config` and contains the main logic for this step. It mainly invokes `get_config`, `get_moe_configs`, `get_default_config`, `min`, and `dict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `down_config`, `max_block_m`, `override_config`, `config`, and `E` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `try_get_optimal_moe_config`，并承载这一阶段的核心逻辑。 它主要调用 `get_config`、`get_moe_configs`、`get_default_config`、`min` 以及 `dict`，说明该流程会编排底层辅助函数或计算内核。 像 `down_config`、`max_block_m`、`override_config`、`config` 以及 `E` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 279-298: `get_config_dtype_str` getter for config dtype str
```python
def get_config_dtype_str(
    dtype: torch.dtype,
    use_int8_w8a16: Optional[bool] = False,
    use_int4_w4a16: Optional[bool] = False,
    use_fp8_w8a8: Optional[bool] = False,
    use_int8_w8a8: Optional[bool] = False,
):
    if use_fp8_w8a8:
        return "fp8_w8a8"
    elif use_int8_w8a8:
        return "int8_w8a8"
    elif use_int4_w4a16:
        return "int4_w4a16"
    elif use_int8_w8a16:
        return "int8_w8a16"
    elif dtype == torch.float:
        # avoiding cases where kernel fails when float32 MoE
        # use fp16/bfloat16 configs
        return "float32"
    return None
```
**EN:** This block defines `get_config_dtype_str` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_config_dtype_str`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `get_config_file_name`, `get_moe_configs`, `get_default_config`, `try_get_optimal_moe_config`, and `get_config_dtype_str`. / **主要符号**：核心入口包括 `get_config_file_name`、`get_moe_configs`、`get_default_config`、`try_get_optimal_moe_config` 以及 `get_config_dtype_str`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `functools`, `json`, `logging`, `os`, `typing.Any`, `typing.Dict`, `typing.List`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`functools`、`json`、`logging`、`os`、`typing.Any`、`typing.Dict`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch` and `triton` / **第三方依赖**：`torch` 和 `triton`
- **Internal SGLang modules**: `sglang.srt.server_args.get_global_server_args`, `sglang.srt.utils.get_device_name`, `sglang.srt.utils.is_hip`, and `sglang.srt.layers.moe.moe_runner.triton_utils.get_config` / **SGLang 内部模块**：`sglang.srt.server_args.get_global_server_args`、`sglang.srt.utils.get_device_name`、`sglang.srt.utils.is_hip` 以及 `sglang.srt.layers.moe.moe_runner.triton_utils.get_config`
