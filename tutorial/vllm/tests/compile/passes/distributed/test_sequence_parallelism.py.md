# test_sequence_parallelism.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/distributed/test_sequence_parallelism.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / distributed / test_sequence_parallelism, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / distributed / test_sequence_parallelism 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-37)
```python
import pytest
import torch

import vllm.envs as envs
from tests.compile.backend import TestBackend
from tests.utils import TestFP8Layer, multi_gpu_test
from vllm.compilation.passes.fusion.rms_quant_fusion import RMSNormQuantFusionPass
from vllm.compilation.passes.fusion.sequence_parallelism import SequenceParallelismPass
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.compilation.passes.vllm_inductor_pass import VllmInductorPass
from vllm.config import (
    CompilationConfig,
    CUDAGraphMode,
    DeviceConfig,
    ModelConfig,
    PassConfig,
    VllmConfig,
    get_current_vllm_config,
    set_current_vllm_config,
)
from vllm.config.utils import Range
from vllm.distributed import tensor_model_parallel_all_reduce
from vllm.distributed.parallel_state import (
    init_distributed_environment,
    initialize_model_parallel,
)
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8StaticTensorSym,
)
from vllm.platforms import current_platform
from vllm.utils.system_utils import update_environment_variables
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend, tests.utils; and vLLM components like vllm.envs, vllm.compilation.passes.fusion.rms_quant_fusion, vllm.compilation.passes.fusion.sequence_parallelism, vllm.compilation.passes.utility.noop_elimination.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.utils；vLLM 内部组件，例如 vllm.envs、vllm.compilation.passes.fusion.rms_quant_fusion、vllm.compilation.passes.fusion.sequence_parallelism、vllm.compilation.passes.utility.noop_elimination。

### Constants and module state (lines 39-49)
```python
DEVICE_TYPE = current_platform.device_type

pytestmark = pytest.mark.skipif(not current_platform.is_cuda(), reason="Only test CUDA")

FP8_DTYPE = current_platform.fp8_dtype()
prompts = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE_TYPE, FP8_DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE_TYPE、FP8_DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TestAllReduceRMSNormModel` (lines 52-52)
```python
class TestAllReduceRMSNormModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestAllReduceRMSNormModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAllReduceRMSNormModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAllReduceRMSNormModel.__init__` (lines 53-58)
```python
    def __init__(self, hidden_size=16, eps=1e-6):
        super().__init__()
        self.hidden_size = hidden_size
        self.eps = eps
        self.norm = [RMSNorm(hidden_size, eps) for i in range(4)]
        self.w = [torch.rand(hidden_size, hidden_size) for _ in range(3)]
```
**EN:** This method implements the initialization for `TestAllReduceRMSNormModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAllReduceRMSNormModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormModel.forward` (lines 60-79)
```python
    def forward(self, x):
        z = torch.relu(x)
        x = resid = tensor_model_parallel_all_reduce(z)
        y = self.norm[0](x)

        z2 = torch.mm(y, self.w[0])
        x2 = tensor_model_parallel_all_reduce(z2)

        y2, resid = self.norm[1](x2, resid)

        z3 = torch.mm(y2, self.w[1])
        x3 = tensor_model_parallel_all_reduce(z3)

        y3, resid = self.norm[2](x3, resid)

        z4 = torch.mm(y3, self.w[2])
        x4 = tensor_model_parallel_all_reduce(z4)

        y4, resid = self.norm[3](x4, resid)
        return y4
```
**EN:** This method on `TestAllReduceRMSNormModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormModel.ops_in_model_before` (lines 81-82)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.all_reduce.default]
```
**EN:** This method on `TestAllReduceRMSNormModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormModel.ops_in_model_after` (lines 84-88)
```python
    def ops_in_model_after(self):
        return [
            torch.ops.vllm.all_gather.default,
            torch.ops.vllm.reduce_scatter.default,
        ]
```
**EN:** This method on `TestAllReduceRMSNormModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormModel.ops_in_model` (lines 90-94)
```python
    def ops_in_model(self):
        return [
            torch.ops.vllm_ir.rms_norm,
            torch.ops.vllm_ir.fused_add_rms_norm,
        ]
```
**EN:** This method on `TestAllReduceRMSNormModel` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormModel` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAllReduceRMSNormStaticQuantFP8Model` (lines 97-99)
```python
class TestAllReduceRMSNormStaticQuantFP8Model(torch.nn.Module):
    quant_key = kFp8StaticTensorSym
```
**EN:** This helper class groups the state and behavior needed for TestAllReduceRMSNormStaticQuantFP8Model. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAllReduceRMSNormStaticQuantFP8Model 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.__init__` (lines 100-114)
```python
    def __init__(self, hidden_size=16, eps=1e-6):
        super().__init__()
        self.vllm_config = get_current_vllm_config()
        self.hidden_size = hidden_size
        self.eps = eps
        self.norm = [RMSNorm(hidden_size, eps) for i in range(4)]
        self.fp8_linear_layers = [
            TestFP8Layer(
                weight_shape=(hidden_size, hidden_size),
                activation_quant_key=self.quant_key,
                weight_quant_key=self.quant_key,
                input_dtype=self.vllm_config.model_config.dtype,
            )
            for i in range(3)
        ]
```
**EN:** This method implements the initialization for `TestAllReduceRMSNormStaticQuantFP8Model`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAllReduceRMSNormStaticQuantFP8Model` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.forward` (lines 116-135)
```python
    def forward(self, hidden_states):
        # avoid having graph input be an arg to a pattern directly
        z = torch.relu(hidden_states)
        x = resid = tensor_model_parallel_all_reduce(z)
        y = self.norm[0](x)

        z2 = self.fp8_linear_layers[0](y)

        x2 = tensor_model_parallel_all_reduce(z2)
        y2, resid = self.norm[1](x2, resid)

        z3 = self.fp8_linear_layers[1](y2)

        x3 = tensor_model_parallel_all_reduce(z3)
        y3, resid = self.norm[2](x3, resid)  # use resid here

        z4 = self.fp8_linear_layers[2](y3)
        x4 = tensor_model_parallel_all_reduce(z4)
        y4, resid = self.norm[3](x4, resid)  # use resid here
        return y4
```
**EN:** This method on `TestAllReduceRMSNormStaticQuantFP8Model` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormStaticQuantFP8Model` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.ops_in_model_after` (lines 137-141)
```python
    def ops_in_model_after(self):
        return [
            torch.ops.vllm.all_gather.default,
            torch.ops.vllm.reduce_scatter.default,
        ]
```
**EN:** This method on `TestAllReduceRMSNormStaticQuantFP8Model` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormStaticQuantFP8Model` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.ops_in_model_before` (lines 143-146)
```python
    def ops_in_model_before(self):
        return [
            torch.ops.vllm.all_reduce.default,
        ]
```
**EN:** This method on `TestAllReduceRMSNormStaticQuantFP8Model` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormStaticQuantFP8Model` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.ops_in_model` (lines 148-161)
```python
    def ops_in_model(self):
        if self.vllm_config.compilation_config.pass_config.fuse_norm_quant:
            return [torch.ops._C.fused_add_rms_norm_static_fp8_quant.default]
        else:
            quant_ops = (
                [torch.ops._C.static_scaled_fp8_quant.default]
                if any(layer.is_quant_fp8_enabled() for layer in self.fp8_linear_layers)
                else [torch.ops.aten.reciprocal]
            )
            return [
                torch.ops.vllm_ir.rms_norm,
                torch.ops.vllm_ir.fused_add_rms_norm,
                *quant_ops,
            ]
```
**EN:** This method on `TestAllReduceRMSNormStaticQuantFP8Model` implements ops in model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormStaticQuantFP8Model` 中的这个方法实现了 ops in model。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_sequence_parallelism_pass` (lines 164-214)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "test_model_cls, custom_ops",
    [
        (TestAllReduceRMSNormModel, "+rms_norm"),
        (TestAllReduceRMSNormModel, "-rms_norm"),
        (TestAllReduceRMSNormStaticQuantFP8Model, "+rms_norm,+quant_fp8"),
        (TestAllReduceRMSNormStaticQuantFP8Model, "+rms_norm,-quant_fp8"),
        (TestAllReduceRMSNormStaticQuantFP8Model, "-rms_norm,+quant_fp8"),
        (TestAllReduceRMSNormStaticQuantFP8Model, "-rms_norm,-quant_fp8"),
    ],
)
@pytest.mark.parametrize("batch_size", [8])
@pytest.mark.parametrize("seq_len", [16])
@pytest.mark.parametrize("hidden_size", [16])
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
@pytest.mark.parametrize("fuse_norm_quant", [True, False])
@pytest.mark.parametrize("dynamic", [False, True])
@pytest.mark.skipif(envs.VLLM_TARGET_DEVICE not in ["cuda"], reason="Only test on CUDA")
def test_sequence_parallelism_pass(
    test_model_cls: type[torch.nn.Module],
    custom_ops: str,
    batch_size: int,
    seq_len: int,
    hidden_size: int,
    dtype: torch.dtype,
    fuse_norm_quant: bool,
    dynamic: bool,
):
    num_processes = 2

    def run_torch_spawn(fn, nprocs):
        # need to use torch.mp.spawn otherwise will have problems with
        # torch.distributed and cuda
        torch.multiprocessing.spawn(
            fn,
            args=(
                num_processes,
                test_model_cls,
                custom_ops,
                batch_size,
                seq_len,
                hidden_size,
                dtype,
                fuse_norm_quant,
                dynamic,
            ),
            nprocs=nprocs,
        )

    run_torch_spawn(sequence_parallelism_pass_on_test_model, num_processes)
```
**EN:** This pytest case verifies sequence parallelism pass. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as test_model_cls, custom_ops, batch_size, seq_len. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 sequence parallelism pass 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 test_model_cls、custom_ops、batch_size、seq_len 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_sequence_parallelism_pass_requires_full_graph_compilation` (lines 217-232)
```python
def test_sequence_parallelism_pass_requires_full_graph_compilation():
    vllm_config = VllmConfig()
    vllm_config.compilation_config.use_inductor_graph_partition = False
    vllm_config.compilation_config.splitting_ops = [
        "vllm::unified_attention_with_output"
    ]

    sequence_parallelism_pass = object.__new__(SequenceParallelismPass)
    sequence_parallelism_pass.compilation_config = vllm_config.compilation_config
    sequence_parallelism_pass.min_token_num = 1

    with pytest.raises(
        AssertionError,
        match="SequenceParallelismPass requires full-graph compilation",
    ):
        sequence_parallelism_pass.is_applicable_for_range(Range(start=8, end=8))
```
**EN:** This pytest case verifies sequence parallelism pass requires full graph compilation. the expected failure path is asserted explicitly.
**CN:** 该 pytest 用例验证 sequence parallelism pass requires full graph compilation 的行为。 代码会显式断言预期的失败路径。

### Function `sequence_parallelism_pass_on_test_model` (lines 235-343)
```python
def sequence_parallelism_pass_on_test_model(
    local_rank: int,
    world_size: int,
    test_model_cls: type[torch.nn.Module],
    custom_ops: str,
    batch_size: int,
    seq_len: int,
    hidden_size: int,
    dtype: torch.dtype,
    fuse_norm_quant: bool,
    dynamic: bool,
):
    set_random_seed(0)

    device = torch.device(f"{DEVICE_TYPE}:{local_rank}")
    torch.accelerator.set_device_index(device)
    torch.set_default_device(device)
    torch.set_default_dtype(dtype)

    update_environment_variables(
        {
            "RANK": str(local_rank),
            "LOCAL_RANK": str(local_rank),
            "WORLD_SIZE": str(world_size),
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": "12345",
        }
    )

    # initialize distributed
    init_distributed_environment()

    # configure vllm config for SequenceParallelismPass
    custom_ops_list = custom_ops.split(",") if custom_ops else []
    compilation_config = CompilationConfig(
        splitting_ops=[],  # avoid automatic rms_norm enablement
        cudagraph_mode=CUDAGraphMode.NONE,  # avoid piecewise warnings
        custom_ops=custom_ops_list,
        pass_config=PassConfig(
            enable_sp=True,
# ... excerpt ...
            == vllm_config.compilation_config.use_inductor_graph_partition
        )
        passes_for_backend: list[VllmInductorPass] = [
            noop_pass,
            sequence_parallelism_pass,
        ]

        if fuse_norm_quant:
            fusion_pass = RMSNormQuantFusionPass(vllm_config)
            passes_for_backend.append(fusion_pass)

        passes_for_backend.append(cleanup_pass)

        backend = TestBackend(*passes_for_backend)

        model = test_model_cls(hidden_size)

        hidden_states = torch.randn((batch_size * seq_len, hidden_size), dtype=dtype)

        if dynamic:
            torch._dynamo.mark_dynamic(hidden_states, 0)

        compiled_model = torch.compile(model, backend=backend)
        compiled_model(hidden_states)

        assert sequence_parallelism_pass.matched_count == 4

        # In pre-nodes, all reduce should be there,
        # reduce scatter and all gather should not
        for op in model.ops_in_model_before():
            assert backend.op_count(op, before=True) == 4

        # In post-nodes, reduce scatter and all gather should be there,
        # all reduce should not
        for op in model.ops_in_model_after():
            assert backend.op_count(op, before=False) == 4

        for op in model.ops_in_model():
            assert backend.op_count(op, before=False) > 0
```
**EN:** This helper function implements the shared logic for sequence parallelism pass on test model. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 sequence parallelism pass on test model 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.envs`
- `tests.compile.backend -> TestBackend`
- `tests.utils -> TestFP8Layer, multi_gpu_test`
- `vllm.compilation.passes.fusion.rms_quant_fusion -> RMSNormQuantFusionPass`
- `vllm.compilation.passes.fusion.sequence_parallelism -> SequenceParallelismPass`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.compilation.passes.vllm_inductor_pass -> VllmInductorPass`
- `vllm.config -> CompilationConfig, CUDAGraphMode, DeviceConfig, ModelConfig, PassConfig, VllmConfig, get_current_vllm_config, set_current_vllm_config`
- `vllm.config.utils -> Range`
- `vllm.distributed -> tensor_model_parallel_all_reduce`
- `vllm.distributed.parallel_state -> init_distributed_environment, initialize_model_parallel`
- `vllm.model_executor.layers.layernorm -> RMSNorm`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> kFp8StaticTensorSym`
