# test_rope_kvcache_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_rope_kvcache_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_rope_kvcache_fusion, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_rope_kvcache_fusion 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-36)
```python
import pytest
import torch

import vllm.config
from tests.compile.backend import TestBackend
from tests.v1.attention.utils import BatchSpec, create_common_attn_metadata
from vllm._aiter_ops import is_aiter_found_and_supported, rocm_aiter_ops
from vllm.compilation.passes.fusion.matcher_utils import ROTARY_OP
from vllm.compilation.passes.fusion.rope_kvcache_fusion import RopeKVCacheFusionPass
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.compilation.passes.utility.scatter_split_replace import (
    ScatterSplitReplacementPass,
)
from vllm.compilation.passes.utility.split_coalescing import SplitCoalescingPass
from vllm.config import (
    CacheConfig,
    CompilationConfig,
    CompilationMode,
    ModelConfig,
    PassConfig,
    VllmConfig,
)
from vllm.forward_context import get_forward_context, set_forward_context
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding
from vllm.platforms import current_platform
from vllm.utils.torch_utils import _encode_layer_name
from vllm.v1.attention.backend import (
    AttentionBackend,
    CommonAttentionMetadata,
)
from vllm.v1.attention.backends.registry import AttentionBackendEnum
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend, tests.v1.attention.utils; and vLLM components like vllm.config, vllm._aiter_ops, vllm.compilation.passes.fusion.matcher_utils, vllm.compilation.passes.fusion.rope_kvcache_fusion.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.v1.attention.utils；vLLM 内部组件，例如 vllm.config、vllm._aiter_ops、vllm.compilation.passes.fusion.matcher_utils、vllm.compilation.passes.fusion.rope_kvcache_fusion。

### Constants and module state (lines 38-40)
```python
INDEX_SELECT_OP = torch.ops.aten.index.Tensor
VLLM_UNIFIED_KV_CACHE_UPDATE_OP = torch.ops.vllm.unified_kv_cache_update
FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This block centralizes shared constants and parameter grids, including INDEX_SELECT_OP, VLLM_UNIFIED_KV_CACHE_UPDATE_OP, FP8_DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 INDEX_SELECT_OP、VLLM_UNIFIED_KV_CACHE_UPDATE_OP、FP8_DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `QKRoPEKVCacheTestModel` (lines 43-43)
```python
class QKRoPEKVCacheTestModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for QKRoPEKVCacheTestModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 QKRoPEKVCacheTestModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `QKRoPEKVCacheTestModel.__init__` (lines 44-109)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        attn_backend: AttentionBackendEnum,
        num_heads: int,
        num_kv_heads: int,
        head_size: int,
        is_neox: bool,
        dtype: torch.dtype,
        device: torch.device,
        prefix: str = "model.layers.0.self_attn.attn",
    ):
        super().__init__()
        self.num_heads = num_heads
        self.num_kv_heads = num_kv_heads
        self.head_size = head_size
        self.block_size = vllm_config.cache_config.block_size
        self.q_size = num_heads * head_size
        self.kv_size = num_kv_heads * head_size
        self.is_neox = is_neox
        self.dtype = dtype
        self.device = device
        self.layer_name = prefix

        self.rotary_emb = RotaryEmbedding(
            head_size,
            rotary_dim=head_size,
            max_position_embeddings=4096,
            base=10000,
            is_neox_style=is_neox,
            dtype=self.dtype,
        )

        # Whether to check for the RoPE custom op or component index_select
        self.enable_rope_custom_op = self.rotary_emb.enabled()

        # Register layer metadata for the fusion pass via Attention.
        self.attn = Attention(
            num_heads=num_heads,
            head_size=head_size,
            scale=1.0 / head_size**0.5,
            num_kv_heads=num_kv_heads,
            cache_config=vllm_config.cache_config,
            quant_config=vllm_config.quant_config,
            prefix=prefix,
            attn_backend=attn_backend.get_class(),
        )
        self.attn_backend: type[AttentionBackend] = self.attn.get_attn_backend()
        assert not self.attn_backend.forward_includes_kv_cache_update, (
            f"Attention backend {self.attn_backend} does not support fuse_rope_kvcache."
        )
        self.attn._k_scale = self.attn._k_scale.to(device)
        self.attn._v_scale = self.attn._v_scale.to(device)

        kv_cache_dtype_str = vllm_config.cache_config.cache_dtype
        self.kv_cache_dtype = (
            FP8_DTYPE if kv_cache_dtype_str.startswith("fp8") else self.dtype
        )

        # Initialize attn MetadataBuilder
        self.builder = self.attn_backend.get_builder_cls()(
            kv_cache_spec=self.attn.get_kv_cache_spec(vllm_config),
            layer_names=[self.attn.layer_name],
            vllm_config=vllm_config,
            device=device,
        )
```
**EN:** This method implements the initialization for `QKRoPEKVCacheTestModel`. assertions at the end lock in the intended behavior or graph shape.
**CN:** 这个方法实现了 `QKRoPEKVCacheTestModel` 的初始化逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Method `QKRoPEKVCacheTestModel.build_attn_metadata` (lines 111-152)
```python
    def build_attn_metadata(self, batch_size: int) -> CommonAttentionMetadata:
        """Initialize attention metadata."""
        # Create common attn metadata
        batch_spec = BatchSpec(seq_lens=[1] * batch_size, query_lens=[1] * batch_size)
        common_attn_metadata = create_common_attn_metadata(
            batch_spec, self.block_size, self.device, arange_block_indices=True
        )

        max_blocks = (max(batch_spec.seq_lens) + self.block_size - 1) // self.block_size
        num_blocks = batch_size * max_blocks

        # Fetch the attention backend and kv cache shape and stride order
        kv_cache_shape = self.attn_backend.get_kv_cache_shape(
            num_blocks, self.block_size, self.num_kv_heads, self.head_size
        )
        try:
            kv_cache_stride_order = self.attn_backend.get_kv_cache_stride_order()
        except (AttributeError, NotImplementedError):
            kv_cache_stride_order = tuple(range(len(kv_cache_shape)))

        kv_cache_shape = tuple(kv_cache_shape[i] for i in kv_cache_stride_order)
        inv_order = [
            kv_cache_stride_order.index(i) for i in range(len(kv_cache_stride_order))
        ]

        # Create dummy KV cache
        raw_tensor = torch.zeros(
            2 * num_blocks * self.block_size * self.num_kv_heads * self.head_size,
            dtype=self.kv_cache_dtype,
            device=self.device,
        )
        raw_tensor = raw_tensor.view(kv_cache_shape)
        kv_cache = raw_tensor.permute(*inv_order)

        self.attn.kv_cache = kv_cache

        # Build attn metadata
        attn_metadata = self.builder.build(
            common_prefix_len=0, common_attn_metadata=common_attn_metadata
        )

        return attn_metadata
```
**EN:** This method on `QKRoPEKVCacheTestModel` implements attn metadata. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKRoPEKVCacheTestModel` 中的这个方法实现了 attn metadata。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `QKRoPEKVCacheTestModel.forward` (lines 154-169)
```python
    def forward(
        self, qkv: torch.Tensor, positions: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
        # Create copy so inplace ops do not modify the original tensors
        qkv = qkv.clone()
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)

        # Instead of a full forward pass, match only the KV cache update op here
        q = q.view(-1, self.num_heads, self.head_size)
        k = k.view(-1, self.num_kv_heads, self.head_size)
        v = v.view(-1, self.num_kv_heads, self.head_size)
        kv_cache_dummy_dep = torch.ops.vllm.unified_kv_cache_update(
            k, v, _encode_layer_name(self.layer_name)
        )
        return q, k, v, kv_cache_dummy_dep
```
**EN:** This method on `QKRoPEKVCacheTestModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKRoPEKVCacheTestModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `QKRoPEKVCacheTestModel.ops_in_model_before` (lines 171-181)
```python
    def ops_in_model_before(self) -> list[torch._ops.OpOverload]:
        ops = []
        if self.enable_rope_custom_op:
            if rocm_aiter_ops.is_triton_rotary_embed_enabled():
                ops.append(torch.ops.vllm.rocm_aiter_triton_rotary_embedding.default)
            else:
                ops.append(ROTARY_OP)
        else:
            ops.append(INDEX_SELECT_OP)
        ops.append(torch.ops.vllm.unified_kv_cache_update.default)
        return ops
```
**EN:** This method on `QKRoPEKVCacheTestModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKRoPEKVCacheTestModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `QKRoPEKVCacheTestModel.ops_in_model_after` (lines 183-184)
```python
    def ops_in_model_after(self) -> list[torch._ops.OpOverload]:
        return [torch.ops.vllm.fused_rope_and_unified_kv_cache_update.default]
```
**EN:** This method on `QKRoPEKVCacheTestModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKRoPEKVCacheTestModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_rope_kvcache_fusion` (lines 187-328)
```python
@pytest.mark.parametrize(
    "attn_backend",
    [
        AttentionBackendEnum.ROCM_AITER_UNIFIED_ATTN,
        AttentionBackendEnum.TRITON_ATTN,
        AttentionBackendEnum.ROCM_ATTN,
        AttentionBackendEnum.ROCM_AITER_FA,
    ],
)
@pytest.mark.parametrize("enable_rope_custom_op", [True])  # [True, False])
@pytest.mark.parametrize("enable_aiter_triton_rope", [True, False])
@pytest.mark.parametrize("num_heads", [64])
@pytest.mark.parametrize("num_kv_heads", [8])
@pytest.mark.parametrize("head_size", [64])
@pytest.mark.parametrize("block_size", [16])
@pytest.mark.parametrize("is_neox", [True, False])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8"])
@pytest.mark.skipif(
    not is_aiter_found_and_supported(),
    reason="Only test on ROCm with AITER installed and supported",
)
def test_rope_kvcache_fusion(
    attn_backend: AttentionBackendEnum,
    enable_rope_custom_op: bool,
    enable_aiter_triton_rope: bool,
    num_heads: int,
    num_kv_heads: int,
    head_size: int,
    block_size: int,
    is_neox: bool,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    monkeypatch: pytest.MonkeyPatch,
):
    torch.set_default_device("cuda")
    torch.set_default_dtype(dtype)
    torch.manual_seed(0)

    custom_ops: list[str] = []
# ... excerpt ...
            q_unfused, k_unfused, v_unfused, dummy = model(qkv_unfused, pos_unfused)
            attn_layer = forward_context.no_compile_layers[model.layer_name]
            kv_cache_unfused = attn_layer.kv_cache
        del dummy

        torch._dynamo.mark_dynamic(qkv, 0)
        torch._dynamo.mark_dynamic(pos, 0)
        with set_forward_context(None, vllm_config):
            model_fused = torch.compile(model, backend=backend)
            forward_context = get_forward_context()
            attn_metadata = model_fused.build_attn_metadata(T)
            forward_context.slot_mapping = {
                model.layer_name: attn_metadata.slot_mapping
            }
            q_fused, k_fused, v_fused, dummy = model_fused(qkv, pos)
            attn_layer = forward_context.no_compile_layers[model.layer_name]
            kv_cache_fused = attn_layer.kv_cache
        del dummy

        assert fusion_pass.matched_count == 1

        backend.check_before_ops(model.ops_in_model_before())
        backend.check_after_ops(model.ops_in_model_after())

        if dtype == torch.float16:
            ATOL, RTOL = (2e-3, 2e-3)
        else:
            ATOL, RTOL = (1e-2, 1e-2)

        torch.testing.assert_close(q_unfused, q_fused, atol=ATOL, rtol=RTOL)
        torch.testing.assert_close(k_unfused, k_fused, atol=ATOL, rtol=RTOL)
        torch.testing.assert_close(v_unfused, v_fused, atol=ATOL, rtol=RTOL)
        # Cannot compare fp8_* directly here, cast to model dtype instead
        torch.testing.assert_close(
            kv_cache_unfused.view(dtype),
            kv_cache_fused.view(dtype),
            atol=ATOL,
            rtol=RTOL,
        )
```
**EN:** This pytest case verifies rope kvcache fusion. It is parameterized across 10 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as attn_backend, enable_rope_custom_op, enable_aiter_triton_rope, num_heads. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 rope kvcache fusion 的行为。 它通过 10 组参数化输入覆盖多种场景；它会使用诸如 attn_backend、enable_rope_custom_op、enable_aiter_triton_rope、num_heads 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.config`
- `tests.compile.backend -> TestBackend`
- `tests.v1.attention.utils -> BatchSpec, create_common_attn_metadata`
- `vllm._aiter_ops -> is_aiter_found_and_supported, rocm_aiter_ops`
- `vllm.compilation.passes.fusion.matcher_utils -> ROTARY_OP`
- `vllm.compilation.passes.fusion.rope_kvcache_fusion -> RopeKVCacheFusionPass`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.compilation.passes.utility.scatter_split_replace -> ScatterSplitReplacementPass`
- `vllm.compilation.passes.utility.split_coalescing -> SplitCoalescingPass`
- `vllm.config -> CacheConfig, CompilationConfig, CompilationMode, ModelConfig, PassConfig, VllmConfig`
- `vllm.forward_context -> get_forward_context, set_forward_context`
- `vllm.model_executor.layers.attention -> Attention`
- `vllm.model_executor.layers.rotary_embedding -> RotaryEmbedding`
