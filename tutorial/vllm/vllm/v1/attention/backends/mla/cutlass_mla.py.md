# cutlass_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/cutlass_mla.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CutlassMLAMetadataBuilder`, `CutlassMLABackend`, `SM100Workspace` for the V1 `attention/backends/mla` subsystem. / 为 V1 的 `attention/backends/mla` 子系统实现 `CutlassMLAMetadataBuilder`, `CutlassMLABackend`, `SM100Workspace`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import os
from typing import ClassVar

import torch

import vllm._custom_ops as ops
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import (
    MLACommonBackend,
    MLACommonImpl,
    MLACommonMetadata,
    MLACommonMetadataBuilder,
)
from vllm.platforms.interface import DeviceCapability
from vllm.utils.platform_utils import num_compute_units
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import (
    AttentionCGSupport,
    AttentionLayer,
    AttentionType,
    MultipleOf,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `CutlassMLAMetadataBuilder` class / `CutlassMLAMetadataBuilder` 类
```python
class CutlassMLAMetadataBuilder(MLACommonMetadataBuilder[MLACommonMetadata]):
    # enable full CUDA Graph support for decode-only capture
    _cudagraph_support: ClassVar[AttentionCGSupport] = (
        AttentionCGSupport.UNIFORM_SINGLE_TOKEN_DECODE
    )
```
**EN:** Introduces the `CutlassMLAMetadataBuilder` class on top of `MLACommonMetadataBuilder[MLACommonMetadata]`. Core methods include its methods defined below.
**CN:** 这里定义 `CutlassMLAMetadataBuilder` 类，其基类包括 `MLACommonMetadataBuilder[MLACommonMetadata]`。核心方法包括 下方定义的方法。

### `CutlassMLABackend` class / `CutlassMLABackend` 类
```python
class CutlassMLABackend(MLACommonBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
        "fp8",
        "fp8_e4m3",
    ]
```
**EN:** Introduces the `CutlassMLABackend` class on top of `MLACommonBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `supports_compute_capability`.
**CN:** 这里定义 `CutlassMLABackend` 类，其基类包括 `MLACommonBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `supports_compute_capability`。

### `CutlassMLABackend.get_supported_kernel_block_sizes` method / `CutlassMLABackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [128]
```
**EN:** This method returns or derives a value within `CutlassMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`CutlassMLABackend`。

### `CutlassMLABackend.get_name` method / `CutlassMLABackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "CUTLASS_MLA"
```
**EN:** This method returns or derives a value within `CutlassMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`CutlassMLABackend`。

### `CutlassMLABackend.get_impl_cls` method / `CutlassMLABackend.get_impl_cls` 方法
```python
    @staticmethod
    def get_impl_cls() -> type["CutlassMLAImpl"]:
        return CutlassMLAImpl
```
**EN:** This method returns or derives a value within `CutlassMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`CutlassMLABackend`。

### `CutlassMLABackend.get_builder_cls` method / `CutlassMLABackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["CutlassMLAMetadataBuilder"]:
        return CutlassMLAMetadataBuilder
```
**EN:** This method returns or derives a value within `CutlassMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`CutlassMLABackend`。

### `CutlassMLABackend.supports_compute_capability` method / `CutlassMLABackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: DeviceCapability) -> bool:
        return capability.major == 10
```
**EN:** This method implements `supports_compute_capability` within `CutlassMLABackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`CutlassMLABackend`。

### `SM100Workspace` class / `SM100Workspace` 类
```python
class SM100Workspace:
```
**EN:** Introduces the `SM100Workspace` class. Core methods include `__init__`, `get_buf`, `ensure_size`.
**CN:** 这里定义 `SM100Workspace` 类。核心方法包括 `__init__`, `get_buf`, `ensure_size`。

### `SM100Workspace.__init__` method / `SM100Workspace.__init__` 方法
```python
    def __init__(self, initial_workspace_size):
        self._workspace_buf = torch.empty(
            initial_workspace_size, device="cuda", dtype=torch.uint8
        )

        self._block_size = 128  # Forced to 128

        # Pre-compute sm_count to avoid recomputing it. Use device 0 as a proxy
        # (assumes all devices are similar)
        self._sm_count = num_compute_units(0)
```
**EN:** This method initializes the object state within `SM100Workspace`. Key calls include `empty`, `num_compute_units`. It touches state such as `_workspace_buf`, `_block_size`, `_sm_count`.
**CN:** 该方法会初始化对象状态，其作用域位于`SM100Workspace`。 关键调用包括 `empty`, `num_compute_units`。 它会读写 `_workspace_buf`, `_block_size`, `_sm_count` 等状态。

### `SM100Workspace.get_buf` method / `SM100Workspace.get_buf` 方法
```python
    def get_buf(self):
        return self._workspace_buf
```
**EN:** This method returns or derives a value within `SM100Workspace`.
**CN:** 该方法会返回或推导一个值，其作用域位于`SM100Workspace`。

### `SM100Workspace.ensure_size` method / `SM100Workspace.ensure_size` 方法
```python
    def ensure_size(self, attn_metadata: MLACommonMetadata, num_kv_splits: int):
        batch_size = attn_metadata.num_reqs
        max_seq_len = attn_metadata.max_query_len

        workspace_size = ops.sm100_cutlass_mla_get_workspace_size(
            max_seq_len * self._block_size,
            batch_size,
            self._sm_count,
            num_kv_splits=num_kv_splits,
        )

        if self._workspace_buf.shape[0] < workspace_size:
            self._workspace_buf.resize_(workspace_size)
```
**EN:** This method implements `ensure_size` within `SM100Workspace`. Key calls include `sm100_cutlass_mla_get_workspace_size`, `resize_`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `ensure_size`，其作用域位于`SM100Workspace`。 关键调用包括 `sm100_cutlass_mla_get_workspace_size`, `resize_`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
g_sm100_workspace = SM100Workspace(128 * 1024 * 1024)  # 128MB

MAX_HEADS = 128
```
**EN:** Defines module-level constants or aliases such as `g_sm100_workspace`, `MAX_HEADS`, which are reused by later definitions.
**CN:** 定义 `g_sm100_workspace`, `MAX_HEADS` 等模块级常量或别名，供后续定义复用。

### `CutlassMLAImpl` class / `CutlassMLAImpl` 类
```python
class CutlassMLAImpl(MLACommonImpl[MLACommonMetadata]):
    can_return_lse_for_decode: bool = True
```
**EN:** Introduces the `CutlassMLAImpl` class on top of `MLACommonImpl[MLACommonMetadata]`. Core methods include `__init__`, `_sm100_cutlass_mla_decode`, `forward_mqa`.
**CN:** 这里定义 `CutlassMLAImpl` 类，其基类包括 `MLACommonImpl[MLACommonMetadata]`。核心方法包括 `__init__`, `_sm100_cutlass_mla_decode`, `forward_mqa`。

### `CutlassMLAImpl.__init__` method / `CutlassMLAImpl.__init__` 方法
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int,
        alibi_slopes: list[float] | None,
        sliding_window: int | None,
        kv_cache_dtype: str,
        logits_soft_cap: float | None,
        attn_type: str,
        kv_sharing_target_layer_name: str | None,
        # MLA Specific Arguments
        **mla_args,
    ) -> None:
        super().__init__(
            num_heads,
            head_size,
            scale,
            num_kv_heads,
            alibi_slopes,
            sliding_window,
            kv_cache_dtype,
            logits_soft_cap,
            attn_type,
            kv_sharing_target_layer_name,
            q_pad_num_heads=MAX_HEADS,
            **mla_args,
        )

        unsupported_features = [alibi_slopes, sliding_window, logits_soft_cap]
        if any(unsupported_features):
            raise NotImplementedError(
                "CutlassMLAImpl does not support one of the following: "
                "alibi_slopes, sliding_window, logits_soft_cap"
            )

        if attn_type != AttentionType.DECODER:
            raise NotImplementedError(
                "Encoder self-attention and "
                "encoder/decoder cross-attention "
                "are not implemented for "
                "CutlassMLAImpl"
            )

        # TODO: Currently, num_kv_splits is limited to 16 to avoid hanging
        #       issues. In case the code hangs, use:
        #       FORCE_NUM_KV_SPLITS=1
        force_num_kv_splits = os.environ.get("FORCE_NUM_KV_SPLITS", None)
        if force_num_kv_splits:
            logger.debug_once("Forcing num_kv_splits to %d", int(force_num_kv_splits))
            self._num_kv_splits = int(force_num_kv_splits)
        else:
            self._num_kv_splits = -1  # => Auto-detect

        # Share workspace buffer across all executions
        self._workspace = g_sm100_workspace
```
**EN:** This method initializes the object state within `CutlassMLAImpl`. Key calls include `__init__`, `any`, `get`, `NotImplementedError`, `debug_once`, `int`. It touches state such as `_workspace`, `_num_kv_splits`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`CutlassMLAImpl`。 关键调用包括 `__init__`, `any`, `get`, `NotImplementedError`, `debug_once`, `int`。 它会读写 `_workspace`, `_num_kv_splits` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CutlassMLAImpl._sm100_cutlass_mla_decode` method / `CutlassMLAImpl._sm100_cutlass_mla_decode` 方法
```python
    def _sm100_cutlass_mla_decode(
        self,
        q_nope: torch.Tensor,
        q_pe: torch.Tensor,
        kv_c_and_k_pe_cache: torch.Tensor,
        seq_lens: torch.Tensor,
        page_table: torch.Tensor,
        workspace: torch.Tensor,
        sm_scale: float,
        num_kv_splits: int,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        assert q_nope.ndim == 3, f"q_nope must be a 3D tensor, but got {q_nope.ndim}"
        assert q_pe.ndim == 3, f"q_pe must be a 3D tensor, but got {q_pe.ndim}"
        assert kv_c_and_k_pe_cache.ndim == 3, (
            "kv_c_and_k_pe_cache must be a 3D tensor, but got {}".format(
                kv_c_and_k_pe_cache.ndim
            )
        )

        B_q, H, D_q_nope = q_nope.shape
        B_q_2, H_2, D_q_pe = q_pe.shape
        assert (B_q == B_q_2) and (H == H_2)

        _, PAGE_SIZE, D_ckv = kv_c_and_k_pe_cache.shape

        D_latent = 512
        D_rope = 64
        assert D_q_nope == D_latent
        assert D_q_pe == D_rope
        assert D_ckv == D_latent + D_rope

        MAX_HEADS = 128
        assert H <= MAX_HEADS, f"H must be <= {MAX_HEADS}, but got {H}"

        assert len(page_table.shape) == 2
        B_block_table, block_num = page_table.shape
        assert B_block_table == B_q
        assert block_num > 0, f"block num must be greater than 0, got {block_num}"
        assert block_num % (128 / PAGE_SIZE) == 0

        assert q_nope.dtype in (torch.float16, torch.bfloat16, torch.float8_e4m3fn), (
            f"q_nope.dtype needs to be fp16 or bf16 or e4m3 but got {q_nope.dtype}."
        )
        assert q_nope.dtype == q_pe.dtype == kv_c_and_k_pe_cache.dtype
        assert seq_lens.dtype == torch.int32, (
            f"seq_lens.dtype needs to be int32 but got {seq_lens.dtype}."
        )
        assert page_table.dtype == torch.int32, (
            f"page_table.dtype needs to be int32 but got {page_table.dtype}."
        )

        dtype = (
            torch.bfloat16
            if is_quantized_kv_cache(self.kv_cache_dtype)
            else q_nope.dtype
        )
        out = q_nope.new_empty((B_q, MAX_HEADS, D_latent), dtype=dtype)
        lse = (
            torch.empty((B_q, MAX_HEADS), dtype=torch.float32, device=q_nope.device)
            if self.need_to_return_lse_for_decode
            else torch.Tensor()
        )

        ops.sm100_cutlass_mla_decode(
            out,
            lse,
            q_nope,
            q_pe,
            kv_c_and_k_pe_cache,
            seq_lens,
            page_table,
            workspace,
            sm_scale,
            num_kv_splits,
        )

        if H < MAX_HEADS:
            # Extract the subsets of the outputs
            lse = lse[:, :H] if self.need_to_return_lse_for_decode else lse
            out = out[:, :H]

        return out, lse
```
**EN:** This method implements `_sm100_cutlass_mla_decode` within `CutlassMLAImpl`. Key calls include `format`, `new_empty`, `sm100_cutlass_mla_decode`, `len`, `is_quantized_kv_cache`, `empty`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_sm100_cutlass_mla_decode`，其作用域位于`CutlassMLAImpl`。 关键调用包括 `format`, `new_empty`, `sm100_cutlass_mla_decode`, `len`, `is_quantized_kv_cache`, `empty`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CutlassMLAImpl.forward_mqa` method / `CutlassMLAImpl.forward_mqa` 方法
```python
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: MLACommonMetadata,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        assert kv_c_and_k_pe_cache.numel() > 0
        assert attn_metadata.decode is not None

        if layer._q_scale_float != 1.0 or layer._k_scale_float != 1.0:
            raise NotImplementedError(
                "CutlassMLAImpl does not support scaling for q and kv_latent yet"
            )

        if type(q) is tuple:
            q_nope, q_pe = q
        else:
            q_nope, q_pe = torch.split(
                q, [self.kv_lora_rank, self.qk_rope_head_dim], dim=-1
            )

        # Adjust workspace size (if necessary)
        self._workspace.ensure_size(attn_metadata, self._num_kv_splits)

        # Run MLA
        o, lse = self._sm100_cutlass_mla_decode(
            q_nope,
            q_pe,
            kv_c_and_k_pe_cache,
            attn_metadata.decode.seq_lens,
            attn_metadata.decode.block_table,
            self._workspace.get_buf(),
            self.scale,
            self._num_kv_splits,
        )

        return o, (lse if self.need_to_return_lse_for_decode else None)
```
**EN:** This method drives the forward-pass computation within `CutlassMLAImpl`. Key calls include `ensure_size`, `_sm100_cutlass_mla_decode`, `numel`, `NotImplementedError`, `type`, `split`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`CutlassMLAImpl`。 关键调用包括 `ensure_size`, `_sm100_cutlass_mla_decode`, `numel`, `NotImplementedError`, `type`, `split`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `CutlassMLAMetadataBuilder`: central class or interface in this module. / `CutlassMLAMetadataBuilder`：本模块中的核心类或接口。
- `CutlassMLABackend`: central class or interface in this module. / `CutlassMLABackend`：本模块中的核心类或接口。
- `SM100Workspace`: central class or interface in this module. / `SM100Workspace`：本模块中的核心类或接口。
- `CutlassMLAImpl`: central class or interface in this module. / `CutlassMLAImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `os`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm._custom_ops`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.platforms.interface`, `vllm.utils.platform_utils`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`
