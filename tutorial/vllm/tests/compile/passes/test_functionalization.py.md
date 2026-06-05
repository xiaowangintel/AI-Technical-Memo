# test_functionalization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_functionalization.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_functionalization, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_functionalization 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-36)
```python
import copy

import pytest
import torch

from tests.compile.backend import TestBackend
from tests.utils import TestFP8Layer
from vllm.compilation.passes.fusion.act_quant_fusion import (
    ActivationQuantFusionPass,
)
from vllm.compilation.passes.fusion.rms_quant_fusion import RMSNormQuantFusionPass
from vllm.compilation.passes.fx_utils import find_auto_fn, find_auto_fn_maybe, is_func
from vllm.compilation.passes.utility.fix_functionalization import (
    FixFunctionalizationPass,
)
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.config import (
    CompilationConfig,
    ModelConfig,
    PassConfig,
    VllmConfig,
    get_current_vllm_config,
    set_current_vllm_config,
)
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8StaticTensorSym,
)
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, pytest, torch; shared test helpers from tests.compile.backend, tests.utils; and vLLM components like vllm.compilation.passes.fusion.act_quant_fusion, vllm.compilation.passes.fusion.rms_quant_fusion, vllm.compilation.passes.fx_utils, vllm.compilation.passes.utility.fix_functionalization.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.utils；vLLM 内部组件，例如 vllm.compilation.passes.fusion.act_quant_fusion、vllm.compilation.passes.fusion.rms_quant_fusion、vllm.compilation.passes.fx_utils、vllm.compilation.passes.utility.fix_functionalization。

### Constants and module state (lines 38-39)
```python
TEST_FP8 = current_platform.supports_fp8()
FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This block centralizes shared constants and parameter grids, including TEST_FP8, FP8_DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 TEST_FP8、FP8_DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TestSiluMul` (lines 42-44)
```python
class TestSiluMul(torch.nn.Module):
    quant_key = kFp8StaticTensorSym
```
**EN:** This helper class groups the state and behavior needed for TestSiluMul. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestSiluMul 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestSiluMul.__init__` (lines 45-54)
```python
    def __init__(self, hidden_size: int = 128):
        super().__init__()
        self.silu_and_mul = SiluAndMul()
        if TEST_FP8:
            self.fp8_linear = TestFP8Layer(
                weight_shape=(hidden_size, hidden_size),
                activation_quant_key=self.quant_key,
                weight_quant_key=self.quant_key,
                input_dtype=get_current_vllm_config().model_config.dtype,
            )
```
**EN:** This method implements the initialization for `TestSiluMul`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestSiluMul` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMul.forward` (lines 56-61)
```python
    def forward(self, x):
        y = self.silu_and_mul(x)
        if TEST_FP8:
            return self.fp8_linear(y)
        else:
            return y
```
**EN:** This method on `TestSiluMul` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMul` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMul.example_inputs` (lines 63-64)
```python
    def example_inputs(self, num_tokens=32, hidden_size=128):
        return (torch.rand(num_tokens, hidden_size * 2),)
```
**EN:** This method on `TestSiluMul` implements example inputs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMul` 中的这个方法实现了 example inputs。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMul.ops_in_model` (lines 66-70)
```python
    def ops_in_model(self, do_fusion):
        if TEST_FP8 and do_fusion:
            return [torch.ops._C.silu_and_mul_quant.default]
        else:
            return [torch.ops._C.silu_and_mul.default]
```
**EN:** This method on `TestSiluMul` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMul` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMul.ops_not_in_model` (lines 72-73)
```python
    def ops_not_in_model(self):
        return []
```
**EN:** This method on `TestSiluMul` implements ops not in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMul` 中的这个方法实现了 ops not in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestFusedAddRMSNorm` (lines 76-78)
```python
class TestFusedAddRMSNorm(torch.nn.Module):
    quant_key = kFp8StaticTensorSym
```
**EN:** This helper class groups the state and behavior needed for TestFusedAddRMSNorm. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestFusedAddRMSNorm 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestFusedAddRMSNorm.__init__` (lines 79-98)
```python
    def __init__(self, hidden_size=16, intermediate_size=32):
        super().__init__()
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size

        self.gate_proj = torch.nn.Parameter(
            torch.empty((intermediate_size, hidden_size))
        )
        self.norm = RMSNorm(intermediate_size, 1e-05)
        self.norm.weight = torch.nn.Parameter(torch.ones(intermediate_size))

        torch.nn.init.normal_(self.gate_proj, std=0.02)

        if TEST_FP8:
            self.fp8_linear = TestFP8Layer(
                weight_shape=(hidden_size, intermediate_size),
                activation_quant_key=self.quant_key,
                weight_quant_key=self.quant_key,
                input_dtype=get_current_vllm_config().model_config.dtype,
            )
```
**EN:** This method implements the initialization for `TestFusedAddRMSNorm`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestFusedAddRMSNorm` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFusedAddRMSNorm.forward` (lines 100-118)
```python
    def forward(self, hidden_states, residual):
        # Reshape input
        view = hidden_states.reshape(-1, self.hidden_size)

        # matrix multiplication
        permute = self.gate_proj.permute(1, 0)
        mm = torch.mm(view, permute)

        # layer normalization
        norm_output, residual_output = self.norm(mm, residual)

        if TEST_FP8:
            # scaled_mm with static input quantization
            fp8_linear_result = self.fp8_linear(norm_output)

            return fp8_linear_result, residual_output

        else:
            return norm_output, residual_output
```
**EN:** This method on `TestFusedAddRMSNorm` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFusedAddRMSNorm` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFusedAddRMSNorm.example_inputs` (lines 120-123)
```python
    def example_inputs(self, batch_size=8, seq_len=16):
        hidden_states = torch.randn((batch_size * seq_len, self.hidden_size))
        residual = torch.randn((batch_size * seq_len, self.intermediate_size))
        return (hidden_states, residual)
```
**EN:** This method on `TestFusedAddRMSNorm` implements example inputs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFusedAddRMSNorm` 中的这个方法实现了 example inputs。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFusedAddRMSNorm.ops_in_model` (lines 125-129)
```python
    def ops_in_model(self, do_fusion):
        if TEST_FP8 and do_fusion:
            return [torch.ops._C.fused_add_rms_norm_static_fp8_quant.default]
        else:
            return []
```
**EN:** This method on `TestFusedAddRMSNorm` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFusedAddRMSNorm` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFusedAddRMSNorm.ops_not_in_model` (lines 131-132)
```python
    def ops_not_in_model(self):
        return []
```
**EN:** This method on `TestFusedAddRMSNorm` implements ops not in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFusedAddRMSNorm` 中的这个方法实现了 ops not in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestRotaryEmbedding` (lines 135-135)
```python
class TestRotaryEmbedding(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestRotaryEmbedding. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestRotaryEmbedding 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestRotaryEmbedding.__init__` (lines 136-144)
```python
    def __init__(self, head_dim=64, max_position=2048, base=10000):
        super().__init__()
        self.head_dim = head_dim

        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position,
            rope_parameters={"rope_type": "default", "rope_theta": base},
        )
```
**EN:** This method implements the initialization for `TestRotaryEmbedding`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestRotaryEmbedding` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbedding.forward` (lines 146-148)
```python
    def forward(self, positions, q, k):
        q_rotated, k_rotated = self.rotary_emb(positions, q, k)
        return q_rotated, k_rotated
```
**EN:** This method on `TestRotaryEmbedding` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbedding` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbedding.example_inputs` (lines 150-154)
```python
    def example_inputs(self, num_tokens=32, head_dim=64):
        positions = torch.arange(num_tokens, dtype=torch.long)
        q = torch.randn(num_tokens, head_dim)
        k = torch.randn(num_tokens, head_dim)
        return (positions, q, k)
```
**EN:** This method on `TestRotaryEmbedding` implements example inputs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbedding` 中的这个方法实现了 example inputs。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbedding.ops_in_model` (lines 156-157)
```python
    def ops_in_model(self, do_fusion):
        return [torch.ops._C.rotary_embedding.default]
```
**EN:** This method on `TestRotaryEmbedding` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbedding` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbedding.ops_not_in_model` (lines 159-160)
```python
    def ops_not_in_model(self):
        return []
```
**EN:** This method on `TestRotaryEmbedding` implements ops not in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbedding` 中的这个方法实现了 ops not in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestRotaryEmbeddingSliceScatter` (lines 163-163)
```python
class TestRotaryEmbeddingSliceScatter(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestRotaryEmbeddingSliceScatter. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestRotaryEmbeddingSliceScatter 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestRotaryEmbeddingSliceScatter.__init__` (lines 164-178)
```python
    def __init__(self, head_dim=64, num_heads=4, max_position=2048, base=10000):
        super().__init__()
        self.head_dim = head_dim
        self.num_heads = num_heads
        self.hidden_size = head_dim * num_heads

        self.qkv_proj = torch.nn.Linear(
            self.hidden_size, self.hidden_size * 3, bias=False
        )

        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position,
            rope_parameters={"rope_type": "default", "rope_theta": base},
        )
```
**EN:** This method implements the initialization for `TestRotaryEmbeddingSliceScatter`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestRotaryEmbeddingSliceScatter` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbeddingSliceScatter.forward` (lines 180-191)
```python
    def forward(self, positions, hidden_states):
        # Simulate the pattern: mm -> split_with_sizes -> rotary_embedding
        # -> slice_scatter -> split_with_sizes

        qkv = self.qkv_proj(hidden_states)
        split_sizes = [self.hidden_size, self.hidden_size, self.hidden_size]
        q, k, v = torch.split(qkv, split_sizes, dim=-1)

        q_rotated, k_rotated = self.rotary_emb(positions, q, k)

        qkv_updated = torch.cat([q_rotated, k_rotated, v], dim=-1)
        return qkv_updated
```
**EN:** This method on `TestRotaryEmbeddingSliceScatter` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbeddingSliceScatter` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbeddingSliceScatter.example_inputs` (lines 193-197)
```python
    def example_inputs(self, num_tokens=32, head_dim=64, num_heads=4):
        hidden_size = head_dim * num_heads
        positions = torch.arange(num_tokens, dtype=torch.long)
        hidden_states = torch.randn(num_tokens, hidden_size)
        return (positions, hidden_states)
```
**EN:** This method on `TestRotaryEmbeddingSliceScatter` implements example inputs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbeddingSliceScatter` 中的这个方法实现了 example inputs。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbeddingSliceScatter.ops_in_model` (lines 199-200)
```python
    def ops_in_model(self, do_fusion):
        return [torch.ops._C.rotary_embedding.default]
```
**EN:** This method on `TestRotaryEmbeddingSliceScatter` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbeddingSliceScatter` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestRotaryEmbeddingSliceScatter.ops_not_in_model` (lines 202-203)
```python
    def ops_not_in_model(self):
        return [torch.ops.aten.slice_scatter.default]
```
**EN:** This method on `TestRotaryEmbeddingSliceScatter` implements ops not in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestRotaryEmbeddingSliceScatter` 中的这个方法实现了 ops not in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestFunctionWithMutatedArgsAndReturn` (lines 206-208)
```python
class TestFunctionWithMutatedArgsAndReturn(torch.nn.Module):
    OP_REGISTERED = False
```
**EN:** This helper class groups the state and behavior needed for TestFunctionWithMutatedArgsAndReturn. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestFunctionWithMutatedArgsAndReturn 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestFunctionWithMutatedArgsAndReturn.__init__` (lines 209-211)
```python
    def __init__(self):
        super().__init__()
        self.register_test_custom_op()
```
**EN:** This method implements the initialization for `TestFunctionWithMutatedArgsAndReturn`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestFunctionWithMutatedArgsAndReturn` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFunctionWithMutatedArgsAndReturn.register_test_custom_op` (lines 213-236)
```python
    @classmethod
    def register_test_custom_op(cls):
        if not cls.OP_REGISTERED:

            def function_with_mutated_args_and_return_impl(
                x: torch.Tensor,
            ) -> torch.Tensor:
                ret = x + 1
                x.add_(2)
                return ret

            def function_with_mutated_args_and_return_fake(
                x: torch.Tensor,
            ) -> torch.Tensor:
                return torch.empty_like(x)

            direct_register_custom_op(
                op_name="function_with_mutated_args_and_return",
                op_func=function_with_mutated_args_and_return_impl,
                mutates_args=["x"],
                fake_impl=function_with_mutated_args_and_return_fake,
            )

            cls.OP_REGISTERED = True
```
**EN:** This method on `TestFunctionWithMutatedArgsAndReturn` implements register test custom op. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFunctionWithMutatedArgsAndReturn` 中的这个方法实现了 register test custom op。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFunctionWithMutatedArgsAndReturn.forward` (lines 238-241)
```python
    def forward(self, x: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        # Clone x to avoid mutating the original tensor
        ret = torch.ops.vllm.function_with_mutated_args_and_return(x)
        return x, ret
```
**EN:** This method on `TestFunctionWithMutatedArgsAndReturn` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFunctionWithMutatedArgsAndReturn` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFunctionWithMutatedArgsAndReturn.example_inputs` (lines 243-245)
```python
    def example_inputs(self, num_tokens=32):
        hidden_states = torch.randn(num_tokens)
        return (hidden_states,)
```
**EN:** This method on `TestFunctionWithMutatedArgsAndReturn` implements example inputs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFunctionWithMutatedArgsAndReturn` 中的这个方法实现了 example inputs。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFunctionWithMutatedArgsAndReturn.ops_in_model` (lines 247-248)
```python
    def ops_in_model(self, do_fusion):
        return [torch.ops.vllm.function_with_mutated_args_and_return.default]
```
**EN:** This method on `TestFunctionWithMutatedArgsAndReturn` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFunctionWithMutatedArgsAndReturn` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestFunctionWithMutatedArgsAndReturn.ops_not_in_model` (lines 250-251)
```python
    def ops_not_in_model(self):
        return []
```
**EN:** This method on `TestFunctionWithMutatedArgsAndReturn` implements ops not in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestFunctionWithMutatedArgsAndReturn` 中的这个方法实现了 ops not in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 254-260)
```python
MODELS_AND_DO_FUSION = {
    TestSiluMul: [True, False],
    TestFusedAddRMSNorm: [True, False],
    TestRotaryEmbedding: [False],
    TestRotaryEmbeddingSliceScatter: [False],
    TestFunctionWithMutatedArgsAndReturn: [False],
}
```
**EN:** This block centralizes shared constants and parameter grids, including MODELS_AND_DO_FUSION. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MODELS_AND_DO_FUSION。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_fix_functionalization` (lines 263-340)
```python
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
@pytest.mark.parametrize(
    "model_class, do_fusion",
    [
        (model_class, do_fusion)
        for model_class, fusions in MODELS_AND_DO_FUSION.items()
        for do_fusion in fusions
    ],
)
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Only test on cuda and rocm platform",
)
def test_fix_functionalization(
    model_class: torch.nn.Module, do_fusion: bool, dtype: torch.dtype
):
    torch.set_default_device("cuda")
    torch.set_default_dtype(dtype)
    torch.manual_seed(0)

    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            custom_ops=["all"],
            pass_config=PassConfig(
                fuse_norm_quant=do_fusion,
                fuse_act_quant=do_fusion,
                eliminate_noops=True,
            ),
        ),
    )

    with set_current_vllm_config(vllm_config):
        assert RMSNorm.enabled()
        noop_pass = NoOpEliminationPass(vllm_config)
        fusion_pass = RMSNormQuantFusionPass(vllm_config)
        cleanup_pass = PostCleanupPass(vllm_config)
        act_quant_fusion_pass = ActivationQuantFusionPass(vllm_config)

        passes = (
            [noop_pass, fusion_pass, act_quant_fusion_pass, cleanup_pass]
            if do_fusion
            else [noop_pass, cleanup_pass]
        )
        func_pass = FixFunctionalizationPass(vllm_config)

        backend_func = TestBackend(*passes, func_pass)
        backend_no_func = TestBackend(*passes)

        model = model_class()
        inputs_func = model.example_inputs()
        inputs_no_func = copy.deepcopy(inputs_func)
        model_func = copy.deepcopy(model)
        model_no_func = copy.deepcopy(model)
        model_func = torch.compile(model_func, backend=backend_func)
        model_no_func = torch.compile(model_no_func, backend=backend_no_func)

        # deepcopy inputs to prevent potential in place mutation
        outputs_func = model_func(*copy.deepcopy(inputs_func))
        outputs_no_func = model_no_func(*copy.deepcopy(inputs_no_func))
        torch.testing.assert_close(outputs_func, outputs_no_func)

        # check if the functionalization pass is applied
        for op in model.ops_in_model(do_fusion):
            find_auto_fn(backend_no_func.graph_post_pass.nodes, op)
            assert find_auto_fn_maybe(backend_func.graph_post_pass.nodes, op) is None

        # make sure the ops were all de-functionalized
        found = dict()
        for node in backend_func.graph_post_pass.nodes:
            for op in model.ops_in_model(do_fusion):
                if is_func(node, op):
                    found[op] = True
            for op in model.ops_not_in_model():
                if is_func(node, op):
                    found[op] = True
        assert all(found[op] for op in model.ops_in_model(do_fusion))
        assert all(not found.get(op) for op in model.ops_not_in_model())
```
**EN:** This pytest case verifies fix functionalization. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as model_class, do_fusion, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 fix functionalization 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 model_class、do_fusion、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `copy`
- `pytest`
- `torch`
- `tests.compile.backend -> TestBackend`
- `tests.utils -> TestFP8Layer`
- `vllm.compilation.passes.fusion.act_quant_fusion -> ActivationQuantFusionPass`
- `vllm.compilation.passes.fusion.rms_quant_fusion -> RMSNormQuantFusionPass`
- `vllm.compilation.passes.fx_utils -> find_auto_fn, find_auto_fn_maybe, is_func`
- `vllm.compilation.passes.utility.fix_functionalization -> FixFunctionalizationPass`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> CompilationConfig, ModelConfig, PassConfig, VllmConfig, get_current_vllm_config, set_current_vllm_config`
- `vllm.model_executor.layers.activation -> SiluAndMul`
- `vllm.model_executor.layers.layernorm -> RMSNorm`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> kFp8StaticTensorSym`
- `vllm.model_executor.layers.rotary_embedding -> get_rope`
