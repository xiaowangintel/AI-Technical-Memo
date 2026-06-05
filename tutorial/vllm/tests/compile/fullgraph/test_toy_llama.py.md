# test_toy_llama.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fullgraph/test_toy_llama.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fullgraph / test_toy_llama, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fullgraph / test_toy_llama 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-10)
```python
"""
Test the piecewise compilation with a simple model, comparing the output
with and without the piecewise compilation.

This is a tractable model, the weights and computation are specially designed
if the config `tractable_init` is set to True. Otherwise, the weights are
initialized randomly with a fixed seed.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 12-34)
```python
from copy import deepcopy
from dataclasses import dataclass
from typing import Any

import pytest
import torch
from torch import nn

from vllm.compilation.decorators import support_torch_compile
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
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as copy, dataclasses, typing, pytest; shared test helpers from ...utils, ..; and vLLM components like vllm.compilation.decorators, vllm.config, vllm.forward_context, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 copy、dataclasses、typing、pytest；共享测试辅助模块，例如 ...utils、..；vLLM 内部组件，例如 vllm.compilation.decorators、vllm.config、vllm.forward_context、vllm.utils.torch_utils。

### Class `LlamaConfig` (lines 37-46)
```python
@dataclass
class LlamaConfig:
    hidden_size: int = 128
    mlp_size: int = 256
    vocab_size: int = 128
    num_layers: int = 2
    init_value: float = 1.0
    tractable_init: bool = False
    random_seed: int = 0
```
**EN:** This dataclass packages the fields needed to describe LlamaConfig. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 LlamaConfig 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `LlamaConfig.compute_hash` (lines 47-56)
```python
    def compute_hash(self) -> str:
        factors: list[Any] = []
        for k, v in self.__dict__.items():
            if k == "random_seed":
                continue
            factors.append((k, v))
        factors.sort()
        import hashlib

        return hashlib.md5(str(factors).encode(), usedforsecurity=False).hexdigest()
```
**EN:** This method on `LlamaConfig` implements compute hash. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math.
**CN:** `LlamaConfig` 中的这个方法实现了 compute hash。 这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。

### Method `LlamaConfig.__post_init__` (lines 58-59)
```python
    def __post_init__(self):
        assert self.mlp_size >= self.hidden_size
```
**EN:** This method on `LlamaConfig` implements post init. assertions at the end lock in the intended behavior or graph shape.
**CN:** `LlamaConfig` 中的这个方法实现了 post init。 结尾处的断言会固定预期行为或计算图形态。

### Class `LlamaMLP` (lines 62-62)
```python
class LlamaMLP(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for LlamaMLP. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 LlamaMLP 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `LlamaMLP.__init__` (lines 63-90)
```python
    def __init__(self, config: LlamaConfig) -> None:
        super().__init__()
        self.gate_up_projection = nn.Linear(
            in_features=config.hidden_size,
            out_features=config.mlp_size * 2,
            bias=False,
        )
        self.down_projection = nn.Linear(
            in_features=config.mlp_size,
            out_features=config.hidden_size,
            bias=False,
        )

        if config.tractable_init:
            nn.init.eye_(self.gate_up_projection.weight.data[: config.mlp_size])
            nn.init.eye_(self.gate_up_projection.weight.data[config.mlp_size :])
            nn.init.eye_(self.down_projection.weight.data)
        else:
            nn.init.xavier_normal_(
                self.gate_up_projection.weight.data,
                generator=torch.Generator().manual_seed(config.random_seed),
                gain=0.001,
            )
            nn.init.xavier_normal_(
                self.down_projection.weight.data,
                generator=torch.Generator().manual_seed(config.random_seed),
                gain=0.001,
            )
```
**EN:** This method implements the initialization for `LlamaMLP`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `LlamaMLP` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `LlamaMLP.forward` (lines 92-98)
```python
    def forward(self, x):
        # for tractable_init and positive input, this is
        # essentially an elementwise-square
        x = self.gate_up_projection(x)
        x = x[:, : x.size(1) // 2] * torch.nn.functional.relu(x[:, x.size(1) // 2 :])
        x = self.down_projection(x)
        return x
```
**EN:** This method on `LlamaMLP` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `LlamaMLP` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `LlamaAttention` (lines 101-101)
```python
class LlamaAttention(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for LlamaAttention. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 LlamaAttention 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `LlamaAttention.__init__` (lines 102-135)
```python
    def __init__(self, config: LlamaConfig) -> None:
        super().__init__()
        self.qkv_projection = nn.Linear(
            in_features=config.hidden_size,
            out_features=config.hidden_size * 3,
            bias=False,
        )

        self.output_projection = nn.Linear(
            in_features=config.hidden_size,
            out_features=config.hidden_size,
            bias=False,
        )

        if config.tractable_init:
            nn.init.eye_(self.qkv_projection.weight.data[: config.hidden_size])
            nn.init.eye_(
                self.qkv_projection.weight.data[
                    config.hidden_size : 2 * config.hidden_size
                ]
            )
            nn.init.eye_(self.qkv_projection.weight.data[2 * config.hidden_size :])
            nn.init.eye_(self.output_projection.weight.data)
        else:
            nn.init.xavier_normal_(
                self.qkv_projection.weight.data,
                generator=torch.Generator().manual_seed(config.random_seed),
                gain=0.001,
            )
            nn.init.xavier_normal_(
                self.output_projection.weight.data,
                generator=torch.Generator().manual_seed(config.random_seed),
                gain=0.001,
            )
```
**EN:** This method implements the initialization for `LlamaAttention`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `LlamaAttention` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `LlamaAttention.forward` (lines 137-155)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        # for tractable_init, this is:
        # output = (hidden_states * 3 + positions * 2)
        qkv = self.qkv_projection(hidden_states)
        hidden_size = qkv.size(-1) // 3
        q, k, v = qkv.split([hidden_size, hidden_size, hidden_size], dim=-1)

        q = q + positions.unsqueeze(1)
        k = k + positions.unsqueeze(1)

        attn_output = torch.empty_like(q)
        torch.ops.silly.attention(q, k, v, attn_output)

        output = self.output_projection(attn_output)
        return output
```
**EN:** This method on `LlamaAttention` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `LlamaAttention` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `LlamaDecoderLayer` (lines 158-158)
```python
class LlamaDecoderLayer(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for LlamaDecoderLayer. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 LlamaDecoderLayer 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `LlamaDecoderLayer.__init__` (lines 159-162)
```python
    def __init__(self, config: LlamaConfig) -> None:
        super().__init__()
        self.self_attention = LlamaAttention(config)
        self.mlp = LlamaMLP(config)
```
**EN:** This method implements the initialization for `LlamaDecoderLayer`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `LlamaDecoderLayer` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `LlamaDecoderLayer.forward` (lines 164-196)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        For tractable computation:
        - if residual is None, the outputs are:
            - residual = (hidden_states + 1) * 3 + positions * 2 + hidden_states = hidden_states * 4 + positions * 2 + 3
            - hidden_states = (residual + 1) ** 2
        - if residual is not None, the outputs are:
            - residual = (hidden_states + residual + 1) * 3 + positions * 2 + hidden_states + residual = (hidden_states + residual) * 4 + positions * 2 + 3
            - hidden_states = (residual + 1) ** 2
        """  # noqa
        if residual is None:
            residual = hidden_states
            hidden_states = hidden_states + 1
        else:
            hidden_states = hidden_states + residual
            residual = hidden_states
            hidden_states = hidden_states + 1

        hidden_states = self.self_attention(
            positions=positions, hidden_states=hidden_states
        )

        hidden_states = hidden_states + residual
        residual = hidden_states
        hidden_states = hidden_states + 1
        hidden_states = self.mlp(hidden_states)

        return hidden_states, residual
```
**EN:** This method on `LlamaDecoderLayer` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `LlamaDecoderLayer` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `LlamaModel` (lines 199-200)
```python
@support_torch_compile
class LlamaModel(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for LlamaModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 LlamaModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `LlamaModel.__init__` (lines 201-219)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        config: LlamaConfig,
        prefix: str = "",
        **kwargs,
    ) -> None:
        super().__init__()
        self.embedding_tokens = nn.Embedding(
            num_embeddings=config.vocab_size,
            embedding_dim=config.hidden_size,
        )
        self.layers = nn.ModuleList(
            [LlamaDecoderLayer(config) for _ in range(config.num_layers)]
        )

        # this is the initial value of the hidden states
        self.embedding_tokens.weight.data.fill_(config.init_value)
```
**EN:** This method implements the initialization for `LlamaModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `LlamaModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `LlamaModel.forward` (lines 221-230)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
    ) -> torch.Tensor:
        hidden_states = self.embedding_tokens(input_ids)
        residual = None
        for layer in self.layers:
            hidden_states, residual = layer(positions, hidden_states, residual)
        return hidden_states
```
**EN:** This method on `LlamaModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `LlamaModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `tractable_computation` (lines 233-259)
```python
def tractable_computation(
    input_ids: torch.Tensor,
    positions: torch.Tensor,
    config: LlamaConfig,
    init_value: float = 1.0,
) -> torch.Tensor:
    hidden_states = (
        torch.ones(
            input_ids.size(0),
            config.hidden_size,
            device=input_ids.device,
            dtype=input_ids.dtype,
        )
        * init_value
    )

    # first layer
    residual = hidden_states * 4 + positions.unsqueeze(1) * 2 + 3
    hidden_states = (residual + 1) ** 2

    # following layers
    for _ in range(config.num_layers - 1):
        hidden_states = hidden_states + residual
        residual = hidden_states * 4 + positions.unsqueeze(1) * 2 + 3
        hidden_states = (residual + 1) ** 2

    return hidden_states
```
**EN:** This helper function implements the shared logic for tractable computation. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 tractable computation 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_model` (lines 262-327)
```python
@torch.inference_mode
def run_model(llama_config, compile_config: CompilationConfig) -> torch.Tensor:
    # Start with a fresh copy to make sure there's no cache dir sharing
    compile_config = deepcopy(compile_config)
    cudagraph_runtime_mode = compile_config.cudagraph_mode

    vllm_config = VllmConfig(
        compilation_config=compile_config, additional_config=llama_config
    )
    with set_current_vllm_config(vllm_config):
        model = (
            LlamaModel(config=llama_config, vllm_config=vllm_config, prefix="")
            .eval()
            .cuda()
        )

    with set_forward_context({}, vllm_config=vllm_config):  # background context
        B = 16  # max batch size
        input_ids = torch.randint(0, llama_config.vocab_size, (B,)).cuda()
        positions = torch.arange(B).cuda()

        # warmup for the model with cudagraph_mode NONE
        model(input_ids, positions)

        # simulate cudagraphs capturing
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            model(input_ids[:2], positions[:2])
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=1,
            ),
        ):
            model(input_ids[:1], positions[:1])

        input_ids[:2].zero_()
        # simulate cudagraphs replay
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            output = model(input_ids[:2], positions[:2])

        output = output.cpu()

        if llama_config.tractable_init:
            expected_output = tractable_computation(
                input_ids[:2], positions[:2], llama_config
            ).cpu()

            assert torch.allclose(output, expected_output)
        else:
            return output.cpu()
```
**EN:** This helper function implements the shared logic for run model. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_toy_llama` (lines 330-426)
```python
@pytest.mark.parametrize(
    "backend, use_inductor_graph_partition",
    [
        ("eager", False),  # No inductor
        ("inductor", False),  # Inductor, Dynamo partition
        ("inductor", True),  # Inductor, Inductor partition
    ],
)
@create_new_process_for_each_test("spawn")
def test_toy_llama(
    backend: str, use_inductor_graph_partition: bool, monkeypatch, tmp_path
):
    from vllm.compilation.counter import compilation_counter

    # We disable the vLLM compile cache into a new tmp dir for 1 reason:
    # 1. To make sure we can properly track the number of Inductor compilations.
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    if use_inductor_graph_partition and not is_torch_equal_or_newer("2.9.0.dev"):
        pytest.skip("Inductor graph partition only supported in torch>=2.9")

    # compare output with and without piecewise compilation

    llama_config = LlamaConfig(
        hidden_size=128, mlp_size=256, vocab_size=128, num_layers=12
    )

    tractable_config = LlamaConfig(
        hidden_size=128, mlp_size=256, vocab_size=128, num_layers=2, tractable_init=True
    )

    compile_config_no_compile = CompilationConfig(
        mode=CompilationMode.NONE,
        cudagraph_mode=CUDAGraphMode.NONE,
        backend="eager",
    )

    compile_config_no_split = CompilationConfig(
        mode=CompilationMode.VLLM_COMPILE,
        use_inductor_graph_partition=use_inductor_graph_partition,
# ... excerpt ...
    run_model(tractable_config, compile_config_no_compile)

    if backend == "inductor":
        kwargs = {"num_inductor_compiles": 1, "num_eager_compiles": 0}
    else:
        kwargs = {"num_eager_compiles": 1, "num_inductor_compiles": 0}

    with compilation_counter.expect(
        num_graphs_seen=1,  # one graph for the model
        num_piecewise_graphs_seen=1,
        num_piecewise_capturable_graphs_seen=1,
        num_backend_compilations=1,  # num_piecewise_capturable_graphs_seen
        num_cudagraph_captured=2,
        **kwargs,
    ):
        outputs.append(run_model(llama_config, compile_config_no_split))

    run_model(tractable_config, compile_config_no_split)

    if use_inductor_graph_partition:
        num_piecewise_fx = 1
        num_piecewise_capturable_fx = 1
    else:
        num_piecewise_fx = 2 * llama_config.num_layers + 1
        num_piecewise_capturable_fx = 1 + llama_config.num_layers

    with compilation_counter.expect(
        num_graphs_seen=1,  # one graph for the model
        num_piecewise_graphs_seen=num_piecewise_fx,
        num_piecewise_capturable_graphs_seen=num_piecewise_capturable_fx,
        num_backend_compilations=num_piecewise_capturable_fx,
        # num_cudagraph_sizes * num_partitions
        num_cudagraph_captured=2 * (1 + llama_config.num_layers),
    ):
        outputs.append(run_model(llama_config, compile_config_split))
    run_model(tractable_config, compile_config_split)

    for i in range(1, len(outputs)):
        assert torch.allclose(outputs[0], outputs[i])
```
**EN:** This pytest case verifies toy llama. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as backend, use_inductor_graph_partition, monkeypatch, tmp_path. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 toy llama 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 backend、use_inductor_graph_partition、monkeypatch、tmp_path 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `benchmark` (lines 429-516)
```python
@torch.inference_mode
def benchmark():
    from triton.testing import do_bench

    # similar to llama 3.1-8B
    llama_config = LlamaConfig(
        hidden_size=4096, mlp_size=14336, vocab_size=128 * 1024, num_layers=32
    )

    # a tiny model to measure the overhead
    # of piecewise cudagraph
    llama_config = LlamaConfig(
        hidden_size=40, mlp_size=80, vocab_size=128, num_layers=2
    )

    cudagraph_sizes = [1, 2, 4] + [i * 8 for i in range(1, 33)]

    eager_time = {}
    full_cudagraph_time = {}
    piecewise_cudagraph_time = {}

    pool = torch.cuda.graph_pool_handle()

    for piecewise in [False, True]:
        if piecewise:
            compilation_config = CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
                splitting_ops=["silly::attention"],
                cudagraph_capture_sizes=cudagraph_sizes,
            )
        else:
            compilation_config = CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
                cudagraph_capture_sizes=cudagraph_sizes,
            )

        vllm_config = VllmConfig(compilation_config=compilation_config)
        with set_current_vllm_config(vllm_config):
            model = (
                LlamaModel(config=llama_config, vllm_config=vllm_config, prefix="")
# ... excerpt ...
        graphs = {}

        model(input_ids, positions)
        for b in cudagraph_sizes[::-1]:
            if not piecewise:
                graph = torch.cuda.CUDAGraph()
                with torch.cuda.graph(graph, pool=pool):
                    output = model(input_ids[:b], positions[:b])
                graphs[b] = (graph, output)
            else:
                output = model(input_ids[:b], positions[:b])
                graphs[b] = (model, output)
        for b in cudagraph_sizes:
            if piecewise:
                # noqa is for `Function definition does not bind loop variable`
                # it will be problematic if we save the created lambda function
                # and use it later, because it will look up the name `b` in the
                # enclosing scope, and the value of `b` will always be 256.
                # it is fine here, because we only use the lambda function once.
                runtime = do_bench(
                    lambda: graphs[b][0](  # noqa
                        input_ids[:b],  # noqa
                        positions[:b],  # noqa
                    )
                )
                piecewise_cudagraph_time[b] = runtime
            else:
                runtime = do_bench(lambda: graphs[b][0].replay())  # noqa
                eager_runtime = do_bench(lambda: model(input_ids[:b], positions[:b]))  # noqa
                full_cudagraph_time[b] = runtime
                eager_time[b] = eager_runtime

    # print in tabular format
    print("batch size\teager mode\tfull cudagraph\tpiecewise cudagraph")
    for b in cudagraph_sizes:
        print(
            f"{b}\t{eager_time[b]:.3f}\t{full_cudagraph_time[b]:.3f}"
            f"\t{piecewise_cudagraph_time[b]:.3f}"
        )
```
**EN:** This helper function implements the shared logic for benchmark. It keeps the surrounding module logic factored into a reusable unit. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 benchmark 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。 由于该代码块较大，这里只展示关键片段。

### Top-level block starting at line 519 (lines 519-524)
```python
if __name__ == "__main__":
    # Protect against subprocess reimport when using spawn_new_process_for_each_test
    import os

    if os.environ.get("RUNNING_IN_SUBPROCESS") != "1":
        benchmark()
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `copy -> deepcopy`
- `dataclasses -> dataclass`
- `typing -> Any`
- `pytest`
- `torch`
- `torch -> nn`
- `vllm.compilation.decorators -> support_torch_compile`
- `vllm.config -> CompilationConfig, CompilationMode, CUDAGraphMode, VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> BatchDescriptor, set_forward_context`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `...utils -> create_new_process_for_each_test`
- `.. -> silly_attention`
