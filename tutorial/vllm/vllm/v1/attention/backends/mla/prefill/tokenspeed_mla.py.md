# tokenspeed_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/prefill/tokenspeed_mla.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TokenSpeed CuTe DSL backend for MLA prefill. / 该模块位于 `attention/backends/mla/prefill` 子系统，主要围绕 `TokenspeedMLAPrefillBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""TokenSpeed CuTe DSL backend for MLA prefill."""

from typing import TYPE_CHECKING

import torch

from vllm.v1.attention.backends.mla.prefill.base import MLAPrefillBackend

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.model_executor.layers.attention.mla_attention import (
        MLACommonPrefillMetadata,
    )
    from vllm.platforms.interface import DeviceCapability
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `TokenspeedMLAPrefillBackend` class / `TokenspeedMLAPrefillBackend` 类
```python
class TokenspeedMLAPrefillBackend(MLAPrefillBackend):
    """TokenSpeed CuTe DSL backend for MLA prefill."""

    requires_r1_mla_dimensions = True
```
**EN:** Introduces the `TokenspeedMLAPrefillBackend` class on top of `MLAPrefillBackend`. Core methods include `get_name`, `supports_compute_capability`, `is_available`, `validate_configuration`, `__init__`, `prepare_metadata`. Docstring signal: TokenSpeed CuTe DSL backend for MLA prefill.
**CN:** 这里定义 `TokenspeedMLAPrefillBackend` 类，其基类包括 `MLAPrefillBackend`。核心方法包括 `get_name`, `supports_compute_capability`, `is_available`, `validate_configuration`, `__init__`, `prepare_metadata`。

### `TokenspeedMLAPrefillBackend.get_name` method / `TokenspeedMLAPrefillBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "TOKENSPEED_MLA"
```
**EN:** This method returns or derives a value within `TokenspeedMLAPrefillBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TokenspeedMLAPrefillBackend`。

### `TokenspeedMLAPrefillBackend.supports_compute_capability` method / `TokenspeedMLAPrefillBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, device_capability: "DeviceCapability") -> bool:
        return device_capability.major == 10
```
**EN:** This method implements `supports_compute_capability` within `TokenspeedMLAPrefillBackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`TokenspeedMLAPrefillBackend`。

### `TokenspeedMLAPrefillBackend.is_available` method / `TokenspeedMLAPrefillBackend.is_available` 方法
```python
    @classmethod
    def is_available(cls) -> bool:
        try:
            from tokenspeed_mla import (
                tokenspeed_mla_prefill,  # noqa: F401
            )

            return True
        except ImportError:
            return False
```
**EN:** This method answers a boolean capability check within `TokenspeedMLAPrefillBackend`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会回答布尔能力判断，其作用域位于`TokenspeedMLAPrefillBackend`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLAPrefillBackend.validate_configuration` method / `TokenspeedMLAPrefillBackend.validate_configuration` 方法
```python
    @classmethod
    def validate_configuration(
        cls,
        device_capability,
        selector_config,
    ) -> list[str]:
        # Replace the generic "required dependencies not available" message
        # from the base class with a specific install hint so users know
        # exactly which package to install when they explicitly select this
        # backend without having tokenspeed_mla installed.
        reasons = super().validate_configuration(device_capability, selector_config)
        return [
            cls._INSTALL_HINT if r == "required dependencies not available" else r
            for r in reasons
        ]
```
**EN:** This method validates assumptions or constraints within `TokenspeedMLAPrefillBackend`. Key calls include `validate_configuration`, `super`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`TokenspeedMLAPrefillBackend`。 关键调用包括 `validate_configuration`, `super`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLAPrefillBackend.__init__` method / `TokenspeedMLAPrefillBackend.__init__` 方法
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

        # Pre-JIT BF16 and FP8 prefill kernels. Idempotent — also called from
        # TokenspeedMLAImpl.__init__; second call is a no-op.
        from tokenspeed_mla import warmup_compile_prefill

        for q_dtype in (torch.bfloat16, torch.float8_e4m3fn):
            warmup_compile_prefill(
                q_dtype=q_dtype,
                d_qk=qk_nope_head_dim + qk_rope_head_dim,
                d_v=v_head_dim,
                enable_pdl=False,
            )
```
**EN:** This method initializes the object state within `TokenspeedMLAPrefillBackend`. Key calls include `__init__`, `warmup_compile_prefill`, `super`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TokenspeedMLAPrefillBackend`。 关键调用包括 `__init__`, `warmup_compile_prefill`, `super`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLAPrefillBackend.prepare_metadata` method / `TokenspeedMLAPrefillBackend.prepare_metadata` 方法
```python
    def prepare_metadata(
        self,
        prefill_metadata: "MLACommonPrefillMetadata",
    ) -> None:
        super().prepare_metadata(prefill_metadata)
        # Kernel signature requires `seq_lens` but the implementation never reads
        # it (per-batch lengths are derived from `cum_seq_lens` diffs); compute
        # for parity with trtllm_ragged. cuda-graph padding in
        # `query_start_loc` is saturated to `total_num_tokens`
        # (gpu_model_runner.py:1905), so trailing diffs are 0 and padded batches
        # are kernel no-ops — same reason trtllm passes the padded length as
        # batch_size directly.
        self._query_seq_lens = (
            prefill_metadata.query_start_loc[1:] - prefill_metadata.query_start_loc[:-1]
        )
```
**EN:** This method prepares inputs and state within `TokenspeedMLAPrefillBackend`. Key calls include `prepare_metadata`, `super`. It touches state such as `_query_seq_lens`.
**CN:** 该方法会准备输入与状态，其作用域位于`TokenspeedMLAPrefillBackend`。 关键调用包括 `prepare_metadata`, `super`。 它会读写 `_query_seq_lens` 等状态。

### `TokenspeedMLAPrefillBackend.run_prefill_new_tokens` method / `TokenspeedMLAPrefillBackend.run_prefill_new_tokens` 方法
```python
    def run_prefill_new_tokens(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        return_softmax_lse: bool,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        from tokenspeed_mla import tokenspeed_mla_prefill

        # `v` arrives as the second half of `kv_nope.split(...)` in
        # mla_attention.forward_mha — a non-contiguous view of `kv_nope` along
        # dim=-1. The kernel does `v.reshape(1, total_kv, h_k, 1, d_v)` which
        # would silently copy on a non-contiguous tensor; force contiguity here
        # so the copy (if any) happens once outside the kernel call.
        v = v.contiguous()

        ret = tokenspeed_mla_prefill(
            query=q,
            key=k,
            value=v,
            seq_lens=self._query_seq_lens,
            cum_seq_lens=self._prefill_metadata.query_start_loc,
            max_seq_len=self._prefill_metadata.max_query_len,
            batch_size=self._query_seq_lens.shape[0],
            softmax_scale=self.scale,
            is_causal=True,
            return_lse=return_softmax_lse,
            enable_pdl=False,
        )

        if isinstance(ret, tuple):
            # Convert from (q_len, num_heads) to (num_heads, q_len)
            return ret[0], ret[1].transpose(0, 1).contiguous()
        return ret
```
**EN:** This method runs the main execution path within `TokenspeedMLAPrefillBackend`. Key calls include `contiguous`, `tokenspeed_mla_prefill`, `isinstance`, `transpose`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`TokenspeedMLAPrefillBackend`。 关键调用包括 `contiguous`, `tokenspeed_mla_prefill`, `isinstance`, `transpose`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLAPrefillBackend.run_prefill_context_chunk` method / `TokenspeedMLAPrefillBackend.run_prefill_context_chunk` 方法
```python
    def run_prefill_context_chunk(
        self,
        chunk_idx: int,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        from tokenspeed_mla import tokenspeed_mla_prefill

        assert self._prefill_metadata.chunked_context is not None
        chunked = self._prefill_metadata.chunked_context

        # See note in run_prefill_new_tokens — `v` is a split-view of `kv_nope`
        # in `_compute_prefill_context` and arrives non-contiguous.
        v = v.contiguous()

        attn_out, lse = tokenspeed_mla_prefill(
            query=q,
            key=k,
            value=v,
            seq_lens=chunked.seq_lens[chunk_idx],
            cum_seq_lens=chunked.cu_seq_lens[chunk_idx],
            max_seq_len=chunked.max_seq_lens[chunk_idx],
            batch_size=chunked.seq_lens[chunk_idx].shape[0],
            softmax_scale=self.scale,
            is_causal=False,
            return_lse=True,
            cum_seq_lens_q=self._prefill_metadata.query_start_loc,
            max_seq_len_q=self._prefill_metadata.max_query_len,
            enable_pdl=False,
        )

        # Convert from (q_len, num_heads) to (num_heads, q_len)
        return attn_out, lse.transpose(0, 1).contiguous()
```
**EN:** This method runs the main execution path within `TokenspeedMLAPrefillBackend`. Key calls include `contiguous`, `tokenspeed_mla_prefill`, `transpose`.
**CN:** 该方法会执行主要运行路径，其作用域位于`TokenspeedMLAPrefillBackend`。 关键调用包括 `contiguous`, `tokenspeed_mla_prefill`, `transpose`。

## Key Concepts / 关键概念
- `TokenspeedMLAPrefillBackend`: central class or interface in this module. / `TokenspeedMLAPrefillBackend`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`, `tokenspeed_mla`
- Internal vLLM / 内部依赖: `vllm.v1.attention.backends.mla.prefill.base`, `vllm.config`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.platforms.interface`
