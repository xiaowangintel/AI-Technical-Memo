# test_multiple_graphs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fullgraph/test_multiple_graphs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fullgraph / test_multiple_graphs, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fullgraph / test_multiple_graphs 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""
Test (piecewise) compilation with a simple model where multiple submodules
are compiled and graph captured separately.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-28)
```python
import pytest
import torch
from torch import nn

from vllm.compilation.backends import set_model_tag
from vllm.compilation.counter import compilation_counter
from vllm.compilation.decorators import ignore_torch_compile, support_torch_compile
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    CUDAGraphMode,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.forward_context import BatchDescriptor, set_forward_context
from vllm.utils.torch_utils import is_torch_equal_or_newer

from ...utils import create_new_process_for_each_test

# This import automatically registers `torch.ops.silly.attention`
from .. import silly_attention  # noqa: F401
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from ...utils, ..; and vLLM components like vllm.compilation.backends, vllm.compilation.counter, vllm.compilation.decorators, vllm.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 ...utils、..；vLLM 内部组件，例如 vllm.compilation.backends、vllm.compilation.counter、vllm.compilation.decorators、vllm.config。

### Constants and module state (lines 30-33)
```python
BATCH_SIZE = 32
MLP_SIZE = 128
HIDDEN_SIZE = 1024
RANDOM_SEED = 0
```
**EN:** This block centralizes shared constants and parameter grids, including BATCH_SIZE, MLP_SIZE, HIDDEN_SIZE, RANDOM_SEED. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BATCH_SIZE、MLP_SIZE、HIDDEN_SIZE、RANDOM_SEED。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `ParentModel` (lines 36-37)
```python
@support_torch_compile
class ParentModel(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for ParentModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 ParentModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `ParentModel.__init__` (lines 38-39)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs) -> None:
        super().__init__()
```
**EN:** This method implements the initialization for `ParentModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `ParentModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `ParentModel.forward` (lines 41-42)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return x
```
**EN:** This method on `ParentModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `ParentModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `Attention` (lines 45-45)
```python
class Attention(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for Attention. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 Attention 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `Attention.__init__` (lines 46-62)
```python
    def __init__(self, mlp_size: int, hidden_size: int) -> None:
        super().__init__()
        self.pre_attn = nn.Linear(mlp_size, hidden_size, bias=False)
        self.post_attn = nn.Linear(hidden_size, mlp_size, bias=False)
        self.rms_norm_weight = nn.Parameter(torch.ones(hidden_size))

        # Initialize to same weights for testing
        nn.init.xavier_normal_(
            self.pre_attn.weight.data,
            generator=torch.Generator().manual_seed(RANDOM_SEED),
            gain=0.001,
        )
        nn.init.xavier_normal_(
            self.post_attn.weight.data,
            generator=torch.Generator().manual_seed(RANDOM_SEED),
            gain=0.001,
        )
```
**EN:** This method implements the initialization for `Attention`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `Attention` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Attention.rms_norm_ref` (lines 64-70)
```python
    def rms_norm_ref(self, x: torch.Tensor) -> torch.Tensor:
        x_f32 = x.float()
        return (
            x_f32
            * torch.rsqrt(torch.mean(x_f32.square(), dim=-1, keepdim=True) + 1e-6)
            * self.rms_norm_weight
        ).to(x.dtype)
```
**EN:** This method on `Attention` implements rms norm ref. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Attention` 中的这个方法实现了 rms norm ref。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Attention.forward` (lines 72-80)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = self.pre_attn(x)
        x = self.rms_norm_ref(x)
        attn_output = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, attn_output)
        x = attn_output
        x = self.rms_norm_ref(x)
        x = self.post_attn(x)
        return x
```
**EN:** This method on `Attention` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Attention` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `CompiledAttention` (lines 83-84)
```python
@support_torch_compile
class CompiledAttention(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for CompiledAttention. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 CompiledAttention 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `CompiledAttention.__init__` (lines 85-95)
```python
    def __init__(
        self,
        *,
        mlp_size: int,
        hidden_size: int,
        vllm_config: VllmConfig,
        prefix: str = "",
        **kwargs,
    ) -> None:
        super().__init__()
        self.attn = Attention(mlp_size, hidden_size)
```
**EN:** This method implements the initialization for `CompiledAttention`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `CompiledAttention` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `CompiledAttention.forward` (lines 97-98)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.attn(x)
```
**EN:** This method on `CompiledAttention` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `CompiledAttention` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `CompiledAttentionTwo` (lines 101-102)
```python
@support_torch_compile
class CompiledAttentionTwo(CompiledAttention):
```
**EN:** This helper class groups the state and behavior needed for CompiledAttentionTwo. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 CompiledAttentionTwo 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `CompiledAttentionTwo.forward` (lines 103-104)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.attn(x) + x
```
**EN:** This method on `CompiledAttentionTwo` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `CompiledAttentionTwo` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `SimpleModelWithTwoGraphs` (lines 107-108)
```python
@ignore_torch_compile
class SimpleModelWithTwoGraphs(ParentModel):
```
**EN:** This helper class groups the state and behavior needed for SimpleModelWithTwoGraphs. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 SimpleModelWithTwoGraphs 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `SimpleModelWithTwoGraphs.__init__` (lines 109-139)
```python
    def __init__(
        self,
        *,
        mlp_size: int,
        hidden_size: int,
        vllm_config: VllmConfig,
        prefix: str = "",
        **kwargs,
    ) -> None:
        super().__init__(vllm_config=vllm_config, prefix=prefix)
        # Test will fail without set_model_tag here with error:
        # "ValueError: too many values to unpack (expected 3)"
        # This is because CompiledAttention and CompiledAttentionTwo
        # have different implementations but the same torch.compile
        # cache dir will be used as default prefix is 'model_tag'
        with set_model_tag("attn_one"):
            self.attn_one = CompiledAttention(
                mlp_size=mlp_size,
                hidden_size=hidden_size,
                vllm_config=vllm_config,
                prefix=f"{prefix}.attn_one",
            )
        with set_model_tag("attn_two"):
            self.attn_two = CompiledAttentionTwo(
                mlp_size=mlp_size,
                hidden_size=hidden_size,
                vllm_config=vllm_config,
                prefix=f"{prefix}.attn_two",
            )

        self.hidden_states = torch.zeros((BATCH_SIZE, MLP_SIZE)).cuda()
```
**EN:** This method implements the initialization for `SimpleModelWithTwoGraphs`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `SimpleModelWithTwoGraphs` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `SimpleModelWithTwoGraphs.forward` (lines 141-148)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        bsz = x.shape[0]
        # CUDAGraph expects same tensor addresses for each run
        self.hidden_states[:bsz].copy_(x)
        x = self.attn_one(self.hidden_states[:bsz])
        self.hidden_states[:bsz].copy_(x)
        x = self.attn_two(self.hidden_states[:bsz])
        return x
```
**EN:** This method on `SimpleModelWithTwoGraphs` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `SimpleModelWithTwoGraphs` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_model` (lines 151-194)
```python
@torch.inference_mode
def run_model(
    vllm_config: VllmConfig,
    model: nn.Module,
    inputs: torch.Tensor,
    cudagraph_runtime_mode: CUDAGraphMode,
):
    with set_forward_context({}, vllm_config=vllm_config):
        # warmup for the model with cudagraph_mode NONE
        model(inputs)

        # simulate cudagraphs capturing
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            model(inputs[:2])
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=1,
            ),
        ):
            model(inputs[:1])

        # simulate cudagraphs replay
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            output = model(inputs[:2])

        output = output.cpu()
        return output.cpu()
```
**EN:** This helper function implements the shared logic for run model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_multi_graph_piecewise_compile` (lines 197-326)
```python
@pytest.mark.parametrize("use_inductor_graph_partition", [False, True])
@pytest.mark.parametrize("use_bytecode_hook", [True, False])
@create_new_process_for_each_test("spawn")
def test_multi_graph_piecewise_compile(
    use_inductor_graph_partition: bool, use_bytecode_hook: bool, monkeypatch
):
    # Set the environment variable for this test
    monkeypatch.setenv("VLLM_USE_BYTECODE_HOOK", "1" if use_bytecode_hook else "0")

    if use_inductor_graph_partition and not is_torch_equal_or_newer("2.9.0.dev"):
        pytest.skip("inductor graph partition is only available in PyTorch 2.9+")

    outputs = []

    # vllmcompile compile
    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            cudagraph_mode=CUDAGraphMode.PIECEWISE,
            splitting_ops=["silly::attention"],
            cudagraph_capture_sizes=[1, 2],
            use_inductor_graph_partition=use_inductor_graph_partition,
        )
    )
    cudagraph_runtime_mode = CUDAGraphMode.PIECEWISE

    with set_current_vllm_config(vllm_config):
        model = (
            SimpleModelWithTwoGraphs(
                mlp_size=MLP_SIZE,
                hidden_size=HIDDEN_SIZE,
                vllm_config=vllm_config,
                prefix="",
            )
            .eval()
            .cuda()
        )

    # Pre-allocate memory for CUDAGraph which expects
    # static tensor addresses
# ... excerpt ...

    # piecewise compile without CUDA graph
    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            cudagraph_mode=CUDAGraphMode.NONE,
            splitting_ops=["silly::attention"],
            use_inductor_graph_partition=use_inductor_graph_partition,
        )
    )
    cudagraph_runtime_mode = CUDAGraphMode.PIECEWISE

    with set_current_vllm_config(vllm_config):
        model = (
            SimpleModelWithTwoGraphs(
                mlp_size=MLP_SIZE,
                hidden_size=HIDDEN_SIZE,
                vllm_config=vllm_config,
                prefix="",
            )
            .eval()
            .cuda()
        )

    with compilation_counter.expect(
        num_graphs_seen=2,
        num_piecewise_graphs_seen=num_piecewise_fx,
        num_piecewise_capturable_graphs_seen=num_piecewise_capturable_fx,
        num_backend_compilations=num_piecewise_capturable_fx,
        num_cudagraph_captured=0,  # no cudagraph captured
    ):
        outputs.append(run_model(vllm_config, model, inputs, cudagraph_runtime_mode))

    # Generally don't expect outputs with and without inductor
    # to be bitwise equivalent
    assert torch.allclose(outputs[0], outputs[1])

    # Expect bitwise equivalence using inductor w/ and w/o cudagraph
    assert torch.equal(outputs[0], outputs[2])
```
**EN:** This pytest case verifies multi graph piecewise compile. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as use_inductor_graph_partition, use_bytecode_hook, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 multi graph piecewise compile 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 use_inductor_graph_partition、use_bytecode_hook、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch -> nn`
- `vllm.compilation.backends -> set_model_tag`
- `vllm.compilation.counter -> compilation_counter`
- `vllm.compilation.decorators -> ignore_torch_compile, support_torch_compile`
- `vllm.config -> CompilationConfig, CompilationMode, CUDAGraphMode, VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> BatchDescriptor, set_forward_context`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `...utils -> create_new_process_for_each_test`
- `.. -> silly_attention`
