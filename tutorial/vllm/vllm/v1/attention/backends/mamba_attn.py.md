# mamba_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mamba_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `BaseMambaAttentionMetadata`, `BaseMambaAttentionMetadataBuilder` for the V1 `attention/backends` subsystem. / 为 V1 的 `attention/backends` 子系统实现 `BaseMambaAttentionMetadata`, `BaseMambaAttentionMetadataBuilder`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import abc
from dataclasses import dataclass, replace
from typing import Any, ClassVar, TypeVar

import torch

from vllm.config import VllmConfig
from vllm.utils.math_utils import cdiv
from vllm.utils.torch_utils import async_tensor_h2d
from vllm.v1.attention.backend import (
    AttentionCGSupport,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
)
from vllm.v1.attention.backends.utils import (
    NULL_BLOCK_ID,
    compute_causal_conv1d_metadata,
    mamba_get_block_table_tensor,
    split_decodes_and_prefills,
)
from vllm.v1.kv_cache_interface import AttentionSpec, MambaSpec

M = TypeVar("M", bound="BaseMambaAttentionMetadata")
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `M`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `M`。

### `BaseMambaAttentionMetadata` class / `BaseMambaAttentionMetadata` 类
```python
@dataclass
class BaseMambaAttentionMetadata:
    num_prefills: int
    num_prefill_tokens: int
    num_decodes: int
    num_decode_tokens: int
    num_reqs: int

    # The following tensors only contain prefill requests and will be None if
    # the batch has no prefill requests.
    has_initial_states_p: torch.Tensor | None
    query_start_loc_p: torch.Tensor | None
    num_computed_tokens_p: torch.Tensor | None
    state_indices_tensor_p: torch.Tensor | None

    # The following tensors are used for decode requests and
    # speculative decoding compatibility, and will be None if the batch
    # has no decode requests.
    state_indices_tensor_d: torch.Tensor | None
    query_start_loc_d: torch.Tensor | None  # shape: [num_decodes + 1,]

    # Number of accepted tokens for each spec sequence (for loading correct checkpoint)
    # Includes the bonus token (so minimum is 1)
    num_accepted_tokens: torch.Tensor | None  # shape: [batch,]

    # The following tensors are only used for prefix caching in all mode and
    # are None if disabled
    block_idx_last_scheduled_token: torch.Tensor | None
    block_idx_first_scheduled_token_p: torch.Tensor | None
    block_idx_last_computed_token: torch.Tensor | None
    block_idx_last_scheduled_token_prev_step: torch.Tensor | None

    # The following tensor is only used for prefix caching in align mode
    seq_lens: torch.Tensor

    # cu_chunk_seqlen_p is a tensor of shape (nchunks+1,) that contains, for
    # each chunk, its offsets into the varlen sequence dimension. It is defined
    # such that the i-th chunk contains tokens from cu_chunk_seqlen_p[i] to
    # cu_chunk_seqlen_p[i+1].
    cu_chunk_seqlen_p: torch.Tensor | None = None
    # last_chunk_indices_p is a tensor of shape (batch,) that contains the
    # index of the last chunk for every sequence in the (prefill) batch.
    last_chunk_indices_p: torch.Tensor | None = None

    # The following attributes are for triton implementation of causal_conv1d
    nums_dict: dict | None = None
    batch_ptr: torch.Tensor | None = None
    token_chunk_offset_ptr: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `BaseMambaAttentionMetadata`. Typical fields include `num_prefills`, `num_prefill_tokens`, `num_decodes`, `num_decode_tokens`, `num_reqs`, `has_initial_states_p`.
**CN:** `BaseMambaAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_prefills`, `num_prefill_tokens`, `num_decodes`, `num_decode_tokens`, `num_reqs`, `has_initial_states_p`。

### `BaseMambaAttentionMetadataBuilder` class / `BaseMambaAttentionMetadataBuilder` 类
```python
class BaseMambaAttentionMetadataBuilder(AttentionMetadataBuilder[M], abc.ABC):
    metadata_cls: type[M]
    reorder_batch_threshold: int = 1
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.UNIFORM_BATCH

    # Will be disabled if speculative decoding is used
    supports_update_block_table: bool = True
```
**EN:** Declares the `BaseMambaAttentionMetadataBuilder` interface. Downstream implementations are expected to provide methods such as `__init__`, `build_for_cudagraph_capture`, `build`, `_compute_chunk_metadata`, `_build_chunk_metadata_tensors`, `_compute_prefix_caching_block_indices`.
**CN:** `BaseMambaAttentionMetadataBuilder` 声明了一组接口约定。下游实现需要提供 `__init__`, `build_for_cudagraph_capture`, `build`, `_compute_chunk_metadata`, `_build_chunk_metadata_tensors`, `_compute_prefix_caching_block_indices` 等方法。

### `BaseMambaAttentionMetadataBuilder.__init__` method / `BaseMambaAttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)

        # Enable speculative decoding support
        self.speculative_config = vllm_config.speculative_config
        self.compilation_config = vllm_config.compilation_config
        self.num_spec_tokens: int = vllm_config.num_speculative_tokens
        self.use_spec_decode = self.num_spec_tokens > 0

        assert isinstance(kv_cache_spec, MambaSpec)
        scheduler_config = vllm_config.scheduler_config
        self.decode_cudagraph_max_bs: int = scheduler_config.max_num_seqs
        if self.compilation_config.max_cudagraph_capture_size is not None:
            self.decode_cudagraph_max_bs = min(
                self.decode_cudagraph_max_bs,
                self.compilation_config.max_cudagraph_capture_size,
            )

        if self.vllm_config.cache_config.mamba_cache_mode == "all":
            max_num_blocks = (
                cdiv(
                    self.vllm_config.model_config.max_model_len,
                    kv_cache_spec.block_size,
                )
                + kv_cache_spec.num_speculative_blocks
            )
            # TODO: reduce this size as needed for decode-only cudagraph capture
            self.state_indices_tensor_d: torch.Tensor = torch.empty(
                (
                    self.decode_cudagraph_max_bs,
                    max_num_blocks,
                ),
                dtype=torch.int32,
                device=device,
            )
            self.block_idx_last_scheduled_token: torch.Tensor = torch.empty(
                (self.decode_cudagraph_max_bs,),
                dtype=torch.int32,
                device=device,
            )
            self.block_idx_last_computed_token: torch.Tensor = torch.empty(
                (self.decode_cudagraph_max_bs,),
                dtype=torch.int32,
                device=device,
            )
            if self.use_spec_decode:
                self.block_idx_last_scheduled_token_prev_step: torch.Tensor = (
                    torch.empty(
                        (self.decode_cudagraph_max_bs,),
                        dtype=torch.int32,
                        device=device,
                    )
                )
        else:
            self.state_indices_tensor_d = torch.empty(
                (self.decode_cudagraph_max_bs, 1 + self.num_spec_tokens),
                dtype=torch.int32,
                device=device,
            )

        # For speculative decoding, we need to store the following buffers
        # for CUDA graph capture during decode
        if self.num_spec_tokens > 0:
            self.decode_num_accepted_tokens: torch.Tensor = torch.empty(
                (self.decode_cudagraph_max_bs,),
                dtype=torch.int32,
                device=device,
            )

        self._init_reorder_batch_threshold(1, self.use_spec_decode)
        if self.use_spec_decode:
            self.supports_update_block_table = False
```
**EN:** This method initializes the object state within `BaseMambaAttentionMetadataBuilder`. Key calls include `__init__`, `isinstance`, `_init_reorder_batch_threshold`, `min`, `empty`, `super`. It touches state such as `speculative_config`, `compilation_config`, `num_spec_tokens`, `use_spec_decode`, `decode_cudagraph_max_bs`, `state_indices_tensor_d`, `block_idx_last_scheduled_token`, `block_idx_last_computed_token`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`BaseMambaAttentionMetadataBuilder`。 关键调用包括 `__init__`, `isinstance`, `_init_reorder_batch_threshold`, `min`, `empty`, `super`。 它会读写 `speculative_config`, `compilation_config`, `num_spec_tokens`, `use_spec_decode`, `decode_cudagraph_max_bs`, `state_indices_tensor_d`, `block_idx_last_scheduled_token`, `block_idx_last_computed_token` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BaseMambaAttentionMetadataBuilder._compute_common_metadata` method / `BaseMambaAttentionMetadataBuilder._compute_common_metadata` 方法
```python
    def _compute_common_metadata(
        self,
        common_attn_metadata: CommonAttentionMetadata,
        *,
        num_accepted_tokens: torch.Tensor | None = None,
        prev_last_scheduled_idx: torch.Tensor | None = None,
    ) -> M:
        """
        Compute metadata common to both Mamba1 and Mamba2.
        """
        num_reqs = common_attn_metadata.num_reqs

        # Treat multi-token queries as decode requests when
        # speculative decoding is enabled. Otherwise, use the
        # default decode threshold to prevent misclassification
        # of prefill queries as decode requests.
        decode_threshold = (
            self.reorder_batch_threshold if num_accepted_tokens is not None else 1
        )

        num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
            split_decodes_and_prefills(
                common_attn_metadata,
                decode_threshold=decode_threshold,
                treat_short_extends_as_decodes=False,
            )
        )

        # Need flags to indicate if there are initial states
        has_initial_states_p = None
        query_start_loc_p = None
        query_start_loc_d = None
        num_computed_tokens = None
        num_computed_tokens_p = None

        # for prefix caching
        block_idx_first_scheduled_token = None
        block_idx_first_scheduled_token_p = None
        block_idx_last_computed_token = None
        block_idx_last_scheduled_token = None
        block_idx_last_scheduled_token_prev_step = None

        # for causal_conv1d
        nums_dict, batch_ptr, token_chunk_offset_ptr = None, None, None

        if self.vllm_config.cache_config.mamba_cache_mode == "all":
            num_computed_tokens = common_attn_metadata.compute_num_computed_tokens()

            # Return a tensor of shape (#requests, #max blocks)
            state_indices_tensor = common_attn_metadata.block_table_tensor
    # ... omitted for brevity ...
            num_prefills=num_prefills,
            num_prefill_tokens=num_prefill_tokens,
            num_decodes=num_decodes,
            num_decode_tokens=num_decode_tokens,
            query_start_loc_p=query_start_loc_p,
            has_initial_states_p=has_initial_states_p,
            state_indices_tensor_p=state_indices_tensor_p,
            state_indices_tensor_d=state_indices_tensor_d,
            num_accepted_tokens=num_accepted_tokens,
            query_start_loc_d=query_start_loc_d,
            block_idx_last_scheduled_token=block_idx_last_scheduled_token,
            block_idx_first_scheduled_token_p=block_idx_first_scheduled_token_p,
            block_idx_last_computed_token=block_idx_last_computed_token,
            block_idx_last_scheduled_token_prev_step=(
                block_idx_last_scheduled_token_prev_step
            ),
            num_computed_tokens_p=num_computed_tokens_p,
            num_reqs=num_reqs,
            seq_lens=common_attn_metadata.seq_lens,
            nums_dict=nums_dict,
            batch_ptr=batch_ptr,
            token_chunk_offset_ptr=token_chunk_offset_ptr,
        )

        return self._update_metadata_for_cudagraph_capture(metadata)
```
**EN:** This method implements `_compute_common_metadata` within `BaseMambaAttentionMetadataBuilder`. The docstring frames it as: Compute metadata common to both Mamba1 and Mamba2. Key calls include `split_decodes_and_prefills`, `split`, `metadata_cls`, `_update_metadata_for_cudagraph_capture`, `compute_num_computed_tokens`, `_compute_prefix_caching_block_indices`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_compute_common_metadata`，其作用域位于`BaseMambaAttentionMetadataBuilder`。 关键调用包括 `split_decodes_and_prefills`, `split`, `metadata_cls`, `_update_metadata_for_cudagraph_capture`, `compute_num_computed_tokens`, `_compute_prefix_caching_block_indices`。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `BaseMambaAttentionMetadataBuilder._update_metadata_for_cudagraph_capture` method / `BaseMambaAttentionMetadataBuilder._update_metadata_for_cudagraph_capture` 方法
```python
    def _update_metadata_for_cudagraph_capture(
        self,
        metadata: M,
    ) -> M:
        """
        Update the metadata for cudagraph capture.
        Currently, only decode is supported for full cudagraphs with Mamba.
        """
        state_indices_tensor_d = metadata.state_indices_tensor_d
        query_start_loc_d = metadata.query_start_loc_d
        num_accepted_tokens = metadata.num_accepted_tokens
        block_idx_last_scheduled_token = metadata.block_idx_last_scheduled_token
        block_idx_last_computed_token = metadata.block_idx_last_computed_token
        block_idx_last_scheduled_token_prev_step = (
            metadata.block_idx_last_scheduled_token_prev_step
        )
        if (
            metadata.num_prefills == 0
            and metadata.num_decodes <= self.decode_cudagraph_max_bs
            and self.compilation_config.cudagraph_mode.has_full_cudagraphs()
        ):
            padded_bs = metadata.num_reqs
            self.state_indices_tensor_d[: metadata.num_decodes].copy_(
                state_indices_tensor_d, non_blocking=True
            )
            state_indices_tensor_d = self.state_indices_tensor_d[:padded_bs]
            state_indices_tensor_d[metadata.num_decodes :] = NULL_BLOCK_ID

            if self.use_spec_decode and num_accepted_tokens is not None:
                assert query_start_loc_d is not None
                query_start_loc_d = query_start_loc_d[: padded_bs + 1]
                self.decode_num_accepted_tokens[: metadata.num_decodes].copy_(
                    num_accepted_tokens, non_blocking=True
                )
                num_accepted_tokens = self.decode_num_accepted_tokens[:padded_bs]
                num_accepted_tokens[metadata.num_decodes :] = (
                    1  # pad with 1st slot index
                )

            if self.vllm_config.cache_config.mamba_cache_mode == "all":
                assert block_idx_last_scheduled_token is not None
                assert block_idx_last_computed_token is not None
                self.block_idx_last_scheduled_token[: metadata.num_decodes].copy_(
                    block_idx_last_scheduled_token[: metadata.num_decodes],
                    non_blocking=True,
                )
                block_idx_last_scheduled_token = self.block_idx_last_scheduled_token[
                    :padded_bs
                ]
                block_idx_last_scheduled_token[metadata.num_decodes :] = 0

                self.block_idx_last_computed_token[: metadata.num_decodes].copy_(
                    block_idx_last_computed_token[: metadata.num_decodes],
                    non_blocking=True,
                )
                block_idx_last_computed_token = self.block_idx_last_computed_token[
                    :padded_bs
                ]
                block_idx_last_computed_token[metadata.num_decodes :] = 0

                if (
                    self.use_spec_decode
                    and block_idx_last_scheduled_token_prev_step is not None
                ):
                    self.block_idx_last_scheduled_token_prev_step[
                        : metadata.num_decodes
                    ].copy_(
                        block_idx_last_scheduled_token_prev_step[
                            : metadata.num_decodes
                        ],
                        non_blocking=True,
                    )
                    block_idx_last_scheduled_token_prev_step = (
                        self.block_idx_last_scheduled_token_prev_step[:padded_bs]
                    )
                    block_idx_last_scheduled_token_prev_step[metadata.num_decodes :] = 0

        return replace(
            metadata,
            state_indices_tensor_d=state_indices_tensor_d,
            query_start_loc_d=query_start_loc_d,
            num_accepted_tokens=num_accepted_tokens,
            block_idx_last_scheduled_token=block_idx_last_scheduled_token,
            block_idx_last_computed_token=block_idx_last_computed_token,
            block_idx_last_scheduled_token_prev_step=(
                block_idx_last_scheduled_token_prev_step
            ),
        )
```
**EN:** This method implements `_update_metadata_for_cudagraph_capture` within `BaseMambaAttentionMetadataBuilder`. The docstring frames it as: Update the metadata for cudagraph capture. Key calls include `replace`, `has_full_cudagraphs`, `copy_`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_update_metadata_for_cudagraph_capture`，其作用域位于`BaseMambaAttentionMetadataBuilder`。 关键调用包括 `replace`, `has_full_cudagraphs`, `copy_`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `BaseMambaAttentionMetadata`: central class or interface in this module. / `BaseMambaAttentionMetadata`：本模块中的核心类或接口。
- `BaseMambaAttentionMetadataBuilder`: central class or interface in this module. / `BaseMambaAttentionMetadataBuilder`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`
