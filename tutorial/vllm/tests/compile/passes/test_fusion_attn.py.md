# test_fusion_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_fusion_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_fusion_attn, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_fusion_attn 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-42)
```python
import copy

import pytest
import torch._dynamo

from tests.compile.backend import LazyInitPass, TestBackend
from tests.utils import TestFP8Layer, flat_product
from tests.v1.attention.utils import BatchSpec, create_common_attn_metadata
from vllm._custom_ops import cutlass_scaled_fp4_mm, scaled_fp4_quant
from vllm.compilation.passes.fusion.attn_quant_fusion import (
    ATTN_OP,
    AttnQuantFusionPass,
)
from vllm.compilation.passes.fusion.matcher_utils import QUANT_OPS
from vllm.compilation.passes.fx_utils import find_op_nodes
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.config import (
    AttentionConfig,
    CacheConfig,
    CompilationConfig,
    CompilationMode,
    ModelConfig,
    PassConfig,
    SchedulerConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.forward_context import get_forward_context, set_forward_context
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.kv_cache_interface import AttentionSpec, get_kv_quant_mode
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, pytest, torch._dynamo; shared test helpers from tests.compile.backend, tests.utils, tests.v1.attention.utils; and vLLM components like vllm._custom_ops, vllm.compilation.passes.fusion.attn_quant_fusion, vllm.compilation.passes.fusion.matcher_utils, vllm.compilation.passes.fx_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、pytest、torch._dynamo；共享测试辅助模块，例如 tests.compile.backend、tests.utils、tests.v1.attention.utils；vLLM 内部组件，例如 vllm._custom_ops、vllm.compilation.passes.fusion.attn_quant_fusion、vllm.compilation.passes.fusion.matcher_utils、vllm.compilation.passes.fx_utils。

### Constants and module state (lines 44-46)
```python
DEVICE_TYPE = current_platform.device_type
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE_TYPE, FP8_DTYPE, FP4_DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE_TYPE、FP8_DTYPE、FP4_DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `AttentionQuantPatternModel` (lines 49-51)
```python
class AttentionQuantPatternModel(torch.nn.Module):
    """Base model for AttentionQuantPattern fusion."""
```
**EN:** This helper class groups the state and behavior needed for AttentionQuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 AttentionQuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `AttentionQuantPatternModel.__init__` (lines 52-95)
```python
    def __init__(
        self,
        num_qo_heads: int,
        num_kv_heads: int,
        head_size: int,
        device: torch.device,
        vllm_config: VllmConfig,
        block_size: int,
        **kwargs,
    ):
        super().__init__()
        self.num_qo_heads = num_qo_heads
        self.num_kv_heads = num_kv_heads
        self.head_size = head_size
        self.device = device
        self.vllm_config = vllm_config
        self.dtype = vllm_config.model_config.dtype

        self.attn = Attention(
            num_heads=self.num_qo_heads,
            head_size=self.head_size,
            scale=1.0 / (self.head_size**0.5),
            num_kv_heads=self.num_kv_heads,
            cache_config=vllm_config.cache_config,
            prefix="model.layers.0.self_attn.attn",
        )
        self.attn._k_scale = self.attn._k_scale.to(device)
        self.attn._v_scale = self.attn._v_scale.to(device)

        self.block_size = block_size

        # Initialize attn MetadataBuilder (match Attention.get_kv_cache_spec)
        self.builder = self.attn.attn_backend.get_builder_cls()(
            kv_cache_spec=AttentionSpec(
                block_size=self.block_size,
                num_kv_heads=self.num_kv_heads,
                head_size=self.head_size,
                dtype=self.attn.kv_cache_torch_dtype,
                kv_quant_mode=get_kv_quant_mode(self.attn.kv_cache_dtype),
            ),
            layer_names=[self.attn.layer_name],
            vllm_config=self.vllm_config,
            device=self.device,
        )
```
**EN:** This method implements the initialization for `AttentionQuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `AttentionQuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `AttentionQuantPatternModel.build_attn_metadata` (lines 97-142)
```python
    def build_attn_metadata(self, batch_size: int) -> AttentionMetadata:
        """Initialize attention metadata."""

        # TODO (Rohan138) reuse utils from vllm/v1/worker/gpu/attn_utils.py

        # Create common attn metadata
        batch_spec = BatchSpec(seq_lens=[1] * batch_size, query_lens=[1] * batch_size)
        common_attn_metadata = create_common_attn_metadata(
            batch_spec, self.block_size, self.device, arange_block_indices=True
        )

        max_blocks = (max(batch_spec.seq_lens) + self.block_size - 1) // self.block_size
        num_blocks = batch_size * max_blocks

        # Fetch the attention backend and kv cache shape and stride order
        attn_backend = self.attn.attn_backend
        kv_cache_shape = attn_backend.get_kv_cache_shape(
            num_blocks, self.block_size, self.num_kv_heads, self.head_size
        )
        try:
            kv_cache_stride_order = attn_backend.get_kv_cache_stride_order()
        except (AttributeError, NotImplementedError):
            kv_cache_stride_order = tuple(range(len(kv_cache_shape)))

        kv_cache_shape = tuple(kv_cache_shape[i] for i in kv_cache_stride_order)
        inv_order = [
            kv_cache_stride_order.index(i) for i in range(len(kv_cache_stride_order))
        ]

        # Create dummy KV cache
        raw_tensor = torch.zeros(
            2 * num_blocks * self.block_size * self.num_kv_heads * self.head_size,
            dtype=self.attn.kv_cache_torch_dtype,
            device=self.device,
        )
        raw_tensor = raw_tensor.view(kv_cache_shape)
        kv_cache = raw_tensor.permute(*inv_order)

        self.attn.kv_cache = kv_cache

        # Build attn metadata
        self.attn_metadata = self.builder.build(
            common_prefix_len=0, common_attn_metadata=common_attn_metadata
        )

        return self.attn_metadata
```
**EN:** This method on `AttentionQuantPatternModel` implements attn metadata. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `AttentionQuantPatternModel` 中的这个方法实现了 attn metadata。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAttentionFp8StaticQuantPatternModel` (lines 145-149)
```python
class TestAttentionFp8StaticQuantPatternModel(AttentionQuantPatternModel):
    """Test model for AttentionFp8StaticQuantPattern fusion."""

    quant_key = kFp8StaticTensorSym
```
**EN:** This helper class groups the state and behavior needed for TestAttentionFp8StaticQuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAttentionFp8StaticQuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAttentionFp8StaticQuantPatternModel.__init__` (lines 150-172)
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        hidden_size = self.num_qo_heads * self.head_size
        self.fp8_linear = TestFP8Layer(
            weight_shape=(hidden_size, hidden_size),
            activation_quant_key=self.quant_key,
            weight_quant_key=self.quant_key,
            device=self.device,
            input_dtype=self.dtype,
        )

        w = kwargs.get("w")
        if w is not None:
            self.fp8_linear.weight = w["weight"]
            self.fp8_linear.weight_scale = w["wscale"]
            self.fp8_linear.input_scale = w["scale"]

        self.w = {
            "weight": self.fp8_linear.weight,
            "wscale": self.fp8_linear.weight_scale,
            "scale": self.fp8_linear.input_scale,
        }
```
**EN:** This method implements the initialization for `TestAttentionFp8StaticQuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAttentionFp8StaticQuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAttentionFp8StaticQuantPatternModel.forward` (lines 174-177)
```python
    def forward(self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor):
        """Forward pass that creates the pattern to be fused."""
        attn_output = self.attn(q, k, v)
        return self.fp8_linear(attn_output)
```
**EN:** This method on `TestAttentionFp8StaticQuantPatternModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAttentionFp8StaticQuantPatternModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAttentionNvfp4QuantPatternModel` (lines 180-184)
```python
class TestAttentionNvfp4QuantPatternModel(AttentionQuantPatternModel):
    """Test model for AttentionNvfp4QuantPattern fusion."""

    quant_key = kNvfp4Dynamic
```
**EN:** This helper class groups the state and behavior needed for TestAttentionNvfp4QuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAttentionNvfp4QuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAttentionNvfp4QuantPatternModel.__init__` (lines 185-204)
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        hidden_size = self.num_qo_heads * self.head_size
        self.w = kwargs.get(
            "w",
            {
                "weight": torch.randint(
                    256,
                    (hidden_size, hidden_size // 2),
                    dtype=FP4_DTYPE,
                    device=self.device,
                ),
                "wscale_swizzled": torch.randn(hidden_size, hidden_size // 16).to(
                    dtype=FP8_DTYPE, device=self.device
                ),
                "wscale": torch.tensor([500], dtype=torch.float32, device=self.device),
                "scale": torch.tensor([0.002], dtype=torch.float32, device=self.device),
            },
        )
```
**EN:** This method implements the initialization for `TestAttentionNvfp4QuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAttentionNvfp4QuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAttentionNvfp4QuantPatternModel.forward` (lines 206-219)
```python
    def forward(self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor):
        """Forward pass that creates the pattern to be fused."""
        attn_output = self.attn(q, k, v)
        quant_output, output_block_scale = scaled_fp4_quant(
            attn_output, 1 / self.w["scale"]
        )
        return cutlass_scaled_fp4_mm(
            a=quant_output,
            b=self.w["weight"],
            block_scale_a=output_block_scale,
            block_scale_b=self.w["wscale_swizzled"],
            alpha=self.w["scale"] * self.w["wscale"],
            out_dtype=attn_output.dtype,
        )
```
**EN:** This method on `TestAttentionNvfp4QuantPatternModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAttentionNvfp4QuantPatternModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 222-227)
```python
PATTERN_TEST_MODELS_FP8: list[tuple[str, type]] = []
PATTERN_TEST_MODELS_FP4: list[tuple[str, type]] = []
HEADS: list[tuple[int, int]] = []
SPLIT_ATTENTION: list[bool] = []
BACKENDS_FP8: list[AttentionBackendEnum] = []
BACKENDS_FP4: list[AttentionBackendEnum] = []
```
**EN:** This block centralizes shared constants and parameter grids, including PATTERN_TEST_MODELS_FP8, PATTERN_TEST_MODELS_FP4, HEADS, SPLIT_ATTENTION, BACKENDS_FP8. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 PATTERN_TEST_MODELS_FP8、PATTERN_TEST_MODELS_FP4、HEADS、SPLIT_ATTENTION、BACKENDS_FP8。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 229 (lines 229-255)
```python
if current_platform.is_cuda():
    HEADS = [(64, 8), (40, 8)]
    PATTERN_TEST_MODELS_FP8 = [
        (
            "RedHatAI/Meta-Llama-3.1-8B-FP8",
            TestAttentionFp8StaticQuantPatternModel,
        )
    ]
    PATTERN_TEST_MODELS_FP4 = [
        (
            "nvidia/Llama-3.1-8B-Instruct-NVFP4",
            TestAttentionNvfp4QuantPatternModel,
        )
    ]
    BACKENDS_FP8 = [AttentionBackendEnum.TRITON_ATTN, AttentionBackendEnum.FLASHINFER]
    BACKENDS_FP4 = [AttentionBackendEnum.FLASHINFER]

elif current_platform.is_rocm():
    HEADS = [(32, 8), (40, 8)]
    PATTERN_TEST_MODELS_FP8 = [
        ("amd/Llama-3.1-8B-Instruct-FP8-KV", TestAttentionFp8StaticQuantPatternModel)
    ]
    BACKENDS_FP8 = [
        AttentionBackendEnum.ROCM_AITER_UNIFIED_ATTN,
        AttentionBackendEnum.ROCM_ATTN,
        AttentionBackendEnum.TRITON_ATTN,
    ]
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `test_attention_quant_pattern` (lines 258-482)
```python
@pytest.mark.parametrize("num_qo_heads, num_kv_heads", HEADS)
@pytest.mark.parametrize("head_size", [128])
@pytest.mark.parametrize(
    "batch_size", [7, 256, 533] if current_platform.is_cuda() else [8]
)
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize(
    "backend, model_name, model_class, custom_ops",
    # Test attention+quant_fp8 fusion with custom and torch impls of QuantFP8
    list(
        flat_product(
            BACKENDS_FP8, PATTERN_TEST_MODELS_FP8, ["+quant_fp8", "-quant_fp8"]
        )
    )
    # quant_fp4 only has the custom impl
    + list(flat_product(BACKENDS_FP4, PATTERN_TEST_MODELS_FP4, [""])),
)
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(), reason="Only test ROCm or CUDA"
)
@pytest.mark.skipif(not current_platform.supports_fp8(), reason="Need FP8")
def test_attention_quant_pattern(
    num_qo_heads: int,
    num_kv_heads: int,
    head_size: int,
    batch_size: int,
    dtype: torch.dtype,
    custom_ops: str,
    model_name: str,
    model_class: type[AttentionQuantPatternModel],
    backend: AttentionBackendEnum,
    dist_init,
    monkeypatch,
    use_fresh_inductor_cache,
):
    """Test AttentionStaticQuantPattern fusion pass"""
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    if backend == AttentionBackendEnum.FLASHINFER and (
        not current_platform.is_device_capability((10, 0)) or not has_flashinfer()
# ... excerpt ...
    attn_nodes_pre = list(find_op_nodes(ATTN_OP, test_backend.graph_pre_pass))
    attn_nodes_post = list(find_op_nodes(ATTN_OP, test_backend.graph_post_pass))

    assert len(attn_nodes_pre) > 0, "Should have attention nodes before fusion"
    assert len(attn_nodes_pre) == len(attn_nodes_post), (
        "Should have same number of attention nodes before and after fusion"
    )
    assert attn_nodes_pre[0].kwargs.get("output_scale") is None, (
        "Attention should not have output_scale before fusion"
    )
    assert attn_nodes_post[0].kwargs.get("output_scale") is not None, (
        "Attention should have output_scale after fusion"
    )

    assert attn_nodes_pre[0].kwargs.get("output_block_scale") is None, (
        "Attention should not have output_block_scale before fusion"
    )

    kv_cache_dummy_dep_pre_is_none = (
        attn_nodes_pre[0].kwargs.get("kv_cache_dummy_dep") is None
    )
    kv_cache_dummy_dep_post_is_none = (
        attn_nodes_post[0].kwargs.get("kv_cache_dummy_dep") is None
    )
    assert not (kv_cache_dummy_dep_pre_is_none ^ kv_cache_dummy_dep_post_is_none), (
        "The kv_cache_dummy_dep should be consistent before and after fusion"
    )

    if quant_key.dtype == FP8_DTYPE:
        assert attn_nodes_post[0].kwargs.get("output_block_scale") is None, (
            "Attention should not have output_block_scale after FP8 fusion"
        )
    elif quant_key.dtype == FP4_DTYPE:
        assert attn_nodes_post[0].kwargs.get("output_block_scale") is not None, (
            "Attention should have output_block_scale after FP4 fusion"
        )

    # Check that results are close
    torch.testing.assert_close(result_unfused, result_fused, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies attention quant pattern. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_qo_heads, num_kv_heads, head_size, batch_size. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 attention quant pattern 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 num_qo_heads、num_kv_heads、head_size、batch_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `copy`
- `pytest`
- `torch._dynamo`
- `tests.compile.backend -> LazyInitPass, TestBackend`
- `tests.utils -> TestFP8Layer, flat_product`
- `tests.v1.attention.utils -> BatchSpec, create_common_attn_metadata`
- `vllm._custom_ops -> cutlass_scaled_fp4_mm, scaled_fp4_quant`
- `vllm.compilation.passes.fusion.attn_quant_fusion -> ATTN_OP, AttnQuantFusionPass`
- `vllm.compilation.passes.fusion.matcher_utils -> QUANT_OPS`
- `vllm.compilation.passes.fx_utils -> find_op_nodes`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> AttentionConfig, CacheConfig, CompilationConfig, CompilationMode, ModelConfig, PassConfig, SchedulerConfig, VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> get_forward_context, set_forward_context`
- `vllm.model_executor.layers.attention -> Attention`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> QuantKey, kFp8StaticTensorSym, kNvfp4Dynamic`
