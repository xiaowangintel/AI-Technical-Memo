# deepep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/deepep.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `_deepep_precompile_tp_barrier`, `DeepEPPDispatchHooks`, `DeepEPNormalDispatchOutput`, and `DeepEPLLDispatchOutput` and connects them to backend-specific paths such as `CUDA`, `Triton`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `_deepep_precompile_tp_barrier`、`DeepEPPDispatchHooks`、`DeepEPNormalDispatchOutput` 以及 `DeepEPLLDispatchOutput` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-69: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
import os
from contextlib import nullcontext
from dataclasses import dataclass
from typing import TYPE_CHECKING, List, NamedTuple, Optional, Tuple, Union

from sglang.srt.distributed.parallel_state import get_tp_group
from sglang.srt.environ import envs
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.layers import deep_gemm_wrapper
from sglang.srt.layers.dp_attention import get_is_extend_in_batch
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    BaseDispatcherConfig,
    CombineInput,
    CombineInputFormat,
    DispatcherBaseHooks,
    DispatchOutput,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.topk import TopKOutput
from sglang.srt.layers.moe.utils import (
    DeepEPMode,
    DeepEPOutputDtype,
    get_deepep_config,
    get_deepep_output_dtype,
    is_tbo_enabled,
)
from sglang.srt.utils import (
    get_bool_env_var,
    is_blackwell,
    is_hip,
    is_npu,
    load_json_config,
)

_is_npu = is_npu()

if TYPE_CHECKING:
    from sglang.srt.batch_overlap.single_batch_overlap import CombineOverlapArgs

try:
    if _is_npu and envs.SGLANG_ZBAL_LOCAL_MEM_SIZE.get() > 0:
        from zbal.zbal.deepep_adaptor import Config
        from zbal.zbal_buffer import Buffer
    else:
        from deep_ep import Buffer, Config

    if not _is_npu:
        from sglang.srt.layers.quantization.fp8_kernel import (
            sglang_per_token_group_quant_fp8,
        )

    use_deepep = True
except ImportError:
    use_deepep = False

from enum import Enum, IntEnum, auto

import torch
import torch.distributed as dist

_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and is_hip()

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `os`, `contextlib.nullcontext`, `dataclasses.dataclass`, and `typing.TYPE_CHECKING`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_npu`, `use_deepep`, `_use_aiter`, and `logger` capture configuration, cached handles, or feature flags. Control structures like `If` and `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`os`、`contextlib.nullcontext`、`dataclasses.dataclass` 以及 `typing.TYPE_CHECKING`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_npu`、`use_deepep`、`_use_aiter` 以及 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 和 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 70-78: Internal helper `_deepep_precompile_tp_barrier`
```python
def _deepep_precompile_tp_barrier() -> None:
    # DeepEP's all-to-all operation has a much shorter timeout compared to torch.distributed,
    # so if different ranks compile at different speeds, it may quickly trigger a timeout.
    # To avoid this, we use torch.distributed's barrier during the compile stage.
    # We apply this barrier only in the compile stage to prevent extra all-reduce overhead at runtime.
    if envs.SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE.get():
        get_tp_group().barrier()
```
**EN:** This block defines `_deepep_precompile_tp_barrier` and contains the main logic for this step. It mainly invokes `envs.SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE.get`, `get_tp_group.barrier`, and `get_tp_group`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `_deepep_precompile_tp_barrier`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE.get`、`get_tp_group.barrier` 以及 `get_tp_group`，说明该流程会编排底层辅助函数或计算内核。

### Lines 79-79: Class `DeepEPPDispatchHooks` declaration and shared state
```python
class DeepEPPDispatchHooks(DispatcherBaseHooks):
```
**EN:** This block introduces class `DeepEPPDispatchHooks` and the state shared by its methods. It inherits from `DispatcherBaseHooks`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepEPPDispatchHooks`，并定义其方法共享的状态。 它继承自 `DispatcherBaseHooks`，说明了它在 SRT 层栈中的接入方式。

### Lines 80-84: `DeepEPPDispatchHooks.__call__` callable execution entry
```python
    def __call__(self, dispatcher: BaseDispatcher):
        for hook_fun in self.hook_dict.values():
            hook_fun(dispatcher)
```
**EN:** This block defines `DeepEPPDispatchHooks.__call__` and contains the main logic for this step. It mainly invokes `self.hook_dict.values` and `hook_fun`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPPDispatchHooks.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.hook_dict.values` 和 `hook_fun`，说明该流程会编排底层辅助函数或计算内核。

### Lines 85-93: Class `DeepEPNormalDispatchOutput` declaration and shared state
```python
class DeepEPNormalDispatchOutput(NamedTuple):
    """DeepEP normal dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
    num_recv_tokens_per_expert: List[int]
```
**EN:** This block introduces class `DeepEPNormalDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: DeepEP normal dispatch output.
**CN:** 该代码块引入类 `DeepEPNormalDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 94-98: Function `DeepEPNormalDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.DEEPEP_NORMAL
```
**EN:** This block defines `DeepEPNormalDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPNormalDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 99-108: Class `DeepEPLLDispatchOutput` declaration and shared state
```python
class DeepEPLLDispatchOutput(NamedTuple):
    """DeepEP low latency dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
    masked_m: torch.Tensor
    expected_m: int
```
**EN:** This block introduces class `DeepEPLLDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: DeepEP low latency dispatch output.
**CN:** 该代码块引入类 `DeepEPLLDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 109-113: Function `DeepEPLLDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.DEEPEP_LL
```
**EN:** This block defines `DeepEPLLDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPLLDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 114-117: Module-level helper logic
```python
assert isinstance(DeepEPNormalDispatchOutput, DispatchOutput)
assert isinstance(DeepEPLLDispatchOutput, DispatchOutput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 118-124: Class `DeepEPNormalCombineInput` declaration and shared state
```python
class DeepEPNormalCombineInput(NamedTuple):
    """DeepEP normal combine input."""

    hidden_states: torch.Tensor
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
```
**EN:** This block introduces class `DeepEPNormalCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: DeepEP normal combine input.
**CN:** 该代码块引入类 `DeepEPNormalCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 125-129: Function `DeepEPNormalCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.DEEPEP_NORMAL
```
**EN:** This block defines `DeepEPNormalCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPNormalCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 130-136: Class `DeepEPLLCombineInput` declaration and shared state
```python
class DeepEPLLCombineInput(NamedTuple):
    """DeepEP low latency combine input."""

    hidden_states: torch.Tensor
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
```
**EN:** This block introduces class `DeepEPLLCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: DeepEP low latency combine input.
**CN:** 该代码块引入类 `DeepEPLLCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 137-141: Function `DeepEPLLCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.DEEPEP_LL
```
**EN:** This block defines `DeepEPLLCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPLLCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 142-145: Module-level helper logic
```python
assert isinstance(DeepEPNormalCombineInput, CombineInput)
assert isinstance(DeepEPLLCombineInput, CombineInput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 146-150: Class `DeepEPDispatchMode` declaration and shared state
```python
class DeepEPDispatchMode(IntEnum):
    NORMAL = auto()
    LOW_LATENCY = auto()
```
**EN:** This block introduces class `DeepEPDispatchMode` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepEPDispatchMode`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 151-157: Class `DeepEPBuffer` declaration and shared state
```python
class DeepEPBuffer:
    _buffer = None
    _dispatch_mode: Optional[DeepEPDispatchMode] = None
    _hidden_size: Optional[int] = None
    _num_max_dispatch_tokens_per_rank: Optional[int] = None
    _num_experts: Optional[int] = None
```
**EN:** This block introduces class `DeepEPBuffer` and the state shared by its methods.
**CN:** 该代码块引入类 `DeepEPBuffer`，并定义其方法共享的状态。

### Lines 158-246: `DeepEPBuffer.get_deepep_buffer` getter for deepep buffer
```python
    @classmethod
    def get_deepep_buffer(
        cls,
        group: dist.ProcessGroup,
        hidden_size: int,
        param_bytes: int,
        deepep_mode: DeepEPMode,
        num_max_dispatch_tokens_per_rank: int = -1,
        num_experts: int = -1,
    ):
        if cls._buffer is not None:
            return cls._buffer

        cls._hidden_size = hidden_size
        cls._num_max_dispatch_tokens_per_rank = num_max_dispatch_tokens_per_rank
        cls._num_experts = num_experts

        num_nvl_bytes, num_rdma_bytes = 0, 0
        if deepep_mode.enable_normal():
            hidden_bytes = hidden_size * param_bytes
            for config in (
                DeepEPConfig.get_instance().normal_dispatch_config
                or Buffer.get_dispatch_config(group.size()),
                DeepEPConfig.get_instance().normal_combine_config
                or Buffer.get_combine_config(group.size()),
            ):
                num_nvl_bytes = max(
                    config.get_nvl_buffer_size_hint(hidden_bytes, group.size()),
                    num_nvl_bytes,
                )
                num_rdma_bytes = max(
                    config.get_rdma_buffer_size_hint(hidden_bytes, group.size()),
                    num_rdma_bytes,
                )
        if deepep_mode.enable_low_latency():
            assert num_max_dispatch_tokens_per_rank != -1
            assert num_experts != -1 and num_experts % group.size() == 0
            num_rdma_bytes = max(
                Buffer.get_low_latency_rdma_size_hint(
                    num_max_dispatch_tokens_per_rank,
                    hidden_size,
                    group.size(),
                    num_experts,
                ),
                num_rdma_bytes,
            )

        # We should calculate num_qps_per_rank consistently with DeepEP's test script logic:
        if deepep_mode == DeepEPMode.NORMAL:
            # refer: https://github.com/deepseek-ai/DeepEP/blob/main/tests/test_internode.py#L235
            num_qps_per_rank = DeepEPConfig.get_instance().num_sms
        elif deepep_mode == DeepEPMode.LOW_LATENCY:
            # refer: https://github.com/deepseek-ai/DeepEP/blob/main/tests/test_low_latency.py#L176
            num_qps_per_rank = num_experts // group.size()
        elif deepep_mode == DeepEPMode.AUTO:
            # low-latency and normal mode all need run
            # refer: https://github.com/deepseek-ai/DeepEP/blob/main/tests/test_internode.py#L235
            num_qps_per_rank = max(
                DeepEPConfig.get_instance().num_sms, num_experts // group.size()
            )
        else:
            raise NotImplementedError

        if not _is_npu:
            total_num_sms = torch.cuda.get_device_properties(
                device="cuda"
            ).multi_processor_count
            if (
                (deepep_mode != DeepEPMode.LOW_LATENCY)
                and not is_tbo_enabled()
                and (DeepEPConfig.get_instance().num_sms < total_num_sms // 2)
            ):
                logger.warning(
                    f"Only use {DeepEPConfig.get_instance().num_sms} SMs for DeepEP communication. "
                    f"This may result in highly suboptimal performance. "
                    f"Consider using --deepep-config to change the behavior."
                )

        cls._buffer = Buffer(
            group,
            num_nvl_bytes,
            num_rdma_bytes,
            low_latency_mode=deepep_mode.enable_low_latency(),
            num_qps_per_rank=num_qps_per_rank,
            # TODO can be false when unneeded
            allow_mnnvl=True,
        )
        return cls._buffer
```
**EN:** This block defines `DeepEPBuffer.get_deepep_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `deepep_mode.enable_normal`, `deepep_mode.enable_low_latency`, `Buffer`, `max`, and `Buffer.get_low_latency_rdma_size_hint`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_nvl_bytes`, `num_rdma_bytes`, `hidden_bytes`, `num_qps_per_rank`, and `total_num_sms` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPBuffer.get_deepep_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `deepep_mode.enable_normal`、`deepep_mode.enable_low_latency`、`Buffer`、`max` 以及 `Buffer.get_low_latency_rdma_size_hint`，说明该流程会编排底层辅助函数或计算内核。 像 `num_nvl_bytes`、`num_rdma_bytes`、`hidden_bytes`、`num_qps_per_rank` 以及 `total_num_sms` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 247-256: Function `DeepEPBuffer.clean_buffer` and its core logic
```python
    @classmethod
    def clean_buffer(cls):
        if not cls._buffer.low_latency_mode:
            return
        cls._buffer.clean_low_latency_buffer(
            cls._num_max_dispatch_tokens_per_rank,
            cls._hidden_size,
            cls._num_experts,
        )
```
**EN:** This block defines `DeepEPBuffer.clean_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._buffer.clean_low_latency_buffer`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPBuffer.clean_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._buffer.clean_low_latency_buffer`，说明该流程会编排底层辅助函数或计算内核。

### Lines 257-260: `DeepEPBuffer.set_dispatch_mode_as_normal` setter for dispatch mode as normal
```python
    @classmethod
    def set_dispatch_mode_as_normal(cls):
        cls._dispatch_mode = DeepEPDispatchMode.NORMAL
```
**EN:** This block defines `DeepEPBuffer.set_dispatch_mode_as_normal` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `DeepEPBuffer.set_dispatch_mode_as_normal`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 261-266: `DeepEPBuffer.set_dispatch_mode_as_low_latency` setter for dispatch mode as low latency
```python
    @classmethod
    def set_dispatch_mode_as_low_latency(cls):
        if cls._dispatch_mode == DeepEPDispatchMode.NORMAL:
            cls.clean_buffer()
        cls._dispatch_mode = DeepEPDispatchMode.LOW_LATENCY
```
**EN:** This block defines `DeepEPBuffer.set_dispatch_mode_as_low_latency` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls.clean_buffer`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPBuffer.set_dispatch_mode_as_low_latency`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls.clean_buffer`，说明该流程会编排底层辅助函数或计算内核。

### Lines 267-276: `DeepEPBuffer.set_dispatch_mode` setter for dispatch mode
```python
    @classmethod
    def set_dispatch_mode(cls, mode: DeepEPMode):
        if mode.is_low_latency():
            cls.set_dispatch_mode_as_low_latency()
        elif mode.is_normal():
            cls.set_dispatch_mode_as_normal()
        else:
            raise Exception("unsupported mode")
```
**EN:** This block defines `DeepEPBuffer.set_dispatch_mode` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `mode.is_low_latency`, `cls.set_dispatch_mode_as_low_latency`, `mode.is_normal`, `cls.set_dispatch_mode_as_normal`, and `Exception`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPBuffer.set_dispatch_mode`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `mode.is_low_latency`、`cls.set_dispatch_mode_as_low_latency`、`mode.is_normal`、`cls.set_dispatch_mode_as_normal` 以及 `Exception`，说明该流程会编排底层辅助函数或计算内核。

### Lines 277-279: Class `DeepEPConfig` declaration and shared state
```python
class DeepEPConfig(BaseDispatcherConfig):
    _instance = None
```
**EN:** This block introduces class `DeepEPConfig` and the state shared by its methods. It inherits from `BaseDispatcherConfig`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepEPConfig`，并定义其方法共享的状态。 它继承自 `BaseDispatcherConfig`，说明了它在 SRT 层栈中的接入方式。

### Lines 280-298: `DeepEPConfig` initialization and state setup
```python
    def __init__(self):
        config_str = get_deepep_config()
        if config_str:
            config_parsed = load_json_config(config_str)
            if torch.distributed.get_rank() == 0:
                logger.info(f"Use DeepEP Config: {config_parsed}")
            config_dispatch = config_parsed["normal_dispatch"]
            config_combine = config_parsed["normal_combine"]

            self.normal_dispatch_config = Config(**config_dispatch)
            self.normal_combine_config = Config(**config_combine)

            assert config_dispatch["num_sms"] == config_combine["num_sms"]
            self.num_sms = config_dispatch["num_sms"]
        else:
            self.normal_dispatch_config = None
            self.normal_combine_config = None
            self.num_sms = Buffer.num_sms
```
**EN:** This block defines `DeepEPConfig.__init__` and contains the main logic for this step. It mainly invokes `get_deepep_config`, `load_json_config`, `Config`, `torch.distributed.get_rank`, and `logger.info`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `config_str`, `config_parsed`, `config_dispatch`, `config_combine`, and `self.normal_dispatch_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPConfig.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `get_deepep_config`、`load_json_config`、`Config`、`torch.distributed.get_rank` 以及 `logger.info`，说明该流程会编排底层辅助函数或计算内核。 像 `config_str`、`config_parsed`、`config_dispatch`、`config_combine` 以及 `self.normal_dispatch_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 299-305: `DeepEPConfig.get_instance` getter for instance
```python
    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = DeepEPConfig()
        return cls._instance
```
**EN:** This block defines `DeepEPConfig.get_instance` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `DeepEPConfig`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPConfig.get_instance`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `DeepEPConfig`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 306-306: Class `_DeepEPDispatcherImplBase` declaration and shared state
```python
class _DeepEPDispatcherImplBase:
```
**EN:** This block introduces class `_DeepEPDispatcherImplBase` and the state shared by its methods.
**CN:** 该代码块引入类 `_DeepEPDispatcherImplBase`，并定义其方法共享的状态。

### Lines 307-350: `_DeepEPDispatcherImplBase` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        permute_fusion: bool,
        num_experts: int,
        num_local_experts: int,
        hidden_size: int,
        params_dtype: torch.dtype,
        deepep_mode: DeepEPMode,
    ):
        if not use_deepep:
            raise ImportError(
                "DeepEP is not installed. Please install DeepEP package from "
                "https://github.com/deepseek-ai/deepep."
            )

        self.group = group
        self.router_topk = router_topk
        self.permute_fusion = permute_fusion
        self.num_experts = num_experts
        self.num_local_experts = num_local_experts
        self.hidden_size = hidden_size
        self.params_dtype = params_dtype
        self.deepep_mode = deepep_mode

        self.params_bytes = 2
        # A large value will lead to large memory occupation, thus users should change it accordingly
        self.num_max_dispatch_tokens_per_rank = (
            envs.SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get()
        )
        # DeepEP internode_ll dispatch uses FINISHED_SUM_TAG=1024
        # and the logic requires num-tokens-sent-from-one-rank-to-another-rank less than it
        assert self.num_max_dispatch_tokens_per_rank <= 1024

        self.handle = None

        self.quant_config: Optional[dict] = None

        self.overlap_args: Optional[CombineOverlapArgs] = None
        self.meta_overlap_args: Optional[dict] = None

        self.set_deepep_dispatcher_dtype()
```
**EN:** This block defines `_DeepEPDispatcherImplBase.__init__` and contains the main logic for this step. It mainly invokes `envs.SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get`, `self.set_deepep_dispatcher_dtype`, and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.group`, `self.router_topk`, `self.permute_fusion`, `self.num_experts`, and `self.num_local_experts` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get`、`self.set_deepep_dispatcher_dtype` 以及 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `self.group`、`self.router_topk`、`self.permute_fusion`、`self.num_experts` 以及 `self.num_local_experts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 351-357: `_DeepEPDispatcherImplBase.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        raise NotImplementedError
```
**EN:** This block defines `_DeepEPDispatcherImplBase.dispatch_a` and contains the main logic for this step.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.dispatch_a`，并承载这一阶段的核心逻辑。

### Lines 358-360: `_DeepEPDispatcherImplBase.dispatch_b` step for b
```python
    def dispatch_b(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `_DeepEPDispatcherImplBase.dispatch_b` and contains the main logic for this step.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.dispatch_b`，并承载这一阶段的核心逻辑。

### Lines 361-368: Function `_DeepEPDispatcherImplBase.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        raise NotImplementedError
```
**EN:** This block defines `_DeepEPDispatcherImplBase.combine_a` and contains the main logic for this step.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.combine_a`，并承载这一阶段的核心逻辑。

### Lines 369-371: Function `_DeepEPDispatcherImplBase.combine_b` and its core logic
```python
    def combine_b(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `_DeepEPDispatcherImplBase.combine_b` and contains the main logic for this step.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.combine_b`，并承载这一阶段的核心逻辑。

### Lines 372-374: Internal helper `_DeepEPDispatcherImplBase._get_buffer`
```python
    def _get_buffer(self):
        raise NotImplementedError
```
**EN:** This block defines `_DeepEPDispatcherImplBase._get_buffer` and contains the main logic for this step.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase._get_buffer`，并承载这一阶段的核心逻辑。

### Lines 375-378: `_DeepEPDispatcherImplBase.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict) -> None:
        self.quant_config = quant_config
        self.set_deepep_dispatcher_dtype()
```
**EN:** This block defines `_DeepEPDispatcherImplBase.set_quant_config` and contains the main logic for this step. It mainly invokes `self.set_deepep_dispatcher_dtype`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.quant_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `self.set_deepep_dispatcher_dtype`，说明该流程会编排底层辅助函数或计算内核。 像 `self.quant_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 379-416: `_DeepEPDispatcherImplBase.set_deepep_dispatcher_dtype` setter for deepep dispatcher dtype
```python
    def set_deepep_dispatcher_dtype(self) -> None:
        self.deepep_output_dtype = get_deepep_output_dtype(self)

        # Configuration mapping for each dtype
        config_map = {
            DeepEPOutputDtype.BF16: {
                "use_fp8": False,
                "use_nvfp4": False,
            },
            DeepEPOutputDtype.FP8: {
                "use_fp8": True,
                "use_nvfp4": False,
            },
            # Needed for Ascend A2/A3 NPU case,
            # despite the use_fp8 flag,
            # quantization will be performed in int8
            DeepEPOutputDtype.INT8: {
                "use_fp8": True,
                "use_nvfp4": False,
            },
            DeepEPOutputDtype.NVFP4: {
                "use_fp8": False,
                "use_nvfp4": True,
            },
        }

        # Validate and apply hardware-specific adjustments
        self._validate_and_adjust_dtype()

        # Apply configuration
        config = config_map[self.deepep_output_dtype]
        self.use_fp8 = config["use_fp8"]
        self.use_nvfp4 = config["use_nvfp4"]

        # Handle environment variables
        if _is_npu:
            self._update_int8_quant_env()
```
**EN:** This block defines `_DeepEPDispatcherImplBase.set_deepep_dispatcher_dtype` and contains the main logic for this step. It mainly invokes `get_deepep_output_dtype`, `self._validate_and_adjust_dtype`, and `self._update_int8_quant_env`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_output_dtype`, `config_map`, `config`, `self.use_fp8`, and `self.use_nvfp4` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.set_deepep_dispatcher_dtype`，并承载这一阶段的核心逻辑。 它主要调用 `get_deepep_output_dtype`、`self._validate_and_adjust_dtype` 以及 `self._update_int8_quant_env`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_output_dtype`、`config_map`、`config`、`self.use_fp8` 以及 `self.use_nvfp4` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 417-438: Internal helper `_DeepEPDispatcherImplBase._validate_and_adjust_dtype`
```python
    def _validate_and_adjust_dtype(self) -> None:
        """Validate dtype against hardware and adjust if necessary."""
        if _is_npu:
            if self.deepep_output_dtype == DeepEPOutputDtype.FP8:
                logger.warning_once(
                    "Ascend A2/A3 NPU does not support fp8 "
                    "deepep_dispatcher_output_dtype, switching to int8..."
                )
                self.deepep_output_dtype = DeepEPOutputDtype.INT8
            elif self.deepep_output_dtype == DeepEPOutputDtype.NVFP4:
                raise RuntimeError(
                    "Ascend A2/A3 NPU does not support nvfp4 deepep_dispatcher_output_dtype."
                )
        else:
            if self.deepep_output_dtype == DeepEPOutputDtype.INT8:
                logger.warning_once(
                    "GPU does not support int8 "
                    "deepep_dispatcher_output_dtype, switching to fp8..."
                )
                self.deepep_output_dtype = DeepEPOutputDtype.FP8
            # NVFP4 is supported on GPU, no adjustment needed
```
**EN:** This block defines `_DeepEPDispatcherImplBase._validate_and_adjust_dtype` and contains the main logic for this step. It mainly invokes `logger.warning_once` and `RuntimeError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_output_dtype` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase._validate_and_adjust_dtype`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning_once` 和 `RuntimeError`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_output_dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 439-445: Internal helper `_DeepEPDispatcherImplBase._update_int8_quant_env`
```python
    def _update_int8_quant_env(self) -> None:
        """Update the DEEP_NORMAL_MODE_USE_INT8_QUANT environment variable."""
        if self.use_fp8:
            os.environ["DEEP_NORMAL_MODE_USE_INT8_QUANT"] = "1"
        else:
            os.environ["DEEP_NORMAL_MODE_USE_INT8_QUANT"] = "0"
```
**EN:** This block defines `_DeepEPDispatcherImplBase._update_int8_quant_env` and contains the main logic for this step.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase._update_int8_quant_env`，并承载这一阶段的核心逻辑。

### Lines 446-451: `_DeepEPDispatcherImplBase.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(
        self, combine_overlap_args: CombineOverlapArgs, meta_overlap_args: dict
    ) -> None:
        self.overlap_args = combine_overlap_args
        self.meta_overlap_args = meta_overlap_args
```
**EN:** This block defines `_DeepEPDispatcherImplBase.set_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.set_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 452-456: Function `_DeepEPDispatcherImplBase.clear_overlap_args` and its core logic
```python
    def clear_overlap_args(self) -> None:
        self.overlap_args = None
        self.meta_overlap_args = None
```
**EN:** This block defines `_DeepEPDispatcherImplBase.clear_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplBase.clear_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 457-457: Class `_DeepEPDispatcherImplNormal` declaration and shared state
```python
class _DeepEPDispatcherImplNormal(_DeepEPDispatcherImplBase):
```
**EN:** This block introduces class `_DeepEPDispatcherImplNormal` and the state shared by its methods. It inherits from `_DeepEPDispatcherImplBase`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_DeepEPDispatcherImplNormal`，并定义其方法共享的状态。 它继承自 `_DeepEPDispatcherImplBase`，说明了它在 SRT 层栈中的接入方式。

### Lines 458-464: `_DeepEPDispatcherImplNormal` initialization and state setup
```python
    def __init__(self, async_finish: bool, **kwargs):
        super().__init__(**kwargs)

        self.async_finish = async_finish
        self.src2dst = None
        self.quant_config = {}
```
**EN:** This block defines `_DeepEPDispatcherImplNormal.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.async_finish`, `self.src2dst`, and `self.quant_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.async_finish`、`self.src2dst` 以及 `self.quant_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 465-483: `_DeepEPDispatcherImplNormal.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids
        topk_ids = topk_ids.to(torch.int64)
        if deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM and self.use_fp8:
            # TODO hard code 128 block quant,use fp8 communication
            hidden_states = sglang_per_token_group_quant_fp8(
                hidden_states,
                128,
                column_major_scales=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                scale_tma_aligned=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
            )
        previous_event = Buffer.capture() if self.async_finish else None
        return hidden_states, topk_ids, topk_weights, previous_event
```
**EN:** This block defines `_DeepEPDispatcherImplNormal.dispatch_a` and contains the main logic for this step. It mainly invokes `topk_ids.to`, `sglang_per_token_group_quant_fp8`, and `Buffer.capture`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weights`, `topk_ids`, `previous_event`, and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `topk_ids.to`、`sglang_per_token_group_quant_fp8` 以及 `Buffer.capture`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weights`、`topk_ids`、`previous_event` 以及 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 484-506: `_DeepEPDispatcherImplNormal.dispatch_b` step for b
```python
    def dispatch_b(self, hidden_states, topk_ids, topk_weights, previous_event):
        (
            hidden_states,
            topk_ids,
            topk_weights,
            num_recv_tokens_per_expert,
            event,
        ) = self._dispatch_core(hidden_states, topk_ids, topk_weights, previous_event)
        event.current_stream_wait() if self.async_finish else ()

        if isinstance(hidden_states, tuple):
            hidden_states, hidden_states_scale = hidden_states
        else:
            hidden_states_scale = None

        return DeepEPNormalDispatchOutput(
            hidden_states,
            hidden_states_scale,
            topk_ids,
            topk_weights,
            num_recv_tokens_per_expert,
        )
```
**EN:** This block defines `_DeepEPDispatcherImplNormal.dispatch_b` and contains the main logic for this step. It mainly invokes `self._dispatch_core`, `isinstance`, `DeepEPNormalDispatchOutput`, and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_ids`, `topk_weights`, `num_recv_tokens_per_expert`, and `event` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._dispatch_core`、`isinstance`、`DeepEPNormalDispatchOutput` 以及 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_ids`、`topk_weights`、`num_recv_tokens_per_expert` 以及 `event` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 507-568: Internal helper `_DeepEPDispatcherImplNormal._dispatch_core`
```python
    def _dispatch_core(
        self,
        x: Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]],
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
        previous_event,
    ):
        buffer = self._get_buffer()
        (
            num_tokens_per_rank,
            num_tokens_per_rdma_rank,
            num_tokens_per_expert,
            is_token_in_rank,
            previous_event,
        ) = buffer.get_dispatch_layout(
            topk_ids,
            self.num_experts,
            previous_event=previous_event,
            async_finish=self.async_finish,
            allocate_on_comm_stream=previous_event is not None,
        )
        # FIXME: `handle` should be transmitted with tokens from dispatch to combine.
        # However, doing this would incur an unknown synchronization error, but keeping
        # `handle` as a member variable works.

        _deepep_precompile_tp_barrier()
        (
            recv_x,
            recv_topk_ids,
            recv_topk_weights,
            num_recv_tokens_per_expert,
            self.handle,
            event,
        ) = buffer.dispatch(
            x,
            topk_idx=topk_ids,
            topk_weights=topk_weights,
            num_tokens_per_rank=num_tokens_per_rank,
            num_tokens_per_rdma_rank=num_tokens_per_rdma_rank,
            is_token_in_rank=is_token_in_rank,
            num_tokens_per_expert=num_tokens_per_expert,
            previous_event=previous_event,
            async_finish=self.async_finish,
            allocate_on_comm_stream=(previous_event is not None) and self.async_finish,
            expert_alignment=128 if deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM else 1,
            config=DeepEPConfig.get_instance().normal_dispatch_config,
        )
        get_global_expert_distribution_recorder().on_deepep_dispatch_normal(
            num_recv_tokens_per_expert,
            num_tokens_per_rank=num_tokens_per_rank,
            num_tokens_per_rdma_rank=num_tokens_per_rdma_rank,
            num_tokens_per_expert=num_tokens_per_expert,
        )

        return (
            recv_x,
            recv_topk_ids,
            recv_topk_weights,
            num_recv_tokens_per_expert,
            event,
        )
```
**EN:** This block defines `_DeepEPDispatcherImplNormal._dispatch_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `buffer.get_dispatch_layout`, `_deepep_precompile_tp_barrier`, `buffer.dispatch`, and `get_global_expert_distribution_recorder.on_deepep_dispatch_normal`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`, `num_tokens_per_expert`, and `is_token_in_rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal._dispatch_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`buffer.get_dispatch_layout`、`_deepep_precompile_tp_barrier`、`buffer.dispatch` 以及 `get_global_expert_distribution_recorder.on_deepep_dispatch_normal`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`num_tokens_per_rank`、`num_tokens_per_rdma_rank`、`num_tokens_per_expert` 以及 `is_token_in_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 569-583: Function `_DeepEPDispatcherImplNormal.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):

        if deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM or _use_aiter or _is_npu:
            output = hidden_states
        else:
            raise NotImplementedError()  # triton runner was supported but it's temporarily disabled

        previous_event = Buffer.capture() if self.async_finish else None
        return output, previous_event
```
**EN:** This block defines `_DeepEPDispatcherImplNormal.combine_a` and contains the main logic for this step. It mainly invokes `NotImplementedError` and `Buffer.capture`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `previous_event` and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `NotImplementedError` 和 `Buffer.capture`，说明该流程会编排底层辅助函数或计算内核。 像 `previous_event` 和 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 584-590: Function `_DeepEPDispatcherImplNormal.combine_b` and its core logic
```python
    def combine_b(self, output, previous_event):
        hidden_states, event = self._combine_core(output, previous_event)
        event.current_stream_wait() if self.async_finish else ()
        self.handle = None
        self.src2dst = None
        return hidden_states
```
**EN:** This block defines `_DeepEPDispatcherImplNormal.combine_b` and contains the main logic for this step. It mainly invokes `self._combine_core` and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `event`, `self.handle`, and `self.src2dst` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._combine_core` 和 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`event`、`self.handle` 以及 `self.src2dst` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 591-603: Internal helper `_DeepEPDispatcherImplNormal._combine_core`
```python
    def _combine_core(self, x: torch.Tensor, previous_event):
        buffer = self._get_buffer()
        _deepep_precompile_tp_barrier()
        combined_x, _, event = buffer.combine(
            x,
            self.handle,
            async_finish=self.async_finish,
            previous_event=previous_event,
            allocate_on_comm_stream=previous_event is not None,
            config=DeepEPConfig.get_instance().normal_combine_config,
        )
        return combined_x, event
```
**EN:** This block defines `_DeepEPDispatcherImplNormal._combine_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `_deepep_precompile_tp_barrier`, `buffer.combine`, and `DeepEPConfig.get_instance`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `combined_x`, `_`, and `event` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal._combine_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`_deepep_precompile_tp_barrier`、`buffer.combine` 以及 `DeepEPConfig.get_instance`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`combined_x`、`_` 以及 `event` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 604-616: Internal helper `_DeepEPDispatcherImplNormal._get_buffer`
```python
    def _get_buffer(self):
        DeepEPBuffer.set_dispatch_mode_as_normal()

        return DeepEPBuffer.get_deepep_buffer(
            self.group,
            self.hidden_size,
            self.params_bytes,
            self.deepep_mode,
            self.num_max_dispatch_tokens_per_rank,
            self.num_experts,
        )
```
**EN:** This block defines `_DeepEPDispatcherImplNormal._get_buffer` and contains the main logic for this step. It mainly invokes `DeepEPBuffer.set_dispatch_mode_as_normal` and `DeepEPBuffer.get_deepep_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplNormal._get_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `DeepEPBuffer.set_dispatch_mode_as_normal` 和 `DeepEPBuffer.get_deepep_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 617-617: Class `_DeepEPDispatcherImplLowLatency` declaration and shared state
```python
class _DeepEPDispatcherImplLowLatency(_DeepEPDispatcherImplBase):
```
**EN:** This block introduces class `_DeepEPDispatcherImplLowLatency` and the state shared by its methods. It inherits from `_DeepEPDispatcherImplBase`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_DeepEPDispatcherImplLowLatency`，并定义其方法共享的状态。 它继承自 `_DeepEPDispatcherImplBase`，说明了它在 SRT 层栈中的接入方式。

### Lines 618-628: `_DeepEPDispatcherImplLowLatency` initialization and state setup
```python
    def __init__(self, return_recv_hook: bool, **kwargs):
        super().__init__(**kwargs)

        """
        num_max_dispatch_tokens_per_rank: the actual batch size in the decoding engine should be less than 256
        https://github.com/deepseek-ai/DeepEP?tab=readme-ov-file#example-use-in-inference-decoding
        """
        self.return_recv_hook = return_recv_hook
        self.device_module = torch.get_device_module()
        self.quant_config = {}
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `torch.get_device_module`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.return_recv_hook`, `self.device_module`, and `self.quant_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `torch.get_device_module`，说明该流程会编排底层辅助函数或计算内核。 像 `self.return_recv_hook`、`self.device_module` 以及 `self.quant_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 629-654: `_DeepEPDispatcherImplLowLatency.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        buffer = self._get_buffer()
        topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids
        topk_ids = topk_ids.to(torch.int64)
        expected_m = (
            hidden_states.shape[0] * buffer.group_size * topk_ids.shape[1]
            + self.num_experts
        ) // self.num_experts
        hidden_states, masked_m, event, hook = self._dispatch_core(
            hidden_states,
            topk_ids,
        )
        return (
            hidden_states,
            topk_ids,
            topk_weights,
            masked_m,
            expected_m,
            event,
            hook,
        )
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency.dispatch_a` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `topk_ids.to`, and `self._dispatch_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `topk_weights`, `topk_ids`, `expected_m`, and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`topk_ids.to` 以及 `self._dispatch_core`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`topk_weights`、`topk_ids`、`expected_m` 以及 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 655-685: `_DeepEPDispatcherImplLowLatency.dispatch_b` step for b
```python
    def dispatch_b(
        self,
        hidden_states,
        topk_ids,
        topk_weights,
        masked_m,
        expected_m,
        event,
        hook,
    ):
        hook() if self.return_recv_hook else event.current_stream_wait()

        get_global_expert_distribution_recorder().on_deepep_dispatch_low_latency(
            masked_m
        )

        if isinstance(hidden_states, tuple):
            hidden_states, hidden_states_scale = hidden_states
        else:
            hidden_states_scale = None

        deepep_output = DeepEPLLDispatchOutput(
            hidden_states,
            hidden_states_scale,
            topk_ids,
            topk_weights,
            masked_m,
            expected_m,
        )
        return deepep_output
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency.dispatch_b` and contains the main logic for this step. It mainly invokes `get_global_expert_distribution_recorder.on_deepep_dispatch_low_latency`, `isinstance`, `DeepEPLLDispatchOutput`, `hook`, and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `deepep_output`, `hidden_states`, and `hidden_states_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_expert_distribution_recorder.on_deepep_dispatch_low_latency`、`isinstance`、`DeepEPLLDispatchOutput`、`hook` 以及 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 像 `deepep_output`、`hidden_states` 以及 `hidden_states_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 686-728: Internal helper `_DeepEPDispatcherImplLowLatency._dispatch_core`
```python
    def _dispatch_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
    ):
        input_global_scale = self.quant_config.get("input_global_scale", None)

        # round_scale / use_ue8m0 are FP8-DeepGEMM specific; they cause DeepEP
        # to return int32-packed UE8M0 scales that don't feed the flashinfer
        # cutedsl kernel.
        fp8_deepgemm_scale_opts = (
            dict(
                round_scale=deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
                and deep_gemm_wrapper.DEEPGEMM_BLACKWELL,
                use_ue8m0=deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
                and deep_gemm_wrapper.DEEPGEMM_BLACKWELL,
            )
            if self.use_fp8
            else dict()
        )

        buffer = self._get_buffer()
        _deepep_precompile_tp_barrier()
        packed_recv_hidden, self.packed_recv_count, self.handle, event, hook = (
            buffer.low_latency_dispatch(
                hidden_states,
                topk_ids,
                self.num_max_dispatch_tokens_per_rank,
                self.num_experts,
                use_fp8=self.use_fp8,
                **(dict(use_nvfp4=True) if self.use_nvfp4 else dict()),
                **(
                    dict(x_global_scale=input_global_scale)
                    if input_global_scale is not None
                    else dict()
                ),
                async_finish=not self.return_recv_hook,
                return_recv_hook=self.return_recv_hook,
                **fp8_deepgemm_scale_opts,
            )
        )
        return packed_recv_hidden, self.packed_recv_count, event, hook
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency._dispatch_core` and contains the main logic for this step. It mainly invokes `self.quant_config.get`, `self._get_buffer`, `_deepep_precompile_tp_barrier`, `buffer.low_latency_dispatch`, and `dict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `input_global_scale`, `fp8_deepgemm_scale_opts`, `buffer`, `packed_recv_hidden`, and `self.packed_recv_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency._dispatch_core`，并承载这一阶段的核心逻辑。 它主要调用 `self.quant_config.get`、`self._get_buffer`、`_deepep_precompile_tp_barrier`、`buffer.low_latency_dispatch` 以及 `dict`，说明该流程会编排底层辅助函数或计算内核。 像 `input_global_scale`、`fp8_deepgemm_scale_opts`、`buffer`、`packed_recv_hidden` 以及 `self.packed_recv_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 729-741: Function `_DeepEPDispatcherImplLowLatency.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        hidden_states, event, hook = self._combine_core(
            hidden_states,
            topk_ids,
            topk_weights,
        )
        return hidden_states, event, hook
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency.combine_a` and contains the main logic for this step. It mainly invokes `self._combine_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `event`, and `hook` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._combine_core`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`event` 以及 `hook` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 742-753: Function `_DeepEPDispatcherImplLowLatency.combine_b` and its core logic
```python
    def combine_b(self, hidden_states, event, hook):
        overlap_args = self.overlap_args
        if overlap_args is not None:
            overlap_args.stream.wait_stream(self.device_module.current_stream())

        hook() if self.return_recv_hook else event.current_stream_wait()

        if overlap_args is not None:
            self.device_module.current_stream().wait_stream(overlap_args.stream)

        return hidden_states
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency.combine_b` and contains the main logic for this step. It mainly invokes `overlap_args.stream.wait_stream`, `hook`, `event.current_stream_wait`, `self.device_module.current_stream.wait_stream`, and `self.device_module.current_stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `overlap_args` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `overlap_args.stream.wait_stream`、`hook`、`event.current_stream_wait`、`self.device_module.current_stream.wait_stream` 以及 `self.device_module.current_stream`，说明该流程会编排底层辅助函数或计算内核。 像 `overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 754-801: Internal helper `_DeepEPDispatcherImplLowLatency._combine_core`
```python
    def _combine_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        buffer = self._get_buffer()
        overlap_args = self.overlap_args
        meta_overlap_args = self.meta_overlap_args

        ctx = nullcontext()
        if overlap_args is not None:
            overlap_args.stream.wait_event(overlap_args.wait_event)
            ctx = torch.cuda.stream(overlap_args.stream)

            if is_blackwell():
                overlap_args_dict = dict(
                    overlap=overlap_args.overlap,
                    src_signals=overlap_args.signal,
                    src_signal_expect_value=overlap_args.threshold,
                )
            else:
                overlap_args_dict = dict(
                    overlap=overlap_args.overlap,
                    packed_recv_count=self.packed_recv_count,
                    comp_signal=overlap_args.signal,
                    block_m=meta_overlap_args["block_m"],
                    threshold=meta_overlap_args["threshold"],
                    num_sms=overlap_args.num_sms,
                )
        else:
            overlap_args_dict = {}

        with ctx:
            _deepep_precompile_tp_barrier()
            combined_hidden_states, event, hook = buffer.low_latency_combine(
                x=hidden_states,
                topk_idx=topk_ids,
                topk_weights=topk_weights,
                handle=self.handle,
                async_finish=not self.return_recv_hook,
                return_recv_hook=self.return_recv_hook,
                **overlap_args_dict,
            )

        self.packed_recv_count = self.handle = None
        return combined_hidden_states, event, hook
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency._combine_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `nullcontext`, `overlap_args.stream.wait_event`, `torch.cuda.stream`, and `is_blackwell`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `overlap_args`, `meta_overlap_args`, `ctx`, and `self.packed_recv_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency._combine_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`nullcontext`、`overlap_args.stream.wait_event`、`torch.cuda.stream` 以及 `is_blackwell`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`overlap_args`、`meta_overlap_args`、`ctx` 以及 `self.packed_recv_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 802-813: Internal helper `_DeepEPDispatcherImplLowLatency._get_buffer`
```python
    def _get_buffer(self):
        DeepEPBuffer.set_dispatch_mode_as_low_latency()
        return DeepEPBuffer.get_deepep_buffer(
            self.group,
            self.hidden_size,
            self.params_bytes,
            self.deepep_mode,
            self.num_max_dispatch_tokens_per_rank,
            self.num_experts,
        )
```
**EN:** This block defines `_DeepEPDispatcherImplLowLatency._get_buffer` and contains the main logic for this step. It mainly invokes `DeepEPBuffer.set_dispatch_mode_as_low_latency` and `DeepEPBuffer.get_deepep_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_DeepEPDispatcherImplLowLatency._get_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `DeepEPBuffer.set_dispatch_mode_as_low_latency` 和 `DeepEPBuffer.get_deepep_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 814-821: Class `_Stage` declaration and shared state
```python
@dataclass
class _Stage(Enum):
    INITIAL = auto()
    AFTER_DISPATCH_A = auto()
    AFTER_DISPATCH_B = auto()
    AFTER_COMBINE_A = auto()
```
**EN:** This block introduces class `_Stage` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_Stage`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 822-822: Class `DeepEPDispatcher` declaration and shared state
```python
class DeepEPDispatcher(BaseDispatcher):
```
**EN:** This block introduces class `DeepEPDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepEPDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。

### Lines 823-877: `DeepEPDispatcher` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        permute_fusion: bool = False,
        num_experts: int = None,
        num_local_experts: int = None,
        hidden_size: int = None,
        params_dtype: torch.dtype = None,
        deepep_mode: DeepEPMode = DeepEPMode.AUTO,
        async_finish: bool = False,
        return_recv_hook: bool = False,
    ):
        super().__init__()

        self.deepep_mode = deepep_mode

        common_kwargs = dict(
            group=group,
            router_topk=router_topk,
            permute_fusion=permute_fusion,
            num_experts=num_experts,
            num_local_experts=num_local_experts,
            hidden_size=hidden_size,
            params_dtype=params_dtype,
            deepep_mode=deepep_mode,
        )

        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher = _DeepEPDispatcherImplLowLatency(
                return_recv_hook=return_recv_hook,
                **common_kwargs,
            )
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher = _DeepEPDispatcherImplNormal(
                async_finish=async_finish,
                **common_kwargs,
            )

        self._stage = _Stage.INITIAL
        self._deepep_dispatch_hooks = DeepEPPDispatchHooks()

        # DeepEP/Mooncake/Nixl mark invalid topk slots with -1; the AITER
        # pre_permute reroutes them to a sink slot at index num_local_experts,
        # which is masked off here.
        self.expert_mask_gpu = None
        if _use_aiter and num_local_experts is not None:
            expert_mask = torch.zeros(
                num_local_experts + 1,
                device=torch.cuda.current_device(),
                dtype=torch.int,
            )
            expert_mask[:-1] = 1
            self.expert_mask_gpu = expert_mask
```
**EN:** This block defines `DeepEPDispatcher.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `dict`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, and `DeepEPPDispatchHooks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_mode`, `common_kwargs`, `self._stage`, `self._deepep_dispatch_hooks`, and `self.expert_mask_gpu` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`dict`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal` 以及 `DeepEPPDispatchHooks`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_mode`、`common_kwargs`、`self._stage`、`self._deepep_dispatch_hooks` 以及 `self.expert_mask_gpu` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 878-888: Function `DeepEPDispatcher.dispatch` and its core logic
```python
    def dispatch(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ) -> DispatchOutput:
        self.dispatch_a(hidden_states, topk_output)
        if self._deepep_dispatch_hooks is not None:
            self._deepep_dispatch_hooks(self)
        ret = self.dispatch_b()
        return ret
```
**EN:** This block defines `DeepEPDispatcher.dispatch` and contains the main logic for this step. It mainly invokes `self.dispatch_a`, `self.dispatch_b`, and `self._deepep_dispatch_hooks`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPDispatcher.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatch_a`、`self.dispatch_b` 以及 `self._deepep_dispatch_hooks`，说明该流程会编排底层辅助函数或计算内核。 像 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 889-900: `DeepEPDispatcher.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        self._update_stage(_Stage.INITIAL, _Stage.AFTER_DISPATCH_A)
        inner_state = self._get_impl().dispatch_a(
            hidden_states=hidden_states,
            topk_output=topk_output,
        )
        self._dispatch_intermediate_state = inner_state
```
**EN:** This block defines `DeepEPDispatcher.dispatch_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` and `self._dispatch_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPDispatcher.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 和 `self._dispatch_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 901-906: `DeepEPDispatcher.dispatch_b` step for b
```python
    def dispatch_b(self):
        self._update_stage(_Stage.AFTER_DISPATCH_A, _Stage.AFTER_DISPATCH_B)
        inner_state = self._dispatch_intermediate_state
        del self._dispatch_intermediate_state
        return self._get_impl().dispatch_b(*inner_state)
```
**EN:** This block defines `DeepEPDispatcher.dispatch_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPDispatcher.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 907-914: Function `DeepEPDispatcher.combine` and its core logic
```python
    def combine(
        self,
        combine_input: CombineInput,
    ) -> torch.Tensor:
        self.combine_a(combine_input)
        ret = self.combine_b()
        return ret
```
**EN:** This block defines `DeepEPDispatcher.combine` and contains the main logic for this step. It mainly invokes `self.combine_a` and `self.combine_b`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPDispatcher.combine`，并承载这一阶段的核心逻辑。 它主要调用 `self.combine_a` 和 `self.combine_b`，说明该流程会编排底层辅助函数或计算内核。 像 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 915-927: Function `DeepEPDispatcher.combine_a` and its core logic
```python
    def combine_a(
        self,
        combine_input: CombineInput,
    ):
        hidden_states, topk_ids, topk_weights = combine_input
        self._update_stage(_Stage.AFTER_DISPATCH_B, _Stage.AFTER_COMBINE_A)
        inner_state = self._get_impl().combine_a(
            hidden_states=hidden_states,
            topk_ids=topk_ids,
            topk_weights=topk_weights,
        )
        self._combine_intermediate_state = inner_state
```
**EN:** This block defines `DeepEPDispatcher.combine_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_ids`, `topk_weights`, `inner_state`, and `self._combine_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPDispatcher.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_ids`、`topk_weights`、`inner_state` 以及 `self._combine_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 928-933: Function `DeepEPDispatcher.combine_b` and its core logic
```python
    def combine_b(self):
        self._update_stage(_Stage.AFTER_COMBINE_A, _Stage.INITIAL)
        inner_state = self._combine_intermediate_state
        del self._combine_intermediate_state
        return self._get_impl().combine_b(*inner_state)
```
**EN:** This block defines `DeepEPDispatcher.combine_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPDispatcher.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 934-943: Internal helper `DeepEPDispatcher._get_impl`
```python
    def _get_impl(self) -> _DeepEPDispatcherImplBase:
        is_extend_in_batch = get_is_extend_in_batch()
        resolved_deepep_mode = self.deepep_mode.resolve(is_extend_in_batch)
        if resolved_deepep_mode == DeepEPMode.NORMAL:
            return self._normal_dispatcher
        elif resolved_deepep_mode == DeepEPMode.LOW_LATENCY:
            return self._low_latency_dispatcher
        else:
            raise ValueError(f"Invalid deepep_mode: {self.deepep_mode}")
```
**EN:** This block defines `DeepEPDispatcher._get_impl` and contains the main logic for this step. It mainly invokes `get_is_extend_in_batch`, `self.deepep_mode.resolve`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_extend_in_batch` and `resolved_deepep_mode` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPDispatcher._get_impl`，并承载这一阶段的核心逻辑。 它主要调用 `get_is_extend_in_batch`、`self.deepep_mode.resolve` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `is_extend_in_batch` 和 `resolved_deepep_mode` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 944-947: Internal helper `DeepEPDispatcher._update_stage`
```python
    def _update_stage(self, old_stage, new_stage):
        assert self._stage == old_stage
        self._stage = new_stage
```
**EN:** This block defines `DeepEPDispatcher._update_stage` and contains the main logic for this step. Intermediate names such as `self._stage` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPDispatcher._update_stage`，并承载这一阶段的核心逻辑。 像 `self._stage` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 948-954: `DeepEPDispatcher.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict):
        super().set_quant_config(quant_config)
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.set_quant_config(quant_config)
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher.set_quant_config(quant_config)
```
**EN:** This block defines `DeepEPDispatcher.set_quant_config` and contains the main logic for this step. It mainly invokes `super.set_quant_config`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `self._low_latency_dispatcher.set_quant_config`, and `self._normal_dispatcher.set_quant_config`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPDispatcher.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_quant_config`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`self._low_latency_dispatcher.set_quant_config` 以及 `self._normal_dispatcher.set_quant_config`，说明该流程会编排底层辅助函数或计算内核。

### Lines 955-967: `DeepEPDispatcher.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(
        self, combine_overlap_args: CombineOverlapArgs, meta_overlap_args: dict
    ):
        super().set_overlap_args(combine_overlap_args, meta_overlap_args)
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.set_overlap_args(
                combine_overlap_args, meta_overlap_args
            )
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher.set_overlap_args(
                combine_overlap_args, meta_overlap_args
            )
```
**EN:** This block defines `DeepEPDispatcher.set_overlap_args` and contains the main logic for this step. It mainly invokes `super.set_overlap_args`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `self._low_latency_dispatcher.set_overlap_args`, and `self._normal_dispatcher.set_overlap_args`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPDispatcher.set_overlap_args`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_overlap_args`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`self._low_latency_dispatcher.set_overlap_args` 以及 `self._normal_dispatcher.set_overlap_args`，说明该流程会编排底层辅助函数或计算内核。

### Lines 968-974: Function `DeepEPDispatcher.clear_overlap_args` and its core logic
```python
    def clear_overlap_args(self):
        super().clear_overlap_args()
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.clear_overlap_args()
        if self.deepep_mode.enable_normal():
            self._normal_dispatcher.clear_overlap_args()
```
**EN:** This block defines `DeepEPDispatcher.clear_overlap_args` and contains the main logic for this step. It mainly invokes `super.clear_overlap_args`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `self._low_latency_dispatcher.clear_overlap_args`, and `self._normal_dispatcher.clear_overlap_args`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `DeepEPDispatcher.clear_overlap_args`，并承载这一阶段的核心逻辑。 它主要调用 `super.clear_overlap_args`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`self._low_latency_dispatcher.clear_overlap_args` 以及 `self._normal_dispatcher.clear_overlap_args`，说明该流程会编排底层辅助函数或计算内核。

### Lines 975-976: Function `DeepEPDispatcher.register_deepep_dispatch_hook` and its core logic
```python
    def register_deepep_dispatch_hook(self, hook):
        return self._deepep_dispatch_hooks.register_hook(hook)
```
**EN:** This block defines `DeepEPDispatcher.register_deepep_dispatch_hook` and contains the main logic for this step. It mainly invokes `self._deepep_dispatch_hooks.register_hook`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPDispatcher.register_deepep_dispatch_hook`，并承载这一阶段的核心逻辑。 它主要调用 `self._deepep_dispatch_hooks.register_hook`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_deepep_precompile_tp_barrier`, `DeepEPPDispatchHooks`, `DeepEPNormalDispatchOutput`, `DeepEPLLDispatchOutput`, and `DeepEPNormalCombineInput`. / **主要符号**：核心入口包括 `_deepep_precompile_tp_barrier`、`DeepEPPDispatchHooks`、`DeepEPNormalDispatchOutput`、`DeepEPLLDispatchOutput` 以及 `DeepEPNormalCombineInput`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `os`, `contextlib.nullcontext`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.List`, `typing.NamedTuple`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`logging`、`os`、`contextlib.nullcontext`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.List`、`typing.NamedTuple`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch`, `torch.distributed`, `zbal.zbal.deepep_adaptor.Config`, `zbal.zbal_buffer.Buffer`, `deep_ep.Buffer`, and `deep_ep.Config` / **第三方依赖**：`torch`、`torch.distributed`、`zbal.zbal.deepep_adaptor.Config`、`zbal.zbal_buffer.Buffer`、`deep_ep.Buffer` 以及 `deep_ep.Config`
- **Internal SGLang modules**: `sglang.srt.distributed.parallel_state.get_tp_group`, `sglang.srt.environ.envs`, `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`, `sglang.srt.layers.deep_gemm_wrapper`, `sglang.srt.layers.dp_attention.get_is_extend_in_batch`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcherConfig`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.base.DispatcherBaseHooks`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, and `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat` / **SGLang 内部模块**：`sglang.srt.distributed.parallel_state.get_tp_group`、`sglang.srt.environ.envs`、`sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`、`sglang.srt.layers.deep_gemm_wrapper`、`sglang.srt.layers.dp_attention.get_is_extend_in_batch`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcherConfig`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.base.DispatcherBaseHooks`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput` 以及 `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`
