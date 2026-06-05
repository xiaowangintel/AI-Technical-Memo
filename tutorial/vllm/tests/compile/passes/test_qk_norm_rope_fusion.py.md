# test_qk_norm_rope_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_qk_norm_rope_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_qk_norm_rope_fusion, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_qk_norm_rope_fusion 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-32)
```python
import pytest
import torch
from torch._ops import OpOverload, OpOverloadPacket

from tests.compile.backend import TestBackend
from vllm.compilation.passes.fusion.matcher_utils import (
    FLASHINFER_ROTARY_OP,
    ROTARY_OP,
)
from vllm.compilation.passes.fusion.qk_norm_rope_fusion import (
    FUSED_QK_ROPE_OP,
    QKNormRoPEFusionPass,
)
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.compilation.passes.utility.split_coalescing import SplitCoalescingPass
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    ModelConfig,
    PassConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding
from vllm.platforms import current_platform
from vllm.v1.attention.backend import AttentionType
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch, torch._ops; shared test helpers from tests.compile.backend; and vLLM components like vllm.compilation.passes.fusion.matcher_utils, vllm.compilation.passes.fusion.qk_norm_rope_fusion, vllm.compilation.passes.utility.noop_elimination, vllm.compilation.passes.utility.post_cleanup.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch、torch._ops；共享测试辅助模块，例如 tests.compile.backend；vLLM 内部组件，例如 vllm.compilation.passes.fusion.matcher_utils、vllm.compilation.passes.fusion.qk_norm_rope_fusion、vllm.compilation.passes.utility.noop_elimination、vllm.compilation.passes.utility.post_cleanup。

### Constants and module state (lines 34-35)
```python
RSQRT_OP = torch.ops.aten.rsqrt.default
INDEX_SELECT_OP = torch.ops.aten.index.Tensor
```
**EN:** This block centralizes shared constants and parameter grids, including RSQRT_OP, INDEX_SELECT_OP. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 RSQRT_OP、INDEX_SELECT_OP。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `QKNormRoPETestModel` (lines 38-38)
```python
class QKNormRoPETestModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for QKNormRoPETestModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 QKNormRoPETestModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `QKNormRoPETestModel.__init__` (lines 39-85)
```python
    def __init__(
        self,
        *,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        eps: float,
        is_neox: bool,
        vllm_config: VllmConfig,
        dtype: torch.dtype,
        test_scattered_split: bool = False,
        prefix: str = "model.layers.0.self_attn.attn",
    ) -> None:
        super().__init__()
        self.num_heads = num_heads
        self.num_kv_heads = num_kv_heads
        self.head_dim = head_dim
        self.q_size = num_heads * head_dim
        self.kv_size = num_kv_heads * head_dim
        self.rotary_dim = head_dim
        self.eps = eps
        self.dtype = dtype

        # Register layer metadata for the fusion pass via Attention.
        self.attn = Attention(
            num_heads=self.num_heads,
            head_size=self.head_dim,
            scale=1.0 / self.head_dim**0.5,
            num_kv_heads=self.num_kv_heads,
            cache_config=vllm_config.cache_config,
            prefix=prefix,
            attn_type=AttentionType.DECODER,
        )

        self.q_norm = RMSNorm(self.head_dim, eps=self.eps)
        self.k_norm = RMSNorm(self.head_dim, eps=self.eps)
        self.rotary_emb = RotaryEmbedding(
            self.head_dim,
            rotary_dim=self.rotary_dim,
            max_position_embeddings=4096,
            base=10000,
            is_neox_style=is_neox,
            dtype=self.dtype,
        )
        self.test_scattered_split = test_scattered_split
        self.enable_rms_norm_custom_op = self.q_norm.enabled()
        self.enable_rope_custom_op = self.rotary_emb.enabled()
```
**EN:** This method implements the initialization for `QKNormRoPETestModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `QKNormRoPETestModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `QKNormRoPETestModel.forward` (lines 87-101)
```python
    def forward(self, qkv: torch.Tensor, positions: torch.Tensor):
        if self.test_scattered_split:
            q, _, _ = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
            _, k, _ = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
            _, _, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        else:
            q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q_by_head = q.view(*q.shape[:-1], q.shape[-1] // self.head_dim, self.head_dim)
        q_by_head = self.q_norm(q_by_head)
        q = q_by_head.view(q.shape)
        k_by_head = k.view(*k.shape[:-1], k.shape[-1] // self.head_dim, self.head_dim)
        k_by_head = self.k_norm(k_by_head)
        k = k_by_head.view(k.shape)
        q, k = self.rotary_emb(positions, q, k)
        return q, k, v
```
**EN:** This method on `QKNormRoPETestModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKNormRoPETestModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `QKNormRoPETestModel.ops_in_model_before` (lines 103-112)
```python
    def ops_in_model_before(self) -> list[OpOverload | OpOverloadPacket]:
        ops: list[OpOverload | OpOverloadPacket] = [torch.ops.vllm_ir.rms_norm]
        if self.enable_rope_custom_op:
            if self.rotary_emb.use_flashinfer:
                ops.append(FLASHINFER_ROTARY_OP)
            else:
                ops.append(ROTARY_OP)
        else:
            ops.append(INDEX_SELECT_OP)
        return ops
```
**EN:** This method on `QKNormRoPETestModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKNormRoPETestModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `QKNormRoPETestModel.ops_in_model_after` (lines 114-115)
```python
    def ops_in_model_after(self) -> list[OpOverload | OpOverloadPacket]:
        return [FUSED_QK_ROPE_OP]
```
**EN:** This method on `QKNormRoPETestModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `QKNormRoPETestModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_qk_norm_rope_fusion` (lines 118-214)
```python
@pytest.mark.parametrize("scattered_split", [True, False])
@pytest.mark.parametrize("eps", [1e-5, 1e-6])
@pytest.mark.parametrize("is_neox", [True, False])
@pytest.mark.parametrize("enable_rms_norm_custom_op", [True, False])
@pytest.mark.parametrize("enable_rope_custom_op", [True])
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float16])
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Only test on cuda and rocm platform",
)
def test_qk_norm_rope_fusion(
    eps,
    is_neox,
    enable_rms_norm_custom_op,
    enable_rope_custom_op,
    dtype,
    scattered_split,
):
    if not hasattr(torch.ops._C, "fused_qk_norm_rope"):
        pytest.skip("fused_qk_norm_rope custom op not available")

    torch.set_default_device("cuda")
    torch.set_default_dtype(dtype)
    torch.manual_seed(0)

    custom_ops: list[str] = []
    if enable_rms_norm_custom_op:
        custom_ops.append("+rms_norm")
    if enable_rope_custom_op:
        custom_ops.append("+rotary_embedding")

    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=custom_ops,
            pass_config=PassConfig(
                enable_qk_norm_rope_fusion=True,
                eliminate_noops=True,
            ),
# ... excerpt ...
            test_scattered_split=scattered_split,
        )

        noop_pass = NoOpEliminationPass(vllm_config)
        coalesce_pass = SplitCoalescingPass(vllm_config)
        fusion_pass = QKNormRoPEFusionPass(vllm_config)
        cleanup_pass = PostCleanupPass(vllm_config)

        backend = TestBackend(noop_pass, coalesce_pass, fusion_pass, cleanup_pass)
        backend_baseline = TestBackend(noop_pass, cleanup_pass)

        qkv = torch.randn(T, model.q_size + 2 * model.kv_size)
        pos = torch.arange(T, dtype=torch.long, device=qkv.device)
        qkv_unfused = qkv.clone()
        pos_unfused = pos.clone()

        torch._dynamo.mark_dynamic(qkv, 0)
        torch._dynamo.mark_dynamic(pos, 0)
        model_fused = torch.compile(model, backend=backend)
        q_fused, k_fused, v_fused = model_fused(qkv, pos)

        torch._dynamo.mark_dynamic(qkv_unfused, 0)
        torch._dynamo.mark_dynamic(pos_unfused, 0)
        model_unfused = torch.compile(model, backend=backend_baseline)
        q_unfused, k_unfused, v_unfused = model_unfused(qkv_unfused, pos_unfused)

        if dtype == torch.float16:
            ATOL, RTOL = (2e-3, 2e-3)
        else:
            ATOL, RTOL = (1e-2, 1e-2)

        torch.testing.assert_close(q_unfused, q_fused, atol=ATOL, rtol=RTOL)
        torch.testing.assert_close(k_unfused, k_fused, atol=ATOL, rtol=RTOL)
        torch.testing.assert_close(v_unfused, v_fused, atol=ATOL, rtol=RTOL)

        assert fusion_pass.matched_count == 1

        backend.check_before_ops(model.ops_in_model_before())
        backend.check_after_ops(model.ops_in_model_after())
```
**EN:** This pytest case verifies qk norm rope fusion. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as eps, is_neox, enable_rms_norm_custom_op, enable_rope_custom_op. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 qk norm rope fusion 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 eps、is_neox、enable_rms_norm_custom_op、enable_rope_custom_op 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch._ops -> OpOverload, OpOverloadPacket`
- `tests.compile.backend -> TestBackend`
- `vllm.compilation.passes.fusion.matcher_utils -> FLASHINFER_ROTARY_OP, ROTARY_OP`
- `vllm.compilation.passes.fusion.qk_norm_rope_fusion -> FUSED_QK_ROPE_OP, QKNormRoPEFusionPass`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.compilation.passes.utility.split_coalescing -> SplitCoalescingPass`
- `vllm.config -> CompilationConfig, CompilationMode, ModelConfig, PassConfig, VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.attention -> Attention`
- `vllm.model_executor.layers.layernorm -> RMSNorm`
- `vllm.model_executor.layers.rotary_embedding -> RotaryEmbedding`
- `vllm.platforms -> current_platform`
- `vllm.v1.attention.backend -> AttentionType`
