# test_fusion_all_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/distributed/test_fusion_all_reduce.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / distributed / test_fusion_all_reduce, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / distributed / test_fusion_all_reduce 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-42)
```python
from importlib.util import find_spec

import pytest
import torch

import vllm.envs as envs
from tests.compile.backend import TestBackend
from tests.utils import TestFP8Layer, has_module_attribute, multi_gpu_test
from vllm._aiter_ops import IS_AITER_FOUND, rocm_aiter_ops
from vllm._custom_ops import cutlass_scaled_fp4_mm, scaled_fp4_quant
from vllm.compilation.passes.fusion.allreduce_rms_fusion import (
    AllReduceFusionPass,
    RocmAiterAllReduceFusionPass,
)
from vllm.compilation.passes.utility.fix_functionalization import (
    FixFunctionalizationPass,
)
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    DeviceConfig,
    ModelConfig,
    PassConfig,
    VllmConfig,
    set_current_vllm_config,
)
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
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as importlib.util, pytest, torch; shared test helpers from tests.compile.backend, tests.utils; and vLLM components like vllm.envs, vllm._aiter_ops, vllm._custom_ops, vllm.compilation.passes.fusion.allreduce_rms_fusion.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 importlib.util、pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.utils；vLLM 内部组件，例如 vllm.envs、vllm._aiter_ops、vllm._custom_ops、vllm.compilation.passes.fusion.allreduce_rms_fusion。

### Constants and module state (lines 44-44)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE_TYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE_TYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TestAllReduceRMSNormModel` (lines 47-47)
```python
class TestAllReduceRMSNormModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestAllReduceRMSNormModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAllReduceRMSNormModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAllReduceRMSNormModel.__init__` (lines 48-61)
```python
    def __init__(
        self,
        hidden_size=16,
        token_num=16,
        eps=1e-6,
        dtype: torch.dtype = torch.float16,
        use_aiter: bool = False,
    ):
        super().__init__()
        self.hidden_size = hidden_size
        self.eps = eps
        self.norm = [RMSNorm(hidden_size, eps) for i in range(4)]
        self.w = [torch.rand(hidden_size, hidden_size) for _ in range(3)]
        self.use_aiter = use_aiter
```
**EN:** This method implements the initialization for `TestAllReduceRMSNormModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAllReduceRMSNormModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormModel.forward` (lines 63-83)
```python
    def forward(self, x):
        # avoid having graph input be an arg to a pattern directly
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

### Method `TestAllReduceRMSNormModel.ops_in_model_before` (lines 85-86)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm.all_reduce.default]
```
**EN:** This method on `TestAllReduceRMSNormModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormModel.ops_in_model_after` (lines 88-91)
```python
    def ops_in_model_after(self):
        if self.use_aiter:
            return [rocm_aiter_ops.get_fused_allreduce_rmsnorm_op()]
        return [torch.ops.vllm.flashinfer_trtllm_fused_allreduce_norm.default]
```
**EN:** This method on `TestAllReduceRMSNormModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAllReduceRMSNormStaticQuantFP8Model` (lines 94-96)
```python
class TestAllReduceRMSNormStaticQuantFP8Model(torch.nn.Module):
    quant_key = kFp8StaticTensorSym
```
**EN:** This helper class groups the state and behavior needed for TestAllReduceRMSNormStaticQuantFP8Model. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAllReduceRMSNormStaticQuantFP8Model 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.__init__` (lines 97-112)
```python
    def __init__(
        self, hidden_size=16, token_num=16, eps=1e-6, dtype: torch.dtype = torch.float16
    ):
        super().__init__()
        self.hidden_size = hidden_size
        self.eps = eps
        self.norm = [RMSNorm(hidden_size, eps) for i in range(4)]
        self.fp8_linear_layers = [
            TestFP8Layer(
                weight_shape=(hidden_size, hidden_size),
                activation_quant_key=self.quant_key,
                weight_quant_key=self.quant_key,
                input_dtype=dtype,
            )
            for i in range(3)
        ]
```
**EN:** This method implements the initialization for `TestAllReduceRMSNormStaticQuantFP8Model`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAllReduceRMSNormStaticQuantFP8Model` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.forward` (lines 114-134)
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

### Method `TestAllReduceRMSNormStaticQuantFP8Model.ops_in_model_after` (lines 136-137)
```python
    def ops_in_model_after(self):
        return [torch.ops.vllm.flashinfer_trtllm_fused_allreduce_norm.default]
```
**EN:** This method on `TestAllReduceRMSNormStaticQuantFP8Model` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormStaticQuantFP8Model` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceRMSNormStaticQuantFP8Model.ops_in_model_before` (lines 139-145)
```python
    def ops_in_model_before(self):
        return [
            torch.ops.vllm.all_reduce.default,
            torch.ops._C.static_scaled_fp8_quant.default
            if self.fp8_linear_layers[0].is_quant_fp8_enabled()
            else torch.ops.aten.reciprocal.default,
        ]
```
**EN:** This method on `TestAllReduceRMSNormStaticQuantFP8Model` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceRMSNormStaticQuantFP8Model` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` (lines 148-148)
```python
class TestAllReduceFusedAddRMSNormStaticQuantFP4Model(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestAllReduceFusedAddRMSNormStaticQuantFP4Model. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestAllReduceFusedAddRMSNormStaticQuantFP4Model 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestAllReduceFusedAddRMSNormStaticQuantFP4Model.__init__` (lines 149-165)
```python
    def __init__(
        self, hidden_size=16, token_num=16, eps=1e-6, dtype: torch.dtype = torch.float16
    ):
        super().__init__()
        self.hidden_size = hidden_size
        self.eps = eps
        self.norm = [RMSNorm(hidden_size, eps) for i in range(4)]

        self.w = [torch.rand(hidden_size, hidden_size) for _ in range(3)]
        self.agscale = [torch.rand(1, dtype=torch.float32) for _ in range(3)]
        wgscale = [torch.rand(1, dtype=torch.float32) for _ in range(3)]
        self.alpha = [1 / (w * a) for w, a in zip(wgscale, self.agscale)]

        wq_gen, wscale_gen = zip(
            *(scaled_fp4_quant(w, wg) for w, wg in zip(self.w, wgscale))
        )
        self.wq, self.wscale = list(wq_gen), list(wscale_gen)
```
**EN:** This method implements the initialization for `TestAllReduceFusedAddRMSNormStaticQuantFP4Model`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceFusedAddRMSNormStaticQuantFP4Model.forward` (lines 167-195)
```python
    def forward(self, hidden_states):
        # avoid having graph input be an arg to a pattern directly
        z = torch.relu(hidden_states)
        x = resid = tensor_model_parallel_all_reduce(z)
        y = self.norm[0](x)

        yq, y_scale = scaled_fp4_quant(y, self.agscale[0])
        z2 = cutlass_scaled_fp4_mm(
            yq, self.wq[0], y_scale, self.wscale[0], self.alpha[0], out_dtype=y.dtype
        )

        x2 = tensor_model_parallel_all_reduce(z2)
        y2, resid = self.norm[1](x2, resid)

        yq2, y_scale2 = scaled_fp4_quant(y2, self.agscale[1])
        z3 = cutlass_scaled_fp4_mm(
            yq2, self.wq[1], y_scale2, self.wscale[1], self.alpha[1], out_dtype=y2.dtype
        )

        x3 = tensor_model_parallel_all_reduce(z3)
        y3, resid = self.norm[2](x3, resid)  # use resid here

        yq3, y_scale3 = scaled_fp4_quant(y3, self.agscale[2])
        z4 = cutlass_scaled_fp4_mm(
            yq3, self.wq[2], y_scale3, self.wscale[2], self.alpha[2], out_dtype=y3.dtype
        )
        x4 = tensor_model_parallel_all_reduce(z4)
        y4, resid = self.norm[3](x4, resid)  # use resid here
        return y4
```
**EN:** This method on `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceFusedAddRMSNormStaticQuantFP4Model.ops_in_model_after` (lines 197-198)
```python
    def ops_in_model_after(self):
        return [torch.ops.vllm.flashinfer_trtllm_fused_allreduce_norm.default]
```
**EN:** This method on `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestAllReduceFusedAddRMSNormStaticQuantFP4Model.ops_in_model_before` (lines 200-204)
```python
    def ops_in_model_before(self):
        return [
            torch.ops.vllm.all_reduce.default,
            torch.ops._C.scaled_fp4_quant.out,
        ]
```
**EN:** This method on `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestAllReduceFusedAddRMSNormStaticQuantFP4Model` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_all_reduce_fusion_pass_replace` (lines 207-310)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "test_model, enable_quant_fp8_custom_op, use_aiter",
    [
        (TestAllReduceRMSNormModel, False, IS_AITER_FOUND),
        pytest.param(
            TestAllReduceRMSNormStaticQuantFP8Model,
            True,
            False,
            marks=pytest.mark.skipif(
                current_platform.is_rocm(),
                reason="Not supported on ROCm platform",
            ),
        ),
        pytest.param(
            TestAllReduceRMSNormStaticQuantFP8Model,
            False,
            False,
            marks=pytest.mark.skipif(
                current_platform.is_rocm(),
                reason="Not supported on ROCm platform",
            ),
        ),
        pytest.param(
            TestAllReduceFusedAddRMSNormStaticQuantFP4Model,
            False,
            False,
            marks=pytest.mark.skipif(
                current_platform.is_rocm(),
                reason="Not supported on ROCm platform",
            ),
        ),
    ],
)
@pytest.mark.parametrize("batch_size", [8])
@pytest.mark.parametrize("seq_len", [8])
@pytest.mark.parametrize("hidden_size", [64])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("enable_rms_norm_custom_op", [True, False])
@pytest.mark.parametrize("flashinfer_allreduce_backend", ["trtllm", "mnnvl"])
# ... excerpt ...
    flashinfer_allreduce_backend,
    use_aiter: bool,
    monkeypatch: pytest.MonkeyPatch,
):
    if use_aiter:
        with monkeypatch.context() as m:
            m.setenv("VLLM_ROCM_USE_AITER", str(use_aiter))
            rocm_aiter_ops.refresh_env_variables()

    num_processes = 2
    if (
        test_model == TestAllReduceFusedAddRMSNormStaticQuantFP4Model
        and not current_platform.has_device_capability(100)
    ):
        pytest.skip(
            "Skip as nvfp4 is only supported on "
            "devices with compute capability 10.0 (Blackwell)"
        )

    def run_torch_spawn(fn, nprocs):
        torch.multiprocessing.spawn(
            fn,
            args=(
                num_processes,
                test_model,
                batch_size,
                seq_len,
                hidden_size,
                dtype,
                enable_rms_norm_custom_op,
                enable_quant_fp8_custom_op,
                flashinfer_allreduce_backend,
                use_aiter,
                monkeypatch,
            ),
            nprocs=nprocs,
        )

    run_torch_spawn(all_reduce_fusion_pass_on_test_model, num_processes)
```
**EN:** This pytest case verifies all reduce fusion pass replace. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as test_model, batch_size, seq_len, hidden_size. unsupported hardware, backend, or configuration combinations are skipped early. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 all reduce fusion pass replace 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 test_model、batch_size、seq_len、hidden_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。 由于该代码块较大，这里只展示关键片段。

### Function `all_reduce_fusion_pass_on_test_model` (lines 313-407)
```python
def all_reduce_fusion_pass_on_test_model(
    local_rank: int,
    world_size: int,
    test_model_cls: torch.nn.Module,
    batch_size: int,
    seq_len: int,
    hidden_size: int,
    dtype: torch.dtype,
    enable_rms_norm_custom_op,
    enable_quant_fp8_custom_op,
    flashinfer_allreduce_backend,
    use_aiter: bool,
    monkeypatch: pytest.MonkeyPatch,
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
            "VLLM_FLASHINFER_ALLREDUCE_BACKEND": flashinfer_allreduce_backend,
        }
    )

    init_distributed_environment()

    custom_ops = []
    if enable_rms_norm_custom_op:
        custom_ops.append("+rms_norm")
    if enable_quant_fp8_custom_op:
        custom_ops.append("+quant_fp8")

# ... excerpt ...
    )
    with set_current_vllm_config(vllm_config):
        initialize_model_parallel(tensor_model_parallel_size=world_size)
        all_reduce_fusion_pass = (
            RocmAiterAllReduceFusionPass(vllm_config)
            if use_aiter
            else AllReduceFusionPass(vllm_config)
        )
        noop_pass = NoOpEliminationPass(vllm_config)
        func_pass = FixFunctionalizationPass(vllm_config)
        cleanup_pass = PostCleanupPass(vllm_config)

        backend = TestBackend(
            noop_pass, all_reduce_fusion_pass, func_pass, cleanup_pass
        )

        token_num = batch_size * seq_len
        if test_model_cls is TestAllReduceRMSNormModel:
            model = test_model_cls(
                hidden_size, token_num, dtype=dtype, use_aiter=use_aiter
            )
        else:
            model = test_model_cls(hidden_size, token_num, dtype=dtype)

        hidden_states = torch.randn((token_num, hidden_size), requires_grad=False)

        compiled_model = torch.compile(model, backend=backend)
        compiled_model(hidden_states)

        results_unfused = model(hidden_states)
        results_fused = compiled_model(hidden_states)
        torch.testing.assert_close(results_unfused, results_fused, atol=1e-2, rtol=1e-2)

        assert all_reduce_fusion_pass.matched_count == 4, (
            f"{all_reduce_fusion_pass.matched_count=}"
        )
        backend.check_before_ops(model.ops_in_model_before(), fully_replaced=False)
        backend.check_after_ops(model.ops_in_model_after())
        del all_reduce_fusion_pass
```
**EN:** This helper function implements the shared logic for all reduce fusion pass on test model. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 all reduce fusion pass on test model 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `importlib.util -> find_spec`
- `pytest`
- `torch`
- `vllm.envs`
- `tests.compile.backend -> TestBackend`
- `tests.utils -> TestFP8Layer, has_module_attribute, multi_gpu_test`
- `vllm._aiter_ops -> IS_AITER_FOUND, rocm_aiter_ops`
- `vllm._custom_ops -> cutlass_scaled_fp4_mm, scaled_fp4_quant`
- `vllm.compilation.passes.fusion.allreduce_rms_fusion -> AllReduceFusionPass, RocmAiterAllReduceFusionPass`
- `vllm.compilation.passes.utility.fix_functionalization -> FixFunctionalizationPass`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> CompilationConfig, CompilationMode, DeviceConfig, ModelConfig, PassConfig, VllmConfig, set_current_vllm_config`
- `vllm.distributed -> tensor_model_parallel_all_reduce`
- `vllm.distributed.parallel_state -> init_distributed_environment, initialize_model_parallel`
- `vllm.model_executor.layers.layernorm -> RMSNorm`
