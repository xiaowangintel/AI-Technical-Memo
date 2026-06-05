# cp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/cp_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `check_attention_cp_compatibility`, `get_total_cp_world_size` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `check_attention_cp_compatibility`, `get_total_cp_world_size`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from typing import TYPE_CHECKING, Any, cast

from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.distributed import get_dcp_group, get_pcp_group

if TYPE_CHECKING:
    from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
else:
    AttentionLayerBase = object
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.config`, `vllm.distributed`, `vllm.model_executor.layers.attention_layer_base`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.config`, `vllm.distributed`, `vllm.model_executor.layers.attention_layer_base` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `check_attention_cp_compatibility` function / `check_attention_cp_compatibility` 函数
```python
def check_attention_cp_compatibility(vllm_config: VllmConfig) -> None:
    pcp_size = vllm_config.parallel_config.prefill_context_parallel_size
    dcp_size = vllm_config.parallel_config.decode_context_parallel_size
    interleave_size = vllm_config.parallel_config.cp_kv_cache_interleave_size
    if pcp_size * dcp_size > 1:
        layer_type = cast(type[Any], AttentionLayerBase)
        layers = get_layers_from_vllm_config(vllm_config, layer_type)
        for layer in layers.values():
            layer_impl = getattr(layer, "impl", None)
            if layer_impl is None:
                continue
            if vllm_config.speculative_config is not None and interleave_size > 1:
                assert layer_impl.supports_mtp_with_cp_non_trivial_interleave_size, (
                    "MTP with cp_kv_cache_interleave_size > 1 is not "
                    f"supported in {layer_impl.__class__.__name__}."
                )
            if dcp_size > 1:
                assert layer_impl.need_to_return_lse_for_decode, (
                    "Decode Context Parallelism (DCP) requires attention "
                    "implementations to return the softmax LSE during decode, "
                    f"but {layer_impl.__class__.__name__} does not. "
                    "Try a different backend by setting "
                    "--attention-backend or disable DCP."
                )

            if pcp_size > 1:
                assert layer_impl.supports_pcp, (
                    "PCP requires attention impls' support, "
                    f"but the impl {layer_impl.__class__.__name__} "
                    "does not support PCP."
                )
```
**EN:** This function validates assumptions or constraints within the module. Key calls include `cast`, `get_layers_from_vllm_config`, `values`, `getattr`. The control flow contains 5 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `cast`, `get_layers_from_vllm_config`, `values`, `getattr`。 控制流包含 5 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `get_total_cp_world_size` function / `get_total_cp_world_size` 函数
```python
def get_total_cp_world_size():
    try:
        pcp_world_size = get_pcp_group().world_size
    except AssertionError:
        # PCP might not be initialized in testing
        pcp_world_size = 1
    try:
        dcp_world_size = get_dcp_group().world_size
    except AssertionError:
        # DCP might not be initialized in testing
        dcp_world_size = 1
    return dcp_world_size * pcp_world_size
```
**EN:** This function returns or derives a value within the module. Key calls include `get_pcp_group`, `get_dcp_group`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_pcp_group`, `get_dcp_group`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `check_attention_cp_compatibility`: top-level helper or orchestration entry point. / `check_attention_cp_compatibility`：顶层辅助函数或编排入口。
- `get_total_cp_world_size`: top-level helper or orchestration entry point. / `get_total_cp_world_size`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.distributed`, `vllm.model_executor.layers.attention_layer_base`
