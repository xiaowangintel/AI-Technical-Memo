# gdn_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/gdn_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Backend for GatedDeltaNet attention. / 该模块位于 `attention/backends` 子系统，主要围绕 `GDNAttentionBackend`, `GDNAttentionMetadata`, `GDNAttentionMetadataBuilder` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Backend for GatedDeltaNet attention."""

from dataclasses import dataclass

import torch

from vllm.config import VllmConfig
from vllm.v1.attention.backend import (
    AttentionBackend,
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
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `GDNAttentionBackend` class / `GDNAttentionBackend` 类
```python
class GDNAttentionBackend(AttentionBackend):
```
**EN:** Introduces the `GDNAttentionBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_builder_cls`, `is_ssm`.
**CN:** 这里定义 `GDNAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_builder_cls`, `is_ssm`。

### `GDNAttentionBackend.get_name` method / `GDNAttentionBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "GDN_ATTN"
```
**EN:** This method returns or derives a value within `GDNAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`GDNAttentionBackend`。

### `GDNAttentionBackend.get_builder_cls` method / `GDNAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["GDNAttentionMetadataBuilder"]:
        return GDNAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `GDNAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`GDNAttentionBackend`。

### `GDNAttentionBackend.is_ssm` method / `GDNAttentionBackend.is_ssm` 方法
```python
    @classmethod
    def is_ssm(cls) -> bool:
        return True
```
**EN:** This method answers a boolean capability check within `GDNAttentionBackend`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`GDNAttentionBackend`。

### `GDNAttentionMetadata` class / `GDNAttentionMetadata` 类
```python
@dataclass
class GDNAttentionMetadata:
    num_prefills: int
    num_prefill_tokens: int
    num_decodes: int
    num_decode_tokens: int
    num_spec_decodes: int
    num_spec_decode_tokens: int
    num_actual_tokens: int

    has_initial_state: torch.Tensor | None = None

    spec_query_start_loc: torch.Tensor | None = None  # shape: [num_spec_decodes + 1,]
    non_spec_query_start_loc: torch.Tensor | None = (
        None  # shape: [batch - num_spec_decodes + 1,]
    )

    spec_state_indices_tensor: torch.Tensor | None = None  # shape: [batch, num_spec]
    non_spec_state_indices_tensor: torch.Tensor | None = (
        None  # shape: [batch - num_spec_decodes,]
    )
    spec_sequence_masks: torch.Tensor | None = None  # shape: [batch,]
    spec_token_indx: torch.Tensor | None = None
    non_spec_token_indx: torch.Tensor | None = None

    num_accepted_tokens: torch.Tensor | None = None  # shape: [batch,]

    # Pre-computed FLA chunk metadata (avoids GPU->CPU sync in prepare_chunk_indices)
    chunk_indices: torch.Tensor | None = None
    chunk_offsets: torch.Tensor | None = None

    # The following attributes are for triton implementation of causal_conv1d
    nums_dict: dict | None = None
    batch_ptr: torch.Tensor | None = None
    token_chunk_offset_ptr: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `GDNAttentionMetadata`. Typical fields include `num_prefills`, `num_prefill_tokens`, `num_decodes`, `num_decode_tokens`, `num_spec_decodes`, `num_spec_decode_tokens`.
**CN:** `GDNAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_prefills`, `num_prefill_tokens`, `num_decodes`, `num_decode_tokens`, `num_spec_decodes`, `num_spec_decode_tokens`。

### `GDNAttentionMetadataBuilder` class / `GDNAttentionMetadataBuilder` 类
```python
class GDNAttentionMetadataBuilder(AttentionMetadataBuilder[GDNAttentionMetadata]):
    _cudagraph_support = AttentionCGSupport.UNIFORM_BATCH

    reorder_batch_threshold: int = 1
```
**EN:** Introduces the `GDNAttentionMetadataBuilder` class on top of `AttentionMetadataBuilder[GDNAttentionMetadata]`. Core methods include `__init__`, `build`, `build_for_cudagraph_capture`.
**CN:** 这里定义 `GDNAttentionMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[GDNAttentionMetadata]`。核心方法包括 `__init__`, `build`, `build_for_cudagraph_capture`。

### `GDNAttentionMetadataBuilder.__init__` method / `GDNAttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        assert isinstance(kv_cache_spec, MambaSpec)
        self.vllm_config = vllm_config
        self.compilation_config = vllm_config.compilation_config
        self.speculative_config = vllm_config.speculative_config
        self.kv_cache_spec = kv_cache_spec

        if self.speculative_config:
            assert self.speculative_config.num_speculative_tokens is not None
            self.num_spec: int = self.speculative_config.num_speculative_tokens
        else:
            self.num_spec = 0
        self.use_spec_decode: bool = self.num_spec > 0
        self._init_reorder_batch_threshold(1, self.use_spec_decode)

        self.use_full_cuda_graph: bool = (
            self.compilation_config.cudagraph_mode.has_full_cudagraphs()
        )

        self.decode_cudagraph_max_bs: int = (
            self.vllm_config.scheduler_config.max_num_seqs * (self.num_spec + 1)
        )
        if self.compilation_config.max_cudagraph_capture_size is not None:
            self.decode_cudagraph_max_bs = min(
                self.decode_cudagraph_max_bs,
                self.compilation_config.max_cudagraph_capture_size,
            )

        self.spec_state_indices_tensor: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs, self.num_spec + 1),
            dtype=torch.int32,
            device=device,
        )
        self.non_spec_state_indices_tensor: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs,),
            dtype=torch.int32,
            device=device,
        )
        self.spec_sequence_masks: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs,),
            dtype=torch.bool,
            device=device,
        )
        self.spec_token_indx: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs * (self.num_spec + 1),),
            dtype=torch.int32,
            device=device,
        )
        self.non_spec_token_indx: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs * (self.num_spec + 1),),
            dtype=torch.int32,
            device=device,
        )
        self.spec_query_start_loc: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs + 1,),
            dtype=torch.int32,
            device=device,
        )
        self.non_spec_query_start_loc: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs + 1,),
            dtype=torch.int32,
            device=device,
        )
        self.num_accepted_tokens: torch.Tensor = torch.empty(
            (self.decode_cudagraph_max_bs,),
            dtype=torch.int32,
            device=device,
        )
```
**EN:** This method initializes the object state within `GDNAttentionMetadataBuilder`. Key calls include `isinstance`, `_init_reorder_batch_threshold`, `has_full_cudagraphs`, `empty`, `min`. It touches state such as `vllm_config`, `compilation_config`, `speculative_config`, `kv_cache_spec`, `use_spec_decode`, `use_full_cuda_graph`, `decode_cudagraph_max_bs`, `spec_state_indices_tensor`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`GDNAttentionMetadataBuilder`。 关键调用包括 `isinstance`, `_init_reorder_batch_threshold`, `has_full_cudagraphs`, `empty`, `min`。 它会读写 `vllm_config`, `compilation_config`, `speculative_config`, `kv_cache_spec`, `use_spec_decode`, `use_full_cuda_graph`, `decode_cudagraph_max_bs`, `spec_state_indices_tensor` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `GDNAttentionMetadataBuilder.build` method / `GDNAttentionMetadataBuilder.build` 方法
```python
    def build(  # type: ignore[override]
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        num_accepted_tokens: torch.Tensor | None = None,
        num_decode_draft_tokens_cpu: torch.Tensor | None = None,
        fast_build: bool = False,
    ) -> GDNAttentionMetadata:
        m = common_attn_metadata

        query_start_loc = m.query_start_loc
        query_start_loc_cpu = m.query_start_loc_cpu
        context_lens_tensor = m.compute_num_computed_tokens()
        nums_dict, batch_ptr, token_chunk_offset_ptr = None, None, None
        block_table_tensor = mamba_get_block_table_tensor(
            m.block_table_tensor,
            m.seq_lens,
            self.kv_cache_spec,
            self.vllm_config.cache_config.mamba_cache_mode,
        )

        spec_sequence_masks_cpu: torch.Tensor | None = None
        if (
            not self.use_spec_decode
            or num_decode_draft_tokens_cpu is None
            or num_decode_draft_tokens_cpu[num_decode_draft_tokens_cpu >= 0]
            .sum()
            .item()
            == 0
        ):
            spec_sequence_masks = None
            num_spec_decodes = 0
        else:
            spec_sequence_masks_cpu = num_decode_draft_tokens_cpu >= 0
            num_spec_decodes = spec_sequence_masks_cpu.sum().item()
            if num_spec_decodes == 0:
                spec_sequence_masks = None
                spec_sequence_masks_cpu = None
            else:
                spec_sequence_masks = spec_sequence_masks_cpu.to(
                    query_start_loc.device, non_blocking=True
                )

        if spec_sequence_masks is None:
            num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
                split_decodes_and_prefills(m, decode_threshold=1)
            )
            num_spec_decode_tokens = 0
            spec_token_indx = None
            non_spec_token_indx = None
    # ... omitted for brevity ...

        attn_metadata = GDNAttentionMetadata(
            num_prefills=num_prefills,
            num_prefill_tokens=num_prefill_tokens,
            num_decodes=num_decodes,
            num_decode_tokens=num_decode_tokens,
            num_spec_decodes=num_spec_decodes,
            num_spec_decode_tokens=num_spec_decode_tokens,
            num_actual_tokens=m.num_actual_tokens,
            has_initial_state=has_initial_state,
            chunk_indices=chunk_indices,
            chunk_offsets=chunk_offsets,
            spec_query_start_loc=spec_query_start_loc,
            non_spec_query_start_loc=non_spec_query_start_loc,
            spec_state_indices_tensor=spec_state_indices_tensor,
            non_spec_state_indices_tensor=non_spec_state_indices_tensor,
            spec_sequence_masks=spec_sequence_masks,
            spec_token_indx=spec_token_indx,
            non_spec_token_indx=non_spec_token_indx,
            num_accepted_tokens=num_accepted_tokens,
            nums_dict=nums_dict,
            batch_ptr=batch_ptr,
            token_chunk_offset_ptr=token_chunk_offset_ptr,
        )
        return attn_metadata
```
**EN:** This method builds derived structures within `GDNAttentionMetadataBuilder`. Key calls include `compute_num_computed_tokens`, `mamba_get_block_table_tensor`, `GDNAttentionMetadata`, `item`, `split_decodes_and_prefills`, `to`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会构建派生结构，其作用域位于`GDNAttentionMetadataBuilder`。 关键调用包括 `compute_num_computed_tokens`, `mamba_get_block_table_tensor`, `GDNAttentionMetadata`, `item`, `split_decodes_and_prefills`, `to`。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `GDNAttentionMetadataBuilder.build_for_cudagraph_capture` method / `GDNAttentionMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ):
        """
        This method builds the metadata for full cudagraph capture.
        Currently, only decode is supported for full cudagraphs with Mamba.
        """
        m = common_attn_metadata

        assert (
            m.num_reqs <= self.decode_cudagraph_max_bs
            and m.num_actual_tokens <= self.decode_cudagraph_max_bs
        ), (
            f"GDN only supports decode-only full CUDAGraph capture. "
            f"Make sure batch size ({m.num_reqs}) <= "
            f"cudagraph capture sizes ({self.decode_cudagraph_max_bs}), "
            f"and number of tokens ({m.num_actual_tokens}) <= "
            f"cudagraph capture sizes ({self.decode_cudagraph_max_bs})."
        )

        num_accepted_tokens = torch.diff(m.query_start_loc)
        num_decode_draft_tokens_cpu = (num_accepted_tokens - 1).cpu()

        return self.build(0, m, num_accepted_tokens, num_decode_draft_tokens_cpu)
```
**EN:** This method builds derived structures within `GDNAttentionMetadataBuilder`. The docstring frames it as: This method builds the metadata for full cudagraph capture. Key calls include `diff`, `cpu`, `build`.
**CN:** 该方法会构建派生结构，其作用域位于`GDNAttentionMetadataBuilder`。 关键调用包括 `diff`, `cpu`, `build`。

## Key Concepts / 关键概念
- `GDNAttentionBackend`: central class or interface in this module. / `GDNAttentionBackend`：本模块中的核心类或接口。
- `GDNAttentionMetadata`: central class or interface in this module. / `GDNAttentionMetadata`：本模块中的核心类或接口。
- `GDNAttentionMetadataBuilder`: central class or interface in this module. / `GDNAttentionMetadataBuilder`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`, `vllm.model_executor.layers.fla.ops.index`, `vllm.model_executor.layers.fla.ops.utils`
