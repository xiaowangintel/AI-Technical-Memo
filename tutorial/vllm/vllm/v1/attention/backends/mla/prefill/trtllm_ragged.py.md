# trtllm_ragged.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/prefill/trtllm_ragged.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TRT-LLM Ragged backend for MLA prefill. / 该模块位于 `attention/backends/mla/prefill` 子系统，主要围绕 `TrtllmRaggedPrefillBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""TRT-LLM Ragged backend for MLA prefill."""

from typing import TYPE_CHECKING

import torch

import vllm.envs as envs
from vllm.v1.attention.backends.mla.prefill.base import MLAPrefillBackend
from vllm.v1.worker.workspace import current_workspace_manager

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.model_executor.layers.attention.mla_attention import (
        MLACommonPrefillMetadata,
    )
    from vllm.platforms.interface import DeviceCapability
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `TrtllmRaggedPrefillBackend` class / `TrtllmRaggedPrefillBackend` 类
```python
class TrtllmRaggedPrefillBackend(MLAPrefillBackend):
    """TRT-LLM Ragged backend for MLA prefill."""

    requires_r1_mla_dimensions = True
```
**EN:** Introduces the `TrtllmRaggedPrefillBackend` class on top of `MLAPrefillBackend`. Core methods include `get_name`, `supports_compute_capability`, `is_available`, `__init__`, `prepare_metadata`, `run_prefill_new_tokens`. Docstring signal: TRT-LLM Ragged backend for MLA prefill.
**CN:** 这里定义 `TrtllmRaggedPrefillBackend` 类，其基类包括 `MLAPrefillBackend`。核心方法包括 `get_name`, `supports_compute_capability`, `is_available`, `__init__`, `prepare_metadata`, `run_prefill_new_tokens`。

### `TrtllmRaggedPrefillBackend.get_name` method / `TrtllmRaggedPrefillBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "TRTLLM_RAGGED"
```
**EN:** This method returns or derives a value within `TrtllmRaggedPrefillBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TrtllmRaggedPrefillBackend`。

### `TrtllmRaggedPrefillBackend.supports_compute_capability` method / `TrtllmRaggedPrefillBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, device_capability: "DeviceCapability") -> bool:
        return device_capability.major == 10
```
**EN:** This method implements `supports_compute_capability` within `TrtllmRaggedPrefillBackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`TrtllmRaggedPrefillBackend`。

### `TrtllmRaggedPrefillBackend.is_available` method / `TrtllmRaggedPrefillBackend.is_available` 方法
```python
    @classmethod
    def is_available(cls) -> bool:
        try:
            from flashinfer.prefill import (
                trtllm_ragged_attention_deepseek,  # noqa: F401
            )

            return True
        except ImportError:
            return False
```
**EN:** This method answers a boolean capability check within `TrtllmRaggedPrefillBackend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会回答布尔能力判断，其作用域位于`TrtllmRaggedPrefillBackend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TrtllmRaggedPrefillBackend.__init__` method / `TrtllmRaggedPrefillBackend.__init__` 方法
```python
    def __init__(
        self,
        num_heads: int,
        scale: float,
        kv_lora_rank: int,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        v_head_dim: int,
        vllm_config: "VllmConfig",
    ) -> None:
        super().__init__(
            num_heads=num_heads,
            scale=scale,
            kv_lora_rank=kv_lora_rank,
            qk_nope_head_dim=qk_nope_head_dim,
            qk_rope_head_dim=qk_rope_head_dim,
            v_head_dim=v_head_dim,
            vllm_config=vllm_config,
        )
        (self._workspace_buffer,) = current_workspace_manager().get_simultaneous(
            (
                (envs.VLLM_FLASHINFER_WORKSPACE_BUFFER_SIZE,),
                torch.uint8,
            ),
        )
```
**EN:** This method initializes the object state within `TrtllmRaggedPrefillBackend`. Key calls include `__init__`, `get_simultaneous`, `super`, `current_workspace_manager`. It touches state such as `_workspace_buffer`.
**CN:** 该方法会初始化对象状态，其作用域位于`TrtllmRaggedPrefillBackend`。 关键调用包括 `__init__`, `get_simultaneous`, `super`, `current_workspace_manager`。 它会读写 `_workspace_buffer` 等状态。

### `TrtllmRaggedPrefillBackend.prepare_metadata` method / `TrtllmRaggedPrefillBackend.prepare_metadata` 方法
```python
    def prepare_metadata(
        self,
        prefill_metadata: "MLACommonPrefillMetadata",
    ) -> None:
        super().prepare_metadata(prefill_metadata)
        self._query_seq_lens = (
            prefill_metadata.query_start_loc[1:] - prefill_metadata.query_start_loc[:-1]
        )
```
**EN:** This method prepares inputs and state within `TrtllmRaggedPrefillBackend`. Key calls include `prepare_metadata`, `super`. It touches state such as `_query_seq_lens`.
**CN:** 该方法会准备输入与状态，其作用域位于`TrtllmRaggedPrefillBackend`。 关键调用包括 `prepare_metadata`, `super`。 它会读写 `_query_seq_lens` 等状态。

### `TrtllmRaggedPrefillBackend.run_prefill_new_tokens` method / `TrtllmRaggedPrefillBackend.run_prefill_new_tokens` 方法
```python
    def run_prefill_new_tokens(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        return_softmax_lse: bool,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        from flashinfer.prefill import trtllm_ragged_attention_deepseek

        out = torch.empty(
            q.shape[0],
            q.shape[1],
            v.shape[2],
            device=q.device,
            dtype=self._prefill_metadata.output_dtype,
        )

        ret = trtllm_ragged_attention_deepseek(
            query=q,
            key=k,
            value=v,
            workspace_buffer=self._workspace_buffer,
            seq_lens=self._query_seq_lens,
            max_q_len=self._prefill_metadata.max_query_len,
            max_kv_len=self._prefill_metadata.max_query_len,
            bmm1_scale=self.scale,
            bmm2_scale=1.0,
            o_sf_scale=1.0,
            batch_size=self._query_seq_lens.shape[0],
            window_left=-1,
            cum_seq_lens_q=self._prefill_metadata.query_start_loc,
            cum_seq_lens_kv=self._prefill_metadata.query_start_loc,
            enable_pdl=False,
            is_causal=True,
            return_lse=return_softmax_lse,
            out=out,
        )

        if isinstance(ret, tuple):
            # Convert from (q_len, num_heads) to (num_heads, q_len)
            return ret[0], ret[1].transpose(0, 1).contiguous()
        return ret
```
**EN:** This method runs the main execution path within `TrtllmRaggedPrefillBackend`. Key calls include `empty`, `trtllm_ragged_attention_deepseek`, `isinstance`, `contiguous`, `transpose`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`TrtllmRaggedPrefillBackend`。 关键调用包括 `empty`, `trtllm_ragged_attention_deepseek`, `isinstance`, `contiguous`, `transpose`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TrtllmRaggedPrefillBackend.run_prefill_context_chunk` method / `TrtllmRaggedPrefillBackend.run_prefill_context_chunk` 方法
```python
    def run_prefill_context_chunk(
        self,
        chunk_idx: int,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        from flashinfer.prefill import trtllm_ragged_attention_deepseek

        assert self._prefill_metadata.chunked_context is not None
        assert self._prefill_metadata.chunked_context.seq_lens[chunk_idx] is not None

        out = torch.empty(
            q.shape[0],
            q.shape[1],
            v.shape[2],
            device=q.device,
            dtype=self._prefill_metadata.output_dtype,
        )

        attn_out, lse = trtllm_ragged_attention_deepseek(
            query=q,
            key=k,
            value=v,
            workspace_buffer=self._workspace_buffer,
            seq_lens=self._prefill_metadata.chunked_context.seq_lens[chunk_idx],
            max_q_len=self._prefill_metadata.max_query_len,
            max_kv_len=self._prefill_metadata.chunked_context.max_seq_lens[chunk_idx],
            bmm1_scale=self.scale,
            bmm2_scale=1.0,
            o_sf_scale=1.0,
            batch_size=self._prefill_metadata.chunked_context.seq_lens[chunk_idx].shape[
                0
            ],
            window_left=-1,
            cum_seq_lens_q=self._prefill_metadata.query_start_loc,
            cum_seq_lens_kv=self._prefill_metadata.chunked_context.cu_seq_lens[
                chunk_idx
            ],
            enable_pdl=False,
            is_causal=False,
            return_lse=True,
            out=out,
        )

        # Convert from (q_len, num_heads) to (num_heads, q_len)
        return attn_out, lse.transpose(0, 1).contiguous()
```
**EN:** This method runs the main execution path within `TrtllmRaggedPrefillBackend`. Key calls include `empty`, `trtllm_ragged_attention_deepseek`, `contiguous`, `transpose`.
**CN:** 该方法会执行主要运行路径，其作用域位于`TrtllmRaggedPrefillBackend`。 关键调用包括 `empty`, `trtllm_ragged_attention_deepseek`, `contiguous`, `transpose`。

## Key Concepts / 关键概念
- `TrtllmRaggedPrefillBackend`: central class or interface in this module. / `TrtllmRaggedPrefillBackend`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`, `flashinfer`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.v1.attention.backends.mla.prefill.base`, `vllm.v1.worker.workspace`, `vllm.config`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.platforms.interface`
