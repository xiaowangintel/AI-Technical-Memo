# test_async_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/distributed/test_async_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / distributed / test_async_tp, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / distributed / test_async_tp 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-33)
```python
import pytest
import torch

import vllm.envs as envs
from tests.compile.backend import TestBackend
from tests.utils import (
    multi_gpu_test,
)
from vllm.compilation.passes.fusion.collective_fusion import AsyncTPPass
from vllm.config import (
    CompilationConfig,
    DeviceConfig,
    ModelConfig,
    PassConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.config.utils import Range
from vllm.distributed import (
    tensor_model_parallel_all_gather,
    tensor_model_parallel_reduce_scatter,
)
from vllm.distributed.parallel_state import (
    init_distributed_environment,
    initialize_model_parallel,
)
from vllm.platforms import current_platform
from vllm.utils.system_utils import update_environment_variables
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend, tests.utils; and vLLM components like vllm.envs, vllm.compilation.passes.fusion.collective_fusion, vllm.config, vllm.config.utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.utils；vLLM 内部组件，例如 vllm.envs、vllm.compilation.passes.fusion.collective_fusion、vllm.config、vllm.config.utils。

### Constants and module state (lines 35-43)
```python
DEVICE_TYPE = current_platform.device_type
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

### Class `TestMMRSModel` (lines 46-46)
```python
class TestMMRSModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestMMRSModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestMMRSModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestMMRSModel.__init__` (lines 47-55)
```python
    def __init__(self, hidden_size=16, dtype=torch.float16):
        super().__init__()
        self.hidden_size = hidden_size
        self.dtype = dtype
        self.gate_proj = torch.nn.Parameter(
            torch.empty((self.hidden_size * 2, hidden_size)), requires_grad=False
        )
        # Initialize weights
        torch.nn.init.normal_(self.gate_proj, std=0.02)
```
**EN:** This method implements the initialization for `TestMMRSModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestMMRSModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMMRSModel.forward` (lines 57-69)
```python
    def forward(self, hidden_states):
        """
        Forward pass implementing the mm + reduce scatter in the FX graph

        """
        # Reshape input
        view = hidden_states.reshape(-1, self.hidden_size)

        # matrix multiplication
        permute = self.gate_proj.permute(1, 0)
        mm = torch.mm(view, permute)
        reduce_scatter = tensor_model_parallel_reduce_scatter(mm, dim=0)
        return reduce_scatter
```
**EN:** This method on `TestMMRSModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMMRSModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMMRSModel.ops_in_model_before` (lines 71-72)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.reduce_scatter.default]
```
**EN:** This method on `TestMMRSModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMMRSModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMMRSModel.ops_in_model_after` (lines 74-75)
```python
    def ops_in_model_after(self):
        return [torch.ops.symm_mem.fused_matmul_reduce_scatter.default]
```
**EN:** This method on `TestMMRSModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMMRSModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAGMMModel` (lines 78-78)
```python
class TestAGMMModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestAGMMModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAGMMModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAGMMModel.__init__` (lines 79-87)
```python
    def __init__(self, hidden_size=16, dtype=torch.float16):
        super().__init__()
        self.hidden_size = hidden_size
        self.dtype = dtype
        self.weight = torch.nn.Parameter(
            torch.empty((hidden_size, hidden_size)), requires_grad=False
        )
        # Initialize weights
        torch.nn.init.normal_(self.weight, std=0.02)
```
**EN:** This method implements the initialization for `TestAGMMModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAGMMModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGMMModel.forward` (lines 89-98)
```python
    def forward(self, hidden_states):
        """
        Forward pass implementing the mm + all gather in the FX graph
        """
        # Reshape input
        view = hidden_states.reshape(-1, self.hidden_size)
        all_gather = tensor_model_parallel_all_gather(view, dim=0)
        permute = self.weight.permute(1, 0)
        mm = torch.mm(all_gather, permute)
        return mm
```
**EN:** This method on `TestAGMMModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGMMModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGMMModel.ops_in_model_before` (lines 100-101)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.all_gather.default]
```
**EN:** This method on `TestAGMMModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGMMModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGMMModel.ops_in_model_after` (lines 103-104)
```python
    def ops_in_model_after(self):
        return [torch.ops.symm_mem.fused_all_gather_matmul.default]
```
**EN:** This method on `TestAGMMModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGMMModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `_BaseScaledMMModel` (lines 107-107)
```python
class _BaseScaledMMModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for BaseScaledMMModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 BaseScaledMMModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `_BaseScaledMMModel.__init__` (lines 108-119)
```python
    def __init__(self, hidden_size=16, dtype=torch.float16):
        super().__init__()
        self.hidden_size = hidden_size
        self.dtype = dtype
        self.weight = (
            torch.empty([hidden_size, hidden_size], dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )

        # Initialize scale_b for _scaled_mm.
        self.scale_b = torch.ones(1, self.hidden_size, dtype=torch.float32)
```
**EN:** This method implements the initialization for `_BaseScaledMMModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `_BaseScaledMMModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestScaledMMRSModel` (lines 122-122)
```python
class TestScaledMMRSModel(_BaseScaledMMModel):
```
**EN:** This helper class groups the state and behavior needed for TestScaledMMRSModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestScaledMMRSModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestScaledMMRSModel.forward` (lines 123-138)
```python
    def forward(self, input: torch.Tensor):
        """
        Forward pass implementing the scaled_mm + reduce scatter in the FX graph

        """
        fp8_input = input.to(FP8_DTYPE)
        scale_a = torch.ones(input.shape[0], 1, dtype=torch.float32)
        scaled_mm = torch._scaled_mm(
            fp8_input,
            self.weight,
            scale_a=scale_a,
            scale_b=self.scale_b,
            out_dtype=self.dtype,
        )
        reduce_scatter = tensor_model_parallel_reduce_scatter(scaled_mm, dim=0)
        return reduce_scatter
```
**EN:** This method on `TestScaledMMRSModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestScaledMMRSModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestScaledMMRSModel.ops_in_model_before` (lines 140-141)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.reduce_scatter.default]
```
**EN:** This method on `TestScaledMMRSModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestScaledMMRSModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestScaledMMRSModel.ops_in_model_after` (lines 143-144)
```python
    def ops_in_model_after(self):
        return [torch.ops.vllm.patched_fused_scaled_matmul_reduce_scatter.default]
```
**EN:** This method on `TestScaledMMRSModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestScaledMMRSModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAGScaledMMModel` (lines 147-147)
```python
class TestAGScaledMMModel(_BaseScaledMMModel):
```
**EN:** This helper class groups the state and behavior needed for TestAGScaledMMModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAGScaledMMModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAGScaledMMModel.forward` (lines 148-164)
```python
    def forward(self, input: torch.Tensor):
        """
        Forward pass implementing the all gather + scaled_mm in the FX graph
        """
        # Reshape input
        fp8_input = input.to(FP8_DTYPE)
        all_gather = tensor_model_parallel_all_gather(fp8_input, dim=0)

        scale_a = torch.ones(all_gather.shape[0], 1, dtype=torch.float32)
        scaled_mm = torch._scaled_mm(
            all_gather,
            self.weight,
            scale_a=scale_a,
            scale_b=self.scale_b,
            out_dtype=self.dtype,
        )
        return scaled_mm
```
**EN:** This method on `TestAGScaledMMModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGScaledMMModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGScaledMMModel.ops_in_model_before` (lines 166-167)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.all_gather.default]
```
**EN:** This method on `TestAGScaledMMModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGScaledMMModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGScaledMMModel.ops_in_model_after` (lines 169-170)
```python
    def ops_in_model_after(self):
        return [torch.ops.symm_mem.fused_all_gather_scaled_matmul.default]
```
**EN:** This method on `TestAGScaledMMModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGScaledMMModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestCutlassScaledMMRSModel` (lines 173-173)
```python
class TestCutlassScaledMMRSModel(_BaseScaledMMModel):
```
**EN:** This helper class groups the state and behavior needed for TestCutlassScaledMMRSModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestCutlassScaledMMRSModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestCutlassScaledMMRSModel.forward` (lines 174-191)
```python
    def forward(self, input: torch.Tensor):
        """
        Forward pass implementing the cutlass_scaled_mm + reduce scatter
        in the FX graph

        """
        fp8_input = input.to(FP8_DTYPE)
        scale_a = torch.ones(input.shape[0], 1, dtype=torch.float32)
        mm_out = torch.empty(
            (fp8_input.shape[0], self.weight.shape[1]),
            dtype=self.dtype,
            device=input.device,
        )
        torch.ops._C.cutlass_scaled_mm(
            mm_out, fp8_input, self.weight, scale_a, self.scale_b, None
        )
        reduce_scatter = tensor_model_parallel_reduce_scatter(mm_out, dim=0)
        return reduce_scatter
```
**EN:** This method on `TestCutlassScaledMMRSModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestCutlassScaledMMRSModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestCutlassScaledMMRSModel.ops_in_model_before` (lines 193-194)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.reduce_scatter.default]
```
**EN:** This method on `TestCutlassScaledMMRSModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestCutlassScaledMMRSModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestCutlassScaledMMRSModel.ops_in_model_after` (lines 196-197)
```python
    def ops_in_model_after(self):
        return [torch.ops.vllm.patched_fused_scaled_matmul_reduce_scatter.default]
```
**EN:** This method on `TestCutlassScaledMMRSModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestCutlassScaledMMRSModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAGCutlassScaledMMModel` (lines 200-200)
```python
class TestAGCutlassScaledMMModel(_BaseScaledMMModel):
```
**EN:** This helper class groups the state and behavior needed for TestAGCutlassScaledMMModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAGCutlassScaledMMModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAGCutlassScaledMMModel.forward` (lines 201-220)
```python
    def forward(self, input: torch.Tensor):
        """
        Forward pass implementing the all gather + cutlass_scaled_mm
        in the FX graph
        """
        # Reshape input
        fp8_input = input.to(FP8_DTYPE)
        all_gather = tensor_model_parallel_all_gather(fp8_input, dim=0)

        scale_a = torch.ones(all_gather.shape[0], 1, dtype=torch.float32)

        mm_out = torch.empty(
            (all_gather.shape[0], self.weight.shape[1]),
            dtype=self.dtype,
            device=all_gather.device,
        )
        torch.ops._C.cutlass_scaled_mm(
            mm_out, all_gather, self.weight, scale_a, self.scale_b, None
        )
        return mm_out
```
**EN:** This method on `TestAGCutlassScaledMMModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGCutlassScaledMMModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGCutlassScaledMMModel.ops_in_model_before` (lines 222-223)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.all_gather.default]
```
**EN:** This method on `TestAGCutlassScaledMMModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGCutlassScaledMMModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAGCutlassScaledMMModel.ops_in_model_after` (lines 225-226)
```python
    def ops_in_model_after(self):
        return [torch.ops.symm_mem.fused_all_gather_scaled_matmul.default]
```
**EN:** This method on `TestAGCutlassScaledMMModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAGCutlassScaledMMModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_async_tp_pass_replace` (lines 229-289)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "test_model",
    [
        TestMMRSModel,
        TestAGMMModel,
        TestScaledMMRSModel,
        TestAGScaledMMModel,
        TestCutlassScaledMMRSModel,
        TestAGCutlassScaledMMModel,
    ],
)
@pytest.mark.parametrize("batch_size", [8])
@pytest.mark.parametrize("seq_len", [16])
@pytest.mark.parametrize("hidden_size", [16])
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
@pytest.mark.parametrize("dynamic", [True, False])
@pytest.mark.skipif(envs.VLLM_TARGET_DEVICE not in ["cuda"], reason="Only test on CUDA")
def test_async_tp_pass_replace(
    test_model: str,
    batch_size: int,
    seq_len: int,
    hidden_size: int,
    dtype: torch.dtype,
    dynamic: bool,
):
    if (
        test_model
        in (
            TestScaledMMRSModel,
            TestAGScaledMMModel,
            TestCutlassScaledMMRSModel,
            TestAGCutlassScaledMMModel,
        )
        and dtype == torch.float16
    ):
        pytest.skip(
            "Only bf16 high precision output types are supported for "
            "per-token (row-wise) scaling"
        )

    num_processes = 2

    def run_torch_spawn(fn, nprocs):
        # need to use torch.mp.spawn otherwise will have problems with
        # torch.distributed and cuda
        torch.multiprocessing.spawn(
            fn,
            args=(
                num_processes,
                test_model,
                batch_size,
                seq_len,
                hidden_size,
                dtype,
                dynamic,
            ),
            nprocs=nprocs,
        )

    run_torch_spawn(async_tp_pass_on_test_model, num_processes)
```
**EN:** This pytest case verifies async tp pass replace. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as test_model, batch_size, seq_len, hidden_size. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 async tp pass replace 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 test_model、batch_size、seq_len、hidden_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_async_tp_pass_requires_full_graph_compilation` (lines 292-305)
```python
def test_async_tp_pass_requires_full_graph_compilation():
    vllm_config = VllmConfig()
    vllm_config.compilation_config.use_inductor_graph_partition = False
    vllm_config.compilation_config.splitting_ops = [
        "vllm::unified_attention_with_output"
    ]

    async_tp_pass = object.__new__(AsyncTPPass)
    async_tp_pass.compilation_config = vllm_config.compilation_config

    with pytest.raises(
        AssertionError, match="AsyncTPPass requires full-graph compilation"
    ):
        async_tp_pass.is_applicable_for_range(Range(start=8, end=8))
```
**EN:** This pytest case verifies async tp pass requires full graph compilation. the expected failure path is asserted explicitly.
**CN:** 该 pytest 用例验证 async tp pass requires full graph compilation 的行为。 代码会显式断言预期的失败路径。

### Function `async_tp_pass_on_test_model` (lines 308-389)
```python
def async_tp_pass_on_test_model(
    local_rank: int,
    world_size: int,
    test_model_cls: torch.nn.Module,
    batch_size: int,
    seq_len: int,
    hidden_size: int,
    dtype: torch.dtype,
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
    vllm_config = VllmConfig()
    vllm_config.compilation_config = CompilationConfig(
        pass_config=PassConfig(
            fuse_gemm_comms=True,
        ),
    )
    vllm_config.device_config = DeviceConfig(device=torch.device(DEVICE_TYPE))

    # this is a fake model name to construct the model config
# ... excerpt ...
        model=model_name, trust_remote_code=True, dtype=dtype, seed=42
    )

    with set_current_vllm_config(vllm_config):
        initialize_model_parallel(tensor_model_parallel_size=world_size)

        async_tp_pass = AsyncTPPass(vllm_config)
        backend = TestBackend(async_tp_pass)

        assert (
            async_tp_pass.compilation_config.splitting_ops
            == vllm_config.compilation_config.splitting_ops
        )
        assert (
            async_tp_pass.compilation_config.use_inductor_graph_partition
            == vllm_config.compilation_config.use_inductor_graph_partition
        )

        model = test_model_cls(hidden_size, dtype)  # Pass dtype to model constructor

        hidden_states = torch.randn(
            (batch_size * seq_len, hidden_size), dtype=dtype, requires_grad=False
        )

        if dynamic:
            torch._dynamo.mark_dynamic(hidden_states, 0)

        compiled_model = torch.compile(model, backend=backend)
        compiled_model(hidden_states)

        assert async_tp_pass.matched_count == 1

        # In pre-nodes, all gather or reduce scatter should exist,
        # fused_matmul_reduce_scatter or fused_all_gather_matmul should not
        backend.check_before_ops(model.ops_in_model_before(), fully_replaced=False)

        # In post-nodes, fused_matmul_reduce_scatter or \
        # fused_all_gather_matmul should exist
        backend.check_after_ops(model.ops_in_model_after())
```
**EN:** This helper function implements the shared logic for async tp pass on test model. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 async tp pass on test model 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

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
- `tests.utils -> multi_gpu_test`
- `vllm.compilation.passes.fusion.collective_fusion -> AsyncTPPass`
- `vllm.config -> CompilationConfig, DeviceConfig, ModelConfig, PassConfig, VllmConfig, set_current_vllm_config`
- `vllm.config.utils -> Range`
- `vllm.distributed -> tensor_model_parallel_all_gather, tensor_model_parallel_reduce_scatter`
- `vllm.distributed.parallel_state -> init_distributed_environment, initialize_model_parallel`
- `vllm.platforms -> current_platform`
- `vllm.utils.system_utils -> update_environment_variables`
- `vllm.utils.torch_utils -> set_random_seed`
