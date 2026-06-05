# model_runner_kv_cache_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/model_runner_kv_cache_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `model_runner_kv_cache_mixin`. It exposes primary entry points such as `ModelRunnerKVCacheMixin`. / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `model_runner_kv_cache_mixin` 的逻辑。 它对外提供的主要入口包括 `ModelRunnerKVCacheMixin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch

from sglang.srt.configs.model_config import (
    get_nsa_index_head_dim,
    is_deepseek_nsa,
    is_deepseek_v4,
)
from sglang.srt.distributed.parallel_state import get_world_group
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import get_attention_tp_size
from sglang.srt.mem_cache.allocator import (
    PagedTokenToKVPoolAllocator,
    TokenToKVPoolAllocator,
)
from sglang.srt.mem_cache.deepseek_v4_memory_pool import DeepSeekV4TokenToKVPool
from sglang.srt.mem_cache.hisparse_memory_pool import (
    DeepSeekV4HiSparseTokenToKVPoolAllocator,
    HiSparseNSATokenToKVPool,
    HiSparseTokenToKVPoolAllocator,
)
from sglang.srt.mem_cache.memory_pool import (
    HybridLinearKVPool,
    HybridReqToTokenPool,
    MHATokenToKVPool,
    MHATokenToKVPoolFP4,
    MLATokenToKVPool,
    MLATokenToKVPoolFP4,
    NoOpMHATokenToKVPool,
    NSATokenToKVPool,
    ReqToTokenPool,
)
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool, SWATokenToKVPoolAllocator
from sglang.srt.utils.common import (
    get_available_gpu_memory,
    is_float4_e2m1fn_x2,
    is_hip,
    is_npu,
)

if TYPE_CHECKING:
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.model_executor.pool_configurator import MemoryPoolConfig


# the ratio of mamba cache pool size to max_running_requests
MAMBA_CACHE_SIZE_MAX_RUNNING_REQUESTS_RATIO = 3
MAMBA_CACHE_V2_ADDITIONAL_RATIO_OVERLAP = 2
MAMBA_CACHE_V2_ADDITIONAL_RATIO_NO_OVERLAP = 1

logger = logging.getLogger(__name__)

_is_npu = is_npu()
_is_hip = is_hip()


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 61-61: Class ModelRunnerKVCacheMixin
```python
class ModelRunnerKVCacheMixin:
```
**EN:** This range introduces `ModelRunnerKVCacheMixin` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ModelRunnerKVCacheMixin`，并定义其后续方法依赖的结构或元数据。

### Lines 62-76: Method ModelRunnerKVCacheMixin._profile_available_bytes
```python
    def _profile_available_bytes(self: ModelRunner, pre_model_load_memory: int) -> int:
        post_model_load_memory = get_available_gpu_memory(
            self.device,
            self.gpu_id,
            distributed=get_world_group().world_size > 1,
            cpu_group=get_world_group().cpu_group,
        )

        rest_memory = post_model_load_memory - pre_model_load_memory * (
            1 - self.mem_fraction_static
        )
        if self.mambaish_config is not None:
            rest_memory = self.handle_max_mamba_cache(rest_memory)

        return int(rest_memory * (1 << 30))  # return in bytes
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._profile_available_bytes`. It takes `pre_model_load_memory` and mainly implements profile available bytes.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._profile_available_bytes`。它接收 `pre_model_load_memory`，主要用于实现 profile available bytes 相关逻辑。

### Lines 78-137: Method ModelRunnerKVCacheMixin.handle_max_mamba_cache
```python
    def handle_max_mamba_cache(self: ModelRunner, total_rest_memory):
        config = self.mambaish_config
        server_args = self.server_args
        assert config is not None

        # reserve the memory for the intermediate mamba states used for spec dec
        if not self.spec_algorithm.is_none():
            assert server_args.speculative_num_draft_tokens is not None
            assert server_args.max_running_requests is not None

            max_running_requests = server_args.max_running_requests // (
                self.dp_size if server_args.enable_dp_attention else 1
            )
            mamba_state_intermediate_size = (
                config.mamba2_cache_params.mamba_cache_per_req
                * max_running_requests
                * server_args.speculative_num_draft_tokens
            )
            total_rest_memory = total_rest_memory - (
                mamba_state_intermediate_size / (1 << 30)
            )

        if server_args.max_mamba_cache_size is not None:
            # Use explicitly set max_mamba_cache_size
            server_args.max_mamba_cache_size = server_args.max_mamba_cache_size // (
                server_args.dp_size if server_args.enable_dp_attention else 1
            )
        elif (
            server_args.disable_radix_cache
            and server_args.max_running_requests is not None
        ):
            # Use explicitly set max_running_requests when radix cache is disabled
            server_args.max_mamba_cache_size = server_args.max_running_requests // (
                server_args.dp_size if server_args.enable_dp_attention else 1
            )
        else:
            # Use ratio-based calculation to auto-fit available memory
            assert config.mamba2_cache_params.mamba_cache_per_req > 0

            # allocate the memory based on the ratio between mamba state memory vs. full kv cache memory
            # solve the equations:
            # 1. mamba_state_memory + full_kv_cache_memory == total_rest_memory
            # 2. mamba_state_memory / full_kv_cache_memory == server_args.mamba_full_memory_ratio
            mamba_state_memory_raw = (
                total_rest_memory
                * server_args.mamba_full_memory_ratio
                / (1 + server_args.mamba_full_memory_ratio)
            )
            # calculate the max_mamba_cache_size based on the given total mamba memory
            server_args.max_mamba_cache_size = int(
                (mamba_state_memory_raw * (1 << 30))
                // config.mamba2_cache_params.mamba_cache_per_req
            )

        mamba_state_memory = (
            server_args.max_mamba_cache_size
            * config.mamba2_cache_params.mamba_cache_per_req
            / (1 << 30)
        )
        return total_rest_memory - mamba_state_memory
```
**EN:** This callable implements `ModelRunnerKVCacheMixin.handle_max_mamba_cache`. It takes `total_rest_memory` and mainly implements handle max mamba cache. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin.handle_max_mamba_cache`。它接收 `total_rest_memory`，主要用于实现 handle max mamba cache 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 139-184: Method ModelRunnerKVCacheMixin.calculate_mla_kv_cache_dim
```python
    def calculate_mla_kv_cache_dim(self: ModelRunner) -> int:
        is_nsa_model = is_deepseek_nsa(self.model_config.hf_config)
        kv_cache_dtype = self.kv_cache_dtype
        kv_lora_rank = self.model_config.kv_lora_rank
        qk_rope_head_dim = self.model_config.qk_rope_head_dim
        kv_cache_dim = kv_lora_rank + qk_rope_head_dim  # default mla kv cache dim

        # For non-NSA models, MLA kv cache dim is simply kv_lora_rank + qk_rope_head_dim
        if not is_nsa_model:
            return kv_cache_dim

        # TRTLLM backend does not override kv_cache_dim for MLA kv cache
        # Assuming nsa prefill and decode backends are the same when using trtllm MLA backend,
        # since it is not compatible for trtllm and other mla attn backend due to the different
        # kv cache layout.
        if (
            self.server_args.nsa_prefill_backend == "trtllm"
            or self.server_args.nsa_decode_backend == "trtllm"
        ):
            return kv_cache_dim

        # On HIP with TileLang backend, keep the default MLA KV cache dimension.
        # FP8 attention uses the nope(512 fp8) + rope(64 fp8) layout, without extra per-block scales.
        if _is_hip and (
            self.server_args.nsa_prefill_backend == "tilelang"
            or self.server_args.nsa_decode_backend == "tilelang"
        ):
            return kv_cache_dim

        quant_block_size = NSATokenToKVPool.quant_block_size
        rope_storage_dtype = NSATokenToKVPool.rope_storage_dtype
        # Calculate override_kv_cache_dim for FP8 storage in backends that use scaled KV layout (excluding TRTLLM and HIP+TileLang).
        # kv_lora_rank + scale storage (kv_lora_rank // quant_block_size * 4 bytes) + rope dimension storage
        # Note: rope dimension is stored in original dtype (bf16), not quantized to fp8
        if kv_cache_dtype == torch.float8_e4m3fn:
            assert (
                kv_lora_rank % quant_block_size == 0
            ), f"kv_lora_rank {kv_lora_rank} must be multiple of quant_block_size {quant_block_size}"

            return (
                kv_lora_rank
                + kv_lora_rank // quant_block_size * 4
                + qk_rope_head_dim * rope_storage_dtype.itemsize
            )

        return kv_cache_dim
```
**EN:** This callable implements `ModelRunnerKVCacheMixin.calculate_mla_kv_cache_dim` and mainly implements calculate mla kv cache dim. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin.calculate_mla_kv_cache_dim`，主要用于实现 calculate mla kv cache dim 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 186-198: Method ModelRunnerKVCacheMixin._calculate_mamba_ratio
```python
    def _calculate_mamba_ratio(self: ModelRunner) -> int:
        if self.server_args.disable_radix_cache:
            return 1

        additional_ratio = 0
        if self.server_args.enable_mamba_extra_buffer():
            # ping-pong buffer size is 2 when overlap schedule is on, 1 otherwise.
            if not self.server_args.disable_overlap_schedule:
                additional_ratio = MAMBA_CACHE_V2_ADDITIONAL_RATIO_OVERLAP
            else:
                additional_ratio = MAMBA_CACHE_V2_ADDITIONAL_RATIO_NO_OVERLAP

        return MAMBA_CACHE_SIZE_MAX_RUNNING_REQUESTS_RATIO + additional_ratio
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._calculate_mamba_ratio` and mainly implements calculate mamba ratio.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._calculate_mamba_ratio`，主要用于实现 calculate mamba ratio 相关逻辑。

### Lines 200-236: Method ModelRunnerKVCacheMixin._validate_prefill_only_disable_kv_cache_pool_family
```python
    def _validate_prefill_only_disable_kv_cache_pool_family(
        self: ModelRunner,
        is_nsa_model: bool,
        is_dsv4_model: bool,
        current_platform,
    ):
        if not self.server_args.prefill_only_disable_kv_cache or self.is_draft_worker:
            return

        unsupported_pool_family = None
        if is_dsv4_model:
            unsupported_pool_family = "DeepSeekV4TokenToKVPool"
        elif current_platform.is_out_of_tree() and not self.mambaish_config:
            unsupported_pool_family = "out-of-tree platform KV pool"
        elif (
            self.server_args.attention_backend == "ascend" and not self.mambaish_config
        ):
            unsupported_pool_family = "NPU/Ascend KV pool"
        elif self.use_mla_backend and is_nsa_model:
            unsupported_pool_family = "NSA/MLA KV pool"
        elif self.use_mla_backend and not self.mambaish_config:
            unsupported_pool_family = "MLA KV pool"
        elif self.is_hybrid_swa:
            unsupported_pool_family = "SWA KV pool"
        elif self.mambaish_config:
            unsupported_pool_family = "hybrid linear/Mamba KV pool"
        elif is_float4_e2m1fn_x2(self.kv_cache_dtype):
            unsupported_pool_family = "FP4 MHA KV pool"

        if unsupported_pool_family is not None:
            raise RuntimeError(
                "--prefill-only-disable-kv-cache is not supported for "
                f"{unsupported_pool_family}. Supported configurations today: plain MHA "
                "models on CUDA with the FA (fa3/fa4) prefill backend, --is-embedding, "
                "--chunked-prefill-size=-1, --disable-radix-cache, no context-parallel "
                "attention, no HiSparse, and --kv-cache-dtype != fp4_e2m1."
            )
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._validate_prefill_only_disable_kv_cache_pool_family`. It takes `is_nsa_model`, `is_dsv4_model`, `current_platform` and mainly validates inputs and invariants. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._validate_prefill_only_disable_kv_cache_pool_family`。它接收 `is_nsa_model`, `is_dsv4_model`, `current_platform`，主要用于校验输入与不变量。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 238-297: Method ModelRunnerKVCacheMixin._init_pools (part 1/10)
```python
    def _init_pools(self: ModelRunner):
        """Initialize the memory pools."""
        max_num_reqs = self.max_running_requests

        # Initialize req_to_token_pool
        if self.req_to_token_pool is None:
            # FIXME(lsyin): this is the temporary fix for the context length issue when using speculative decoding
            extra_max_context_len = 4
            if self.server_args.speculative_num_draft_tokens is not None:
                extra_max_context_len += self.server_args.speculative_num_draft_tokens

            if self.server_args.disaggregation_mode == "decode":
                from sglang.srt.disaggregation.decode import (
                    DecodeReqToTokenPool,
                    HybridMambaDecodeReqToTokenPool,
                )

                # subscribe memory for pre-allocated requests
                # if max_num_reqs <= 32, we pre-allocate 2x requests

                pre_alloc_size = envs.SGLANG_DISAGGREGATION_NUM_PRE_ALLOCATE_REQS.get()
                pre_alloc_size = (
                    max_num_reqs * 2 if max_num_reqs <= 32 else pre_alloc_size
                )
                if config := self.mambaish_config:
                    self.req_to_token_pool = HybridMambaDecodeReqToTokenPool(
                        size=max_num_reqs,
                        max_context_len=self.model_config.context_len
                        + extra_max_context_len,
                        device=self.device,
                        enable_memory_saver=self.server_args.enable_memory_saver,
                        cache_params=config.mamba2_cache_params,
                        mamba_layer_ids=(
                            [
                                i
                                for i in config.mamba2_cache_params.layers
                                if self.start_layer <= i < self.end_layer
                            ]
                        ),
                        speculative_num_draft_tokens=self.server_args.speculative_num_draft_tokens,
                        enable_mamba_extra_buffer=self.server_args.enable_mamba_extra_buffer(),
                        pre_alloc_size=pre_alloc_size,
                        enable_overlap_schedule=not self.server_args.disable_overlap_schedule,
                        mamba_size=self.server_args.max_mamba_cache_size,
                        start_layer=self.start_layer,
                    )
                else:
                    self.req_to_token_pool = DecodeReqToTokenPool(
                        size=max_num_reqs,
                        max_context_len=self.model_config.context_len
                        + extra_max_context_len,
                        device=self.device,
                        enable_memory_saver=self.server_args.enable_memory_saver,
                        pre_alloc_size=pre_alloc_size,
                    )
            elif config := self.mambaish_config:
                self.req_to_token_pool = HybridReqToTokenPool(
                    size=max_num_reqs,
                    mamba_size=self.server_args.max_mamba_cache_size,
                    mamba_spec_state_size=max_num_reqs,
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 1 of 10 for the same logical block. In this range it sets up imports and shared symbols; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 1/10 部分。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置。

### Lines 298-357: Method ModelRunnerKVCacheMixin._init_pools (part 2/10)
```python
                    max_context_len=self.model_config.context_len
                    + extra_max_context_len,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    cache_params=config.mamba2_cache_params,
                    mamba_layer_ids=(
                        [
                            i
                            for i in config.mamba2_cache_params.layers
                            if self.start_layer <= i < self.end_layer
                        ]
                    ),
                    enable_mamba_extra_buffer=self.server_args.enable_mamba_extra_buffer(),
                    speculative_num_draft_tokens=self.server_args.speculative_num_draft_tokens,
                    enable_overlap_schedule=not self.server_args.disable_overlap_schedule,
                    start_layer=self.start_layer,
                )
            else:
                self.req_to_token_pool = ReqToTokenPool(
                    size=max_num_reqs,
                    max_context_len=self.model_config.context_len
                    + extra_max_context_len,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                )
        else:
            # Draft worker shares req_to_token_pool with the target worker.
            assert self.is_draft_worker

        # Initialize token_to_kv_pool
        is_nsa_model = is_deepseek_nsa(self.model_config.hf_config)
        is_dsv4_model = is_deepseek_v4(self.model_config.hf_config)

        # Out-of-tree platform plugin system — used by elif below
        from sglang.srt.platforms import current_platform

        self._validate_prefill_only_disable_kv_cache_pool_family(
            is_nsa_model, is_dsv4_model, current_platform
        )

        if is_dsv4_model:
            swa_page_size = self.page_size
            assert swa_page_size == 256, "In paged swa mode, page_size must be 256."

            if self.is_draft_worker:
                from sglang.srt.models.deepseek_v4_nextn import (
                    COMPRESS_RATIO_NEXTN_LAYER,
                )

                compression_ratios = [
                    COMPRESS_RATIO_NEXTN_LAYER
                ] * self.num_effective_layers
            else:
                compression_ratios = self.model_config.compress_ratios
            self.token_to_kv_pool = DeepSeekV4TokenToKVPool(
                max_num_reqs=self.max_running_requests,
                swa_size=self.swa_max_total_num_tokens,
                c4_size=self.c4_max_total_num_tokens,
                c128_size=self.c128_max_total_num_tokens,
                c4_state_pool_size=self.c4_state_pool_size,
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 2 of 10 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 2/10 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 358-417: Method ModelRunnerKVCacheMixin._init_pools (part 3/10)
```python
                c128_state_pool_size=self.c128_state_pool_size,
                page_size=self.page_size,
                swa_page_size=swa_page_size,
                dtype=self.kv_cache_dtype,
                state_dtype=self.state_dtype,
                qk_nope_head_dim=self.model_config.qk_nope_head_dim,
                qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                indexer_head_dim=self.model_config.index_head_dim,
                layer_num=self.num_effective_layers,
                device=self.device,
                enable_memory_saver=self.server_args.enable_memory_saver,
                compression_ratios=compression_ratios,
                start_layer=self.start_layer,
                end_layer=self.end_layer,
                enable_hisparse=self.enable_hisparse,
            )
        elif current_platform.is_out_of_tree() and not self.mambaish_config:
            if self.use_mla_backend and is_nsa_model:
                PoolCls = current_platform.get_nsa_kv_pool_cls()
                self.token_to_kv_pool = PoolCls(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    kv_lora_rank=self.model_config.kv_lora_rank,
                    qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    kv_cache_dim=self.calculate_mla_kv_cache_dim(),
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
                    end_layer=self.end_layer,
                    index_head_dim=get_nsa_index_head_dim(self.model_config.hf_config),
                )
            elif self.use_mla_backend:
                PoolCls = current_platform.get_mla_kv_pool_cls()
                self.token_to_kv_pool = PoolCls(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    kv_lora_rank=self.model_config.kv_lora_rank,
                    qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                    index_head_dim=(
                        self.model_config.index_head_dim if is_nsa_model else None
                    ),
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
                    end_layer=self.end_layer,
                )
            else:
                PoolCls = current_platform.get_mha_kv_pool_cls()
                self.token_to_kv_pool = PoolCls(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    head_num=self.model_config.get_num_kv_heads(
                        get_attention_tp_size()
                    ),
                    head_dim=self.model_config.head_dim,
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 3 of 10 for the same logical block.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 3/10 部分。

### Lines 418-477: Method ModelRunnerKVCacheMixin._init_pools (part 4/10)
```python
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
                    end_layer=self.end_layer,
                )
        elif (
            self.server_args.attention_backend == "ascend" and not self.mambaish_config
        ):
            if self.is_hybrid_swa:
                from sglang.srt.hardware_backend.npu.memory_pool_npu import (
                    NPUMHATokenToKVPool,
                )

                kwargs = {}
                if self.is_hybrid_swa_compress:
                    kwargs = {
                        "swa_head_num": max(
                            1,
                            self.model_config.hf_text_config.swa_num_key_value_heads
                            // get_attention_tp_size(),
                        ),
                        "swa_head_dim": self.model_config.hf_text_config.swa_head_dim,
                        "swa_v_head_dim": self.model_config.hf_text_config.swa_v_head_dim,
                        "v_head_dim": self.model_config.hf_text_config.v_head_dim,
                    }
                self.token_to_kv_pool = SWAKVPool(
                    size=self.full_max_total_num_tokens,
                    size_swa=self.swa_max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    head_num=self.model_config.get_num_kv_heads(
                        get_attention_tp_size()
                    ),
                    head_dim=self.model_config.head_dim,
                    swa_attention_layer_ids=self.model_config.swa_attention_layer_ids,
                    full_attention_layer_ids=self.model_config.full_attention_layer_ids,
                    enable_kvcache_transpose=False,
                    device=self.device,
                    token_to_kv_pool_class=NPUMHATokenToKVPool,
                    **kwargs,
                )
            elif self.use_mla_backend:
                from sglang.srt.hardware_backend.npu.memory_pool_npu import (
                    NPUMLATokenToKVPool,
                )

                self.token_to_kv_pool = NPUMLATokenToKVPool(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    kv_lora_rank=self.model_config.kv_lora_rank,
                    qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                    index_head_dim=(
                        self.model_config.index_head_dim if is_nsa_model else None
                    ),
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 4 of 10 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 4/10 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 478-537: Method ModelRunnerKVCacheMixin._init_pools (part 5/10)
```python
                    end_layer=self.end_layer,
                )
            else:
                from sglang.srt.hardware_backend.npu.memory_pool_npu import (
                    NPUMHATokenToKVPool,
                )

                self.token_to_kv_pool = NPUMHATokenToKVPool(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    head_num=self.model_config.get_num_kv_heads(
                        get_attention_tp_size()
                    ),
                    head_dim=self.model_config.head_dim,
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
                    end_layer=self.end_layer,
                )
        elif self.use_mla_backend and is_nsa_model:
            PoolCls = (
                HiSparseNSATokenToKVPool if self.enable_hisparse else NSATokenToKVPool
            )
            pool_kwargs = {}
            if self.enable_hisparse:
                from sglang.srt.mem_cache.sparsity import parse_hisparse_config

                pool_kwargs["host_to_device_ratio"] = parse_hisparse_config(
                    self.server_args
                ).host_to_device_ratio
            self.token_to_kv_pool = PoolCls(
                self.max_total_num_tokens,
                page_size=self.page_size,
                dtype=self.kv_cache_dtype,
                kv_lora_rank=self.model_config.kv_lora_rank,
                qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                layer_num=self.num_effective_layers,
                device=self.device,
                kv_cache_dim=self.calculate_mla_kv_cache_dim(),
                enable_memory_saver=self.server_args.enable_memory_saver,
                start_layer=self.start_layer,
                end_layer=self.end_layer,
                index_head_dim=get_nsa_index_head_dim(self.model_config.hf_config),
                **pool_kwargs,
            )
        elif self.use_mla_backend and not self.mambaish_config:
            assert not is_nsa_model
            if is_float4_e2m1fn_x2(self.kv_cache_dtype):
                self.token_to_kv_pool = MLATokenToKVPoolFP4(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    kv_lora_rank=self.model_config.kv_lora_rank,
                    qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 5 of 10 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 5/10 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 538-597: Method ModelRunnerKVCacheMixin._init_pools (part 6/10)
```python
                    end_layer=self.end_layer,
                )
            else:
                self.token_to_kv_pool = MLATokenToKVPool(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    kv_lora_rank=self.model_config.kv_lora_rank,
                    qk_rope_head_dim=self.model_config.qk_rope_head_dim,
                    layer_num=self.num_effective_layers,
                    device=self.device,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    start_layer=self.start_layer,
                    end_layer=self.end_layer,
                )
        else:
            if self.is_hybrid_swa:
                kwargs = {}
                if self.is_hybrid_swa_compress:
                    kwargs = {
                        "swa_head_num": max(
                            1,
                            self.model_config.hf_text_config.swa_num_key_value_heads
                            // get_attention_tp_size(),
                        ),
                        "swa_head_dim": self.model_config.hf_text_config.swa_head_dim,
                        "swa_v_head_dim": self.model_config.hf_text_config.swa_v_head_dim,
                        "v_head_dim": self.model_config.hf_text_config.v_head_dim,
                    }
                self.token_to_kv_pool = SWAKVPool(
                    size=self.full_max_total_num_tokens,
                    size_swa=self.swa_max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    head_num=self.model_config.get_num_kv_heads(
                        get_attention_tp_size()
                    ),
                    head_dim=self.model_config.head_dim,
                    swa_attention_layer_ids=self.model_config.swa_attention_layer_ids,
                    full_attention_layer_ids=self.model_config.full_attention_layer_ids,
                    enable_kvcache_transpose=False,
                    device=self.device,
                    **kwargs,
                )
            elif config := self.mambaish_config:
                extra_args = {}
                if self.use_mla_backend:
                    extra_args = {
                        "kv_lora_rank": self.model_config.kv_lora_rank,
                        "qk_rope_head_dim": self.model_config.qk_rope_head_dim,
                    }
                self.token_to_kv_pool = HybridLinearKVPool(
                    page_size=self.page_size,
                    size=self.max_total_num_tokens,
                    dtype=self.kv_cache_dtype,
                    head_num=self.model_config.get_num_kv_heads(
                        get_attention_tp_size()
                    ),
                    head_dim=self.model_config.head_dim,
                    # if draft worker, we only need 1 attention layer's kv pool
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 6 of 10 for the same logical block.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 6/10 部分。

### Lines 598-657: Method ModelRunnerKVCacheMixin._init_pools (part 7/10)
```python
                    full_attention_layer_ids=(
                        [0]
                        if self.is_draft_worker
                        else [
                            i
                            for i in config.full_attention_layer_ids
                            if self.start_layer <= i < self.end_layer
                        ]
                    ),
                    enable_kvcache_transpose=False,
                    device=self.device,
                    mamba_pool=self.req_to_token_pool.mamba_pool,
                    enable_memory_saver=self.server_args.enable_memory_saver,
                    use_mla=self.use_mla_backend,
                    start_layer=self.start_layer,
                    **extra_args,
                )
            else:
                if is_float4_e2m1fn_x2(self.kv_cache_dtype):
                    self.token_to_kv_pool = MHATokenToKVPoolFP4(
                        self.max_total_num_tokens,
                        page_size=self.page_size,
                        dtype=self.kv_cache_dtype,
                        head_num=self.model_config.get_num_kv_heads(
                            get_attention_tp_size()
                        ),
                        head_dim=self.model_config.head_dim,
                        v_head_dim=self.model_config.v_head_dim,
                        layer_num=self.num_effective_layers,
                        device=self.device,
                        enable_memory_saver=self.server_args.enable_memory_saver,
                        start_layer=self.start_layer,
                        end_layer=self.end_layer,
                        enable_alt_stream=not self.server_args.enable_pdmux,
                        enable_kv_cache_copy=(
                            self.server_args.speculative_algorithm is not None
                        ),
                    )
                else:
                    pool_cls = (
                        NoOpMHATokenToKVPool
                        if self.server_args.prefill_only_disable_kv_cache
                        else MHATokenToKVPool
                    )
                    self.token_to_kv_pool = pool_cls(
                        self.max_total_num_tokens,
                        page_size=self.page_size,
                        dtype=self.kv_cache_dtype,
                        head_num=self.model_config.get_num_kv_heads(
                            get_attention_tp_size()
                        ),
                        head_dim=self.model_config.head_dim,
                        v_head_dim=self.model_config.v_head_dim,
                        layer_num=self.num_effective_layers,
                        device=self.device,
                        enable_memory_saver=self.server_args.enable_memory_saver,
                        start_layer=self.start_layer,
                        end_layer=self.end_layer,
                        enable_alt_stream=not self.server_args.enable_pdmux,
                        enable_kv_cache_copy=(
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 7 of 10 for the same logical block.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 7/10 部分。

### Lines 658-717: Method ModelRunnerKVCacheMixin._init_pools (part 8/10)
```python
                            self.server_args.speculative_algorithm is not None
                        ),
                    )

        # Initialize token_to_kv_pool_allocator
        need_sort = self.server_args.disaggregation_mode in ("decode", "prefill")
        if self.token_to_kv_pool_allocator is None:
            if current_platform.is_out_of_tree():
                AllocatorCls = current_platform.get_paged_allocator_cls()
                self.token_to_kv_pool_allocator = AllocatorCls(
                    self.max_total_num_tokens,
                    page_size=self.page_size,
                    dtype=self.kv_cache_dtype,
                    device=self.device,
                    kvcache=self.token_to_kv_pool,
                    need_sort=need_sort,
                )
            elif _is_npu and (
                self.server_args.attention_backend == "ascend"
                or self.hybrid_gdn_config is not None
            ):
                if self.is_hybrid_swa:
                    self.token_to_kv_pool_allocator = SWATokenToKVPoolAllocator(
                        self.full_max_total_num_tokens,
                        self.swa_max_total_num_tokens,
                        page_size=self.page_size,
                        dtype=self.kv_cache_dtype,
                        device=self.device,
                        kvcache=self.token_to_kv_pool,
                        need_sort=need_sort,
                    )
                else:
                    from sglang.srt.hardware_backend.npu.allocator_npu import (
                        NPUPagedTokenToKVPoolAllocator,
                    )

                    self.token_to_kv_pool_allocator = NPUPagedTokenToKVPoolAllocator(
                        self.max_total_num_tokens,
                        page_size=self.page_size,
                        dtype=self.kv_cache_dtype,
                        device=self.device,
                        kvcache=self.token_to_kv_pool,
                        need_sort=need_sort,
                    )
            else:
                if self.is_hybrid_swa:
                    self.token_to_kv_pool_allocator = SWATokenToKVPoolAllocator(
                        self.full_max_total_num_tokens,
                        self.swa_max_total_num_tokens,
                        page_size=self.page_size,
                        dtype=self.kv_cache_dtype,
                        device=self.device,
                        kvcache=self.token_to_kv_pool,
                        need_sort=need_sort,
                    )
                else:
                    if self.enable_hisparse:
                        from sglang.srt.mem_cache.sparsity import (
                            parse_hisparse_config,
                        )
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 8 of 10 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 8/10 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 718-777: Method ModelRunnerKVCacheMixin._init_pools (part 9/10)
```python

                        hisparse_cfg = parse_hisparse_config(self.server_args)
                        self.token_to_kv_pool_allocator = (
                            HiSparseTokenToKVPoolAllocator(
                                self.max_total_num_tokens,
                                page_size=self.page_size,
                                dtype=self.kv_cache_dtype,
                                device=self.device,
                                kvcache=self.token_to_kv_pool,
                                need_sort=need_sort,
                                host_to_device_ratio=hisparse_cfg.host_to_device_ratio,
                            )
                        )
                    elif self.page_size == 1:
                        self.token_to_kv_pool_allocator = TokenToKVPoolAllocator(
                            self.max_total_num_tokens,
                            dtype=self.kv_cache_dtype,
                            device=self.device,
                            kvcache=self.token_to_kv_pool,
                            need_sort=need_sort,
                        )
                    else:
                        self.token_to_kv_pool_allocator = PagedTokenToKVPoolAllocator(
                            self.max_total_num_tokens,
                            page_size=self.page_size,
                            dtype=self.kv_cache_dtype,
                            device=self.device,
                            kvcache=self.token_to_kv_pool,
                            need_sort=need_sort,
                        )

            if self.enable_hisparse and is_dsv4_model:
                assert self.is_hybrid_swa, "DeepSeek V4 HiSparse requires SWA mode."
                self.token_to_kv_pool_allocator = (
                    DeepSeekV4HiSparseTokenToKVPoolAllocator(
                        self.token_to_kv_pool_allocator
                    )
                )

        else:
            assert self.is_draft_worker
            if self.is_hybrid_swa:
                swa_allocator = getattr(
                    self.token_to_kv_pool_allocator,
                    "logical_attn_allocator",
                    self.token_to_kv_pool_allocator,
                )
                assert swa_allocator.__class__ == SWATokenToKVPoolAllocator
                self.token_to_kv_pool.full_to_swa_index_mapping = (
                    swa_allocator.full_to_swa_index_mapping
                )

        # Defensive check: the explicit validation above should reject known
        # unsupported pool families before allocation. Keep this guard here so
        # future pool-selection refactors fail at boot instead of on first use.
        if (
            self.server_args.prefill_only_disable_kv_cache
            and not self.is_draft_worker
            and not isinstance(self.token_to_kv_pool, NoOpMHATokenToKVPool)
        ):
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 9 of 10 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 9/10 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 778-786: Method ModelRunnerKVCacheMixin._init_pools (part 10/10)
```python
            raise RuntimeError(
                "--prefill-only-disable-kv-cache expected NoOpMHATokenToKVPool but the "
                f"runtime pool is {type(self.token_to_kv_pool).__name__}. This pool "
                "family is not yet supported by --prefill-only-disable-kv-cache. "
                "Supported configurations today: plain MHA models on CUDA with the FA "
                "(fa3/fa4) prefill backend, --is-embedding, --chunked-prefill-size=-1, "
                "--disable-radix-cache, no context-parallel attention, no HiSparse, "
                "and --kv-cache-dtype != fp4_e2m1."
            )
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._init_pools` and mainly implements init pools. The docstring states: "Initialize the memory pools." This chunk is part 10 of 10 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._init_pools`，主要用于实现 init pools 相关逻辑。 该片段是同一逻辑块的第 10/10 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 788-815: Method ModelRunnerKVCacheMixin._apply_token_constraints
```python
    def _apply_token_constraints(self: ModelRunner, token_capacity: int) -> int:
        """Apply external constraints to token capacity: user cap, PP sync.

        Page alignment is handled by the configurator, not here.
        If constraints change the value, the configurator re-runs and re-aligns.
        """
        user_limit = self.server_args.max_total_tokens

        # Apply user-specified upper bound
        if user_limit is not None:
            if user_limit > token_capacity:
                logging.warning(
                    f"max_total_tokens={user_limit} is larger than the profiled value "
                    f"{token_capacity}. Use the profiled value instead."
                )
            token_capacity = min(token_capacity, user_limit)

        # Sync across PP ranks (each may have different layer counts)
        if self.pp_size > 1:
            tensor = torch.tensor(token_capacity, dtype=torch.int64)
            torch.distributed.all_reduce(
                tensor,
                op=torch.distributed.ReduceOp.MIN,
                group=get_world_group().cpu_group,
            )
            token_capacity = tensor.item()

        return token_capacity
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._apply_token_constraints`. It takes `token_capacity` and mainly converts data into another representation. The docstring states: "Apply external constraints to token capacity: user cap, PP sync." In this range it emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._apply_token_constraints`。它接收 `token_capacity`，主要用于将数据转换为另一种表示。 在这一范围内，它会输出日志以便诊断；协调分布式通信。

### Lines 817-836: Method ModelRunnerKVCacheMixin._resolve_max_num_reqs
```python
    def _resolve_max_num_reqs(self: ModelRunner, token_capacity: int) -> int:
        """Compute max concurrent requests (per dp worker) from the finalized
        token capacity."""
        # Estimate pool size (used as upper bound when user specifies max_running_requests)
        estimated = int(token_capacity / self.model_config.context_len * 512)
        estimated = max(min(estimated, 4096), 2048)

        max_num_reqs = self.server_args.max_running_requests
        if max_num_reqs is not None:
            max_num_reqs = min(max_num_reqs // self.dp_size, estimated)
        else:
            max_num_reqs = min(estimated, token_capacity // 2)

        if self.mambaish_config is not None:
            ratio = self._calculate_mamba_ratio()
            max_num_reqs = min(
                max_num_reqs, self.server_args.max_mamba_cache_size // ratio
            )

        return max_num_reqs
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._resolve_max_num_reqs`. It takes `token_capacity` and mainly implements resolve max num reqs. The docstring states: "Compute max concurrent requests (per dp worker) from the finalized token capacity." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._resolve_max_num_reqs`。它接收 `token_capacity`，主要用于实现 resolve max num reqs 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 838-867: Method ModelRunnerKVCacheMixin._apply_memory_pool_config
```python
    def _apply_memory_pool_config(self: ModelRunner, config: MemoryPoolConfig):
        """Apply a resolved MemoryPoolConfig and initialize pools."""
        self.max_total_num_tokens = config.max_total_num_tokens
        self.max_running_requests = config.max_running_requests
        if self.is_hybrid_swa:
            self.full_max_total_num_tokens = config.full_max_total_num_tokens
            self.swa_max_total_num_tokens = config.swa_max_total_num_tokens

        # DSV4 compressed-attention pool sizes. Draft worker reuses target's
        # full/swa sizes but does NOT own c4/c128/state pools (those live on
        # the target rank only); zero them out regardless of what config holds.
        if self.is_draft_worker:
            self.c4_max_total_num_tokens = 0
            self.c128_max_total_num_tokens = 0
            self.c4_state_pool_size = 0
            self.c128_state_pool_size = 0
        else:
            self.c4_max_total_num_tokens = config.c4_max_total_num_tokens
            self.c128_max_total_num_tokens = config.c128_max_total_num_tokens
            self.c4_state_pool_size = config.c4_state_pool_size
            self.c128_state_pool_size = config.c128_state_pool_size

        # state_dtype is a DSV4 architectural constant (fp32 for c4/c128
        # state buffers); set unconditionally so draft workers have it before
        # _init_pools reads it (target path also overwrites this in the
        # configurator's resolve() for parity, harmless here).
        if is_deepseek_v4(self.model_config.hf_config):
            self.state_dtype = torch.float32

        self._init_pools()
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._apply_memory_pool_config`. It takes `config` and mainly implements apply memory pool config. The docstring states: "Apply a resolved MemoryPoolConfig and initialize pools."
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._apply_memory_pool_config`。它接收 `config`，主要用于实现 apply memory pool config 相关逻辑。

### Lines 869-894: Method ModelRunnerKVCacheMixin._resolve_memory_pool_config
```python
    def _resolve_memory_pool_config(
        self: ModelRunner, pre_model_load_memory: int
    ) -> MemoryPoolConfig:
        """Profile GPU memory and resolve all pool parameters into a config."""
        from sglang.srt.model_executor.pool_configurator import (
            create_memory_pool_configurator,
        )

        available_bytes = self._profile_available_bytes(pre_model_load_memory)
        page_size = self.server_args.page_size

        configurator = create_memory_pool_configurator(self)
        config = configurator.calculate_pool_sizes(available_bytes, page_size)

        # Apply external constraints (user cap, page alignment, PP sync)
        constrained = self._apply_token_constraints(config.max_total_num_tokens)
        if constrained != config.max_total_num_tokens:
            config = configurator.calculate_pool_sizes_from_max_tokens(
                constrained, page_size
            )

        config.max_running_requests = self._resolve_max_num_reqs(
            config.max_total_num_tokens
        )
        config.mem_fraction_static = self.server_args.mem_fraction_static
        return config
```
**EN:** This callable implements `ModelRunnerKVCacheMixin._resolve_memory_pool_config`. It takes `pre_model_load_memory` and mainly implements resolve memory pool config. The docstring states: "Profile GPU memory and resolve all pool parameters into a config." In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin._resolve_memory_pool_config`。它接收 `pre_model_load_memory`，主要用于实现 resolve memory pool config 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 896-911: Method ModelRunnerKVCacheMixin.init_memory_pool
```python
    def init_memory_pool(self: ModelRunner, pre_model_load_memory: int):
        if not self.spec_algorithm.is_none() and self.is_draft_worker:
            assert (
                self.memory_pool_config is not None
            ), "Draft worker requires memory_pool_config"
        else:
            self.memory_pool_config = self._resolve_memory_pool_config(
                pre_model_load_memory
            )

        self._apply_memory_pool_config(self.memory_pool_config)

        logger.info(
            f"Memory pool end. "
            f"avail mem={get_available_gpu_memory(self.device, self.gpu_id):.2f} GB"
        )
```
**EN:** This callable implements `ModelRunnerKVCacheMixin.init_memory_pool`. It takes `pre_model_load_memory` and mainly implements init memory pool. In this range it performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `ModelRunnerKVCacheMixin.init_memory_pool`。它接收 `pre_model_load_memory`，主要用于实现 init memory pool 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断。

## Key Concepts / 关键概念
- `MAMBA_CACHE_SIZE_MAX_RUNNING_REQUESTS_RATIO`: module constant or capability flag / 模块常量或能力标记
- `MAMBA_CACHE_V2_ADDITIONAL_RATIO_OVERLAP`: module constant or capability flag / 模块常量或能力标记
- `MAMBA_CACHE_V2_ADDITIONAL_RATIO_NO_OVERLAP`: module constant or capability flag / 模块常量或能力标记
- `ModelRunnerKVCacheMixin`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.configs.model_config`, `sglang.srt.distributed.parallel_state`, `sglang.srt.environ`, `sglang.srt.layers.dp_attention`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.deepseek_v4_memory_pool`, `sglang.srt.mem_cache.hisparse_memory_pool`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.swa_memory_pool`, `sglang.srt.utils.common`, `sglang.srt.model_executor.model_runner`, `sglang.srt.model_executor.pool_configurator` + 6 more
