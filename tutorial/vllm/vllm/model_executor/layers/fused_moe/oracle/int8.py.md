# int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/oracle/int8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: reference/oracle quantization behavior for MoE kernels; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 内核的参考量化行为；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-26 — imports and setup
```python
from enum import Enum

import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.config.kernel import MoEBackend
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
    int8_w8a8_moe_quant_config,
    int8_w8a16_moe_quant_config,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kInt8DynamicTokenSym,
    kInt8StaticChannelSym,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 29-30 — class `Int8MoeBackend`
```python
class Int8MoeBackend(Enum):
    TRITON = "TRITON"
```
**EN:** This class defines `Int8MoeBackend`. It inherits from `Enum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `TRITON`.
**CN:** 该类定义了 `Int8MoeBackend`。 它继承自 `Enum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `TRITON`。

### Lines 33-39 — function `_get_priority_backends`
```python
def _get_priority_backends(
    moe_config: FusedMoEConfig,
) -> list[Int8MoeBackend]:
    """
    Get available backends in priority order based on platform and config.
    """
    return [Int8MoeBackend.TRITON]
```
**EN:** This function defines `_get_priority_backends`. Get available backends in priority order based on platform and config. The main inputs are `moe_config`.
**CN:** 该函数定义 `_get_priority_backends`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `moe_config`。

### Lines 42-53 — function `backend_to_kernel_cls`
```python
def backend_to_kernel_cls(
    backend: Int8MoeBackend,
) -> list[type[mk.FusedMoEExperts]]:
    if backend == Int8MoeBackend.TRITON:
        from vllm.model_executor.layers.fused_moe.experts.triton_moe import (
            TritonExperts,
        )

        return [TritonExperts]

    else:
        raise ValueError(f"Unknown Int8 MoE backend: {backend.value}")
```
**EN:** This function defines `backend_to_kernel_cls`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `backend`. Key calls include `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `backend_to_kernel_cls`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `backend`。 关键调用包括 `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 56-66 — function `map_int8_backend`
```python
def map_int8_backend(runner_backend: MoEBackend) -> Int8MoeBackend:
    """Map user's MoEBackend to Int8MoeBackend."""
    mapping = {
        "triton": Int8MoeBackend.TRITON,
    }
    if backend := mapping.get(runner_backend):
        return backend
    raise ValueError(
        f"moe_backend='{runner_backend}' is not supported for Int8 MoE. "
        f"Expected one of {list(mapping.keys())}."
    )
```
**EN:** This function defines `map_int8_backend`. Map user's MoEBackend to Int8MoeBackend. The main inputs are `runner_backend`. Key calls include `ValueError`, `mapping.get`, `list`, `mapping.keys`. It writes or updates `mapping`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `map_int8_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `runner_backend`。 关键调用包括 `ValueError`, `mapping.get`, `list`, `mapping.keys`。 它会写入或更新 `mapping`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 69-142 — function `select_int8_moe_backend`
```python
def select_int8_moe_backend(
    config: FusedMoEConfig,
    weight_key: QuantKey | None = kInt8StaticChannelSym,
    activation_key: QuantKey | None = kInt8DynamicTokenSym,
) -> tuple[Int8MoeBackend, type[mk.FusedMoEExperts]]:
    """
    Select the primary Int8 MoE backend.
    Note: Shape-specific fallbacks may still occur at runtime.
    """

    AVAILABLE_BACKENDS = _get_priority_backends(config)

    activation_format = (
        mk.FusedMoEActivationFormat.BatchedExperts
        if config.moe_parallel_config.use_batched_activation_format
        else mk.FusedMoEActivationFormat.Standard
    )

    def _make_log_backend(backend: Int8MoeBackend) -> str:
        available_backend_strs = [b.value for b in AVAILABLE_BACKENDS]
        return (
            f"Using {backend.value} Int8 MoE backend out "
            f"of potential backends: {available_backend_strs}."
        )

    def _make_log_unsupported(backend: Int8MoeBackend, reason: str | None) -> str:
        if reason:
            return (
                f"Int8 MoE backend {backend.value} does not support the "
                f"deployment configuration since {reason}."
            )
        else:
            return (
                f"Int8 MoE backend '{backend.value}' does not support the "
                "deployment configuration."
            )

    def _return_or_raise(
        backend: Int8MoeBackend,
    ) -> tuple[Int8MoeBackend, type[mk.FusedMoEExperts]]:
        for k_cls in backend_to_kernel_cls(backend):
            supported, reason = k_cls.is_supported_config(
                k_cls, config, weight_key, activation_key, activation_format
            )
# ... omitted for brevity ...
        "No Int8 MoE backend supports the deployment configuration."
    )
```
**EN:** This function defines `select_int8_moe_backend`. Select the primary Int8 MoE backend. The main inputs are `config`, `weight_key`, `activation_key`. Key calls include `_get_priority_backends`, `NotImplementedError`, `backend_to_kernel_cls`, `ValueError`, `map_int8_backend`, `_return_or_raise`. It writes or updates `AVAILABLE_BACKENDS`, `activation_format`, `runner_backend`, `available_backend_strs`, `requested_backend`, `supported`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `select_int8_moe_backend`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `config`, `weight_key`, `activation_key`。 关键调用包括 `_get_priority_backends`, `NotImplementedError`, `backend_to_kernel_cls`, `ValueError`, `map_int8_backend`, `_return_or_raise`。 它会写入或更新 `AVAILABLE_BACKENDS`, `activation_format`, `runner_backend`, `available_backend_strs`, `requested_backend`, `supported`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 145-170 — function `make_int8_moe_quant_config`
```python
def make_int8_moe_quant_config(
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    per_act_token_quant: bool = False,
) -> FusedMoEQuantConfig:
    assert (a1_scale is None and a2_scale is None) or (
        a1_scale is not None and a2_scale is not None
    ), "a1_scale and a2_scale must both be provided or both be None"

    if a1_scale is None or a2_scale is None:
        return int8_w8a16_moe_quant_config(
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            w1_zp=None,
            w2_zp=None,
        )

    return int8_w8a8_moe_quant_config(
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
        per_act_token_quant=per_act_token_quant,
    )
```
**EN:** This function defines `make_int8_moe_quant_config`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `w1_scale`, `w2_scale`, `a1_scale`, `a2_scale`, `per_act_token_quant`. Key calls include `int8_w8a8_moe_quant_config`, `int8_w8a16_moe_quant_config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_int8_moe_quant_config`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `w1_scale`, `w2_scale`, `a1_scale`, `a2_scale`, `per_act_token_quant`。 关键调用包括 `int8_w8a8_moe_quant_config`, `int8_w8a16_moe_quant_config`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 173-213 — function `make_int8_moe_kernel`
```python
def make_int8_moe_kernel(
    moe_quant_config: FusedMoEQuantConfig,
    moe_config: FusedMoEConfig,
    experts_cls: type[mk.FusedMoEExperts],
    routing_tables: tuple[torch.Tensor, torch.Tensor, torch.Tensor] | None = None,
) -> mk.FusedMoEKernel:
    # Create Prepare/Finalize.
    prepare_finalize = maybe_make_prepare_finalize(
        moe=moe_config,
        quant_config=moe_quant_config,
        routing_tables=routing_tables,
        allow_new_interface=True,
        use_monolithic=issubclass(experts_cls, mk.FusedMoEExpertsMonolithic),
    )
    assert prepare_finalize is not None

    logger.info_once("Using %s", prepare_finalize.__class__.__name__)

    # Create Experts.
    if prepare_finalize.activation_format == mk.FusedMoEActivationFormat.BatchedExperts:
        max_num_tokens = prepare_finalize.max_num_tokens_per_rank()
        assert max_num_tokens is not None
        experts = experts_cls(
            moe_config=moe_config,
            quant_config=moe_quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=prepare_finalize.num_dispatchers(),
        )
    else:
        experts = experts_cls(
            moe_config=moe_config,
            quant_config=moe_quant_config,
        )

    kernel = mk.FusedMoEKernel(
        prepare_finalize,
        experts,
        inplace=not moe_config.disable_inplace,
    )

    return kernel
```
**EN:** This function defines `make_int8_moe_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `moe_quant_config`, `moe_config`, `experts_cls`, `routing_tables`. Key calls include `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`, `issubclass`. It writes or updates `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_int8_moe_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `moe_quant_config`, `moe_config`, `experts_cls`, `routing_tables`。 关键调用包括 `maybe_make_prepare_finalize`, `logger.info_once`, `mk.FusedMoEKernel`, `prepare_finalize.max_num_tokens_per_rank`, `experts_cls`, `issubclass`。 它会写入或更新 `prepare_finalize`, `kernel`, `max_num_tokens`, `experts`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Reference/oracle quantization behavior for moe kernels / [CN] MoE 内核的参考量化行为
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `Int8MoeBackend`, `_get_priority_backends`, `backend_to_kernel_cls`, `map_int8_backend` / [CN] 核心符号：`Int8MoeBackend`, `_get_priority_backends`, `backend_to_kernel_cls`, `map_int8_backend`

## Dependencies / 依赖关系
- **External**: `enum`, `torch` / **外部依赖**: `enum`, `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.quant_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config.kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.utils.quant_utils`
- **Runtime traits**: Triton kernels, distributed collectives / **运行时特征**: Triton kernels, distributed collectives
