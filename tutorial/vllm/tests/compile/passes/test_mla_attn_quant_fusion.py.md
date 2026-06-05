# test_mla_attn_quant_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_mla_attn_quant_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_mla_attn_quant_fusion, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_mla_attn_quant_fusion 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-50)
```python
import copy

import pytest
import torch._dynamo

from tests.compile.backend import LazyInitPass, TestBackend
from tests.utils import TestFP8Layer, flat_product
from tests.v1.attention.utils import BatchSpec, create_common_attn_metadata
from vllm._custom_ops import cutlass_scaled_fp4_mm, scaled_fp4_quant
from vllm.compilation.passes.fusion.matcher_utils import QUANT_OPS
from vllm.compilation.passes.fusion.mla_attn_quant_fusion import (
    MLA_ATTN_OP,
    MLAAttnQuantFusionPass,
)
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
from vllm.model_executor.kernels.linear.scaled_mm.cutlass import (
    CutlassFp8BlockScaledMMKernel,
)
from vllm.model_executor.layers.attention import MLAAttention
from vllm.model_executor.layers.linear import ColumnParallelLinear
from vllm.model_executor.layers.quantization.fp8 import Fp8Config
from vllm.model_executor.layers.quantization.modelopt import ModelOptNvFp4Config
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    QuantKey,
    create_fp8_quant_key,
    kFp8Dynamic128Sym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.platforms import current_platform
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.kv_cache_interface import MLAAttentionSpec
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, pytest, torch._dynamo; shared test helpers from tests.compile.backend, tests.utils, tests.v1.attention.utils; and vLLM components like vllm._custom_ops, vllm.compilation.passes.fusion.matcher_utils, vllm.compilation.passes.fusion.mla_attn_quant_fusion, vllm.compilation.passes.fx_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、pytest、torch._dynamo；共享测试辅助模块，例如 tests.compile.backend、tests.utils、tests.v1.attention.utils；vLLM 内部组件，例如 vllm._custom_ops、vllm.compilation.passes.fusion.matcher_utils、vllm.compilation.passes.fusion.mla_attn_quant_fusion、vllm.compilation.passes.fx_utils。

### Constants and module state (lines 52-54)
```python
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
DEVICE_TYPE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including FP8_DTYPE, FP4_DTYPE, DEVICE_TYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP8_DTYPE、FP4_DTYPE、DEVICE_TYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `MLAAttentionQuantPatternModel` (lines 57-59)
```python
class MLAAttentionQuantPatternModel(torch.nn.Module):
    """Base model for MLA AttentionQuantPattern fusion."""
```
**EN:** This helper class groups the state and behavior needed for MLAAttentionQuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MLAAttentionQuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MLAAttentionQuantPatternModel.__init__` (lines 60-132)
```python
    def __init__(
        self,
        num_heads: int,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        v_head_dim: int,
        kv_lora_rank: int,
        kv_cache_dtype: torch.dtype,
        device: torch.device,
        vllm_config: VllmConfig,
        **kwargs,
    ):
        super().__init__()
        self.num_heads = num_heads
        self.qk_nope_head_dim = qk_nope_head_dim
        self.qk_rope_head_dim = qk_rope_head_dim
        self.qk_head_dim = qk_nope_head_dim + qk_rope_head_dim
        self.v_head_dim = v_head_dim
        self.kv_lora_rank = kv_lora_rank
        self.output_dim = num_heads * v_head_dim
        self.head_size = kv_lora_rank + qk_rope_head_dim
        self.kv_cache_dtype = kv_cache_dtype
        self.device = device
        self.vllm_config = vllm_config
        self.dtype = vllm_config.model_config.dtype

        kv_b_proj = ColumnParallelLinear(
            input_size=kv_lora_rank,
            output_size=num_heads * (qk_nope_head_dim + v_head_dim),
            bias=False,
            prefix="model.layers.0.self_attn.kv_b_proj",
        ).to(device)
        kv_b_proj_weight = kwargs.get("kv_b_proj_weight")
        if kv_b_proj_weight is not None:
            kv_b_proj.weight.data.copy_(kv_b_proj_weight)
        else:
            kv_b_proj.weight.data.normal_()

        # Create MLAAttention
        self.mla_attn = MLAAttention(
            num_heads=num_heads,
            scale=1.0 / (self.qk_head_dim**0.5),
            qk_nope_head_dim=qk_nope_head_dim,
            qk_rope_head_dim=qk_rope_head_dim,
            v_head_dim=v_head_dim,
            q_lora_rank=None,
            kv_lora_rank=kv_lora_rank,
            kv_b_proj=kv_b_proj,
            cache_config=vllm_config.cache_config,
            quant_config=self.quant_config,
            prefix="model.layers.0.self_attn.attn",
        )
        self.mla_attn._k_scale = self.mla_attn._k_scale.to(device)
        self.mla_attn._v_scale = self.mla_attn._v_scale.to(device)

        # Initialize W_UK_T and W_UV from kv_b_proj weights
        self.mla_attn.process_weights_after_loading(torch.get_default_dtype())
        self.kv_b_proj_weight = kv_b_proj.weight.data.clone()

        self.block_size = 16

        # Initialize MLA MetadataBuilder
        self.builder = self.mla_attn.attn_backend.get_builder_cls()(
            kv_cache_spec=MLAAttentionSpec(
                block_size=self.block_size,
                num_kv_heads=1,
                head_size=self.head_size,
                dtype=self.kv_cache_dtype,
            ),
            layer_names=[self.mla_attn.layer_name],
            vllm_config=self.vllm_config,
            device=self.device,
        )
```
**EN:** This method implements the initialization for `MLAAttentionQuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `MLAAttentionQuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MLAAttentionQuantPatternModel.build_attn_metadata` (lines 134-179)
```python
    def build_attn_metadata(self, batch_size: int) -> AttentionMetadata:
        """Initialize MLA attention metadata.

        NOTE: Uses decode-only batch (query_len=1 per request). The prefill
        (forward_mha) path is not separately tested here because it requires
        FlashAttention availability and different input tensor shapes. The
        quant logic in forward_impl is identical for both paths — it quantizes
        the full output[:num_actual_toks] buffer after both forward_mha and
        forward_mqa have written their results.
        """

        batch_spec = BatchSpec(seq_lens=[1] * batch_size, query_lens=[1] * batch_size)
        common_attn_metadata = create_common_attn_metadata(
            batch_spec, self.block_size, self.device, arange_block_indices=True
        )

        max_blocks = (max(batch_spec.seq_lens) + self.block_size - 1) // self.block_size
        num_blocks = batch_size * max_blocks

        # MLA KV cache is 3D: (num_blocks, block_size, head_size)
        attn_backend = self.mla_attn.attn_backend
        kv_cache_shape = attn_backend.get_kv_cache_shape(
            num_blocks, self.block_size, 1, self.head_size
        )
        try:
            kv_cache_stride_order = attn_backend.get_kv_cache_stride_order()
        except (AttributeError, NotImplementedError):
            kv_cache_stride_order = tuple(range(len(kv_cache_shape)))

        ordered_shape = tuple(kv_cache_shape[i] for i in kv_cache_stride_order)
        inv_order = [
            kv_cache_stride_order.index(i) for i in range(len(kv_cache_stride_order))
        ]

        raw_tensor = torch.zeros(
            ordered_shape, dtype=self.kv_cache_dtype, device=self.device
        )
        kv_cache = raw_tensor.permute(*inv_order)

        self.mla_attn.kv_cache = kv_cache

        self.attn_metadata = self.builder.build(
            common_prefix_len=0, common_attn_metadata=common_attn_metadata
        )

        return self.attn_metadata
```
**EN:** This method on `MLAAttentionQuantPatternModel` implements attn metadata. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MLAAttentionQuantPatternModel` 中的这个方法实现了 attn metadata。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestMLAAttentionFp8StaticQuantPatternModel` (lines 182-187)
```python
class TestMLAAttentionFp8StaticQuantPatternModel(MLAAttentionQuantPatternModel):
    """Test model for MLA Attention + FP8 static quant fusion."""

    quant_key = kFp8StaticTensorSym
    quant_config = Fp8Config()
```
**EN:** This helper class groups the state and behavior needed for TestMLAAttentionFp8StaticQuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestMLAAttentionFp8StaticQuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestMLAAttentionFp8StaticQuantPatternModel.__init__` (lines 188-209)
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.fp8_linear = TestFP8Layer(
            weight_shape=(self.output_dim, self.output_dim),
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
**EN:** This method implements the initialization for `TestMLAAttentionFp8StaticQuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestMLAAttentionFp8StaticQuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMLAAttentionFp8StaticQuantPatternModel.forward` (lines 211-224)
```python
    def forward(
        self,
        q: torch.Tensor,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
    ):
        """Forward pass that creates the MLA attention + FP8 quant pattern."""
        attn_output = self.mla_attn(
            q,
            kv_c_normed,
            k_pe,
            output_shape=(q.shape[0], self.output_dim),
        )
        return self.fp8_linear(attn_output)
```
**EN:** This method on `TestMLAAttentionFp8StaticQuantPatternModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMLAAttentionFp8StaticQuantPatternModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestMLAAttentionNvfp4QuantPatternModel` (lines 227-236)
```python
class TestMLAAttentionNvfp4QuantPatternModel(MLAAttentionQuantPatternModel):
    """Test model for MLA Attention + NVFP4 quant fusion."""

    quant_key = kNvfp4Dynamic
    quant_config = ModelOptNvFp4Config(
        is_checkpoint_nvfp4_serialized=False,
        kv_cache_quant_algo=None,
        exclude_modules=[],
    )
```
**EN:** This helper class groups the state and behavior needed for TestMLAAttentionNvfp4QuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestMLAAttentionNvfp4QuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestMLAAttentionNvfp4QuantPatternModel.__init__` (lines 237-255)
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.w = kwargs.get(
            "w",
            {
                "weight": torch.randint(
                    256,
                    (self.output_dim, self.output_dim // 2),
                    dtype=FP4_DTYPE,
                    device=self.device,
                ),
                "wscale_swizzled": torch.randn(
                    self.output_dim, self.output_dim // 16
                ).to(dtype=FP8_DTYPE, device=self.device),
                "wscale": torch.tensor([500], dtype=torch.float32, device=self.device),
                "scale": torch.tensor([0.002], dtype=torch.float32, device=self.device),
            },
        )
```
**EN:** This method implements the initialization for `TestMLAAttentionNvfp4QuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestMLAAttentionNvfp4QuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMLAAttentionNvfp4QuantPatternModel.forward` (lines 257-280)
```python
    def forward(
        self,
        q: torch.Tensor,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
    ):
        """Forward pass that creates the MLA attention + NVFP4 quant pattern."""
        attn_output = self.mla_attn(
            q,
            kv_c_normed,
            k_pe,
            output_shape=(q.shape[0], self.output_dim),
        )
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
**EN:** This method on `TestMLAAttentionNvfp4QuantPatternModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMLAAttentionNvfp4QuantPatternModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestMLAAttentionFp8GroupQuantPatternModel` (lines 283-291)
```python
class TestMLAAttentionFp8GroupQuantPatternModel(MLAAttentionQuantPatternModel):
    """Test model for MLA Attention + per-group FP8 (block quant) fusion."""

    quant_key = kFp8Dynamic128Sym
    quant_config = Fp8Config(
        is_checkpoint_fp8_serialized=True,
        weight_block_size=[128, 128],
    )
```
**EN:** This helper class groups the state and behavior needed for TestMLAAttentionFp8GroupQuantPatternModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestMLAAttentionFp8GroupQuantPatternModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestMLAAttentionFp8GroupQuantPatternModel.__init__` (lines 292-326)
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(128, 128)
        )
        device = kwargs.get("device", torch.device("cuda:0"))

        # Subclass to set weight_block_size before process_weights_after_loading
        class _BlockFP8Layer(TestFP8Layer):
            def __init__(self, *a, **kw):
                self.weight_block_size = [128, 128]
                super().__init__(*a, **kw)

        # Force CutlassFp8BlockScaledMMKernel to ensure the graph uses
        # per_token_group_fp8_quant (not the deepgemm packed variant).
        self.block_fp8_linear = _BlockFP8Layer(
            weight_shape=(self.output_dim, self.output_dim),
            activation_quant_key=self.quant_key,
            weight_quant_key=weight_quant_key,
            input_dtype=self.dtype,
            device=device,
            force_kernel=CutlassFp8BlockScaledMMKernel,
        )

        w = kwargs.get("w")
        if w is not None:
            self.block_fp8_linear.weight = w["weight"]
            # Block-wise uses weight_scale_inv, not weight_scale
            self.block_fp8_linear.weight_scale_inv = w["wscale"]

        self.w = {
            "weight": self.block_fp8_linear.weight,
            "wscale": self.block_fp8_linear.weight_scale_inv,
        }
```
**EN:** This method implements the initialization for `TestMLAAttentionFp8GroupQuantPatternModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestMLAAttentionFp8GroupQuantPatternModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMLAAttentionFp8GroupQuantPatternModel.forward` (lines 328-341)
```python
    def forward(
        self,
        q: torch.Tensor,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
    ):
        """Forward pass: MLA attention -> block FP8 linear (group quant)."""
        attn_output = self.mla_attn(
            q,
            kv_c_normed,
            k_pe,
            output_shape=(q.shape[0], self.output_dim),
        )
        return self.block_fp8_linear(attn_output)
```
**EN:** This method on `TestMLAAttentionFp8GroupQuantPatternModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMLAAttentionFp8GroupQuantPatternModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `is_nvfp4_supported` (lines 344-345)
```python
def is_nvfp4_supported():
    return current_platform.has_device_capability(100)
```
**EN:** This helper function implements the shared logic for is nvfp4 supported. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 is nvfp4 supported 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 349-354)
```python
MLA_DIMS: list[tuple[int, int, int, int, int]] = []
PATTERN_TEST_MODELS_MLA_FP8: list[tuple[str, type]] = []
PATTERN_TEST_MODELS_MLA_GROUP_FP8: list[tuple[str, type]] = []
PATTERN_TEST_MODELS_MLA_FP4: list[tuple[str, type]] = []
BACKENDS_MLA_FP8: list[AttentionBackendEnum] = []
BACKENDS_MLA_FP4: list[AttentionBackendEnum] = []
```
**EN:** This block centralizes shared constants and parameter grids, including MLA_DIMS, PATTERN_TEST_MODELS_MLA_FP8, PATTERN_TEST_MODELS_MLA_GROUP_FP8, PATTERN_TEST_MODELS_MLA_FP4, BACKENDS_MLA_FP8. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MLA_DIMS、PATTERN_TEST_MODELS_MLA_FP8、PATTERN_TEST_MODELS_MLA_GROUP_FP8、PATTERN_TEST_MODELS_MLA_FP4、BACKENDS_MLA_FP8。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 356 (lines 356-378)
```python
if current_platform.is_cuda():
    # (num_heads, qk_nope_head_dim, qk_rope_head_dim, v_head_dim, kv_lora_rank)
    MLA_DIMS = [(16, 128, 64, 128, 512)]
    PATTERN_TEST_MODELS_MLA_FP8 = [
        (
            "deepseek-ai/DeepSeek-V2-Lite",
            TestMLAAttentionFp8StaticQuantPatternModel,
        )
    ]
    PATTERN_TEST_MODELS_MLA_GROUP_FP8 = [
        (
            "deepseek-ai/DeepSeek-V3",
            TestMLAAttentionFp8GroupQuantPatternModel,
        )
    ]
    PATTERN_TEST_MODELS_MLA_FP4 = [
        (
            "deepseek-ai/DeepSeek-V2-Lite",
            TestMLAAttentionNvfp4QuantPatternModel,
        )
    ]
    BACKENDS_MLA_FP8 = [AttentionBackendEnum.TRITON_MLA]
    BACKENDS_MLA_FP4 = [AttentionBackendEnum.TRITON_MLA]
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `test_mla_attention_quant_pattern` (lines 381-587)
```python
@pytest.mark.parametrize(
    "num_heads, qk_nope_head_dim, qk_rope_head_dim, v_head_dim, kv_lora_rank",
    MLA_DIMS,
)
@pytest.mark.parametrize("batch_size", [7, 256] if current_platform.is_cuda() else [8])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize(
    "backend, model_name, model_class, custom_ops",
    list(
        flat_product(
            BACKENDS_MLA_FP8,
            PATTERN_TEST_MODELS_MLA_FP8,
            ["+quant_fp8", "-quant_fp8"],
        )
    )
    + list(
        flat_product(
            BACKENDS_MLA_FP8,
            PATTERN_TEST_MODELS_MLA_GROUP_FP8,
            ["+quant_fp8"],
        )
    )
    + list(flat_product(BACKENDS_MLA_FP4, PATTERN_TEST_MODELS_MLA_FP4, [""])),
)
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(), reason="Only test ROCm or CUDA"
)
@pytest.mark.skipif(not current_platform.supports_fp8(), reason="Need FP8")
def test_mla_attention_quant_pattern(
    num_heads: int,
    qk_nope_head_dim: int,
    qk_rope_head_dim: int,
    v_head_dim: int,
    kv_lora_rank: int,
    batch_size: int,
    dtype: torch.dtype,
    custom_ops: str,
    model_name: str,
    model_class: type[MLAAttentionQuantPatternModel],
    backend: AttentionBackendEnum,
# ... excerpt ...
    is_per_group = quant_key.scale.group_shape.is_per_group()
    quant_op = (
        torch.ops.aten.reciprocal
        if "-quant_fp8" in custom_ops_list
        else QUANT_OPS[quant_key]
    )
    test_backend.check_before_ops([quant_op], fully_replaced=is_per_group)

    assert attn_pass.pass_.matched_count == sum(attn_fusion_supported)

    # Check MLA attention ops in the graph
    attn_nodes_pre = list(find_op_nodes(MLA_ATTN_OP, test_backend.graph_pre_pass))
    attn_nodes_post = list(find_op_nodes(MLA_ATTN_OP, test_backend.graph_post_pass))

    assert len(attn_nodes_pre) > 0, "Should have MLA attention nodes before fusion"
    assert len(attn_nodes_pre) == len(attn_nodes_post), (
        "Should have same number of MLA attention nodes before and after fusion"
    )

    # Before fusion: neither scale should be set
    assert attn_nodes_pre[0].kwargs.get("output_scale") is None
    assert attn_nodes_pre[0].kwargs.get("output_block_scale") is None

    # After fusion: derive expected scale presence from quant_key properties.
    # - output_scale: present for static quant or non-FP8 (NVFP4 carries input_scale)
    # - output_block_scale: present when quant uses per-group/block scaling
    has_output_scale = attn_nodes_post[0].kwargs.get("output_scale") is not None
    has_block_scale = attn_nodes_post[0].kwargs.get("output_block_scale") is not None

    expects_output_scale = quant_key.scale.static or quant_key.dtype != FP8_DTYPE
    assert has_output_scale == expects_output_scale, (
        f"output_scale: expected present={expects_output_scale}, got {has_output_scale}"
    )
    assert has_block_scale == is_per_group, (
        f"output_block_scale: expected present={is_per_group}, got {has_block_scale}"
    )

    # Check numerical correctness
    torch.testing.assert_close(result_unfused, result_fused, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies mla attention quant pattern. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_heads, qk_nope_head_dim, qk_rope_head_dim, v_head_dim. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 mla attention quant pattern 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_heads、qk_nope_head_dim、qk_rope_head_dim、v_head_dim 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

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
- `vllm.compilation.passes.fusion.matcher_utils -> QUANT_OPS`
- `vllm.compilation.passes.fusion.mla_attn_quant_fusion -> MLA_ATTN_OP, MLAAttnQuantFusionPass`
- `vllm.compilation.passes.fx_utils -> find_op_nodes`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> AttentionConfig, CacheConfig, CompilationConfig, CompilationMode, ModelConfig, PassConfig, SchedulerConfig, VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> get_forward_context, set_forward_context`
- `vllm.model_executor.kernels.linear.scaled_mm.cutlass -> CutlassFp8BlockScaledMMKernel`
- `vllm.model_executor.layers.attention -> MLAAttention`
